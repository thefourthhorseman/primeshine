# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PrimeShine is a multi-tenant cleaning service platform that connects homeowners with cleaning service providers. The application consists of two separate repositories:
- Frontend: React application (`primeshine-front/`) - https://github.com/thefourthhorseman/primeshine-front.git
- Backend: Node.js/Express API (`primeshine-back/`) - https://github.com/thefourthhorseman/primeshine-back.git

## Development Commands

### Backend (primeshine-back/)
```bash
# Development server with hot reload
npm run dev

# Production server
npm start

# Run tests
npm test

# Database operations
npx sequelize-cli db:migrate          # Run pending migrations
npx sequelize-cli db:migrate:undo     # Undo last migration
npx sequelize-cli db:seed:all         # Run all seeders
npx sequelize-cli db:seed:undo:all    # Undo all seeders

# Generate new migration
npx sequelize-cli migration:generate --name migration-name

# Generate new seeder
npx sequelize-cli seed:generate --name seeder-name
```

### Frontend (primeshine-front/)
```bash
# Development server
npm run dev

# Production build
npm run build

# Serve production build locally (uses 'serve' package)
npm start

# Install dependencies and build (used in deployment)
npm run postinstall

# Run tests
npm test

# Run tests in watch mode
npm test -- --watch
```

## Architecture Overview

### Backend Architecture
- **Framework**: Express.js with Sequelize ORM
- **Database**: PostgreSQL
- **Authentication**: JWT with bcryptjs
- **Payment Processing**: Stripe integration
- **Main Entry**: `primeshine-back/src/server.js` → `primeshine-back/src/app.js`

**Key Backend Components:**
- **Models**: Sequelize models in `src/models/` (User, Booking, Quotation, MaidProfile, ClientProfile, etc.)
- **Controllers**: Business logic in `src/controllers/` 
- **Routes**: API endpoints in `src/routes/`
- **Middleware**: Authentication middleware in `src/middleware/auth.js`
- **Database**: Migrations in `src/migrations/`, seeders in `src/seeders/`

### Frontend Architecture  
- **Framework**: React 18 with Material-UI (MUI) 
- **Build Tool**: Create React App (react-scripts)
- **Routing**: React Router DOM v6
- **State Management**: React Context (AuthContext, TenantContext, BookingContext)
- **Styling**: Material-UI theme system with custom theme in `src/theme.js`
- **HTTP Client**: Axios for API communication
- **Calendar**: FullCalendar for scheduling interfaces
- **Animation**: Framer Motion for UI animations
- **Maps**: Google Maps API integration via `@react-google-maps/api`
- **Payment**: Stripe integration with `@stripe/react-stripe-js`
- **Main Entry**: `primeshine-front/src/index.js` → `primeshine-front/src/App.js`

**Key Frontend Components:**
- **Pages**: Main application pages in `src/pages/`
- **Components**: Reusable UI components in `src/components/`
- **Contexts**: Global state management in `src/contexts/`
- **Voucher Flow**: Multi-step voucher booking process in `src/components/voucherflow/`
- **Booking Flow**: Unified booking system in `src/components/booking/`
- **Hooks**: Custom React hooks in `src/hooks/`

### Multi-Tenant Architecture

PrimeShine implements a comprehensive multi-tenant architecture driven by environment variables and tenant-specific configurations stored in the `TenantProfile` table.

#### **Environment Variable-Driven Multi-Tenancy**
- **`REACT_APP_TENANT_ID`**: Core tenant identifier that determines which tenant instance the application serves
- **`REACT_APP_CITY_CODE`**: Geographic service area identifier (e.g., 'ORL' for Orlando, 'KIS' for Kissimmee)
- These variables control data isolation, tenant-specific configurations, and business logic throughout the entire application

#### **Data Isolation & Tenant-Specific Features**
- **User Registration**: All users are automatically assigned to the tenant based on `REACT_APP_TENANT_ID`
- **Booking Management**: Bookings are isolated by `tenantId` and `cityCode` for proper geographic service delivery
- **Maid Profiles**: Maid onboarding and profiles include tenant/city assignment for service area management
- **Payment Processing**: Payment configurations and supported methods vary by tenant
- **Contact Information**: Each tenant has specific contact details stored in `TenantProfile`

#### **TenantProfile Configuration Management**
The `TenantProfile` table controls critical tenant-specific settings:
- **Contact Information**: Tenant-specific email and phone number for customer communications
- **Customer Payment Methods**: Configurable list of payment methods available to customers (Stripe, PayPal, etc.)
- **Maid Payment Methods**: Configurable list of payment methods available to maids (Zelle, CashApp, Direct Deposit, etc.)
- **Service Areas**: Geographic boundaries and city codes served by the tenant
- **Business Rules**: Tenant-specific pricing, policies, and operational configurations

