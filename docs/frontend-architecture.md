# Scientific Research Management Platform (SRMP) Frontend Architecture Document

## Introduction

This document defines the frontend architecture for the Scientific Research Management Platform (SRMP), providing detailed implementation guidelines for the React-based client application. It serves as the authoritative reference for AI-assisted development tools and frontend developers, ensuring consistency with the main architecture document while providing frontend-specific implementation details.

### Change Log

| Date | Version | Description | Author |
|------|---------|-------------|--------|
| 2025-07-30 | 1.0 | Initial frontend architecture based on main architecture and UI/UX spec | Winston (Architect) |

## Template and Framework Selection

After reviewing the main architecture document and PRD, the following decisions have been made:

**Framework Decision:** Custom React setup (no starter template)
- The main architecture specifies React 18.2+ with TypeScript
- Monorepo structure using Nx provides build tooling
- Vite specified as the bundler for optimal development experience

**Rationale:** The project requires specific integrations (WeChat Work, Alibaba Cloud services) and performance optimizations for the Chinese market that are better served by a custom setup rather than a generic starter template.

## Frontend Tech Stack

This section is synchronized with the main architecture document's Technology Stack Table.

### Technology Stack Table

| Category | Technology | Version | Purpose | Rationale |
|----------|------------|---------|---------|-----------|
| Framework | React | 18.2+ | UI library | Large ecosystem, excellent China community support |
| UI Library | Ant Design | 5.12+ | Pre-built components | Designed for Chinese users, enterprise-ready |
| State Management | Zustand + React Query | 4.5+ / 5.x | Client and server state | Lightweight, excellent DX, built-in caching |
| Routing | React Router | 6.20+ | Client-side routing | De facto standard, type-safe with v6 |
| Build Tool | Vite | 5.x | Frontend bundling | Fast HMR, optimized builds, ESM support |
| Styling | Tailwind CSS + CSS Modules | 3.4+ | Utility CSS + component styles | Rapid development + encapsulation |
| Testing | Vitest + React Testing Library | 1.x / 14.x | Unit/integration tests | Fast, ESM support, good React integration |
| Component Library | Custom + Ant Design | - | Reusable components | Consistent with design system |
| Form Handling | React Hook Form | 7.47+ | Form state management | Performance, validation, TypeScript support |
| Animation | Framer Motion | 10.16+ | Animations and gestures | Declarative API, performance optimized |
| Dev Tools | React DevTools + Zustand DevTools | Latest | Development debugging | Essential for state inspection |

## Project Structure

```plaintext
apps/web/
├── src/
│   ├── app/                      # Application core
│   │   ├── App.tsx              # Root component
│   │   ├── App.css              # Global styles
│   │   └── providers.tsx        # Context providers wrapper
│   ├── components/              # Reusable components
│   │   ├── common/              # Shared across all roles
│   │   │   ├── Layout/
│   │   │   │   ├── Layout.tsx
│   │   │   │   ├── Layout.module.css
│   │   │   │   ├── Layout.test.tsx
│   │   │   │   └── index.ts
│   │   │   ├── Navigation/
│   │   │   ├── DataTable/
│   │   │   ├── StatusBadge/
│   │   │   ├── ProgressRing/
│   │   │   ├── ActionCard/
│   │   │   └── LoadingStates/
│   │   ├── student/             # Student-specific components
│   │   │   ├── Dashboard/
│   │   │   ├── ProjectCard/
│   │   │   ├── MilestoneTracker/
│   │   │   └── ApplicationForm/
│   │   ├── professor/           # Professor-specific components
│   │   │   ├── ApplicationReview/
│   │   │   ├── ProjectForm/
│   │   │   └── StudentComparison/
│   │   └── secretary/           # Secretary-specific components
│   │       ├── BatchOperations/
│   │       └── AtRiskMonitor/
│   ├── features/                # Feature modules
│   │   ├── auth/                # Authentication feature
│   │   │   ├── components/
│   │   │   ├── hooks/
│   │   │   ├── services/
│   │   │   └── types/
│   │   ├── projects/            # Projects feature
│   │   ├── training/            # Training feature
│   │   └── analytics/           # Analytics feature
│   ├── hooks/                   # Custom React hooks
│   │   ├── useAuth.ts
│   │   ├── usePermissions.ts
│   │   ├── useErrorHandler.ts
│   │   └── useDebounce.ts
│   ├── layouts/                 # Layout components
│   │   ├── DashboardLayout/
│   │   ├── AuthLayout/
│   │   └── MinimalLayout/
│   ├── pages/                   # Route components
│   │   ├── student/
│   │   │   ├── Dashboard.tsx
│   │   │   ├── Projects.tsx
│   │   │   ├── Applications.tsx
│   │   │   └── Training.tsx
│   │   ├── professor/
│   │   ├── secretary/
│   │   └── common/
│   │       ├── Login.tsx
│   │       ├── NotFound.tsx
│   │       └── Forbidden.tsx
│   ├── routes/                  # Route configuration
│   │   ├── index.tsx            # Root router
│   │   ├── ProtectedRoute.tsx
│   │   ├── student.routes.tsx
│   │   ├── professor.routes.tsx
│   │   └── secretary.routes.tsx
│   ├── services/                # API integration layer
│   │   ├── api/
│   │   │   ├── client.ts        # Axios instance
│   │   │   ├── endpoints.ts     # API endpoints
│   │   │   └── interceptors.ts
│   │   ├── auth.service.ts
│   │   ├── projects.service.ts
│   │   ├── users.service.ts
│   │   └── notifications.service.ts
│   ├── stores/                  # Zustand stores
│   │   ├── auth.store.ts
│   │   ├── ui.store.ts
│   │   └── preferences.store.ts
│   ├── styles/                  # Global styles
│   │   ├── index.css            # Global CSS
│   │   ├── variables.css        # CSS custom properties
│   │   └── tailwind.css         # Tailwind imports
│   ├── types/                   # TypeScript types
│   │   ├── api.types.ts         # API response types
│   │   ├── models.types.ts      # Domain models
│   │   └── utils.types.ts       # Utility types
│   ├── utils/                   # Utility functions
│   │   ├── constants.ts
│   │   ├── formatters.ts
│   │   ├── validators.ts
│   │   └── permissions.ts
│   ├── i18n/                    # Internationalization
│   │   ├── locales/
│   │   │   ├── zh-CN/
│   │   │   └── en/
│   │   └── index.ts
│   ├── main.tsx                 # Application entry
│   └── vite-env.d.ts           # Vite types
├── public/                      # Static assets
│   ├── favicon.ico
│   └── locales/                 # Public translation files
├── tests/                       # Test utilities
│   ├── setup.ts
│   ├── utils.tsx
│   └── mocks/
├── index.html                   # HTML entry
├── vite.config.ts              # Vite configuration
├── tsconfig.json               # TypeScript config
├── tailwind.config.js          # Tailwind config
└── package.json
```

