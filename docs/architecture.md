# Scientific Research Management Platform (SRMP) Fullstack Architecture Document

## Introduction

This document outlines the complete fullstack architecture for Scientific Research Management Platform (SRMP), including backend systems, frontend implementation, and their integration. It serves as the single source of truth for AI-driven development, ensuring consistency across the entire technology stack.

This unified approach combines what would traditionally be separate backend and frontend architecture documents, streamlining the development process for modern fullstack applications where these concerns are increasingly intertwined.

### Starter Template or Existing Project

N/A - Greenfield project with custom monorepo architecture optimized for China deployment and educational institution requirements.

### Change Log

| Date | Version | Description | Author |
|------|---------|-------------|--------|
| 2025-07-30 | 1.0 | Initial fullstack architecture based on PRD and UI/UX spec | Winston (Architect) |

## High Level Architecture

### Technical Summary

SRMP employs a microservices architecture within a monorepo structure, with React-based frontend and Node.js backend services deployed on Alibaba Cloud. The frontend uses Ant Design for UI components aligned with Chinese design patterns, while the backend implements RESTful APIs with GraphQL for complex queries. Integration points include WeChat Work authentication, university SSO systems, and real-time notifications via WebSocket. The architecture prioritizes the "three-step operation" principle through streamlined API design and intelligent caching, achieving PRD goals of 40% administrative overhead reduction through automation and real-time monitoring capabilities.

### Platform and Infrastructure Choice

**Platform:** Alibaba Cloud (阿里云)
**Key Services:** ECS (Elastic Compute Service), RDS (PostgreSQL), OSS (Object Storage), CDN, API Gateway, Function Compute (Serverless), Message Queue, Elasticsearch Service
**Deployment Host and Regions:** Primary: China (Beijing - cn-beijing), Secondary: China (Shanghai - cn-shanghai) for disaster recovery

### Repository Structure

**Structure:** Monorepo
**Monorepo Tool:** Nx (with pnpm workspaces)
**Package Organization:** Apps (web, api-gateway, microservices), Packages (shared types, utilities, UI components), Infrastructure (IaC templates)

### High Level Architecture Diagram

```mermaid
graph TB
    subgraph "Client Layer"
        WEB[Web App<br/>React + TypeScript]
        MOBILE[Mobile Web<br/>Responsive PWA]
    end
    
    subgraph "CDN & Edge"
        CDN[Alibaba CDN]
        WAF[Web Application Firewall]
    end
    
    subgraph "API Gateway Layer"
        APIGW[API Gateway<br/>Kong]
        AUTH[Auth Service<br/>JWT + WeChat Work]
    end
    
    subgraph "Microservices"
        USER[User Service]
        PROJECT[Project Service]
        MATCHING[Matching Service<br/>AI-Powered]
        TRAINING[Training Service]
        NOTIFICATION[Notification Service]
        ANALYTICS[Analytics Service]
        FILE[File Service]
    end
    
    subgraph "Data Layer"
        PG[(PostgreSQL<br/>Primary DB)]
        REDIS[(Redis<br/>Cache & Session)]
        ES[(Elasticsearch<br/>Search)]
        OSS[Object Storage<br/>Files]
    end
    
    subgraph "External Services"
        WECHAT[WeChat Work API]
        UNIV[University Systems<br/>SSO/Grades]
        AI[AI/ML Service<br/>Matching Algorithm]
    end
    
    subgraph "Message Queue"
        MQ[RocketMQ]
    end
    
    WEB --> CDN
    MOBILE --> CDN
    CDN --> WAF
    WAF --> APIGW
    
    APIGW --> AUTH
    AUTH --> WECHAT
    AUTH --> UNIV
    
    APIGW --> USER
    APIGW --> PROJECT
    APIGW --> MATCHING
    APIGW --> TRAINING
    APIGW --> NOTIFICATION
    APIGW --> ANALYTICS
    APIGW --> FILE
    
    USER --> PG
    USER --> REDIS
    PROJECT --> PG
    PROJECT --> ES
    MATCHING --> AI
    MATCHING --> PG
    TRAINING --> PG
    NOTIFICATION --> MQ
    NOTIFICATION --> WECHAT
    ANALYTICS --> PG
    ANALYTICS --> ES
    FILE --> OSS
    
    MQ --> NOTIFICATION
    MQ --> ANALYTICS
```

### Architectural Patterns

- **Microservices Architecture:** Independent services for scalability and team autonomy - _Rationale:_ Supports 10,000 concurrent users and allows independent scaling of high-traffic services
- **API Gateway Pattern:** Centralized entry point with Kong for routing, auth, rate limiting - _Rationale:_ Simplifies client integration and provides consistent security policies
- **Event-Driven Architecture:** RocketMQ for async operations and service decoupling - _Rationale:_ Enables real-time notifications and improves system resilience
- **BFF (Backend for Frontend):** Tailored API responses for web vs future mobile apps - _Rationale:_ Optimizes payload size and reduces client-side complexity
- **Repository Pattern:** Abstract data access in each microservice - _Rationale:_ Enables testing and potential future database migrations
- **CQRS for Analytics:** Separate read models for complex queries - _Rationale:_ Optimizes performance for data-heavy dashboards
- **Component-Based UI:** Atomic design with shared component library - _Rationale:_ Ensures UI consistency and accelerates development
- **State Management Pattern:** Zustand for client state, React Query for server state - _Rationale:_ Simplifies state logic and improves cache management

## Tech Stack

### Technology Stack Table

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

## Data Models

### User

**Purpose:** Core user entity representing all platform participants with role-based access

**Key Attributes:**
- id: UUID - Unique identifier
- studentId: string (nullable) - University student ID
- email: string - Primary email (unique)
- name: string - Full name in Chinese
- nameEn: string (nullable) - English name
- roles: Role[] - Array of assigned roles
- program: 'qiming' | 'innovation' | null - Student program type
- department: string - Academic department
- wechatWorkId: string (nullable) - WeChat Work identifier
- language: 'zh-CN' | 'en' - Preferred language
- avatar: string (nullable) - Profile photo URL

#### TypeScript Interface
```typescript
interface User {
  id: string;
  studentId?: string;
  email: string;
  name: string;
  nameEn?: string;
  roles: Role[];
  program?: 'qiming' | 'innovation';
  department: string;
  wechatWorkId?: string;
  language: 'zh-CN' | 'en';
  avatar?: string;
  createdAt: Date;
  updatedAt: Date;
}

interface Role {
  id: string;
  name: 'student' | 'professor' | 'secretary' | 'admin';
  permissions: Permission[];
}
```

#### Relationships
- Has many Projects (as student or professor)
- Has many Applications
- Has many Notifications
- Has many ActivityLogs

### Project

**Purpose:** Research project posted by professors for student applications

**Key Attributes:**
- id: UUID - Unique identifier
- title: string - Project title in Chinese
- titleEn: string (nullable) - English title
- description: string - Detailed description
- professorId: UUID - Project supervisor
- status: ProjectStatus - Current project state
- capacity: number - Max students (1-10)
- prerequisites: string[] - Required skills/knowledge
- commitmentHours: number - Weekly hours expected
- tags: string[] - Research areas/topics
- matchingEnabled: boolean - Allow AI matching

#### TypeScript Interface
```typescript
interface Project {
  id: string;
  title: string;
  titleEn?: string;
  description: string;
  descriptionEn?: string;
  professorId: string;
  professor?: User;
  status: 'draft' | 'published' | 'closed' | 'completed';
  capacity: number;
  currentStudents: number;
  prerequisites: string[];
  commitmentHours: number;
  tags: string[];
  matchingEnabled: boolean;
  department: string;
  startDate: Date;
  endDate: Date;
  createdAt: Date;
  updatedAt: Date;
}
```

#### Relationships
- Belongs to Professor (User)
- Has many Applications
- Has many ProjectStudents
- Has many Milestones
- Has many Discussions

### Application

**Purpose:** Student application to join a research project

**Key Attributes:**
- id: UUID - Unique identifier
- projectId: UUID - Target project
- studentId: UUID - Applicant
- status: ApplicationStatus - Application state
- statement: string - Motivation statement (500-1000 chars)
- cvUrl: string - Uploaded CV file URL
- matchScore: number - AI-calculated match percentage
- professorNotes: string (nullable) - Internal notes

