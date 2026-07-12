# 🎨 System Design Document
# Laravel SaaS Boilerplate

## 1. Architecture Overview

### 1.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Client Layer                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │   Web    │  │  Mobile  │  │   API    │  │  Admin   │   │
│  │ Browser  │  │   App    │  │  Client  │  │  Panel   │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      Load Balancer                           │
│                    (Nginx / Cloudflare)                      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                   Application Layer                          │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Laravel Application                      │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │   │
│  │  │  Middleware │  │   Routes    │  │ Controllers │  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │   │
│  │  │   Actions   │  │  Services   │  │   Events    │  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │   │
│  │  │    Jobs     │  │ Mail/Notif  │  │   Policies  │  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      Data Layer                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │  MySQL   │  │  Redis   │  │   S3     │  │MeiliSearch│  │
│  │ Primary  │  │  Cache   │  │ Storage  │  │  (Opt)    │  │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Multi-Tenancy Architecture

**Strategy:** Single Database with Tenant Discrimination

```
┌─────────────────────────────────────────────────────────────┐
│                     Request Flow                             │
│                                                              │
│  1. Request → tenant.domain.com                             │
│  2. Middleware identifies tenant by subdomain               │
│  3. Set tenant context in application                       │
│  4. Global scope applies tenant_id to all queries           │
│  5. Response returned with tenant-specific data             │
└─────────────────────────────────────────────────────────────┘

Database Schema:
┌─────────────────────────────────────────────────────────────┐
│  users                  tenants                teams         │
│  ├─ id (UUID)          ├─ id (UUID)          ├─ id (UUID)   │
│  ├─ tenant_id (FK)     ├─ name               ├─ tenant_id   │
│  ├─ name               ├─ subdomain          ├─ name        │
│  ├─ email              ├─ slug               └─ ...         │
│  └─ ...                └─ ...                               │
│                                                              │
│  All tenant resources include tenant_id for isolation       │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Component Design

### 2.1 Authentication Component

```php
┌─────────────────────────────────────────────────────────────┐
│                 Authentication Flow                          │
│                                                              │
│  ┌─────────┐    ┌─────────────┐    ┌──────────────┐        │
│  │  Login  │───▶│ AuthService │───▶│ UserProvider │        │
│  │  Form   │    │             │    │              │        │
│  └─────────┘    └─────────────┘    └──────────────┘        │
│                        │                       │             │
│                        ▼                       ▼             │
│                 ┌─────────────┐    ┌──────────────┐        │
│                 │ TokenGuard  │    │ SessionGuard │        │
│                 └─────────────┘    └──────────────┘        │
│                        │                       │             │
│                        └───────────┬───────────┘             │
│                                    ▼                         │
│                           ┌─────────────┐                   │
│                           │   User      │                   │
│                           │   Model     │                   │
│                           └─────────────┘                   │
└─────────────────────────────────────────────────────────────┘

Key Classes:
- app/Services/Auth/AuthService.php
- app/Http/Controllers/Auth/LoginController.php
- app/Http/Controllers/Auth/RegisterController.php
- app/Http/Middleware/Authenticate.php
- app/Models/User.php
```

### 2.2 Multi-Tenancy Component

```php
┌─────────────────────────────────────────────────────────────┐
│                 Tenant Resolution Flow                       │
│                                                              │
│  ┌─────────────┐    ┌──────────────┐    ┌──────────────┐   │
│  │   Request   │───▶│TenantResolver│───▶│TenantContext │   │
│  │ (subdomain) │    │  Middleware  │    │   Manager    │   │
│  └─────────────┘    └──────────────┘    └──────────────┘   │
│                                              │               │
│                                              ▼               │
│                                     ┌──────────────┐        │
│                                     │Global Scope  │        │
│                                     │  Auto-applied│        │
│                                     └──────────────┘        │
└─────────────────────────────────────────────────────────────┘

Key Classes:
- app/Http/Middleware/IdentifyTenant.php
- app/Models/Tenant.php
- app/Scopes/TenantScope.php
- app/Services/Tenant/TenantResolver.php
```

### 2.3 Subscription & Billing Component

```php
┌─────────────────────────────────────────────────────────────┐
│              Subscription Management Flow                    │
│                                                              │
│  ┌──────────┐   ┌─────────────┐   ┌──────────────────┐     │
│  │  Plan    │──▶│ Checkout    │──▶│ Payment Gateway  │     │
│  │Selection │   │  Session    │   │ (Stripe/Paddle)  │     │
│  └──────────┘   └─────────────┘   └──────────────────┘     │
│                                          │                   │
│                                          ▼                   │
│  ┌──────────┐   ┌─────────────┐   ┌──────────────────┐     │
│  │Invoice   │◀──│Subscription │◀──│   Webhook        │     │
│  │Generation│   │   Update    │   │   Handler        │     │
│  └──────────┘   └─────────────┘   └──────────────────┘     │
└─────────────────────────────────────────────────────────────┘

