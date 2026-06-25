# PrimeShine Job Management Suite

A comprehensive job management system that converts bookings into trackable jobs with task lists, team assignments, and progress monitoring.

## Overview

The Job Management Suite extends PrimeShine's booking system by creating structured jobs with predefined tasks based on service type. Each job tracks progress, manages team assignments, and ensures quality completion.

## High-Level Architecture

```
Booking (CONFIRMED) → Job → Tasks + Assignments → Completion
```

### Core Components

1. **Job**: Main work unit created from a confirmed booking
2. **JobTask**: Individual checklist items that must be completed
3. **JobAssignment**: Team member assignments with roles (LEAD/ASSISTANT)
4. **JobTaskTemplate**: Predefined task templates for different job types

## Database Schema

### Jobs Table
- **Purpose**: Central job management and progress tracking
- **Key Fields**:
  - `jobNumber`: User-friendly identifier (e.g., "JOB123456")
  - `status`: NOT_STARTED → IN_PROGRESS → COMPLETED → CANCELLED
  - `progressPercentage`: Auto-calculated based on completed tasks
  - `scheduledDate/Time`: When the job should be performed
  - `actualStartTime/EndTime`: Track actual work duration

### JobTasks Table
- **Purpose**: Individual checklist items for job completion
- **Key Fields**:
  - `category`: Groups tasks (General, Bedrooms, Bathrooms, Kitchen, etc.)
  - `isRequired`: Must be completed before job can be marked complete
  - `isCompleted`: Completion status
  - `estimatedMinutes`: Time estimate for task
  - `completedBy`: Which team member completed the task

### JobAssignments Table
- **Purpose**: Team management and maid assignments
- **Key Fields**:
  - `role`: LEAD (job supervisor) or ASSISTANT
  - `status`: ASSIGNED → ACCEPTED/DECLINED → COMPLETED
  - `assignedAt`: When assignment was made

### JobTaskTemplates Table
- **Purpose**: Predefined task lists for different job types
- **Task Sources**: Based on `homePageData.json` task definitions
- **Job Types**: RESIDENTIAL, VACATION_RENTAL, COMMERCIAL

## Task Management System

### Task Categories by Job Type

#### Residential Cleaning
- **General**: Surface cleaning, vacuuming, floor care
- **Bedrooms**: Bed making, furniture dusting, organization  
- **Bathrooms**: Sanitization, restocking, deep cleaning
- **Kitchen**: Appliance cleaning, counter care, dishwashing
- **Laundry**: Linen washing, towel management, care

#### Vacation Rental Cleaning
- **Guest-Ready Essentials**: Linens, bathrooms, kitchen, floors
- **Guest Experience Details**: Amenities, restocking, sanitization
- **Property Protection**: Damage inspection, security, documentation

#### Commercial Cleaning
- **General**: Trash removal, vacuuming, floor care
- **Bathrooms**: Sanitization and restocking
- **Common Areas**: Surface cleaning and maintenance

### Task Generation
Tasks are automatically created from templates when a job is created. The system:
1. Looks up templates for the job type and tenant
2. Creates individual tasks with proper ordering
3. Sets estimated time and priority levels
4. Updates job progress tracking

## API Endpoints

### Job Management
```javascript
GET    /api/jobs                     // List jobs with filtering
GET    /api/jobs/:id                 // Get job details with tasks/team
POST   /api/jobs                     // Create job from booking
PATCH  /api/jobs/:id                 // Update job details
POST   /api/jobs/:id/start           // Start job execution
POST   /api/jobs/:id/complete        // Complete job (validates required tasks)
POST   /api/jobs/:id/cancel          // Cancel job with reason
```

### Task Management
```javascript
GET    /api/jobs/:id/tasks           // Get job task list
GET    /api/jobs/:id/tasks/progress  // Get completion statistics
PATCH  /api/jobs/:jobId/tasks/:taskId // Mark task complete/incomplete
POST   /api/jobs/:id/tasks           // Add custom task
DELETE /api/jobs/:jobId/tasks/:taskId // Remove task
```

### Team Assignment
```javascript
POST   /api/jobs/:id/assign          // Assign maid to job
GET    /api/jobs/:id/team            // Get job team
PATCH  /api/jobs/:jobId/assignments/:assignmentId // Update assignment status
GET    /api/jobs/maid/:maidId        // Get maid's assigned jobs
```

## Business Logic

### Job Status Lifecycle
1. **NOT_STARTED**: Job created, waiting to begin
2. **IN_PROGRESS**: Job started, tasks being completed
3. **COMPLETED**: All required tasks finished
4. **CANCELLED**: Job cancelled for various reasons

### Task Completion Rules
- **Required Tasks**: Must be completed before job can finish
- **Optional Tasks**: Enhance service quality but not mandatory
- **Progress Tracking**: Auto-calculated percentage based on completed vs total tasks
- **Validation**: Job completion blocked until all required tasks done

