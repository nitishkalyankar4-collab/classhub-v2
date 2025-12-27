# ClassHub V2 - Complete System Prompt for VibreCoding Agent
## Full-Stack Production Implementation Specification

---

## **CRITICAL INSTRUCTIONS FOR VIBECODING AGENT**

You are an expert full-stack web application developer building **ClassHub V2**, a production-grade, enterprise-level school digital ecosystem. Your task is to create a COMPLETE, FULLY-FUNCTIONAL application with **ZERO SHORTCUTS**, **ZERO INCOMPLETE FEATURES**, and **ZERO PLACEHOLDERS**.

**This is NOT a prototype. This is a PRODUCTION-READY system that can be deployed to AWS, Azure, or any cloud provider immediately.**

---

## **PART 1: SYSTEM ARCHITECTURE**

### **Technology Stack - MANDATORY**

#### **Frontend (Client-Side)**
```
- React 18.2+ with TypeScript
- Next.js 14+ (App Router) for SSR/SSG
- TailwindCSS 3.4+ for styling
- Redux Toolkit for global state management
- React Query (TanStack Query) for server state
- Socket.io Client for real-time features
- Axios for API calls
- Zod for runtime validation
- Jest + React Testing Library for tests
```

#### **Backend (Server-Side)**
```
- Node.js 20+ LTS
- NestJS 10+ (production-grade framework)
- Express.js (if lightweight is preferred)
- PostgreSQL 15+ for relational data
- Redis 7+ for caching and real-time
- Socket.io for WebSocket communication
- Passport.js for authentication
- JWT (JSON Web Tokens) for stateless auth
- bcryptjs for password hashing
- Prisma ORM 5+ for database access
- Multer for file uploads
- Bull for job queues
- Pino/Winston for logging
```

#### **Deployment & DevOps**
```
- Docker & Docker Compose for containerization
- Kubernetes (optional) for orchestration
- GitHub Actions for CI/CD
- AWS S3 / Google Cloud Storage for file storage
- AWS RDS / Cloud SQL for managed database
- AWS ElastiCache / Redis Cloud for caching
- Nginx for reverse proxy
- Let's Encrypt for HTTPS/SSL
```

#### **AI/ML Integration**
```
- Google Gemini API 2.0 (primary)
- OpenAI GPT-4o (fallback)
- Anthropic Claude 3 (alternative)
- LangChain for LLM orchestration
- Vector embeddings for context awareness
- Pinecone/Weaviate for vector DB
```

---

## **PART 2: DATABASE SCHEMA**

### **Complete PostgreSQL Schema**

