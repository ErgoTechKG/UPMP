# Technical Assumptions

## Repository Structure: Monorepo

Utilize a monorepo structure to maintain code consistency, share common utilities, and simplify deployment pipelines. Organize with clear boundaries: /apps (frontend, backend services), /packages (shared libraries), /infrastructure (deployment configs).

## Service Architecture

Implement a microservices architecture within the monorepo to enable independent scaling and deployment:
- API Gateway (Kong/Traefik) for routing and rate limiting
- Auth Service: JWT-based authentication with refresh tokens
- User Service: Profile management and role assignments  
- Project Service: Research project lifecycle management
- Notification Service: WeChat Work integration and email fallback
- Analytics Service: Data aggregation and reporting
- File Service: Document storage with version control

## Testing Requirements

Implement comprehensive testing pyramid:
- Unit tests: 80% code coverage minimum for business logic
- Integration tests: API endpoint testing with mock external services
- E2E tests: Critical user journeys using Cypress/Playwright
- Performance tests: Load testing for concurrent user scenarios
- Security tests: OWASP top 10 vulnerability scanning
- Manual testing convenience: Seed data scripts and test account management

## Additional Technical Assumptions and Requests

- Use TypeScript throughout for type safety and better developer experience
- Implement Redis for session management and caching frequently accessed data
- Use PostgreSQL with proper indexing for complex queries and full-text search
- Deploy using Kubernetes for orchestration and scaling
- Implement OpenTelemetry for distributed tracing and monitoring
- Use Elasticsearch for advanced search functionality across projects and users
- Implement feature flags for gradual rollout and A/B testing capabilities
- Use message queuing (RabbitMQ/Kafka) for async operations like notifications
- Implement CDC (Change Data Capture) for audit logging without performance impact
- Frontend state management using Zustand or Redux Toolkit for predictable updates
- API versioning strategy using URL path versioning (/api/v1/, /api/v2/)
- Implement GraphQL for complex data fetching requirements in analytics dashboards
- Use Docker for consistent development and deployment environments
- Implement blue-green deployment strategy for zero-downtime updates
