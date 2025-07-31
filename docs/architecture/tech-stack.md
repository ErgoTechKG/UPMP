# Tech Stack

## Technology Stack Table

| Category | Technology | Version | Purpose | Rationale |
|----------|------------|---------|---------|-----------|
| Frontend Language | TypeScript | 5.3+ | Type-safe development | Catches errors early, improves IDE support |
| Frontend Framework | React | 18.2+ | UI library | Large ecosystem, excellent China community |
| UI Component Library | Ant Design | 5.12+ | Pre-built components | Designed for Chinese users, enterprise-ready |
| State Management | Zustand + React Query | 4.5+ / 5.x | Client and server state | Lightweight, excellent DX, built-in caching |
| Backend Language | TypeScript | 5.3+ | Type-safe backend | Shared types with frontend, better refactoring |
| Backend Framework | NestJS | 10.x | Enterprise Node.js framework | Modular architecture, built-in microservices support |
| API Style | REST + GraphQL | - | Mixed approach | REST for simple CRUD, GraphQL for complex queries |
| Database | PostgreSQL | 15+ | Primary data store | ACID compliance, JSON support, proven scale |
| Cache | Redis | 7.2+ | Session & data cache | Fast, supports pub/sub for real-time features |
| File Storage | Alibaba OSS | - | Object storage | China-optimized, integrated CDN |
| Authentication | Custom JWT + WeChat Work | - | Multi-method auth | Supports required auth methods per PRD |
| Frontend Testing | Vitest + React Testing Library | 1.x / 14.x | Unit/integration tests | Fast, ESM support, good React integration |
| Backend Testing | Jest | 29.x | Unit/integration tests | Mature, NestJS integration |
| E2E Testing | Playwright | 1.40+ | End-to-end tests | Cross-browser, reliable, good debugging |
| Build Tool | Nx | 17.x | Monorepo management | Excellent caching, affected commands |
| Bundler | Vite | 5.x | Frontend bundling | Fast HMR, optimized builds |
| IaC Tool | Terraform | 1.6+ | Infrastructure as Code | Multi-cloud support, Alibaba Cloud provider |
| CI/CD | GitHub Actions + Alibaba Cloud Deploy | - | Automation | Good integration, supports monorepo |
| Monitoring | Alibaba Cloud ARMS | - | APM and monitoring | Native integration, good for China |
| Logging | Alibaba Cloud SLS | - | Centralized logging | Elasticsearch-compatible, China-compliant |
| CSS Framework | Tailwind CSS | 3.4+ | Utility CSS | Rapid styling, small bundle with purging |
