# EPIC & SPRINT PLAN - Laravel SaaS Boilerplate

**Project**: Laravel SaaS Boilerplate (Open Source Free Tier)  
**Timeline**: 8 Weeks (2 Months)  
**Methodology**: Agile Scrum (1-Week Sprints)  
**Version**: 1.0.0

---

## 1. EPIC OVERVIEW

| Epic ID | Epic Name | Description | Priority | Estimated Duration |
| :--- | :--- | :--- | :--- | :--- |
| **E-01** | **Foundation & Auth** | Setup repositori, konfigurasi dasar Laravel, sistem autentikasi lengkap (Email, Social, 2FA). | P0 (Critical) | Sprint 1-2 |
| **E-02** | **Multi-Tenancy Core** | Implementasi arsitektur multi-tenant (Single DB, Scoped), routing subdomain, dan isolasi data. | P0 (Critical) | Sprint 3-4 |
| **E-03** | **Billing & Subscription** | Integrasi Payment Gateway (Stripe/Paddle), manajemen plan, webhook handling, dan trial logic. | P0 (Critical) | Sprint 3-5 |
| **E-04** | **RBAC & Team Mgmt** | Sistem Role-Based Access Control, manajemen tim, invitasi member, dan permission granular. | P1 (High) | Sprint 5-6 |
| **E-05** | **Admin & Observability** | Super Admin Dashboard, activity logging, notification system, dan monitoring setup. | P1 (High) | Sprint 6-7 |
| **E-06** | **Polish & Launch** | Dokumentasi final, testing menyeluruh, CI/CD pipeline, dan rilis publik ke GitHub. | P2 (Medium) | Sprint 8 |

---

## 2. SPRINT BREAKDOWN

### 🏁 SPRINT 1: Foundation Setup & Basic Auth
**Goal**: Repositori siap, database schema dasar, registrasi & login berfungsi.

| Task ID | User Story | Technical Tasks | Est. Points | Assignee | DoD (Definition of Done) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **S1-T1** | Sebagai dev, saya ingin repo terstruktur agar mudah dikembangkan. | - Init Laravel 11<br>- Setup Docker/Sail<br>- Configure ESLint/Prettier/PHP-CS-Fixer<br>- Setup Git Hooks (Husky) | 3 | Lead Dev | Repo clone & run berhasil |
| **S1-T2** | Sebagai user, saya ingin register menggunakan email. | - Install Breeze/Jetstream<br>- Custom Register Controller<br>- Email Verification Flow | 5 | Backend A | Email terkirim & verified |
| **S1-T3** | Sebagai user, saya ingin login aman. | - Login Form UI<br>- Rate Limiting<br>- Remember Me feature | 3 | Backend A | Login sukses, session valid |
| **S1-T4** | Sebagai user, saya ingin reset password jika lupa. | - Forgot Password Request<br>- Token generation & expiry<br>- Reset Password Form | 5 | Backend B | Email reset diterima & berfungsi |
| **S1-T5** | Sebagai sysadmin, saya ingin migrasi database awal. | - Design Users Table<br>- Create Migrations<br>- Seeders (Admin default) | 3 | Backend B | Migration run tanpa error |

**Sprint 1 Deliverables**: Working Auth System, Docker Env, Basic DB Schema.

---

### 🔐 SPRINT 2: Advanced Auth & Profile Management
**Goal**: Keamanan akun meningkat (2FA, Social Login) dan manajemen profil user.

