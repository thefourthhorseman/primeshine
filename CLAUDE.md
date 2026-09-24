# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Ontology: Technician (formerly "maid")

Service providers are called **technicians** in every user-facing surface
(UI labels, emails, SMS, app copy) and in ALL NEW code (variables, routes,
components — e.g. `/api/technicians`, `technicianId`). Legacy identifiers
(`MaidProfile`, `maidId`, `/api/maids`, `/maid-portal`, DB tables) are kept
for stability and aliased where needed — never introduce NEW code using the
legacy word. The mobile app (`primeshine-tech/`) is technician-vocabulary
throughout; its `lib/api/adapter.ts` is the only place that translates to
the legacy wire format.

## Project Overview

PrimeShine is a multi-tenant cleaning service platform that connects homeowners with cleaning service providers. The application consists of two separate repositories:
- Frontend: React application (`primeshine-front/`) - https://github.com/thefourthhorseman/primeshine-front.git
- Backend: Node.js/Express API (`primeshine-back/`) - https://github.com/thefourthhorseman/primeshine-back.git


### Current Architecture Status (Latest)
- **Job Management System**: Fully implemented with `Job`, `JobAssignment`, and `JobTask` models
- **Team Management**: Team and TeamMember models for maid organization
- **Service Areas**: Geographic service area management with database-driven boundaries
- **HouseCallPro Integration**: External job sync capabilities via API integration
- **Enhanced Maid Profiles**: Comprehensive onboarding with JSONB schema validation
- **Multi-Context State Management**: Separate contexts for Booking, Quote, Questionnaire, Auth, Tenant, and ServiceCatalog

## Development Commands

