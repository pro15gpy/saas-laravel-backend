# Commit Instructions

## Git Commit Message Conventions for Laravel SaaS Boilerplate

This project follows the [Conventional Commits](https://www.conventionalcommits.org/) specification for commit messages. This ensures clear, consistent, and automated changelog generation.

---

## Table of Contents

1. [Commit Message Format](#commit-message-format)
2. [Types](#types)
3. [Scopes](#scopes)
4. [Examples](#examples)
5. [Tools](#tools)

---

## Commit Message Format

Each commit message consists of a **header**, an optional **body**, and an optional **footer**.

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Header

The header is mandatory and must follow this format:

```
<type>(<scope>): <subject>
```

- **type**: The type of change (see Types below)
- **scope**: Optional. The module/area affected
- **subject**: Brief description (max 72 characters)

### Body

Optional. Provide more detailed explanation. Use imperative mood ("add" not "added").

### Footer

Optional. Include breaking changes or issue references.

```
BREAKING CHANGE: description of what changed and migration steps

Closes #123
Refs #456
```

---

## Types

### Primary Types

| Type | Description | Example |
|------|-------------|---------|
| `feat` | New feature | `feat(billing): add Paddle payment gateway` |
| `fix` | Bug fix | `fix(auth): resolve session timeout issue` |
| `docs` | Documentation only | `docs: update installation guide` |
| `style` | Code style (formatting, semicolons) | `style: fix indentation in user controller` |
| `refactor` | Code change that neither fixes a bug nor adds a feature | `refactor(auth): simplify password reset logic` |
| `perf` | Performance improvement | `perf(queries): add index to users table` |
| `test` | Adding or updating tests | `test(billing): add subscription lifecycle tests` |
| `chore` | Maintenance tasks | `chore: update dependencies` |

### Secondary Types

| Type | Description | Example |
|------|-------------|---------|
| `build` | Build system changes | `build: upgrade to Laravel 11` |
| `ci` | CI/CD configuration | `ci: add GitHub Actions workflow` |
| `revert` | Revert previous commit | `revert: revert "feat: add WhatsApp integration"` |

---

## Scopes

Use module/feature names as scopes:

| Scope | Description |
|-------|-------------|
| `auth` | Authentication module |
| `tenant` | Multi-tenancy module |
| `billing` | Billing & subscriptions |
| `rbac` | Roles & permissions |
| `notification` | Email, WhatsApp, in-app notifications |
| `onboarding` | User onboarding wizard |
| `admin` | Admin dashboard |
| `api` | API endpoints |
| `ui` | Frontend components |
| `config` | Configuration files |
| `deps` | Dependencies |
| `docs` | Documentation |

---

## Examples

### Feature Addition

```bash
git commit -m "feat(billing): add Midtrans payment gateway support"
```

```bash
git commit -m "feat(notification): integrate Twilio WhatsApp API

- Add Twilio client configuration
- Implement WhatsApp message sending
- Add delivery status webhooks

Closes #234"
```

### Bug Fix

```bash
git commit -m "fix(auth): prevent brute force attacks with rate limiting"
```

```bash
git commit -m "fix(tenant): resolve cross-tenant data leak

Global scope was not applied to soft-deleted models.
Added TenantScope to all tenant-related queries.

Fixes #189"
```

### Documentation

```bash
git commit -m "docs: add comprehensive security guide"
```

```bash
git commit -m "docs(installation): add Docker setup instructions"
```

### Breaking Change

```bash
git commit -m "feat(auth): migrate to Sanctum from Passport

BREAKING CHANGE: API authentication now uses Laravel Sanctum instead of Passport.
Migration steps:
1. Remove passport routes from routes/api.php
2. Update API middleware to use 'auth:sanctum'
3. Regenerate API tokens using sanctum:create-token

Closes #301"
```

### Refactoring

```bash
git commit -m "refactor(billing): extract payment processing to service class"
```

```bash
git commit -m "refactor: apply SOLID principles to notification system

- Create NotificationStrategy interface
- Extract EmailNotifier and WhatsAppNotifier classes
- Use factory pattern for provider selection"
```

### Performance

```bash
git commit -m "perf(queries): add database indexes for frequent queries"
```

```bash
git commit -m "perf: implement Redis caching for tenant identification

Reduces database queries by 80% for tenant resolution.
Cache TTL set to 5 minutes with automatic invalidation."
```

### Tests

```bash
git commit -m "test(billing): add unit tests for subscription lifecycle"
```

```bash
git commit -m "test: increase coverage to 85%

- Add tests for RBAC middleware
- Test edge cases in onboarding flow
- Mock external API calls"
```

### Chores

```bash
git commit -m "chore: update composer dependencies"
```

```bash
git commit -m "chore(deps): bump stripe/stripe-php from 12.0 to 13.0"
```

---

## Tools

### Commitizen (Interactive Commits)

```bash
# Install globally
composer global require php-llama/commitizen

# Use interactive commit
git cz
```

### Husky + Commitlint (Pre-commit Hooks)

```bash
# Install
npm install --save-dev @commitlint/config-conventional @commitlint/cli husky

# Configure
npx husky install
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit $1'
```

### Laravel Pint (Code Style)

```bash
# Install
composer require laravel/pint --dev

# Run before commit
./vendor/bin/pint
```

### Pre-commit Script

Add to `.git/hooks/pre-commit`:

```bash
#!/bin/bash

# Run linters
./vendor/bin/pint --test
npm run lint

# Run tests
php artisan test --stop-on-failure

# If any command fails, commit is aborted
```

---

## Common Mistakes to Avoid

❌ **Bad:**
```bash
git commit -m "fixed stuff"
git commit -m "WIP"
git commit -m "Update file.php"
git commit -m "asdfasdf"
```

✅ **Good:**
```bash
git commit -m "fix(auth): resolve password reset token expiration"
git commit -m "feat(billing): implement trial period handling"
git commit -m "docs: clarify multi-tenancy configuration"
```

---

## Automated Changelog

Commits following this convention enable automatic changelog generation:

```bash
# Generate changelog from commits
npx conventional-changelog-cli -p angular -i CHANGELOG.md -s
```

---

## Questions?

Need help with commit messages?

- 📖 [Conventional Commits Specification](https://www.conventionalcommits.org/)
- 💬 Ask in Discord: https://discord.gg/yourlink
- 📧 Email: contributors@yoursaas.com

---

*Last Updated: 2025-01-12*