Key Classes:
- app/Services/Billing/SubscriptionService.php
- app/Http/Controllers/Billing/SubscriptionController.php
- app/Http/Controllers/Webhook/StripeWebhookController.php
- app/Models/Subscription.php
- app/Jobs/ProcessSubscriptionWebhook.php
```

### 2.4 Team Management Component

```php
┌─────────────────────────────────────────────────────────────┐
│                 Team Collaboration Flow                      │
│                                                              │
│  ┌──────────┐   ┌─────────────┐   ┌──────────────────┐     │
│  │  Create  │──▶│  Invite     │──▶│  Email           │     │
│  │  Team    │   │  Members    │   │  Notification    │     │
│  └──────────┘   └─────────────┘   └──────────────────┘     │
│                        │                                      │
│                        ▼                                      │
│  ┌──────────┐   ┌─────────────┐   ┌──────────────────┐     │
│  │  Switch  │◀──│  Role       │◀──│  Accept          │     │
│  │  Teams   │   │  Assignment │   │  Invitation      │     │
│  └──────────┘   └─────────────┘   └──────────────────┘     │
└─────────────────────────────────────────────────────────────┘

Key Classes:
- app/Models/Team.php
- app/Models/TeamUser.php
- app/Actions/Team/CreateTeam.php
- app/Actions/Team/InviteMember.php
- app/Actions/Team/SwitchTeam.php
```

### 2.5 RBAC Component

```php
┌─────────────────────────────────────────────────────────────┐
│                 Permission Checking Flow                     │
│                                                              │
│  ┌──────────┐   ┌─────────────┐   ┌──────────────────┐     │
│  │ @can()   │──▶│   Gate      │──▶│ PermissionCache  │     │
│  │ Check    │   │             │   │                  │     │
│  └──────────┘   └─────────────┘   └──────────────────┘     │
│                        │                                      │
│                        ▼                                      │
│  ┌──────────┐   ┌─────────────┐   ┌──────────────────┐     │
│  │  Access  │◀──│   Result    │◀──│ Role + Perms     │     │
│  │ Granted  │   │  (true/false)│ │  Aggregation     │     │
│  └──────────┘   └─────────────┘   └──────────────────┘     │
└─────────────────────────────────────────────────────────────┘

Integration dengan Spatie Laravel Permission:
- Roles stored in: roles table
- Permissions stored in: permissions table
- Mappings in: model_has_roles, model_has_permissions, role_has_permissions
```

---

## 3. Database Design

### 3.1 Entity Relationship Diagram

```
┌─────────────────┐         ┌─────────────────┐
│     users       │         │     tenants     │
├─────────────────┤         ├─────────────────┤
│ id (PK, UUID)   │         │ id (PK, UUID)   │
│ tenant_id (FK)  │◀───────▶│ owner_id (FK)   │
│ name            │         │ name            │
│ email           │         │ subdomain       │
│ password        │         │ slug            │
│ ...             │         │ ...             │
└─────────────────┘         └─────────────────┘
        │                          │
        │                          │
        ▼                          ▼
┌─────────────────┐         ┌─────────────────┐
│   team_users    │         │      teams      │
├─────────────────┤         ├─────────────────┤
│ id (PK)         │         │ id (PK, UUID)   │
│ team_id (FK)    │◀────────│ tenant_id (FK)  │
│ user_id (FK)    │────────▶│ name            │
│ role            │         │ slug            │
│ ...             │         │ ...             │
└─────────────────┘         └─────────────────┘

┌─────────────────┐         ┌─────────────────┐
│  subscriptions  │         │   activity_log  │
├─────────────────┤         ├─────────────────┤
│ id (PK)         │         │ id (PK)         │
│ user_id (FK)    │         │ subject_type    │
│ stripe_id       │         │ subject_id      │
│ type            │         │ causer_type     │
│ status          │         │ causer_id       │
│ ...             │         │ properties      │
└─────────────────┘         │ ...             │
                            └─────────────────┘