## Component Standards

### Component Template

```typescript
// components/common/StatusBadge/StatusBadge.tsx
import React, { memo } from 'react';
import { Badge, BadgeProps } from 'antd';
import { useTranslation } from 'react-i18next';
import clsx from 'clsx';
import styles from './StatusBadge.module.css';

export interface StatusBadgeProps {
  status: 'on-track' | 'at-risk' | 'delayed' | 'completed';
  size?: 'small' | 'default' | 'large';
  showText?: boolean;
  className?: string;
  onClick?: () => void;
}

const statusConfig: Record<string, { status: BadgeProps['status']; color: string }> = {
  'on-track': { status: 'success', color: '#52c41a' },
  'at-risk': { status: 'warning', color: '#faad14' },
  'delayed': { status: 'error', color: '#f5222d' },
  'completed': { status: 'default', color: '#d9d9d9' }
};

export const StatusBadge = memo<StatusBadgeProps>(({ 
  status, 
  size = 'default',
  showText = true,
  className,
  onClick
}) => {
  const { t } = useTranslation();
  const config = statusConfig[status];
  
  return (
    <div 
      className={clsx(
        styles.container,
        styles[`size-${size}`],
        { [styles.clickable]: onClick },
        className
      )}
      onClick={onClick}
      role={onClick ? 'button' : undefined}
      tabIndex={onClick ? 0 : undefined}
      onKeyDown={(e) => {
        if (onClick && (e.key === 'Enter' || e.key === ' ')) {
          e.preventDefault();
          onClick();
        }
      }}
    >
      <Badge 
        status={config.status}
        color={config.color}
        text={showText ? t(`status.${status}`) : undefined}
      />
    </div>
  );
});

StatusBadge.displayName = 'StatusBadge';

// components/common/StatusBadge/index.ts
export { StatusBadge } from './StatusBadge';
export type { StatusBadgeProps } from './StatusBadge';
```

### Naming Conventions

- **Components**: PascalCase (e.g., `ProjectCard`, `MilestoneTracker`)
- **Component Files**: PascalCase with .tsx extension (e.g., `ProjectCard.tsx`)
- **Hooks**: camelCase with 'use' prefix (e.g., `useAuth`, `useProjectData`)
- **Services**: camelCase with .service.ts suffix (e.g., `auth.service.ts`)
- **Stores**: camelCase with .store.ts suffix (e.g., `auth.store.ts`)
- **Types**: PascalCase for interfaces/types (e.g., `User`, `ProjectStatus`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `API_BASE_URL`, `MAX_FILE_SIZE`)
- **CSS Modules**: camelCase for classes (e.g., `.container`, `.isActive`)
- **Test Files**: Same name with .test.tsx suffix (e.g., `ProjectCard.test.tsx`)

## State Management

### Store Structure

```plaintext
stores/
├── auth.store.ts          # Authentication state
├── ui.store.ts            # UI preferences and state
├── preferences.store.ts   # User preferences
└── types/                 # Store TypeScript types
    ├── auth.types.ts
    └── ui.types.ts
```

### State Management Template

