# Nx Monorepo Setup Commands - SRMP

## Initial Setup Commands

### 1. Create Nx Workspace

```bash
# Create the workspace
npx create-nx-workspace@latest srmp \
  --preset=empty \
  --packageManager=pnpm \
  --nx-cloud=true

cd srmp

# Install additional Nx plugins
pnpm add -D @nx/react @nx/nest @nx/node @nx/web @nx/jest @nx/eslint @nx/workspace
```

### 2. Configure Workspace Structure

```bash
# Create directory structure
mkdir -p apps/web
mkdir -p apps/api-gateway  
mkdir -p apps/services/{auth,user,project,matching,training,notification,analytics,file}
mkdir -p packages/types
mkdir -p packages/utils
mkdir -p packages/ui
mkdir -p infrastructure/{terraform,k8s,docker}
```

### 3. Generate Applications

```bash
# Generate React application
nx g @nx/react:app web \
  --style=css \
  --routing=true \
  --bundler=vite \
  --unitTestRunner=vitest \
  --e2eTestRunner=playwright \
  --tags="scope:frontend,type:app"

# Generate API Gateway
nx g @nx/nest:app api-gateway \
  --tags="scope:backend,type:app"

# Generate microservices
for service in auth user project matching training notification analytics file; do
  nx g @nx/nest:app services/$service \
    --tags="scope:backend,type:service,domain:$service"
done
```

### 4. Generate Shared Libraries

```bash
# Generate shared types library
nx g @nx/workspace:lib types \
  --directory=packages \
  --tags="scope:shared,type:types" \
  --importPath="@srmp/types"

# Generate utilities library  
nx g @nx/workspace:lib utils \
  --directory=packages \
  --tags="scope:shared,type:utils" \
  --importPath="@srmp/utils"

# Generate UI component library
nx g @nx/react:lib ui \
  --directory=packages \
  --component=false \
  --tags="scope:frontend,type:ui" \
  --importPath="@srmp/ui"
```

### 5. Configure Import Paths

```bash
# Update tsconfig.base.json
cat > tsconfig.base.json << 'EOF'
{
  "compileOnSave": false,
  "compilerOptions": {
    "rootDir": ".",
    "sourceMap": true,
    "declaration": false,
    "moduleResolution": "node",
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "importHelpers": true,
    "target": "es2015",
    "module": "esnext",
    "lib": ["es2020", "dom"],
    "skipLibCheck": true,
    "skipDefaultLibCheck": true,
    "baseUrl": ".",
    "paths": {
      "@srmp/types": ["packages/types/src/index.ts"],
      "@srmp/utils": ["packages/utils/src/index.ts"],
      "@srmp/ui": ["packages/ui/src/index.ts"]
    }
  },
  "exclude": ["node_modules", "tmp"]
}
EOF
```

### 6. Set Up pnpm Workspace

```bash
# Create pnpm-workspace.yaml
cat > pnpm-workspace.yaml << 'EOF'
packages:
  - 'apps/*'
  - 'apps/services/*'
  - 'packages/*'
  - 'infrastructure/*'
EOF
```

### 7. Configure Nx Task Orchestration

```bash
# Update nx.json
cat > nx.json << 'EOF'
{
  "$schema": "./node_modules/nx/schemas/nx-schema.json",
  "npmScope": "srmp",
  "tasksRunnerOptions": {
    "default": {
      "runner": "@nx/nx-cloud",
      "options": {
        "cacheableOperations": ["build", "lint", "test", "e2e"],
        "parallel": 3,
        "cacheDirectory": ".nx/cache"
      }
    }
  },
  "targetDefaults": {
    "build": {
      "dependsOn": ["^build"],
      "inputs": ["production", "^production"]
    },
    "test": {
      "inputs": ["default", "^production", "{workspaceRoot}/jest.preset.js"]
    }
  },
  "namedInputs": {
    "default": ["{projectRoot}/**/*", "sharedGlobals"],
    "production": [
      "default",
      "!{projectRoot}/**/?(*.)+(spec|test).[jt]s?(x)?(.snap)",
      "!{projectRoot}/tsconfig.spec.json",
      "!{projectRoot}/jest.config.[jt]s",
      "!{projectRoot}/.eslintrc.json"
    ],
    "sharedGlobals": ["{workspaceRoot}/tsconfig.base.json"]
  },
  "workspaceLayout": {
    "appsDir": "apps",
    "libsDir": "packages"
  },
  "generators": {
    "@nx/react": {
      "application": {
        "babel": true
      }
    }
  }
}
EOF
```

### 8. Install Common Dependencies

