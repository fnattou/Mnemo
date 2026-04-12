# Team Conventions & Development Workflows

## Decision Summary
TaskBoard enforces **GitHub Flow with required code reviews, conventional commits, and automated linting** ensuring code quality and institutional knowledge capture without bureaucratic ceremony.

## Code Review Standards
- **Mandatory reviewers**: 2 approvals required (1 from backend/frontend lead, 1 from any team member)
- **Review checklist**: logic correctness, test coverage, security (no secrets leaked), performance (no N+1), documentation
- **Approval process**: Discussions resolved in PR comments, not Slack; async communication enables timezone distribution
- **Dismissal authority**: PR author can dismiss stale approvals after addressing feedback; re-review triggered by force-pushing new commits
- **Review SLA**: 24-hour response time maximum; blocking PRs escalated if reviewers unavailable

## GitHub Flow Branch Strategy
- **Feature branches**: Fork from main, named `feature/task-board-ui` or `bugfix/duplicate-delete`
- **Naming convention**: Lowercase, hyphens separate words, reference issue number for traceability
- **Branch protection**: main protected, requires status checks passing (tests, linting) and reviews before merge
- **Squash merges**: Combine commits before merge, maintaining clean history; amended commits forbidden to preserve audit trail
- **Automatic cleanup**: Delete branches immediately post-merge, preventing stale references

## Commit Message Convention (Conventional Commits)
Format: `<type>(<scope>): <description>`

Types: feat, fix, docs, style, refactor, perf, test, chore
```
feat(tasks): add bulk-edit workflow
Implement multi-select task UI enabling simultaneous status/assignee updates.
Closes #234

fix(auth): prevent refresh token reuse
Invalidate token on refresh, forcing new token generation preventing replay attacks.
```

Benefits:
- Automated changelog generation from commit prefixes
- Searchable history (git log --grep="feat")
- Enforced consistency via commit-msg hook

## Linting & Formatting
- **ESLint**: Catches logic errors (unused variables, incorrect null checks), security issues (eval usage)
- **Prettier**: Opinionated code formatting (semicolons, quotes, line width 80) preventing style debates
- **Pre-commit hooks**: Lint and format run automatically before commit; failure blocks commit until fixed
- **CI verification**: Linting rules re-run in CI; discrepancies (e.g., manual overrides) caught before merge

## Architecture Decision Records (ADRs)
- Major decisions documented in design documents (auth.md, db-schema.md, etc.) at project root
- Format: Problem statement, proposed solution, alternatives considered, consequences
- Review requirement: Technical leads approve before implementation
- Living documents: Updated when decisions evolve; stale ADRs marked "superseded"

## Release & Versioning
- **Semantic versioning**: MAJOR.MINOR.PATCH (1.2.3)
- **Version bumping**: Automated from conventional commits; breaking changes (feat! syntax) bump MAJOR
- **Release process**: GitHub release tag created with auto-generated changelog, sent to #announcements
- **Release cadence**: Weekly releases; hotfixes bypass queue for critical bugs

## Documentation Standards
- **README**: Project overview, quick-start setup, architecture diagram, development workflow
- **Code comments**: Explain "why", not "what"; code should be self-documenting
- **API docs**: OpenAPI/Swagger specs auto-generated from JSDoc comments, served at /api/docs
- **Runbooks**: Incident response procedures (database failover, service restart) linked in README

## Onboarding Checklist
- [ ] Clone repo, run `npm install && npm run dev`
- [ ] Create feature branch off main
- [ ] Read auth.md, db-schema.md, api-design.md for architecture context
- [ ] Make first contribution (fix typo, add test) to validate setup
- [ ] Pair with team member on first PR review cycle

## Critical Enforcement Mechanisms
- Pre-commit hooks prevent committing with console.log, debugger statements
- Linting enforced by branch protection rules; merge blocked until all status checks pass
- Force-push to main forbidden; accidental commits require explicit revert commit