#### **Multi-Tenant Data Flow Examples**
```javascript
// User Registration - Auto-assigned tenant
const registrationData = {
  firstName, lastName, email, password,
  tenantId: process.env.REACT_APP_TENANT_ID || 'primeshine',
  role: 'MAID'
};

// Booking Creation - Tenant/city isolation
const bookingData = {
  ...customerData,
  tenantId: process.env.REACT_APP_TENANT_ID,
  cityCode: process.env.REACT_APP_CITY_CODE || 'ORL'
};

// Maid Onboarding - Multi-tenant profile
const maidProfileData = {
  ...onboardingData,
  tenantId: process.env.REACT_APP_TENANT_ID,
  cityCode: process.env.REACT_APP_CITY_CODE
};
```

#### **TenantContext State Management**
- **Tenant-aware components**: All major components access tenant configuration through TenantContext
- **Dynamic payment options**: Payment method lists are fetched from TenantProfile configuration
- **Geographic filtering**: Service areas and availability are filtered by cityCode
- **Contact integration**: Customer communications use tenant-specific contact information

### Key Features
- **Authentication**: User registration/login with role-based access
- **Quotation System**: Dynamic pricing for cleaning services
- **Booking Management**: Service scheduling and management
- **Maid Matching**: Algorithm to match clients with available service providers  
- **Payment Integration**: Stripe payment processing
- **Voucher System**: Promotional voucher booking flow
- **Multi-tenant Support**: Geographic service area management

## Unified Booking Flow Architecture

### Overview
The PrimeShine booking system uses a **unified 6-step booking flow** with progressive data collection. The system captures user data incrementally as they progress through the booking process, allowing for analytics and follow-up on incomplete bookings.

### Booking Flow Steps
1. **Customer Information** (guests only) - User registration/login
2. **Property Details** (guests only) - Address and property specifications
3. **Service Selection** - Choose cleaning service and add-ons
4. **Scheduling** - Select date, time, and frequency
5. **Payment** - Stripe payment processing
6. **Confirmation** - Booking confirmation and details

**Authenticated users** skip steps 1-2 if their profile data is complete, resulting in a 4-step flow.

### Frontend Architecture

**Main Components:**
- **`UnifiedBookingFlow.js`** - Main booking orchestrator and step navigation
- **`BookingContext.js`** - Global state management using useReducer pattern
- **Step Components** - Individual step implementations with validation
- **Progressive Auto-Save** - Automatic saving of partial booking data

**State Management:**
```javascript
// BookingContext provides unified state with:
- bookingState: Complete booking data object
- Auto-save functionality for authenticated users
- Step validation and navigation helpers
- Error handling and user feedback
- Multi-tenant flow control via flowStartedAsGuest flag
```

**Key Features:**
- **Progressive Data Collection**: Auto-saves at every step change
- **Multi-tenant Support**: Dynamic step flow based on user authentication
- **Real-time Validation**: Field-level and step-level validation
- **Error Recovery**: Graceful handling of network and validation errors

### Backend Architecture

**Core Endpoints:**
```javascript
// Initial booking creation
POST /api/bookings/create-booking
// Creates NON_COMMITTED booking, allows null values for incomplete data

// Progressive updates 
PATCH /api/bookings/{bookingId}/draft
// Updates existing NON_COMMITTED booking with new data

// Final commit after payment
POST /api/bookings/{bookingId}/commit  
// Changes status from NON_COMMITTED → PENDING after successful payment
```

**Validation Strategy:**
- **Draft Phase**: Custom validators allow `null`/`undefined` values for incomplete bookings
- **Commit Phase**: Full validation enforced when finalizing booking
- **Status-Based Logic**: Different validation rules for `NON_COMMITTED` vs `COMMITTED` bookings
### HouscallProIntegration
The backoffice and booking capture process is handled by HousecallPro
Developer documentation:  https://docs.housecallpro.com/docs/housecall-public-api/46e9e1be07621-webhooks


### Data Flow & Progressive Saving

**Initial Save Flow:**
1. User starts booking flow → BookingContext initializes
2. First meaningful input (e.g., property address) → triggers `lastModified` update
3. Auto-save useEffect (2-second debounce) → calls `saveAsBooking()`
4. `POST /create-booking` with `status: NON_COMMITTED` → returns `bookingId`
5. Frontend stores `bookingId` for subsequent updates

**Progressive Update Flow:**
1. User updates any field → triggers action dispatch → updates `lastModified`
2. Auto-save useEffect → calls `saveAsBooking()` with existing `bookingId`
3. `PATCH /bookings/{bookingId}/draft` → updates NON_COMMITTED booking
4. Backend accepts partial data (null values allowed for incomplete fields)