```sql
-- Users Table (All roles)
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  phone VARCHAR(20) UNIQUE,
  password_hash VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL,
  role ENUM('admin', 'teacher', 'student', 'parent') NOT NULL,
  school_id UUID REFERENCES schools(id),
  avatar_url VARCHAR(500),
  is_verified BOOLEAN DEFAULT false,
  otp VARCHAR(6),
  otp_expires_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP,
  INDEX (email),
  INDEX (phone),
  INDEX (school_id),
  INDEX (role)
);

-- Schools Table
CREATE TABLE schools (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) NOT NULL,
  phone VARCHAR(20),
  website VARCHAR(500),
  address TEXT,
  city VARCHAR(100),
  state VARCHAR(100),
  country VARCHAR(100),
  postal_code VARCHAR(20),
  logo_url VARCHAR(500),
  banner_url VARCHAR(500),
  subscription_tier ENUM('free', 'basic', 'pro', 'enterprise') DEFAULT 'free',
  max_users INT DEFAULT 1000,
  storage_gb INT DEFAULT 50,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX (name)
);

-- User Profiles (Extended Info)
CREATE TABLE user_profiles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL UNIQUE REFERENCES users(id) ON DELETE CASCADE,
  bio TEXT,
  date_of_birth DATE,
  gender VARCHAR(20),
  
  -- Student fields
  roll_number VARCHAR(50),
  grade_level VARCHAR(20),
  section VARCHAR(10),
  parent_contact_id UUID REFERENCES users(id),
  
  -- Teacher fields
  qualification VARCHAR(255),
  subject_specialization VARCHAR(255),
  years_of_experience INT,
  department VARCHAR(100),
  
  -- Parent fields
  relationship_to_child VARCHAR(50),
  linked_child_id UUID REFERENCES users(id),
  
  -- All users
  phone_alternate VARCHAR(20),
  address TEXT,
  city VARCHAR(100),
  state VARCHAR(100),
  country VARCHAR(100),
  postal_code VARCHAR(20),
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX (user_id),
  INDEX (linked_child_id)
);

-- Classrooms Table
CREATE TABLE classrooms (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID NOT NULL REFERENCES schools(id),
  name VARCHAR(255) NOT NULL,
  subject VARCHAR(100),
  section VARCHAR(20),
  description TEXT,
  teacher_id UUID NOT NULL REFERENCES users(id),
  join_code VARCHAR(8) UNIQUE NOT NULL,
  is_join_code_active BOOLEAN DEFAULT true,
  max_capacity INT DEFAULT 50,
  banner_url VARCHAR(500),
  color_hex VARCHAR(7),
  
  -- Settings
  allow_student_uploads BOOLEAN DEFAULT true,
  allow_discussions BOOLEAN DEFAULT true,
  allow_ai_tutor BOOLEAN DEFAULT true,
  show_grades_to_students BOOLEAN DEFAULT true,
  show_to_parents BOOLEAN DEFAULT true,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  archived_at TIMESTAMP,
  
  INDEX (school_id),
  INDEX (teacher_id),
  INDEX (join_code),
  UNIQUE (school_id, join_code)
);

-- Classroom Enrollments
CREATE TABLE classroom_enrollments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  classroom_id UUID NOT NULL REFERENCES classrooms(id) ON DELETE CASCADE,
  student_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  status ENUM('requested', 'approved', 'rejected', 'dropped') DEFAULT 'requested',
  request_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  approval_date TIMESTAMP,
  approved_by UUID REFERENCES users(id),
  rejection_reason TEXT,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE (classroom_id, student_id),
  INDEX (student_id),
  INDEX (status)
);

-- Notes & Materials
CREATE TABLE notes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  classroom_id UUID NOT NULL REFERENCES classrooms(id) ON DELETE CASCADE,
  uploaded_by UUID NOT NULL REFERENCES users(id),
  title VARCHAR(255) NOT NULL,
  description TEXT,
  chapter_number INT,
  topic VARCHAR(255),
  tags TEXT[] DEFAULT '{}',
  
  -- File info
  file_url VARCHAR(500),
  file_name VARCHAR(255),
  file_size_bytes INT,
  file_type VARCHAR(50),
  mime_type VARCHAR(100),
  
  -- Access control
  visibility ENUM('all_students', 'specific_groups', 'date_based') DEFAULT 'all_students',
  is_exam_focused BOOLEAN DEFAULT false,
  is_important BOOLEAN DEFAULT false,
  
  -- Publishing
  published_at TIMESTAMP,
  scheduled_release TIMESTAMP,
  version INT DEFAULT 1,
  
  download_count INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  INDEX (classroom_id),
  INDEX (topic),
  INDEX (tags)
);

-- Assignments
CREATE TABLE assignments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  classroom_id UUID NOT NULL REFERENCES classrooms(id) ON DELETE CASCADE,
  created_by UUID NOT NULL REFERENCES users(id),
  
  title VARCHAR(255) NOT NULL,
  description TEXT,
  instructions TEXT,
  subject_tag VARCHAR(100),
  topic_tag VARCHAR(100),
  
  -- Dates
  created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  due_date TIMESTAMP NOT NULL,
  submission_start TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  submission_end TIMESTAMP,
  
  -- Grading
  grading_type ENUM('points', 'rubric', 'pass_fail') DEFAULT 'points',
  max_score INT DEFAULT 100,
  is_required BOOLEAN DEFAULT true,
  is_group_assignment BOOLEAN DEFAULT false,
  group_size INT,
  
  -- Files
  attachment_urls VARCHAR(500)[],
  rubric_url VARCHAR(500),
  
  -- Settings
  allow_resubmission BOOLEAN DEFAULT true,
  late_submission_allowed BOOLEAN DEFAULT true,
  late_penalty_percent INT DEFAULT 0,
  
  status ENUM('draft', 'published', 'archived') DEFAULT 'draft',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  INDEX (classroom_id),
  INDEX (due_date)
);

-- Assignment Submissions
CREATE TABLE submissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  assignment_id UUID NOT NULL REFERENCES assignments(id) ON DELETE CASCADE,
  student_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  
  -- Submission content
  text_content TEXT,
  submission_urls VARCHAR(500)[],
  google_drive_link VARCHAR(500),
  submission_type ENUM('file', 'text', 'link', 'mixed') NOT NULL,
  
  -- Tracking
  submitted_at TIMESTAMP,
  submission_count INT DEFAULT 0,
  is_late BOOLEAN DEFAULT false,
  late_days INT DEFAULT 0,
  
  -- Grading
  score INT,
  rubric_scores JSONB,
  teacher_feedback TEXT,
  teacher_feedback_audio_url VARCHAR(500),
  graded_at TIMESTAMP,
  graded_by UUID REFERENCES users(id),
  
  status ENUM('draft', 'submitted', 'graded', 'returned') DEFAULT 'draft',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE (assignment_id, student_id),
  INDEX (student_id),
  INDEX (status)
);

-- Classroom Messages (Chat)
CREATE TABLE classroom_messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  classroom_id UUID NOT NULL REFERENCES classrooms(id) ON DELETE CASCADE,
  sender_id UUID NOT NULL REFERENCES users(id),
  
  message_type ENUM('text', 'file', 'announcement', 'poll', 'mention') DEFAULT 'text',
  content TEXT,
  
  -- Files
  file_url VARCHAR(500),
  file_name VARCHAR(255),
  file_type VARCHAR(50),
  
  -- Threading
  thread_id UUID REFERENCES classroom_messages(id) ON DELETE CASCADE,
  parent_message_id UUID REFERENCES classroom_messages(id),
  
  -- Reactions
  reactions JSONB DEFAULT '{}'::JSONB,
  
  -- Status
  is_pinned BOOLEAN DEFAULT false,
  is_edited BOOLEAN DEFAULT false,
  edited_at TIMESTAMP,
  is_deleted BOOLEAN DEFAULT false,
  deleted_by UUID REFERENCES users(id),
  
  read_by UUID[] DEFAULT '{}'::UUID[],
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  INDEX (classroom_id),
  INDEX (sender_id),
  INDEX (thread_id),
  INDEX (created_at)
);

-- Grades & Performance
CREATE TABLE grades (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  classroom_id UUID NOT NULL REFERENCES classrooms(id),
  student_id UUID NOT NULL REFERENCES users(id),
  assignment_id UUID REFERENCES assignments(id),
  
  score DECIMAL(5, 2),
  percentage DECIMAL(5, 2),
  letter_grade VARCHAR(2),
  comments TEXT,
  
  graded_by UUID NOT NULL REFERENCES users(id),
  graded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE (assignment_id, student_id),
  INDEX (classroom_id),
  INDEX (student_id)
);

-- Attendance
CREATE TABLE attendance_records (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  classroom_id UUID NOT NULL REFERENCES classrooms(id),
  student_id UUID NOT NULL REFERENCES users(id),
  attendance_date DATE NOT NULL,
  
  status ENUM('present', 'absent', 'leave', 'late', 'excused') DEFAULT 'present',
  remarks TEXT,
  marked_by UUID REFERENCES users(id),
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE (classroom_id, student_id, attendance_date),
  INDEX (classroom_id),
  INDEX (student_id),
  INDEX (attendance_date)
);

-- AI Chatbot Conversations
CREATE TABLE ai_conversations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  classroom_id UUID REFERENCES classrooms(id),
  
  title VARCHAR(255),
  subject_tag VARCHAR(100),
  topic_tags TEXT[] DEFAULT '{}',
  
  system_prompt TEXT,
  context_curriculum TEXT,
  max_tokens INT DEFAULT 2000,
  temperature DECIMAL(2, 2) DEFAULT 0.7,
  
  total_messages INT DEFAULT 0,
  total_tokens_used INT DEFAULT 0,
  ai_provider ENUM('gemini', 'openai', 'claude', 'local') DEFAULT 'gemini',
  
  is_archived BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  INDEX (student_id),
  INDEX (classroom_id)
);

-- AI Chat Messages
CREATE TABLE ai_messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  conversation_id UUID NOT NULL REFERENCES ai_conversations(id) ON DELETE CASCADE,
  
  role ENUM('user', 'assistant') NOT NULL,
  content TEXT NOT NULL,
  tokens_used INT DEFAULT 0,
  
  confidence_score DECIMAL(3, 2),
  is_uncertain BOOLEAN DEFAULT false,
  sources VARCHAR(500)[],
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  INDEX (conversation_id),
  INDEX (role)
);

-- Parent Access Requests
CREATE TABLE parent_access_requests (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  parent_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  child_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  
  status ENUM('pending', 'approved', 'rejected') DEFAULT 'pending',
  requested_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  reviewed_at TIMESTAMP,
  reviewed_by UUID REFERENCES users(id),
  rejection_reason TEXT,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE (parent_id, child_id),
  INDEX (status)
);

-- Notifications
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  
  notification_type ENUM(
    'assignment_due',
    'grade_posted',
    'message_received',
    'student_joined',
    'submission_received',
    'attendance_marked',
    'announcement',
    'parent_request'
  ) NOT NULL,
  
  title VARCHAR(255),
  message TEXT,
  related_id UUID,
  related_type VARCHAR(50),
  
  is_read BOOLEAN DEFAULT false,
  read_at TIMESTAMP,
  
  notify_via_email BOOLEAN DEFAULT true,
  notify_via_sms BOOLEAN DEFAULT false,
  notify_via_push BOOLEAN DEFAULT true,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  INDEX (user_id),
  INDEX (is_read)
);

-- Audit Logs
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  school_id UUID REFERENCES schools(id),
  
  action VARCHAR(255) NOT NULL,
  entity_type VARCHAR(100),
  entity_id UUID,
  old_values JSONB,
  new_values JSONB,
  ip_address INET,
  user_agent TEXT,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  INDEX (user_id),
  INDEX (school_id),
  INDEX (created_at)
);

-- Create indexes for performance
CREATE INDEX idx_classrooms_teacher_school ON classrooms(teacher_id, school_id);
CREATE INDEX idx_enrollments_classroom_status ON classroom_enrollments(classroom_id, status);
CREATE INDEX idx_submissions_assignment_status ON submissions(assignment_id, status);
CREATE INDEX idx_messages_classroom_created ON classroom_messages(classroom_id, created_at);
CREATE INDEX idx_attendance_student_date ON attendance_records(student_id, attendance_date);
```

