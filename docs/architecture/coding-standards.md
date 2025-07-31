# Coding Standards

## Critical Fullstack Rules

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

## Naming Conventions

| Element | Frontend | Backend | Example |
|---------|----------|---------|---------|
| Components | PascalCase | - | `UserProfile.tsx` |
| Hooks | camelCase with 'use' | - | `useAuth.ts` |
| API Routes | - | kebab-case | `/api/user-profile` |
| Database Tables | - | snake_case | `user_profiles` |
| Constants | UPPER_SNAKE_CASE | UPPER_SNAKE_CASE | `MAX_FILE_SIZE` |
| Types/Interfaces | PascalCase | PascalCase | `UserProfile` |
| File Names | PascalCase/camelCase | camelCase | `UserProfile.tsx`, `auth.service.ts` |