| Task ID | User Story | Technical Tasks | Est. Points | Assignee | DoD |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **S2-T1** | Sebagai user, saya ingin login via Google/GitHub. | - Install Laravel Socialite<br>- Config Providers<br>- Callback handling & linking accounts | 8 | Backend A | Social login berfungsi |
| **S2-T2** | Sebagai user, saya ingin mengaktifkan 2FA. | - Generate QR Code<br>- TOTP Validation<br>- Recovery Codes generation | 8 | Backend B | 2FA aktif & validasi sukses |
| **S2-T3** | Sebagai user, saya ingin update profil & avatar. | - Profile Update Form<br>- Image Upload (S3/Local)<br>- Validation rules | 5 | Frontend A | Profil tersimpan, avatar muncul |
| **S2-T4** | Sebagai user, saya ingin ganti password & hapus akun. | - Current Password Validation<br>- Soft Delete Account<br>- Data Anonymization logic | 5 | Backend A | Akun terhapus aman |
| **S2-T5** | Sebagai dev, saya ingin API Token management. | - Laravel Sanctum Setup<br>- Token creation/revocation UI | 3 | Backend B | API Token bisa dibuat/dihapus |

**Sprint 2 Deliverables**: Social Auth, 2FA, Profile Management, Sanctum API.

---

### 🏢 SPRINT 3: Multi-Tenancy Architecture
**Goal**: Sistem dapat membedakan data antar tenant (organisasi) secara otomatis.

| Task ID | User Story | Technical Tasks | Est. Points | Assignee | DoD |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **S3-T1** | Sebagai owner, saya ingin membuat Organisasi/Tenant baru. | - Tenants Table Migration<br>- Tenant Model & Scope<br>- Creation Wizard UI | 8 | Lead Dev | Tenant tercipta di DB |
| **S3-T2** | Sebagai sistem, saya ingin memisahkan data tiap tenant. | - Global Scope Implementation<br>- Middleware for Tenant Identification<br>- Subdomain Routing config | 13 | Lead Dev | Data tidak bocor antar tenant |
| **S3-T3** | Sebagai user, saya ingin switch antar organisasi. | - Organization Switcher Component<br>- Session update logic<br>- Permission check on switch | 5 | Frontend A | Switch konteks lancar |
| **S3-T4** | Sebagai dev, saya ingin seeding data per tenant. | - Tenant-aware Seeders<br>- Factory patterns update | 3 | Backend B | Seeder berjalan per tenant |
| **S3-T5** | Sebagai user, saya ingin melihat dashboard kosong saat baru buat tenant. | - Dashboard Skeleton<br>- Empty State UI | 3 | Frontend A | Dashboard tampil sesuai tenant |

**Sprint 3 Deliverables**: Multi-tenancy Core, Subdomain Routing, Data Isolation.

---

### 💳 SPRINT 4: Billing Integration (Stripe/Paddle)
**Goal**: Monetisasi siap, user bisa berlangganan dan upgrade plan.

| Task ID | User Story | Technical Tasks | Est. Points | Assignee | DoD |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **S4-T1** | Sebagai owner, saya ingin melihat daftar harga (Pricing). | - Pricing Plans Config (DB/Array)<br>- Pricing Page UI<br>- Feature Comparison Table | 5 | Frontend A | Halaman pricing tampil |
| **S4-T2** | Sebagai owner, saya ingin berlangganan plan. | - Install Cashier (Stripe/Paddle)<br>- Checkout Session creation<br>- Success/Cancel redirect | 8 | Backend A | Checkout flow selesai |
| **S4-T3** | Sebagai sistem, saya ingin menangani webhook pembayaran. | - Webhook Route & Controller<br>- Handle subscription events (created, updated, cancelled)<br>- Sync DB status | 13 | Backend B | Status subs sinkron otomatis |
| **S4-T4** | Sebagai owner, saya ingin manage subscription. | - Portal Link (Update card/Cancel)<br>- Invoice Download history | 5 | Frontend A | Customer portal berfungsi |
| **S4-T5** | Sebagai sistem, saya ingin enforce limit based on plan. | - Middleware for Plan Features<br>- Usage Counting Logic | 5 | Backend A | Fitur terkunci jika plan tidak match |

**Sprint 4 Deliverables**: Payment Integration, Webhooks, Subscription Management.

---

### 👥 SPRINT 5: Team Management & RBAC Basics
**Goal**: Kolaborasi tim dalam satu tenant dengan hak akses dasar.

