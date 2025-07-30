# AI UI Generation Prompts for SRMP

## Overview

These prompts are designed for use with AI-driven frontend development tools like Vercel v0, Lovable.ai, or similar platforms. Each prompt follows a structured framework to ensure high-quality, production-ready UI components.

## Tech Stack Context (Include in Every Prompt)

```
Project: Scientific Research Management Platform (SRMP)
Tech Stack: React 18 with TypeScript, Ant Design (antd) 5.x, Tailwind CSS for custom styling
State Management: Zustand or Redux Toolkit
Internationalization: react-i18next (Chinese primary, English secondary)
Build Tool: Vite or Next.js
Key Features: Role-based access (Student, Professor, Secretary, Admin), Bilingual support, Mobile-responsive
Design Principles: Three-step operation, Academic professionalism, WCAG AA compliant
```

---

## Prompt 1: Student Dashboard

### High-Level Goal
Create a responsive, role-optimized dashboard for student researchers that provides at-a-glance visibility of all research activities, deadlines, and progress metrics following the three-step operation principle.

### Detailed Instructions

```
1. Create a new file named `StudentDashboard.tsx` using React with TypeScript
2. Import and use Ant Design components: Card, Progress, Badge, Timeline, Row, Col, Statistic
3. Implement a responsive grid layout:
   - Mobile (320-767px): Single column, stacked cards
   - Tablet (768-1023px): 2 columns for main content
   - Desktop (1024px+): 3-column layout with sidebar

4. Create these specific sections in order:
   a. Welcome Banner:
      - Display student name with program badge (启明班/创新班)
      - Show current semester and academic year
      - Language toggle button (中文/EN) in top right

   b. Action Cards Row (3 cards):
      - "Pending Applications" with count badge
      - "Upcoming Deadlines" with nearest date
      - "New Matches" with AI match percentage

   c. Active Projects Section:
      - List of current research projects with progress rings
      - Each item shows: Project title, Professor name, Current milestone, Days until deadline
      - Status indicator (green/yellow/red) based on progress

   d. Research Training Progress:
      - Horizontal progress bar showing 12-step methodology completion
      - "Continue Learning" button if incomplete
      - Certificate badge if completed

   e. Recent Notifications Feed:
      - Last 5 notifications with timestamps
      - Icon indicators for type (deadline, feedback, match)
      - "View All" link at bottom

5. Implement these interactions:
   - Hover effects on all cards (slight elevation with shadow)
   - Click any card to navigate to detail view
   - Pull-to-refresh on mobile
   - Skeleton loading states for all data sections

6. Color Scheme:
   - Primary: #1890FF (Ant Design blue)
   - Success: #52C41A
   - Warning: #FAAD14
   - Error: #F5222D
   - Background: #F0F2F5

7. Ensure all text has Chinese translations using i18next
8. Add ARIA labels for screen reader accessibility
9. Use Tailwind classes for spacing (p-4, m-2, gap-4) but Ant Design for components
```

### Code Context & Constraints

```typescript
// Example API response structure
interface DashboardData {
  student: {
    name: string;
    program: 'qiming' | 'innovation';
    studentId: string;
  };
  stats: {
    pendingApplications: number;
    activeProjects: number;
    upcomingDeadlines: number;
    newMatches: number;
  };
  projects: Array<{
    id: string;
    title: string;
    professor: string;
    milestone: string;
    progress: number;
    daysUntilDeadline: number;
    status: 'on-track' | 'at-risk' | 'delayed';
  }>;
  notifications: Array<{
    id: string;
    type: 'deadline' | 'feedback' | 'match';
    message: string;
    timestamp: Date;
    read: boolean;
  }>;
}

// DO NOT create any authentication logic
// DO NOT implement API calls - use mock data
// DO NOT modify any routing logic
// ONLY create the dashboard component UI
```

### Strict Scope
- Create only `StudentDashboard.tsx`
- Import types from existing `types/index.ts` (assume it exists)
- Use mock data that matches the interface above
- Focus solely on UI rendering and layout
- Do not create any additional components or files

---

## Prompt 2: Project Discovery & Search Page

### High-Level Goal
Build an intuitive project discovery interface that enables students to browse, search, and filter research opportunities with AI-powered matching scores, following academic UI patterns familiar to Chinese university students.

