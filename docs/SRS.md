# 📝 Software Requirements Specification (SRS)
# Laravel SaaS Boilerplate

## 1. Introduction

### 1.1 Purpose
Dokumen ini menjelaskan requirement detail untuk Laravel SaaS Boilerplate, mencakup functional requirements, non-functional requirements, dan technical specifications.

### 1.2 Scope
Boilerplate ini adalah solusi lengkap untuk membangun aplikasi SaaS berbasis Laravel dengan fitur multi-tenancy, subscription billing, team management, dan RBAC.

### 1.3 Definitions & Acronyms
| Term | Definition |
|------|------------|
| SaaS | Software as a Service |
| RBAC | Role-Based Access Control |
| API | Application Programming Interface |
| UUID | Universally Unique Identifier |
| 2FA | Two-Factor Authentication |
| OAuth | Open Authorization |
| webhook | HTTP callback untuk event notification |

---

## 2. Overall Description

### 2.1 Product Perspective
Boilerplate ini berdiri sebagai starter kit yang dapat di-clone dan dikustomisasi. Bukan package atau library, melainkan full application template.

### 2.2 Product Functions
1. User authentication & authorization
2. Multi-tenant architecture
3. Subscription management
4. Team collaboration
5. Permission management
6. Notification system
7. Activity logging
8. Admin dashboard

### 2.3 User Classes and Characteristics
| User Class | Characteristics | Technical Skills |
|------------|----------------|------------------|
| End User | Menggunakan aplikasi SaaS | Basic computer literacy |
| Tenant Admin | Mengelola tim & subscription | Intermediate |
| Developer | Mengembangkan custom features | Advanced Laravel knowledge |
| Super Admin | Mengelola seluruh platform | System administration |

### 2.4 Operating Environment
- **Server**: Linux (Ubuntu 22.04+) / macOS
- **Web Server**: Nginx / Apache
- **PHP**: 8.2+
- **Database**: MySQL 8.0+ / PostgreSQL 15+
- **Cache**: Redis 7.0+
- **Browser**: Modern browsers (Chrome, Firefox, Safari, Edge)

### 2.5 Design & Implementation Constraints
- Harus compatible dengan Laravel 11/12
- Menggunakan MIT License
- Support PHP 8.2 minimum
- Database agnostic (MySQL & PostgreSQL)
- Mobile-responsive design

### 2.6 Assumptions and Dependencies
**Assumptions:**
- User memiliki basic knowledge Laravel
- Server environment sudah configured
- Third-party services (Stripe, etc.) tersedia

**Dependencies:**
- Laravel Framework 11/12
- Spatie Laravel Permission
- Laravel Cashier (Stripe/Paddle)
- Laravel Sanctum
- Stancl Tenancy (optional)

---

## 3. Functional Requirements

### 3.1 Authentication Module

#### FR-AUTH-001: User Registration
**Priority:** High  
**Description:** User dapat mendaftar akun baru dengan email dan password  
**Input:** Email, password, name  
**Process:** 
1. Validate input
2. Check email uniqueness
3. Hash password
4. Create user record
5. Send verification email  
**Output:** User account created, verification email sent  
**Acceptance Criteria:**
- Email harus valid dan unik
- Password minimal 8 karakter
- Verification email terkirim dalam 5 detik

#### FR-AUTH-002: User Login
**Priority:** High  
**Description:** User dapat login dengan credentials  
**Input:** Email, password  
**Process:**
1. Validate credentials
2. Check email verification status
3. Create session
4. Generate CSRF token  
**Output:** Authenticated session, redirect to dashboard  
**Acceptance Criteria:**
- Login berhasil dalam < 2 detik
- Failed login tidak reveal apakah email exists
- Remember me functionality tersedia

#### FR-AUTH-003: Password Reset
**Priority:** High  
**Description:** User dapat reset password via email  
**Input:** Email address  
**Process:**
1. Validate email exists
2. Generate reset token
3. Send reset email
4. Token expires in 1 hour  
**Output:** Password reset email sent  
**Acceptance Criteria:**
- Email terkirim dalam 5 detik
- Token expired setelah 1 jam
- Rate limiting: max 3 requests per hour

