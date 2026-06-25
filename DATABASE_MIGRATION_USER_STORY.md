# Technical User Story: Database Migration from NEON to Heroku Postgres

## Epic
**Infrastructure Migration** - Migrate PrimeShine backend database from NEON cloud hosting to Heroku Postgres for improved integration and management.

## User Story
**As a** DevOps Engineer and Backend Developer  
**I want to** migrate the PrimeShine production database from NEON to Heroku Postgres  
**So that** I can have better integration with our Heroku deployment pipeline, improved monitoring, and centralized infrastructure management.

## Current State Analysis

### Current NEON Configuration
- **Host**: `ep-proud-sun-a8mm12bb-pooler.eastus2.azure.neon.tech`
- **Database**: `neondb`
- **User**: `neondb_owner`
- **Region**: East US 2 (Azure)
- **SSL**: Required with `rejectUnauthorized: false`
- **Connection**: Pooled connection via NEON pooler

### Database Schema Overview
The database contains **43 migrations** with the following key components:
- **Core Tables**: Users, Roles, MaidProfiles, ClientProfiles, Bookings, Quotations
- **Business Logic**: Job Management, Team Management, Service Catalog, Reviews
- **Recent Features**: Questionnaire Progress (lead tracking), Payment Integration
- **Multi-tenant Architecture**: TenantId and CityCode support

### Current Configuration Files
- **Production Config**: `src/config/database.js` (hardcoded NEON credentials)
- **Sequelize Config**: `src/config/config.json` (uses `DATABASE_URL` env var)
- **Model Integration**: `src/models/index.js` (supports both formats)

## Acceptance Criteria

### ✅ Pre-Migration Requirements
1. **Heroku Postgres Setup**
   - [ ] Create Heroku Postgres add-on (Standard-0 or higher for production)
   - [ ] Verify Heroku Postgres version compatibility (PostgreSQL 14+)
   - [ ] Configure connection pooling if needed
   - [ ] Set up database monitoring and alerting

2. **Backup Strategy**
   - [ ] Create full NEON database backup using `pg_dump`
   - [ ] Verify backup integrity with test restore
   - [ ] Document current database size and performance metrics
   - [ ] Identify peak usage times for migration window

3. **Environment Preparation**
   - [ ] Update Heroku environment variables
   - [ ] Configure `DATABASE_URL` with Heroku Postgres connection string
   - [ ] Test database connectivity from staging environment
   - [ ] Prepare rollback environment variables

### ✅ Migration Process
4. **Schema Migration**
   - [ ] Create empty Heroku Postgres database
   - [ ] Run all 43 migrations in sequence using `npx sequelize-cli db:migrate`
   - [ ] Verify all tables and indexes are created correctly
   - [ ] Compare schema between NEON and Heroku using `pg_dump --schema-only`

5. **Data Migration** 
   - [ ] Export data from NEON using `pg_dump --data-only --no-owner --no-privileges`
   - [ ] Import data to Heroku Postgres using `psql`
   - [ ] Verify row counts match between source and destination
   - [ ] Test foreign key constraints and relationships
   - [ ] Validate recent questionnaire progress data

6. **Application Configuration**
   - [ ] Remove hardcoded NEON credentials from `src/config/database.js`
   - [ ] Update production configuration to use `DATABASE_URL` exclusively
   - [ ] Test application startup with new database connection
   - [ ] Verify all Sequelize models initialize correctly

### ✅ Testing & Validation
7. **Functional Testing**
   - [ ] Test user authentication and registration
   - [ ] Verify booking creation and payment processing
   - [ ] Test maid onboarding flow end-to-end
   - [ ] Validate questionnaire lead tracking functionality
   - [ ] Test job management and mapping features

8. **Performance Validation**
   - [ ] Compare query performance between NEON and Heroku
   - [ ] Monitor connection pool usage
   - [ ] Verify SSL connections are working properly
   - [ ] Test high-load scenarios (if applicable)

### ✅ Go-Live & Monitoring
9. **Deployment**
   - [ ] Schedule maintenance window (low-traffic period)
   - [ ] Deploy configuration changes to production
   - [ ] Monitor application logs for database connection issues
   - [ ] Verify all scheduled jobs and background tasks work correctly

10. **Post-Migration Validation**
    - [ ] Run full regression test suite
    - [ ] Monitor error rates and response times for 48 hours
    - [ ] Validate database backups are working in Heroku
    - [ ] Confirm all integrations (Stripe, email, file uploads) function correctly