### Detailed Instructions

```
1. Create `ProjectDiscovery.tsx` with these layout sections:
   
2. Top Search Bar:
   - Large search input with icon
   - Placeholder: "搜索项目、教授或关键词..." / "Search projects, professors, or keywords..."
   - Auto-complete dropdown showing suggestions
   - Search button with loading state

3. Filter Sidebar (collapsible on mobile):
   - Department multi-select with checkbox
   - Commitment level slider (5-20 hours/week)
   - Prerequisites tags (can select multiple)
   - Project type radio buttons (Research/Development/Analysis)
   - "Clear Filters" link at bottom
   - Show active filter count badge

4. Results Section:
   - Sorting dropdown: "Best Match" (default), "Newest", "Deadline Soon", "Most Popular"
   - Results count: "Showing X of Y projects"
   - Project cards in grid:
     * Mobile: 1 column
     * Tablet: 2 columns  
     * Desktop: 3 columns

5. Project Card Design:
   - Professor avatar (left) with name and department
   - Project title (max 2 lines with ellipsis)
   - Brief description (3 lines max)
   - Match score as circular progress (color-coded):
     * 80-100%: Green (#52C41A)
     * 50-79%: Orange (#FAAD14)  
     * <50%: Grey (#D9D9D9)
   - Tags: Department, Type, Commitment
   - Footer: Application deadline + Save bookmark icon
   - Hover: Slight scale transform and shadow

6. Empty States:
   - No results: Friendly illustration with "No projects match your criteria"
   - No saved: "Start exploring projects to build your wishlist"

7. Pagination:
   - Ant Design Pagination component at bottom
   - 12 projects per page
   - Show page size changer on desktop

8. Mobile Optimizations:
   - Sticky filter button that opens drawer
   - Infinite scroll instead of pagination
   - Swipe to bookmark gesture
```

### Code Context & Constraints

```typescript
interface Project {
  id: string;
  title: string;
  titleEn?: string;
  professor: {
    name: string;
    avatar: string;
    department: string;
  };
  description: string;
  descriptionEn?: string;
  matchScore: number; // 0-100
  prerequisites: string[];
  commitmentHours: number;
  type: 'research' | 'development' | 'analysis';
  deadline: Date;
  applicants: number;
  capacity: number;
  saved: boolean;
}

// Use Ant Design components: Input.Search, Select, Slider, Tag, Card, Avatar, Progress
// Use react-query for data fetching simulation (with mock data)
// Implement debounced search (300ms)
// DO NOT implement actual API calls
// DO NOT create backend logic
```

### Strict Scope
- Only create `ProjectDiscovery.tsx`
- Import shared components from `components/common/`
- Use Ant Design's ConfigProvider for theme consistency
- Mock 20-30 diverse projects for demonstration

---

## Prompt 3: Student-Professor Matching Flow

### High-Level Goal
Design a streamlined application flow that guides students through applying to research projects in exactly three steps: Select → Apply → Confirm, with clear progress indication and error prevention.

### Detailed Instructions

```
1. Create `ProjectApplication.tsx` with a three-step wizard:

2. Step 1 - Review & Prepare:
   - Project details summary card
   - Prerequisites checklist (checkbox list)
   - Required documents reminder
   - "I'm Ready to Apply" button (disabled until all checked)

3. Step 2 - Submit Application:
   - Split layout:
     * Left: Project summary (collapsed)
     * Right: Application form
   - Form fields:
     * CV upload (drag-drop zone)
       - Accept: PDF, DOC, DOCX (max 5MB)
       - Show file preview after upload
     * Personal statement textarea
       - Character counter (500-1000)
       - Real-time validation
       - Rich text toolbar (bold, italic, lists)
     * Previous experience dropdown (optional)
   - Auto-save indicator every 30 seconds
   - "Preview Application" button
   - "Submit Application" primary button

4. Step 3 - Confirmation:
   - Success animation (Ant Design Result component)
   - Application reference number
   - What happens next timeline:
     * Professor review (1-3 days)
     * Possible interview
     * Final decision (within 7 days)
   - "View My Applications" button
   - "Browse More Projects" secondary button

5. Progress Indicator:
   - Ant Design Steps component at top
   - Shows: Review → Apply → Confirm
   - Current step highlighted

6. Error Handling:
   - File too large: Inline error with suggestion
   - Network error: Toast notification with retry
   - Validation errors: Field-level messages
   - Prevent navigation with unsaved changes

7. Mobile Adaptations:
   - Full-screen steps (no split layout)
   - Larger touch targets (min 44px)
   - Native file picker for uploads
```