**Payment & Commit Flow:**
1. Payment step → Stripe payment processing
2. Payment success → calls `commitBooking(paymentIntentId)`
3. `POST /bookings/{bookingId}/commit` → validates complete data & updates status
4. Status changes: `NON_COMMITTED` → `PENDING`
5. Frontend sets `bookingCreated: true` & `isDraft: false`
6. Auto-save stops (prevents "already committed" errors)

### Auto-Save Logic

**Conditions for Auto-Save:**
```javascript
if (user && 
    bookingState.isDraft && 
    bookingState.lastModified && 
    !bookingState.payment.bookingCreated) {
  // Trigger auto-save
}
```

**Error Handling:**
- Graceful handling of "already committed" errors post-payment
- Network error recovery with user feedback
- Validation error display without blocking progress

### Multi-Tenant Flow Control

**Guest Flow (flowStartedAsGuest: true):**
- 6 steps: Customer → Property → Service → Scheduling → Payment → Confirmation
- Auto-user creation when sufficient contact info provided
- Full data collection from scratch

**Authenticated Flow (flowStartedAsGuest: false):**  
- 4 steps: Service → Scheduling → Payment → Confirmation
- Pre-filled customer and property data
- Faster checkout experience

### Benefits
- **Analytics**: Complete funnel analysis with step-by-step drop-off data
- **Recovery**: Sales team can follow up on incomplete bookings
- **UX**: No data loss if user closes browser or loses connection
- **Performance**: Efficient updates with minimal data transfer
- **Scalability**: Stateless auto-save that works across devices/sessions

## Database Configuration
- Development: Local PostgreSQL (`prime_shine` database)
- Production: Neon PostgreSQL (Azure East US 2)
- Connection managed through Sequelize with SSL support in production

## Environment Configuration

### Required Environment Variables

**Backend (.env)**:
- `JWT_SECRET`: Secret key for JWT token generation
- `STRIPE_SECRET_KEY`: Stripe API secret key
- `GOOGLE_MAPS_API_KEY`: Google Maps API key (if using server-side geocoding)
- Database credentials are managed in `config/database.js`

