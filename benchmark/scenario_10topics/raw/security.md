# Security Architecture & Threat Mitigation

## Decision Summary
TaskBoard implements **defense-in-depth security**: input validation at API boundaries, CSRF tokens for state-changing operations, password policies enforced via bcrypt with 12-round salt, and automated dependency auditing in CI/CD.

## Input Validation Strategy
- **Server-side mandatory**: Client-side validation improves UX, never trusted for security
- **Whitelist validation**: Reject inputs not matching expected format (task title max 255 chars, ASCII only)
- **Type coercion prevention**: JSON schema validation rejects type mismatches; "5" !== 5
- **SQL injection prevention**: Parameterized queries non-negotiable; ORM query builders default-safe
- **XSS prevention**: HTML entities escaped in templates; Content-Security-Policy headers restrict script execution origins

## CSRF Protection
- **SameSite cookies**: Set to Strict for state-changing operations, preventing cross-site requests
- **CSRF tokens**: Issued per session, validated on POST/PATCH/DELETE requests
- **Double-submit pattern**: Token stored in cookie and request body; mismatch detected server-side
- **Exemption rationale**: GET requests read-only; state-modifying operations always token-protected

## Password Management
- **bcrypt with cost 12**: ~100ms hashing time prevents brute-force attacks (rate-limiting secondary control)
- **Minimum 12 characters**: Entropy >= 50 bits; dictionary attacks unfeasible
- **No complexity requirements**: "correcthorsebatterystaple" (26 chars, all lowercase) stronger than "P@ss123!" (8 mixed)
- **Breach detection**: Passwords checked against Pwned Passwords API during signup; rejected if found

## Dependency Management
- **npm audit** runs on every commit; fails build if high-severity vulnerabilities detected
- **Dependabot** auto-creates PRs for security updates; reviewed and merged within 48 hours
- **Lockfiles** version-controlled, preventing supply chain attacks via dependency resolution changes
- **Third-party packages** use scoped imports; malicious package substitution (left-pad attack) prevented via @org/package

## Infrastructure Security
- **HTTPS enforcement**: HSTS header forces TLS for 1 year; downgrade attacks prevented
- **Database encryption**: RDS encryption at rest, TLS for in-transit traffic
- **Secrets rotation**: API keys, database credentials rotated quarterly via Secrets Manager
- **Network segmentation**: Private RDS subnet isolated from public-facing ECS services via NAT

## Alternatives Considered
1. **Session-based CSRF**: Fewer tokens to manage; cookies simpler but vulnerable to cross-site set-cookie attacks
2. **OAuth for password**: Outsourced security; complexity for single-identity SaaS
3. **Bespoke encryption**: Homegrown crypto introduces backdoors; standard algorithms (AES-256) proven superior

## Threat Modeling
- **DDoS**: CloudFlare WAF rate-limiting blocks volumetric attacks; application-layer DoS mitigated by request quota
- **Account takeover**: 2FA (TOTP) optional for all users; suspicious login from new geography triggers verification email
- **Data exfiltration**: Database backups encrypted; audit logs capture all data access; data retention policies delete old records

## Critical Non-Negotiables
- No logging of passwords or tokens in any form (request bodies, error messages, debug logs)
- PII (email, phone) accessed via RBAC; audit trails mandatory
- Security headers: CSP, X-Frame-Options, X-Content-Type-Options, X-XSS-Protection
