# Authentication & Authorization Design

## Decision Summary
TaskBoard adopts **JWT with Redis session validation** for stateless API authentication while maintaining server-side logout capability and refresh token management.

## Background & Rationale
Traditional session-based authentication requires sticky sessions in distributed environments, complicating horizontal scaling. JWT provides stateless verification, enabling any server instance to validate tokens. However, pure stateless JWT prevents immediate logout without token expiration. We solve this with Redis blacklisting, creating a hybrid approach balancing scalability and security.

## JWT Implementation Details
- **Access Token**: 15-minute expiry, contains user ID and role claims
- **Refresh Token**: 7-day expiry, stored in HTTP-only cookies for automatic rotation
- **Signing Algorithm**: RS256 (asymmetric), public keys distributed to all API servers
- **Token Scope**: Minimal claims prevent payload bloat; roles fetched from database on service boundaries

## Alternatives Evaluated
1. **Pure Session Storage**: Requires Redis/Memcached, sticky load balancer routing, complex user tracking across services
2. **OAuth2 Password Flow**: Adds operational complexity for internal-first SaaS; external provider dependency
3. **Stateless JWT Only**: Cannot immediately revoke tokens; user logout requires waiting for expiry

## Redis Blacklist Strategy
- Logout adds token to Redis set with TTL matching token expiry
- Token validation queries Redis first; found tokens rejected immediately
- Minimal memory overhead: only revoked tokens stored, not valid ones
- Scales to millions of active tokens with commodity Redis instances

## Critical Considerations
- Clock skew between servers must be < 1 minute; use NTP synchronization
- Refresh token rotation on each use prevents token replay attacks
- HTTPS enforcement mandatory; tokens exposed otherwise
- API keys issued separately for machine-to-machine auth, never JWT