```

### 3.2 Index Strategy

```sql
-- Users
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_tenant_id ON users(tenant_id);

-- Tenants
CREATE UNIQUE INDEX idx_tenants_subdomain ON tenants(subdomain);
CREATE UNIQUE INDEX idx_tenants_slug ON tenants(slug);

-- Teams
CREATE INDEX idx_teams_tenant_id ON teams(tenant_id);
CREATE INDEX idx_team_users_user_id ON team_users(user_id);
CREATE INDEX idx_team_users_team_id ON team_users(team_id);

-- Subscriptions
CREATE INDEX idx_subscriptions_user_id ON subscriptions(user_id);
CREATE INDEX idx_subscriptions_stripe_id ON subscriptions(stripe_id);

-- Activity Log
CREATE INDEX idx_activity_subject ON activity_log(subject_type, subject_id);
CREATE INDEX idx_activity_causer ON activity_log(causer_type, causer_id);
CREATE INDEX idx_activity_created_at ON activity_log(created_at);
```

---

## 4. API Design

### 4.1 RESTful API Structure

```
Base URL: https://api.{domain}/v1

Authentication:
POST   /auth/register          - Register new user
POST   /auth/login             - Login
POST   /auth/logout            - Logout
POST   /auth/password/email    - Request password reset
POST   /auth/password/reset    - Reset password
POST   /auth/two-factor        - Enable 2FA
DELETE /auth/two-factor        - Disable 2FA

Tenants:
GET    /tenants                - List user's tenants
POST   /tenants                - Create tenant
GET    /tenants/{id}           - Get tenant details
PUT    /tenants/{id}           - Update tenant
DELETE /tenants/{id}           - Delete tenant

Teams:
GET    /teams                  - List teams
POST   /teams                  - Create team
GET    /teams/{id}             - Get team details
PUT    /teams/{id}             - Update team
DELETE /teams/{id}             - Delete team
POST   /teams/{id}/invite      - Invite member
DELETE /teams/{id}/members/{userId} - Remove member

Subscriptions:
GET    /subscription           - Get current subscription
POST   /subscription/{plan}    - Subscribe to plan
PUT    /subscription           - Update subscription
DELETE /subscription          - Cancel subscription
GET    /invoices               - List invoices

Users:
GET    /user/profile           - Get profile
PUT    /user/profile           - Update profile
GET    /user/api-tokens        - List API tokens
POST   /user/api-tokens        - Create token
DELETE /user/api-tokens/{id}   - Revoke token
```

### 4.2 API Response Format

```json
// Success Response
{
  "success": true,
  "data": { ... },
  "meta": {
    "pagination": { ... }
  }
}

// Error Response
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The given data was invalid.",
    "details": {
      "email": ["The email has already been taken."]
    }
  }
}
```

---

## 5. Security Design

### 5.1 Authentication & Authorization

```
┌─────────────────────────────────────────────────────────────┐
│                    Security Layers                           │
│                                                              │
│  Layer 1: HTTPS/TLS                                         │
│  - All traffic encrypted                                    │
│  - HSTS enabled                                             │
│                                                              │
│  Layer 2: Authentication                                    │
│  - Session-based for web                                    │
│  - Token-based for API (Sanctum)                            │
│  - 2FA support                                              │
│                                                              │
│  Layer 3: Authorization                                     │
│  - Middleware checks                                        │
│  - Policy-based access control                              │
│  - Gate integration                                         │
│                                                              │
│  Layer 4: Input Validation                                  │
│  - Form Request classes                                     │
│  - Type validation                                          │
│  - Sanitization                                             │
│                                                              │
│  Layer 5: Rate Limiting                                     │
│  - API throttling                                           │
│  - Login attempt limiting                                   │
│  - Action cooldowns                                         │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 Data Protection

```php
// Password Hashing
Algorithm: Argon2id
Memory: 65536 KB
Threads: 4
Time: 3 seconds

// Encryption
Cipher: AES-256-CBC
Key: Generated via php artisan key:generate

// CSRF Protection
Token per session
Validated on all state-changing requests

// XSS Prevention
All output escaped automatically via Blade
Content-Security-Policy headers
```

---

## 6. Frontend Design

### 6.1 Component Architecture

