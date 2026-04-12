# Performance Optimization & Caching Strategy

## Decision Summary
TaskBoard targets **<200ms p95 latency** through Redis caching, eliminating N+1 database queries, and serving static assets via CloudFront CDN while carefully avoiding premature optimization in non-critical paths.

## Redis Caching Architecture
- **User data cache**: 5-minute TTL for user roles and permissions, reducing database queries 90%
- **Project metadata cache**: 15-minute TTL for project names and settings; invalidated on update
- **Task list pagination**: Cursor-based queries benefit from database indexes more than caching; Redis used only for filters
- **Cache-aside pattern**: Application checks Redis first, queries database on miss, populates cache (vs write-through complexity)

Cache invalidation on write:
```
UpdateTask(id, data) {
  query.update('tasks', data).where({id})
  redis.delete(`task:${id}`)  // Immediate invalidation
  redis.delete(`project:${task.project_id}:tasks`)  // Dependent cache
}
```

## N+1 Query Elimination
Common mistake:
```javascript
const tasks = db.select('*').from('tasks').where({project_id})
// N queries:
tasks.forEach(t => t.assignee = db.select('*').from('users').where({id: t.assignee_id}))
```

Solution via eager loading:
```javascript
const tasks = db.select('tasks.*', 'users.name as assignee_name')
  .from('tasks')
  .leftJoin('users', 'users.id', 'tasks.assignee_id')
  .where({project_id})
```

Database query analyzers (EXPLAIN ANALYZE) run on PRs, failing if query complexity exceeds threshold.

## CDN Strategy with CloudFront
- **Origin**: S3 bucket for static assets (JS, CSS, images), ECS load balancer for dynamic content
- **Cache headers**: index.html (Cache-Control: no-cache, must-revalidate), assets (Cache-Control: max-age=31536000 with content hash)
- **Versioned assets**: Webpack builds include content hash in filenames (bundle.abc123.js); hash change = new CDN cache entry
- **Purge invalidation**: On deployment, explicitly purge old file patterns preventing stale assets

## Page Load Performance Targets
- **Time to First Byte (TTFB)**: <100ms via edge caching and regional distribution
- **First Contentful Paint (FCP)**: <1.5s via critical CSS inlining, minimal JavaScript
- **Largest Contentful Paint (LCP)**: <2.5s via lazy-loaded images and deferred non-critical scripts
- **Cumulative Layout Shift (CLS)**: <0.1 via fixed placeholder dimensions and font-display: swap

## Alternatives Considered
1. **Server-side rendering (SSR)**: Improves TTFB but complicates deployment; client-side rendering + CDN sufficient for SPA
2. **Heavy caching (30min TTL)**: Stale data frustrates users; 5min TTL balances freshness and efficiency
3. **Infinite scrolling**: Pagination lighter; cursor-based keyset pagination scales to millions

## Performance Monitoring
- Real User Monitoring (RUM) via analytics library collecting Core Web Vitals from browsers
- Synthetic monitoring via k6 scripted scenarios detecting regressions before user impact
- Budget alerts: deployment fails if bundle size increases >15%, preventing creep

## Critical Practices
- Premature optimization avoided: optimize only demonstrable bottlenecks identified via profiling
- Database connection pooling (min 5, max 20) prevents pool exhaustion under load
- Microservice timeout policies prevent cascading failures (task-api calls auth-service with 2s timeout)