| Task ID | User Story | Technical Tasks | Est. Points | Assignee | DoD |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **S5-T1** | Sebagai owner, saya ingin mengundang member ke tim. | - Invitations Table<br>- Send Invite Email<br>- Accept Invitation Flow | 8 | Backend B | Undangan terkirim & diterima |
| **S5-T2** | Sebagai owner, saya ingin menetapkan role (Admin, Member). | - Roles & Permissions Schema<br>- Spatie Permission Package Setup<br>- Role Assignment UI | 8 | Lead Dev | Role dapat diubah |
| **S5-T3** | Sebagai member, saya ingin melihat daftar anggota tim. | - Team Members List UI<br>- Search & Filter<br>- Remove Member action | 5 | Frontend A | List anggota akurat |
| **S5-T4** | Sebagai sistem, saya ingin melindungi route berdasarkan role. | - Gate & Policy Definitions<br>- Middleware integration | 5 | Backend A | Akses ditolak jika tidak punya role |
| **S5-T5** | Sebagai user, saya ingin meninggalkan tim. | - Leave Team Action<br>- Ownership transfer validation | 3 | Backend B | User keluar dari tim aman |

**Sprint 5 Deliverables**: Invitation System, Basic RBAC, Team Listing.

---

### ⚙️ SPRINT 6: Advanced RBAC & Notifications
**Goal**: Kontrol akses granular dan sistem notifikasi real-time/email.

| Task ID | User Story | Technical Tasks | Est. Points | Assignee | DoD |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **S6-T1** | Sebagai admin, saya ingin kustomisasi permission. | - Permission Matrix UI<br>- Dynamic Permission creation | 8 | Frontend A | Permission bisa diatur detail |
| **S6-T2** | Sebagai user, saya ingin menerima notifikasi in-app. | - Database Notifications<br>- Notification Bell UI<br>- Mark as read logic | 5 | Fullstack A | Notif muncul & terbaca |
| **S6-T3** | Sebagai user, saya ingin notifikasi email penting. | - Mail Queue Setup<br>- Notification Classes (Welcome, Billing, Invite) | 5 | Backend B | Email terkirim via queue |
| **S6-T4** | Sebagai admin, saya ingin melihat audit log aktivitas. | - Activity Log Package (e.g., spatie/laravel-activitylog)<br>- Log Viewer UI | 8 | Backend A | Aktivitas tercatat & terlihat |
| **S6-T5** | Sebagai dev, saya ingin broadcast event realtime (opsional). | - Laravel Reverb/Pusher Setup<br>- Simple "New Notification" event | 5 | Backend B | Realtime notif bekerja |

**Sprint 6 Deliverables**: Granular Permissions, Notification System, Activity Logs.

---

### 🛡️ SPRINT 7: Super Admin & Polish
**Goal**: Dashboard untuk pemilik platform (SaaS Owner) dan perbaikan UX.

| Task ID | User Story | Technical Tasks | Est. Points | Assignee | DoD |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **S7-T1** | Sebagai Super Admin, saya ingin melihat seluruh tenant. | - Super Admin Middleware<br>- Tenant List & Stats Dashboard<br>- Impersonate User feature | 8 | Lead Dev | Dashboard Super Admin siap |
| **S7-T2** | Sebagai Super Admin, saya ingin manage global plans. | - CRUD Plans UI<br>- Feature toggles per plan | 5 | Backend A | Plan bisa diedit dari admin |
| **S7-T3** | Sebagai user, saya ingin pengalaman mobile yang baik. | - Responsive Testing & Fixes<br>- Mobile Menu Optimization | 5 | Frontend A | Lighthouse score > 90 Mobile |
| **S7-T4** | Sebagai dev, saya ingin error handling yang rapi. | - Custom Exception Pages<br>- Logging Channel Setup (Sentry/Logflare) | 3 | Backend B | Error page custom tampil |
| **S7-T5** | Sebagai user, saya ingin performa cepat. | - Query Optimization (N+1 fix)<br>- Caching Strategy (Redis) | 8 | Backend A | Response time < 200ms |

**Sprint 7 Deliverables**: Super Admin Panel, Performance Tuning, Error Handling.