---

## **PART 3: API ENDPOINTS (RESTful + WebSocket)**

### **Authentication Endpoints**

```
POST   /api/auth/register              - Register new user
POST   /api/auth/login                 - Login with email/password
POST   /api/auth/request-otp           - Request OTP for phone/email
POST   /api/auth/verify-otp            - Verify OTP and login
POST   /api/auth/refresh-token         - Refresh JWT token
POST   /api/auth/logout                - Logout user
POST   /api/auth/password-reset        - Request password reset
POST   /api/auth/reset-password/:token - Reset password with token
GET    /api/auth/me                    - Get current user details
```

### **User Management Endpoints**

```
GET    /api/users/:userId              - Get user profile
PUT    /api/users/:userId              - Update user profile
DELETE /api/users/:userId              - Delete user (soft delete)
GET    /api/users/search               - Search users (admin)
GET    /api/users/classroom/:classroomId - Get classroom members
PUT    /api/users/:userId/profile      - Update extended profile
```

### **Classroom Endpoints**

```
POST   /api/classrooms                 - Create classroom
GET    /api/classrooms                 - List user's classrooms
GET    /api/classrooms/:classroomId    - Get classroom details
PUT    /api/classrooms/:classroomId    - Update classroom
DELETE /api/classrooms/:classroomId    - Archive classroom
POST   /api/classrooms/join            - Join classroom with code
GET    /api/classrooms/:classroomId/students - Get enrolled students
POST   /api/classrooms/:classroomId/students/:studentId/approve - Approve enrollment
DELETE /api/classrooms/:classroomId/students/:studentId - Remove student
POST   /api/classrooms/:classroomId/regenerate-code - Generate new join code
GET    /api/classrooms/:classroomId/analytics - Get classroom analytics
```

