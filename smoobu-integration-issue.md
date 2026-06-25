# Integrate Smoobu Vacation Rental Management API for automated cleaning scheduling

## Type
`Feature Request`

## Description

Integrate Smoobu's vacation rental management API to enable automated cleaning scheduling for short-term rental properties. This integration will allow PrimeShine to tap into the growing vacation rental market by providing seamless turnover cleaning services synchronized with guest bookings.

### Business Value
- **Market Expansion**: Access to vacation rental property owners and managers
- **Recurring Revenue**: Automated cleaning scheduling between guest bookings
- **Premium Pricing**: Vacation rentals typically require higher-value cleaning services
- **Operational Efficiency**: Automated scheduling reduces manual coordination
- **Competitive Advantage**: Direct API integration provides seamless booking-to-cleaning workflow

### Current State
- PrimeShine currently offers generic cleaning services without vacation rental specialization
- Manual scheduling and coordination required for short-term rental properties
- No integration with property management systems
- Limited visibility into guest checkout/checkin schedules

## Proposed Smoobu Integration Features

### 1. **Property Management Integration**
- Sync vacation rental properties from Smoobu accounts
- Import property details (address, type, amenities, special requirements)
- Support for multiple property types (apartments, houses, boutique hotels, etc.)
- Property categorization for specialized cleaning templates

### 2. **Booking Data Synchronization**
- Real-time retrieval of guest reservations and booking details
- Checkout/checkin date tracking for cleaning window identification
- Guest count and property usage intensity for cleaning scope determination
- Cancellation handling and schedule adjustments

### 3. **Automated Cleaning Scheduling**
- Automatic scheduling of turnover cleaning between bookings
- Integration with existing Airbnb Turnover Pro service template
- Configurable buffer times between checkout and checkin
- Priority scheduling for same-day turnovers

### 4. **Service Template Enhancement**
- Extend existing service templates for vacation rental specific needs:
  - **Airbnb Turnover Pro**: Enhanced with Smoobu booking data
  - **Move-In/Move-Out Specialist**: Adapted for seasonal rental transitions
  - **Premium Deep Clean**: For monthly/seasonal deep cleaning cycles

## Technical Implementation Plan

### Phase 1: Core API Integration (2-3 weeks)

#### Backend Infrastructure
```javascript
// New Smoobu service integration
src/services/smoobuService.js          // Core API client
src/models/SmoobuProperty.js           // Property model
src/models/SmoobuReservation.js        // Booking model  
src/models/SmoobuIntegration.js        // Integration settings
src/controllers/smoobuController.js    // API endpoints
src/routes/smoobu.js                   // Route definitions
```

#### Database Schema
```sql
-- Smoobu Properties Table
CREATE TABLE smoobu_properties (
  id UUID PRIMARY KEY,
  smoobu_property_id VARCHAR(100) NOT NULL,
  user_id INTEGER REFERENCES users(id),
  tenant_id INTEGER NOT NULL,
  property_name VARCHAR(200),
  property_type VARCHAR(50),
  address JSONB,
  amenities JSONB,
  cleaning_requirements JSONB,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(smoobu_property_id, tenant_id)
);

-- Smoobu Reservations Table  
CREATE TABLE smoobu_reservations (
  id UUID PRIMARY KEY,
  smoobu_reservation_id VARCHAR(100) NOT NULL,
  smoobu_property_id UUID REFERENCES smoobu_properties(id),
  guest_name VARCHAR(200),
  guest_count INTEGER,
  check_in_date DATE,
  check_out_date DATE,
  booking_status VARCHAR(50),
  cleaning_scheduled BOOLEAN DEFAULT false,
  job_id UUID REFERENCES jobs(id),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(smoobu_reservation_id)
);

-- Smoobu Integration Settings
CREATE TABLE smoobu_integrations (
  id UUID PRIMARY KEY,
  user_id INTEGER REFERENCES users(id),
  tenant_id INTEGER NOT NULL,
  api_key_encrypted TEXT,
  webhook_url VARCHAR(500),
  auto_schedule_enabled BOOLEAN DEFAULT true,
  buffer_hours INTEGER DEFAULT 4,
  default_service_template VARCHAR(100),
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id, tenant_id)
);
```

