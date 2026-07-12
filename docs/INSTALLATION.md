# Installation Guide

## Laravel SaaS Boilerplate - TALL Stack

**Version:** 2.0  
**Last Updated:** 2025-01-12

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Quick Start](#quick-start)
3. [Detailed Installation](#detailed-installation)
4. [Module-Specific Setup](#module-specific-setup)
5. [Testing Installation](#testing-installation)
6. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### System Requirements

- **PHP:** 8.3 or higher
- **Composer:** 2.6+
- **Node.js:** 18.x or 20.x
- **npm:** 9.x or 10.x
- **Database:** MySQL 8.0+ / PostgreSQL 15+
- **Cache:** Redis 7.x (recommended)
- **Web Server:** Nginx / Apache with mod_rewrite

### PHP Extensions

Ensure the following PHP extensions are enabled:

```bash
php -m | grep -E "bcmath|ctype|curl|dom|fileinfo|json|mbstring|openssl|pdo|tokenizer|xml"
```

Required extensions:
- `bcmath`
- `ctype`
- `curl`
- `dom`
- `fileinfo`
- `json`
- `mbstring`
- `openssl`
- `pdo_mysql` or `pdo_pgsql`
- `tokenizer`
- `xml`

### Optional but Recommended

- `redis` extension for Redis cache
- `imagick` or `gd` for image processing
- `zip` for backup/export features

---

## Quick Start

### Step 1: Clone Repository

```bash
git clone https://github.com/yourusername/laravel-saas-boilerplate.git
cd laravel-saas-boilerplate
```

### Step 2: Install Dependencies

```bash
# Install PHP dependencies
composer install --no-interaction --prefer-dist --optimize-autoloader

# Install JavaScript dependencies
npm install
```

### Step 3: Environment Setup

```bash
# Copy environment file
cp .env.example .env

# Generate application key
php artisan key:generate
```

### Step 4: Database Configuration

Edit `.env` file and configure your database:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=saas_boilerplate
DB_USERNAME=root
DB_PASSWORD=your_password

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379
```

### Step 5: Run Migrations & Seeders

```bash
# Create database
mysql -u root -p -e "CREATE DATABASE saas_boilerplate CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# Run migrations
php artisan migrate --seed

# Optimize (optional for production)
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

### Step 6: Build Frontend Assets

```bash
# Development mode
npm run dev

# Production build
npm run build
```

### Step 7: Start Development Server

```bash
php artisan serve
```

Visit: `http://localhost:8000`

---

## Detailed Installation

### Local Development Setup

#### Using Docker (Recommended)

```bash
# Using Laravel Sail
./vendor/bin/sail up -d

# Access container
./vendor/bin/sail shell

# Run migrations inside container
./vendor/bin/sail artisan migrate --seed
```

#### Manual Setup

1. **Install MySQL/MariaDB**

```bash
# Ubuntu/Debian
sudo apt-get install mysql-server-8.0

# macOS (Homebrew)
brew install mysql@8.0
```

2. **Install Redis**

```bash
# Ubuntu/Debian
sudo apt-get install redis-server

# macOS (Homebrew)
brew install redis
```

3. **Configure Virtual Host (Apache)**

```apache
<VirtualHost *:80>
    ServerName saas-boilerplate.test
    DocumentRoot /var/www/laravel-saas-boilerplate/public
    
    <Directory /var/www/laravel-saas-boilerplate/public>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

4. **Configure Nginx**

```nginx
server {
    listen 80;
    server_name saas-boilerplate.test;
    root /var/www/laravel-saas-boilerplate/public;
    
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    
    index index.php;
    
    charset utf-8;
    
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    
    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }
    
    error_page 404 /index.php;
    
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
    
    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

---

## Module-Specific Setup

### Authentication Module

No additional setup required. Default Laravel authentication is extended with:

- Socialite for OAuth (Google, GitHub)
- Two-factor authentication via QR code

Configure social auth in `.env`:

```env
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
GOOGLE_REDIRECT_URI=http://localhost:8000/auth/google/callback

GITHUB_CLIENT_ID=your_github_client_id
GITHUB_CLIENT_SECRET=your_github_client_secret
GITHUB_REDIRECT_URI=http://localhost:8000/auth/github/callback
```

### Multi-Tenancy Module

Configure tenant domains in `.env`:

```env
CENTRAL_DOMAIN=localhost
TENANT_MODE=subdomain # or 'domain'
```

For local testing with subdomains, edit `/etc/hosts`:

```
127.0.0.1   localhost
127.0.0.1   tenant1.localhost
127.0.0.1   tenant2.localhost
```

### Billing Module

#### Stripe Configuration

```env
STRIPE_KEY=pk_test_...
STRIPE_SECRET=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

#### Paddle Configuration

```env
PADDLE_VENDOR_ID=12345
PADDLE_API_KEY=live_...
PADDLE_ENV=sandbox # or 'production'
```

#### Midtrans (Indonesia)

```env
MIDTRANS_SERVER_KEY=SB-Mid-server-...
MIDTRANS_CLIENT_KEY=SB-Mid-client-...
MIDTRANS_IS_PRODUCTION=false
```

### Notification Module

#### Email Configuration

```env
MAIL_MAILER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=your_username
MAIL_PASSWORD=your_password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=noreply@yoursaas.com
MAIL_FROM_NAME="${APP_NAME}"
```

#### WhatsApp Gateway (Twilio)

```env
TWILIO_SID=ACxxxxxxxxxxxxx
TWILIO_TOKEN=your_auth_token
TWILIO_WHATSAPP_NUMBER=whatsapp:+14155238886
```

### Queue Configuration

For production, use Redis queue:

```env
QUEUE_CONNECTION=redis
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379
```

Start queue worker:

```bash
php artisan queue:work --tries=3
```

For development with auto-reload:

```bash
php artisan queue:restart && php artisan queue:work --tries=3 --timeout=120
```

---

## Testing Installation

### Run Tests

```bash
# Run all tests
php artisan test

# Run with coverage
php artisan test --coverage

# Run specific test suite
php artisan test --testsuite=Unit
php artisan test --testsuite=Feature
```

### Verify Modules

```bash
# Check module status
php artisan module:list

# Verify routes
php artisan route:list

# Check configuration
php artisan config:clear && php artisan config:cache
```

### Smoke Test

1. Visit homepage: `http://localhost:8000`
2. Register new account
3. Verify email (check logs or mailtrap)
4. Login with credentials
5. Create a team
6. Invite a member
7. Subscribe to a plan (test mode)

---

## Troubleshooting

### Common Issues

#### Issue: Class not found errors

```bash
composer dump-autoload
php artisan optimize:clear
```

#### Issue: Migration errors

```bash
php artisan migrate:fresh --seed
```

#### Issue: Permission denied on storage

```bash
chmod -R 775 storage bootstrap/cache
chown -R www-data:www-data storage bootstrap/cache
```

#### Issue: npm install fails

```bash
rm -rf node_modules package-lock.json
npm cache clean --force
npm install
```

#### Issue: Redis connection failed

```bash
# Check if Redis is running
redis-cli ping

# Should return: PONG
```

#### Issue: Queue not processing jobs

```bash
# Restart queue worker
php artisan queue:restart

# Check failed jobs
php artisan queue:failed
```

### Debug Mode

Enable debug mode temporarily:

```env
APP_DEBUG=true
APP_ENV=local
```

Check Laravel logs:

```bash
tail -f storage/logs/laravel.log
```

### Get Help

- 📖 [Documentation Index](README.md)
- 🐛 [GitHub Issues](https://github.com/yourusername/laravel-saas-boilerplate/issues)
- 💬 [Discord Community](https://discord.gg/yourlink)

---

**Next Steps:** [Configuration Guide](CONFIGURATION.md)