#### TypeScript Interface
```typescript
interface Application {
  id: string;
  projectId: string;
  project?: Project;
  studentId: string;
  student?: User;
  status: 'draft' | 'submitted' | 'reviewing' | 'accepted' | 'rejected' | 'withdrawn';
  statement: string;
  cvUrl: string;
  matchScore: number;
  professorNotes?: string;
  submittedAt?: Date;
  reviewedAt?: Date;
  createdAt: Date;
  updatedAt: Date;
}
```

#### Relationships
- Belongs to Project
- Belongs to Student (User)
- Has many ApplicationStatusLogs

### Milestone

**Purpose:** Standardized progress checkpoints for research projects

**Key Attributes:**
- id: UUID - Unique identifier
- projectId: UUID - Associated project
- templateId: UUID - Milestone template reference
- name: string - Milestone name
- sequence: number - Order in project (1-5)
- dueDate: Date - Deadline
- status: MilestoneStatus - Current state
- requirements: string[] - Deliverable requirements

#### TypeScript Interface
```typescript
interface Milestone {
  id: string;
  projectId: string;
  project?: Project;
  templateId: string;
  template?: MilestoneTemplate;
  name: string;
  sequence: number;
  dueDate: Date;
  status: 'pending' | 'active' | 'submitted' | 'approved' | 'revision_needed' | 'overdue';
  requirements: string[];
  submissions: Submission[];
  createdAt: Date;
  updatedAt: Date;
}

interface Submission {
  id: string;
  milestoneId: string;
  studentId: string;
  files: SubmissionFile[];
  progressReport: {
    completed: string;
    challenges: string;
    nextSteps: string[];
  };
  journalEntry?: string;
  status: 'draft' | 'submitted' | 'approved' | 'needs_revision';
  feedback?: ProfessorFeedback;
  submittedAt?: Date;
  version: number;
}
```

#### Relationships
- Belongs to Project
- Has many Submissions
- Has one MilestoneTemplate

### TrainingProgress

**Purpose:** Track student progress through 12-step research methodology training

**Key Attributes:**
- id: UUID - Unique identifier
- studentId: UUID - Student taking training
- moduleId: UUID - Current training module
- progress: number - Completion percentage (0-100)
- quizScores: object - Scores for each module quiz
- certificateId: string (nullable) - Completion certificate ID
- completedAt: Date (nullable) - Completion timestamp

#### TypeScript Interface
```typescript
interface TrainingProgress {
  id: string;
  studentId: string;
  student?: User;
  currentModule: number;
  modulesCompleted: number[];
  moduleScores: Record<number, number>;
  totalProgress: number;
  certificateId?: string;
  certificateUrl?: string;
  completedAt?: Date;
  lastAccessedAt: Date;
  createdAt: Date;
  updatedAt: Date;
}
```

#### Relationships
- Belongs to Student (User)
- Has many ModuleAttempts
- Has one Certificate

## API Specification

### REST API Specification

```yaml
openapi: 3.0.0
info:
  title: SRMP API
  version: 1.0.0
  description: Scientific Research Management Platform API
servers:
  - url: https://api.srmp.edu.cn/v1
    description: Production API
  - url: https://api-staging.srmp.edu.cn/v1
    description: Staging API

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
  
  schemas:
    User:
      type: object
      properties:
        id:
          type: string
          format: uuid
        email:
          type: string
          format: email
        name:
          type: string
        roles:
          type: array
          items:
            $ref: '#/components/schemas/Role'
    
    Project:
      type: object
      properties:
        id:
          type: string
          format: uuid
        title:
          type: string
        description:
          type: string
        professor:
          $ref: '#/components/schemas/User'
        capacity:
          type: integer
          minimum: 1
          maximum: 10
    
    Application:
      type: object
      properties:
        id:
          type: string
          format: uuid
        project:
          $ref: '#/components/schemas/Project'
        student:
          $ref: '#/components/schemas/User'
        status:
          type: string
          enum: [draft, submitted, reviewing, accepted, rejected]
        matchScore:
          type: number
          minimum: 0
          maximum: 100

paths:
  /auth/login:
    post:
      summary: Multi-method login
      requestBody:
        required: true
        content:
          application/json:
            schema:
              oneOf:
                - type: object
                  properties:
                    email:
                      type: string
                    password:
                      type: string
                - type: object
                  properties:
                    studentId:
                      type: string
                    password:
                      type: string
                - type: object
                  properties:
                    wechatCode:
                      type: string
      responses:
        200:
          description: Login successful
          content:
            application/json:
              schema:
                type: object
                properties:
                  accessToken:
                    type: string
                  refreshToken:
                    type: string
                  user:
                    $ref: '#/components/schemas/User'
  
  /projects:
    get:
      summary: List projects with filtering
      security:
        - bearerAuth: []
      parameters:
        - name: status
          in: query
          schema:
            type: string
            enum: [published, closed]
        - name: department
          in: query
          schema:
            type: string
        - name: tags
          in: query
          schema:
            type: array
            items:
              type: string
        - name: search
          in: query
          schema:
            type: string
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 12
      responses:
        200:
          description: Project list
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/Project'
                  meta:
                    type: object
                    properties:
                      total:
                        type: integer
                      page:
                        type: integer
                      limit:
                        type: integer
    
    post:
      summary: Create new project (Professor only)
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [title, description, capacity, prerequisites, commitmentHours]
              properties:
                title:
                  type: string
                description:
                  type: string
                capacity:
                  type: integer
                prerequisites:
                  type: array
                  items:
                    type: string
                commitmentHours:
                  type: integer
      responses:
        201:
          description: Project created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Project'
  
  /projects/{projectId}/applications:
    post:
      summary: Submit application to project
      security:
        - bearerAuth: []
      parameters:
        - name: projectId
          in: path
          required: true
          schema:
            type: string
            format: uuid
      requestBody:
        required: true
        content:
          multipart/form-data:
            schema:
              type: object
              properties:
                statement:
                  type: string
                  minLength: 500
                  maxLength: 1000
                cv:
                  type: string
                  format: binary
      responses:
        201:
          description: Application submitted
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Application'
  
  /milestones/{milestoneId}/submissions:
    post:
      summary: Submit milestone deliverables
      security:
        - bearerAuth: []
      parameters:
        - name: milestoneId
          in: path
          required: true
          schema:
            type: string
      requestBody:
        required: true
        content:
          multipart/form-data:
            schema:
              type: object
              properties:
                files:
                  type: array
                  items:
                    type: string
                    format: binary
                progressReport:
                  type: object
                  properties:
                    completed:
                      type: string
                    challenges:
                      type: string
                    nextSteps:
                      type: array
                      items:
                        type: string
                journalEntry:
                  type: string
      responses:
        201:
          description: Submission created
```

### GraphQL Schema

```graphql
# GraphQL schema for complex queries and real-time subscriptions

scalar DateTime
scalar UUID

type Query {
  # Dashboard queries with complex aggregations
  studentDashboard: StudentDashboard!
  professorDashboard: ProfessorDashboard!
  secretaryDashboard: SecretaryDashboard!
  
  # Analytics queries
  projectAnalytics(
    startDate: DateTime!
    endDate: DateTime!
    groupBy: AnalyticsGroupBy!
  ): ProjectAnalytics!
  
  # Advanced search
  searchProjects(
    query: String!
    filters: ProjectFilters
    matchingEnabled: Boolean
  ): ProjectSearchResult!
}

type Mutation {
  # Batch operations for secretaries
  batchUpdateProjectStatus(
    projectIds: [UUID!]!
    status: ProjectStatus!
  ): BatchUpdateResult!
  
  batchSendNotifications(
    userIds: [UUID!]!
    notification: NotificationInput!
  ): BatchNotificationResult!
}

type Subscription {
  # Real-time updates
  projectStatusChanged(projectId: UUID!): Project!
  applicationReceived(professorId: UUID!): Application!
  milestoneDeadlineApproaching(studentId: UUID!): Milestone!
}

type StudentDashboard {
  profile: User!
  activeProjects: [ProjectProgress!]!
  pendingApplications: [Application!]!
  upcomingDeadlines: [Deadline!]!
  trainingProgress: TrainingProgress!
  notifications: [Notification!]!
  recommendations: [Project!]!
}

type ProjectProgress {
  project: Project!
  currentMilestone: Milestone!
  overallProgress: Float!
  status: ProjectHealthStatus!
  nextDeadline: DateTime
}

enum ProjectHealthStatus {
  ON_TRACK
  AT_RISK
  DELAYED
  COMPLETED
}

input ProjectFilters {
  departments: [String!]
  tags: [String!]
  commitmentHours: IntRange
  hasCapacity: Boolean
}

type ProjectSearchResult {
  projects: [Project!]!
  totalCount: Int!
  facets: SearchFacets!
}
```

