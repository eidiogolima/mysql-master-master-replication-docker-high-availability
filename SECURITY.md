# Security Policy

## 🔒 Overview

Security is a top priority for this MySQL Master x Master replication project. This document outlines security policies, best practices, and how to report vulnerabilities.

## 📋 Table of Contents

- [Supported Versions](#supported-versions)
- [Reporting a Vulnerability](#reporting-a-vulnerability)
- [Security Best Practices](#security-best-practices)
- [Production Security Checklist](#production-security-checklist)
- [Known Security Considerations](#known-security-considerations)

## 🛡️ Supported Versions

| Version | Supported          | MySQL Version |
| ------- | ------------------ | ------------- |
| 1.0.x   | :white_check_mark: | 8.0          |

## 🚨 Reporting a Vulnerability

### Please DO NOT report security vulnerabilities through public GitHub issues.

Instead, please report them responsibly by:

1. **Email**: Send details to the repository owner
   - Subject: `[SECURITY] Brief description`
   - Include detailed steps to reproduce
   - Attach logs or screenshots if applicable

2. **What to include**:
   - Type of vulnerability
   - Full paths of source file(s) related to the vulnerability
   - Location of the affected source code (tag/branch/commit or direct URL)
   - Step-by-step instructions to reproduce the issue
   - Proof-of-concept or exploit code (if possible)
   - Impact of the issue, including how an attacker might exploit it

3. **Expected Response**:
   - Initial response within 48 hours
   - Regular updates on progress
   - Credit in security advisory (if desired)

### Disclosure Policy

- We follow responsible disclosure principles
- Security fixes will be released as soon as possible
- Public disclosure only after fix is available
- Reporter will be credited (unless anonymity requested)

## 🔐 Security Best Practices

### 1. Credential Management

#### ⚠️ Default Credentials (Development Only)

The following default credentials are provided for **development only**:

```bash
DB_ROOT_PASSWORD=teste123
DB_PASSWORD=teste123
```

#### ✅ Production Requirements

**CRITICAL**: Change all default passwords before deploying to production!

```bash
# Generate strong passwords
openssl rand -base64 32

# Update .env files on both servers
DB_ROOT_PASSWORD=<strong-random-password>
DB_PASSWORD=<strong-random-password>
```

**Important**: 
- Use different passwords for different environments
- Never commit `.env` files to version control
- Rotate passwords regularly (every 90 days recommended)
- Use password managers for credential storage

### 2. Network Security

#### Firewall Configuration

**Production servers must restrict MySQL port access**:

```bash
# Server 1 (192.168.1.10)
sudo ufw allow from 192.168.1.20 to any port 3306
sudo ufw deny 3306

# Server 2 (192.168.1.20)
sudo ufw allow from 192.168.1.10 to any port 3306
sudo ufw deny 3306
```

#### Network Isolation

- MySQL ports (3306) should **NOT** be exposed to the internet
- Use VPN or private network for server-to-server communication
- phpMyAdmin should be behind authentication/VPN
- Consider using SSH tunnels for administrative access

```bash
# SSH tunnel example for phpMyAdmin
ssh -L 8085:localhost:8085 user@production-server
```

### 3. SSL/TLS Encryption

#### Enabling SSL for Replication

**Highly recommended for production environments**

Generate SSL certificates:

```bash
# On Server 1
openssl req -x509 -newkey rsa:4096 -keyout ca-key.pem -out ca-cert.pem -days 365 -nodes
openssl req -newkey rsa:4096 -keyout server-key.pem -out server-req.pem -nodes
openssl x509 -req -in server-req.pem -days 365 -CA ca-cert.pem -CAkey ca-key.pem -out server-cert.pem -set_serial 01
```

Update `my-config-1.cnf`:

```ini
[mysqld]
ssl-ca=/etc/mysql/ssl/ca-cert.pem
ssl-cert=/etc/mysql/ssl/server-cert.pem
ssl-key=/etc/mysql/ssl/server-key.pem
require_secure_transport=ON
```

Configure replication user with SSL:

```sql
CREATE USER 'replicador'@'%' IDENTIFIED BY 'secure-password' REQUIRE SSL;
```

### 4. Docker Security

#### Container Security

```yaml
# docker-compose.yml security enhancements
services:
  mysql-master-1:
    security_opt:
      - no-new-privileges:true
    read_only: false
    tmpfs:
      - /tmp
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - SETGID
      - SETUID
```

#### Image Security

- Use official MySQL images only
- Pin specific image versions (avoid `latest` tag)
- Regularly update images for security patches

```yaml
image: mysql:8.0.35  # Specify exact version
```

### 5. Access Control

#### Principle of Least Privilege

```sql
-- Replication user should have minimal permissions
GRANT REPLICATION SLAVE ON *.* TO 'replicador'@'specific-ip';
GRANT SELECT ON *.* TO 'replicador'@'specific-ip';
GRANT REPLICATION CLIENT ON *.* TO 'replicador'@'specific-ip';

-- Remove unnecessary privileges
REVOKE ALL PRIVILEGES ON *.* FROM 'replicador'@'%';
```

#### Disable Remote Root Access

```sql
-- Allow root only from localhost
DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');
FLUSH PRIVILEGES;
```

### 6. Audit and Logging

#### Enable MySQL Audit Log

```ini
[mysqld]
# Audit log configuration
plugin-load-add=audit_log.so
audit_log_format=JSON
audit_log_file=/var/log/mysql/audit.log
```

#### Monitor Failed Login Attempts

```sql
-- Check failed login attempts
SELECT * FROM performance_schema.accounts;
SELECT * FROM mysql.general_log WHERE command_type = 'Connect' AND argument LIKE '%Access denied%';
```

### 7. Backup Security

#### Encrypted Backups

```bash
# Backup with encryption
docker exec mysql-master-1 mysqldump -uroot -p${DB_ROOT_PASSWORD} --all-databases | \
  gpg --encrypt --recipient your-email@example.com > backup-$(date +%Y%m%d).sql.gpg

# Restore from encrypted backup
gpg --decrypt backup-20251218.sql.gpg | \
  docker exec -i mysql-master-1 mysql -uroot -p${DB_ROOT_PASSWORD}
```

#### Backup Storage

- Store backups in separate, secure location
- Encrypt backups at rest
- Test backup restoration regularly
- Implement backup retention policy

### 8. Data Protection

#### Sensitive Data Handling

```sql
-- Use encryption for sensitive columns
CREATE TABLE users (
    id INT PRIMARY KEY,
    email VARCHAR(255),
    sensitive_data VARBINARY(255)  -- Store encrypted data
);

-- Enable transparent data encryption (TDE)
-- Requires MySQL Enterprise or Percona Server
```

## ✅ Production Security Checklist

Before deploying to production, ensure:

### Configuration
- [ ] Changed all default passwords
- [ ] Updated `.env` files with strong credentials
- [ ] Removed or secured phpMyAdmin access
- [ ] Configured SSL/TLS for replication
- [ ] Set `require_secure_transport=ON` in MySQL config

### Network
- [ ] Configured firewall rules (port 3306 restricted)
- [ ] Disabled public access to MySQL ports
- [ ] Implemented VPN or private network
- [ ] Configured fail2ban or similar intrusion prevention

### Access Control
- [ ] Applied principle of least privilege to all users
- [ ] Disabled remote root access
- [ ] Created separate users for different applications
- [ ] Reviewed and minimized user permissions

### Monitoring
- [ ] Enabled audit logging
- [ ] Configured log rotation
- [ ] Set up monitoring alerts for failed logins
- [ ] Implemented intrusion detection system

### Docker
- [ ] Using specific image versions (not `latest`)
- [ ] Applied container security options
- [ ] Limited container capabilities
- [ ] Regularly updating base images

### Backup & Recovery
- [ ] Automated backup schedule implemented
- [ ] Backups are encrypted
- [ ] Tested backup restoration process
- [ ] Backups stored in secure, separate location
- [ ] Backup retention policy defined

### Documentation
- [ ] Documented incident response procedures
- [ ] Created runbook for security incidents
- [ ] Trained team on security procedures
- [ ] Established communication channels for security issues

## ⚠️ Known Security Considerations

### 1. Development Environment

The `dev/` environment is configured for ease of use, not security:
- Uses default passwords
- Exposes phpMyAdmin on port 8085
- No SSL/TLS configured
- Should **never** be used in production

### 2. Auto-increment Strategy

The auto-increment configuration (odd/even IDs) is a functional requirement, not a security feature. Do not rely on ID patterns for security.

### 3. GTID Replication

While GTID improves reliability, it doesn't provide security features. Secure the replication channel with SSL.

### 4. phpMyAdmin

phpMyAdmin is a powerful tool but represents a security risk if exposed:
- Disable in production or restrict to internal network only
- Use strong authentication
- Keep updated to latest version
- Consider alternatives like MySQL Workbench with SSH tunnel

### 5. Binary Log Security

Binary logs contain all data changes:
- Protect binary log files from unauthorized access
- Encrypt binary logs if possible
- Rotate and archive securely
- Consider log retention policies

## 🔄 Security Update Policy

- Security patches are prioritized over feature development
- Critical vulnerabilities will be patched within 48 hours
- Security updates will be announced in GitHub releases
- Users will be notified via repository watch notifications

## 📚 Additional Resources

### MySQL Security Documentation
- [MySQL Security Guide](https://dev.mysql.com/doc/refman/8.0/en/security.html)
- [MySQL Replication Security](https://dev.mysql.com/doc/refman/8.0/en/replication-security.html)
- [MySQL Encrypted Connections](https://dev.mysql.com/doc/refman/8.0/en/encrypted-connections.html)

### Docker Security
- [Docker Security Best Practices](https://docs.docker.com/engine/security/)
- [CIS Docker Benchmark](https://www.cisecurity.org/benchmark/docker)

### General Security
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [CWE Database](https://cwe.mitre.org/)

## 📞 Contact

For security concerns that don't constitute vulnerabilities:
- Open a GitHub issue with the `security` label
- Discuss in GitHub Discussions

For actual vulnerabilities:
- Follow the private reporting process described above

---

**Last Updated**: December 18, 2025  
**Version**: 1.0
