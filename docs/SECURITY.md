# Security Guide

## Laravel SaaS Boilerplate - TALL Stack

**Version:** 2.0  
**Last Updated:** 2025-01-12

---

## Table of Contents

1. [Security Architecture](#security-architecture)
2. [Authentication Security](#authentication-security)
3. [Authorization & RBAC](#authorization--rbac)
4. [Data Protection](#data-protection)
5. [API Security](#api-security)
6. [Multi-Tenancy Security](#multi-tenancy-security)
7. [Payment Security](#payment-security)
8. [Infrastructure Security](#infrastructure-security)
9. [Security Monitoring](#security-monitoring)
10. [Incident Response](#incident-response)

---

## Security Architecture

### Defense in Depth

Our boilerplate implements a 5-layer security approach:

```
┌─────────────────────────────────────┐
│ Layer 5: Infrastructure Security    │
│ - WAF, DDoS Protection, SSL/TLS     │
├─────────────────────────────────────┤
│ Layer 4: Application Security       │
│ - Input Validation, XSS, CSRF       │
├─────────────────────────────────────┤
│ Layer 3: Authentication Security    │
│ - 2FA, Session Management, OAuth    │
├─────────────────────────────────────┤
│ Layer 2: Authorization Security     │
│ - RBAC, Permissions, Policies       │
├─────────────────────────────────────┤
│ Layer 1: Data Security              │
│ - Encryption, Hashing, Sanitization │
└─────────────────────────────────────┘
```

---

## Authentication Security

### Password Security

```php
// Minimum password requirements
'password' => [
    'required',
    'min:12',
    'mixedCase',
    'numbers',
    'symbols',
    Password::defaults(),
],

// Password hashing (Laravel default: bcrypt)
Hash::make($password, ['rounds' => 12]);
```

### Two-Factor Authentication (2FA)

Implementation using TOTP (Time-based One-Time Password):

```php
use PragmaRX\Google2FA;

class TwoFactorAuthController extends Controller
{
    public function enable(Request $request)
    {
        $user = $request->user();
        
        // Generate secret
        $secret = $google2fa->generateSecretKey();
        
        // Save to user model (encrypted)
        $user->two_factor_secret = encrypt($secret);
        $user->two_factor_enabled_at = now();
        $user->save();
        
        // Return QR code for setup
        return $google2fa->getQRCodeInline(
            config('app.name'),
            $user->email,
            $secret
        );
    }
    
    public function verify(Request $request)
    {
        $valid = $google2fa->verifyKey(
            decrypt($request->user()->two_factor_secret),
            $request->code
        );
        
        if ($valid) {
            session(['2fa_passed' => true]);
            return redirect()->intended('/dashboard');
        }
        
        return back()->withErrors(['code' => 'Invalid code']);
    }
}
```

### Session Security

```php
// config/session.php
return [
    'driver' => env('SESSION_DRIVER', 'redis'),
    'lifetime' => env('SESSION_LIFETIME', 120),
    'expire_on_close' => false,
    'encrypt' => true,
    'files' => storage_path('framework/sessions'),
    'connection' => env('SESSION_CONNECTION'),
    'store' => env('SESSION_STORE'),
    'lottery' => [2, 100],
    'cookie' => env('SESSION_COOKIE', 'saas_session'),
    'path' => '/',
    'domain' => env('SESSION_DOMAIN'),
    'secure' => env('SESSION_SECURE_COOKIE', true),
    'http_only' => true,
    'same_site' => 'lax',
];
```

### Brute Force Protection

```php
// Rate limiting for login attempts
RateLimiter::for('login', function (Request $request) {
    return Limit::perMinute(5)->by(
        $request->email.$request->ip()
    )->response(function () {
        return response()->json([
            'message' => 'Too many login attempts. Please try again later.',
        ], 429);
    });
});
```

---

## Authorization & RBAC

### Role-Based Access Control

```php
// Middleware for role checking
class RoleMiddleware
{
    public function handle($request, Closure $next, ...$roles)
    {
        if (!$request->user() || !$request->user()->hasRole($roles)) {
            abort(403, 'Unauthorized action.');
        }
        
        return $next($request);
    }
}

// Usage in routes
Route::middleware(['auth', 'role:admin'])->group(function () {
    Route::resource('tenants', TenantController::class);
});
```

### Permission-Based Authorization

```php
// Policy example
class TeamPolicy
{
    public function invite(User $user, Team $team)
    {
        return $user->hasPermission('teams.invite') 
            && $user->belongsToTeam($team);
    }
    
    public function delete(User $user, Team $team)
    {
        return $user->hasPermission('teams.delete')
            && $user->isOwner($team);
    }
}

// In controller
public function invite(Request $request, Team $team)
{
    $this->authorize('invite', $team);
    // ...
}
```

---

## Data Protection

### Encryption at Rest

```php
use Illuminate\Support\Facades\Crypt;

// Encrypt sensitive data
$encrypted = Crypt::encryptString($value);

// Decrypt when needed
$decrypted = Crypt::decryptString($encrypted);

// Model encryption (Laravel 8+)
class User extends Model
{
    protected $casts = [
        'two_factor_secret' => 'encrypted',
        'api_token' => 'encrypted',
    ];
}
```

### Encryption in Transit

All communications must use HTTPS:

```apache
# .htaccess force HTTPS
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
```

```nginx
# Nginx force HTTPS
server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    # SSL configuration...
}
```

### SQL Injection Prevention

Always use Eloquent ORM or parameterized queries:

```php
// ✅ SAFE - Eloquent ORM
$users = User::where('status', 'active')->get();

// ✅ SAFE - Query Builder with bindings
$users = DB::table('users')
    ->where('status', '=', $status)
    ->get();

// ❌ UNSAFE - Raw query without bindings
$users = DB::select("SELECT * FROM users WHERE status = '$status'");
```

### XSS Prevention

Blade automatically escapes output:

```blade
{{-- ✅ SAFE - Auto-escaped --}}
{{ $userInput }}

{{-- ❌ UNSAFE - Only if explicitly unescaped --}}
{!! $userInput !!}

{{-- For rich text, use purification --}}
@php
    echo Purifier::clean($richText);
@endphp
```

---

## API Security

### Sanctum Authentication

```php
// Create API token with abilities
$token = $user->createToken('mobile-app', ['billing:read', 'teams:write']);

// Revoke tokens
$user->currentAccessToken()->delete();

// Token expiration
config('sanctum.expiration') // days
```

### API Rate Limiting

```php
// config/sanctum.php
return [
    'middleware' => [
        'throttle:api',
        \Illuminate\Http\Middleware\AuthenticateWithSanctum::class,
    ],
];

// RateLimiterServiceProvider
RateLimiter::for('api', function (Request $request) {
    return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
});
```

### CORS Configuration

```php
// config/cors.php
return [
    'paths' => ['api/*', 'sanctum/csrf-cookie'],
    'allowed_methods' => ['*'],
    'allowed_origins' => explode(',', env('CORS_ALLOWED_ORIGINS')),
    'allowed_origins_patterns' => [],
    'allowed_headers' => ['*'],
    'exposed_headers' => [],
    'max_age' => 0,
    'supports_credentials' => true,
];
```

---

## Multi-Tenancy Security

### Tenant Isolation

```php
// Middleware to ensure tenant isolation
class TenantScopeMiddleware
{
    public function handle($request, Closure $next)
    {
        $tenantId = tenant()->id;
        
        // Apply global scope to all queries
        Model::addGlobalScope(new TenantScope($tenantId));
        
        return $next($request);
    }
}

// Global scope
class TenantScope implements Scope
{
    public function apply(Builder $builder, Model $model)
    {
        $builder->where($model->getTable().'.tenant_id', tenant()->id);
    }
}
```

### Cross-Tenant Access Prevention

```php
// Always verify ownership
public function show(Team $team)
{
    // Check team belongs to current tenant
    if ($team->tenant_id !== tenant()->id) {
        abort(403, 'Access denied to this resource.');
    }
    
    return view('teams.show', compact('team'));
}
```

---

## Payment Security

### PCI DSS Compliance

- Never store raw credit card numbers
- Use Stripe Elements or Paddle.js for client-side processing
- Store only token/subscription IDs

```php
// ✅ SAFE - Store only Stripe ID
$user->update([
    'stripe_id' => $stripeCustomer->id,
    'card_brand' => $card->brand,
    'card_last_four' => $card->last4,
]);

// ❌ NEVER store full card number
```

### Webhook Verification

```php
// Verify Stripe webhook signature
public function handleWebhook(Request $request)
{
    $payload = $request->getContent();
    $sigHeader = $request->header('Stripe-Signature');
    
    try {
        $event = \Stripe\Webhook::constructEvent(
            $payload,
            $sigHeader,
            config('services.stripe.webhook_secret')
        );
    } catch (\UnexpectedValueException $e) {
        return response('Invalid signature', 400);
    }
    
    // Process event...
}
```

---

## Infrastructure Security

### Environment Variables Security

```bash
# .env should never be committed
# Add to .gitignore
.env
.env.local
.env.*.local

# Production secrets management
# Use: AWS Secrets Manager, HashiCorp Vault, or GitHub Secrets
```

### File Upload Security

```php
// Validate file uploads
$request->validate([
    'avatar' => 'required|image|mimes:jpeg,png|max:2048',
]);

// Store outside webroot if possible
$path = $request->file('avatar')->store(
    'avatars', 
    ['disk' => 'private']
);

// Generate unique filenames
$filename = Str::uuid().'.'.$file->extension();
```

### Dependency Security

```bash
# Regularly check for vulnerabilities
composer audit
npm audit

# Update dependencies
composer update
npm update

# Use Dependabot for automated updates
```

---

## Security Monitoring

### Logging Security Events

```php
// Log important security events
Log::channel('security')->info('User logged in', [
    'user_id' => $user->id,
    'ip' => $request->ip(),
    'user_agent' => $request->userAgent(),
]);

Log::channel('security')->warning('Failed login attempt', [
    'email' => $request->email,
    'ip' => $request->ip(),
    'attempts' => $attempts,
]);
```

### Security Headers

```php
// app/Http/Middleware/SecurityHeaders.php
public function handle($request, Closure $next)
{
    $response = $next($request);
    
    $response->headers->set('X-Frame-Options', 'DENY');
    $response->headers->set('X-Content-Type-Options', 'nosniff');
    $response->headers->set('X-XSS-Protection', '1; mode=block');
    $response->headers->set('Referrer-Policy', 'strict-origin-when-cross-origin');
    $response->headers->set('Permissions-Policy', 'geolocation=(), microphone=(), camera=()');
    
    return $response;
}
```

---

## Incident Response

### Security Breach Checklist

1. **Immediate Actions**
   - [ ] Identify and contain the breach
   - [ ] Preserve evidence (logs, backups)
   - [ ] Reset compromised credentials
   - [ ] Notify affected users

2. **Investigation**
   - [ ] Determine scope and impact
   - [ ] Identify root cause
   - [ ] Document timeline
   - [ ] Assess data exposure

3. **Recovery**
   - [ ] Patch vulnerabilities
   - [ ] Restore from clean backup
   - [ ] Implement additional controls
   - [ ] Monitor for recurrence

4. **Post-Incident**
   - [ ] Conduct post-mortem
   - [ ] Update security policies
   - [ ] Train team on lessons learned
   - [ ] Report to authorities if required

### Contact Information

- Security Team: security@yoursaas.com
- Emergency: +1-XXX-XXX-XXXX (24/7)
- Bug Bounty: https://hackerone.com/yoursaas

---

## Security Checklist

### Pre-Launch

- [ ] All dependencies updated
- [ ] Security headers configured
- [ ] HTTPS enforced
- [ ] Rate limiting enabled
- [ ] 2FA available
- [ ] Backup strategy tested
- [ ] Error pages don't leak information
- [ ] Admin paths not predictable

### Ongoing

- [ ] Weekly dependency audits
- [ ] Monthly security scans
- [ ] Quarterly penetration testing
- [ ] Annual security training
- [ ] Regular backup verification

---

**Related Documentation:**
- [Configuration Guide](CONFIGURATION.md)
- [Troubleshooting Guide](TROUBLESHOOTING.md)
- [Installation Guide](INSTALLATION.md)