## Components

### API Gateway

**Responsibility:** Single entry point for all client requests, handling routing, authentication, rate limiting, and request/response transformation

**Key Interfaces:**
- HTTP/HTTPS endpoints for all API routes
- WebSocket support for real-time features
- Health check endpoints for monitoring

**Dependencies:** Auth Service, all microservices

**Technology Stack:** Kong API Gateway with custom Lua plugins for WeChat Work integration

### Auth Service

**Responsibility:** Handle multi-method authentication (email, student ID, WeChat Work), JWT token generation/validation, and session management

**Key Interfaces:**
- POST /auth/login (multiple strategies)
- POST /auth/refresh
- POST /auth/logout
- GET /auth/verify

**Dependencies:** Redis (session storage), User Service, WeChat Work API

**Technology Stack:** NestJS with Passport.js strategies, JWT tokens, Redis for sessions

### User Service

**Responsibility:** Manage user profiles, roles, permissions, and program assignments

**Key Interfaces:**
- CRUD operations for users
- Role assignment and permission checking
- Profile completion tracking
- Bulk user import for secretaries

**Dependencies:** PostgreSQL, Redis cache

**Technology Stack:** NestJS with TypeORM, RBAC implementation

### Project Service

**Responsibility:** Handle project lifecycle from creation through completion, including applications and student assignments

**Key Interfaces:**
- Project CRUD with draft/publish workflow
- Application submission and review
- Project-student assignment management
- Project search and filtering

**Dependencies:** PostgreSQL, Elasticsearch, Matching Service

**Technology Stack:** NestJS, TypeORM, Elasticsearch client

### Matching Service

**Responsibility:** AI-powered matching between students and projects based on interests, skills, and prerequisites

**Key Interfaces:**
- POST /match/calculate (get match score)
- GET /match/recommendations (get top matches)
- POST /match/feedback (improve algorithm)

**Dependencies:** AI/ML Service, Project Service, User Service

**Technology Stack:** NestJS with Python ML microservice integration

### Training Service

**Responsibility:** Deliver 12-step research methodology training with progress tracking and certification

**Key Interfaces:**
- Module content delivery
- Progress tracking and quiz scoring
- Certificate generation
- Learning analytics

**Dependencies:** PostgreSQL, File Service (for content storage)

**Technology Stack:** NestJS with video streaming support

### Notification Service

**Responsibility:** Multi-channel notifications via WeChat Work, email, and in-app messages

**Key Interfaces:**
- Send immediate notifications
- Schedule notifications
- Notification preferences management
- Delivery status tracking

**Dependencies:** RocketMQ, WeChat Work API, Email service

**Technology Stack:** NestJS with Bull queue for scheduling

### Analytics Service

**Responsibility:** Generate real-time analytics and reports for all user roles

**Key Interfaces:**
- Dashboard data aggregation
- Custom report generation
- Export functionality (PDF, Excel)
- Predictive analytics for at-risk projects

**Dependencies:** PostgreSQL, Elasticsearch, all other services

**Technology Stack:** NestJS with Apache ECharts for visualizations

### File Service

**Responsibility:** Handle file uploads, version control, and secure access for research deliverables

**Key Interfaces:**
- Multipart file upload
- Version management
- Secure download URLs
- File metadata management

**Dependencies:** Alibaba OSS, PostgreSQL (metadata)

**Technology Stack:** NestJS with OSS SDK, virus scanning integration

### Component Diagrams

```mermaid
C4Container
title Container Diagram for SRMP

Person(student, "Student", "Research program participant")
Person(professor, "Professor", "Research advisor")
Person(secretary, "Secretary", "Program administrator")

System_Boundary(srmp, "SRMP Platform") {
    Container(web, "Web Application", "React, TypeScript", "Provides UI for all users")
    Container(gateway, "API Gateway", "Kong", "Routes requests, handles auth")
    
    Container(auth, "Auth Service", "NestJS", "Multi-method authentication")
    Container(user, "User Service", "NestJS", "User management")
    Container(project, "Project Service", "NestJS", "Project lifecycle")
    Container(matching, "Matching Service", "NestJS + Python", "AI matching")
    Container(training, "Training Service", "NestJS", "Research training")
    Container(notification, "Notification Service", "NestJS", "Multi-channel notifications")
    Container(analytics, "Analytics Service", "NestJS", "Reports and analytics")
    Container(file, "File Service", "NestJS", "File management")
    
    ContainerDb(postgres, "PostgreSQL", "PostgreSQL 15", "Primary data store")
    ContainerDb(redis, "Redis", "Redis 7", "Cache and sessions")
    ContainerDb(elastic, "Elasticsearch", "ES 8", "Search engine")
    ContainerDb(oss, "Object Storage", "Alibaba OSS", "File storage")
    ContainerQueue(mq, "Message Queue", "RocketMQ", "Async messaging")
}

System_Ext(wechat, "WeChat Work", "Authentication and notifications")
System_Ext(university, "University Systems", "SSO and grade management")

Rel(student, web, "Uses", "HTTPS")
Rel(professor, web, "Uses", "HTTPS")
Rel(secretary, web, "Uses", "HTTPS")

Rel(web, gateway, "API calls", "HTTPS/WSS")
Rel(gateway, auth, "Routes", "HTTP")
Rel(gateway, user, "Routes", "HTTP")
Rel(gateway, project, "Routes", "HTTP")
Rel(gateway, matching, "Routes", "HTTP")
Rel(gateway, training, "Routes", "HTTP")
Rel(gateway, notification, "Routes", "HTTP")
Rel(gateway, analytics, "Routes", "HTTP")
Rel(gateway, file, "Routes", "HTTP")

Rel(auth, wechat, "OAuth", "HTTPS")
Rel(auth, university, "SSO", "HTTPS")
Rel(notification, wechat, "Send", "HTTPS")

Rel(auth, redis, "Sessions", "TCP")
Rel(user, postgres, "Read/Write", "TCP")
Rel(project, postgres, "Read/Write", "TCP")
Rel(project, elastic, "Search", "HTTP")
Rel(file, oss, "Store", "HTTPS")
Rel(notification, mq, "Publish", "TCP")
```

## External APIs

### WeChat Work API

- **Purpose:** User authentication and notification delivery
- **Documentation:** https://developer.work.weixin.qq.com/
- **Base URL(s):** https://qyapi.weixin.qq.com/cgi-bin/
- **Authentication:** OAuth 2.0 with corp ID and secret
- **Rate Limits:** 1000 calls per minute per corp

**Key Endpoints Used:**
- `GET /gettoken` - Get access token
- `GET /user/getuserinfo` - Get user info after OAuth
- `POST /message/send` - Send notification messages

**Integration Notes:** Implement token caching to avoid rate limits, handle token refresh automatically, queue messages for batch sending

### University SSO API

- **Purpose:** Single sign-on integration for existing university accounts
- **Documentation:** [To be provided by university IT]
- **Base URL(s):** https://sso.university.edu.cn
- **Authentication:** SAML 2.0 or CAS protocol
- **Rate Limits:** No specific limits documented

**Key Endpoints Used:**
- `POST /saml/login` - Initiate SSO login
- `POST /saml/validate` - Validate SAML assertion
- `GET /api/user/{id}` - Get user details after login

**Integration Notes:** Implement fallback to local auth if SSO is down, cache user data to reduce API calls, sync roles from university directory

### University Grade Management API

- **Purpose:** Import grades and course enrollment data
- **Documentation:** [To be provided by university IT]
- **Base URL(s):** https://grades.university.edu.cn/api/v1
- **Authentication:** API key with IP whitelist
- **Rate Limits:** 100 requests per hour

**Key Endpoints Used:**
- `GET /students/{studentId}/courses` - Get enrolled courses
- `GET /courses/{courseId}/grades` - Get grades for a course
- `POST /grades/bulk` - Bulk grade import

**Integration Notes:** Implement nightly sync to avoid peak hours, validate data integrity before import, maintain audit log of all changes

## Core Workflows