#### FR-AUTH-004: Two-Factor Authentication
**Priority:** Medium  
**Description:** User dapat enable 2FA untuk security tambahan  
**Input:** User enables 2FA from settings  
**Process:**
1. Generate secret key
2. Display QR code
3. User scans with authenticator app
4. Verify TOTP code
5. Backup codes generation  
**Output:** 2FA enabled, backup codes displayed  
**Acceptance Criteria:**
- Support Google Authenticator, Authy
- Generate 10 backup codes
- Backup codes dapat di-download

#### FR-AUTH-005: Social Authentication
**Priority:** Medium  
**Description:** User dapat login/register menggunakan social providers  
**Input:** OAuth provider selection  
**Process:**
1. Redirect to OAuth provider
2. User authorizes application
3. Callback with authorization code
4. Exchange code for user info
5. Create/link user account  
**Output:** User authenticated  
**Acceptance Criteria:**
- Support Google, GitHub, LinkedIn
- Auto-create account jika belum ada
- Link social account ke existing user

---

### 3.2 Multi-Tenancy Module

#### FR-TENANT-001: Tenant Creation
**Priority:** High  
**Description:** User dapat membuat tenant baru saat signup  
**Input:** Tenant name, subdomain  
**Process:**
1. Validate subdomain availability
2. Create tenant record
3. Setup tenant database/schema
4. Assign user as tenant owner  
**Output:** Tenant created, user assigned  
**Acceptance Criteria:**
- Subdomain unik dan valid
- Tenant setup < 10 detik
- Owner role otomatis assigned

#### FR-TENANT-002: Subdomain Routing
**Priority:** High  
**Description:** Setiap tenant accessible via unique subdomain  
**Input:** Subdomain dalam URL  
**Process:**
1. Parse subdomain dari request
2. Resolve tenant dari subdomain
3. Set tenant context
4. Apply tenant scope to queries  
**Output:** Request scoped to correct tenant  
**Acceptance Criteria:**
- Routing bekerja dalam < 100ms
- Tenant isolation terjamin
- Fallback ke apex domain untuk landing

#### FR-TENANT-003: Tenant Switching
**Priority:** Medium  
**Description:** User dapat switch antara multiple tenants  
**Input:** Tenant selection  
**Process:**
1. Verify user membership
2. Update session tenant context
3. Refresh permissions  
**Output:** Context switched to selected tenant  
**Acceptance Criteria:**
- Switch terjadi tanpa logout
- Permissions reload correctly
- UI updates immediately

#### FR-TENANT-004: Custom Domain Support
**Priority:** Low  
**Description:** Tenant dapat menggunakan custom domain  
**Input:** Custom domain configuration  
**Process:**
1. Validate domain ownership
2. Configure DNS records
3. SSL certificate provisioning
4. Map domain to tenant  
**Output:** Custom domain active  
**Acceptance Criteria:**
- Support CNAME/A record validation
- Auto SSL via Let's Encrypt
- Propagation detection

---

### 3.3 Subscription & Billing Module

#### FR-BILL-001: Plan Selection
**Priority:** High  
**Description:** User dapat memilih subscription plan  
**Input:** Plan selection  
**Process:**
1. Display available plans
2. Show features comparison
3. Handle plan selection
4. Redirect to checkout  
**Output:** User proceeds to payment  
**Acceptance Criteria:**
- Plans configurable via database
- Trial period clearly displayed
- Features list accurate

#### FR-BILL-002: Payment Processing
**Priority:** High  
**Description:** Process subscription payment via Stripe/Paddle  
**Input:** Payment method details  
**Process:**
1. Create customer on payment gateway
2. Setup subscription
3. Process initial payment
4. Store subscription record  
**Output:** Subscription active  
**Acceptance Criteria:**
- Support credit cards, Apple Pay, Google Pay
- PCI compliance maintained
- Payment confirmation < 5 detik