```typescript
// stores/auth.store.ts
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';
import type { User, Role } from '@/types/models.types';

interface AuthState {
  // State
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  error: string | null;
  
  // Actions
  setUser: (user: User) => void;
  updateUser: (updates: Partial<User>) => void;
  logout: () => void;
  setLoading: (isLoading: boolean) => void;
  setError: (error: string | null) => void;
  
  // Computed
  hasRole: (role: Role) => boolean;
  hasPermission: (permission: string) => boolean;
}

export const useAuthStore = create<AuthState>()(
  devtools(
    persist(
      immer((set, get) => ({
        // Initial state
        user: null,
        isAuthenticated: false,
        isLoading: false,
        error: null,
        
        // Actions
        setUser: (user) => set((state) => {
          state.user = user;
          state.isAuthenticated = true;
          state.error = null;
        }),
        
        updateUser: (updates) => set((state) => {
          if (state.user) {
            Object.assign(state.user, updates);
          }
        }),
        
        logout: () => set((state) => {
          state.user = null;
          state.isAuthenticated = false;
          state.error = null;
        }),
        
        setLoading: (isLoading) => set((state) => {
          state.isLoading = isLoading;
        }),
        
        setError: (error) => set((state) => {
          state.error = error;
          state.isLoading = false;
        }),
        
        // Computed
        hasRole: (role) => {
          const { user } = get();
          return user?.roles.some(r => r.name === role) ?? false;
        },
        
        hasPermission: (permission) => {
          const { user } = get();
          return user?.roles.some(role => 
            role.permissions.some(p => p === permission)
          ) ?? false;
        }
      })),
      {
        name: 'auth-storage',
        partialize: (state) => ({ 
          user: state.user,
          isAuthenticated: state.isAuthenticated 
        })
      }
    ),
    { name: 'AuthStore' }
  )
);

// Selector hooks for performance
export const useUser = () => useAuthStore((state) => state.user);
export const useIsAuthenticated = () => useAuthStore((state) => state.isAuthenticated);
export const useHasRole = (role: Role) => useAuthStore((state) => state.hasRole(role));
```

## API Integration

### Service Template

```typescript
// services/projects.service.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { apiClient } from './api/client';
import type { 
  Project, 
  ProjectFilters, 
  CreateProjectDTO, 
  UpdateProjectDTO,
  PaginatedResponse 
} from '@/types/api.types';

// API functions
const projectsApi = {
  list: async (filters: ProjectFilters): Promise<PaginatedResponse<Project>> => {
    const { data } = await apiClient.get('/projects', { params: filters });
    return data;
  },
  
  getById: async (id: string): Promise<Project> => {
    const { data } = await apiClient.get(`/projects/${id}`);
    return data;
  },
  
  create: async (project: CreateProjectDTO): Promise<Project> => {
    const { data } = await apiClient.post('/projects', project);
    return data;
  },
  
  update: async (id: string, updates: UpdateProjectDTO): Promise<Project> => {
    const { data } = await apiClient.patch(`/projects/${id}`, updates);
    return data;
  },
  
  delete: async (id: string): Promise<void> => {
    await apiClient.delete(`/projects/${id}`);
  }
};

// Query Keys
export const projectKeys = {
  all: ['projects'] as const,
  lists: () => [...projectKeys.all, 'list'] as const,
  list: (filters: ProjectFilters) => [...projectKeys.lists(), filters] as const,
  details: () => [...projectKeys.all, 'detail'] as const,
  detail: (id: string) => [...projectKeys.details(), id] as const,
};

// React Query Hooks
export const useProjects = (filters: ProjectFilters) => {
  return useQuery({
    queryKey: projectKeys.list(filters),
    queryFn: () => projectsApi.list(filters),
    staleTime: 5 * 60 * 1000, // 5 minutes
    gcTime: 10 * 60 * 1000, // 10 minutes (formerly cacheTime)
  });
};

export const useProject = (id: string, enabled = true) => {
  return useQuery({
    queryKey: projectKeys.detail(id),
    queryFn: () => projectsApi.getById(id),
    enabled: enabled && !!id,
    staleTime: 5 * 60 * 1000,
  });
};

export const useCreateProject = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: projectsApi.create,
    onSuccess: (newProject) => {
      // Invalidate lists
      queryClient.invalidateQueries({ queryKey: projectKeys.lists() });
      
      // Optionally set the new project in cache
      queryClient.setQueryData(
        projectKeys.detail(newProject.id), 
        newProject
      );
    },
  });
};

export const useUpdateProject = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: ({ id, updates }: { id: string; updates: UpdateProjectDTO }) => 
      projectsApi.update(id, updates),
    onSuccess: (updatedProject) => {
      // Update the specific project
      queryClient.setQueryData(
        projectKeys.detail(updatedProject.id), 
        updatedProject
      );
      
      // Invalidate lists to ensure consistency
      queryClient.invalidateQueries({ queryKey: projectKeys.lists() });
    },
  });
};

export const useDeleteProject = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: projectsApi.delete,
    onSuccess: (_, deletedId) => {
      // Remove from cache
      queryClient.removeQueries({ queryKey: projectKeys.detail(deletedId) });
      
      // Invalidate lists
      queryClient.invalidateQueries({ queryKey: projectKeys.lists() });
    },
  });
};
```

### API Client Configuration