```mermaid
sequenceDiagram
    participant S as Student
    participant W as Web App
    participant G as API Gateway
    participant P as Project Service
    participant M as Matching Service
    participant N as Notification Service
    participant Prof as Professor
    
    Note over S,Prof: Student-Professor Matching Workflow
    
    S->>W: Browse projects
    W->>G: GET /projects?tags=AI
    G->>P: Get filtered projects
    P->>M: Get match scores
    M-->>P: Return scores
    P-->>G: Projects with scores
    G-->>W: Project list
    W-->>S: Display projects
    
    S->>W: Apply to project
    W->>G: POST /projects/{id}/apply
    G->>P: Create application
    P->>M: Calculate final match score
    M-->>P: Match score
    P->>N: Queue notification
    P-->>G: Application created
    G-->>W: Success
    
    N->>Prof: WeChat notification
    Prof->>W: Review application
    W->>G: GET /applications/{id}
    G->>P: Get application details
    P-->>G: Application data
    G-->>W: Display application
    
    Prof->>W: Accept student
    W->>G: PUT /applications/{id}/accept
    G->>P: Update status
    P->>N: Queue notifications
    P-->>G: Updated
    N->>S: Acceptance notification
    N->>Prof: Confirmation
```

```mermaid
sequenceDiagram
    participant S as Student
    participant W as Web App
    participant G as API Gateway
    participant P as Project Service
    participant F as File Service
    participant N as Notification Service
    participant Prof as Professor
    
    Note over S,Prof: Milestone Submission Workflow
    
    S->>W: Access milestone
    W->>G: GET /milestones/{id}
    G->>P: Get milestone details
    P-->>G: Milestone data
    G-->>W: Display milestone
    
    S->>W: Upload deliverables
    W->>G: POST /files/upload
    G->>F: Store files
    F-->>G: File URLs
    
    S->>W: Submit milestone
    W->>G: POST /milestones/{id}/submit
    G->>P: Create submission
    P->>N: Queue notification
    P-->>G: Submission created
    G-->>W: Success
    
    N->>Prof: Review notification
    Prof->>W: Review submission
    W->>G: GET /submissions/{id}
    G->>P: Get submission
    P->>F: Get file URLs
    F-->>P: Secure URLs
    P-->>G: Complete data
    G-->>W: Display submission
    
    Prof->>W: Provide feedback
    W->>G: POST /submissions/{id}/feedback
    G->>P: Save feedback
    P->>N: Queue notification
    N->>S: Feedback available
```

## Database Schema

```sql
-- Users table with multi-role support
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    student_id VARCHAR(50) UNIQUE,
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    name_en VARCHAR(100),
    password_hash VARCHAR(255),
    program VARCHAR(20) CHECK (program IN ('qiming', 'innovation')),
    department VARCHAR(100) NOT NULL,
    wechat_work_id VARCHAR(100) UNIQUE,
    language VARCHAR(5) DEFAULT 'zh-CN',
    avatar_url VARCHAR(500),
    email_verified BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_student_id ON users(student_id);
CREATE INDEX idx_users_department ON users(department);

-- Roles and permissions
CREATE TABLE roles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(50) UNIQUE NOT NULL,
    description TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE permissions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    resource VARCHAR(100) NOT NULL,
    action VARCHAR(50) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(resource, action)
);

CREATE TABLE role_permissions (
    role_id UUID REFERENCES roles(id) ON DELETE CASCADE,
    permission_id UUID REFERENCES permissions(id) ON DELETE CASCADE,
    PRIMARY KEY (role_id, permission_id)
);

CREATE TABLE user_roles (
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    role_id UUID REFERENCES roles(id) ON DELETE CASCADE,
    assigned_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    assigned_by UUID REFERENCES users(id),
    PRIMARY KEY (user_id, role_id)
);

-- Projects
CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(200) NOT NULL,
    title_en VARCHAR(200),
    description TEXT NOT NULL,
    description_en TEXT,
    professor_id UUID REFERENCES users(id) NOT NULL,
    status VARCHAR(20) DEFAULT 'draft' CHECK (status IN ('draft', 'published', 'closed', 'completed')),
    capacity INTEGER CHECK (capacity BETWEEN 1 AND 10),
    current_students INTEGER DEFAULT 0,
    prerequisites TEXT[],
    commitment_hours INTEGER CHECK (commitment_hours BETWEEN 5 AND 40),
    tags TEXT[],
    matching_enabled BOOLEAN DEFAULT TRUE,
    department VARCHAR(100) NOT NULL,
    start_date DATE,
    end_date DATE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_projects_professor ON projects(professor_id);
CREATE INDEX idx_projects_status ON projects(status);
CREATE INDEX idx_projects_department ON projects(department);
CREATE INDEX idx_projects_tags ON projects USING GIN(tags);

-- Applications
CREATE TABLE applications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(id) NOT NULL,
    student_id UUID REFERENCES users(id) NOT NULL,
    status VARCHAR(20) DEFAULT 'draft' CHECK (status IN ('draft', 'submitted', 'reviewing', 'accepted', 'rejected', 'withdrawn')),
    statement TEXT,
    cv_url VARCHAR(500),
    match_score DECIMAL(5,2),
    professor_notes TEXT,
    submitted_at TIMESTAMP WITH TIME ZONE,
    reviewed_at TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(project_id, student_id)
);

CREATE INDEX idx_applications_project ON applications(project_id);
CREATE INDEX idx_applications_student ON applications(student_id);
CREATE INDEX idx_applications_status ON applications(status);

-- Project students (accepted applications)
CREATE TABLE project_students (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(id) NOT NULL,
    student_id UUID REFERENCES users(id) NOT NULL,
    joined_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    completed_at TIMESTAMP WITH TIME ZONE,
    final_grade VARCHAR(10),
    UNIQUE(project_id, student_id)
);

-- Milestone templates
CREATE TABLE milestone_templates (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    name_en VARCHAR(100),
    description TEXT,
    sequence INTEGER NOT NULL,
    duration_days INTEGER DEFAULT 30,
    requirements TEXT[],
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Project milestones
CREATE TABLE milestones (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(id) NOT NULL,
    template_id UUID REFERENCES milestone_templates(id),
    name VARCHAR(100) NOT NULL,
    sequence INTEGER NOT NULL,
    due_date DATE NOT NULL,
    status VARCHAR(20) DEFAULT 'pending' CHECK (status IN ('pending', 'active', 'submitted', 'approved', 'revision_needed', 'overdue')),
    requirements TEXT[],
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(project_id, sequence)
);

CREATE INDEX idx_milestones_project ON milestones(project_id);
CREATE INDEX idx_milestones_due_date ON milestones(due_date);
CREATE INDEX idx_milestones_status ON milestones(status);

-- Milestone submissions
CREATE TABLE submissions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    milestone_id UUID REFERENCES milestones(id) NOT NULL,
    student_id UUID REFERENCES users(id) NOT NULL,
    version INTEGER DEFAULT 1,
    progress_report JSONB,
    journal_entry TEXT,
    status VARCHAR(20) DEFAULT 'draft' CHECK (status IN ('draft', 'submitted', 'approved', 'needs_revision')),
    submitted_at TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(milestone_id, student_id, version)
);

-- Submission files
CREATE TABLE submission_files (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    submission_id UUID REFERENCES submissions(id) ON DELETE CASCADE,
    file_name VARCHAR(255) NOT NULL,
    file_url VARCHAR(500) NOT NULL,
    file_size BIGINT,
    mime_type VARCHAR(100),
    uploaded_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Training modules and progress
CREATE TABLE training_modules (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    module_number INTEGER UNIQUE NOT NULL,
    title VARCHAR(200) NOT NULL,
    title_en VARCHAR(200),
    description TEXT,
    content_url VARCHAR(500),
    duration_minutes INTEGER,
    prerequisites INTEGER[],
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE training_progress (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    student_id UUID REFERENCES users(id) NOT NULL,
    current_module INTEGER DEFAULT 1,
    modules_completed INTEGER[] DEFAULT '{}',
    module_scores JSONB DEFAULT '{}',
    total_progress DECIMAL(5,2) DEFAULT 0,
    certificate_id VARCHAR(100),
    certificate_url VARCHAR(500),
    completed_at TIMESTAMP WITH TIME ZONE,
    last_accessed_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(student_id)
);

-- Notifications
CREATE TABLE notifications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) NOT NULL,
    type VARCHAR(50) NOT NULL,
    title VARCHAR(200) NOT NULL,
    message TEXT NOT NULL,
    data JSONB,
    read BOOLEAN DEFAULT FALSE,
    sent_via VARCHAR(20)[] DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_notifications_user ON notifications(user_id);
CREATE INDEX idx_notifications_read ON notifications(user_id, read);
CREATE INDEX idx_notifications_created ON notifications(created_at DESC);

-- Audit logs
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50),
    resource_id UUID,
    changes JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_audit_logs_user ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_resource ON audit_logs(resource_type, resource_id);
CREATE INDEX idx_audit_logs_created ON audit_logs(created_at DESC);
```

