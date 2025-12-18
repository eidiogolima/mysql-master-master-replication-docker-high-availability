# Contributing to MySQL Master x Master Replication

Thank you for your interest in contributing to this project! This document provides guidelines and standards for contributing.

## 📋 Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Branch Strategy](#branch-strategy)
- [Development Workflow](#development-workflow)
- [Commit Guidelines](#commit-guidelines)
- [Pull Request Process](#pull-request-process)
- [Testing Requirements](#testing-requirements)
- [Documentation](#documentation)

## 🤝 Code of Conduct

- Be respectful and inclusive
- Provide constructive feedback
- Focus on what is best for the community
- Show empathy towards other community members

## 🚀 Getting Started

### Prerequisites

- Docker and Docker Compose installed
- Basic understanding of MySQL replication
- Familiarity with Git and GitHub workflow

### Fork and Clone

1. Fork the repository on GitHub
2. Clone your fork locally:
```bash
git clone https://github.com/YOUR_USERNAME/mysql-master-master-replication-docker-high-availability.git
cd mysql-master-master-replication-docker-high-availability
```

3. Add upstream remote:
```bash
git remote add upstream https://github.com/eidiogolima/mysql-master-master-replication-docker-high-availability.git
```

## 🌿 Branch Strategy

### Branch Types

#### `main`
- **Protected branch** - production-ready code
- All commits must be via Pull Request
- Requires at least 1 approval
- Must pass all tests

#### Feature Branches
**Naming Convention**: `feature/<short-description>`

Examples:
- `feature/add-ssl-support`
- `feature/improved-health-checks`
- `feature/monitoring-dashboard`

#### Bug Fix Branches
**Naming Convention**: `fix/<issue-description>`

Examples:
- `fix/replication-timeout`
- `fix/connection-error-handling`
- `fix/docker-network-issue`

#### Documentation Branches
**Naming Convention**: `docs/<description>`

Examples:
- `docs/update-readme`
- `docs/add-troubleshooting-guide`
- `docs/translate-to-german`

#### Hotfix Branches
**Naming Convention**: `hotfix/<critical-issue>`

Examples:
- `hotfix/security-vulnerability`
- `hotfix/data-loss-prevention`

### Creating a Branch

```bash
# Update your local main branch
git checkout main
git pull upstream main

# Create a new feature branch
git checkout -b feature/your-feature-name
```

## 💻 Development Workflow

### 1. Setup Development Environment

```bash
cd dev/
docker-compose up -d
./setup-replication.sh mysql-master-2
```

### 2. Make Your Changes

- Write clean, maintainable code
- Follow existing code style and patterns
- Add comments for complex logic
- Update documentation as needed

### 3. Test Your Changes

```bash
# Run replication checks
./check-replication.sh

# Run failover tests
./test-failover-resilience.sh

# Check Docker logs
docker logs mysql-master-1
docker logs mysql-master-2
```

### 4. Commit Your Changes

See [Commit Guidelines](#commit-guidelines) below.

## 📝 Commit Guidelines

### Commit Message Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

#### Types

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

#### Examples

```
feat(replication): add SSL/TLS support for secure connections

- Implemented SSL certificate generation
- Updated docker-compose configurations
- Added SSL verification scripts

Closes #42
```

```
fix(setup): resolve GTID inconsistency on startup

Fixed an issue where GTID positions were not properly
synchronized after container restart.

Fixes #38
```

```
docs(readme): add troubleshooting section for common errors

Added detailed troubleshooting guide covering:
- Connection timeouts
- Permission issues
- GTID conflicts
```

### Commit Best Practices

- Use present tense ("add feature" not "added feature")
- Use imperative mood ("move cursor to..." not "moves cursor to...")
- First line should be 50 characters or less
- Reference issues and pull requests where applicable
- Keep commits atomic (one logical change per commit)

## 🔄 Pull Request Process

### Before Submitting

1. **Sync with upstream main**:
```bash
git checkout main
git pull upstream main
git checkout your-branch
git rebase main
```

2. **Run all tests**:
```bash
cd dev/
./check-replication.sh
./test-failover-resilience.sh
```

3. **Update documentation**:
- Update README.md if adding features
- Update TROUBLESHOOTING.md if fixing bugs
- Add comments to complex code

### Submitting the Pull Request

1. Push your branch to your fork:
```bash
git push origin feature/your-feature-name
```

2. Open a Pull Request on GitHub with:
   - Clear title describing the change
   - Detailed description of what and why
   - Reference to related issues
   - Screenshots/logs if applicable

### PR Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Documentation update
- [ ] Performance improvement

## Testing
- [ ] Tested in development environment
- [ ] Replication status verified
- [ ] Failover test passed
- [ ] No data loss confirmed

## Related Issues
Closes #XX

## Screenshots (if applicable)
```

### Review Process

- At least 1 approval required
- Address all review comments
- Keep the PR focused and small when possible
- Be responsive to feedback

## 🧪 Testing Requirements

### Mandatory Tests

All contributions must pass these tests:

#### 1. Replication Status Check
```bash
./check-replication.sh
```
Expected output:
- `Slave_IO_Running: Yes`
- `Slave_SQL_Running: Yes`
- `Seconds_Behind_Master: 0`

#### 2. Failover Resilience Test
```bash
./test-failover-resilience.sh
```
Must pass all stages:
- ✅ Single server failure recovery
- ✅ Dual server failure recovery
- ✅ Data consistency verification

#### 3. Container Health
```bash
docker ps
```
All containers must show `healthy` status

### Test Environments

- **Development**: Test locally in `dev/` environment
- **Production Simulation**: Test in both `prod/server-1/` and `prod/server-2/` configurations

## 📚 Documentation

### When to Update Documentation

- Adding new features → Update README.md
- Fixing bugs → Update TROUBLESHOOTING.md
- Changing configuration → Update relevant .md files
- Adding scripts → Add inline comments and usage examples

### Documentation Standards

- Use clear, concise language
- Provide examples for complex configurations
- Include command-line examples with expected output
- Add troubleshooting tips for common issues
- Keep translations in sync (at least English and Portuguese)

### Translation Guidelines

If updating README or documentation:

1. Update the main README.md (English)
2. Optional: Update other language files in `lang/` directory

## 🔒 Security

### Reporting Security Issues

Instead:
1. Email the maintainer directly
2. Provide detailed description of the vulnerability
3. Include steps to reproduce
4. Suggest a fix if possible

### Security Best Practices

- Never commit passwords or secrets
- Use `.env` files for sensitive data
- Update default credentials in production
- Review security implications of changes

## 🎯 Areas for Contribution

We welcome contributions in:

- **SSL/TLS Implementation**: Secure replication connections
- **Monitoring Tools**: Grafana/Prometheus integration
- **Backup Automation**: Automated backup scripts
- **Performance Optimization**: Query optimization, buffer tuning
- **Documentation**: More languages, detailed guides
- **Testing**: Additional test scenarios
- **Cloud Deployments**: AWS, Azure, GCP guides

## ❓ Questions?

- Open an issue with the `question` label
- Check existing issues and discussions
- Review TROUBLESHOOTING.md

## 📜 License

By contributing, you agree that your contributions will be licensed under the same license as the project.

---

**Thank you for contributing!** 🎉