```typescript
// services/api/client.ts
import axios, { AxiosError, InternalAxiosRequestConfig } from 'axios';
import { notification } from 'antd';
import { getAuthToken, refreshAuthToken, clearAuth } from '@/utils/auth';
import { useAuthStore } from '@/stores/auth.store';

const API_BASE_URL = import.meta.env.VITE_API_BASE_URL || 'http://localhost:3000/api';
const API_TIMEOUT = 30000; // 30 seconds

// Create axios instance
export const apiClient = axios.create({
  baseURL: API_BASE_URL,
  timeout: API_TIMEOUT,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor
apiClient.interceptors.request.use(
  (config: InternalAxiosRequestConfig) => {
    // Add auth token
    const token = getAuthToken();
    if (token && config.headers) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    
    // Add language header
    const language = localStorage.getItem('language') || 'zh-CN';
    if (config.headers) {
      config.headers['Accept-Language'] = language;
    }
    
    // Add request ID for tracing
    if (config.headers) {
      config.headers['X-Request-ID'] = crypto.randomUUID();
    }
    
    return config;
  },
  (error: AxiosError) => {
    return Promise.reject(error);
  }
);

// Response interceptor
let isRefreshing = false;
let refreshSubscribers: ((token: string) => void)[] = [];

const subscribeTokenRefresh = (cb: (token: string) => void) => {
  refreshSubscribers.push(cb);
};

const onTokenRefreshed = (token: string) => {
  refreshSubscribers.forEach(cb => cb(token));
  refreshSubscribers = [];
};

apiClient.interceptors.response.use(
  (response) => response,
  async (error: AxiosError<{ error: { code: string; message: string } }>) => {
    const originalRequest = error.config as InternalAxiosRequestConfig & { _retry?: boolean };
    
    // Handle network errors
    if (!error.response) {
      notification.error({
        message: '网络错误 Network Error',
        description: '请检查您的网络连接 Please check your network connection',
        duration: 5,
      });
      return Promise.reject(error);
    }
    
    // Handle 401 Unauthorized
    if (error.response.status === 401 && !originalRequest._retry) {
      if (isRefreshing) {
        // Wait for token refresh
        return new Promise((resolve) => {
          subscribeTokenRefresh((token: string) => {
            if (originalRequest.headers) {
              originalRequest.headers.Authorization = `Bearer ${token}`;
            }
            resolve(apiClient(originalRequest));
          });
        });
      }
      
      originalRequest._retry = true;
      isRefreshing = true;
      
      try {
        const newToken = await refreshAuthToken();
        isRefreshing = false;
        onTokenRefreshed(newToken);
        
        if (originalRequest.headers) {
          originalRequest.headers.Authorization = `Bearer ${newToken}`;
        }
        return apiClient(originalRequest);
      } catch (refreshError) {
        isRefreshing = false;
        clearAuth();
        useAuthStore.getState().logout();
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }
    
    // Handle other errors
    if (error.response.data?.error) {
      const { code, message } = error.response.data.error;
      
      // Don't show notification for expected errors (e.g., validation)
      const silentCodes = ['VALIDATION_FAILED', 'DUPLICATE_ENTRY'];
      if (!silentCodes.includes(code)) {
        notification.error({
          message: `错误 Error: ${code}`,
          description: message,
          duration: 5,
        });
      }
    }
    
    return Promise.reject(error);
  }
);

// Helper function for GraphQL requests
export const graphqlClient = async <T = any>(
  query: string, 
  variables?: Record<string, any>
): Promise<T> => {
  const { data } = await apiClient.post('/graphql', {
    query,
    variables,
  });
  
  if (data.errors) {
    throw new Error(data.errors[0].message);
  }
  
  return data.data;
};
```

## Routing

### Route Configuration

