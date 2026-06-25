# Smoobu Integration - Admin UI Guide

## Overview
The Smoobu integration includes a complete admin interface for importing vacation rental reservations and creating cleaning jobs automatically.

## Accessing the Smoobu Import Manager

### Location
The Smoobu import feature is available in the **Admin Dashboard**:
1. Navigate to the Dashboard (admin users only)
2. Look for the **"Smoobu Import"** card with the Hotel/Vacation Rental icon
3. Click **"Import Reservations"** button

### UI Components Created

#### 1. SmoobuJobImportManager Component
**File**: `primeshine-front/src/components/admin/SmoobuJobImportManager.js`

**Features**:
- Date range selection for import
- Preview mode (dry run)
- Actual import execution
- Real-time import statistics
- Detailed import results
- Error handling and validation

---

## How to Use the Smoobu Import Manager

### Step 1: Open the Manager
Click the **"Import Reservations"** button on the Smoobu Import card in the dashboard.

### Step 2: Select Date Range
Choose the date range for reservations to import:

**Quick Presets Available**:
- **Today** - Import today's checkouts only
- **Next 7 Days** - Import upcoming week
- **Next 30 Days** - Import next month
- **Next 90 Days** - Import next quarter

**Custom Range**:
- Use date pickers to select any start and end date
- Based on Smoobu arrival dates

**Recommendations**:
- For initial setup: Import next 30-90 days
- For regular use: Import next 7-30 days
- Avoid ranges larger than 90 days for performance

### Step 3: Preview Import (Recommended)
Before importing, click **"Preview"** to see what will be imported:

**Preview Shows**:
- Total reservations found
- New jobs that will be created
- Existing jobs that will be updated
- Jobs with no changes (will be skipped)
- Invalid reservations (errors)

**Table Details**:
- Reservation ID from Smoobu
- Apartment name/ID
- Guest name
- Checkout date/time
- Action type (Create, Update, Skip, Error)

**What to Check**:
- Verify the number of new jobs makes sense
- Check for unexpected duplicates
- Review error messages if any

### Step 4: Execute Import
After reviewing the preview, click **"Import"** to execute:

**Progress Indicator**:
- Loading bar appears
- Status message shows "Importing reservations..."

**Import Process**:
1. Fetches reservations from Smoobu API
2. Validates each reservation
3. Creates or updates cleaning jobs
4. Geocodes property addresses
5. Removes cancelled reservations

**Time to Complete**:
- 10 reservations: ~5-10 seconds
- 50 reservations: ~20-30 seconds
- 100 reservations: ~40-60 seconds

### Step 5: Review Results
After import completes, review the results:

**Summary Statistics**:
- **Total Fetched**: Reservations retrieved from Smoobu
- **Imported**: New cleaning jobs created
- **Updated**: Existing jobs updated
- **Skipped**: No changes detected
- **Errors**: Failed reservations

**Detailed Results Table**:
- Shows each reservation processed
- Status (imported, updated, skipped, error)
- Success/error messages
- Apartment and guest details

---

## Import Statistics Card

**Location**: Top of the Smoobu Import Manager dialog

**Shows**:
- **Total Imported**: All-time count of Smoobu jobs
- **Last Import**: Timestamp of most recent import

**Refresh Icon**: Click to update statistics

---

## Understanding Job Creation

### What Gets Created
For each Smoobu reservation, a cleaning job is created with:

**Job Details**:
- **Job Type**: VACATION_RENTAL
- **Scheduled Date**: Checkout date from Smoobu
- **Scheduled Time**: Checkout time (default 10:00 AM if not specified)
- **Duration**: 2 hours (default for vacation rental cleaning)
- **Address**: From Smoobu apartment
  - Street from apartment address
  - Apartment name stored in street_line_2
- **Guest Info**: Name, email, phone from reservation

**Job Identification**:
- Unique job number generated
- Linked to Smoobu reservation ID
- Linked to Smoobu apartment ID

### Duplicate Prevention
The system automatically prevents duplicates:
- Checks for existing job with same Smoobu reservation ID
- If exists, updates instead of creating new
- Shows as "Updated" or "Skipped" in results

### Change Detection
Jobs are updated only when changes detected:
- Checkout date/time changed
- Guest information changed
- Reservation price changed
- Otherwise marked as "Skipped"

---

## Common Use Cases

### Daily Import Workflow
**Recommended**: Import upcoming 7-30 days each morning
```
1. Open Smoobu Import Manager
2. Click "Next 30 Days" preset
3. Click "Preview" to review
4. Click "Import" to execute
5. Review results
6. Close dialog
```

### Initial Setup
**First time setup**: Import all future reservations
```
1. Open Smoobu Import Manager
2. Click "Next 90 Days" preset (or custom range)
3. Click "Preview" - expect many "Create New" actions
4. Click "Import"
5. Wait for completion (may take 1-2 minutes)
6. Verify all jobs created successfully
```

### After Reservation Changes in Smoobu
**When reservations are modified**:
```
1. Open Smoobu Import Manager
2. Select date range covering changed reservations
3. Click "Preview"
4. Look for "Update" actions
5. Click "Import"
6. Verify changes applied
```

### Handling Cancelled Reservations
**When guest cancels**:
- The system automatically deletes corresponding cleaning jobs
- Shows as "Deleted" in results
- Removed from schedule

---

## Troubleshooting

### No Reservations Found
**Possible Causes**:
- No reservations in selected date range
- Smoobu API key not configured
- Date range too far in future/past

