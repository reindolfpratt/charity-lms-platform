# Charity LMS Platform

> **Multi-tenant Learning Management System for charities built with Next.js, TypeScript, and Supabase**

A production-ready SaaS platform enabling multiple charities to create courses, enroll learners, track progress, and measure impact—all with strict tenant isolation and role-based access control.

## 🌟 Key Features

- ✅ **Multi-tenant Architecture** - Complete data isolation between organizations
- ✅ **Row-Level Security (RLS)** - Database-level security enforced by Supabase
- ✅ **Role-Based Access** - Four distinct roles: Org Admin, Learning Manager, Tutor, Learner
- ✅ **Course Authoring** - Rich course builder with sections, lessons (video/file/embed/rich text), and quizzes
- ✅ **Flexible Enrollment** - Manual, bulk CSV import, or self-enrollment via shareable links
- ✅ **Progress Tracking** - Real-time tracking of lesson completion and quiz scores
- ✅ **Impact Reporting** - Project tags for grant/funder reporting with aggregate metrics
- ✅ **Automated Notifications** - Email reminders for deadlines, overdue courses, and completions
- ✅ **Mobile-Friendly** - Responsive design optimized for volunteers and staff on the go

## 🏗️ Architecture

### Tech Stack

- **Frontend**: Next.js 14 (App Router), React, TypeScript, Tailwind CSS
- **Backend**: Supabase (PostgreSQL, Auth, Storage, Edge Functions)
- **Deployment**: Vercel (frontend) + Supabase Cloud (backend)
- **Authentication**: Supabase Auth (email/password, magic links, future SSO)

### Database Schema

The system uses a single-database multi-tenant architecture with `tenant_id` on all scoped tables:

**Core Tables:**
- `tenants` - Charity organizations
- `profiles` - User profiles with roles
- `courses` - Course metadata
- `course_sections` - Course sections
- `lessons` - Individual lessons (video, file, embed, rich text)
- `quizzes` - Assessments with pass marks
- `quiz_questions` - MCQ questions
- `enrolments` - User course enrollments
- `lesson_progress` - Progress tracking
- `quiz_attempts` - Quiz submission records
- `project_tags` - For impact reporting
- `notifications` - Notification queue

## 📁 Repository Structure

```
charity-lms-platform/
├── app/                      # Next.js App Router
│   ├── (auth)/              # Auth pages (login, signup, invite)
│   ├── (dashboard)/         # Protected dashboard routes
│   │   ├── dashboard/       # Learner dashboard
│   │   ├── courses/         # Course viewer
│   │   ├── admin/           # Org admin pages
│   │   └── manager/         # Learning manager pages
│   └── api/                 # API routes
├── components/              # React components
│   ├── ui/                  # UI primitives
│   ├── course/              # Course-related components
│   └── dashboard/           # Dashboard components
├── lib/
│   ├── supabase/            # Supabase clients
│   ├── services/            # Data access layer
│   ├── hooks/               # React hooks
│   └── database.types.ts    # Generated TypeScript types
├── supabase/
│   ├── migrations/          # SQL migration files
│   └── functions/           # Edge functions
├── public/                  # Static assets
└── docs/                    # Extended documentation
    ├── SETUP.md             # Detailed setup guide
    ├── DEPLOYMENT.md        # Deployment instructions
    ├── DATABASE.md          # Database schema documentation
    └── API.md               # API documentation
```

## 🚀 Quick Start

### Prerequisites

- Node.js 18+ and npm
- Supabase account (free tier works)
- Git

### 1. Clone Repository

```bash
git clone https://github.com/reindolfpratt/charity-lms-platform.git
cd charity-lms-platform
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Set Up Supabase

1. Create a new project at [supabase.com](https://supabase.com)
2. Run migrations in order from `supabase/migrations/`:
   - `001_core_schema.sql` - Core tables and enums
   - `002_functions.sql` - Helper functions
   - `003_rls_policies.sql` - Row-Level Security policies
   - `004_storage.sql` - Storage buckets and policies

3. In Supabase Dashboard > SQL Editor, paste and execute each migration file

### 4. Configure Environment Variables

Create `.env.local` in project root:

```bash
NEXT_PUBLIC_SUPABASE_URL=your-project-url.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

Get these values from Supabase Dashboard > Project Settings > API

### 5. Run Development Server

```bash
npm run dev
```