### Frontend (primeshine-front/)
- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run start` - Serve production build
- `npm test` - Run tests
- `npm run test:watch` - Run tests in watch mode
- `npm run test:coverage` - Run tests with coverage
- `npm run test:ci` - Run tests in CI mode

### Backend (primeshine-back/)
- `npm run dev` - Start development server with nodemon
- `npm run start` - Start production server
- `npm test` - Run all tests
- `npm run test:watch` - Run tests in watch mode
- `npm run test:coverage` - Run tests with coverage
- `npm run test:unit` - Run unit tests only
- `npm run test:integration` - Run integration tests only
- `npm run test:ci` - Run tests in CI mode

## Architecture Overview

### Database Architecture
- **PostgreSQL** database with Sequelize ORM
- **Multi-tenant** architecture with tenant isolation
- **Models** located in `primeshine-back/src/models/`
- **Migrations** in `primeshine-back/src/migrations/`
- **Seeders** in `primeshine-back/src/seeders/`

### Key Database Models
- **User** - Base user authentication with role-based access
- **MaidProfile** - Service provider profiles with comprehensive JSONB-validated onboarding
- **ClientProfile** - Customer profiles with address and contact information
- **Booking** - Service bookings with quotation integration and draft status support
- **Quotation** - Service quotes and pricing with customer name splitting
- **ServiceCatalog** - Available services with checklist and pricing data
- **ServiceArea** - Geographic boundaries for service delivery areas
- **Team/TeamMember** - Team organization and maid assignment management
- **Job/JobAssignment/JobTask** - Complete job lifecycle management with progress tracking
- **QuestionnaireProgress** - Lead tracking and questionnaire responses with session-based storage
- **TenantProfile** - Multi-tenant configuration with payment method support
- **Payment** - Enhanced payment tracking with detailed metadata

### API Architecture
- **REST API** with Express.js
- **Controllers** in `primeshine-back/src/controllers/`
- **Routes** in `primeshine-back/src/routes/` including:
  - `jobRoutes.js` - Job management and assignment
  - `maidRoutes.js` - Maid profile and onboarding
  - `teamRoutes.js` - Team organization
  - `serviceAreaRoutes.js` - Geographic service boundaries
  - `externalJobsRoutes.js` - HouseCallPro integration
  - `autoAssignmentRoutes.js` - Automated job assignment
- **Middleware** for authentication, validation, and multi-tenant isolation
- **Swagger documentation** configured with comprehensive API specs
- **External Integrations**: HouseCallPro API sync, Stripe payments, AWS S3 storage

### Frontend Architecture
- **React 18** with functional components and hooks
- **Material-UI v5** with emotion styling system
- **Context API** for state management:
  - `BookingContext` - Booking flow with auto-save and validation
  - `AuthContext` - User authentication and profile management
  - `TenantContext` - Multi-tenant switching and configuration
  - `QuoteContext` - Independent quote request handling
  - `QuestionnaireContext` - Database persistence with session tracking
  - `ServiceCatalogContext` - Service catalog caching and management
- **Multi-step forms** with validation, auto-save, and progress tracking
- **Component Structure**:
  - `components/questionnaire/` - JSON-driven questionnaire engine
  - `components/booking/` - Booking flow components
  - `components/maid-onboarding/` - Maid profile setup wizard
  - `components/JobManagement/` - Job dashboard and interface
  - `components/cleaning-tips/` - Content management system
- **Google Maps Integration** with geocoding and location services
- **Framer Motion** for animations and transitions
- **Separated Quote and Booking Systems** - Quote requests independent of booking creation

### Key Frontend Patterns
- **Booking Flow** - Multi-step wizard with progress tracking and draft support
- **Quote Request Flow** - Independent questionnaire system with database persistence and email delivery
- **Maid Onboarding** - Comprehensive profile setup wizard with JSONB validation
- **Job Management** - Dashboard for service providers with Google Maps visualization
- **Team Management** - Team creation and maid assignment interface
- **Service Area Management** - Geographic boundary configuration with map selection
- **Authentication** - JWT-based with role-based access and tenant isolation
- **HouseCallPro Integration** - External job sync with status mapping

## Logging 

### Debug Logging
- To log debug messages, use `logger.log` from the utility located at `@primeshine-front/src/utils/logger.js`

## Existing Reusable Components

### Questionnaire System
- **QuestionnaireEngine** (`src/components/questionnaire/QuestionnaireEngine.js`) - JSON-driven questionnaire engine with validation and auto-save
- **Question Types**: TextQuestion, EmailQuestion, SelectQuestion, SliderQuestion, DateQuestion, ConditionalQuestion, TextareaQuestion, MultiSelectQuestion, PhoneQuestion, StepperQuestion
- **QuoteQuestionnaireAdapter** (`src/components/questionnaire/QuoteQuestionnaireAdapter.js`) - Quote-specific questionnaire with database persistence and email delivery
- **BookingQuestionnaireAdapter** (`src/components/questionnaire/BookingQuestionnaireAdapter.js`) - Booking flow questionnaire integration
- **QuestionnaireContext** (`src/contexts/QuestionnaireContext.js`) - Database persistence for questionnaire responses with session tracking

### Job Management Components
- **MaidJobInterface** (`src/components/JobManagement/MaidJobInterface.js`) - Job dashboard for service providers
- **JobDashboard** (`src/components/JobManagement/JobDashboard.js`) - Administrative job overview
- **MaidMapDisplay** (`src/components/MaidMapDisplay.jsx`) - Google Maps integration with maid locations, job visualization, distance calculations, and job value totals

### Team and Maid Management
- **ComprehensiveOnboardingView** (`src/components/ComprehensiveOnboardingView.js`) - Complete maid onboarding interface
- **MaidStatusToggle** (`src/components/MaidStatusToggle.js`) - Active/inactive status management
- **ServiceAreaSelector** (`src/components/ServiceAreaSelector.js`) - Multi-select checkbox component with API integration
- **InviteMaidDialog** (`src/components/InviteMaidDialog.js`) - Maid invitation interface

### Form Field Components
- **EmailAutocompleteTextField** (`src/components/EmailAutocompleteTextField.js`) - Enhanced email input with domain autocompletion (gmail.com, yahoo.com, etc.)
- **InlineEditableField** (`src/components/InlineEditableField.js`) - Comprehensive editable field with validation, loading states, and API integration
- **ServiceAreaSelector** (`src/components/ServiceAreaSelector.js`) - Multi-select checkbox component with API integration

### Password Components
- **Password Input Pattern** - Standardized password visibility toggle using `Visibility`/`VisibilityOff` icons (found in ProgressiveCustomerInfo, CustomerInfoStep, PersonalInfoStep)

### Phone Number Components
- **Phone Number Formatting** - `formatPhoneNumber` utility function for (XXX) XXX-XXXX format (used across multiple components)

### Progress Indicators
- **BookingProgress** (`src/components/booking/shared/BookingProgress.js`) - Step indicator with progress bar and step chips
- **LinearProgress Pattern** - Simple progress bar with percentage calculation

### Modal/Dialog Components
- **ForgotPasswordDialog** (`src/components/ForgotPasswordDialog.js`) - Complete forgot password flow with email validation
- **CustomerInfoModal** (`src/components/CustomerInfoModal.js`) - Modal wrapper with responsive design and engaging content
- **PhoneContactDialog** (`src/components/PhoneContactDialog.js`) - Contact dialog with phone formatting utilities

### Validation Utilities
- **Email Validation** - Email regex pattern: `/^[^\s@]+@[^\s@]+\.[^\s@]+$/` (used across multiple components)
- **Password Validation** - Minimum length validation (6-8 characters)
- **BookingContext Validation** (`src/contexts/BookingContext.js`) - `validateCurrentStep` function with comprehensive rules

### Loading States and Button Components
- **Loading Button Pattern** - `CircularProgress` with disabled state and loading text (consistent pattern)
- **Button with Icons** - Standard Material-UI pattern with startIcon/endIcon props

### Specialized Components
- **EditableMaidDetailsModal** - Advanced maid profile editing interface
- **ChecklistEditor** (`src/components/ChecklistEditor.js`) - Service checklist management
- **ServiceCatalogForm** (`src/components/ServiceCatalogForm.js`) - Service configuration interface
- **AdminEmailTool** (`src/components/AdminEmailTool.js`) - Administrative email utilities
- **CookieConsent** (`src/components/CookieConsent.js`) - GDPR compliance component
- **ErrorBoundary** (`src/components/ErrorBoundary.js`) - React error boundary for fault tolerance

### Content Management
- **Cleaning Tips Components** (`src/components/cleaning-tips/`) - Content management system:
  - `TipsHeader.js`, `TipsGrid.js`, `FeaturedTips.js`
  - Room-specific guides: `KitchenTips.js`, `BathroomTips.js`, `BedroomTips.js`, `LivingRoomTips.js`
  - `SeasonalTips.js`, `SeasonalGuides.js`, `ShortTermRentalGuides.js`
  - `CategoryNavigation.js`, `ProductTips.js`

### Context and State Management
- **BookingContext** (`src/contexts/BookingContext.js`) - Comprehensive form state management with auto-save, validation, step management
- **QuoteContext** (`src/contexts/QuoteContext.js`) - Independent quote request handling with email delivery
- **QuestionnaireContext** (`src/contexts/QuestionnaireContext.js`) - Database persistence for questionnaire responses with session-based storage
- **AuthContext** (`src/contexts/AuthContext.js`) - User authentication and profile management
- **TenantContext** (`src/contexts/TenantContext.js`) - Multi-tenant configuration and switching
- **ServiceCatalogContext** (`src/contexts/ServiceCatalogContext.js`) - Service catalog caching with 5-minute cache duration
- **useClientProfile Hook** (`src/hooks/useClientProfile.js`) - Client profile data fetching and updates
- **useProfessionals Hook** (`src/hooks/useProfessionals.js`) - Maid data management

## Quote vs Booking System Architecture

### Design Principle: Separated Concerns
The application maintains clear separation between quote requests and actual bookings to prevent premature booking creation.

### Quote Request Flow
- **Purpose**: Collect lead information and send email quotes
- **Components**: `QuoteQuestionnaireAdapter` + `QuoteContext`
- **Database**: Saves to `questionnaire_progress` table for lead tracking
- **API Endpoint**: `/api/contact/quick-quote` for email delivery
- **No Booking Creation**: Does not create booking records

### Booking Flow  
- **Purpose**: Create actual service bookings with payment
- **Components**: `BookingQuestionnaireAdapter` + `BookingContext` 
- **Database**: Creates records in `bookings` table
- **API Endpoint**: `/api/bookings/create-booking`
- **Payment Integration**: Handles payment processing and booking confirmation

### Usage Guidelines
- **"Get Quote" buttons**: Always use `QuoteQuestionnaireAdapter`
- **"Book Now" buttons**: Use `BookingQuestionnaireAdapter` or direct booking flow
- **Lead tracking**: Quote responses automatically save to database with session IDs
- **Data separation**: Quote and booking systems share no state or database records

## Latest Technology Stack (2024)

### Frontend Dependencies
- **React 18.2.0** with functional components and hooks
- **Material-UI 5.17.1** with emotion styling system
- **React Router Dom 6.22.1** for client-side routing
- **Framer Motion 12.16.0** for animations and micro-interactions
- **Google Maps API** via `@react-google-maps/api 2.20.6`
- **FullCalendar 6.1.17** for scheduling interfaces
- **Recharts 3.1.0** for data visualization
- **Stripe Integration** for payment processing
- **Axios 1.9.0** for API communication

### Backend Dependencies
- **Node.js/Express 4.18.2** REST API framework
- **PostgreSQL** with **Sequelize 6.37.7** ORM
- **JWT Authentication** with bcryptjs password hashing
- **AWS S3 Integration** for file storage
- **Stripe 18.2.1** for payment processing
- **Nodemailer 7.0.5** for email delivery
- **Node-cron 3.0.3** for scheduled tasks
- **Swagger** for API documentation
- **Twilio 5.8.0** for SMS notifications

### Development Tools
- **Jest 29.7.0** for testing (both frontend and backend)
- **Nodemon 3.0.2** for development server
- **Sequelize CLI** for database migrations
- **ESLint** with React app configuration

## Database Enhancements (Latest)

### New Database Features
- **JSONB Schema Validation** for maid profiles with comprehensive onboarding data
- **PostGIS Support** for geographic queries and service area boundaries
- **Multi-tenant Architecture** with tenant isolation at database level
- **External Integration Support** with HouseCallPro sync capabilities
- **Enhanced Audit Trail** with created_by/updated_by tracking
- **Draft Status Support** for bookings with non-committed states

### Migration Pattern
- Database uses numbered migration files with descriptive names
- JSONB fields use validation schemas located in `src/schemas/`
- Foreign key relationships use UUID primary keys
- Geographic data uses PostGIS POINT types for location storage

## Development Workflow (Latest)

### Testing Strategy
- **Unit Tests** for individual components and functions
- **Integration Tests** for API endpoints and database operations
- **Coverage Requirements**: 70% threshold for branches, functions, lines, and statements
- **CI Mode** testing for automated deployment pipelines

### Code Organization Patterns
- **Feature-based Folder Structure** (e.g., `components/questionnaire/`, `components/JobManagement/`)
- **Shared Utilities** in `src/utils/` with common formatting and validation functions
- **Context Providers** for global state management with caching strategies
- **Service Layer** separation between UI components and API calls

### General Guidelines
- **Always check for existing components** before creating new ones
- **Extract common patterns** into reusable utilities when found in 3+ places
- **Enhance existing components** rather than replacing them
- **Use consistent validation patterns** from BookingContext and existing utilities
- **Maintain quote/booking separation** - never mix quote and booking creation logic
- **Follow JSONB schema validation** for complex data structures
- **Implement proper error boundaries** for fault tolerance
- **Use PostGIS functions** for geographic calculations and queries

## HCP (HouseCallPro) Job Import Service

### Overview
The HCP Job Import Service synchronizes jobs from the HousecallPro API into the local PostgreSQL database. Located in `primeshine-back/src/services/hcpJobImportService.js`, it handles bi-directional sync with change detection, validation, and geocoding.

### Architecture

#### Core Service Files
- **Service**: `src/services/hcpJobImportService.js` - Main import logic
- **Controller**: `src/controllers/hcpImportController.js` - HTTP request handlers
- **Routes**: `src/routes/hcpImportRoutes.js` - API endpoint definitions
- **Model Methods**: `src/models/Job.js` - `createFromHCP()`, `updateFromHCP()`, `findByHCPId()`

#### API Endpoints
| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/admin/hcp/import-jobs` | POST | Execute full import with date range |
| `/api/admin/hcp/preview-import` | POST | Preview what would be imported (dry run) |
| `/api/admin/hcp/import-status` | GET | Get import statistics and history |
| `/api/admin/hcp/imported-jobs` | GET | List imported jobs with pagination |
| `/api/admin/hcp/extract-customers` | POST | Extract customer data from HCP jobs |