#### FR-BILL-003: Webhook Handling
**Priority:** High  
**Description:** Handle payment webhooks untuk subscription events  
**Input:** Webhook payload from payment provider  
**Process:**
1. Verify webhook signature
2. Parse event type
3. Update subscription status
4. Trigger notifications  
**Output:** Subscription state updated  
**Acceptance Criteria:**
- Idempotent processing
- Support invoice.paid, customer.subscription.deleted, dll
- Failed webhook retry logic

#### FR-BILL-004: Invoice Management
**Priority:** Medium  
**Description:** Generate dan store invoices  
**Input:** Successful payment event  
**Process:**
1. Generate PDF invoice
2. Store in cloud storage
3. Email to customer
4. Record in database  
**Output:** Invoice available for download  
**Acceptance Criteria:**
- PDF format professional
- Include tax details if applicable
- Accessible from user dashboard

#### FR-BILL-005: Subscription Lifecycle
**Priority:** High  
**Description:** Manage subscription upgrades, downgrades, cancellations  
**Input:** User action (upgrade/downgrade/cancel)  
**Process:**
1. Validate action eligibility
2. Prorate charges if needed
3. Update subscription on gateway
4. Update local records  
**Output:** Subscription modified  
**Acceptance Criteria:**
- Proration calculated correctly
- Changes effective immediately or next cycle
- Cancellation allows access until period end

---

### 3.4 Team Management Module

#### FR-TEAM-001: Team Creation
**Priority:** High  
**Description:** User dapat membuat team dalam tenant  
**Input:** Team name, description  
**Process:**
1. Validate team name uniqueness
2. Create team record
3. Assign creator as team owner  
**Output:** Team created  
**Acceptance Criteria:**
- Multiple teams per tenant allowed
- Owner role automatically assigned
- Team slug auto-generated

#### FR-TEAM-002: Member Invitation
**Priority:** High  
**Description:** Invite users to join team via email  
**Input:** Email address, role  
**Process:**
1. Generate invitation token
2. Send invitation email
3. Track invitation status
4. Token expires in 7 days  
**Output:** Invitation sent  
**Acceptance Criteria:**
- Email dengan personalized message
- Resend invitation option
- Cancel pending invitation

#### FR-TEAM-003: Role Assignment
**Priority:** High  
**Description:** Assign roles to team members  
**Input:** User, role selection  
**Process:**
1. Verify assigner permissions
2. Update user-team role
3. Refresh cached permissions  
**Output:** Role updated  
**Acceptance Criteria:**
- Predefined roles: Owner, Admin, Member, Viewer
- Custom roles support
- Role hierarchy enforced

#### FR-TEAM-004: Team Settings
**Priority:** Medium  
**Description:** Configure team settings and branding  
**Input:** Team configuration  
**Process:**
1. Update team record
2. Upload logo/avatar
3. Configure preferences  
**Output:** Team settings updated  
**Acceptance Criteria:**
- Logo upload dengan validation
- Settings saved atomically
- Changes reflected immediately

---

### 3.5 RBAC Module

#### FR-RBAC-001: Permission Definition
**Priority:** High  
**Description:** Define granular permissions  
**Input:** Permission name, description  
**Process:**
1. Create permission record
2. Associate with guards
3. Cache for performance  
**Output:** Permission available  
**Acceptance Criteria:**
- Permission naming convention enforced
- Grouped by module/resource
- Descriptive documentation

#### FR-RBAC-002: Role-Permission Assignment
**Priority:** High  
**Description:** Assign permissions to roles  
**Input:** Role, permissions array  
**Process:**
1. Validate permissions exist
2. Sync role-permission relationship
3. Clear permission cache  
**Output:** Role updated with permissions  
**Acceptance Criteria:**
- Bulk assignment supported
- Permission inheritance optional
- Audit log of changes