Visit [http://localhost:3000](http://localhost:3000)

### 6. Create Your First Tenant

1. Navigate to `/signup`
2. Enter organization details
3. You'll be created as the first `ORG_ADMIN`
4. Start inviting users from `/admin/users`

## 📖 Complete Code Implementation

The full implementation includes:

### Database Migrations (3 files)

See [`docs/DATABASE.md`](docs/DATABASE.md) for complete SQL migrations:
- Migration 001: Core schema with all tables, enums, indexes, and triggers
- Migration 002: Helper functions (tenant context, invitations, completion calculations)
- Migration 003: Row-Level Security policies for all tables
- Migration 004: Storage bucket setup and policies

### TypeScript Services (Data Access Layer)

See [`docs/API.md`](docs/API.md) for full service implementations:
- `TenantService` - Tenant management, logo uploads
- `UserService` - User CRUD, invitations, role management
- `CourseService` - Course authoring, publishing
- `EnrolmentService` - Enrollment management, bulk imports
- `ReportService` - Analytics and exports
- `LessonService` - Lesson progress tracking
- `QuizService` - Quiz attempts and scoring

### Next.js Pages & Components

See [`docs/SETUP.md`](docs/SETUP.md) for complete page implementations:

**Authentication:**
- Login page with magic link support
- Signup with tenant creation
- Invite acceptance flow

**Learner Dashboard:**
- Course list with progress indicators
- Deadline tracking
- Course player with navigation
- Quiz interface with immediate feedback

**Learning Manager:**
- Course builder with drag-drop section/lesson ordering
- Rich text editor for content
- Quiz builder with multiple choice questions
- Enrollment management
- CSV bulk import
- Reports dashboard

**Org Admin:**
- User management
- Role assignment
- Tenant settings and branding
- Project tag management

### Edge Functions

See [`supabase/functions/`](supabase/functions/) for:
- `process-notifications` - Daily cron job for reminder emails
- `handle-enrollment` - Post-enrollment webhook
- `generate-certificate` - PDF certificate generation (future)

## 🔒 Security & Multi-Tenancy

### Row-Level Security (RLS)

All tenant-scoped tables enforce RLS policies:

```sql
-- Example: Courses policy
CREATE POLICY "Users can view courses in their tenant"
  ON courses FOR SELECT
  USING (
    tenant_id = current_tenant_id() AND
    (status = 'PUBLISHED' OR current_user_role() IN ('ORG_ADMIN', 'LEARNING_MANAGER'))
  );
```

### Tenant Context

Every authenticated request automatically resolves tenant context:

```typescript
// Helper function in Postgres
CREATE FUNCTION current_tenant_id() RETURNS UUID AS $$
  SELECT tenant_id FROM profiles WHERE id = auth.uid();
$$ LANGUAGE SQL STABLE;
```

### Role-Based Access

Four roles with hierarchical permissions:

1. **ORG_ADMIN** - Full tenant access, user management, settings
2. **LEARNING_MANAGER** - Course creation, enrollment, reports
3. **TUTOR** - Limited course editing on assigned courses
4. **LEARNER** - Course viewing and completion

## 📊 Reporting & Analytics

### Course Completion Reports

```typescript
// Get completion stats by course
const report = await reportService.getCourseCompletionReport();
// Returns: total enrolled, completed, in_progress, overdue, completion_rate
```

### Project Tag Impact Reports

Track learning outcomes by grant/funder:

```typescript
const impact = await reportService.getProjectTagReport();
// Returns: total learners, completions, learning hours per tag
```

### CSV Exports

Export learner data for external reporting:

```typescript
const data = await reportService.exportCourseData(courseId);
// Includes: learner details, enrollment dates, completion status, scores
```

## 📧 Notifications

### Automated Email Triggers

- **On Enrollment**: Welcome email with course link
- **3 Days Before Deadline**: Reminder with progress summary  
- **On Deadline**: Overdue notice
- **On Completion**: Congratulations with certificate (future)

### Implementation

Edge function runs daily via cron:

```typescript
// supabase/functions/process-notifications/index.ts
// Queries upcoming/overdue enrollments
// Creates notification records
// Sends via email provider (Resend/SendGrid)
```

## 🌐 Deployment

### Production Deployment

See [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md) for detailed instructions.

**Quick Deploy:**

1. **Supabase**: Already deployed (managed service)
2. **Vercel**:
   ```bash
   npm install -g vercel
   vercel --prod
   ```

3. Set environment variables in Vercel dashboard
4. Deploy edge functions:
   ```bash
   supabase functions deploy process-notifications
   ```

5. Set up cron job in Supabase Dashboard > Edge Functions

### Post-Deployment

- Configure custom domain
- Set up email provider (Resend recommended)
- Enable Supabase Auth providers (Google SSO, etc.)
- Configure storage CDN
- Set up monitoring (Sentry, LogRocket)

## 🧪 Testing

### RLS Policy Testing

```bash
npm run test:rls
```

Verifies:
- Users cannot access other tenants' data
- Role permissions are correctly enforced
- Public enrollment links work correctly

### Integration Tests

```bash
npm run test:integration
```

Tests:
- User invitation flow
- Course creation and publishing
- Enrollment and progress tracking
- Quiz submission and scoring

## 📚 Extended Documentation

- **[Setup Guide](docs/SETUP.md)** - Detailed local development setup
- **[Database Schema](docs/DATABASE.md)** - Complete SQL migrations and schema docs
- **[API Documentation](docs/API.md)** - Service layer and RPC functions
- **[Deployment Guide](docs/DEPLOYMENT.md)** - Production deployment checklist
- **[Contributing](docs/CONTRIBUTING.md)** - How to contribute to this project

## 🗺️ Roadmap

### v1.0 (Current)
- ✅ Core multi-tenant architecture
- ✅ Course authoring and enrollment
- ✅ Progress tracking and quizzes
- ✅ Basic reporting
- ✅ Email notifications

### v1.1 (Next)
- [ ] SCORM/xAPI support
- [ ] Video progress tracking (watch time)
- [ ] Discussion forums per course
- [ ] Certificate generation
- [ ] Advanced analytics dashboard

### v2.0 (Future)
- [ ] Mobile apps (React Native)
- [ ] Live virtual classrooms (WebRTC)
- [ ] Gamification (badges, leaderboards)
- [ ] AI content recommendations
- [ ] Offline mode with sync

## 🤝 Contributing

Contributions are welcome! See [`CONTRIBUTING.md`](docs/CONTRIBUTING.md) for guidelines.

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details.

## 💬 Support

- **Issues**: [GitHub Issues](https://github.com/reindolfpratt/charity-lms-platform/issues)
- **Discussions**: [GitHub Discussions](https://github.com/reindolfpratt/charity-lms-platform/discussions)
- **Email**: support@charity-lms.org

## 🙏 Acknowledgments

Built with:
- [Next.js](https://nextjs.org/)
- [Supabase](https://supabase.com/)
- [Tailwind CSS](https://tailwindcss.com/)
- [Radix UI](https://www.radix-ui.com/)

---

**Made with ❤️ for charities making a difference**