### Code Context & Constraints

```typescript
interface ApplicationData {
  projectId: string;
  studentId: string;
  cv: File;
  statement: string;
  experience?: string;
  submittedAt?: Date;
  status: 'draft' | 'submitted' | 'reviewing' | 'accepted' | 'rejected';
}

// Use Ant Design: Steps, Form, Upload, Input.TextArea, Result
// Implement with react-hook-form for form management
// Use Zustand for application state management
// DO NOT implement actual file upload
// DO NOT connect to real APIs
// Mock successful submission after 2-second delay
```

### Strict Scope
- Create only the application flow component
- Assume project data is passed as props
- Use sessionStorage for draft saving
- Focus on UX flow, not backend integration

---

## Prompt 4: Progress Tracking Milestone View

### High-Level Goal
Create an intuitive milestone tracking interface that visualizes research progress, enables deliverable uploads, and facilitates professor-student feedback exchanges with clear status indicators.

### Detailed Instructions

```
1. Create `MilestoneTracker.tsx` with vertical timeline layout:

2. Timeline Component Structure:
   - Use Ant Timeline with custom milestone nodes
   - Each node shows:
     * Milestone name and number
     * Due date (with countdown if upcoming)
     * Status icon (pending/active/complete/overdue)
     * Progress percentage in node

3. Active Milestone Expanded View:
   - Milestone details card:
     * Title and description
     * Requirements checklist
     * Deliverables needed
   - Upload Section:
     * Drag-and-drop zone with file type hints
     * Uploaded files list with versions
     * File actions: View, Download, Delete
   - Progress Report Form:
     * "Work Completed" rich text editor
     * "Challenges Faced" text area
     * "Next Steps" bullet list
   - Research Journal Entry:
     * Date auto-filled
     * Reflection text area
     * Mood selector (5 emoji options)

4. Completed Milestones (collapsed by default):
   - Show completion date
   - Professor feedback summary
   - Grade/Status badge
   - Expand to see full details

5. Status Indicators:
   - Green: On track, submitted on time
   - Yellow: Approaching deadline (3 days)
   - Red: Overdue or needs revision
   - Blue: Under review

6. Professor Feedback Section:
   - Feedback card with professor avatar
   - Structured feedback: Strengths, Improvements, Next Steps
   - Inline comments on documents (preview)
   - Reply/Clarification thread

7. Interactions:
   - Click milestone to expand/collapse
   - Drag files to upload with progress bar
   - Auto-save draft every minute
   - Submit button with confirmation modal
```

### Code Context & Constraints

```typescript
interface Milestone {
  id: string;
  number: number;
  title: string;
  description: string;
  dueDate: Date;
  status: 'pending' | 'active' | 'submitted' | 'approved' | 'revision-needed';
  progress: number;
  deliverables: string[];
  submissions: Submission[];
  feedback?: Feedback;
}

interface Submission {
  id: string;
  files: Array<{name: string; size: number; url: string}>;
  report: {
    completed: string;
    challenges: string;
    nextSteps: string[];
  };
  journalEntry?: string;
  submittedAt: Date;
  version: number;
}

// Use Ant Design: Timeline, Card, Upload, Form, Badge, Avatar
// Implement optimistic UI updates
// Show skeleton loading for data fetching
// DO NOT implement real file upload to server
// Use IndexedDB for local file storage simulation
```

### Strict Scope
- Focus only on the milestone tracking UI
- Assume authentication is handled elsewhere
- Create reusable milestone components
- Mock 5 milestones with various states

---

## Prompt 5: Research Secretary Dashboard

### High-Level Goal
Build a comprehensive monitoring dashboard for research secretaries to track all projects, identify at-risk students, and perform batch operations efficiently with real-time status updates.

### Detailed Instructions