**Frontend (.env)**:
- `REACT_APP_API_URL`: Backend API URL (e.g., http://localhost:3001)
- `REACT_APP_TENANT_ID`: Tenant identifier for multi-tenant setup (determines which tenant instance)
- `REACT_APP_CITY_CODE`: Geographic service area code (e.g., 'ORL' for Orlando, 'KIS' for Kissimmee)
- `REACT_APP_STRIPE_PUBLISHABLE_KEY`: Stripe publishable key
- `REACT_APP_GOOGLE_MAPS_API_KEY`: Google Maps API key

## Development Notes
- Backend runs on port 3001, frontend proxies API calls during development via proxy setting
- Frontend build is served statically in production via `serve` package  
- Database configuration uses both `config/database.js` and `config/config.json` files
- Stripe integration requires proper API key configuration in environment variables
- JWT authentication tokens are used for API authentication
- Development database: Local PostgreSQL, Production: Neon PostgreSQL (Azure)
- Frontend uses Create React App's built-in ESLint configuration (react-app/react-app-jest)

## Testing and Quality
- **Backend**: Jest testing framework configured
- **Frontend**: React Testing Library with Jest (via react-scripts)
- No custom linting setup beyond Create React App defaults
- Always run `npm run build` to check for syntax/build errors before committing

## GitHub Repositories
- Frontend: https://github.com/thefourthhorseman/primeshine-front.git
- Backend: https://github.com/thefourthhorseman/primeshine-back.git

## GitHub Authentication
- **AI Agent Account**: thefourthhorseman-agentAI
- **Personal Access Token**:

## Development Workflow Guidelines

### Repository Structure
- **Working Directory**: The main `/home/rarnouxp/primeshine/` directory contains both frontend and backend as subdirectories
- **Frontend Work**: Navigate to `/home/rarnouxp/primeshine/primeshine-front/` for React development
- **Backend Work**: Navigate to `/home/rarnouxp/primeshine/primeshine-back/` for Node.js development
- **Important**: Each subdirectory (`primeshine-front/` and `primeshine-back/`) is its own git repository with separate remotes

### Git Operations
- **Always verify your current working directory** before creating branches or committing
- **Use `git remote -v`** to confirm which repository you're working in
- **Navigate to the correct subdirectory** before performing git operations:
  - Frontend: `cd /home/rarnouxp/primeshine/primeshine-front/`
  - Backend: `cd /home/rarnouxp/primeshine/primeshine-back/`

### Branch Management
- Use feature branch naming convention: `<task-type>/<short-description>`
  - Examples: `enhancement/hide-hero-video-mobile`, `feature/user-authentication`, `fix/payment-bug`
- Create branches from the appropriate repository's main branch
- Always push feature branches early with `-u` flag: `git push -u origin <branch-name>`

### Issue Management
- **Always close GitHub issues** after completing implementation and creating pull requests
- **Link pull requests to issues** using PR descriptions or issue references
- **Update issue status** with implementation details and PR links
- **Use `gh issue close <number>` command** to programmatically close resolved issues

### Common Mistakes to Avoid
1. **Wrong Directory**: Don't create git branches from the parent directory - always work within the specific repository
2. **File Path Assumptions**: Always verify file paths exist before reading/editing (use LS tool first)
3. **Build Verification**: Run `npm run build` to ensure no syntax errors before committing
4. **Duplicate Keys**: Watch for duplicate object keys in JavaScript (e.g., duplicate 'y' in animation properties)
5. **Issue Status**: Remember to close GitHub issues after implementation is complete

## Recent Development Context (Session Resume Information)

### Last Session Accomplishments
**Date**: January 2025
**Focus**: Feature Flag Implementation & Reusable FloatingActionButton Component

#### **Completed Tasks:**
1. **Feature Flag Implementation** (`REACT_APP_FEATUREFLIP_HOUSEPRO=1`)
   - Created `src/utils/featureFlags.js` utility for Housepro booking flow switching
   - Implemented conditional routing in `App.js` (HouseproBookingFlow vs UnifiedBookingFlow)
   - Applied feature flag logic to hide authentication elements across components:
     - `TopBar.js`: Hide login/logout buttons when Housepro enabled
     - `Footer.js`: Hide "Join our team" and "Become a Cleaner" sections
     - `Home.js`: Hide promotion banner, PromotionVouchers, and sticky CTA
     - `VacationRentals.js`: Hide ProfessionalBookingFlow component

2. **Page Layout Consistency** 
   - Converted multiple pages to use `PageLayout` component for consistent auth handling:
     - `BookingDetails.js`: Added PageLayout wrapper
     - `Login.js`: Converted to use PageLayout (preserved split-screen design)
     - `Register.js`: Converted to use PageLayout (maintained marketing sections)
     - `Dashboard.js`: Added PageLayout wrapper

3. **Reusable FloatingActionButton Component** ✅ **COMPLETED**
   - **Created**: `src/components/FloatingActionButton.js` - Fully reusable component
   - **Features**:
     - Flexible positioning (bottom-right, bottom-left, top-right, top-left, center, custom)
     - Content variants: icon, text, extended (icon + text)
     - Multiple size options and responsive behavior
     - Animation support (pulse, bounce, glow, none)
     - Accessibility features (ARIA labels, tooltips)
     - **No feature flag conditioning** - always visible as requested
   - **Implemented**: Updated `ResidentialCleaningPage.jsx` to use new component
   - **Tested**: Build completed successfully with no errors

#### **Component Architecture:**
```javascript
// Reusable FloatingActionButton usage:
<FloatingActionButton
  text="BOOK NOW"
  onClick={() => navigate('/booking')}
  tooltip="Book Your Clean Now!"
  position="bottom-right"
  size="auto"
  variant="text"
/>

// Alternative configurations:
// Icon only: variant="icon" icon={<CalendarMonth />}
// Extended: variant="extended" icon={<Phone />} text="Call Us"
// Custom position: position="custom" offset={{ bottom: 20, left: 20 }}
```

#### **File Changes Made:**
- **Created**: `src/components/FloatingActionButton.js` (new reusable component)
- **Modified**: `src/pages/ResidentialCleaningPage.jsx` (updated to use new component)
- **Updated**: Feature flag logic across multiple components (TopBar, Footer, Home, etc.)
- **Converted**: Multiple pages to use PageLayout pattern

#### **Current State:**
- ✅ FloatingActionButton component is production-ready and reusable
- ✅ Can be used on any page without feature flag dependencies
- ✅ Build tested and working (no errors, only ESLint warnings)
- ✅ All authentication elements properly hidden when Housepro feature flag active
- ✅ Page layout consistency achieved across main pages

#### **Ready for Next Steps:**
- FloatingActionButton can be deployed to other pages (Home, VacationRentals, etc.)
- Component supports all major use cases (icon, text, extended variants)
- Positioning system allows placement anywhere on screen
- Animation options provide visual polish

#### **Technical Notes:**
- Component uses Material-UI theming system
- Responsive design with mobile/desktop considerations
- Accessibility compliant with proper ARIA labels
- No external dependencies beyond existing MUI components
- Follows existing codebase patterns and conventions

###