#### FR-RBAC-003: Permission Checking
**Priority:** High  
**Description:** Check user permissions in application  
**Input:** User, permission name  
**Process:**
1. Retrieve user roles
2. Aggregate permissions
3. Check against requested permission  
**Output:** Boolean result  
**Acceptance Criteria:**
- Works with Blade directives (@can)
- Works with middleware
- Works with gate policies

#### FR-RBAC-004: Super Admin Bypass
**Priority:** Medium  
**Description:** Super admin bypasses all permission checks  
**Input:** User with super admin flag  
**Process:**
1. Check super admin status
2. Return true for all permission checks  
**Output:** Access granted  
**Acceptance Criteria:**
- Configurable in .env
- Logged for audit purposes
- Not shown in regular UI

---

### 3.6 User Profile Module

#### FR-PROF-001: Profile Management
**Priority:** High  
**Description:** User dapat manage profile information  
**Input:** Profile data (name, avatar, bio)  
**Process:**
1. Validate input
2. Upload avatar if provided
3. Update user record  
**Output:** Profile updated  
**Acceptance Criteria:**
- Avatar resize dan optimize
- Validation errors clear
- Success feedback shown

#### FR-PROF-002: Account Settings
**Priority:** High  
**Description:** Manage account preferences  
**Input:** Settings (email, password, timezone)  
**Process:**
1. Validate current password for sensitive changes
2. Update settings
3. Send confirmation email for email change  
**Output:** Settings updated  
**Acceptance Criteria:**
- Email change requires verification
- Password change requires current password
- Timezone affects timestamps

#### FR-PROF-003: API Token Management
**Priority:** Medium  
**Description:** Create and manage API tokens  
**Input:** Token name, scopes  
**Process:**
1. Generate plain text token
2. Hash and store token
3. Display token once  
**Output:** API token created  
**Acceptance Criteria:**
- Token shown only once
- Scopes limit token permissions
- Revoke token anytime

#### FR-PROF-004: Account Deletion
**Priority:** Medium  
**Description:** User dapat delete account dengan data export  
**Input:** Confirmation password  
**Process:**
1. Generate data export
2. Schedule account deletion (grace period)
3. Anonymize or delete data
4. Cancel subscriptions  
**Output:** Account scheduled for deletion  
**Acceptance Criteria:**
- 30-day grace period
- Data export in JSON/CSV
- All subscriptions cancelled

---

### 3.7 Notification Module

#### FR-NOTIF-001: Email Notifications
**Priority:** High  
**Description:** Send transactional emails  
**Trigger:** Various application events  
**Process:**
1. Queue email job
2. Render email template
3. Send via mail driver  
**Output:** Email delivered  
**Acceptance Criteria:**
- Emails queued for performance
- Templates customizable
- Unsubscribe link included

#### FR-NOTIF-002: In-App Notifications
**Priority:** Medium  
**Description:** Real-time notifications dalam aplikasi  
**Trigger:** User-specific events  
**Process:**
1. Create notification record
2. Broadcast via WebSocket (optional)
3. Display in notification center  
**Output:** Notification visible  
**Acceptance Criteria:**
- Mark as read/unread
- Batch notifications
- Clear all option

#### FR-NOTIF-003: Notification Preferences
**Priority:** Low  
**Description:** User dapat configure notification preferences  
**Input:** Notification channel preferences  
**Process:**
1. Save preferences per notification type
2. Filter notifications based on preferences  
**Output:** Preferences saved  
**Acceptance Criteria:**
- Per-channel toggles (email, in-app, SMS)
- Per-notification-type settings
- Defaults sensible

---

### 3.8 Activity Logging Module

#### FR-LOG-001: Action Logging
**Priority:** Medium  
**Description:** Log critical user actions  
**Trigger:** Defined auditable events  
**Process:**
1. Capture action details
2. Store in audit_log table
3. Include metadata (IP, user agent)  
**Output:** Audit record created  
**Acceptance Criteria:**
- Configurable what to log
- Immutable logs
- Searchable interface

