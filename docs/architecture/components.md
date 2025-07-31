# Components

## API Gateway

**Responsibility:** Single entry point for all client requests, handling routing, authentication, rate limiting, and request/response transformation

**Key Interfaces:**
- HTTP/HTTPS endpoints for all API routes
- WebSocket support for real-time features
- Health check endpoints for monitoring

**Dependencies:** Auth Service, all microservices

**Technology Stack:** Kong API Gateway with custom Lua plugins for WeChat Work integration

## Auth Service

**Responsibility:** Handle multi-method authentication (email, student ID, WeChat Work), JWT token generation/validation, and session management

**Key Interfaces:**
- POST /auth/login (multiple strategies)
- POST /auth/refresh
- POST /auth/logout
- GET /auth/verify

**Dependencies:** Redis (session storage), User Service, WeChat Work API

**Technology Stack:** NestJS with Passport.js strategies, JWT tokens, Redis for sessions

## User Service

**Responsibility:** Manage user profiles, roles, permissions, and program assignments

**Key Interfaces:**
- CRUD operations for users
- Role assignment and permission checking
- Profile completion tracking
- Bulk user import for secretaries

**Dependencies:** PostgreSQL, Redis cache

**Technology Stack:** NestJS with TypeORM, RBAC implementation

## Project Service

**Responsibility:** Handle project lifecycle from creation through completion, including applications and student assignments

**Key Interfaces:**
- Project CRUD with draft/publish workflow
- Application submission and review
- Project-student assignment management
- Project search and filtering

**Dependencies:** PostgreSQL, Elasticsearch, Matching Service

**Technology Stack:** NestJS, TypeORM, Elasticsearch client

## Matching Service

**Responsibility:** AI-powered matching between students and projects based on interests, skills, and prerequisites

**Key Interfaces:**
- POST /match/calculate (get match score)
- GET /match/recommendations (get top matches)
- POST /match/feedback (improve algorithm)

**Dependencies:** AI/ML Service, Project Service, User Service

**Technology Stack:** NestJS with Python ML microservice integration

## Training Service

**Responsibility:** Deliver 12-step research methodology training with progress tracking and certification

**Key Interfaces:**
- Module content delivery
- Progress tracking and quiz scoring
- Certificate generation
- Learning analytics

**Dependencies:** PostgreSQL, File Service (for content storage)

**Technology Stack:** NestJS with video streaming support

## Notification Service

**Responsibility:** Multi-channel notifications via WeChat Work, email, and in-app messages

**Key Interfaces:**
- Send immediate notifications
- Schedule notifications
- Notification preferences management
- Delivery status tracking

**Dependencies:** RocketMQ, WeChat Work API, Email service

**Technology Stack:** NestJS with Bull queue for scheduling

## Analytics Service

**Responsibility:** Generate real-time analytics and reports for all user roles

**Key Interfaces:**
- Dashboard data aggregation
- Custom report generation
- Export functionality (PDF, Excel)
- Predictive analytics for at-risk projects

**Dependencies:** PostgreSQL, Elasticsearch, all other services

**Technology Stack:** NestJS with Apache ECharts for visualizations

## File Service

**Responsibility:** Handle file uploads, version control, and secure access for research deliverables

**Key Interfaces:**
- Multipart file upload
- Version management
- Secure download URLs
- File metadata management

**Dependencies:** Alibaba OSS, PostgreSQL (metadata)

**Technology Stack:** NestJS with OSS SDK, virus scanning integration

## Component Diagrams

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
