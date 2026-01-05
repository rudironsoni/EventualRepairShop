# Security Policy

## Supported Versions

We take security seriously and are committed to addressing security issues promptly.

| Version | Supported          |
| ------- | ------------------ |
| Latest  | :white_check_mark: |

**Note:** This project is currently under active development. We recommend always using the latest version from the `main` branch.

## Reporting a Vulnerability

We appreciate your efforts to responsibly disclose your findings and will make every effort to acknowledge your contributions.

### How to Report

**Please do not report security vulnerabilities through public GitHub issues.**

Instead, please report them by:

1. **GitHub Security Advisories** (Preferred)
   - Navigate to the Security tab of this repository
   - Click "Report a vulnerability"
   - Provide detailed information about the vulnerability

2. **Direct Contact**
   - Create a private issue and mention it's security-related
   - We will respond promptly to arrange a secure communication channel

### What to Include

When reporting a vulnerability, please include:

- **Description**: Clear description of the vulnerability
- **Impact**: What could an attacker do with this vulnerability?
- **Steps to Reproduce**: Detailed steps to reproduce the issue
- **Affected Components**: Which services/modules are affected?
- **Suggested Fix**: If you have ideas on how to fix it (optional)
- **Environment Details**: .NET version, OS, deployment scenario

### Example Report

```
Title: SQL Injection in RepairOrder API endpoint

Description:
The GET /api/v1/repair-orders endpoint is vulnerable to SQL injection
through the 'filter' query parameter.

Impact:
An attacker could potentially read, modify, or delete data from the 
event store database.

Steps to Reproduce:
1. Send GET request to /api/v1/repair-orders?filter=1' OR '1'='1
2. Observe that query returns all records regardless of authorization

Affected Components:
- WebAPI project
- RepairOrder.Application layer

Environment:
- .NET 7.0
- Windows 11
- Development environment
```

## What to Expect

- **Acknowledgment**: We will acknowledge receipt of your vulnerability report within 48 hours
- **Assessment**: We will assess the vulnerability and determine severity within 7 days
- **Updates**: We will keep you informed of our progress
- **Resolution**: We aim to release fixes for critical vulnerabilities within 30 days
- **Credit**: With your permission, we will credit you in the security advisory

## Security Best Practices

### For Contributors

- Never commit secrets, API keys, or credentials
- Use environment variables for sensitive configuration
- Follow secure coding practices outlined in CONTRIBUTING.md
- Keep dependencies up to date
- Run security scanning tools before submitting PRs

### For Users

- **Change Default Credentials**: The docker-compose files contain default credentials for development only
- **Use HTTPS**: Always use HTTPS/TLS in production environments
- **Update Regularly**: Keep your deployment up to date with the latest version
- **Secure Infrastructure**: Properly secure SQL Server, RabbitMQ, and other infrastructure components
- **Network Isolation**: Use proper network segmentation for microservices
- **Access Control**: Implement proper authentication and authorization

## Known Security Considerations

### Development Environment

The following are known security considerations for **development environments only**:

1. **Hardcoded Credentials in Docker Compose**
   - The `docker-compose.Development.Infrastructure.yaml` contains default credentials
   - **DO NOT use these in production**
   - Change all passwords before deploying to production

2. **CORS Configuration**
   - WebAPI is configured with `AllowAnyOrigin()` for development
   - **Restrict CORS** in production to specific domains

3. **SQL Server SA Password**
   - Default SA password is `!MyStrongPassword`
   - **Change immediately** for any non-development environment

4. **RabbitMQ Guest Account**
   - Default credentials are `guest/guest`
   - **Disable guest account** and create dedicated users in production

### Production Recommendations

For production deployments:

- Use Azure Key Vault, AWS Secrets Manager, or similar for secrets
- Implement proper authentication (JWT, OAuth2, etc.)
- Enable SQL Server encryption (TDE)
- Use RabbitMQ with TLS
- Implement rate limiting on API endpoints
- Enable audit logging
- Use network policies to restrict inter-service communication
- Regular security scanning and penetration testing

## Security Updates

We will publish security advisories for any vulnerabilities found in this project. Subscribe to repository notifications to stay informed.

## Vulnerability Disclosure Timeline

- **Day 0**: Vulnerability reported
- **Day 1-2**: Acknowledgment sent to reporter
- **Day 3-7**: Vulnerability assessed and severity determined
- **Day 8-30**: Fix developed and tested
- **Day 30**: Security advisory published (if applicable)
- **Day 30+**: Fix released and disclosure coordinated with reporter

## Questions?

If you have questions about this security policy, please open a discussion in GitHub Discussions (for general questions) or follow the reporting process above (for security concerns).

---

Thank you for helping keep EventualRepairShop and its users safe!