```typescript
// routes/index.tsx
import React, { lazy, Suspense } from 'react';
import { createBrowserRouter, Navigate } from 'react-router-dom';
import { LoadingPage } from '@/components/common/LoadingStates';
import { ProtectedRoute } from './ProtectedRoute';
import { DashboardLayout } from '@/layouts/DashboardLayout';
import { AuthLayout } from '@/layouts/AuthLayout';

// Lazy load pages
const Login = lazy(() => import('@/pages/common/Login'));
const NotFound = lazy(() => import('@/pages/common/NotFound'));
const Forbidden = lazy(() => import('@/pages/common/Forbidden'));

// Student pages
const StudentDashboard = lazy(() => import('@/pages/student/Dashboard'));
const StudentProjects = lazy(() => import('@/pages/student/Projects'));
const StudentApplications = lazy(() => import('@/pages/student/Applications'));
const StudentTraining = lazy(() => import('@/pages/student/Training'));

// Professor pages
const ProfessorDashboard = lazy(() => import('@/pages/professor/Dashboard'));
const ProfessorProjects = lazy(() => import('@/pages/professor/Projects'));
const ProfessorApplications = lazy(() => import('@/pages/professor/Applications'));

// Secretary pages
const SecretaryDashboard = lazy(() => import('@/pages/secretary/Dashboard'));
const SecretaryMonitoring = lazy(() => import('@/pages/secretary/Monitoring'));
const SecretaryReports = lazy(() => import('@/pages/secretary/Reports'));

// Route configuration
export const router = createBrowserRouter([
  {
    path: '/',
    element: <Navigate to="/dashboard" replace />,
  },
  {
    path: '/login',
    element: (
      <AuthLayout>
        <Suspense fallback={<LoadingPage />}>
          <Login />
        </Suspense>
      </AuthLayout>
    ),
  },
  {
    path: '/dashboard',
    element: (
      <ProtectedRoute>
        <DashboardLayout />
      </ProtectedRoute>
    ),
    children: [
      {
        index: true,
        element: <RoleBasedDashboard />,
      },
      // Student routes
      {
        path: 'projects',
        element: (
          <ProtectedRoute requiredRoles={['student']}>
            <Suspense fallback={<LoadingPage />}>
              <StudentProjects />
            </Suspense>
          </ProtectedRoute>
        ),
      },
      {
        path: 'applications',
        element: (
          <ProtectedRoute requiredRoles={['student']}>
            <Suspense fallback={<LoadingPage />}>
              <StudentApplications />
            </Suspense>
          </ProtectedRoute>
        ),
      },
      {
        path: 'training',
        element: (
          <ProtectedRoute requiredRoles={['student']}>
            <Suspense fallback={<LoadingPage />}>
              <StudentTraining />
            </Suspense>
          </ProtectedRoute>
        ),
      },
      // Professor routes
      {
        path: 'my-projects',
        element: (
          <ProtectedRoute requiredRoles={['professor']}>
            <Suspense fallback={<LoadingPage />}>
              <ProfessorProjects />
            </Suspense>
          </ProtectedRoute>
        ),
      },
      {
        path: 'review-applications',
        element: (
          <ProtectedRoute requiredRoles={['professor']}>
            <Suspense fallback={<LoadingPage />}>
              <ProfessorApplications />
            </Suspense>
          </ProtectedRoute>
        ),
      },
      // Secretary routes
      {
        path: 'monitoring',
        element: (
          <ProtectedRoute requiredRoles={['secretary']}>
            <Suspense fallback={<LoadingPage />}>
              <SecretaryMonitoring />
            </Suspense>
          </ProtectedRoute>
        ),
      },
      {
        path: 'reports',
        element: (
          <ProtectedRoute requiredRoles={['secretary', 'admin']}>
            <Suspense fallback={<LoadingPage />}>
              <SecretaryReports />
            </Suspense>
          </ProtectedRoute>
        ),
      },
    ],
  },
  {
    path: '/403',
    element: (
      <Suspense fallback={<LoadingPage />}>
        <Forbidden />
      </Suspense>
    ),
  },
  {
    path: '*',
    element: (
      <Suspense fallback={<LoadingPage />}>
        <NotFound />
      </Suspense>
    ),
  },
]);

// Role-based dashboard component
function RoleBasedDashboard() {
  const { user } = useAuthStore();
  
  if (!user) return <Navigate to="/login" replace />;
  
  const primaryRole = user.roles[0]?.name;
  
  switch (primaryRole) {
    case 'student':
      return (
        <Suspense fallback={<LoadingPage />}>
          <StudentDashboard />
        </Suspense>
      );
    case 'professor':
      return (
        <Suspense fallback={<LoadingPage />}>
          <ProfessorDashboard />
        </Suspense>
      );
    case 'secretary':
      return (
        <Suspense fallback={<LoadingPage />}>
          <SecretaryDashboard />
        </Suspense>
      );
    default:
      return <Navigate to="/403" replace />;
  }
}

// Protected Route Component
// routes/ProtectedRoute.tsx
import React from 'react';
import { Navigate, useLocation } from 'react-router-dom';
import { useAuthStore } from '@/stores/auth.store';
import { hasRequiredRoles, hasRequiredPermissions } from '@/utils/permissions';

interface ProtectedRouteProps {
  children: React.ReactNode;
  requiredRoles?: string[];
  requiredPermissions?: string[];
}

export const ProtectedRoute: React.FC<ProtectedRouteProps> = ({
  children,
  requiredRoles = [],
  requiredPermissions = [],
}) => {
  const { isAuthenticated, user } = useAuthStore();
  const location = useLocation();
  
  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  if (!user) {
    return <Navigate to="/login" replace />;
  }
  
  const hasRoles = hasRequiredRoles(user, requiredRoles);
  const hasPermissions = hasRequiredPermissions(user, requiredPermissions);
  
  if (!hasRoles || !hasPermissions) {
    return <Navigate to="/403" replace />;
  }
  
  return <>{children}</>;
};
```

## Styling Guidelines

### Styling Approach

The project uses a hybrid approach combining Tailwind CSS for utility classes and CSS Modules for component-specific styles:

1. **Tailwind CSS**: Used for rapid prototyping, spacing, colors, and responsive utilities
2. **CSS Modules**: Used for component encapsulation and complex component styles
3. **Ant Design Theme**: Customized through CSS variables to match brand colors

### Global Theme Variables

