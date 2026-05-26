# Security Policy

## Supported Versions

AIyeah is in early development. Security patches will be applied to the latest release on the `dev` branch.

| Version | Supported          |
| ------- | ------------------ |
| latest  | :white_check_mark: |

## Reporting a Vulnerability

**Do not open a public issue for security vulnerabilities.**

Please send vulnerability reports to **`faisallionel@gmail.com`** with:
- A description of the vulnerability
- Steps to reproduce
- Any potential impact or severity assessment

You will receive a response within **48 hours**. We will keep you updated as we investigate and resolve the issue.

## Disclosure

After a fix is released, we will:
- Publish a security advisory
- Credit the reporter (with consent)
- Document the CVE if applicable

## Security Design

AIyeah follows these security principles:
- **API keys encrypted at rest** (AES-256)
- **Bring-your-own-key (BYOK)** for provider credentials
- **Never log raw keys or secrets**
- **Row-level workspace isolation** in all data stores
- **Input validation** on all API endpoints
