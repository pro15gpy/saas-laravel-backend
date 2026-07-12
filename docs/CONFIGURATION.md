# Configuration Guide

## Laravel SaaS Boilerplate - TALL Stack

**Version:** 2.0  
**Last Updated:** 2025-01-12

---

## Table of Contents

1. [Environment Configuration](#environment-configuration)
2. [Module Configuration](#module-configuration)
3. [Database Configuration](#database-configuration)
4. [Cache & Queue Configuration](#cache--queue-configuration)
5. [Mail Configuration](#mail-configuration)
6. [Billing Gateway Configuration](#billing-gateway-configuration)
7. [Social Authentication Configuration](#social-authentication-configuration)
8. [WhatsApp Gateway Configuration](#whatsapp-gateway-configuration)
9. [Multi-Tenancy Configuration](#multi-tenancy-configuration)
10. [Security Configuration](#security-configuration)

---

## Environment Configuration

### Base .env File

```env
# Application
APP_NAME="Laravel SaaS Boilerplate"
APP_ENV=local
APP_KEY=base64:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
APP_DEBUG=true
APP_URL=http://localhost

# Timezone
APP_TIMEZONE=UTC
```

### Environment-Specific Settings

#### Local Development

```env
APP_ENV=local
APP_DEBUG=true
LOG_CHANNEL=stack
LOG_LEVEL=debug
```

#### Staging

```env
APP_ENV=staging
APP_DEBUG=false
LOG_CHANNEL=errorlog
LOG_LEVEL=info
```

#### Production

```env
APP_ENV=production
APP_DEBUG=false
LOG_CHANNEL=stderr
LOG_LEVEL=info
```

---

## Module Configuration

### Core Module

`config/core.php`:

```php
return [
    'user' => [
        'avatar_path' => 'avatars',
        'max_avatar_size' => 2048, // KB
        'allowed_avatar_types' => ['jpg', 'jpeg', 'png'],
    ],
    
    'session' => [
        'timeout' => 7200, // seconds (2 hours)
        'concurrent_limit' => 5,
    ],
];
```

### Tenant Module

`config/tenant.php`:

```php
return [
    'mode' => env('TENANT_MODE', 'subdomain'), // 'subdomain' or 'domain'
    'central_domain' => env('CENTRAL_DOMAIN', 'localhost'),
    'tenant_route_namespace' => 'App\\Http\\Controllers\\Tenant',
    
    'database' => [
        'isolation' => env('TENANT_DB_ISOLATION', 'single'), // 'single' or 'separate'
        'prefix' => '',
    ],
    
    'features' => [
        'custom_domains' => true,
        'tenant_switching' => true,
    ],
];
```

### RBAC Module

`config/rbac.php`:

```php
return [
    'default_role' => 'member',
    
    'roles' => [
        'admin' => ['name' => 'Administrator', 'level' => 100],
        'manager' => ['name' => 'Manager', 'level' => 50],
        'member' => ['name' => 'Member', 'level' => 10],
        'viewer' => ['name' => 'Viewer', 'level' => 1],
    ],
    
    'permissions' => [
        'users' => ['create', 'read', 'update', 'delete'],
        'teams' => ['create', 'read', 'update', 'delete', 'invite'],
        'billing' => ['subscribe', 'cancel', 'view_invoices'],
    ],
    
    'guard' => 'web',
];
```

### Billing Module

`config/billing.php`:

```php
return [
    'default_gateway' => env('BILLING_GATEWAY', 'stripe'),
    
    'gateways' => [
        'stripe' => [
            'enabled' => env('STRIPE_ENABLED', true),
            'key' => env('STRIPE_KEY'),
            'secret' => env('STRIPE_SECRET'),
            'webhook_secret' => env('STRIPE_WEBHOOK_SECRET'),
            'currency' => env('STRIPE_CURRENCY', 'usd'),
        ],
        
        'paddle' => [
            'enabled' => env('PADDLE_ENABLED', false),
            'vendor_id' => env('PADDLE_VENDOR_ID'),
            'api_key' => env('PADDLE_API_KEY'),
            'environment' => env('PADDLE_ENV', 'sandbox'),
        ],
        
        'midtrans' => [
            'enabled' => env('MIDTRANS_ENABLED', false),
            'server_key' => env('MIDTRANS_SERVER_KEY'),
            'client_key' => env('MIDTRANS_CLIENT_KEY'),
            'is_production' => env('MIDTRANS_IS_PRODUCTION', false),
        ],
    ],
    
    'plans' => [
        'free' => [
            'price' => 0,
            'features' => ['Basic features', '1 team member'],
            'limits' => ['projects' => 3, 'storage_gb' => 1],
        ],
        'pro' => [
            'price' => 2900, // cents
            'features' => ['All features', 'Unlimited team members'],
            'limits' => ['projects' => -1, 'storage_gb' => 100],
        ],
        'enterprise' => [
            'price' => 9900,
            'features' => ['Priority support', 'Custom integrations'],
            'limits' => ['projects' => -1, 'storage_gb' => -1],
        ],
    ],
    
    'trial_days' => 14,
    'invoice_path' => 'invoices',
];
```

### Notification Module

`config/notification.php`:

```php
return [
    'channels' => ['mail', 'database', 'whatsapp'],
    
    'email' => [
        'from_address' => env('MAIL_FROM_ADDRESS'),
        'from_name' => env('MAIL_FROM_NAME', env('APP_NAME')),
    ],
    
    'whatsapp' => [
        'provider' => env('WHATSAPP_PROVIDER', 'twilio'),
        'enabled' => env('WHATSAPP_ENABLED', false),
    ],
    
    'in_app' => [
        'max_notifications' => 50,
        'expiry_days' => 30,
    ],
];
```

### Onboarding Module

`config/onboarding.php`:

```php
return [
    'enabled' => true,
    
    'steps' => [
        'profile_completion' => ['title' => 'Complete Your Profile', 'required' => true],
        'team_creation' => ['title' => 'Create Your Team', 'required' => false],
        'first_project' => ['title' => 'Create First Project', 'required' => false],
        'invite_member' => ['title' => 'Invite a Team Member', 'required' => false],
        'subscription' => ['title' => 'Choose a Plan', 'required' => false],
    ],
    
    'redirect_after_complete' => '/dashboard',
    'show_progress' => true,
];
```

---

## Database Configuration

### MySQL Configuration

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=saas_boilerplate
DB_USERNAME=root
DB_PASSWORD=secret
DB_CHARSET=utf8mb4
DB_COLLATION=utf8mb4_unicode_ci
DB_PREFIX=
DB_STRICT=true
```

### PostgreSQL Configuration

```env
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=saas_boilerplate
DB_USERNAME=postgres
DB_PASSWORD=secret
DB_SCHEMA=public
DB_PREFIX=
```

### Multi-Database Setup (for separate tenant DBs)

```env
# Central database (for tenants table)
DB_CENTRAL_CONNECTION=mysql
DB_CENTRAL_HOST=127.0.0.1
DB_CENTRAL_DATABASE=saas_central
DB_CENTRAL_USERNAME=root
DB_CENTRAL_PASSWORD=secret

# Default tenant database template
DB_TENANT_TEMPLATE=mysql
DB_TENANT_HOST=127.0.0.1
DB_TENANT_PREFIX=tenant_
```

---

## Cache & Queue Configuration

### Redis Configuration

```env
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379
REDIS_DATABASE=0
```

### Cache Driver

```env
CACHE_DRIVER=redis
CACHE_PREFIX=saas_
CACHE_STORE_TTL=3600
```

### Queue Driver

```env
QUEUE_CONNECTION=redis
QUEUE_PREFIX=saas_
QUEUE_RETRY_AFTER=90
QUEUE_MAX_ATTEMPTS=3
QUEUE_TIMEOUT=120
```

### Failed Jobs Table

```env
QUEUE_FAILED_TABLE=failed_jobs
QUEUE_FAILED_DATABASE=mysql
```

---

## Mail Configuration

### SMTP (Gmail)

```env
MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=your-email@gmail.com
MAIL_PASSWORD=your-app-password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=noreply@yoursaas.com
MAIL_FROM_NAME="${APP_NAME}"
```

### SMTP (Mailtrap for Testing)

```env
MAIL_MAILER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=your-mailtrap-username
MAIL_PASSWORD=your-mailtrap-password
MAIL_ENCRYPTION=tls
```

### SES (Amazon)

```env
MAIL_MAILER=ses
MAIL_HOST=email-smtp.us-east-1.amazonaws.com
MAIL_PORT=587
MAIL_USERNAME=your-ses-smtp-username
MAIL_PASSWORD=your-ses-smtp-password
MAIL_ENCRYPTION=tls
AWS_ACCESS_KEY_ID=your-aws-access-key
AWS_SECRET_ACCESS_KEY=your-aws-secret-key
AWS_DEFAULT_REGION=us-east-1
```

### Postmark

```env
MAIL_MAILER=postmark
MAIL_POSTMARK_TOKEN=your-postmark-server-token
```

---

## Billing Gateway Configuration

### Stripe

```env
STRIPE_ENABLED=true
STRIPE_KEY=pk_test_xxxxxxxxxxxxxxxxxxxxx
STRIPE_SECRET=sk_test_xxxxxxxxxxxxxxxxxxxxx
STRIPE_WEBHOOK_SECRET=whsec_xxxxxxxxxxxxxxxxxxxxx
STRIPE_CURRENCY=usd
STRIPE_LOCALE=en
```

### Paddle

```env
PADDLE_ENABLED=true
PADDLE_VENDOR_ID=12345
PADDLE_API_KEY=live_xxxxxxxxxxxxxxxxxxxxx
PADDLE_PUBLIC_KEY=-----BEGIN PUBLIC KEY-----...
PADDLE_ENV=sandbox # or 'production'
PADDLE_WEBHOOK_SECRET=your-webhook-secret
```

### Midtrans (Indonesia)

```env
MIDTRANS_ENABLED=true
MIDTRANS_SERVER_KEY=SB-Mid-server-xxxxxxxxxxxxxxxxxxxxx
MIDTRANS_CLIENT_KEY=SB-Mid-client-xxxxxxxxxxxxxxxxxxxxx
MIDTRANS_IS_PRODUCTION=false
MIDTRANS_MERCHANT_ID=G12345678
```

### PayPal

```env
PAYPAL_ENABLED=false
PAYPAL_CLIENT_ID=your-paypal-client-id
PAYPAL_SECRET=your-paypal-secret
PAYPAL_MODE=sandbox # or 'live'
```

---

## Social Authentication Configuration

### Google OAuth

```env
GOOGLE_ENABLED=true
GOOGLE_CLIENT_ID=xxxxxxxxxxxxx.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=GOCSPX-xxxxxxxxxxxxxxxxxxxx
GOOGLE_REDIRECT_URI=https://yoursaas.com/auth/google/callback
```

### GitHub OAuth

```env
GITHUB_ENABLED=true
GITHUB_CLIENT_ID=Iv1.xxxxxxxxxxxxxxxx
GITHUB_CLIENT_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
GITHUB_REDIRECT_URI=https://yoursaas.com/auth/github/callback
```

### Facebook OAuth

```env
FACEBOOK_ENABLED=false
FACEBOOK_CLIENT_ID=xxxxxxxxxxxxxxxxxx
FACEBOOK_CLIENT_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
FACEBOOK_REDIRECT_URI=https://yoursaas.com/auth/facebook/callback
```

---

## WhatsApp Gateway Configuration

### Twilio

```env
WHATSAPP_PROVIDER=twilio
TWILIO_ENABLED=true
TWILIO_SID=ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_WHATSAPP_NUMBER=whatsapp:+14155238886
TWILIO_VERIFY_SID=VAXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

### Meta (Facebook) WhatsApp API

```env
WHATSAPP_PROVIDER=meta
META_WHATSAPP_ENABLED=true
META_PHONE_NUMBER_ID=xxxxxxxxxxxxxx
META_BUSINESS_ACCOUNT_ID=xxxxxxxxxxxxxx
META_ACCESS_TOKEN=EAAxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
META_VERIFY_TOKEN=your-verify-token
META_WEBHOOK_URL=https://yoursaas.com/webhooks/whatsapp
```

### Wati.io

```env
WHATSAPP_PROVIDER=wati
WATI_ENABLED=true
WATI_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
WATI_PHONE_NUMBER=+60123456789
WATI_WEBHOOK_SECRET=your-webhook-secret
```

---

## Multi-Tenancy Configuration

### Subdomain Mode

```env
TENANT_MODE=subdomain
CENTRAL_DOMAIN=localhost
CENTRAL_PROTOCOL=http
TENANT_PROTOCOL=http
```

### Domain Mode

```env
TENANT_MODE=domain
CENTRAL_DOMAIN=yoursaas.com
```

### Database Isolation

```env
# Single database with tenant_id column
TENANT_DB_ISOLATION=single

# OR separate database per tenant
TENANT_DB_ISOLATION=separate
TENANT_DB_HOST=127.0.0.1
TENANT_DB_USERNAME=tenant_user
TENANT_DB_PASSWORD=tenant_password
```

### Custom Domains

```env
TENANT_CUSTOM_DOMAINS=true
TENANT_DOMAIN_VERIFICATION=true
```

---

## Security Configuration

### Rate Limiting

```env
RATE_LIMIT_API=60 # requests per minute
RATE_LIMIT_AUTH=5 # login attempts per minute
RATE_LIMIT_GUEST=30 # guest requests per minute
```

### Session Security

```env
SESSION_DRIVER=redis
SESSION_LIFETIME=120
SESSION_SECURE_COOKIE=true
SESSION_HTTP_ONLY=true
SESSION_SAME_SITE=lax
```

### CORS Configuration

```env
CORS_ALLOWED_ORIGINS=http://localhost:3000,https://app.yoursaas.com
CORS_ALLOWED_METHODS=GET,POST,PUT,DELETE,PATCH,OPTIONS
CORS_ALLOWED_HEADERS=Content-Type,Authorization,X-Requested-With
CORS_EXPOSED_HEADERS=
CORS_MAX_AGE=86400
```

### Sanctum API

```env
SANCTUM_STATEFUL_DOMAINS=localhost,localhost:3000,yoursaas.test
SANCTUM_EXPIRATION=14 # days
```

---

## Feature Flags

Enable/disable features dynamically:

```env
FEATURE_ONBOARDING=true
FEATURE_WHATSAPP_NOTIFICATIONS=false
FEATURE_USAGE_BILLING=false
FEATURE_AFFILIATE_SYSTEM=false
FEATURE_PLUGIN_MARKETPLACE=false
```

---

## Next Steps

After configuration:

1. Clear and cache config: `php artisan config:clear && php artisan config:cache`
2. Run migrations: `php artisan migrate`
3. Test configuration: `php artisan about`

---

**Related Documentation:**
- [Installation Guide](INSTALLATION.md)
- [Security Guide](SECURITY.md)
- [Troubleshooting](TROUBLESHOOTING.md)
