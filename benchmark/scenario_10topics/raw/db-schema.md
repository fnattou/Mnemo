# Database Schema & PostgreSQL Design

## Decision Summary
TaskBoard uses **PostgreSQL with JSONB columns for flexible metadata storage**, rejecting both NoSQL and sharded architectures in favor of strong ACID guarantees and proven operational simplicity at our scale (< 10M records).

## PostgreSQL Selection Rationale
PostgreSQL's JSONB support eliminates the false choice between strict schemas and flexibility. Unlike MySQL, PostgreSQL provides native full-text search, array types, and recursive queries essential for task hierarchies. Strong consistency prevents subtle data bugs common in distributed systems; our traffic profile (mostly reads) doesn't warrant the complexity costs.

## Core Schema Architecture
- **users**: id, email (unique), password_hash, role, settings (JSONB), created_at, updated_at
- **projects**: id, workspace_id (FK), name, description, owner_id, settings (JSONB)
- **tasks**: id, project_id (FK), title, description, status, priority, assignee_id (FK), due_date, metadata (JSONB), created_at, updated_at
- **task_comments**: id, task_id (FK), user_id (FK), content, created_at, updated_at
- **audit_log**: id, user_id, entity_type, entity_id, action, changes (JSONB), timestamp

## Indexing Strategy
- Composite index (project_id, status, due_date) for common filter queries
- BTREE on timestamps for time-range queries
- GIN index on metadata JSONB for rapid filtering by custom fields
- Partial index on tasks WHERE status != 'archived' reducing index size 40%

## Alternatives Considered
1. **MongoDB**: Lacks transaction support; ACID constraints discovered post-production cause costly migrations
2. **Sharding by workspace_id**: Operations burden unmotivated until 100M+ records; premature optimization
3. **Immutable log storage**: Complicates updates; task description changes require archiving and rewriting

## Migration & Versioning
- Schema migrations using Flyway, versioned in Git
- Blue-green deployment for zero-downtime schema changes
- Backward compatibility: new columns added nullable, deprecated via deprecation warnings

## Critical Notes
- Connection pooling (PgBouncer) mandatory; default connection limits insufficient for high-concurrency microservices
- WAL archiving enabled for point-in-time recovery; tested monthly
- Regular VACUUM jobs prevent table bloat from UPDATE-heavy workloads