```
1. Create `SecretaryDashboard.tsx` with data-dense layout:

2. Top Statistics Row (4 cards):
   - Total Active Projects (with trend arrow)
   - At-Risk Projects (red badge count)
   - Pending Reviews (yellow badge)
   - This Week's Completions (green)

3. At-Risk Projects Alert Section:
   - Red banner if any critical issues
   - Table with columns:
     * Student Name (with avatar)
     * Project Title
     * Professor
     * Issue Type (overdue/no progress/failed)
     * Days Delayed
     * Action buttons: Contact, Intervene, Escalate

4. Project Overview Table:
   - Ant Table with features:
     * Sortable columns
     * Filterable by status, department, professor
     * Row selection for batch operations
     * Expandable rows for details
   - Columns:
     * Project ID
     * Student
     * Professor  
     * Current Milestone
     * Progress %
     * Status (traffic light)
     * Last Update
     * Actions

5. Batch Operations Toolbar:
   - Shows when rows selected
   - Actions: Send Reminder, Export Data, Reassign, Archive
   - Confirmation modal for destructive actions

6. Quick Filters:
   - Department dropdown
   - Status multi-select
   - Date range picker
   - Search by student/professor

7. Charts Section (responsive grid):
   - Progress distribution (pie chart)
   - Weekly submission trends (line chart)
   - Department comparison (bar chart)
   - Professor workload (heatmap)

8. Export & Reports:
   - Generate PDF report button
   - Export to Excel with filters
   - Schedule automated reports modal
```

### Code Context & Constraints

```typescript
interface SecretaryDashboardData {
  stats: {
    totalProjects: number;
    atRiskCount: number;
    pendingReviews: number;
    weeklyCompletions: number;
    trends: {
      projects: number; // percentage change
      atRisk: number;
      completions: number;
    };
  };
  projects: ProjectOverview[];
  alerts: Alert[];
}

interface ProjectOverview {
  id: string;
  student: {name: string; id: string; avatar: string};
  professor: {name: string; department: string};
  milestone: {current: number; total: number};
  progress: number;
  status: 'on-track' | 'at-risk' | 'delayed' | 'completed';
  lastUpdate: Date;
  daysUntilDeadline: number;
}

// Use Ant Design: Table, Statistic, Card, Alert, Select, DatePicker
// Use recharts for data visualization
// Implement virtual scrolling for large datasets
// DO NOT create backend APIs
// Mock 100+ projects for realistic testing
```

### Strict Scope
- Create only the dashboard view component
- Use React.memo for performance optimization
- Implement client-side filtering and sorting
- Mock batch operation results with toast notifications

---

## Usage Instructions

### For Developers

1. **Choose the appropriate prompt** based on which component you need to build
2. **Copy the entire prompt** including the tech stack context
3. **Paste into your AI tool** (v0, Lovable.ai, Claude, etc.)
4. **Review the generated code** carefully for:
   - TypeScript type safety
   - Accessibility compliance
   - Responsive design implementation
   - Internationalization setup
5. **Test thoroughly** on different devices and screen sizes
6. **Iterate** with follow-up prompts for refinements

### Important Notes

- **All generated code requires human review** before production use
- **Security considerations**: Never expose sensitive data in the UI
- **Performance**: Implement lazy loading for data-heavy components
- **Accessibility**: Test with screen readers and keyboard navigation
- **Internationalization**: Ensure all strings are translatable

### Customization Tips

- Replace color schemes to match your brand
- Adjust spacing using Tailwind's spacing scale
- Modify Ant Design theme through ConfigProvider
- Add animations using Framer Motion or CSS transitions
- Implement real API calls to replace mock data

---

## Next Steps After UI Generation

1. **Integration Checklist**:
   - Connect to real APIs using react-query or SWR
   - Implement proper authentication flow
   - Add error boundaries for graceful failures
   - Set up proper routing with protected routes
   - Configure build optimization

2. **Testing Requirements**:
   - Unit tests for component logic
   - Integration tests for user flows
   - Accessibility testing with axe-core
   - Cross-browser compatibility testing
   - Performance testing with Lighthouse

3. **Deployment Preparation**:
   - Environment variable configuration
   - Build optimization (code splitting, tree shaking)
   - CDN setup for static assets
   - Monitoring and error tracking setup
   - Documentation for maintenance team

Remember: AI-generated UI code is a starting point. Always review, test, and refine before considering it production-ready!