```css
/* styles/variables.css */
:root {
  /* Colors - Primary Palette */
  --color-primary-50: #e6f4ff;
  --color-primary-100: #bae0ff;
  --color-primary-200: #91caff;
  --color-primary-300: #69b1ff;
  --color-primary-400: #4096ff;
  --color-primary-500: #1890ff; /* Primary brand color */
  --color-primary-600: #1677ff;
  --color-primary-700: #0958d9;
  --color-primary-800: #003eb3;
  --color-primary-900: #002c8c;

  /* Colors - Success */
  --color-success-50: #f6ffed;
  --color-success-100: #d9f7be;
  --color-success-200: #b7eb8f;
  --color-success-300: #95de64;
  --color-success-400: #73d13d;
  --color-success-500: #52c41a;
  --color-success-600: #389e0d;
  --color-success-700: #237804;
  --color-success-800: #135200;
  --color-success-900: #092b00;

  /* Colors - Warning */
  --color-warning-50: #fffbe6;
  --color-warning-100: #fff1b8;
  --color-warning-200: #ffe58f;
  --color-warning-300: #ffd666;
  --color-warning-400: #ffc53d;
  --color-warning-500: #faad14;
  --color-warning-600: #d48806;
  --color-warning-700: #ad6800;
  --color-warning-800: #874d00;
  --color-warning-900: #613400;

  /* Colors - Error */
  --color-error-50: #fff2f0;
  --color-error-100: #ffccc7;
  --color-error-200: #ffa39e;
  --color-error-300: #ff7875;
  --color-error-400: #ff4d4f;
  --color-error-500: #f5222d;
  --color-error-600: #cf1322;
  --color-error-700: #a8071a;
  --color-error-800: #820014;
  --color-error-900: #5c0011;

  /* Colors - Neutral */
  --color-text-primary: rgba(0, 0, 0, 0.85);
  --color-text-secondary: rgba(0, 0, 0, 0.65);
  --color-text-tertiary: rgba(0, 0, 0, 0.45);
  --color-text-disabled: rgba(0, 0, 0, 0.25);
  --color-border: #d9d9d9;
  --color-background: #f5f5f5;
  --color-background-elevated: #ffffff;

  /* Spacing Scale (4px base) */
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 12px;
  --spacing-lg: 16px;
  --spacing-xl: 24px;
  --spacing-2xl: 32px;
  --spacing-3xl: 48px;
  --spacing-4xl: 64px;

  /* Typography */
  --font-family-primary: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'PingFang SC', 'Microsoft YaHei', sans-serif;
  --font-family-mono: 'SF Mono', Monaco, 'Cascadia Code', Consolas, monospace;
  
  --font-size-xs: 12px;
  --font-size-sm: 14px;
  --font-size-base: 16px;
  --font-size-lg: 18px;
  --font-size-xl: 20px;
  --font-size-2xl: 24px;
  --font-size-3xl: 32px;
  
  --line-height-tight: 1.25;
  --line-height-normal: 1.5;
  --line-height-relaxed: 1.75;

  /* Shadows */
  --shadow-sm: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
  --shadow-base: 0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px 0 rgba(0, 0, 0, 0.06);
  --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
  --shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
  --shadow-xl: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);

  /* Border Radius */
  --radius-sm: 2px;
  --radius-base: 4px;
  --radius-md: 6px;
  --radius-lg: 8px;
  --radius-xl: 12px;
  --radius-full: 9999px;

  /* Transitions */
  --transition-fast: 150ms ease-in-out;
  --transition-base: 200ms ease-in-out;
  --transition-slow: 300ms ease-in-out;

  /* Z-index Scale */
  --z-dropdown: 1000;
  --z-sticky: 1020;
  --z-fixed: 1030;
  --z-modal-backdrop: 1040;
  --z-modal: 1050;
  --z-popover: 1060;
  --z-tooltip: 1070;
}

/* Dark Mode Support */
[data-theme='dark'] {
  --color-text-primary: rgba(255, 255, 255, 0.85);
  --color-text-secondary: rgba(255, 255, 255, 0.65);
  --color-text-tertiary: rgba(255, 255, 255, 0.45);
  --color-text-disabled: rgba(255, 255, 255, 0.25);
  --color-border: #434343;
  --color-background: #141414;
  --color-background-elevated: #1f1f1f;
}

/* Responsive Font Sizes */
@media (max-width: 768px) {
  :root {
    --font-size-base: 14px;
    --font-size-lg: 16px;
    --font-size-xl: 18px;
    --font-size-2xl: 20px;
    --font-size-3xl: 24px;
  }
}
```

## Testing Requirements

### Component Test Template

```typescript
// components/common/StatusBadge/StatusBadge.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { vi } from 'vitest';
import { StatusBadge } from './StatusBadge';

// Mock i18next
vi.mock('react-i18next', () => ({
  useTranslation: () => ({
    t: (key: string) => key,
    i18n: {
      changeLanguage: vi.fn(),
    },
  }),
}));

describe('StatusBadge', () => {
  it('renders with correct status', () => {
    render(<StatusBadge status="on-track" />);
    
    const badge = screen.getByText('status.on-track');
    expect(badge).toBeInTheDocument();
  });
  
  it('applies correct size class', () => {
    const { container } = render(<StatusBadge status="completed" size="small" />);
    
    const badgeContainer = container.firstChild;
    expect(badgeContainer).toHaveClass('size-small');
  });
  
  it('handles click events when onClick provided', () => {
    const handleClick = vi.fn();
    render(<StatusBadge status="at-risk" onClick={handleClick} />);
    
    const badge = screen.getByRole('button');
    fireEvent.click(badge);
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
  
  it('handles keyboard events for accessibility', () => {
    const handleClick = vi.fn();
    render(<StatusBadge status="delayed" onClick={handleClick} />);
    
    const badge = screen.getByRole('button');
    
    // Test Enter key
    fireEvent.keyDown(badge, { key: 'Enter' });
    expect(handleClick).toHaveBeenCalledTimes(1);
    
    // Test Space key
    fireEvent.keyDown(badge, { key: ' ' });
    expect(handleClick).toHaveBeenCalledTimes(2);
  });
  
  it('does not show text when showText is false', () => {
    render(<StatusBadge status="on-track" showText={false} />);
    
    const text = screen.queryByText('status.on-track');
    expect(text).not.toBeInTheDocument();
  });
  
  it('applies custom className', () => {
    const { container } = render(
      <StatusBadge status="completed" className="custom-class" />
    );
    
    const badgeContainer = container.firstChild;
    expect(badgeContainer).toHaveClass('custom-class');
  });
});

// Integration test example
describe('StatusBadge Integration', () => {
  it('works with Ant Design Badge component', () => {
    const { container } = render(<StatusBadge status="on-track" />);
    
    // Check that Ant Badge is rendered
    const antBadge = container.querySelector('.ant-badge-status');
    expect(antBadge).toBeInTheDocument();
    expect(antBadge).toHaveClass('ant-badge-status-success');
  });
});
```

### Testing Best Practices

1. **Unit Tests**: Test individual components in isolation
   - Test all props and their combinations
   - Test user interactions (clicks, keyboard events)
   - Test conditional rendering
   - Mock external dependencies

