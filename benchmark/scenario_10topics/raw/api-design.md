# API Design & REST Architecture

## Decision Summary
TaskBoard implements **REST with JSON payloads**, rejecting GraphQL for operational simplicity while enforcing strict pagination and rate limiting to prevent abuse and maintain predictable server load.

## REST Adoption Rationale
GraphQL's field-level query flexibility introduces complex authorization logic (users querying sensitive fields), performance unpredictability (client-determined fetch depth), and harder monitoring. REST's fixed endpoints, while slightly more verbose, provide client-friendly contracts and straightforward caching. Our API traffic is read-heavy with stable query patterns, not mutation-heavy.

## Endpoint Conventions
- Resources follow `/api/v1/{resource}/{id}/{sub-resource}` pattern
- HTTP verbs: GET (read), POST (create), PATCH (partial update), DELETE (logical delete)
- Request/response wrapped in top-level object: `{ "data": {...}, "meta": {...} }`
- Error responses: `{ "error": { "code": "INVALID_STATUS", "message": "...", "details": [...] } }`
- Timestamps ISO 8601; all dates in UTC

## Pagination & Rate Limiting
- **Offset-limit pagination**: `?offset=20&limit=50`, default limit 20, max 100
- **Keyset pagination** for high-offset queries preventing N² query complexity
- **Rate limits**: 1000 req/min per authenticated user, 100 req/min per IP for public endpoints
- **Burst allowance**: 50 req/10sec; prevents single concurrent spikes while allowing reasonable bursts

## Alternatives Evaluated
1. **GraphQL**: Field-level authorization complexity; N+1 query challenges; harder monitoring and quota enforcement
2. **Cursor-based pagination only**: Prevents reverse iteration and jumping; keyset variant balances performance needs
3. **Unlimited query complexity**: Enables DoS attacks; rate limiting alone insufficient for malformed queries

## API Versioning Strategy
- Version in URL path (v1, v2) rather than headers for cacheability
- Minimum 6-month migration period before deprecating old versions
- Feature flags for gradual rollout of breaking changes within same version

## Critical Considerations
- CORS: Explicitly whitelist origin domains; avoid wildcard in production
- Deprecation headers warn clients before endpoint sunset
- Request IDs (X-Request-ID) propagated through entire stack for tracing
- Webhook payloads include signature for HMAC verification, preventing spoofing
