# Deployment Architecture & CI/CD Pipeline

## Decision Summary
TaskBoard deploys **Node.js services on AWS ECS (Fargate) orchestrated by GitHub Actions**, providing managed container orchestration without Kubernetes's operational overhead while maintaining auto-scaling and zero-downtime deployments.

## AWS ECS Fargate Selection
Fargate eliminates EC2 node management: no SSH access, no AMI maintenance, no security patches to coordinate. Costs scale with container usage, not provisioned capacity. Service definitions define desired task count and CPU/memory; CloudFormation infrastructure-as-code enables repeatable deployments. For our 3-service architecture (auth, task-api, notification-worker), Fargate's per-task billing is more economical than EC2 clusters.

## GitHub Actions CI/CD Workflow
1. **Commit**: Developer pushes to feature branch
2. **Build**: Docker image built, tested, pushed to ECR with Git SHA tag
3. **Test**: Integration tests run against RDS staging database
4. **Security**: Trivy scans image for CVEs; build fails if critical vulnerabilities detected
5. **Deploy**: Blue-green deployment swaps load balancer target group, rollback via previous image tag if health checks fail
6. **Smoke tests**: POST-deployment tests verify critical workflows in production

## Environment Management
- **dev**: Per-developer environments, auto-deleted after 7 days unused
- **staging**: Mirrors production configuration; all PRs deployed here before merge
- **production**: Multi-AZ deployment with read replicas; gradual traffic shift (10% → 50% → 100%)

## Deployment Safety Mechanisms
- **Circuit breaker**: Failed deployments automatically roll back after 10 consecutive 5xx errors
- **Canary releases**: New service version receives 5% traffic for 5 minutes before full rollout
- **Database backward compatibility**: Schema changes deployable without code changes
- **Feature flags**: Critical features toggleable via environment variables without redeployment

## Alternatives Considered
1. **Self-managed Kubernetes**: Superior for complex multi-region setups; overhead unjustified for single-region deployment
2. **Heroku**: Simpler developer experience; prohibitive costs at scale and vendor lock-in
3. **Manual deployments**: Human error risk; inconsistent environments; no audit trail

## Infrastructure as Code
- CloudFormation templates version controlled, reviewed in PRs
- RDS backups retained 30 days; monthly restore test validates recovery procedures
- Secrets stored in AWS Secrets Manager, rotated quarterly
- VPC isolation: private subnets for RDS/ElastiCache, NAT gateway for outbound internet

## Critical Operational Practices
- Deployment window coordination: Friday deployments forbidden, only 9am-3pm window
- Runbooks documented for incident response (database failover, service restart, capacity scaling)
- Canary monitoring: Error rates, latency percentiles compared against previous deployment