2. **Integration Tests**: Test component interactions
   - Test data flow between components
   - Test API integration with MSW (Mock Service Worker)
   - Test routing behavior
   - Test state management integration

3. **E2E Tests**: Test critical user flows (using Cypress/Playwright)
   - Student project application flow
   - Professor application review flow
   - Milestone submission and review
   - Authentication flow

4. **Coverage Goals**: Aim for 80% code coverage
   - 100% coverage for utility functions
   - 90%+ for business logic
   - 80%+ for components
   - Focus on critical paths

5. **Test Structure**: Arrange-Act-Assert pattern
   - Arrange: Set up test data and component
   - Act: Perform user actions
   - Assert: Verify expected outcomes

6. **Mock External Dependencies**: API calls, routing, state management
   - Use MSW for API mocking
   - Use memory router for routing tests
   - Provide test wrappers for context/stores

## Environment Configuration

```bash
# .env.local (for local development)
# API Configuration
VITE_API_BASE_URL=http://localhost:3000/api
VITE_GRAPHQL_URL=http://localhost:3000/graphql
VITE_WEBSOCKET_URL=ws://localhost:3000

# Authentication
VITE_AUTH_TOKEN_KEY=srmp_auth_token
VITE_AUTH_REFRESH_KEY=srmp_refresh_token
VITE_SESSION_TIMEOUT=1800000 # 30 minutes in ms

# WeChat Work Integration
VITE_WECHAT_CORP_ID=your_corp_id
VITE_WECHAT_AGENT_ID=your_agent_id
VITE_WECHAT_REDIRECT_URI=http://localhost:5173/auth/wechat/callback

# Alibaba Cloud Services
VITE_OSS_BUCKET=srmp-files-dev
VITE_OSS_REGION=oss-cn-beijing
VITE_CDN_URL=https://cdn-dev.srmp.edu.cn

# Feature Flags
VITE_FEATURE_AI_MATCHING=true
VITE_FEATURE_WECHAT_LOGIN=true
VITE_FEATURE_ANALYTICS=true
VITE_FEATURE_DARK_MODE=false

# Monitoring
VITE_SENTRY_DSN=https://your-sentry-dsn@sentry.io/project-id
VITE_SENTRY_ENVIRONMENT=development
VITE_GA_TRACKING_ID=UA-XXXXXXXXX-X

# Development Tools
VITE_ENABLE_DEV_TOOLS=true
VITE_MOCK_API=false
VITE_API_DELAY=0 # Simulate network latency in ms

# Build Configuration
VITE_BUILD_SOURCEMAP=true
VITE_BUNDLE_ANALYZER=false
```

## Frontend Developer Standards

### Critical Coding Rules

1. **Never access process.env directly** - Use import.meta.env for Vite
   ```typescript
   // ❌ Wrong
   const apiUrl = process.env.REACT_APP_API_URL;
   
   // ✅ Correct
   const apiUrl = import.meta.env.VITE_API_BASE_URL;
   ```

2. **Always use TypeScript strict mode** - No `any` types without explicit justification
   ```typescript
   // ❌ Wrong
   const handleData = (data: any) => { ... }
   
   // ✅ Correct
   const handleData = (data: ProjectData) => { ... }
   ```

3. **Never mutate state directly** - Use immutable updates
   ```typescript
   // ❌ Wrong
   state.user.name = 'New Name';
   
   // ✅ Correct (with Immer in Zustand)
   set((state) => {
     state.user.name = 'New Name';
   });
   ```

4. **Always handle loading and error states** - No assuming happy path
   ```typescript
   // ✅ Correct
   if (isLoading) return <LoadingSpinner />;
   if (error) return <ErrorMessage error={error} />;
   if (!data) return <EmptyState />;
   ```

5. **Use semantic HTML** - Accessibility is not optional
   ```typescript
   // ❌ Wrong
   <div onClick={handleClick}>Click me</div>
   
   // ✅ Correct
   <button onClick={handleClick}>Click me</button>
   ```

6. **Validate all user inputs** - Client and server side
   ```typescript
   // ✅ Correct
   const schema = z.object({
     email: z.string().email(),
     age: z.number().min(18).max(100)
   });
   ```

7. **Use React Query for all API calls** - No direct fetch/axios in components
   ```typescript
   // ❌ Wrong
   useEffect(() => {
     fetch('/api/projects').then(...)
   }, []);
   
   // ✅ Correct
   const { data, error, isLoading } = useProjects(filters);
   ```

8. **Memoize expensive computations** - But don't over-optimize
   ```typescript
   // ✅ Correct
   const expensiveValue = useMemo(
     () => calculateComplexValue(data),
     [data]
   );
   ```

9. **Use error boundaries** - Prevent entire app crashes
   ```typescript
   // ✅ Correct
   <ErrorBoundary fallback={<ErrorFallback />}>
     <RiskyComponent />
   </ErrorBoundary>
   ```

10. **Follow the file naming convention** - Consistency matters
    ```
    components/ProjectCard/ProjectCard.tsx (not projectCard.tsx)
    hooks/useAuth.ts (not UseAuth.ts)
    utils/formatters.ts (not Formatters.ts)
    ```

### Quick Reference

**Common Commands:**
```bash
# Development
pnpm nx serve web              # Start dev server
pnpm nx build web              # Build for production
pnpm nx test web               # Run tests
pnpm nx lint web               # Run linter
pnpm nx preview web            # Preview production build

# Code Generation (if using Nx generators)
nx g @nrwl/react:component MyComponent --project=web
nx g @nrwl/react:hook useMyHook --project=web
```