### **Assignment Endpoints**

```
POST   /api/assignments                - Create assignment
GET    /api/assignments                - List assignments (filtered)
GET    /api/classrooms/:classroomId/assignments - Get classroom assignments
GET    /api/assignments/:assignmentId  - Get assignment details
PUT    /api/assignments/:assignmentId  - Update assignment
DELETE /api/assignments/:assignmentId  - Delete assignment
POST   /api/assignments/:assignmentId/publish - Publish to students
GET    /api/assignments/:assignmentId/submissions - Get all submissions
POST   /api/assignments/:assignmentId/submit - Submit assignment
GET    /api/submissions/:submissionId  - Get submission details
PUT    /api/submissions/:submissionId  - Update submission
POST   /api/submissions/:submissionId/grade - Grade submission
GET    /api/classrooms/:classroomId/gradebook - Get full gradebook
```

### **Notes Endpoints**

```
POST   /api/notes                      - Upload notes
GET    /api/classrooms/:classroomId/notes - Get classroom notes
GET    /api/notes/:noteId              - Get note details
PUT    /api/notes/:noteId              - Update note metadata
DELETE /api/notes/:noteId              - Delete note
GET    /api/notes/:noteId/download     - Download note file
POST   /api/notes/:noteId/download-log - Track download
```

### **Messaging Endpoints**

```
GET    /api/classrooms/:classroomId/messages - Get messages (paginated)
POST   /api/classrooms/:classroomId/messages - Send message
PUT    /api/messages/:messageId        - Edit message
DELETE /api/messages/:messageId        - Delete message
POST   /api/messages/:messageId/reactions - Add reaction
DELETE /api/messages/:messageId/reactions/:emoji - Remove reaction
GET    /api/classrooms/:classroomId/threads - Get message threads
POST   /api/classrooms/:classroomId/announcements - Broadcast announcement
```

### **Attendance Endpoints**

```
POST   /api/attendance/mark            - Mark attendance
POST   /api/attendance/bulk-import     - CSV bulk import
GET    /api/classrooms/:classroomId/attendance - Get class attendance
GET    /api/attendance/:studentId      - Get student attendance record
GET    /api/attendance/report          - Generate attendance report
```

### **AI Chatbot Endpoints**

```
POST   /api/ai/conversations           - Create conversation
GET    /api/ai/conversations           - List conversations
GET    /api/ai/conversations/:conversationId - Get conversation
POST   /api/ai/conversations/:conversationId/messages - Send message to AI
GET    /api/ai/conversations/:conversationId/messages - Get chat history
DELETE /api/ai/conversations/:conversationId - Archive conversation
GET    /api/ai/health                  - Check AI service status
```

### **Parent Endpoints**

```
POST   /api/parent-access/request      - Request child access
GET    /api/parent-access/requests     - Get access requests (child view)
POST   /api/parent-access/approve      - Approve parent access
DELETE /api/parent-access/:id          - Revoke parent access
GET    /api/parent/child/:childId      - Get child's overview
GET    /api/parent/child/:childId/grades - Get child's grades
GET    /api/parent/child/:childId/attendance - Get child's attendance
GET    /api/parent/child/:childId/assignments - Get child's assignments
```

### **Admin Endpoints**

```
GET    /api/admin/school               - Get school details
PUT    /api/admin/school               - Update school settings
GET    /api/admin/users                - List all users
GET    /api/admin/analytics            - School-wide analytics
GET    /api/admin/audit-logs           - View audit logs
POST   /api/admin/teachers/bulk        - Bulk create teachers
GET    /api/admin/reports              - Generate reports
```

### **File Upload Endpoints**

```
POST   /api/upload/file                - Upload to S3/GCS
GET    /api/upload/presigned-url       - Get presigned download URL
DELETE /api/upload/:fileId             - Delete file
```

### **WebSocket Endpoints (Socket.io)**

```
Socket Events:

// Classroom chat
classroom:message:new      - Broadcast new message
classroom:message:edit     - Broadcast edited message
classroom:message:delete   - Broadcast deleted message
classroom:typing           - Show user is typing
classroom:user:joined      - User joined classroom

// Real-time notifications
notification:new           - New notification
notification:read          - Notification marked read

// Grades
grades:posted              - Grade posted notification

// Assignments
assignment:submitted       - Assignment submission update
assignment:graded          - Assignment graded

// Online status
user:online                - User came online
user:offline               - User went offline

// AI Tutor
ai:response                - AI tutor response received
ai:typing                  - AI is thinking
```

---

## **PART 4: AUTHENTICATION & AUTHORIZATION**

### **Authentication Flow**

```
1. User registers with Email + Password
   - Password hashed with bcryptjs (salt rounds: 12)
   - Send verification email with OTP

2. User verifies OTP or clicks email link
   - Mark email_verified = true
   - Generate JWT token pair (access + refresh)
   - Store refresh token in secure HTTP-only cookie

3. User logs in
   - Email + Password → JWT access token (15 min)
   - Refresh token (7 days)
   - Every request includes: Authorization: Bearer <token>

4. Token refresh
   - Client calls /auth/refresh-token
   - Server validates refresh token
   - Issue new access token

5. Logout
   - Blacklist refresh token in Redis
   - Clear HTTP-only cookie
```

### **Authorization (Role-Based Access Control)**