**Solutions**:
- Verify reservations exist in Smoobu for that date range
- Check SMOOBU_API_KEY in backend .env file
- Try a different date range

### Preview Shows Errors
**Common Error Messages**:
- "Missing reservation ID" - Smoobu data incomplete
- "Missing dates" - No arrival/departure in reservation
- "Missing apartment data" - Apartment info incomplete

**Solutions**:
- Check reservation data in Smoobu
- Ensure all required fields are filled
- Contact support if issue persists

### Import Failed
**Symptoms**:
- Red error alert appears
- "Import failed" message

**Possible Causes**:
- Network connection issue
- Smoobu API timeout
- Database error

**Solutions**:
1. Check internet connection
2. Try smaller date range
3. Check browser console for errors
4. Try again in a few minutes

### Duplicate Jobs Created
**Should not happen** due to duplicate prevention, but if it does:

**Cause**: Different Smoobu reservation IDs for same property/date

**Solution**:
- Manually delete duplicate jobs
- Contact support to investigate

---

## Best Practices

### Import Frequency
- **Daily**: Import next 7-30 days each morning
- **Weekly**: Import next 60-90 days on Mondays
- **After changes**: Import specific date range when reservations change

### Date Range Selection
- **Smaller is faster**: Prefer 7-30 day ranges
- **Avoid huge ranges**: >90 days can be slow
- **Cover what you need**: Only import dates you need to schedule

### Using Preview
- **Always preview first** for large imports
- **Check for errors** before executing
- **Verify counts** match expectations

### Monitoring Results
- **Review summary** after each import
- **Check error messages** if any
- **Verify jobs created** in job dashboard

---

## Integration with Job Management

### Viewing Imported Jobs
After import, jobs appear in:
1. **Job Dashboard** (`/jobs`)
   - Filter by: `sourceSystem: SMOOBU`
   - Status: `NOT_STARTED`
   - Type: `VACATION_RENTAL`

2. **Calendar View** (`/schedule`)
   - Shows cleaning jobs at checkout times
   - Color-coded by status

3. **Map View**
   - Shows job locations on map
   - Grouped by apartment address

### Managing Imported Jobs
Imported jobs can be:
- **Assigned to maids** (manually or via auto-assignment)
- **Rescheduled** if needed
- **Marked complete** when cleaning done
- **Cancelled** if reservation changes

### Job Details Include
- Smoobu reservation ID (for tracking)
- Apartment name (in address)
- Guest information
- Original reservation data (in smoobuRawData field)

---

## UI Screenshots Guide

### Main Dashboard Card
```
┌─────────────────────────┐
│   🏨 Vacation Icon      │
│   Smoobu Import         │
│                         │
│ Import cleaning jobs    │
│ from Smoobu vacation    │
│ rental reservations     │
│                         │
│ [Import Reservations]   │
└─────────────────────────┘
```

### Import Manager Dialog
```
┌────────────────────────────────────────┐
│ 🏨 Smoobu Reservation Import Manager   │
├────────────────────────────────────────┤
│ Import Statistics                      │
│ Total Imported: 150                    │
│ Last Import: 2025-01-15 10:30 AM      │
├────────────────────────────────────────┤
│ Select Import Date Range               │
│ Tips: Import upcoming checkouts...     │
│                                        │
│ Quick Presets:                         │
│ [Today] [Next 7 Days] [Next 30 Days]  │
│                                        │
│ Start Date: [2025-01-15]              │
│ End Date:   [2025-02-15]              │
│                                        │
│ [Preview]  [Import]                    │
├────────────────────────────────────────┤
│ Preview Results (Expanded)             │
│ Total: 45 | New: 30 | Updates: 10    │
│ No Changes: 3 | Invalid: 2            │
│                                        │
│ Table showing reservations...          │
└────────────────────────────────────────┘
```

---

## Technical Details

### Components
- **SmoobuJobImportManager.js**: Main dialog component
- Integrated into **Dashboard.js**: Admin dashboard

### API Endpoints Used
- `POST /api/admin/smoobu/preview-import` - Preview
- `POST /api/admin/smoobu/import-reservations` - Import
- `GET /api/admin/smoobu/import-status` - Statistics

### State Management
- Local component state (no global context needed)
- Refreshes statistics after successful import
- Clears preview after import

### Permissions Required
- Admin user authentication
- Valid JWT token
- Smoobu API access configured

---

## Support and Resources

### Documentation
- **Backend API**: See `SMOOBU_INTEGRATION_SUMMARY.md`
- **Project Overview**: See `CLAUDE.md`
- **API Docs**: https://your-domain.com/api-docs

### Getting Help
1. Check error messages in UI
2. Review browser console for details
3. Check backend logs for API errors
4. Contact technical support with:
   - Date range attempted
   - Error message
   - Number of reservations
   - Screenshots if helpful

---

## Future Enhancements

### Planned Features (Not Yet Implemented)
- ⏰ Automated scheduled imports (cron jobs)
- 🔔 Webhook support for real-time sync
- 📊 More detailed analytics
- 🏠 Property-specific settings
- ⚙️ Configurable time buffers
- 👥 Employee auto-assignment integration

### Requesting Features
Submit feature requests with:
- Use case description
- Expected behavior
- Priority level

---

## Summary

The Smoobu Import Manager provides a complete, user-friendly interface for:
✅ Importing vacation rental reservations
✅ Creating cleaning jobs automatically
✅ Previewing changes before import
✅ Monitoring import statistics
✅ Handling errors gracefully
✅ Preventing duplicates
✅ Syncing deletions

**Ready to use in production!** 🚀