---

### 🚀 SPRINT 8: Testing, Docs & Launch
**Goal**: Kode stabil, terdokumentasi, dan siap rilis publik.

| Task ID | User Story | Technical Tasks | Est. Points | Assignee | DoD |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **S8-T1** | Sebagai dev, saya ingin automated testing. | - PHPUnit/ Pest Setup<br>- Write Feature Tests (Auth, Billing, Tenancy)<br>- CI Pipeline (GitHub Actions) | 13 | QA / Lead | Green build on PR |
| **S8-T2** | Sebagai pengguna, saya ingin dokumentasi lengkap. | - Write README.md (Installation, Features)<br>- API Documentation (Postman/Scalar)<br>- Deployment Guide | 8 | Tech Writer | Docs published |
| **S8-T3** | Sebagai owner, saya ingin lisensi jelas. | - Add MIT License<br>- Code Cleanup & Commenting<br>- Remove hardcoded secrets | 3 | Lead Dev | Repo bersih & legal |
| **S8-T4** | Sebagai dev, saya ingin deployment mudah. | - Docker Compose Prod Ready<br>- Deploy Script (Vapor/Fortress/Manual)<br>- Environment Example (.env.example) | 5 | DevOps | One-click deploy ready |
| **S8-T5** | **LAUNCH DAY** | - Tag Release v1.0.0<br>- Submit to GitHub Trends/Product Hunt<br>- Social Media Blast | - | All | **Public Release** |

**Sprint 8 Deliverables**: v1.0.0 Release, Full Test Coverage, Public Documentation.

---

## 3. RESOURCE & ROLES

| Role | Responsibilities | Recommended Skills |
| :--- | :--- | :--- |
| **Lead Developer** | Architecture, Core Tenancy, Security, Code Review | Laravel Expert, System Design |
| **Backend Dev A** | Auth, Billing, API, Integrations | PHP, Stripe API, SQL |
| **Backend Dev B** | Notifications, Queues, Testing, Maintenance | Redis, Mail, Pest/PHPUnit |
| **Frontend Dev** | UI Components, Dashboards, Responsiveness | TailwindCSS, Alpine/React/Vue, Livewire |
| **DevOps/QA** | CI/CD, Docker, Server Setup, Testing | GitHub Actions, Docker, Linux |

*(Catatan: Dalam tim kecil, 1 orang bisa merangkap beberapa role)*

---

## 4. DEFINITION OF DONE (DoD) GLOBAL

Sebuah task dianggap "Done" jika:
1.  Kode sudah di-commit dan di-push ke branch feature.
2.  Pull Request (PR) dibuat dan melewati review minimal 1 orang.
3.  Semua Automated Tests (CI) lulus (Green).
4.  Tidak ada bug kritis yang diketahui.
5.  Dokumentasi (jika perlu) sudah diupdate.
6.  Kode mengikuti standar PSR-12 dan style guide project.

---

## 5. RISK MANAGEMENT

| Risk | Impact | Probability | Mitigation Strategy |
| :--- | :--- | :--- | :--- |
| **Complexity of Tenancy** | High | High | Gunakan package proven (seperti `stancl/tenancy`) atau stick to simple single-db scoping di awal. |
| **Payment Gateway Issues** | High | Medium | Gunakan Sandbox mode extensively; Siapkan fallback manual verification. |
| **Scope Creep** | Medium | High | Strictly follow MVP features; Move "nice-to-have" to v1.1 backlog. |
| **Burnout** | High | Medium | Maintain sustainable pace; Jangan lembur di Sprint 1-2. |

---

## 6. BACKLOG (Post-Launch Ideas)

Fitur yang ditunda untuk v1.1 dan seterusnya:
- [ ] Multi-language support (i18n).
- [ ] White-labeling (Custom domain per tenant).
- [ ] Advanced Analytics Dashboard.
- [ ] Marketplace/Plugin System.
- [ ] GraphQL API Support.
- [ ] Mobile App (Flutter/React Native) starter kit.
