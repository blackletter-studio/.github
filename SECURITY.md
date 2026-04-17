# Security Policy

## Reporting a vulnerability

Email **security@blackletter.studio** with details. PGP-signed email preferred; public key at https://blackletter.studio/.well-known/security.asc (published after M5).

**Scope:** any Black Letter product (Fair Copy, Proofmark, marketing site, feedback Worker).

**Response commitment:** acknowledgement within 3 business days, coordinated disclosure within 90 days. Reporters credited in release notes with consent.

**No bug bounty at launch.** A hall-of-fame page is maintained at https://blackletter.studio/trust#disclosure-credits.

## What we consider in scope

- Unauthorized access to license data
- Document content exfiltration paths (there should be none; prove us wrong)
- XSS / injection in task pane or feedback form
- Supply-chain compromise vectors in the dictionary repo

## What is not in scope

- Social engineering of maintainers
- Denial of service via sending extremely large files (Word handles these; we inherit its limits)
- Issues in Microsoft Office itself (report to Microsoft MSRC)
