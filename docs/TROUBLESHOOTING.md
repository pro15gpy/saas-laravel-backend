# Troubleshooting Guide

## Laravel SaaS Boilerplate - TALL Stack

**Version:** 2.0  
**Last Updated:** 2025-01-12

---

## Table of Contents

1. [Installation Issues](#installation-issues)
2. [Database Issues](#database-issues)
3. [Authentication Issues](#authentication-issues)
4. [Billing & Payment Issues](#billing--payment-issues)
5. [Multi-Tenancy Issues](#multi-tenancy-issues)
6. [Notification Issues](#notification-issues)
7. [Frontend Issues](#frontend-issues)
8. [Performance Issues](#performance-issues)
9. [Deployment Issues](#deployment-issues)
10. [Common Error Messages](#common-error-messages)

---

## Installation Issues

### Issue: Composer install fails

**Symptoms:**
```
Composer could not find a composer.json file in /path/to/project
```

**Solution:**
```bash
# Ensure you're in the correct directory
cd /path/to/laravel-saas-boilerplate

# Check if composer.json exists
ls -la composer.json

# If missing, re-clone the repository
git clone https://github.com/yourusername/laravel-saas-boilerplate.git
```

### Issue: PHP extension missing

**Symptoms:**
```
The requested PHP extension ext-redis * is missing from your system.
```

**Solution:**
```bash
# Install Redis extension
# Ubuntu/Debian
sudo apt-get install php-redis

# macOS (Homebrew)
brew install php@8.3-redis

# Verify installation
php -m | grep redis
```

### Issue: npm install fails

**Symptoms:**
```
npm ERR! code ENOENT
npm ERR! syscall open
npm ERR! path /package.json
```

**Solution:**
```bash
# Clear npm cache
npm cache clean --force

# Remove node_modules and reinstall
rm -rf node_modules package-lock.json
npm install

# If still failing, try with legacy peer deps
npm install --legacy-peer-deps
```

---

## Database Issues

### Issue: Migration fails

**Symptoms:**
```
SQLSTATE[HY000] [2002] Connection refused
```

**Solution:**
```bash
# Check if database server is running
# MySQL
sudo systemctl status mysql

# PostgreSQL
sudo systemctl status postgresql

# Start if not running
sudo systemctl start mysql

# Verify connection
mysql -u root -p -e "SHOW DATABASES;"
```

### Issue: Duplicate entry error during migration

**Symptoms:**
```
SQLSTATE[23000]: Integrity constraint violation: 1062 Duplicate entry
```

**Solution:**
```bash
# Fresh migration (WARNING: deletes all data)
php artisan migrate:fresh --seed

# OR rollback and re-run
php artisan migrate:rollback
php artisan migrate
```

### Issue: Tenant database not created

**Symptoms:**
```
Database 'tenant_db' does not exist
```

**Solution:**
```php
// Check tenant configuration
config('tenant.database.isolation') // should be 'separate'

// Manually create tenant database
DB::statement("CREATE DATABASE tenant_{$tenant->id}");

// Or run tenant creation command
php artisan tenant:create
```

---

## Authentication Issues

### Issue: Login returns "Invalid credentials"

**Symptoms:**
User cannot login despite correct credentials

**Solution:**
```bash
# Check if user exists
php artisan tinker
>>> User::where('email', 'user@example.com')->first();

# Verify password hash
>>> Hash::check('password', $user->password);

# Reset password if needed
>>> $user->password = Hash::make('newpassword');
>>> $user->save();
```

### Issue: Email verification not working

**Symptoms:**
```
Your email address is not verified.
```

**Solution:**
```bash
# Check mail configuration
php artisan about | grep Mail

# Test email sending
php artisan make:mail TestMail
php artisan tinker
>>> Mail::to('test@example.com')->send(new \App\Mail\TestMail());

# Manually verify user
>>> $user = User::find(1);
>>> $user->email_verified_at = now();
>>> $user->save();
```

### Issue: 2FA QR code not displaying

**Symptoms:**
QR code image broken or not showing

**Solution:**
```bash
# Check if google2fa package is installed
composer show | grep google2fa

# Verify secret is generated
>>> $user = User::find(1);
>>> $user->two_factor_secret; // should not be null

# Regenerate secret if needed
>>> $google2fa = new \PragmaRX\Google2FA\Google2FA();
>>> $secret = $google2fa->generateSecretKey();
>>> $user->two_factor_secret = encrypt($secret);
>>> $user->save();
```

---

## Billing & Payment Issues

### Issue: Stripe webhook not received

**Symptoms:**
```
Webhook signature verification failed
```

**Solution:**
```bash
# Get correct webhook secret from Stripe dashboard
# Stripe Dashboard > Developers > Webhooks > Signing Secret

# Update .env
STRIPE_WEBHOOK_SECRET=whsec_xxxxxxxxxxxxx

# Clear config cache
php artisan config:clear
php artisan config:cache

# Test webhook locally with Stripe CLI
stripe listen --forward-to localhost:8000/api/webhooks/stripe
```

### Issue: Subscription creation fails

**Symptoms:**
```
No attached payment method found for customer
```

**Solution:**
```php
// Ensure payment method is attached before subscription
$user->addPaymentMethod($paymentMethodId);
$user->updateDefaultPaymentMethod($paymentMethodId);

// Then create subscription
$user->newSubscription('default', $planId)->create($paymentMethodId);
```

### Issue: Invoice PDF not generating

**Symptoms:**
```
Class 'Dompdf\Dompdf' not found
```

**Solution:**
```bash
# Install DomPDF
composer require barryvdh/laravel-dompdf

# Publish config
php artisan vendor:publish --provider="Barryvdh\DomPDF\ServiceProvider"

# Check storage permissions
chmod -R 775 storage/app/invoices
```

---

## Multi-Tenancy Issues

### Issue: Tenant not identified

**Symptoms:**
```
Tenant could not be identified on subdomain
```

**Solution:**
```bash
# Check hosts file for local development
cat /etc/hosts
# Should have: 127.0.0.1 tenant1.localhost

# Verify tenant exists
>>> Tenant::where('subdomain', 'tenant1')->first();

# Check middleware registration
php artisan route:list | grep tenant
```

### Issue: Cross-tenant data leak

**Symptoms:**
Users can see data from other tenants

**Solution:**
```php
// Ensure global scope is applied
Model::addGlobalScope(new TenantScope);

// Check query in controller
// BEFORE (unsafe):
Team::all();

// AFTER (safe):
Team::query()->get(); // Global scope automatically applies
```

---

## Notification Issues

### Issue: Emails not being sent

**Symptoms:**
```
Connection could not be established with host smtp.example.com
```

**Solution:**
```bash
# Check mail configuration
php artisan about | grep Mail

# Test SMTP connection
telnet smtp.gmail.com 587

# Check queue worker
php artisan queue:work --tries=3

# View failed jobs
php artisan queue:failed
php artisan queue:retry <job-id>
```

### Issue: WhatsApp messages not delivered

**Symptoms:**
```
Twilio API error: Invalid phone number format
```

**Solution:**
```bash
# Ensure phone number is in E.164 format
# Correct: +1234567890
# Incorrect: 123-456-7890

// In code:
$phoneNumber = '+' . preg_replace('/[^0-9]/', '', $input);

# Verify Twilio credentials
TWILIO_SID=ACxxxxxxxxxxxxx
TWILIO_TOKEN=xxxxxxxxxxxxx
TWILIO_WHATSAPP_NUMBER=whatsapp:+14155238886
```

---

## Frontend Issues

### Issue: Livewire components not updating

**Symptoms:**
Clicking buttons doesn't trigger actions

**Solution:**
```bash
# Check if Livewire assets are loaded
curl http://localhost:8000 | grep livewire

# Clear view cache
php artisan view:clear
php artisan view:cache

# Check JavaScript console for errors
# Common issue: Alpine.js not loaded before Livewire
```

### Issue: Tailwind styles not applying

**Symptoms:**
Page looks unstyled

**Solution:**
```bash
# Rebuild assets
npm run dev

# Check tailwind.config.js content paths
module.exports = {
  content: [
    './resources/**/*.blade.php',
    './resources/**/*.js',
    './resources/**/*.vue',
  ],
}

# Clear browser cache
# Or add ?v=timestamp to CSS link
```

---

## Performance Issues

### Issue: Slow page load

**Symptoms:**
Pages take > 3 seconds to load

**Solution:**
```bash
# Enable query logging
DB::enableQueryLog();

# Check slow queries
Route::get('/debug', function() {
    return DB::getQueryLog();
});

# Add indexes to frequently queried columns
Schema::table('users', function(Blueprint $table) {
    $table->index('email');
    $table->index('tenant_id');
});

# Enable caching
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

### Issue: Queue jobs backing up

**Symptoms:**
```
Jobs table has 1000+ pending jobs
```

**Solution:**
```bash
# Check queue worker status
ps aux | grep queue:work

# Restart workers
php artisan queue:restart

# Scale workers
php artisan queue:work --tries=3 &
php artisan queue:work --tries=3 &

# Clear failed jobs
php artisan queue:flush
```

---

## Deployment Issues

### Issue: Production build fails

**Symptoms:**
```
npm ERR! Missing script: "build"
```

**Solution:**
```bash
# Use correct build command
npm run build

# For production
NODE_ENV=production npm run build

# Check Node version
node -v # should be 18.x or 20.x
```

### Issue: Permissions error on production

**Symptoms:**
```
Permission denied: storage/logs/laravel.log
```

**Solution:**
```bash
# Set correct permissions
sudo chown -R www-data:www-data storage bootstrap/cache
sudo chmod -R 775 storage bootstrap/cache

# Or use ACL
sudo setfacl -R -m u:www-data:rwX storage bootstrap/cache
```

---

## Common Error Messages

### "Class not found" errors

```bash
composer dump-autoload
php artisan optimize:clear
```

### "CSRF token mismatch"

```blade
<!-- Ensure form has @csrf directive -->
<form method="POST">
    @csrf
    <!-- form fields -->
</form>
```

### "419 Page Expired"

```bash
# Check session configuration
php artisan about | grep Session

# Increase session lifetime
SESSION_LIFETIME=120 # minutes
```

### "500 Internal Server Error"

```bash
# Check logs
tail -f storage/logs/laravel.log

# Enable debug mode temporarily
APP_DEBUG=true

# Check PHP error log
tail -f /var/log/php/error.log
```

### "404 Not Found" for routes

```bash
# List all routes
php artisan route:list

# Clear route cache
php artisan route:clear
php artisan route:cache

# Check route registration
routes/web.php
routes/api.php
```

---

## Getting Help

### Before Asking for Help

1. ✅ Check this troubleshooting guide
2. ✅ Search existing GitHub issues
3. ✅ Review error logs
4. ✅ Try fresh installation
5. ✅ Document steps to reproduce

### How to Report Issues

```markdown
**Description:**
Clear description of the problem

**Steps to Reproduce:**
1. Step 1
2. Step 2
3. Step 3

**Expected Behavior:**
What should happen

**Actual Behavior:**
What actually happens

**Environment:**
- PHP Version: 8.3.x
- Laravel Version: 11.x
- Database: MySQL 8.0
- OS: Ubuntu 22.04

**Logs:**
Paste relevant error logs

**Screenshots:**
If applicable
```

### Support Channels

- 📖 Documentation: `/docs`
- 🐛 GitHub Issues: https://github.com/yourusername/laravel-saas-boilerplate/issues
- 💬 Discord: https://discord.gg/yourlink
- 📧 Email: support@yoursaas.com

---

**Related Documentation:**
- [Installation Guide](INSTALLATION.md)
- [Configuration Guide](CONFIGURATION.md)
- [Security Guide](SECURITY.md)
