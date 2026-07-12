# saas-laravel-backend

SaaS Laravel Boilerplate - Free & Open Source

A production-ready Laravel SaaS starter kit with multi-tenancy, subscription billing, team management, and RBAC.

## 📚 Documentation

- [Product Requirements (PRD)](./docs/PRD.md)
- [Software Requirements Specification (SRS)](./docs/SRS.md)
- [System Design](./docs/DESIGN.md)
- [Implementation Tasks & Roadmap](./docs/TASKS.md)

## 🚀 Quick Start

```bash
# Clone the repository
git clone https://github.com/your-org/saas-laravel-backend.git

# Install dependencies
composer install
npm install

# Setup environment
cp .env.example .env
php artisan key:generate

# Run migrations
php artisan migrate

# Start development server
php artisan serve
```

## ✨ Features

### Core Features (MVP)
- ✅ **Authentication**: Registration, login, password reset, 2FA, social auth
- ✅ **Multi-Tenancy**: Subdomain routing, tenant isolation, custom domains
- ✅ **Subscription Billing**: Stripe & Paddle integration, webhooks, invoices
- ✅ **Team Management**: Create teams, invite members, role assignment
- ✅ **RBAC**: Roles & permissions, policy-based access control
- ✅ **User Profile**: Profile management, API tokens, account deletion

### Secondary Features
- ✅ Email & in-app notifications
- ✅ Activity logging & audit trail
- ✅ Admin dashboard
- ✅ RESTful API with Sanctum

## 🛠 Tech Stack

- **Backend**: Laravel 11/12, PHP 8.2+
- **Database**: MySQL 8.0 / PostgreSQL 15+
- **Cache/Queue**: Redis
- **Frontend**: Blade + Alpine.js or Inertia.js + React/Vue
- **UI**: Tailwind CSS
- **Testing**: PHPUnit + Pest
- **DevOps**: Docker, GitHub Actions

## 📦 Key Packages

- [spatie/laravel-permission](https://github.com/spatie/laravel-permission) - RBAC
- [laravel/cashier-stripe](https://laravel.com/docs/billing) - Stripe billing
- [laravel/cashier-paddle](https://laravel.com/docs/billing) - Paddle billing
- [stancl/tenancy](https://tenancy.stancl.io/) - Multi-tenancy
- [spatie/laravel-activitylog](https://github.com/spatie/laravel-activitylog) - Activity logging
- [laravel/sanctum](https://laravel.com/docs/sanctum) - API authentication

## 🎯 Roadmap

### Phase 1: Foundation (Weeks 1-2)
- Project setup & architecture
- Authentication system
- Basic multi-tenancy

### Phase 2: Core Features (Weeks 3-4)
- Subscription & billing (Stripe)
- Team management
- RBAC system

### Phase 3: Polish & Testing (Weeks 5-6)
- Complete test coverage
- Documentation
- Security audit

### Phase 4: Launch (Weeks 7-8)
- Beta testing
- Public launch
- Community outreach

See [TASKS.md](./docs/TASKS.md) for detailed implementation plan.

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guide](./CONTRIBUTING.md) for details.

## 📄 License

This project is open-sourced software licensed under the [MIT license](./LICENSE).

## 🙏 Acknowledgments

Inspired by:
- [php-saas/php-saas](https://github.com/php-saas/php-saas)
- [Laravel Spark](https://spark.laravel.com/)
- [Laravel Jetstream](https://jetstream.laravel.com/)

---

**Status**: In Development  
**Version**: 0.1.0  
**Last Updated**: 2025-07-12