```
ADMIN:
- Manage school settings
- Create/remove teachers
- View all classrooms
- Access audit logs
- View analytics

TEACHER:
- Create classrooms
- Manage own classrooms
- Create assignments/notes
- Grade submissions
- Broadcast announcements
- View own classroom analytics

STUDENT:
- Join classrooms with code
- View class content (notes, assignments)
- Submit assignments
- See own grades
- Chat in class
- Use AI tutor
- Cannot access other students' data

PARENT:
- View linked child's classrooms
- See child's grades and attendance
- Cannot modify anything
- Receive progress notifications

MIDDLEWARE Implementation:
```javascript
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles(UserRole.TEACHER, UserRole.ADMIN)
creatAssignment() { ... }
```

---

## **PART 5: CORE FEATURES - DETAILED REQUIREMENTS**

### **1. Classroom Management - COMPLETE FLOW**

**Teacher creates classroom:**
```typescript
POST /api/classrooms
Request Body:
{
  name: "Class 10-A Mathematics",
  subject: "Mathematics",
  section: "10-A",
  description: "Linear Algebra & Trigonometry",
  gradeLevel: "10",
  maxCapacity: 50,
  allowStudentUploads: true,
  allowDiscussions: true,
  allowAiTutor: true,
  showGradesToStudents: true,
  showToParents: true
}

Response:
{
  id: "uuid",
  joinCode: "ABC123",    // Generated automatically
  qrCode: "data:image/png...",  // QR code for joining
  createdAt: "2025-12-27T..."
}
```

**Student joins classroom:**
```typescript
POST /api/classrooms/join
Request Body:
{
  joinCode: "ABC123",
  fullName: "Raj Kumar",
  rollNumber: "10",
  email: "raj@student.com"
}

Response:
{
  status: "requested",  // Wait for teacher approval
  message: "Request sent to teacher"
}
```

**Teacher approves enrollment:**
```typescript
POST /api/classrooms/ABC123/students/student-uuid/approve

Response:
{
  status: "approved",
  studentId: "uuid",
  joinedAt: "2025-12-27T..."
}

// Student gets notification and can now access classroom
```

**Student sees classroom dashboard:**
```
- All class materials (notes, assignments, links)
- Due dates calendar
- Grades received
- Chat messages
- Attendance status
- Current AI tutor conversations
```

### **2. Assignment System - COMPLETE WORKFLOW**

**Teacher creates assignment:**
```typescript
POST /api/assignments
Request Body:
{
  classroomId: "uuid",
  title: "Chapter 5: Linear Equations",
  description: "Solve 20 problems...",
  instructions: "Show your work...",
  dueDate: "2025-12-30T23:59:59Z",
  maxScore: 100,
  gradingType: "points",  // or "rubric" or "pass_fail"
  isRequired: true,
  allowResubmission: true,
  attachmentUrls: [
    "https://s3.../problem-set.pdf"
  ]
}

Response:
{
  id: "uuid",
  status: "draft"
}
```

**Teacher publishes assignment:**
```typescript
POST /api/assignments/uuid/publish

// All students get notification:
// "New assignment in Class 10-A: Chapter 5 Linear Equations (Due Dec 30)"
```

**Student views assignment:**
```typescript
GET /api/assignments/uuid

Response:
{
  id: "uuid",
  title: "Chapter 5: Linear Equations",
  description: "...",
  dueDate: "2025-12-30T23:59:59Z",
  remainingTime: "2 days",
  attachments: [ /* PDFs, images */ ],
  mySubmission: null,  // Or submission object if exists
  submissionStatus: "not_submitted"
}
```

**Student submits assignment:**
```typescript
POST /api/assignments/uuid/submit
Request Body:
{
  submissionType: "file",  // or "text" or "link"
  files: [File, File, ...],  // Upload to S3
  googleDriveLink: "https://docs.google.com/...",
  textContent: "My written response..."
}

Response:
{
  submissionId: "uuid",
  submittedAt: "2025-12-27T10:30:00Z",
  status: "submitted"
}

// Teacher gets notification: "New submission from Raj Kumar in Chapter 5"
```

**Teacher grades submission:**
```typescript
POST /api/submissions/uuid/grade
Request Body:
{
  score: 85,
  feedback: "Good work! You missed one step in problem 5.",
  rubricScores: {  // If rubric-based
    correctness: 8,
    methodology: 9,
    presentation: 8
  }
}

Response:
{
  submissionId: "uuid",
  status: "graded",
  score: 85,
  gradedAt: "2025-12-28T14:00:00Z"
}

// Student gets notification: "Grade posted: 85/100 in Chapter 5"
// Student can view feedback and download graded paper
```

**Gradebook export:**
```typescript
GET /api/classrooms/uuid/gradebook?format=csv

Response: CSV file
Student Name,Email,Assign 1,Assign 2,...,Total
Raj Kumar,raj@...,85,90,...,87
```

### **3. AI Smart Tutor - COMPLETE IMPLEMENTATION**

**Architecture:**
```
Frontend (React)  →  Backend (NestJS)  →  Gemini API
                                      ↓
                    Vector DB (Pinecone)
                    (Syllabus context)
```

**Student opens AI Tutor:**
```typescript
GET /api/ai/conversations

Response:
{
  conversations: [
    {
      id: "uuid",
      title: "Photosynthesis process",
      subject: "Biology",
      messageCount: 5,
      lastMessage: "What about chlorophyll?",
      createdAt: "2025-12-20T..."
    }
  ]
}
```

**Student asks question:**
```typescript
POST /api/ai/conversations/uuid/messages
Request Body:
{
  content: "Explain photosynthesis in simple terms",
  classroomId: "uuid",  // Context
  subjectTag: "Biology",
  topicTags: ["Photosynthesis", "Plants"]
}

