# Security

This repository is maintained by **Nexus Communications Technology** (Nexuscomm LLC).

## Reporting a Security Issue

If you discover a security vulnerability, **do not** open a public issue.

Instead, email **office@nexusct.com** with:
- A description of the vulnerability
- Steps to reproduce (if applicable)
- The commit/branch where you observed it
- Your suggested remediation (if any)

We will acknowledge receipt within 2 business days and provide a timeline for remediation.

## What Should Never Be Committed

**Never commit these to any repository:**
- Passwords, API keys, tokens, OAuth secrets
- Private keys, certificates, keystores
- Internal pricing data, dealer costs, margin percentages
- Customer PII (Personally Identifiable Information) or PHI (Protected Health Information)
- Internal network addresses, VPN credentials, or infrastructure details
- Database connection strings with credentials
- Database dumps or backups containing real data
- Cloud provider credentials (AWS, Azure, GCP)

**Always use:**
- Environment variables (via `.env` files that are gitignored)
- WordPress constants in `wp-config.php` (never committed)
- Gitignored runtime config files (e.g., `config.local.js`)
- Secret management services (AWS Secrets Manager, Azure Key Vault, etc.)

## Pre-commit Hooks

This repository uses [pre-commit](https://pre-commit.com/) with multiple security layers:

### Security Tools
- **`gitleaks`** — Industry-standard secret detection
- **`detect-secrets`** — Secondary scanner with baseline tracking
- **Nexus-specific pattern blocks** — Custom rules for known-bad strings

### Setup (One-time per clone)

```bash
# Install pre-commit framework
pip install pre-commit

# Install hooks in your local repository
pre-commit install

# Verify installation
pre-commit run --all-files
```

Hooks will run automatically on every commit. If a secret is detected, the commit will be blocked with a descriptive error message.

### Manual Scanning

Before important pushes or releases:

```bash
pre-commit run --all-files
```

### Bypassing Hooks (NOT RECOMMENDED)

Only bypass hooks for documented false positives:

```bash
git commit --no-verify -m "your message"
```

**⚠️ Warning:** Document why you bypassed hooks in your commit message.

## Security Incident History

### NCT-SEC-2026-04-19-001 (April 19, 2026)

**Incident:** A Google Maps API key was inadvertently committed to the repository in a security configuration file.

**Immediate Response:**
- Google Maps API key rotated immediately
- Admin password changed (was referenced in commit message)
- All team members notified
- Repository audited for additional exposures

**Preventive Measures Implemented:**
- Comprehensive `.gitignore` template deployed across all repositories
- Pre-commit hooks with `gitleaks` and `detect-secrets` made mandatory
- Custom pattern blocks added for known-bad strings and patterns
- Security policy documented and distributed
- Team training on secure credential management

**Lessons Learned:**
- Even security configuration files can expose secrets
- Pre-commit hooks must be installed immediately after cloning
- API keys should never appear in any file, even as "examples"
- Git history is permanent; rotate credentials immediately if exposed

### NCT-SEC-2026-07-29 (July 29, 2026)

**Incident:** Secondary review discovered that the `.pre-commit-config.yaml` file itself contained the rotated API key as a grep literal for blocking purposes, creating an ironic situation where the security tool embedded the secret it was meant to prevent.

**Specific Issue:**
```yaml
# This check inadvertently embedded the key in the repository
if git diff --cached | grep -q "AIzaSyAd72xUaF049-dbkwTAfSvsjQhmp9YLDpk"; then
  echo "❌ BLOCKED: commit contains the live rotated Google Maps key..."
  FAIL=1
fi
```

**Resolution:**
- Removed the specific API key literal from `.pre-commit-config.yaml`
- Generic AIza pattern detection (`AIza[0-9A-Za-z_-]{35}`) already provided adequate coverage
- Updated error messages to be more generic and not reference specific paths
- No additional key rotation needed (key was already rotated in April)

**Lessons Learned:**
- Security tooling must be reviewed for embedded secrets
- Generic pattern matching is preferable to specific literal values
- Defense in depth: multiple pattern checks catch edge cases

## Git History Disclosure

⚠️ **Important Notice:** The Google Maps API key `AIzaSyAd72xUaF049-dbkwTAfSvsjQhmp9YLDpk` exists in this repository's git history (commit `93ede1f` from 2026-04-19).

**Current Status:**
- ✅ Key has been **rotated and is no longer valid**
- ✅ Key cannot be used to access any services
- ✅ Removed from all current files (as of commit `b428e04`)
- ❌ Remains in git history (commits are immutable)

**Why Not Rewrite History?**

While tools like `git filter-branch` or `BFG Repo-Cleaner` can remove content from git history, doing so would require:

1. Force-pushing to rewrite all commit hashes
2. All team members and CI/CD systems re-cloning the repository
3. Breaking any references to old commits (PRs, issues, external documentation)
4. Coordinating across all forks and clones
5. Potential data loss if anyone has unpushed commits

**Risk Assessment:**

Since the key has been rotated and is invalid, the risk of keeping it in history is **minimal** compared to the disruption of rewriting history. This is a documented best practice: **rotate first, consider history rewriting only if necessary**.

**This serves as a permanent reminder:** Once something is committed to git, it's effectively permanent. Always use pre-commit hooks and never commit secrets.

## Best Practices

### For Developers

1. **Install pre-commit hooks immediately** after cloning any repository
2. **Never commit secrets** — use environment variables or gitignored files
3. **Review hook output** — if a commit is blocked, investigate and fix the issue
4. **Rotate immediately** — if a secret is accidentally committed, rotate it before anything else
5. **Report incidents** — email `office@nexusct.com` with full details
6. **Use `.example` files** — for configuration templates, use `.example` or `.template` suffixes

### For API Keys and Credentials

#### Environment Variables (Recommended)

```bash
# .env (gitignored)
GOOGLE_MAPS_API_KEY=your-key-here
OPENAI_API_KEY=your-key-here
DATABASE_URL=postgres://user:pass@localhost/db
```

#### WordPress Constants

```php
// wp-config.php (never committed to public repos)
define('CLASSIFAI_GOOGLE_API_KEY', getenv('GOOGLE_MAPS_API_KEY'));
define('CLASSIFAI_OPENAI_KEY', getenv('OPENAI_API_KEY'));

// In plugin code
$api_key = defined('CLASSIFAI_GOOGLE_API_KEY') ? CLASSIFAI_GOOGLE_API_KEY : '';
if (empty($api_key)) {
    wp_die('API key not configured. Please set CLASSIFAI_GOOGLE_API_KEY in wp-config.php');
}
```

#### Gitignored Config Files

```javascript
// config.local.js (gitignored)
module.exports = {
    googleMapsApiKey: process.env.GOOGLE_MAPS_API_KEY || '',
    openaiApiKey: process.env.OPENAI_API_KEY || '',
};
```

### For Configuration Examples

When creating example configurations:

```javascript
// config.example.js (committed)
module.exports = {
    // Get your API key from https://console.cloud.google.com
    googleMapsApiKey: 'PASTE_YOUR_GOOGLE_MAPS_API_KEY_HERE',
    
    // Get your API key from https://platform.openai.com
    openaiApiKey: 'PASTE_YOUR_OPENAI_API_KEY_HERE',
};
```

**Key points:**
- Use placeholder text like `PASTE_YOUR_API_KEY_HERE`
- Never use real API keys, even expired ones
- Name files with `.example` or `.template` extensions
- Include comments with links to where users can obtain keys

## Automated Security Checks

This repository includes checks for:

- ✅ Google API keys (AIza... pattern)
- ✅ AWS access keys (AKIA... pattern)
- ✅ Private keys (BEGIN PRIVATE KEY, BEGIN RSA PRIVATE KEY, etc.)
- ✅ Known compromised credentials (nexus2026 password)
- ✅ Internal pricing constants (MARGIN = 0.25)
- ✅ Large files (>5MB) that might contain secrets
- ✅ Merge conflict markers
- ✅ Trailing whitespace and file endings

## Compliance

Nexus Communications Technology repositories must comply with:

- **GDPR** — No personal data in repositories
- **HIPAA** — No PHI in repositories (for healthcare projects)
- **PCI DSS** — No payment card data in repositories
- **SOC 2** — Audit trail of all security incidents

## Additional Resources

- [OWASP Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
- [GitHub Secret Scanning](https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning)
- [WordPress Security Best Practices](https://developer.wordpress.org/plugins/security/)
- [Pre-commit Framework Documentation](https://pre-commit.com/)

## Contact

**Security Issues:** office@nexusct.com  
**General Support:** Refer to the main ClassifAI plugin repository  
**Nexus Communications Technology:** https://nexusct.com
