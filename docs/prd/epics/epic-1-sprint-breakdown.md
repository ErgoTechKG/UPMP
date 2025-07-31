# Epic 1 Sprint Breakdown - SRMP

## Sprint Overview (5 Sprints Total)

### Sprint 1: Foundation & External Dependencies (10 points)
**Theme:** Set up project structure and resolve external blockers

#### Stories:
1. **Story 1.0: Initialize Nx Monorepo Structure** (3 points)
   - Day 1-2: Nx setup and configuration
   - Day 3: Create shared packages
   - Day 4-5: Verify builds and tests

2. **Story 1.1: Initialize Project Infrastructure** (2 points)
   - Day 6: Git setup, branching strategy
   - Day 7: PR templates, protection rules

3. **Story 1.2.5: Configure External Services** (3 points) - PARALLEL
   - Day 1-3: WeChat Work registration (start immediately)
   - Day 4-5: Create service mocks
   - Day 8-9: Credential management setup

4. **Story 1.9: Create Development Documentation** (2 points) - START
   - Day 8-10: Initial README and setup guide
   - Continue updating throughout epic

**Sprint Goal:** Development environment ready with external service mocks

### Sprint 2: Data & Infrastructure (13 points)
**Theme:** Database, infrastructure services, and supporting systems

#### Stories:
1. **Story 1.2: Implement Complete Database Schema** (5 points)
   - Day 1-3: Design all entities and relationships
   - Day 4: Create migrations
   - Day 5: Seed data scripts

2. **Story 1.7: Set up Infrastructure Services** (5 points)
   - Day 6-7: Redis deployment and configuration
   - Day 8: Elasticsearch setup
   - Day 9: RocketMQ configuration
   - Day 10: OSS and monitoring setup

3. **Story 1.8: CI/CD Pipeline Foundation** (3 points) - START
   - Day 8-10: GitHub Actions basic setup
   - Complete in Sprint 3

**Sprint Goal:** All data and infrastructure services operational

### Sprint 3: Authentication & Authorization (11 points)
**Theme:** Complete security implementation

#### Stories:
1. **Story 1.3: Build Multi-Method Authentication** (8 points)
   - Day 1-2: JWT infrastructure
   - Day 3-4: Email/password auth
   - Day 5-6: Student ID auth
   - Day 7-8: WeChat Work OAuth
   - Day 9-10: Account linking

2. **Story 1.8: CI/CD Pipeline Foundation** (2 points) - COMPLETE
   - Day 9-10: Testing and deployment automation

3. **Story 1.9: Documentation** (1 point) - CONTINUE
   - Update with auth implementation details

**Sprint Goal:** Users can register and login via multiple methods

### Sprint 4: RBAC & User Management (8 points)
**Theme:** Authorization and user features

#### Stories:
1. **Story 1.4: Implement RBAC** (5 points)
   - Day 1-2: Role and permission models
   - Day 3-4: Middleware implementation
   - Day 5: Admin interface

2. **Story 1.5: Create User Profile Management** (3 points)
   - Day 6-7: Profile APIs
   - Day 8: Photo upload
   - Day 9-10: Privacy settings

**Sprint Goal:** Complete user management with role-based access

### Sprint 5: UI Shell & Finalization (10 points)
**Theme:** Frontend integration and production readiness

#### Stories:
1. **Story 1.6: Develop Initial UI Shell** (5 points)
   - Day 1-2: React app setup in monorepo
   - Day 3-4: Navigation and routing
   - Day 5: Role-based UI

2. **Story 1.8: CI/CD Pipeline** (2 points) - FINALIZE
   - Day 6-7: Production deployment setup

3. **Story 1.9: Documentation** (3 points) - COMPLETE
   - Day 8-10: Comprehensive docs, ADRs, onboarding

**Sprint Goal:** Complete Epic 1 with working authenticated UI

## Resource Allocation

### Team Composition:
- **Backend Developers:** 2 (Focus on services and APIs)
- **Frontend Developer:** 1 (Joins in Sprint 4)
- **DevOps Engineer:** 1 (Throughout, heavier in Sprints 1-2)
- **QA Engineer:** 1 (Joins in Sprint 2)

### Parallel Work Streams:
- **Sprint 1:** External services + Monorepo setup (parallel)
- **Sprint 2:** Database + Infrastructure (some parallel)
- **Sprint 3:** Auth implementation (mostly sequential)
- **Sprint 4:** RBAC + Profiles (can parallelize)
- **Sprint 5:** UI + Polish (parallel with docs)

## Risk Mitigation Timeline

| Risk | Sprint 1 | Sprint 2 | Sprint 3 | Sprint 4 | Sprint 5 |
|------|----------|----------|----------|----------|----------|
| WeChat Work | Start registration | Complete registration | Implement OAuth | Test integration | Production ready |
| External Services | Create all mocks | Test mocks | Integrate real services | Fallback testing | Monitor health |
| Database Schema | Design complete schema | Implement & test | - | - | Migration docs |
| Infrastructure | Plan services | Deploy all | Configure | Optimize | Monitor |
| Documentation | Start immediately | Update continuously | Auth docs | RBAC docs | Complete |

## Definition of Done per Sprint

### Sprint 1 DoD:
- [ ] Nx monorepo builds successfully
- [ ] All external service mocks working
- [ ] Development setup < 30 minutes
- [ ] WeChat Work registration submitted

### Sprint 2 DoD:
- [ ] All database tables created
- [ ] Redis, Elasticsearch, MQ operational
- [ ] Basic CI/CD running
- [ ] 80% test coverage target met

### Sprint 3 DoD:
- [ ] Users can register/login 3 ways
- [ ] JWT tokens properly validated
- [ ] WeChat Work OAuth working
- [ ] Auth errors in both languages

### Sprint 4 DoD:
- [ ] Role-based access enforced
- [ ] User profiles fully functional
- [ ] Permission system tested
- [ ] API documentation complete

### Sprint 5 DoD:
- [ ] UI shows role-appropriate content
- [ ] Staging deployment working
- [ ] Load testing completed
- [ ] Onboarding guide tested