#### FR-LOG-002: Activity Timeline
**Priority:** Low  
**Description:** Display user activity timeline  
**Input:** User or resource selection  
**Process:**
1. Query audit logs
2. Format for display
3. Paginate results  
**Output:** Timeline displayed  
**Acceptance Criteria:**
- Chronological order
- Human-readable descriptions
- Filter by action type

---

### 3.9 Admin Dashboard Module

#### FR-ADMIN-001: User Management
**Priority:** High  
**Description:** Admin dapat manage all users  
**Features:** List, search, edit, suspend, delete users  
**Acceptance Criteria:**
- Bulk actions supported
- Advanced filtering
- Export to CSV

#### FR-ADMIN-002: Tenant Management
**Priority:** High  
**Description:** Admin dapat manage tenants  
**Features:** List tenants, view details, suspend, impersonate  
**Acceptance Criteria:**
- Tenant health metrics visible
- Impersonation logged
- Resource usage displayed

#### FR-ADMIN-003: Analytics Dashboard
**Priority:** Medium  
**Description:** Platform analytics overview  
**Metrics:** MRR, churn rate, user growth, active tenants  
**Acceptance Criteria:**
- Real-time data
- Date range selection
- Exportable reports

#### FR-ADMIN-004: System Health
**Priority:** Medium  
**Description:** Monitor system health  
**Metrics:** Queue status, cache status, database connections, uptime  
**Acceptance Criteria:**
- Visual indicators (green/yellow/red)
- Alert thresholds configurable
- Historical data available

---

## 4. Non-Functional Requirements

### 4.1 Performance Requirements
- **NFR-PERF-001:** Page load time < 2 seconds on 3G connection
- **NFR-PERF-002:** API response time p95 < 500ms
- **NFR-PERF-003:** Database queries optimized (no N+1)
- **NFR-PERF-004:** Support 1000 concurrent users minimum
- **NFR-PERF-005:** Cache hit ratio > 80%

### 4.2 Security Requirements
- **NFR-SEC-001:** All passwords hashed dengan Argon2id
- **NFR-SEC-002:** HTTPS enforced in production
- **NFR-SEC-003:** CSRF protection pada semua forms
- **NFR-SEC-004:** SQL injection prevention via Eloquent
- **NFR-SEC-005:** XSS prevention via escaping
- **NFR-SEC-006:** Rate limiting pada API endpoints
- **NFR-SEC-007:** Session timeout after 2 hours inactivity
- **NFR-SEC-008:** Security headers (CSP, HSTS, X-Frame-Options)

### 4.3 Reliability Requirements
- **NFR-REL-001:** System uptime 99.9%
- **NFR-REL-002:** Graceful degradation on dependency failure
- **NFR-REL-003:** Automatic retry pada failed jobs
- **NFR-REL-004:** Database backups daily
- **NFR-REL-005:** Error tracking integration (Sentry/Bugsnag)

### 4.4 Scalability Requirements
- **NFR-SCAL-001:** Horizontal scaling ready
- **NFR-SCAL-002:** Stateless session handling
- **NFR-SCAL-003:** Queue-based async processing
- **NFR-SCAL-004:** Database read replicas support
- **NFR-SCAL-005:** CDN integration untuk static assets

### 4.5 Maintainability Requirements
- **NFR-MAIN-001:** PSR-12 coding standards
- **NFR-MAIN-002:** Type hints throughout codebase
- **NFR-MAIN-003:** Test coverage > 80%
- **NFR-MAIN-004:** PHPStan level 5 minimum
- **NFR-MAIN-005:** Comprehensive inline documentation
- **NFR-MAIN-006:** Changelog maintained

### 4.6 Usability Requirements
- **NFR-USA-001:** Mobile-responsive design
- **NFR-USA-002:** WCAG 2.1 AA accessibility compliance
- **NFR-USA-003:** Loading states untuk all async actions
- **NFR-USA-004:** Error messages helpful dan actionable
- **NFR-USA-005:** Consistent UI patterns

