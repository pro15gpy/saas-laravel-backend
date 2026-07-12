# Contributing to Laravel SaaS Boilerplate

Thank you for considering contributing to the Laravel SaaS Boilerplate! We appreciate your interest in making this project better.

## Table of Contents

1. [Code of Conduct](#code-of-conduct)
2. [How to Contribute](#how-to-contribute)
3. [Development Setup](#development-setup)
4. [Coding Standards](#coding-standards)
5. [Pull Request Process](#pull-request-process)
6. [Reporting Issues](#reporting-issues)
7. [Recognition](#recognition)

---

## Code of Conduct

This project adheres to the [Contributor Covenant Code of Conduct](https://www.contributor-covenant.org/version/2/0/code_of_conduct.html). By participating, you are expected to uphold this code. Please report unacceptable behavior to conduct@yoursaas.com.

### Our Pledge

We pledge to make participation in our project a harassment-free experience for everyone, regardless of age, body size, disability, ethnicity, sex characteristics, gender identity and expression, level of experience, education, socio-economic status, nationality, personal appearance, race, religion, or sexual identity and orientation.

---

## How to Contribute

### Types of Contributions We Welcome

1. **Bug Reports**: Found a bug? Open an issue with detailed reproduction steps.
2. **Feature Requests**: Have an idea? Submit a feature request with use cases.
3. **Documentation**: Improve existing docs or add new guides.
4. **Code Contributions**: Fix bugs or implement new features.
5. **Testing**: Add test cases or improve test coverage.
6. **Security Reports**: Report vulnerabilities responsibly (see SECURITY.md).

### First Time Contributors

New to open source? Here's how to start:

1. Look for issues labeled `good first issue` or `help wanted`
2. Fork the repository
3. Create a branch for your changes
4. Make your changes following our coding standards
5. Submit a pull request

---

## Development Setup

### Prerequisites

- PHP 8.3+
- Composer 2.6+
- Node.js 18.x+
- MySQL 8.0+ / PostgreSQL 15+
- Redis 7.x

### Setup Steps

```bash
# Fork and clone
git clone https://github.com/YOUR_USERNAME/laravel-saas-boilerplate.git
cd laravel-saas-boilerplate

# Install dependencies
composer install
npm install

# Setup environment
cp .env.example .env
php artisan key:generate

# Run migrations
php artisan migrate --seed

# Start development
npm run dev
php artisan serve
```

---

## Coding Standards

### PHP Standards

- Follow [PSR-12 Coding Standard](https://www.php-fig.org/psr/psr-12/)
- Use [Laravel PHP CS Fixer](https://github.com/laravel/pint) for formatting
- Maintain type hints where possible
- Write meaningful variable and method names

```bash
# Check code style
./vendor/bin/pint --test

# Auto-fix code style
./vendor/bin/pint
```

### JavaScript Standards

- Follow ESLint configuration
- Use Prettier for formatting
- Write JSDoc comments for complex functions

```bash
# Check code style
npm run lint

# Auto-fix
npm run lint:fix
```

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/) format:

```
feat: add WhatsApp notification support
fix: resolve tenant isolation bug
docs: update installation guide
test: add unit tests for billing module
refactor: simplify authentication flow
```

See [COMMIT_INSTRUCTIONS.md](COMMIT_INSTRUCTIONS.md) for details.

---

## Pull Request Process

### Before Submitting

1. **Fork** the repository
2. **Create a branch** from `main`:
   ```bash
   git checkout -b feat/your-feature-name
   # or
   git checkout -b fix/issue-number-description
   ```
3. **Make changes** following coding standards
4. **Write/update tests** for your changes
5. **Update documentation** if needed
6. **Run tests** locally:
   ```bash
   composer test
   npm run test
   ```

### PR Checklist

Before submitting your PR, ensure:

- [ ] Code follows PSR-12 standards
- [ ] Tests pass locally
- [ ] Documentation is updated
- [ ] Commit messages follow convention
- [ ] No sensitive data committed
- [ ] CHANGELOG.md updated (if applicable)

### Submitting PR

1. Push your branch:
   ```bash
   git push origin feat/your-feature-name
   ```
2. Open Pull Request on GitHub
3. Fill out the PR template completely
4. Link related issues
5. Wait for review

### Review Process

1. **Automated Checks**: CI/CD will run tests and linting
2. **Code Review**: Maintainers will review within 48 hours
3. **Feedback**: Address any requested changes
4. **Approval**: Once approved, PR will be merged

---

## Reporting Issues

### Bug Reports

Use the bug report template and include:

- Clear description
- Steps to reproduce
- Expected vs actual behavior
- Environment details (PHP version, OS, etc.)
- Error logs/screenshots

### Feature Requests

Use the feature request template with:

- Problem statement
- Proposed solution
- Alternatives considered
- Use cases

### Security Vulnerabilities

**DO NOT** create public issues for security vulnerabilities. Email security@yoursaas.com with:

- Description of vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if any)

See [SECURITY.md](docs/SECURITY.md) for details.

---

## Recognition

We believe in recognizing contributors:

### Contributor Levels

- **First-time Contributor**: Welcome badge on Discord
- **Regular Contributor**: Listed in README.md
- **Core Contributor**: Write access to repository
- **Maintainer**: Decision-making authority

### Benefits

- Shout-out in release notes
- Featured in monthly community updates
- Swag for significant contributions (t-shirts, stickers)
- Invitation to private contributor channels

### Hall of Fame

Top contributors are featured in our README.md Hall of Fame section.

---

## Questions?

Need help? Reach out via:

- 💬 Discord: https://discord.gg/yourlink
- 📧 Email: contributors@yoursaas.com
- 🐛 GitHub Discussions: https://github.com/yourusername/laravel-saas-boilerplate/discussions

---

## License

By contributing, you agree that your contributions will be licensed under the [MIT License](LICENSE).

---

**Thank you for contributing to Laravel SaaS Boilerplate!** 🎉