### How It Works

#### Import Flow (Sequential Process)
1. **Fetch from HCP API** (`fetchJobsFromHCP()`)
   - Calls HCP `/jobs` endpoint with date filters
   - Filters by work status: `scheduled`, `in_progress`, `completed`
   - Page size: 100 jobs per request
   - 15-second timeout per request

2. **Data Sanitization** (`sanitizeHcpJobData()`)
   - Validates field types (notes, description must be strings)
   - Uses `Job.sanitizeField()` for non-string fields
   - Ensures required fields exist with defaults

3. **Validation** (`validateHCPJob()`)
   - Required fields: `id`, `schedule.scheduled_start`
   - Validates date formats
   - Logs warnings for missing customer/address data

4. **Import/Update Decision** (`importSingleJob()`)
   - Checks existence via `Job.findByHCPId(hcpJobId)`
   - **If exists**: Compares scheduled dates, updates only if changed
   - **If new**: Creates via `Job.createFromHCP()`

5. **Geocoding** (Lazy Loading)
   - Attempts to geocode job address using Google Maps API
   - Non-blocking - errors don't fail import
   - Updates latitude/longitude in database

6. **Deletion Sync** (`handleJobDeletions()`)
   - Finds local HCP jobs within date range not in HCP response
   - Performs hard delete via `job.destroy()`