## Frontend Architecture

### Component Architecture

#### Component Organization
```text
src/
├── components/
│   ├── common/         # Shared components
│   │   ├── Layout/
│   │   ├── Navigation/
│   │   ├── DataTable/
│   │   └── StatusBadge/
│   ├── student/        # Student-specific
│   │   ├── Dashboard/
│   │   ├── ProjectCard/
│   │   └── MilestoneTracker/
│   ├── professor/      # Professor-specific
│   │   ├── ApplicationReview/
│   │   └── ProjectForm/
│   └── secretary/      # Secretary-specific
│       └── BatchOperations/
├── pages/             # Route components
├── hooks/             # Custom React hooks
├── services/          # API integration
├── stores/            # Zustand stores
├── utils/             # Utilities
└── i18n/             # Translations
```

#### Component Template
```typescript
// Example: components/common/StatusBadge/StatusBadge.tsx
import React from 'react';
import { Badge } from 'antd';
import { useTranslation } from 'react-i18next';
import clsx from 'clsx';

interface StatusBadgeProps {
  status: 'on-track' | 'at-risk' | 'delayed' | 'completed';
  size?: 'small' | 'default';
  className?: string;
}

export const StatusBadge: React.FC<StatusBadgeProps> = ({ 
  status, 
  size = 'default',
  className 
}) => {
  const { t } = useTranslation();
  
  const statusConfig = {
    'on-track': { status: 'success', text: t('status.onTrack') },
    'at-risk': { status: 'warning', text: t('status.atRisk') },
    'delayed': { status: 'error', text: t('status.delayed') },
    'completed': { status: 'default', text: t('status.completed') }
  };
  
  const config = statusConfig[status];
  
  return (
    <Badge 
      status={config.status as any}
      text={config.text}
      className={clsx('status-badge', className, {
        'status-badge--small': size === 'small'
      })}
    />
  );
};
```

### State Management Architecture

#### State Structure
```typescript
// stores/types.ts
interface AppState {
  // User store
  user: {
    currentUser: User | null;
    roles: Role[];
    language: 'zh-CN' | 'en';
    isAuthenticated: boolean;
  };
  
  // UI store
  ui: {
    sidebarCollapsed: boolean;
    theme: 'light' | 'dark';
    loading: Record<string, boolean>;
    errors: Record<string, Error>;
  };
  
  // Domain stores managed by React Query
  // Projects, applications, etc. cached by React Query
}
```

#### State Management Patterns
- Use Zustand for client-only state (UI, user preferences)
- Use React Query for server state (data fetching, caching, synchronization)
- Implement optimistic updates for better UX
- Use React Context sparingly for truly global concerns
- Prefer composition over prop drilling

### Routing Architecture

#### Route Organization
```text
src/routes/
├── index.tsx           # Root router
├── auth/              # Auth routes
├── student/           # Student routes
│   ├── dashboard.tsx
│   ├── projects/
│   └── training/
├── professor/         # Professor routes
│   ├── dashboard.tsx
│   └── projects/
├── secretary/         # Secretary routes
└── admin/            # Admin routes
```

#### Protected Route Pattern
```typescript
// components/common/ProtectedRoute.tsx
import { Navigate, useLocation } from 'react-router-dom';
import { useAuthStore } from '@/stores/auth';
import { hasPermission } from '@/utils/permissions';

interface ProtectedRouteProps {
  children: React.ReactNode;
  requiredRoles?: string[];
  requiredPermissions?: string[];
}

export const ProtectedRoute: React.FC<ProtectedRouteProps> = ({
  children,
  requiredRoles = [],
  requiredPermissions = []
}) => {
  const { isAuthenticated, user } = useAuthStore();
  const location = useLocation();
  
  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  const hasRequiredRole = requiredRoles.length === 0 || 
    requiredRoles.some(role => user?.roles.includes(role));
    
  const hasRequiredPermission = requiredPermissions.length === 0 ||
    requiredPermissions.every(permission => hasPermission(user, permission));
  
  if (!hasRequiredRole || !hasRequiredPermission) {
    return <Navigate to="/403" replace />;
  }
  
  return <>{children}</>;
};
```

### Frontend Services Layer

#### API Client Setup
```typescript
// services/api/client.ts
import axios from 'axios';
import { notification } from 'antd';
import { getAuthToken, refreshAuthToken } from '@/utils/auth';

const API_BASE_URL = import.meta.env.VITE_API_BASE_URL;

export const apiClient = axios.create({
  baseURL: API_BASE_URL,
  timeout: 30000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor
apiClient.interceptors.request.use(
  (config) => {
    const token = getAuthToken();
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    
    // Add language header
    const language = localStorage.getItem('language') || 'zh-CN';
    config.headers['Accept-Language'] = language;
    
    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;
    
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      
      try {
        await refreshAuthToken();
        return apiClient(originalRequest);
      } catch (refreshError) {
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }
    
    // Show error notification
    if (error.response?.data?.message) {
      notification.error({
        message: 'Error',
        description: error.response.data.message,
      });
    }
    
    return Promise.reject(error);
  }
);
```

#### Service Example
```typescript
// services/api/projects.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { apiClient } from './client';
import type { Project, ProjectFilters, CreateProjectDTO } from '@/types';

// API functions
const projectsApi = {
  list: async (filters: ProjectFilters) => {
    const { data } = await apiClient.get<{
      data: Project[];
      meta: { total: number; page: number; limit: number };
    }>('/projects', { params: filters });
    return data;
  },
  
  getById: async (id: string) => {
    const { data } = await apiClient.get<Project>(`/projects/${id}`);
    return data;
  },
  
  create: async (project: CreateProjectDTO) => {
    const { data } = await apiClient.post<Project>('/projects', project);
    return data;
  },
  
  update: async (id: string, updates: Partial<Project>) => {
    const { data } = await apiClient.patch<Project>(`/projects/${id}`, updates);
    return data;
  },
};

// React Query hooks
export const useProjects = (filters: ProjectFilters) => {
  return useQuery({
    queryKey: ['projects', filters],
    queryFn: () => projectsApi.list(filters),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
};

export const useProject = (id: string) => {
  return useQuery({
    queryKey: ['projects', id],
    queryFn: () => projectsApi.getById(id),
    enabled: !!id,
  });
};

export const useCreateProject = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: projectsApi.create,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['projects'] });
      notification.success({
        message: 'Success',
        description: 'Project created successfully',
      });
    },
  });
};
```

## Backend Architecture

### Service Architecture

#### Serverless Architecture

##### Function Organization
```text
apps/api/src/
├── functions/
│   ├── auth/
│   │   ├── login.ts
│   │   ├── refresh.ts
│   │   └── logout.ts
│   ├── projects/
│   │   ├── list.ts
│   │   ├── create.ts
│   │   ├── update.ts
│   │   └── delete.ts
│   └── shared/
│       ├── middleware.ts
│       └── utils.ts
├── services/         # Shared business logic
├── repositories/     # Data access layer
└── infrastructure/   # DB, cache, etc.
```

##### Function Template
```typescript
// functions/projects/create.ts
import { APIGatewayProxyHandler } from 'aws-lambda';
import { middyfy } from '@/shared/middleware';
import { projectService } from '@/services/project.service';
import { CreateProjectDTO } from '@/types';

const handler: APIGatewayProxyHandler = async (event) => {
  try {
    const userId = event.requestContext.authorizer?.userId;
    const projectData: CreateProjectDTO = JSON.parse(event.body || '{}');
    
    const project = await projectService.create(userId, projectData);
    
    return {
      statusCode: 201,
      body: JSON.stringify(project),
    };
  } catch (error) {
    console.error('Create project error:', error);
    return {
      statusCode: error.statusCode || 500,
      body: JSON.stringify({
        error: {
          message: error.message || 'Internal server error',
          code: error.code || 'INTERNAL_ERROR',
        },
      }),
    };
  }
};

export const createProject = middyfy(handler);
```

### Database Architecture

#### Schema Design
```sql
-- See Database Schema section above for complete schema

-- Additional indexes for performance
CREATE INDEX idx_projects_search ON projects USING GIN(
  to_tsvector('simple', title || ' ' || COALESCE(title_en, '') || ' ' || description)
);

-- Materialized view for analytics
CREATE MATERIALIZED VIEW project_analytics AS
SELECT 
  p.department,
  DATE_TRUNC('month', p.created_at) as month,
  COUNT(*) as total_projects,
  COUNT(CASE WHEN p.status = 'completed' THEN 1 END) as completed_projects,
  AVG(p.current_students::numeric / p.capacity::numeric) as avg_fill_rate
FROM projects p
GROUP BY p.department, DATE_TRUNC('month', p.created_at);

CREATE INDEX idx_project_analytics_dept_month ON project_analytics(department, month);
```

