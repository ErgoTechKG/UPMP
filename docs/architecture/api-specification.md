# API Specification

## REST API Specification

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

## GraphQL Schema

```graphql