**Key Import Patterns:**
```typescript
// Absolute imports (configured in tsconfig)
import { Button } from '@/components/common/Button';
import { useAuth } from '@/hooks/useAuth';
import { apiClient } from '@/services/api/client';
import type { User } from '@/types/models.types';

// Package imports
import React, { useState, useEffect } from 'react';
import { useQuery } from '@tanstack/react-query';
import { Button, Form, Input } from 'antd';
```

**File Naming Conventions:**
- Components: `PascalCase.tsx` (e.g., `ProjectCard.tsx`)
- Hooks: `camelCase.ts` with 'use' prefix (e.g., `useAuth.ts`)
- Services: `camelCase.service.ts` (e.g., `projects.service.ts`)
- Types: `camelCase.types.ts` (e.g., `api.types.ts`)
- Utils: `camelCase.ts` (e.g., `formatters.ts`)
- Tests: `[name].test.tsx` or `[name].test.ts`

**Project-Specific Patterns:**
```typescript
// Role-based rendering
{hasRole('professor') && <ProfessorOnlyComponent />}

// Permission-based rendering
{hasPermission('projects.create') && <CreateProjectButton />}

// Bilingual text
const { t } = useTranslation();
<h1>{t('projects.title')}</h1>

// Status-based styling
<div className={clsx(
  'project-card',
  `status-${project.status}`
)}>
```

## Performance Optimization Strategies

### Bundle Optimization
- Route-based code splitting with React.lazy()
- Component lazy loading for heavy components
- Tree shaking with Vite's Rollup
- Dynamic imports for large libraries

### Runtime Performance
- React Query for intelligent caching
- Virtual scrolling for long lists
- Image lazy loading with Intersection Observer
- Debounced search inputs
- Memoized selectors in Zustand

### Asset Optimization
- WebP images with fallbacks
- Responsive image sizes
- Font subsetting for Chinese characters
- CDN for static assets

### Monitoring
- Web Vitals tracking (LCP, FID, CLS)
- React DevTools Profiler in development
- Sentry performance monitoring in production
- Bundle size analysis with rollup-plugin-visualizer

## Integration with Backend Services

### WebSocket Integration
```typescript
// services/websocket.service.ts
import { io, Socket } from 'socket.io-client';
import { getAuthToken } from '@/utils/auth';

class WebSocketService {
  private socket: Socket | null = null;
  
  connect() {
    if (this.socket?.connected) return;
    
    this.socket = io(import.meta.env.VITE_WEBSOCKET_URL, {
      auth: {
        token: getAuthToken(),
      },
      transports: ['websocket'],
      reconnection: true,
      reconnectionDelay: 1000,
      reconnectionAttempts: 5,
    });
    
    this.setupEventHandlers();
  }
  
  private setupEventHandlers() {
    if (!this.socket) return;
    
    this.socket.on('connect', () => {
      console.log('WebSocket connected');
    });
    
    this.socket.on('notification', (data) => {
      // Handle real-time notifications
    });
    
    this.socket.on('project-update', (data) => {
      // Invalidate React Query cache
    });
  }
  
  disconnect() {
    this.socket?.disconnect();
    this.socket = null;
  }
}

export const wsService = new WebSocketService();
```

### GraphQL Integration
```typescript
// Example GraphQL query with proper typing
const GET_DASHBOARD_DATA = `
  query GetDashboardData($userId: UUID!) {
    studentDashboard(userId: $userId) {
      profile {
        id
        name
        email
      }
      activeProjects {
        id
        title
        currentMilestone {
          name
          dueDate
        }
        overallProgress
      }
      upcomingDeadlines {
        id
        title
        dueDate
        type
      }
    }
  }
`;

export const useDashboardData = (userId: string) => {
  return useQuery({
    queryKey: ['dashboard', userId],
    queryFn: () => graphqlClient<DashboardData>(GET_DASHBOARD_DATA, { userId }),
  });
};
```

## Security Considerations

### Frontend Security Practices
1. **XSS Prevention**: React's automatic escaping + DOMPurify for user content
2. **CSRF Protection**: Token included in all API requests
3. **Secure Storage**: Sensitive tokens in httpOnly cookies
4. **Content Security Policy**: Strict CSP headers
5. **Input Validation**: Client-side validation as first defense
6. **Dependency Scanning**: Regular npm audit
7. **Source Maps**: Disabled in production builds

## Migration and Upgrade Strategy

### Version Management
- Pin exact versions in package.json for consistency
- Test all updates in staging environment first
- Use npm-check-updates for systematic upgrades
- Maintain upgrade log with breaking changes

### Breaking Change Handling
1. Review changelog before any major version upgrade
2. Run full test suite after updates
3. Update TypeScript types if needed
4. Document any API changes for team

## Conclusion

This frontend architecture document provides comprehensive guidelines for building the SRMP frontend application. It ensures consistency with the main architecture while providing specific implementation details for frontend developers and AI tools. Regular updates to this document should be made as the application evolves and new patterns emerge.

Key principles to remember:
- **Type Safety First**: Leverage TypeScript throughout
- **Performance Matters**: Optimize for Chinese users with slower connections
- **Accessibility is Required**: WCAG AA compliance is non-negotiable
- **Developer Experience**: Clear patterns and good tooling improve productivity
- **Maintainability**: Consistent structure and naming ease long-term maintenance