#### API Endpoints
```javascript
// Smoobu Integration Management
POST   /api/smoobu/connect           // Connect Smoobu account
GET    /api/smoobu/properties        // List synced properties
POST   /api/smoobu/sync              // Manual sync properties/reservations
GET    /api/smoobu/reservations      // List upcoming reservations
POST   /api/smoobu/schedule-cleaning // Schedule cleaning for reservation
DELETE /api/smoobu/disconnect        // Disconnect integration

// Webhook Endpoints
POST   /api/smoobu/webhooks/booking-created
POST   /api/smoobu/webhooks/booking-updated  
POST   /api/smoobu/webhooks/booking-cancelled
```

### Phase 2: Automated Scheduling (1-2 weeks)

#### Smart Scheduling Logic
```javascript
// Automated cleaning scheduler
const scheduleVacationRentalCleaning = async (reservation) => {
  const property = await SmoobuProperty.findBySmoobuId(reservation.propertyId);
  const checkoutDateTime = new Date(reservation.checkOutDate);
  const checkinDateTime = new Date(reservation.checkInDate);
  
  // Calculate optimal cleaning window
  const bufferHours = property.integration.bufferHours || 4;
  const cleaningStartTime = new Date(checkoutDateTime.getTime() + (2 * 60 * 60 * 1000)); // 2 hours after checkout
  const cleaningDeadline = new Date(checkinDateTime.getTime() - (bufferHours * 60 * 60 * 1000));
  
  // Determine service template based on property and booking details
  const serviceTemplate = determineServiceTemplate({
    propertyType: property.propertyType,
    guestCount: reservation.guestCount,
    stayDuration: calculateStayDuration(reservation),
    lastCleaningDate: await getLastCleaningDate(property.id)
  });
  
  // Create job with appropriate template
  const job = await createVacationRentalJob({
    propertyId: property.id,
    reservationId: reservation.id,
    serviceTemplate,
    scheduledStartTime: cleaningStartTime,
    deadline: cleaningDeadline,
    priority: calculatePriority(cleaningStartTime, cleaningDeadline)
  });
  
  return job;
};
```

#### Service Template Integration
- Extend existing service templates with vacation rental specific requirements
- Dynamic pricing based on property size, guest count, and turnaround time
- Special handling for same-day turnovers and extended stays

### Phase 3: Advanced Features (2-3 weeks)

#### Webhook Integration
- Real-time booking notifications from Smoobu
- Automatic schedule adjustments for booking changes/cancellations
- Guest communication integration for cleaning confirmations

#### Analytics and Reporting
- Vacation rental cleaning performance metrics
- Revenue tracking for Smoobu-generated bookings
- Property owner satisfaction and retention analytics

#### Multi-tenant Support
- Support for property management companies with multiple clients
- White-label cleaning services for property managers
- Bulk property onboarding and management

## Acceptance Criteria

### Core Integration
- [ ] Smoobu API client with authentication and error handling
- [ ] Property synchronization from Smoobu accounts
- [ ] Reservation data retrieval and storage
- [ ] Database models for properties, reservations, and integration settings
- [ ] Admin interface for managing Smoobu connections

### Automated Scheduling
- [ ] Smart cleaning window calculation between bookings
- [ ] Integration with existing service template system
- [ ] Automatic job creation for checkout cleaning
- [ ] Buffer time configuration for cleaning completion
- [ ] Priority scheduling for tight turnarounds

### Service Template Enhancement
- [ ] Vacation rental specific cleaning checklists
- [ ] Dynamic pricing based on property and booking characteristics
- [ ] Support for amenity restocking and guest preparation
- [ ] Photo documentation for property owner records

### User Experience
- [ ] Property owner dashboard for connected Smoobu properties
- [ ] Cleaning schedule visualization with booking calendar
- [ ] Notification system for scheduled cleaning confirmations
- [ ] Mobile-friendly interface for property managers

### Integration Reliability
- [ ] Webhook handling for real-time booking updates
- [ ] Error handling and retry logic for API failures
- [ ] Data synchronization validation and conflict resolution
- [ ] Security measures for API key storage and transmission

## API Integration Details

### Smoobu API Endpoints to Integrate
```javascript
// Core endpoints based on available API
GET    /user                    // User profile information
GET    /reservations           // List reservations with filtering
POST   /reservations/{id}/cancel // Cancel reservation
GET    /availability           // Check apartment availability
GET    /rates                  // Get apartment rates
POST   /rates                  // Set rates and minimum stay
```

