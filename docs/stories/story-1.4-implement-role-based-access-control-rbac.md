# Story 1.4: Implement Role-Based Access Control (RBAC)

As a system administrator,
I want to assign and manage user roles with specific permissions,
so that users only access features appropriate to their position.

### Acceptance Criteria
1: Role hierarchy implemented: Student, Professor (Advisor/Instructor/Admin), Secretary, Admin
2: Permission system allows granular access control to API endpoints
3: Role assignment API with audit logging of all changes
4: Middleware validates permissions on every protected route
5: Role context switching enabled for users with multiple roles
6: Default permissions configured for each role based on requirements
7: Admin interface displays current user permissions for debugging