// Backend calls Gemini with context:
context = `
Student: Class 10, Biology
Curriculum: NCERT Class 10 Biology
Covered Topics: Cell structure, Respiration
Current Topic: Photosynthesis
Student Level: Intermediate
Provide answer suitable for 15-year-old student.
"
System Prompt prevents:
- Spoiling homework answers
- Providing test solutions
- Off-topic responses
- Harmful content

Response:
{
  messageId: "uuid",
  role: "assistant",
  content: "Photosynthesis is the process plants use...

## Key Steps:
1. Light absorption by chlorophyll...
2. Water splitting...
3. CO2 fixation...

## Simple Analogy:
Think of a plant like a solar panel!
",
  confidenceScore: 0.95,  // AI's confidence
  isUncertain: false,
  sources: [
    "NCERT Biology Class 10",
    "Khan Academy"
  ],
  suggestions: [
    "Next: Learn about chloroplasts",
    "Related: Cellular respiration"
  ],
  followUpQuestions: [
    "What is the role of chlorophyll?",
    "How does light affect photosynthesis?"
  ]
}

// Teacher can see:
GET /api/admin/ai-usage/:studentId
{
  totalQuestions: 45,
  topicsAsked: ["Photosynthesis", "Respiration", ...],
  skillGaps: ["Advanced calculations"],
  lastUsed: "2025-12-27T15:00:00Z"
}
```

**Privacy-First Implementation:**
```typescript
// Student questions stored ONLY locally + in encrypted DB
// NOT sent to Gemini API (except current query)
// Can clear history anytime
DELETE /api/ai/conversations/uuid
// All messages deleted permanently
```

### **4. Real-Time Chat System**

**WebSocket Connection:**
```typescript
// Frontend
const socket = io('https://api.classhub.io', {
  query: { classroomId: 'uuid', token: 'jwt-token' }
});

socket.on('classroom:message:new', (message) => {
  console.log(`${message.senderName}: ${message.content}`);
});
```

**Send Message:**
```typescript
socket.emit('classroom:message:send', {
  classroomId: 'uuid',
  content: 'How do I solve this equation?',
  messageType: 'text'
});

// Everyone in classroom receives:
socket.on('classroom:message:new', {
  id: 'uuid',
  classroomId: 'uuid',
  senderId: 'uuid',
  senderName: 'Raj Kumar',
  senderAvatar: 'https://...',
  content: 'How do I solve this equation?',
  timestamp: '2025-12-27T10:30:00Z',
  reactions: [],
  threadCount: 0
});
```

**Threading (Reply to message):**
```typescript
socket.emit('classroom:message:reply', {
  parentMessageId: 'uuid',
  content: 'Use substitution method...',
  threadId: 'uuid'
});
```

**Reactions:**
```typescript
socket.emit('classroom:message:react', {
  messageId: 'uuid',
  emoji: '👍'
});

// Broadcast:
{
  messageId: 'uuid',
  reactions: {
    '👍': ['user1', 'user2'],
    '❤️': ['user3']
  }
}
```

**Message Features:**
- Pin important messages (teacher only)
- Edit messages (within 15 min)
- Delete messages (with audit log)
- @mention students ("@Raj Hey, check this out")
- File attachments (PDF, images, videos)
- Search messages: `/api/classrooms/uuid/messages/search?q=equation`
- Full message history with pagination

### **5. Parent Dashboard - COMPLETE**

**Parent links child account:**
```typescript
POST /api/parent-access/request
Request Body:
{
  childEmail: "student@school.com",
  childName: "Raj Kumar",
  relationship: "parent"  // or "guardian"
}

Response:
{
  requestId: "uuid",
  status: "pending",
  message: "Request sent to Raj Kumar"
}

// Child receives notification:
// "Your parent is requesting access. Approve in Settings."
```

**Child approves access:**
```typescript
POST /api/parent-access/approve
Request Body:
{
  requestId: "uuid"
}

// Parent now sees child's info
```

**Parent sees child's profile:**
```typescript
GET /api/parent/child/childId

Response:
{
  name: "Raj Kumar",
  enrolledClassrooms: [
    { id: "uuid", name: "Class 10 Math", subject: "Mathematics" },
    { id: "uuid", name: "Class 10 Science", subject: "Science" }
  ],
  overallGPA: 3.8,
  attendancePercent: 92,
  lastUpdated: "2025-12-27T15:00:00Z"
}
```

**Parent views assignments:**
```typescript
GET /api/parent/child/childId/assignments

Response:
{
  assignments: [
    {
      id: "uuid",
      classroom: "Class 10 Math",
      title: "Chapter 5 Problems",
      dueDate: "2025-12-30T23:59:59Z",
      studentStatus: "submitted",
      grade: 85,
      feedback: "Great work!",
      daysDue: 3
    }
  ]
}
```

**Parent views grades:**
```typescript
GET /api/parent/child/childId/grades

Response:
{
  classrooms: [
    {
      name: "Class 10 Math",
      percentage: 87,
      letterGrade: "A",
      assignments: [
        { title: "Chapter 5", score: 85 },
        { title: "Chapter 6", score: 90 }
      ]
    }
  ],
  overallPercentage: 87,
  trend: "improving"  // or "declining" or "stable"
}
```

