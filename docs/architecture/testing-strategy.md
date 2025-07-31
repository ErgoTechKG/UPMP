# Testing Strategy

## Testing Pyramid
```text
         E2E Tests
        /        \
   Integration Tests
      /            \
 Frontend Unit  Backend Unit
```

## Test Organization

### Frontend Tests
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

### Backend Tests
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

### E2E Tests
```text
tests/e2e/
├── fixtures/
├── page-objects/
└── scenarios/
    ├── student-journey.spec.ts
    ├── professor-workflow.spec.ts
    └── secretary-operations.spec.ts
```

## Test Examples

### Frontend Component Test
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

### Backend API Test
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

### E2E Test
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