#### Data Storage Strategy
Jobs stored with dual representation:
- **Normalized fields**: `utcScheduledDate`, `status`, `jobType` (local schema)
- **Raw HCP data**: `hcpRawData` (JSONB) - Complete HCP API response
- **Source tracking**: `sourceSystem: 'HCP'`, `hcpJobId`, `hcpJobNumber`
- **Import timestamp**: `utcImportedAt`

### Current Strengths
✅ **Comprehensive validation** - Multi-level validation prevents bad data
✅ **Change detection** - Only updates when schedule changes (reduces database writes)
✅ **Deletion sync** - Removes jobs deleted in HCP
✅ **Preview mode** - Test imports before execution
✅ **Detailed logging** - Extensive debug information for troubleshooting
✅ **Error isolation** - Single job failures don't break entire import
✅ **Raw data preservation** - Stores complete HCP response for auditing
✅ **Geocoding integration** - Automated address geocoding

### Critical Improvements Needed

#### 1. Pagination Handling ⚠️ **HIGH PRIORITY** (Current Bug)
**Issue**: Only fetches first 100 jobs from HCP API (line 341 in `hcpJobImportService.js`)

**Current Code**:
```javascript
// Only gets first page
queryParams.append('page_size', '100');
const response = await axios.get(url, { headers });
return response.data.jobs || [];
```