```bash
# Backend dependencies
pnpm add -w @nestjs/config @nestjs/jwt @nestjs/passport passport passport-jwt
pnpm add -w @nestjs/typeorm typeorm pg
pnpm add -w @nestjs/swagger swagger-ui-express
pnpm add -w ioredis @nestjs/bull bull
pnpm add -w @elastic/elasticsearch

# Frontend dependencies  
pnpm add -w react-router-dom@^6 @tanstack/react-query zustand
pnpm add -w antd@^5 @ant-design/icons
pnpm add -w axios react-hook-form
pnpm add -w i18next react-i18next

# Development dependencies
pnpm add -wD @types/node @types/react @types/react-dom
pnpm add -wD prettier eslint-config-prettier
pnpm add -wD husky lint-staged
```

### 9. Set Up Git Hooks

```bash
# Initialize Husky
pnpm dlx husky-init && pnpm install

# Add pre-commit hook
cat > .husky/pre-commit << 'EOF'
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

pnpm nx affected:lint --base=HEAD~1
pnpm nx affected:test --base=HEAD~1
EOF

chmod +x .husky/pre-commit
```

### 10. Create Initial Shared Types

```bash
# Create user types
cat > packages/types/src/lib/user.types.ts << 'EOF'
export interface User {
  id: string;
  email: string;
  name: string;
  nameEn?: string;
  studentId?: string;
  roles: Role[];
  department: string;
  language: 'zh-CN' | 'en';
  createdAt: Date;
  updatedAt: Date;
}

export interface Role {
  id: string;
  name: 'student' | 'professor' | 'secretary' | 'admin';
  permissions: Permission[];
}

export interface Permission {
  id: string;
  resource: string;
  action: string;
}
EOF

# Create index file
cat > packages/types/src/index.ts << 'EOF'
export * from './lib/user.types';
export * from './lib/project.types';
export * from './lib/api.types';
EOF
```

## Verification Commands

```bash
# Verify workspace structure
nx graph

# Test builds
nx run-many --target=build --all

# Run tests
nx run-many --target=test --all

# Check affected projects
nx affected:graph --base=main

# Format code
nx format:write

# Run specific service
nx serve api-gateway
nx serve web

# Build for production
nx build web --prod
nx build api-gateway --prod

# Generate dependency graph
nx dep-graph
```

## Useful Nx Commands for Development

```bash
# Generate a new component in UI library
nx g @nx/react:component Button \
  --project=ui \
  --directory=components \
  --export

# Generate a new service in a NestJS app
nx g @nx/nest:service auth \
  --project=services-auth \
  --directory=services

# Generate a new module
nx g @nx/nest:module users \
  --project=services-user

# Run only affected tests
nx affected:test --base=origin/main

# Build only affected projects
nx affected:build --base=origin/main

# See what's affected by your changes
nx affected:apps --base=origin/main
nx affected:libs --base=origin/main

# Update dependencies
nx migrate latest
nx migrate --run-migrations

# Clean cache
nx reset
```

## Environment Setup

```bash
# Create .env files
touch apps/web/.env.local
touch apps/api-gateway/.env

# Web environment
cat > apps/web/.env.local << 'EOF'
VITE_API_BASE_URL=http://localhost:3000/api
VITE_WEBSOCKET_URL=ws://localhost:3000
EOF

# API Gateway environment
cat > apps/api-gateway/.env << 'EOF'
NODE_ENV=development
PORT=3000
DATABASE_URL=postgresql://postgres:password@localhost:5432/srmp
REDIS_URL=redis://localhost:6379
JWT_SECRET=your-super-secret-jwt-key
EOF
```

## Docker Compose for Local Development

```bash
# Create docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: srmp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
    volumes:
      - es_data:/usr/share/elasticsearch/data

volumes:
  postgres_data:
  redis_data:
  es_data:
EOF

# Start services
docker-compose up -d
```

## Next Steps After Setup

1. **Configure ESLint and Prettier**
   ```bash
   nx g @nx/workspace:prettier-config
   nx g @nx/eslint:config
   ```

2. **Set up API documentation**
   ```bash
   # In each service
   nx g @nx/nest:swagger-config
   ```

3. **Configure testing**
   ```bash
   # Set up E2E tests
   nx g @nx/playwright:configuration --project=web-e2e
   ```

4. **Add CI/CD configuration**
   - Copy the GitHub Actions workflows from the infrastructure section
   - Configure secrets in GitHub repository

5. **Start development**
   ```bash
   # Terminal 1: Start all services
   nx run-many --target=serve --projects=api-gateway,services-auth,services-user
   
   # Terminal 2: Start frontend
   nx serve web
   ```