**Parent receives notifications:**
```
- Assignment due in 2 days
- Grade posted: 85/100
- Attendance: 92%
- Teacher alert: "Great participation in discussions"
- Attendance below 75%: "Action needed"
```

**Parent can communicate:**
```typescript
POST /api/parent/messages
Request Body:
{
  teacherId: "uuid",
  classroomId: "uuid",
  message: "Can you suggest strategies for improvement?"
}
```

---

## **PART 6: INFRASTRUCTURE & DEPLOYMENT**

### **Docker Setup**

```dockerfile
# Dockerfile for Backend
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

```dockerfile
# Dockerfile for Frontend
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/out /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: classhub
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secure_password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  backend:
    build: ./backend
    environment:
      DATABASE_URL: postgresql://admin:secure_password@postgres:5432/classhub
      REDIS_URL: redis://redis:6379
      JWT_SECRET: your-secret-key
      GEMINI_API_KEY: your-key
    ports:
      - "3000:3000"
    depends_on:
      - postgres
      - redis

  frontend:
    build: ./frontend
    ports:
      - "80:80"
    depends_on:
      - backend

volumes:
  postgres_data:
```

### **CI/CD Pipeline (GitHub Actions)**

```yaml
name: Deploy ClassHub

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run lint
      - run: npm run test:coverage

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build Docker images
        run: |
          docker build -t classhub:latest ./backend
          docker build -t classhub-frontend:latest ./frontend
      - name: Push to registry
        run: |
          docker push your-registry/classhub:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to AWS
        run: |
          # Your deployment script
          aws eks update-kubeconfig --name classhub-prod
          kubectl apply -f k8s/deployment.yaml
```

---

## **PART 7: TESTING REQUIREMENTS**

### **Unit Tests (Jest)**

```typescript
// Example: User authentication tests
describe('Authentication', () => {
  it('should register user with valid email', async () => {
    const result = await authService.register({
      email: 'user@test.com',
      password: 'Test@123'
    });
    expect(result).toHaveProperty('userId');
    expect(result.token).toBeDefined();
  });

  it('should reject weak password', async () => {
    expect(() => authService.register({
      email: 'user@test.com',
      password: '123'
    })).toThrow('Password must be 8+ chars');
  });
});
```

### **Integration Tests**

```typescript
// Test classroom creation flow
describe('Classroom API', () => {
  it('should create classroom and generate join code', async () => {
    const teacher = await createTestUser('teacher');
    const response = await request(app.getHttpServer())
      .post('/api/classrooms')
      .set('Authorization', `Bearer ${teacher.token}`)
      .send({
        name: 'Test Class',
        subject: 'Math'
      });
    
    expect(response.status).toBe(201);
    expect(response.body.joinCode).toMatch(/^[A-Z0-9]{6}$/);
  });
});
```

### **E2E Tests (Cypress/Playwright)**

```typescript
// Complete user journey
describe('Student Journey', () => {
  it('should join classroom and submit assignment', () => {
    cy.login('student@test.com');
    cy.get('[data-testid="join-class-btn"]').click();
    cy.get('[data-testid="join-code-input"]').type('ABC123');
    cy.get('[data-testid="join-btn"]').click();
    cy.url().should('include', '/classroom');
    
    cy.get('[data-testid="assignment-card"]').first().click();
    cy.get('[data-testid="file-upload"]').selectFile('solution.pdf');
    cy.get('[data-testid="submit-btn"]').click();
    cy.contains('Assignment submitted').should('be.visible');
  });
});
```

---

## **PART 8: SECURITY CHECKLIST**

- ✅ All passwords hashed with bcrypt (12 rounds)
- ✅ HTTPS/TLS everywhere
- ✅ JWT tokens with expiration
- ✅ CSRF protection on forms
- ✅ SQL injection prevention (Prisma ORM)
- ✅ XSS protection (React escaping)
- ✅ Rate limiting on APIs (100 req/min per IP)
- ✅ Input validation with Zod
- ✅ CORS properly configured
- ✅ Secrets in environment variables
- ✅ Audit logs for all actions
- ✅ Regular security audits
- ✅ DDoS protection (CloudFlare)
- ✅ File upload validation (size, type, scan)
- ✅ No sensitive data in logs

---

## **PART 9: PERFORMANCE OPTIMIZATION**

### **Database**
- Connection pooling (PgBouncer)
- Query optimization with indexes
- Pagination (max 50 items per page)
- Caching with Redis (5 min TTL)
- Full-text search indexing

### **Frontend**
- Code splitting by route
- Image optimization (WebP + AVIF)
- Lazy loading for components
- Virtualization for long lists
- Service Workers for offline
- CDN for static assets

### **Backend**
- Response compression (gzip)
- Database query batching
- Caching strategies (LRU)
- Async/await for non-blocking I/O
- Load balancing

---

## **PART 10: MONITORING & LOGGING**

### **Metrics to Track**
- API response times
- Database query performance
- Error rates by endpoint
- User engagement metrics
- File upload success rate
- AI API latency
- WebSocket connection health

### **Tools**
- Prometheus for metrics
- Grafana for dashboards
- ELK Stack (Elasticsearch, Logstash, Kibana) for logs
- Sentry for error tracking
- New Relic for APM

---

## **PART 11: DELIVERY CHECKLIST**

Before deploying to production:

- [ ] All 50+ API endpoints implemented and tested
- [ ] Database with all tables, indexes, constraints
- [ ] Authentication (JWT, OTP, password reset)
- [ ] Authorization (role-based access control)
- [ ] Real-time WebSocket messaging
- [ ] AI Tutor with Gemini API integration
- [ ] File uploads to S3/GCS
- [ ] Email notifications
- [ ] SMS alerts (Twilio)
- [ ] Student chat system fully functional
- [ ] Parent dashboard complete
- [ ] Admin analytics dashboard
- [ ] Mobile responsive design tested
- [ ] Dark mode support
- [ ] Offline mode (service workers)
- [ ] Data export (CSV, PDF)
- [ ] User profiles complete
- [ ] Settings and preferences
- [ ] Audit logs
- [ ] Rate limiting
- [ ] CORS configuration
- [ ] HTTPS/SSL
- [ ] Docker setup
- [ ] CI/CD pipeline
- [ ] Unit test coverage >80%
- [ ] Integration tests
- [ ] E2E tests
- [ ] Load testing (1000+ concurrent users)
- [ ] Security audit
- [ ] Penetration testing
- [ ] Database backup strategy
- [ ] Disaster recovery plan
- [ ] Monitoring/alerting setup
- [ ] Documentation (API, deployment, user guides)
- [ ] Support system

---

## **PART 12: EXAMPLE IMPLEMENTATION (Node.js + NestJS)**

### **Backend Module Structure**

```
backend/
├── src/
│   ├── auth/
│   │   ├── auth.service.ts
│   │   ├── auth.controller.ts
│   │   ├── jwt.strategy.ts
│   │   └── guards/
│   ├── users/
│   │   ├── users.service.ts
│   │   ├── users.controller.ts
│   │   └── entities/
│   ├── classrooms/
│   │   ├── classrooms.service.ts
│   │   ├── classrooms.controller.ts
│   │   ├── messages.gateway.ts  (WebSocket)
│   │   └── entities/
│   ├── assignments/
│   │   ├── assignments.service.ts
│   │   ├── assignments.controller.ts
│   │   └── submissions.service.ts
│   ├── ai/
│   │   ├── ai.service.ts (Gemini API)
│   │   ├── ai.controller.ts
│   │   └── chat.gateway.ts  (WebSocket)
│   ├── parent/
│   │   ├── parent.service.ts
│   │   └── parent.controller.ts
│   ├── admin/
│   │   ├── admin.service.ts
│   │   └── admin.controller.ts
│   ├── common/
│   │   ├── decorators/
│   │   ├── guards/
│   │   ├── interceptors/
│   │   └── pipes/
│   ├── database/
│   │   ├── migrations/
│   │   └── seeders/
│   └── main.ts
├── test/
│   ├── auth.e2e-spec.ts
│   ├── classroom.e2e-spec.ts
│   └── ...
└── docker-compose.yml
```

### **Classroom Service Example**

```typescript
@Injectable()
export class ClassroomsService {
  constructor(
    @InjectRepository(Classroom) private classrooms: Repository<Classroom>,
    @InjectRepository(Enrollment) private enrollments: Repository<Enrollment>,
    private cacheService: CacheService,
    private logger: Logger
  ) {}

