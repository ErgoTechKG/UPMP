# Data Models

## User

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

### TypeScript Interface
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

### Relationships
- Has many Projects (as student or professor)
- Has many Applications
- Has many Notifications
- Has many ActivityLogs

## Project

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

### TypeScript Interface
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

### Relationships
- Belongs to Professor (User)
- Has many Applications
- Has many ProjectStudents
- Has many Milestones
- Has many Discussions

## Application

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

### TypeScript Interface
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

### Relationships
- Belongs to Project
- Belongs to Student (User)
- Has many ApplicationStatusLogs

## Milestone

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

### TypeScript Interface
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

### Relationships
- Belongs to Project
- Has many Submissions
- Has one MilestoneTemplate

## TrainingProgress

**Purpose:** Track student progress through 12-step research methodology training

**Key Attributes:**
- id: UUID - Unique identifier
- studentId: UUID - Student taking training
- moduleId: UUID - Current training module
- progress: number - Completion percentage (0-100)
- quizScores: object - Scores for each module quiz
- certificateId: string (nullable) - Completion certificate ID
- completedAt: Date (nullable) - Completion timestamp

### TypeScript Interface
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

### Relationships
- Belongs to Student (User)
- Has many ModuleAttempts
- Has one Certificate