**Solution**: Implement pagination loop
```javascript
async fetchJobsFromHCP(startDate, endDate) {
  const allJobs = [];
  let page = 1;
  let hasMore = true;

  while (hasMore) {
    const queryParams = new URLSearchParams();
    queryParams.append('page_size', '100');
    queryParams.append('page', page.toString());
    // ... other params

    const response = await axios.get(url, { headers, timeout: 15000 });
    const jobs = response.data.jobs || [];
    allJobs.push(...jobs);

    // Check pagination metadata or assume more if full page
    hasMore = jobs.length === 100;
    page++;

    logger.log(`[HCP Import] Page ${page}: ${jobs.length} jobs, Total: ${allJobs.length}`);
  }

  return allJobs;
}
```

**Impact**: Critical data loss - missing jobs beyond first 100
**Effort**: 2 hours

#### 2. No Scheduled Automation ⚠️ **HIGH PRIORITY**
**Issue**: Manual import only via API calls - no automatic syncing

**Solution**: Add cron job using `node-cron`

Create `/src/jobs/hcpSyncJob.js`:
```javascript
const cron = require('node-cron');
const hcpJobImportService = require('../services/hcpJobImportService');
const logger = require('../utils/logger');

// Run every 15 minutes
const scheduleHCPSync = () => {
  cron.schedule('*/15 * * * *', async () => {
    logger.log('[HCP Sync Job] Starting scheduled sync');

    try {
      const endDate = new Date();
      const startDate = new Date();
      startDate.setHours(startDate.getHours() - 24); // Last 24 hours

      const results = await hcpJobImportService.importJobsFromHCP({
        startDate: startDate.toISOString(),
        endDate: endDate.toISOString(),
        triggeredBy: 'cron'
      });

      logger.log(`[HCP Sync Job] Completed: ${results.imported} imported, ${results.updated} updated`);
    } catch (error) {
      logger.error('[HCP Sync Job] Failed:', error);
    }
  });

  logger.log('[HCP Sync Job] Scheduled to run every 15 minutes');
};

module.exports = { scheduleHCPSync };
```