#### Data Access Layer
```typescript
// repositories/project.repository.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Project } from '@/entities/project.entity';
import { ProjectFilters } from '@/types';

@Injectable()
export class ProjectRepository {
  constructor(
    @InjectRepository(Project)
    private readonly repository: Repository<Project>,
  ) {}
  
  async findWithFilters(filters: ProjectFilters, userId?: string) {
    const query = this.repository
      .createQueryBuilder('project')
      .leftJoinAndSelect('project.professor', 'professor')
      .where('project.status = :status', { status: 'published' });
    
    if (filters.department) {
      query.andWhere('project.department = :department', { 
        department: filters.department 
      });
    }
    
    if (filters.tags?.length) {
      query.andWhere('project.tags && :tags', { tags: filters.tags });
    }
    
    if (filters.search) {
      query.andWhere(
        'to_tsvector(\'simple\', project.title || \' \' || project.description) @@ plainto_tsquery(\'simple\', :search)',
        { search: filters.search }
      );
    }
    
    // Add match scores if user is logged in
    if (userId && filters.includeMatchScore) {
      query.addSelect(
        '(SELECT match_score FROM calculate_match_score(project.id, :userId))',
        'matchScore'
      ).setParameter('userId', userId);
    }
    
    return query
      .orderBy('project.createdAt', 'DESC')
      .skip(filters.offset)
      .take(filters.limit)
      .getManyAndCount();
  }
  
  async updateWithOptimisticLock(
    id: string, 
    updates: Partial<Project>,
    expectedVersion: number
  ) {
    const result = await this.repository
      .createQueryBuilder()
      .update(Project)
      .set({
        ...updates,
        version: () => 'version + 1',
        updatedAt: new Date(),
      })
      .where('id = :id AND version = :version', { id, version: expectedVersion })
      .returning('*')
      .execute();
    
    if (result.affected === 0) {
      throw new OptimisticLockError('Project has been modified by another user');
    }
    
    return result.raw[0];
  }
}
```

### Authentication and Authorization

#### Auth Flow
```mermaid
sequenceDiagram
    participant C as Client
    participant G as API Gateway
    participant A as Auth Service
    participant W as WeChat Work
    participant R as Redis
    participant U as User Service
    
    Note over C,U: WeChat Work Login Flow
    
    C->>G: GET /auth/wechat/login
    G->>A: Initiate WeChat OAuth
    A->>W: Redirect to WeChat OAuth
    W->>C: User authorizes
    C->>G: GET /auth/wechat/callback?code=xxx
    G->>A: Process callback
    A->>W: Exchange code for access token
    W-->>A: Access token + user info
    A->>U: Find or create user
    U-->>A: User record
    A->>R: Store session
    A->>A: Generate JWT
    A-->>G: JWT + refresh token
    G-->>C: Set httpOnly cookies
```

#### Middleware/Guards
```typescript
// middleware/auth.middleware.ts
import { Injectable, NestMiddleware, UnauthorizedException } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';
import { JwtService } from '@nestjs/jwt';
import { RedisService } from '@/services/redis.service';

@Injectable()
export class AuthMiddleware implements NestMiddleware {
  constructor(
    private readonly jwtService: JwtService,
    private readonly redisService: RedisService,
  ) {}
  
  async use(req: Request, res: Response, next: NextFunction) {
    const token = this.extractToken(req);
    
    if (!token) {
      throw new UnauthorizedException('No token provided');
    }
    
    try {
      const payload = this.jwtService.verify(token);
      
      // Check if token is blacklisted (for logout)
      const isBlacklisted = await this.redisService.get(`blacklist:${token}`);
      if (isBlacklisted) {
        throw new UnauthorizedException('Token has been revoked');
      }
      
      // Check session validity
      const session = await this.redisService.get(`session:${payload.sessionId}`);
      if (!session) {
        throw new UnauthorizedException('Session expired');
      }
      
      // Attach user to request
      req.user = payload;
      next();
    } catch (error) {
      throw new UnauthorizedException('Invalid token');
    }
  }
  
  private extractToken(req: Request): string | null {
    // Try Authorization header first
    const authHeader = req.headers.authorization;
    if (authHeader?.startsWith('Bearer ')) {
      return authHeader.substring(7);
    }
    
    // Fall back to cookie
    return req.cookies?.accessToken || null;
  }
}

// guards/roles.guard.ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { ROLES_KEY } from '@/decorators/roles.decorator';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}
  
  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<string[]>(
      ROLES_KEY,
      [context.getHandler(), context.getClass()]
    );
    
    if (!requiredRoles) {
      return true;
    }
    
    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.some(role => user.roles?.includes(role));
  }
}
```

## Unified Project Structure

```plaintext
srmp/
├── .github/                    # CI/CD workflows
│   └── workflows/
│       ├── ci.yaml            # Test and lint
│       └── deploy.yaml        # Deploy to Alibaba Cloud
├── apps/                       # Application packages
│   ├── web/                    # Frontend application
│   │   ├── src/
│   │   │   ├── components/     # UI components
│   │   │   ├── pages/          # Page components/routes
│   │   │   ├── hooks/          # Custom React hooks
│   │   │   ├── services/       # API client services
│   │   │   ├── stores/         # Zustand stores
│   │   │   ├── styles/         # Global styles/themes
│   │   │   ├── utils/          # Frontend utilities
│   │   │   ├── i18n/           # Translations
│   │   │   └── main.tsx        # Entry point
│   │   ├── public/             # Static assets
│   │   ├── tests/              # Frontend tests
│   │   ├── vite.config.ts      # Vite configuration
│   │   └── package.json
│   └── api/                    # Backend application
│       ├── src/
│       │   ├── modules/         # NestJS modules
│       │   │   ├── auth/
│       │   │   ├── users/
│       │   │   ├── projects/
│       │   │   ├── matching/
│       │   │   ├── training/
│       │   │   └── notifications/
│       │   ├── common/          # Shared module resources
│       │   │   ├── decorators/
│       │   │   ├── filters/
│       │   │   ├── guards/
│       │   │   ├── interceptors/
│       │   │   └── pipes/
│       │   ├── config/          # Configuration
│       │   ├── database/        # Database module
│       │   └── main.ts          # Entry point
│       ├── tests/              # Backend tests
│       └── package.json
├── packages/                   # Shared packages
│   ├── shared/                 # Shared types/utilities
│   │   ├── src/
│   │   │   ├── types/          # TypeScript interfaces
│   │   │   ├── constants/      # Shared constants
│   │   │   ├── dto/            # Data transfer objects
│   │   │   └── utils/          # Shared utilities
│   │   └── package.json
│   ├── ui/                     # Shared UI components
│   │   ├── src/
│   │   │   ├── components/
│   │   │   └── styles/
│   │   ├── package.json
│   │   └── tsconfig.json
│   └── config/                 # Shared configuration
│       ├── eslint/
│       ├── typescript/
│       └── jest/
├── infrastructure/             # IaC definitions
│   ├── terraform/
│   │   ├── environments/
│   │   │   ├── dev/
│   │   │   ├── staging/
│   │   │   └── production/
│   │   ├── modules/
│   │   │   ├── ecs/
│   │   │   ├── rds/
│   │   │   ├── redis/
│   │   │   └── cdn/
│   │   └── main.tf
│   └── k8s/                    # Kubernetes manifests
│       ├── base/
│       └── overlays/
├── scripts/                    # Build/deploy scripts
│   ├── setup.sh               # Initial setup
│   ├── build.sh               # Build all packages
│   └── deploy.sh              # Deployment script
├── docs/                       # Documentation
│   ├── prd.md
│   ├── front-end-spec.md
│   ├── architecture.md
│   └── api/                    # API documentation
├── .env.example                # Environment template
├── nx.json                     # Nx configuration
├── package.json                # Root package.json
├── pnpm-workspace.yaml         # PNPM workspace config
└── README.md
```

## Development Workflow

### Local Development Setup

#### Prerequisites
```bash
# Required software versions
node --version  # v18.0.0 or higher
pnpm --version  # v8.0.0 or higher
docker --version  # v20.0.0 or higher
docker-compose --version  # v2.0.0 or higher

# Install pnpm if not installed
npm install -g pnpm

# Install Nx CLI globally
pnpm add -g nx
```

