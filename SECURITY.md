# Security

This repository is maintained by **Nexus Communications Technology** (Nexuscomm LLC).

## Reporting a security issue

If you discover a security vulnerability, **do not** open a public issue.

Instead, email **office@nexusct.com** with:
- A description of the vulnerability
- Steps to reproduce (if applicable)
- The commit/branch where you observed it
- Your assessment of severity (if applicable)

We will acknowledge within 2 business days and provide a timeline for remediation.

## What belongs in this repo

**Never commit:**
- Passwords, API keys, tokens, private keys, certificates
- Internal pricing data, dealer costs, margin percentages
- Customer PII (Personally Identifiable Information) or PHI (Protected Health Information)
- Internal network addresses or credentials
- Database dumps or backups containing real data
- Session tokens, OAuth tokens, or JWT secrets
- Encryption keys or initialization vectors

Use environment variables, `.env` files (gitignored), or a secure runtime
config endpoint — never hardcode secrets.

## Security best practices

### Environment variables
- Copy `.env.example` to `.env` and fill in your values
- NEVER commit `.env` to git (it's in `.gitignore`)
- Use different API keys for dev/staging/production
- Rotate keys immediately if accidentally exposed

### WordPress-specific security
- Always use `$wpdb->prepare()` for database queries with user input
- Verify nonces with `wp_verify_nonce()` for all form submissions
- Escape output with `esc_html()`, `esc_attr()`, or `esc_url()`
- Never use `eval()` - it's dangerous and banned in WordPress plugins
- Check user capabilities with `current_user_can()` before privileged operations
- Sanitize input with `sanitize_text_field()`, `sanitize_email()`, etc.

### API security
- Enable rate limiting on all AI/ML provider APIs
- Monitor API usage for unusual patterns
- Set spending limits on paid services (OpenAI, Google Cloud, etc.)
- Enable MFA on all provider accounts
- Review and revoke unused API keys regularly

## Pre-commit hooks

This repo uses [pre-commit](https://pre-commit.com/) with multiple security layers:
- `gitleaks` (v8.30.1) — detects committed secrets
- `detect-secrets` (v1.5.0) — second-layer scanner
- Nexus-specific pattern blocks — rejects known-bad strings
- WordPress security checks — SQL injection, XSS, nonce verification
- File hygiene checks — YAML/JSON validation, merge conflicts, etc.

**Setup (one-time per clone):**

```bash
pip install pre-commit
pre-commit install
```

Hooks will run automatically on every commit.

**Manual scan:**
```bash
pre-commit run --all-files
```

## GitHub Actions security scanning

This repository includes automated security scanning via GitHub Actions:
- **Secret scanning** - Gitleaks on every push
- **Dependency scanning** - npm/composer audit for vulnerable packages
- **CodeQL analysis** - Static analysis for security vulnerabilities
- **Security hardening checks** - Validates .env.example, SECURITY.md, .gitignore
- **WordPress security** - PHP_CodeSniffer with WordPress security standards

The workflow runs on:
- Every push to main/develop
- Every pull request
- Weekly scheduled scans (Mondays at 9am UTC)
- Manual trigger via GitHub Actions UI

## Incident history

### 2026-04-19 — Credential exposure incident (NCT-SEC-2026-04-19-001)

**What happened:**
A Google Maps API key (`AIzaSyAd72xUaF049-dbkwTAfSvsjQhmp9YLDpk`) was accidentally committed to the repository in commit `93ede1f` within the `.pre-commit-config.yaml` file as part of a security check pattern.

**Impact:**
- Key was exposed in public repository
- Key was valid at time of exposure
- No evidence of unauthorized usage detected

**Remediation:**
1. Key was rotated on 2026-04-19 (same day as discovery)
2. Hardcoded key reference removed from `.pre-commit-config.yaml` in commit `b428e04`
3. Generic pattern detection (`AIza[0-9A-Za-z_-]{35}`) remains active
4. Enhanced pre-commit hooks to prevent future occurrences
5. Added GitHub Actions security scanning workflow

**Current status:** RESOLVED
- Old key is revoked and non-functional
- New key is stored securely in environment variables
- Git history still contains the old key (intentional - see below)

**Why git history was not rewritten:**
1. Key was already rotated and is no longer valid
2. Risk of disrupting collaborators and CI/CD pipelines
3. Industry standard practice: rotate first, history rewriting is optional
4. Transparency: keeping the incident visible serves as a reminder and learning opportunity

**Lessons learned:**
1. Never embed secrets in config files, even for validation purposes
2. Use regex patterns without specific key values
3. Regular automated scanning catches issues early
4. Fast rotation minimizes exposure window

### Other security incidents
- 2026-04-19 — Admin password also rotated as precautionary measure

## Compliance

This project follows security standards from:
- **OWASP Top 10 (2021)** - Web application security risks
- **WordPress Security Best Practices** - Official WordPress.org guidelines
- **GDPR** - No PII stored in repository
- **GitHub Secret Scanning** - Automated detection enabled
- **CWE Top 25** - Common Weakness Enumeration

## Security tools and resources

### Detection tools
- [Gitleaks](https://github.com/gitleaks/gitleaks) - Secret scanning
- [detect-secrets](https://github.com/Yelp/detect-secrets) - Baseline secret detection
- [GitHub Secret Scanning](https://docs.github.com/en/code-security/secret-scanning) - Platform-level detection

### WordPress security
- [WordPress Security White Paper](https://wordpress.org/about/security/)
- [Plugin Security Handbook](https://developer.wordpress.org/plugins/security/)
- [WPScan](https://wpscan.com/) - WordPress vulnerability database

### AI/ML API security
- [OpenAI API Best Practices](https://platform.openai.com/docs/guides/safety-best-practices)
- [Google Cloud Security](https://cloud.google.com/security/best-practices)
- [AWS Security Best Practices](https://aws.amazon.com/security/best-practices/)

## Contact

**Security issues:** office@nexusct.com  
**General inquiries:** Jim Mazza (jim.mazza@nexusct.com)

## Review schedule

- **Pre-commit hooks:** Updated as new versions release
- **Dependency audit:** Weekly automated scans via GitHub Actions
- **Manual security review:** Quarterly (every 90 days)
- **Next scheduled review:** Check GitHub Actions for weekly scan results