Register in `server.js` or `app.js`:
```javascript
const { scheduleHCPSync } = require('./jobs/hcpSyncJob');

// After server initialization
scheduleHCPSync(); // Start cron job
```

**Impact**: Essential for production - enables automatic data sync
**Effort**: 3 hours

#### 3. Rate Limiting Not Implemented ⚠️ **MEDIUM PRIORITY**
**Issue**: No rate limit handling for HCP API (can get 429 errors)

**Solution**: Add retry logic with exponential backoff
```javascript
async fetchJobsFromHCP(startDate, endDate) {
  const maxRetries = 3;
  let attempt = 0;

  while (attempt < maxRetries) {
    try {
      const response = await axios.get(url, {
        headers,
        timeout: 15000
      });
      return response.data.jobs || [];

    } catch (error) {
      if (error.response?.status === 429) { // Rate limited
        attempt++;
        const delay = Math.pow(2, attempt) * 1000; // Exponential backoff: 2s, 4s, 8s
        logger.warn(`[HCP Import] Rate limited (429), retry ${attempt}/${maxRetries} in ${delay}ms`);

        if (attempt >= maxRetries) {
          throw new Error('Max retries exceeded due to rate limiting');
        }

        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
}
```

**Impact**: Prevents API failures during high-volume imports
**Effort**: 2 hours

#### 4. Limited Change Detection ⚠️ **MEDIUM PRIORITY**
**Issue**: Only detects schedule changes (line 136 in `hcpJobImportService.js`)

**Current Code**:
```javascript
// Only checks schedule
let hasScheduleChange = false;
if (hcpScheduledStart && existingScheduledStart) {
  hasScheduleChange = hcpScheduledStart.getTime() !== existingScheduledStart.getTime();
}
```

**Solution**: Comprehensive change detection
```javascript
detectChanges(hcpJob, existingJob) {
  const changes = {
    hasChanges: false,
    schedule: false,
    status: false,
    amount: false,
    address: false,
    notes: false,
    employees: false
  };

  // Schedule change
  const hcpSchedule = new Date(hcpJob.schedule?.scheduled_start);
  if (hcpSchedule.getTime() !== existingJob.utcScheduledDate?.getTime()) {
    changes.schedule = true;
    changes.hasChanges = true;
  }

  // Status change
  if (hcpJob.work_status !== existingJob.hcpRawData?.work_status) {
    changes.status = true;
    changes.hasChanges = true;
  }

  // Amount change
  if (hcpJob.total_amount !== existingJob.hcpRawData?.total_amount) {
    changes.amount = true;
    changes.hasChanges = true;
  }

  // Address change
  const hcpAddress = JSON.stringify(hcpJob.address);
  const existingAddress = JSON.stringify(existingJob.hcpRawData?.address);
  if (hcpAddress !== existingAddress) {
    changes.address = true;
    changes.hasChanges = true;
  }

  // Notes/description change
  if (hcpJob.description !== existingJob.hcpRawData?.description) {
    changes.notes = true;
    changes.hasChanges = true;
  }

  // Assigned employees change
  const hcpEmployees = JSON.stringify(hcpJob.assigned_employees || []);
  const existingEmployees = JSON.stringify(existingJob.hcpRawData?.assigned_employees || []);
  if (hcpEmployees !== existingEmployees) {
    changes.employees = true;
    changes.hasChanges = true;
  }

  return changes;
}
```

Update `importSingleJob()` to use comprehensive detection:
```javascript
if (existingJob) {
  const changes = this.detectChanges(hcpJob, existingJob);

  if (changes.hasChanges) {
    logger.log(`[HCP Import] Changes detected for job ${hcpJob.id}:`,
      Object.entries(changes).filter(([k, v]) => v === true && k !== 'hasChanges').map(([k]) => k)
    );
    await existingJob.updateFromHCP(hcpJob);
    operationType = 'updated';
  } else {
    // Skip - no changes
    results.skipped++;
    return;
  }
}
```

**Impact**: Improves data accuracy and sync completeness
**Effort**: 4 hours

#### 5. No Webhook Support ⚠️ **MEDIUM PRIORITY**
**Issue**: Polling-based only - can have up to 15-minute delay

**Solution**: Add webhook endpoint for real-time updates