### Error Handling Strategy
```javascript
// Robust error handling for external API
const handleSmoobuApiError = (error) => {
  switch (error.status) {
    case 401:
      // API key invalid or expired - notify user to reconnect
      await disableIntegration(integrationId);
      await notifyUserOfAuthError(userId);
      break;
    case 429:
      // Rate limiting - implement exponential backoff
      await scheduleRetryWithBackoff(apiCall, retryCount);
      break;
    case 500:
      // Smoobu server error - log and retry later
      await logExternalServiceError('smoobu', error);
      await scheduleRetry(apiCall, 300); // 5 minute delay
      break;
    default:
      await logUnexpectedError(error);
  }
};
```

## Priority
`High`

## Affected Components

### Backend (primeshine-back)
- `/src/services/smoobuService.js` - New Smoobu API integration service
- `/src/models/SmoobuProperty.js` - Property model for vacation rentals
- `/src/models/SmoobuReservation.js` - Booking data model
- `/src/models/SmoobuIntegration.js` - Integration settings model
- `/src/controllers/smoobuController.js` - API endpoints controller
- `/src/routes/smoobu.js` - Route definitions
- `/src/jobs/smoobuSyncJob.js` - Background sync jobs
- `/src/middleware/smoobuWebhookAuth.js` - Webhook security middleware

### Database
- New tables: `smoobu_properties`, `smoobu_reservations`, `smoobu_integrations`
- Migration scripts for schema creation
- Indexes for efficient querying of booking and property data
- Foreign key relationships with existing job and user tables

### Frontend Integration Points
- Property management dashboard for vacation rentals
- Service template selection with vacation rental options
- Booking calendar integration with cleaning schedules
- Integration settings and account connection UI

## Business Impact

### Revenue Opportunities
- **New Market Segment**: Access to vacation rental property owners and managers
- **Recurring Revenue Model**: Regular cleaning between bookings creates predictable income
- **Premium Service Pricing**: Vacation rentals typically pay 20-40% more for cleaning services
- **Volume Discounts**: Property managers with multiple units provide bulk business opportunities

### Operational Benefits
- **Automated Scheduling**: Reduces manual coordination and scheduling overhead
- **Predictable Workload**: Advance booking information enables better staff planning
- **Quality Control**: Standardized processes for vacation rental cleaning requirements
- **Customer Retention**: Integrated solution increases switching costs for property owners

### Market Positioning
- **Technology Leadership**: First-mover advantage in vacation rental cleaning automation
- **Scalable Solution**: Platform approach enables rapid expansion to new markets
- **Partnership Opportunities**: Potential integration partnerships with other property management tools

## Technical Considerations

### Security
- Encrypted storage of Smoobu API keys using industry-standard encryption
- Webhook signature verification to ensure authentic requests
- Rate limiting and request validation to prevent abuse
- Audit logging for all integration activities

### Performance
- Asynchronous processing for bulk property and reservation synchronization
- Caching strategy for frequently accessed property and booking data
- Database indexing for efficient querying of time-sensitive booking information
- Background job processing for webhook handling and automated scheduling

### Scalability
- Multi-tenant architecture supporting thousands of integrated properties
- Horizontal scaling support for high-volume property management companies
- Queue-based processing for handling peak booking synchronization periods
- Monitoring and alerting for integration health and performance

## Success Metrics

### Integration Adoption
- Number of connected Smoobu accounts within 90 days
- Property count synchronized from Smoobu integrations
- Automated cleaning jobs created vs manual scheduling reduction

### Business Impact
- Revenue generated from Smoobu-connected properties
- Customer acquisition cost reduction through integrated referrals
- Customer lifetime value increase for vacation rental clients
- Market share growth in vacation rental cleaning services

### Operational Efficiency
- Reduction in manual scheduling time and coordination effort
- Cleaning job completion rate and on-time performance for vacation rentals
- Customer satisfaction scores for integrated vs non-integrated services
- Staff utilization optimization through predictable booking schedules

## Additional Context

### Market Research Integration
- Smoobu supports 100+ booking platforms including Airbnb, Booking.com, Expedia
- Vacation rental market growing at 7.9% CAGR, reaching $87.09 billion by 2025
- Average cleaning fee for vacation rentals: $75-150 vs $50-100 for regular homes
- Property managers typically oversee 10-50 units, providing scalable business opportunity

### Integration Benefits for Property Owners
- Automated cleaning coordination reduces management overhead
- Consistent cleaning standards improve guest satisfaction and reviews
- Photo documentation provides accountability and peace of mind
- Integration with existing property management workflow

### Competitive Analysis
- Current cleaning services require manual coordination with property managers
- No major cleaning platforms offer direct vacation rental management integration
- First-mover advantage in automated vacation rental cleaning market
- Potential to become preferred cleaning partner for Smoobu users

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>