### Team Management
- **Lead Role**: Primary responsible person, can start/complete job
- **Assistant Role**: Support team member, can complete tasks
- **Assignment Workflow**: ASSIGNED → ACCEPTED/DECLINED → COMPLETED
- **Auto-Assignment**: Algorithm to assign available maids based on location/availability

## Frontend Components

### Admin Dashboard (`JobDashboard.js`)
- **Statistics Cards**: Total, active, completed, scheduled jobs
- **Job List**: Filterable by status with action buttons
- **Progress Monitoring**: Visual progress bars and completion percentages
- **Real-time Updates**: Refresh capability and status changes

### Maid Interface (`MaidJobInterface.js`)
- **Mobile-Optimized**: Touch-friendly task completion
- **Task Categories**: Expandable sections with progress bars
- **Task Actions**: Checkbox completion, note addition
- **Job Controls**: Start job, complete job, add notes
- **Validation**: Prevents completion until required tasks done

## Integration Points

### Booking System Integration
- **Auto-Creation**: Jobs created when bookings reach CONFIRMED status
- **Data Transfer**: Customer info, service details, scheduling transferred
- **Status Sync**: Job completion updates booking status

### Notification System (Ready for Implementation)
- **Job Creation**: Notify assigned maids and admin
- **Task Progress**: Updates for supervisors and quality control
- **Completion**: Customer notification and review requests
- **Cancellation**: Team notification and rebooking assistance

### Quality Control
- **Quality Checks**: Post-completion review and approval
- **Photo Documentation**: Task completion photos (infrastructure ready)
- **Performance Tracking**: Maid efficiency and quality metrics
- **Customer Feedback**: Integration with review system

## Deployment and Setup

### Database Migration
```bash
# Run migration to create tables
npx sequelize-cli db:migrate

# Populate task templates
npx sequelize-cli db:seed --seed 20250108000001-job-task-templates.js
```

### Environment Configuration
No additional environment variables required. Uses existing:
- `REACT_APP_TENANT_ID`: Multi-tenant job isolation
- `REACT_APP_CITY_CODE`: Geographic service area filtering

### Model Registration
Models are automatically loaded by Sequelize. Ensure imports in routes:
```javascript
const { Job, JobTask, JobAssignment, JobTaskTemplate } = require('../models');
```

## Usage Examples

### Creating a Job from Booking
```javascript
// When booking is confirmed
const job = await JobController.createJobFromBooking(bookingId, 'RESIDENTIAL');

// Automatically creates tasks from templates
// Assigns lead maid if specified in booking
// Sets up progress tracking
```

### Maid Task Completion
```javascript
// Mark task as complete
await JobTask.findByPk(taskId).markComplete(maidId, notes, photoUrl);

// Job progress automatically updates
// Completion validation runs when all required tasks done
```

### Team Assignment
```javascript
// Assign maid to job
const assignment = await JobAssignment.assignMaidToJob(jobId, maidId, 'LEAD');

// Maid receives notification (when implemented)
// Assignment tracked with acceptance workflow
```

## Performance Considerations

### Database Optimization
- **Indexes**: Strategic indexing on jobId, status, scheduledDate
- **Pagination**: Job list endpoints support limit/offset
- **Eager Loading**: Optimized queries with proper includes

### Task Template Caching
- Templates loaded once per job creation
- Minimal database calls for task generation
- Template reuse across similar jobs

### Progress Calculation
- Real-time progress updates on task completion
- Cached statistics for dashboard performance
- Efficient category-based progress tracking

## Future Enhancements

### Mobile App Integration
- Native mobile task completion interface
- Offline task completion with sync
- Photo capture and GPS tracking

### Advanced Scheduling
- Route optimization for multi-job assignments
- Time slot management and scheduling conflicts
- Recurring job automation

### Analytics and Reporting
- Performance metrics and trends
- Efficiency analysis by maid/team
- Quality score tracking over time
- Customer satisfaction correlation

### AI-Powered Features
- Intelligent task time estimation
- Quality prediction based on task completion
- Optimal team composition suggestions

## Maintenance

### Task Template Management
- Admin interface for template editing
- Version control for template changes
- A/B testing for task effectiveness

### Data Cleanup
- Archive completed jobs older than 1 year
- Clean up cancelled jobs and associated tasks
- Performance monitoring and optimization

### Error Handling
- Graceful failure recovery
- Comprehensive logging for debugging
- User-friendly error messages

## Security and Permissions

### Role-Based Access
- **Admins**: Full job management access
- **Maids**: Only assigned jobs and task completion
- **Clients**: Read-only access to their job progress

### Data Protection
- Tenant-isolated job data
- Encrypted sensitive information
- Audit trail for all job actions

The Job Management Suite provides a comprehensive foundation for managing cleaning service delivery, ensuring quality completion, and maintaining customer satisfaction through structured task management and team coordination.