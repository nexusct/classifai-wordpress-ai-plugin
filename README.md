# ClassifAI WordPress Plugin - Security Configuration

This repository contains security configuration and testing fixtures for the ClassifAI WordPress AI plugin.

## Overview

ClassifAI is a WordPress plugin that provides AI-powered content classification, generation, and analysis features. This repository maintains the security baseline and pre-commit hooks used across Nexus Communications Technology projects.

## Repository Contents

- **`.pre-commit-config.yaml`** - Pre-commit security hooks for secret detection
- **`SECURITY.md`** - Security policy and incident reporting guidelines
- **`.gitignore`** - Comprehensive ignore patterns for secrets and sensitive data
- **`tests/test-plugin/`** - Mock API response fixtures for testing
- **`webpack.config.js`** - Webpack configuration for plugin builds
- **`tsconfig.json`** - TypeScript configuration

## Security Features

This repository implements multiple layers of security protection:

### 1. Pre-commit Hooks

The `.pre-commit-config.yaml` file implements automated secret detection using:

- **[gitleaks](https://github.com/gitleaks/gitleaks)** - Industry-standard secret scanner
- **[detect-secrets](https://github.com/Yelp/detect-secrets)** - Additional secret detection layer
- **Custom pattern blocks** - Nexus-specific forbidden patterns including:
  - Google Maps API keys (AIza... pattern)
  - AWS access keys (AKIA... pattern)
  - Private keys (PEM, RSA, etc.)
  - Retired credentials from past incidents

### 2. Gitignore Protection

The `.gitignore` file blocks common secret storage patterns:

- Environment files (`.env`, `.env.*`)
- Private keys and certificates (`.pem`, `.key`, `.crt`)
- Cloud provider credentials (`.aws/`, `.azure/`, `.gcloud/`)
- Configuration files that may contain secrets
- Database dumps and backups
- Nexus-specific private data files

### 3. Security Policy

The `SECURITY.md` file provides:

- Vulnerability reporting procedures
- Contact information for security issues
- Guidelines on what should never be committed
- Setup instructions for security tools

## Setup Instructions

### Prerequisites

- Python 3.7 or higher
- pip (Python package installer)

### One-time Setup (Per Repository Clone)

1. **Install pre-commit:**

   ```bash
   pip install pre-commit
   ```

2. **Install the pre-commit hooks:**

   ```bash
   pre-commit install
   ```

3. **Verify installation:**

   ```bash
   pre-commit run --all-files
   ```

### Usage

Once installed, the pre-commit hooks will automatically run on every commit. If a hook detects a potential secret or security issue, the commit will be blocked with a descriptive error message.

#### Manual Scanning

To manually scan all files (recommended before important pushes):

```bash
pre-commit run --all-files
```

#### Bypassing Hooks (NOT RECOMMENDED)

In rare cases where you need to bypass the hooks (e.g., for false positives), use:

```bash
git commit --no-verify -m "your message"
```

**⚠️ Warning:** Only bypass hooks if you are absolutely certain the flagged content is safe.

## Security Incident History

### NCT-SEC-2026-04-19-001 (April 19, 2026)

A credential exposure incident occurred where a Google Maps API key was inadvertently committed to the repository. The incident response included:

- **Immediate Actions:**
  - API key rotated immediately
  - Admin password changed
  - All team members notified

- **Preventive Measures Implemented:**
  - Comprehensive `.gitignore` template deployed
  - Pre-commit hooks with gitleaks and detect-secrets installed
  - Custom pattern blocks for known-bad strings added
  - Security policy documented in `SECURITY.md`

- **Key Lessons:**
  - Never commit API keys, even in configuration examples
  - Always use environment variables or gitignored config files
  - Pre-commit hooks are mandatory for all repositories

### NCT-SEC-2026-07-29 (July 29, 2026)

A secondary issue was discovered where the pre-commit configuration itself contained a hardcoded API key as a grep literal for blocking purposes. This created an ironic situation where the security tool contained the secret it was meant to prevent.

- **Issue:** The `.pre-commit-config.yaml` included a specific check for the rotated API key `AIzaSyAd72xUaF049-dbkwTAfSvsjQhmp9YLDpk`, embedding the key in the repository.
  
- **Resolution:**
  - Removed the specific API key literal from pre-commit config
  - Generic AIza pattern detection (`AIza[0-9A-Za-z_-]{35}`) already covers all Google API keys
  - Updated error message to be more generic and not reference specific filenames

- **Note:** The exposed key remains in git history at commit `93ede1f` but has been rotated and is no longer valid. Anyone with access to the repository history can see this historical key, but it cannot be used.

## Git History Note

⚠️ **Important:** The Google Maps API key `AIzaSyAd72xUaF049-dbkwTAfSvsjQhmp9YLDpk` exists in this repository's git history (commit `93ede1f` from 2026-04-19). This key has been **rotated and is no longer valid**. 

Removing it from git history would require a force-push and rewriting history, which could break existing clones. Since the key is already rotated and invalid, it poses no security risk. This serves as a reminder that **once something is committed to git, it's very difficult to fully remove**.

### Why Not Rewrite History?

While tools like `git filter-branch` or `BFG Repo-Cleaner` can remove sensitive data from git history, they require:

1. Force-pushing to rewrite history
2. All team members to re-clone the repository
3. Potential breakage of any references to old commits
4. Coordination across all forks and clones

Since the key has been rotated and is no longer valid, the risk of keeping it in history is minimal compared to the disruption of rewriting history.

## Best Practices

### For Developers

1. **Never commit secrets** - Use environment variables or gitignored config files
2. **Install pre-commit hooks** - First thing after cloning any repository
3. **Review hook output** - If a commit is blocked, investigate why
4. **Rotate immediately** - If a secret is accidentally committed, rotate it immediately
5. **Report incidents** - Email `office@nexusct.com` with details

### For API Keys and Credentials

1. **Use environment variables:**

   ```bash
   export GOOGLE_MAPS_API_KEY="your-key-here"
   ```

2. **Use gitignored config files:**

   ```javascript
   // config.local.js (gitignored)
   module.exports = {
     googleMapsApiKey: process.env.GOOGLE_MAPS_API_KEY
   };
   ```

3. **Use WordPress-specific methods:**

   ```php
   // In wp-config.php (never committed)
   define('CLASSIFAI_GOOGLE_API_KEY', 'your-key-here');
   
   // In plugin code
   $api_key = defined('CLASSIFAI_GOOGLE_API_KEY') ? CLASSIFAI_GOOGLE_API_KEY : '';
   ```

### For Configuration Examples

When creating example configurations:

- Use placeholder values like `your-api-key-here` or `XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX`
- Name files with `.example` or `.template` extensions (e.g., `config.example.js`)
- These files are automatically excluded from the Google API key pattern check

## WordPress Plugin Development

### Building the Plugin

```bash
npm install
npm run build
```

The webpack configuration compiles JavaScript and TypeScript files for the plugin's various features including:

- Admin interface
- Content resizing
- Image generation
- Text-to-speech
- Classification features
- And more

### Plugin Features

The ClassifAI plugin provides AI-powered features including:

- **Content Generation** - AI-assisted content writing
- **Image Analysis** - Automatic image classification and alt text
- **Text-to-Speech** - Audio generation from content
- **Content Resizing** - Automatic summarization and expansion
- **Recommended Content** - AI-powered content recommendations
- **OCR** - Text extraction from images
- **Moderation** - Content moderation using AI

## Testing

The `tests/test-plugin/` directory contains mock API responses used for testing:

- `image_analyze.json` - Mock image analysis responses
- `models.json` - Available AI model configurations
- `moderation.json` - Content moderation responses
- `nlu.json` - Natural language understanding responses
- `ollama-*.json` - Ollama AI model responses
- `resize-content*.json` - Content resizing responses
- `text-to-speech-voices.json` - Available TTS voices
- `whisper.json` - Speech-to-text responses
- `pdf.json` - PDF processing responses

## Contributing

### Security Contributions

If you're improving security features:

1. Test pre-commit hooks thoroughly with both positive and negative cases
2. Document any new pattern blocks in the commit message
3. Update this README if adding new security measures
4. Never commit real secrets, even for testing purposes

### Code Contributions

1. Ensure pre-commit hooks are installed and passing
2. Follow WordPress coding standards
3. Add tests for new features
4. Update documentation as needed

## Support

For security issues, email: **office@nexusct.com**

For general plugin support, refer to the main ClassifAI plugin repository.

## License

This repository is maintained by Nexus Communications Technology (Nexuscomm LLC).

## Acknowledgments

- Security tools: [gitleaks](https://github.com/gitleaks/gitleaks), [detect-secrets](https://github.com/Yelp/detect-secrets), [pre-commit](https://pre-commit.com/)
- Incident response best practices from NIST and OWASP
- WordPress security guidelines from the WordPress Security Team
