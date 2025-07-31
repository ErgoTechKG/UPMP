# Frontend Architecture

## Component Architecture

### Component Organization
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

### Component Template
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

## State Management Architecture

### State Structure
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

### State Management Patterns
- Use Zustand for client-only state (UI, user preferences)
- Use React Query for server state (data fetching, caching, synchronization)
- Implement optimistic updates for better UX
- Use React Context sparingly for truly global concerns
- Prefer composition over prop drilling

## Routing Architecture

### Route Organization
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

### Protected Route Pattern
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

## Frontend Services Layer

### API Client Setup
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

### Service Example
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