---

## 5. Database Schema

### 5.1 Core Tables

#### users
```sql
- id (UUID)
- name
- email (unique)
- email_verified_at
- password
- two_factor_secret
- two_factor_recovery_codes
- remember_token
- current_team_id (FK)
- profile_photo_path
- created_at
- updated_at
- deleted_at (soft delete)
```

#### tenants
```sql
- id (UUID)
- name
- slug (unique)
- subdomain (unique)
- custom_domain
- owner_id (FK -> users)
- subscription_status
- trial_ends_at
- created_at
- updated_at
- deleted_at
```

#### teams
```sql
- id (UUID)
- tenant_id (FK)
- name
- slug
- description
- logo_path
- created_at
- updated_at
```

#### team_users
```sql
- id
- team_id (FK)
- user_id (FK)
- role
- created_at
- updated_at
```

#### subscriptions (Laravel Cashier)
```sql
- id
- user_id (FK)
- type
- stripe_id
- stripe_status
- stripe_price
- quantity
- trial_ends_at
- ends_at
- created_at
- updated_at
```

#### permissions (Spatie)
```sql
- id
- name
- guard_name
- created_at
- updated_at
```

#### roles (Spatie)
```sql
- id
- name
- guard_name
- created_at
- updated_at
```

#### model_has_permissions
#### model_has_roles
#### role_has_permissions

#### activity_log
```sql
- id
- log_name
- description
- subject_type
- subject_id
- causer_type
- causer_id
- properties (JSON)
- host
- batch_uuid
- created_at
```

#### api_tokens (Sanctum)
```sql
- id
- tokenable_type
- tokenable_id
- name
- token (hashed)
- abilities (JSON)
- last_used_at
- expires_at
- created_at
- updated_at
```

---

## 6. API Specifications

### 6.1 Authentication APIs

#### POST /api/auth/register
```json
Request:
{
  "name": "string",
  "email": "string",
  "password": "string",
  "password_confirmation": "string"
}

Response (201):
{
  "user": { "id": "uuid", "name": "string", "email": "string" },
  "token": "string"
}
```

#### POST /api/auth/login
```json
Request:
{
  "email": "string",
  "password": "string",
  "remember": boolean
}

Response (200):
{
  "user": { ... },
  "token": "string"
}
```

### 6.2 Tenant APIs

#### GET /api/tenants
```
Response (200):
{
  "data": [
    {
      "id": "uuid",
      "name": "string",
      "slug": "string",
      "role": "string"
    }
  ]
}
```

#### POST /api/tenants
```json
Request:
{
  "name": "string",
  "subdomain": "string"
}

Response (201):
{
  "tenant": { ... }
}
```

### 6.3 Subscription APIs

#### GET /api/subscriptions
```
Response (200):
{
  "active": { ... },
  "available_plans": [ ... ]
}
```

#### POST /api/subscriptions/{plan}
```
Response (200):
{
  "checkout_url": "string"
}
```

---

## 7. Testing Strategy

### 7.1 Unit Tests
- Test individual methods dan functions
- Mock external dependencies
- Coverage target: 80%+

### 7.2 Feature Tests
- Test complete user flows
- Database interactions
- HTTP request/response cycles

### 7.3 Integration Tests
- Third-party service integration
- Webhook handling
- Payment processing

### 7.4 Browser Tests (Dusk)
- Critical user journeys
- JavaScript interactions
- Cross-browser compatibility

---

## 8. Deployment Requirements

### 8.1 Environment Configuration
- `.env.example` provided
- All secrets via environment variables
- Different configs per environment

### 8.2 CI/CD Pipeline
- Automated tests on PR
- Code quality checks
- Auto-deploy on main branch merge

### 8.3 Monitoring
- Application monitoring (New Relic/DataDog)
- Error tracking (Sentry)
- Uptime monitoring

---

**Document Version:** 1.0  
**Last Updated:** 2025-07-12  
**Status:** Draft