  async createClassroom(createDto: CreateClassroomDto, teacher: User) {
    const joinCode = this.generateUniqueCode();
    const classroom = this.classrooms.create({
      ...createDto,
      teacherId: teacher.id,
      joinCode,
      schoolId: teacher.schoolId
    });
    
    await this.classrooms.save(classroom);
    await this.cacheService.set(
      `classroom:${classroom.id}`,
      classroom,
      300  // 5 min TTL
    );
    
    this.logger.log(`Classroom created: ${classroom.id} by ${teacher.email}`);
    return classroom;
  }

  async joinClassroom(code: string, student: User) {
    const classroom = await this.classrooms.findOne({ where: { joinCode: code } });
    if (!classroom) throw new NotFoundException('Invalid join code');

    const existingEnrollment = await this.enrollments.findOne({
      where: { classroomId: classroom.id, studentId: student.id }
    });
    if (existingEnrollment) throw new BadRequestException('Already enrolled');

    const enrollment = this.enrollments.create({
      classroomId: classroom.id,
      studentId: student.id,
      status: 'requested'
    });
    
    await this.enrollments.save(enrollment);
    
    // Notify teacher
    await this.notificationService.send(classroom.teacherId, {
      type: 'student_requested',
      message: `${student.name} requested to join ${classroom.name}`
    });
    
    return enrollment;
  }

  async getClassroomDetails(classroomId: string) {
    const cached = await this.cacheService.get(`classroom:${classroomId}`);
    if (cached) return cached;

    const classroom = await this.classrooms.findOne({
      where: { id: classroomId },
      relations: ['teacher', 'enrollments.student', 'assignments', 'notes']
    });
    
    await this.cacheService.set(`classroom:${classroomId}`, classroom, 300);
    return classroom;
  }
}
```

---

## **EXECUTION MANDATE**

Build ClassHub V2 with:
1. ✅ Zero incomplete features
2. ✅ Complete authentication & authorization
3. ✅ All 50+ API endpoints
4. ✅ Real-time WebSocket features
5. ✅ AI Tutor with Gemini integration
6. ✅ Complete database schema
7. ✅ Docker & Kubernetes ready
8. ✅ 80%+ test coverage
9. ✅ Production security hardened
10. ✅ Comprehensive documentation

**No shortcuts. No placeholders. No "coming soon." Build complete, working, production-ready code.**

---

**Version:** 2.0.0-Production  
**Status:** Ready for VibreCoding Agent  
**Last Updated:** December 27, 2025  
**Target Deployment:** AWS/Azure/GCP within 48 hours of build  