```
resources/js/
├── Components/
│   ├── UI/              # Base UI components
│   │   ├── Button.vue
│   │   ├── Input.vue
│   │   ├── Modal.vue
│   │   └── ...
│   ├── Layout/          # Layout components
│   │   ├── AppLayout.vue
│   │   ├── GuestLayout.vue
│   │   └── ...
│   ├── Forms/           # Form components
│   │   ├── LoginForm.vue
│   │   ├── RegisterForm.vue
│   │   └── ...
│   └── Features/        # Feature-specific
│       ├── TenantSwitcher.vue
│       ├── SubscriptionCard.vue
│       └── ...
├── Pages/               # Page components
│   ├── Dashboard.vue
│   ├── Settings.vue
│   └── ...
├── Composables/         # Vue composables
│   ├── useAuth.js
│   ├── useTenant.js
│   └── ...
└── Stores/              # Pinia stores
    ├── auth.js
    ├── tenant.js
    └── ...
```

### 6.2 State Management

```javascript
// Example: Auth Store (Pinia)
export const useAuthStore = defineStore('auth', {
  state: () => ({
    user: null,
    token: null,
    loading: false
  }),
  
  actions: {
    async login(credentials) {
      this.loading = true;
      try {
        const response = await api.post('/auth/login', credentials);
        this.user = response.data.user;
        this.token = response.data.token;
      } finally {
        this.loading = false;
      }
    },
    
    async logout() {
      await api.post('/auth/logout');
      this.user = null;
      this.token = null;
    }
  }
});
```

---

## 7. Deployment Design

### 7.1 Infrastructure

```
┌─────────────────────────────────────────────────────────────┐
│                 Production Architecture                      │
│                                                              │
│  ┌─────────────┐                                            │
│  │ Cloudflare  │  (CDN, DDoS protection, SSL)              │
│  └─────────────┘                                            │
│         │                                                   │
│         ▼                                                   │
│  ┌─────────────┐                                            │
│  │   Nginx     │  (Load balancer, reverse proxy)           │
│  └─────────────┘                                            │
│         │                                                   │
│    ┌────┴────┐                                              │
│    ▼         ▼                                              │
│ ┌──────┐ ┌──────┐                                           │
│ │App 1 │ │App 2 │  (Laravel applications, stateless)       │
│ └──────┘ └──────┘                                           │
│    │         │                                              │
│    └────┬────┘                                              │
│         │                                                   │
│    ┌────┴────────────────────┐                             │
│    ▼         ▼         ▼     │                             │
│ ┌──────┐ ┌──────┐ ┌──────┐  │                             │
│ │MySQL │ │Redis │ │ S3   │  │                             │
│ │Primary│ │Cache │ │Storage│ │                             │
│ └──────┘ └──────┘ └──────┘  │                             │
│                              │                             │
│ ┌──────────────────────────┐│                             │
│ │Queue Workers (Horizon)   ││                             │
│ └──────────────────────────┘│                             │
└─────────────────────────────────────────────────────────────┘
```

### 7.2 Docker Configuration

```dockerfile
# Dockerfile
FROM php:8.2-fpm

# Install extensions
RUN docker-php-ext-install pdo_mysql bcmath opcache

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /var/www/html

# Copy application
COPY . .

# Install dependencies
RUN composer install --optimize-autoloader --no-dev

# Permissions
RUN chown -R www-data:www-data /var/www/html

EXPOSE 9000
CMD ["php-fpm"]
```

---

## 8. Monitoring & Logging

### 8.1 Observability Stack

```
┌─────────────────────────────────────────────────────────────┐
│                  Monitoring Architecture                     │
│                                                              │
│  Application Logs ──▶ Filebeat ──▶ Elasticsearch           │
│                                                    │         │
│  Metrics ──▶ Prometheus ──▶ Grafana               │         │
│                                                    ▼         │
│  Errors ──▶ Sentry ◀────────────────────── Kibana          │
│                                                              │
│  Uptime ──▶ UptimeRobot / Pingdom                           │
└─────────────────────────────────────────────────────────────┘
```

### 8.2 Key Metrics

```yaml
Application Metrics:
  - Request rate
  - Response time (p50, p95, p99)
  - Error rate
  - Queue size
  - Cache hit ratio

Business Metrics:
  - Active users
  - New signups
  - Subscription MRR
  - Churn rate
  - Trial conversions

Infrastructure Metrics:
  - CPU usage
  - Memory usage
  - Disk I/O
  - Database connections
  - Network throughput
```

---

**Document Version:** 1.0  
**Last Updated:** 2025-07-12  
**Status:** Draft