Create `/src/routes/hcpWebhookRoutes.js`:
```javascript
const express = require('express');
const router = express.Router();
const crypto = require('crypto');
const hcpJobImportService = require('../services/hcpJobImportService');
const { Job } = require('../models');

// Verify HCP webhook signature
function verifyHCPSignature(signature, body) {
  const webhookSecret = process.env.HCP_WEBHOOK_SECRET;
  if (!webhookSecret) return false;

  const hash = crypto
    .createHmac('sha256', webhookSecret)
    .update(JSON.stringify(body))
    .digest('hex');

  return signature === hash;
}

/**
 * @swagger
 * /api/webhooks/hcp:
 *   post:
 *     summary: Receive job updates from HousecallPro webhooks
 *     tags: [HCP Webhooks]
 */
router.post('/hcp', async (req, res) => {
  try {
    // Verify webhook signature
    const signature = req.headers['x-hcp-signature'];
    if (!verifyHCPSignature(signature, req.body)) {
      return res.status(401).json({ error: 'Invalid signature' });
    }

    const { event, data } = req.body;

    switch(event) {
      case 'job.created':
      case 'job.updated':
        const results = {};
        await hcpJobImportService.importSingleJob(
          data,
          process.env.REACT_APP_TENANT_ID || 1,
          process.env.REACT_APP_CITY_CODE || 'MCO',
          results
        );
        break;

      case 'job.deleted':
        await Job.destroy({ where: { hcpJobId: data.id } });
        break;
    }

    res.json({ success: true, event });
  } catch (error) {
    console.error('[HCP Webhook] Error:', error);
    res.status(500).json({ error: 'Webhook processing failed' });
  }
});

module.exports = router;
```

Register in main app:
```javascript
const hcpWebhookRoutes = require('./routes/hcpWebhookRoutes');
app.use('/api/webhooks', hcpWebhookRoutes);
```

Configure in HCP dashboard: `https://yourdomain.com/api/webhooks/hcp`

**Impact**: Real-time job updates instead of 15-minute polling delay
**Effort**: 6 hours

#### 6. No Bulk Operations Optimization ⚠️ **LOW PRIORITY**
**Issue**: Sequential processing (line 60 in `hcpJobImportService.js`)

**Current Code**:
```javascript
// One at a time - slow for large imports
for (const hcpJob of hcpJobs) {
  await this.importSingleJob(hcpJob, tenantId, cityCode, results);
}
```

**Solution**: Batch processing with concurrency control
```javascript
async importJobsFromHCP(options = {}) {
  // ... fetch jobs
  const hcpJobs = await this.fetchJobsFromHCP(startDate, endDate);

  // Process in batches of 10 for optimal performance
  const batchSize = 10;
  for (let i = 0; i < hcpJobs.length; i += batchSize) {
    const batch = hcpJobs.slice(i, i + batchSize);

    await Promise.all(
      batch.map(hcpJob =>
        this.importSingleJob(hcpJob, tenantId, cityCode, results)
          .catch(error => {
            logger.error(`[HCP Import] Error importing job ${hcpJob.id}:`, error);
            results.errors++;
            results.details.push({
              hcpJobId: hcpJob.id,
              status: 'error',
              message: error.message
            });
          })
      )
    );

    logger.log(`[HCP Import] Processed batch ${Math.floor(i / batchSize) + 1}, total: ${i + batch.length}/${hcpJobs.length}`);
  }

  // ... handle deletions
}
```

**Impact**: 5-10x faster imports for large datasets
**Effort**: 2 hours

#### 7. Missing Import History Tracking ⚠️ **LOW PRIORITY**
**Issue**: No audit trail of imports - can't see historical import performance

**Solution**: Create `HCPImportHistory` model

**Migration** (`migrations/YYYYMMDDHHMMSS-create-hcp-import-history.js`):
```javascript
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.createTable('hcp_import_history', {
      id: {
        type: Sequelize.UUID,
        defaultValue: Sequelize.UUIDV4,
        primaryKey: true
      },
      startDate: {
        type: Sequelize.DATE,
        allowNull: false
      },
      endDate: {
        type: Sequelize.DATE,
        allowNull: false
      },
      totalFetched: Sequelize.INTEGER,
      imported: Sequelize.INTEGER,
      updated: Sequelize.INTEGER,
      skipped: Sequelize.INTEGER,
      deleted: Sequelize.INTEGER,
      errors: Sequelize.INTEGER,
      duration: Sequelize.INTEGER, // milliseconds
      triggeredBy: {
        type: Sequelize.ENUM('manual', 'cron', 'webhook'),
        allowNull: false
      },
      status: {
        type: Sequelize.ENUM('in_progress', 'completed', 'failed'),
        allowNull: false
      },
      errorDetails: Sequelize.JSONB,
      createdAt: Sequelize.DATE,
      updatedAt: Sequelize.DATE
    });
  },

  down: async (queryInterface, Sequelize) => {
    await queryInterface.dropTable('hcp_import_history');
  }
};
```