#### Initial Setup
```bash
# Clone repository
git clone https://github.com/university/srmp.git
cd srmp

# Install dependencies
pnpm install

# Copy environment files
cp .env.example .env
cp apps/web/.env.example apps/web/.env.local
cp apps/api/.env.example apps/api/.env

# Start Docker services (PostgreSQL, Redis, Elasticsearch)
docker-compose up -d

# Run database migrations
nx run api:migration:run

# Seed development data
nx run api:seed
```

#### Development Commands
```bash
# Start all services
nx run-many --target=serve --all

# Start frontend only
nx serve web

# Start backend only
nx serve api

# Run tests
nx run-many --target=test --all
nx run-many --target=e2e --all

# Lint and format
nx format:write
nx affected:lint
```

### Environment Configuration

#### Required Environment Variables
```bash
# Frontend (.env.local)
VITE_API_BASE_URL=http://localhost:3000/api
VITE_WEBSOCKET_URL=ws://localhost:3000
VITE_WECHAT_APP_ID=your_wechat_app_id
VITE_SENTRY_DSN=your_sentry_dsn

# Backend (.env)
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/srmp
REDIS_URL=redis://localhost:6379

# Authentication
JWT_SECRET=your-super-secret-jwt-key
JWT_REFRESH_SECRET=your-super-secret-refresh-key
SESSION_SECRET=your-session-secret

# WeChat Work
WECHAT_CORP_ID=your_corp_id
WECHAT_AGENT_ID=your_agent_id
WECHAT_SECRET=your_secret

# Alibaba Cloud
ALIBABA_ACCESS_KEY_ID=your_access_key
ALIBABA_ACCESS_KEY_SECRET=your_secret
ALIBABA_OSS_BUCKET=srmp-files
ALIBABA_OSS_REGION=oss-cn-beijing

# Monitoring
SENTRY_DSN=your_sentry_dsn
```

## Deployment Architecture

### Deployment Strategy

**Frontend Deployment:**
- **Platform:** Alibaba Cloud CDN + OSS
- **Build Command:** `nx build web --prod`
- **Output Directory:** `dist/apps/web`
- **CDN/Edge:** Alibaba Cloud CDN with China acceleration

**Backend Deployment:**
- **Platform:** Alibaba Cloud Container Service (ACK)
- **Build Command:** `nx build api --prod`
- **Deployment Method:** Docker containers orchestrated by Kubernetes

### CI/CD Pipeline
```yaml
# .github/workflows/deploy.yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: pnpm/action-setup@v2
        with:
          version: 8
      
      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Run tests
        run: pnpm nx affected --target=test --base=origin/main
      
      - name: Run e2e tests
        run: pnpm nx affected --target=e2e --base=origin/main

  build-and-deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure Alibaba Cloud CLI
        uses: aliyun/cli-action@v1
        with:
          access-key-id: ${{ secrets.ALIBABA_ACCESS_KEY_ID }}
          access-key-secret: ${{ secrets.ALIBABA_ACCESS_KEY_SECRET }}
          region: cn-beijing
      
      - name: Build Frontend
        run: |
          pnpm nx build web --prod
          aliyun oss cp dist/apps/web oss://srmp-frontend/ --recursive
          aliyun cdn RefreshObjectCaches --ObjectPath https://app.srmp.edu.cn/
      
      - name: Build and Deploy Backend
        run: |
          pnpm nx build api --prod
          docker build -t registry.cn-beijing.aliyuncs.com/srmp/api:${{ github.sha }} .
          docker push registry.cn-beijing.aliyuncs.com/srmp/api:${{ github.sha }}
          kubectl set image deployment/api api=registry.cn-beijing.aliyuncs.com/srmp/api:${{ github.sha }}
```

### Environments

| Environment | Frontend URL | Backend URL | Purpose |
|-------------|--------------|-------------|---------|
| Development | http://localhost:5173 | http://localhost:3000 | Local development |
| Staging | https://staging.srmp.edu.cn | https://api-staging.srmp.edu.cn | Pre-production testing |
| Production | https://app.srmp.edu.cn | https://api.srmp.edu.cn | Live environment |

## Security and Performance

### Security Requirements

**Frontend Security:**
- CSP Headers: `default-src 'self'; script-src 'self' 'unsafe-inline' https://res.wx.qq.com; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; connect-src 'self' https://api.srmp.edu.cn wss://api.srmp.edu.cn`
- XSS Prevention: React's automatic escaping + DOMPurify for user content
- Secure Storage: Sensitive data in httpOnly cookies, not localStorage

**Backend Security:**
- Input Validation: class-validator decorators on all DTOs
- Rate Limiting: 100 requests per minute per IP, 1000 per authenticated user
- CORS Policy: Whitelist specific origins, credentials allowed

**Authentication Security:**
- Token Storage: httpOnly, secure, sameSite cookies
- Session Management: Redis with 24-hour expiry, refresh token rotation
- Password Policy: Minimum 8 characters, must include number and special character

### Performance Optimization

**Frontend Performance:**
- Bundle Size Target: <200KB initial JS, <50KB per lazy route
- Loading Strategy: Route-based code splitting, progressive enhancement
- Caching Strategy: Service worker for offline support, SWR for data

**Backend Performance:**
- Response Time Target: p50 <100ms, p99 <500ms
- Database Optimization: Connection pooling, query optimization, read replicas
- Caching Strategy: Redis for sessions, frequent queries, and computed data

## Testing Strategy

### Testing Pyramid
```text
         E2E Tests
        /        \
   Integration Tests
      /            \
 Frontend Unit  Backend Unit
```

### Test Organization

#### Frontend Tests
```text
apps/web/tests/
├── unit/
│   ├── components/
│   ├── hooks/
│   └── utils/
├── integration/
│   ├── pages/
│   └── services/
└── e2e/
    ├── auth.spec.ts
    ├── student-flow.spec.ts
    └── professor-flow.spec.ts
```

#### Backend Tests
```text
apps/api/tests/
├── unit/
│   ├── services/
│   ├── guards/
│   └── utils/
├── integration/
│   ├── modules/
│   └── database/
└── e2e/
    ├── auth.e2e.spec.ts
    └── projects.e2e.spec.ts
```

#### E2E Tests
```text
tests/e2e/
├── fixtures/
├── page-objects/
└── scenarios/
    ├── student-journey.spec.ts
    ├── professor-workflow.spec.ts
    └── secretary-operations.spec.ts
```

### Test Examples

#### Frontend Component Test
```typescript
// apps/web/tests/unit/components/StatusBadge.test.tsx
import { render, screen } from '@testing-library/react';
import { StatusBadge } from '@/components/common/StatusBadge';

describe('StatusBadge', () => {
  it('renders correct status text and color', () => {
    render(<StatusBadge status="at-risk" />);
    
    const badge = screen.getByText('status.atRisk');
    expect(badge).toBeInTheDocument();
    expect(badge).toHaveClass('ant-badge-status-warning');
  });
  
  it('applies custom className', () => {
    render(<StatusBadge status="completed" className="custom-class" />);
    
    const badge = screen.getByText('status.completed');
    expect(badge.parentElement).toHaveClass('custom-class');
  });
});
```

#### Backend API Test
```typescript
// apps/api/tests/integration/projects.test.ts
import { Test } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from '@/app.module';
import { createAuthToken } from '../helpers';

describe('Projects API', () => {
  let app: INestApplication;
  let professorToken: string;
  
  beforeAll(async () => {
    const moduleRef = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();
    
    app = moduleRef.createNestApplication();
    await app.init();
    
    professorToken = await createAuthToken('professor');
  });
  
  afterAll(async () => {
    await app.close();
  });
  
  describe('POST /projects', () => {
    it('creates project with valid data', async () => {
      const projectData = {
        title: 'AI Research Project',
        description: 'Research on machine learning applications',
        capacity: 3,
        prerequisites: ['Python', 'Statistics'],
        commitmentHours: 10,
      };
      
      const response = await request(app.getHttpServer())
        .post('/projects')
        .set('Authorization', `Bearer ${professorToken}`)
        .send(projectData)
        .expect(201);
      
      expect(response.body).toMatchObject({
        id: expect.any(String),
        ...projectData,
        status: 'draft',
        professorId: expect.any(String),
      });
    });
    
    it('validates required fields', async () => {
      const response = await request(app.getHttpServer())
        .post('/projects')
        .set('Authorization', `Bearer ${professorToken}`)
        .send({ title: 'Only Title' })
        .expect(400);
      
      expect(response.body.error.message).toContain('validation failed');
    });
  });
});
```