## Technical Implementation Details

### Configuration Changes Required

#### 1. Update `src/config/database.js`
```javascript
require('dotenv').config();

module.exports = {
  development: {
    username: 'primeshine',
    password: 'primeshine123',
    database: 'prime_shine',
    host: '127.0.0.1',
    dialect: 'postgres',
    dialectOptions: {
      ssl: false
    }
  },
  production: {
    use_env_variable: 'DATABASE_URL',
    dialect: 'postgres',
    dialectOptions: {
      ssl: {
        require: true,
        rejectUnauthorized: false,
      },
    },
  },
};
```

#### 2. Environment Variables (Heroku)
```bash
# Remove these NEON-specific variables
DATABASE_URL=postgres://username:password@hostname:port/database

# Heroku will automatically provide:
DATABASE_URL=postgres://user:pass@host:port/dbname
```

#### 3. Migration Command
```bash
# Run all migrations on new Heroku database
NODE_ENV=production npx sequelize-cli db:migrate
```

### Data Migration Scripts

#### Export from NEON
```bash
pg_dump "postgresql://neondb_owner:npg_bC1t6dWKVQNH@ep-proud-sun-a8mm12bb-pooler.eastus2.azure.neon.tech/neondb?sslmode=require" \
  --no-owner --no-privileges --clean --if-exists > neon_backup.sql
```

#### Import to Heroku
```bash
heroku pg:psql DATABASE_URL --app your-app-name < neon_backup.sql
```

## Risk Assessment & Mitigation

### High Risk Items
1. **Data Loss During Migration**
   - **Mitigation**: Multiple backups, incremental migration testing
   - **Rollback**: Keep NEON database active during testing period

2. **Application Downtime**
   - **Mitigation**: Schedule during low-traffic hours, prepare rollback plan
   - **Rollback**: Revert `DATABASE_URL` to NEON connection string

3. **Performance Regression**
   - **Mitigation**: Load testing, monitoring alerts
   - **Rollback**: Switch back to NEON if performance issues persist

### Medium Risk Items
1. **SSL Configuration Issues**
   - **Mitigation**: Test SSL connections in staging environment
   
2. **Connection Pool Exhaustion**
   - **Mitigation**: Monitor connection usage, configure appropriate pool sizes

## Rollback Strategy

### Immediate Rollback (< 1 hour)
1. Revert `DATABASE_URL` environment variable to NEON connection
2. Deploy previous application version if needed
3. Monitor application health and error rates

### Data Rollback (if data corruption occurs)
1. Restore NEON database from pre-migration backup
2. Re-point application to NEON database
3. Investigate and resolve migration issues before retry

## Success Metrics

### Technical Metrics
- **Zero data loss**: Row counts match between NEON and Heroku
- **Performance maintained**: Query response times within 10% of baseline
- **Uptime**: < 30 minutes total downtime during migration
- **Error rate**: No increase in application error rates post-migration

### Business Metrics
- **User experience**: No user-reported issues with core functionality
- **Payment processing**: All payment flows continue to work
- **Lead tracking**: Questionnaire submissions continue to save to database

## Timeline Estimate

- **Planning & Setup**: 2-3 days
- **Testing & Validation**: 3-5 days  
- **Migration Execution**: 4-8 hours (including validation)
- **Post-migration Monitoring**: 2-3 days

**Total Estimated Effort**: 1-2 weeks

## Dependencies

- **Heroku Access**: Production deployment permissions
- **Database Access**: NEON database admin access for backup/export
- **Maintenance Window**: Coordinated downtime approval
- **Testing Environment**: Staging environment with Heroku Postgres setup

## Definition of Done

- [ ] All 43 database migrations run successfully on Heroku Postgres
- [ ] All production data migrated with 100% integrity
- [ ] Application connects to Heroku Postgres without hardcoded credentials
- [ ] All critical user flows tested and validated
- [ ] Performance metrics meet or exceed baseline
- [ ] Rollback plan tested and documented
- [ ] Team trained on new database management procedures
- [ ] Monitoring and alerting configured for new database
- [ ] Documentation updated with new connection details
- [ ] NEON database safely decommissioned (after 7-day grace period)

---

**Story Points**: 13 (Large)  
**Priority**: High  
**Labels**: `infrastructure`, `database`, `migration`, `heroku`, `production`