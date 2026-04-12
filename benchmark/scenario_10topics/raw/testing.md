# Testing Strategy & Quality Assurance

## Decision Summary
TaskBoard targets **70% unit test, 20% integration, 10% e2e coverage**, with test pyramid prioritizing fast feedback while maintaining critical path validation through integration and browser automation.

## Test Distribution Rationale
Unit tests catch logic errors instantly (< 100ms suite), enabling fearless refactoring. Integration tests verify service boundaries and database interactions, catching contract mismatches earlier than production. E2E tests validate complete workflows but are 10x slower; reserved for critical user journeys (login, create task, share project).

## Coverage Targets by Layer
- **Backend controllers**: 80% coverage; error paths and edge cases mandatory
- **Business logic**: 85% coverage; core algorithms require exhaustive testing
- **Database queries**: 70% coverage; integration tests catch N+1 and query errors
- **Frontend components**: 60% coverage; Storybook stories supplement unit tests for visual validation
- **Integration flows**: 50% coverage; critical workflows only (signup, payment, exports)

## Test Data Management
- Factory functions generate realistic test fixtures (faker.js for seeded randomness)
- Isolated database per test using transaction rollback; 200ms teardown per test
- Fixtures committed to Git, versioned alongside migration changes
- Seed data never includes production credentials or PII

## Testing Technologies
- **Backend**: Jest with supertest for HTTP assertions, testcontainers for database isolation
- **Frontend**: Vitest + React Testing Library, avoiding Enzyme's implementation-detail testing
- **E2E**: Playwright (not Selenium) for reliability and modern async support
- **Load testing**: k6 for sustained load profiles (1000+ concurrent users)

## Alternatives Evaluated
1. **High E2E coverage (50%)**: Brittleness and slow feedback loops outweigh coverage benefits
2. **Snapshot testing everywhere**: Masks regressions; reserved for CSS-in-JS and visual components only
3. **Manual testing for integration**: Human error and non-determinism; automation essential

## CI/CD Integration
- Tests run on every commit, blocking merge on failure
- Slow tests (>5s) parallelized across workers, maintaining < 5min total runtime
- Code coverage reports in pull requests; declining coverage rejects PR
- Flaky test detection: tests run 5 times, failure recorded for maintenance

## Critical Practices
- Test names describe expected behavior, not implementation ("should prevent duplicate task titles")
- Arrange-act-assert structure; setup separated from test logic
- Mocking external services (email, payment) prevents integration test brittleness
- Test database uses production schema, catching migration issues