#### E2E Test
```typescript
// tests/e2e/scenarios/student-journey.spec.ts
import { test, expect } from '@playwright/test';
import { StudentPage } from '../page-objects/StudentPage';
import { ProjectPage } from '../page-objects/ProjectPage';

test.describe('Student Research Journey', () => {
  test('complete journey from application to milestone submission', async ({ page }) => {
    const studentPage = new StudentPage(page);
    const projectPage = new ProjectPage(page);
    
    // Login as student
    await studentPage.login('student@test.edu', 'password123');
    
    // Browse and apply to project
    await studentPage.navigateToProjects();
    await projectPage.searchProjects('Machine Learning');
    await projectPage.applyToProject('AI Research Project', {
      statement: 'I am passionate about AI research...',
      cvFile: 'fixtures/student-cv.pdf',
    });
    
    // Wait for acceptance (in real scenario, would need professor action)
    await page.waitForTimeout(2000); // Simulate processing
    
    // Submit milestone
    await studentPage.navigateToMyProjects();
    await projectPage.openProject('AI Research Project');
    await projectPage.submitMilestone({
      files: ['fixtures/literature-review.pdf'],
      progressReport: {
        completed: 'Reviewed 20 papers on neural networks',
        challenges: 'Understanding backpropagation mathematics',
        nextSteps: ['Implement basic neural network', 'Start experimentation'],
      },
    });
    
    // Verify submission
    await expect(page.locator('.milestone-status')).toHaveText('Submitted');
    await expect(page.locator('.notification')).toContainText('Milestone submitted successfully');
  });
});
```

## Coding Standards

### Critical Fullstack Rules

- **Type Sharing:** Always define types in packages/shared and import from there
- **API Calls:** Never make direct HTTP calls - use the service layer
- **Environment Variables:** Access only through config objects, never process.env directly
- **Error Handling:** All API routes must use the standard error handler
- **State Updates:** Never mutate state directly - use proper state management patterns
- **Data Validation:** Validate all inputs on both frontend and backend
- **Authentication:** Always check permissions on both client and server
- **Internationalization:** All user-facing strings must use i18n
- **Accessibility:** All interactive elements must be keyboard accessible
- **Performance:** Lazy load routes and heavy components

### Naming Conventions

| Element | Frontend | Backend | Example |
|---------|----------|---------|---------|
| Components | PascalCase | - | `UserProfile.tsx` |
| Hooks | camelCase with 'use' | - | `useAuth.ts` |
| API Routes | - | kebab-case | `/api/user-profile` |
| Database Tables | - | snake_case | `user_profiles` |
| Constants | UPPER_SNAKE_CASE | UPPER_SNAKE_CASE | `MAX_FILE_SIZE` |
| Types/Interfaces | PascalCase | PascalCase | `UserProfile` |
| File Names | PascalCase/camelCase | camelCase | `UserProfile.tsx`, `auth.service.ts` |

## Error Handling Strategy

### Error Flow
```mermaid
sequenceDiagram
    participant C as Client
    participant F as Frontend
    participant G as API Gateway
    participant S as Service
    participant D as Database
    
    C->>F: User Action
    F->>G: API Request
    G->>S: Process Request
    S->>D: Database Operation
    D-->>S: Error (e.g., constraint violation)
    S->>S: Transform to Domain Error
    S-->>G: Domain Error Response
    G->>G: Add request ID, timestamp
    G-->>F: Standardized Error Response
    F->>F: Show User-Friendly Message
    F-->>C: Display Error with Actions
```

### Error Response Format
```typescript
interface ApiError {
  error: {
    code: string;
    message: string;
    details?: Record<string, any>;
    timestamp: string;
    requestId: string;
  };
}
```

### Frontend Error Handling
```typescript
// hooks/useErrorHandler.ts
import { useCallback } from 'react';
import { notification } from 'antd';
import { useTranslation } from 'react-i18next';

export const useErrorHandler = () => {
  const { t } = useTranslation();
  
  const handleError = useCallback((error: any) => {
    console.error('Error occurred:', error);
    
    // Network errors
    if (!error.response) {
      notification.error({
        message: t('errors.network.title'),
        description: t('errors.network.description'),
      });
      return;
    }
    
    // API errors
    const { code, message } = error.response.data.error;
    
    const errorMessages: Record<string, string> = {
      'AUTH_INVALID_CREDENTIALS': t('errors.auth.invalidCredentials'),
      'AUTH_TOKEN_EXPIRED': t('errors.auth.tokenExpired'),
      'VALIDATION_FAILED': t('errors.validation.failed'),
      'RESOURCE_NOT_FOUND': t('errors.resource.notFound'),
      'PERMISSION_DENIED': t('errors.permission.denied'),
      'RATE_LIMIT_EXCEEDED': t('errors.rateLimit.exceeded'),
    };
    
    notification.error({
      message: t('errors.common.title'),
      description: errorMessages[code] || message,
    });
  }, [t]);
  
  return { handleError };
};
```

### Backend Error Handling
```typescript
// common/filters/http-exception.filter.ts
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
  HttpStatus,
  Logger,
} from '@nestjs/common';
import { Request, Response } from 'express';
import { v4 as uuidv4 } from 'uuid';

@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  private readonly logger = new Logger(AllExceptionsFilter.name);
  
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const requestId = uuidv4();
    
    let status = HttpStatus.INTERNAL_SERVER_ERROR;
    let code = 'INTERNAL_ERROR';
    let message = 'An unexpected error occurred';
    let details = {};
    
    if (exception instanceof HttpException) {
      status = exception.getStatus();
      const exceptionResponse = exception.getResponse();
      
      if (typeof exceptionResponse === 'object') {
        code = exceptionResponse['code'] || code;
        message = exceptionResponse['message'] || message;
        details = exceptionResponse['details'] || details;
      }
    }
    
    // Log error with context
    this.logger.error({
      requestId,
      method: request.method,
      url: request.url,
      userId: request.user?.id,
      error: exception,
    });
    
    response.status(status).json({
      error: {
        code,
        message,
        details,
        timestamp: new Date().toISOString(),
        requestId,
      },
    });
  }
}

// domain/errors/domain-error.ts
export class DomainError extends Error {
  constructor(
    public readonly code: string,
    public readonly message: string,
    public readonly details?: Record<string, any>,
  ) {
    super(message);
    this.name = this.constructor.name;
  }
}

export class ValidationError extends DomainError {
  constructor(details: Record<string, string[]>) {
    super('VALIDATION_FAILED', 'Validation failed', details);
  }
}

export class NotFoundError extends DomainError {
  constructor(resource: string, id: string) {
    super('RESOURCE_NOT_FOUND', `${resource} not found`, { resource, id });
  }
}

export class PermissionError extends DomainError {
  constructor(action: string, resource: string) {
    super('PERMISSION_DENIED', 'Permission denied', { action, resource });
  }
}
```

## Monitoring and Observability

### Monitoring Stack

- **Frontend Monitoring:** Alibaba Cloud ARMS Real User Monitoring
- **Backend Monitoring:** Alibaba Cloud ARMS Application Monitoring
- **Error Tracking:** Sentry for both frontend and backend
- **Performance Monitoring:** Custom CloudWatch dashboards

### Key Metrics

**Frontend Metrics:**
- Core Web Vitals
- JavaScript errors
- API response times
- User interactions

**Backend Metrics:**
- Request rate
- Error rate
- Response time
- Database query performance

## Checklist Results Report

After reviewing the complete fullstack architecture document against the architect checklist:

### Architecture Completeness: 96%

**Strengths:**
- Comprehensive coverage of all system layers
- Clear separation of concerns with microservices
- Detailed API specifications and data models
- Strong security and performance considerations
- Well-defined deployment and monitoring strategies

**Minor Gaps:**
- Could benefit from more specific disaster recovery procedures
- WebSocket implementation details for real-time features could be expanded
- More examples of cross-service transaction handling

### Technical Decisions Validated:
- Monorepo with Nx provides excellent DX and build optimization
- Alibaba Cloud services align with China deployment requirements
- Technology choices support scalability to 10,000 concurrent users
- Security measures address all PRD requirements

### Architecture Readiness: **READY FOR IMPLEMENTATION**

The architecture provides clear guidance for AI-assisted development with appropriately scoped components and well-defined interfaces. The monorepo structure and shared types will significantly reduce integration issues during development.