**Model** (`models/HCPImportHistory.js`):
```javascript
const { Model, DataTypes } = require('sequelize');

class HCPImportHistory extends Model {
  static init(sequelize) {
    return super.init({
      id: {
        type: DataTypes.UUID,
        defaultValue: DataTypes.UUIDV4,
        primaryKey: true
      },
      startDate: { type: DataTypes.DATE, allowNull: false },
      endDate: { type: DataTypes.DATE, allowNull: false },
      totalFetched: DataTypes.INTEGER,
      imported: DataTypes.INTEGER,
      updated: DataTypes.INTEGER,
      skipped: DataTypes.INTEGER,
      deleted: DataTypes.INTEGER,
      errors: DataTypes.INTEGER,
      duration: DataTypes.INTEGER,
      triggeredBy: {
        type: DataTypes.ENUM('manual', 'cron', 'webhook'),
        allowNull: false
      },
      status: {
        type: DataTypes.ENUM('in_progress', 'completed', 'failed'),
        allowNull: false
      },
      errorDetails: DataTypes.JSONB
    }, {
      sequelize,
      tableName: 'hcp_import_history',
      timestamps: true
    });
  }
}

module.exports = HCPImportHistory;
```

**Usage in Service**:
```javascript
async importJobsFromHCP(options = {}) {
  const startTime = Date.now();
  const importRecord = await HCPImportHistory.create({
    startDate: options.startDate,
    endDate: options.endDate,
    status: 'in_progress',
    triggeredBy: options.triggeredBy || 'manual'
  });

  try {
    // ... existing import logic

    await importRecord.update({
      totalFetched: results.totalFetched,
      imported: results.imported,
      updated: results.updated,
      skipped: results.skipped,
      deleted: results.deleted,
      errors: results.errors,
      duration: Date.now() - startTime,
      status: 'completed',
      errorDetails: results.details.filter(d => d.status === 'error')
    });

    return results;
  } catch (error) {
    await importRecord.update({
      status: 'failed',
      duration: Date.now() - startTime,
      errorDetails: { message: error.message, stack: error.stack }
    });
    throw error;
  }
}
```

**Impact**: Better monitoring and debugging capabilities
**Effort**: 3 hours

### Implementation Priority

| Priority | Improvement | Impact | Effort | Status |
|----------|------------|--------|--------|--------|
| 🔴 **HIGH** | 1. Pagination handling | Critical - prevents data loss | 2 hours | ❌ Not started |
| 🔴 **HIGH** | 2. Scheduled automation | Essential for production | 3 hours | ❌ Not started |
| 🟡 **MEDIUM** | 3. Rate limiting | API stability | 2 hours | ❌ Not started |
| 🟡 **MEDIUM** | 4. Enhanced change detection | Data accuracy | 4 hours | ❌ Not started |
| 🟡 **MEDIUM** | 5. Webhook support | Real-time sync | 6 hours | ❌ Not started |
| 🟢 **LOW** | 6. Bulk optimization | Performance (5-10x faster) | 2 hours | ❌ Not started |
| 🟢 **LOW** | 7. Import history tracking | Audit trail & monitoring | 3 hours | ❌ Not started |

**Total Estimated Effort**: ~22 hours (3 days for experienced developer)

### Recommended Implementation Phases

**Phase 1 - Critical Fixes** (Week 1)
1. Fix pagination bug - prevents data loss
2. Add cron job automation - makes system production-ready

**Phase 2 - Stability** (Week 2)
3. Implement rate limiting - prevents API failures
4. Add enhanced change detection - improves data accuracy

**Phase 3 - Advanced Features** (Week 3-4)
5. Implement webhook support - enables real-time sync
6. Add bulk optimization - improves performance
7. Create import history tracking - enables monitoring

### Testing Recommendations
- **Unit tests** for each improvement
- **Integration tests** for full import flow
- **Load testing** for bulk operations
- **Webhook testing** using HCP sandbox environment
