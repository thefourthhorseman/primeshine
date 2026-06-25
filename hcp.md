# HouseCallPro API Curl Commands

## Configuration
- **Base URL**: `https://api.housecallpro.com`
- **API Key**: `e679ef85591e40f1a76e89e2200e1044`
- **Date Range**: September 1-30, 2025
- **Authentication**: Bearer token in Authorization header

## Basic Job Retrieval Commands

### 1. Get Jobs from HCP API for September 1-30, 2025:
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-01T00:00:00.000Z&scheduled_start_max=2025-09-30T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044"
```

### 2. Get All September Jobs (no status filter):
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-01T00:00:00.000Z&scheduled_start_max=2025-09-30T23:59:59.999Z&page_size=100" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044"
```

### 3. Get September Jobs - Week 1 (Sept 1-7):
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-01T00:00:00.000Z&scheduled_start_max=2025-09-07T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044"
```

### 4. Get September Jobs - Week 2 (Sept 8-14):
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-08T00:00:00.000Z&scheduled_start_max=2025-09-14T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044"
```

### 5. Get September Jobs - Week 3 (Sept 15-21):
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-15T00:00:00.000Z&scheduled_start_max=2025-09-21T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044"
```

### 6. Get September Jobs - Week 4 (Sept 22-30):
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-22T00:00:00.000Z&scheduled_start_max=2025-09-30T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044"
```

## Employee Information Extraction

### 1. Extract All Employee Names:
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-01T00:00:00.000Z&scheduled_start_max=2025-09-30T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044" | \
jq -r '.jobs[].assigned_employees[] | "\(.first_name) \(.last_name)"' | sort -u
```

### 2. Extract Employee IDs and Names:
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-01T00:00:00.000Z&scheduled_start_max=2025-09-30T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044" | \
jq -r '.jobs[].assigned_employees[] | "ID: \(.id), Name: \(.first_name) \(.last_name), Email: \(.email), Phone: \(.mobile_number)"' | sort -u
```

### 3. Extract Employee Emails:
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-01T00:00:00.000Z&scheduled_start_max=2025-09-30T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044" | \
jq -r '.jobs[].assigned_employees[].email' | sort -u
```

### 4. Count Jobs per Employee:
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-01T00:00:00.000Z&scheduled_start_max=2025-09-30T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044" | \
jq -r '.jobs[].assigned_employees[] | "\(.first_name) \(.last_name)"' | sort | uniq -c | sort -nr
```

## Customer Information Extraction

### 1. Extract Customer Names Only:
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-01T00:00:00.000Z&scheduled_start_max=2025-09-30T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044" | \
jq -r '.jobs[].customer | "\(.first_name) \(.last_name)"' | sort -u
```

### 2. Extract Customer Full Details (Handling null emails):
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-01T00:00:00.000Z&scheduled_start_max=2025-09-30T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044" | \
jq -r '.jobs[].customer | "Name: \(.first_name) \(.last_name), Email: \(.email // "No Email"), Phone: \(.mobile_number // "No Phone")"' | sort -u
```

### 3. Extract Only Customers WITH Email Addresses:
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-01T00:00:00.000Z&scheduled_start_max=2025-09-30T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044" | \
jq -r '.jobs[].customer | select(.email != null and .email != "") | "Name: \(.first_name) \(.last_name), Email: \(.email)"' | sort -u
```

### 4. Extract Customer Emails Only (Skip nulls):
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-01T00:00:00.000Z&scheduled_start_max=2025-09-30T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044" | \
jq -r '.jobs[].customer.email | select(. != null and . != "")' | sort -u
```

### 5. Extract Customer IDs and Names:
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-01T00:00:00.000Z&scheduled_start_max=2025-09-30T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044" | \
jq -r '.jobs[].customer | "ID: \(.id), Name: \(.first_name) \(.last_name)"' | sort -u
```

### 6. Extract Customer Phone Numbers Only:
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-01T00:00:00.000Z&scheduled_start_max=2025-09-30T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044" | \
jq -r '.jobs[].customer.mobile_number' | sort -u
```

### 7. Extract Customer with Lead Source:
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-01T00:00:00.000Z&scheduled_start_max=2025-09-30T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044" | \
jq -r '.jobs[].customer | "Name: \(.first_name) \(.last_name), Lead Source: \(.lead_source // "None")"' | sort -u
```

### 8. Extract Customer with Address:
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-01T00:00:00.000Z&scheduled_start_max=2025-09-30T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044" | \
jq -r '.jobs[] | "Customer: \(.customer.first_name) \(.customer.last_name), Address: \(.address.street), \(.address.city), \(.address.state) \(.address.zip)"' | sort -u
```

### 9. Count Jobs per Customer:
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-01T00:00:00.000Z&scheduled_start_max=2025-09-30T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044" | \
jq -r '.jobs[].customer | "\(.first_name) \(.last_name)"' | sort | uniq -c | sort -nr
```

### 10. Extract Full Customer Contact Details:
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-01T00:00:00.000Z&scheduled_start_max=2025-09-30T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044" | \
jq -r '.jobs[].customer | "Name: \(.first_name) \(.last_name), Email: \(.email), Mobile: \(.mobile_number), Home: \(.home_number // "None"), Work: \(.work_number // "None")"' | sort -u
```

### 11. Extract Customer with Notes:
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-01T00:00:00.000Z&scheduled_start_max=2025-09-30T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044" | \
jq -r '.jobs[].customer | "Name: \(.first_name) \(.last_name), Notes: \(.notes // "None")"' | sort -u
```

## Debug Commands

### Debug: Show all customer data structure:
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-01T00:00:00.000Z&scheduled_start_max=2025-09-30T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044" | \
jq '.jobs[0].customer'
```

### Debug: Show all employee data structure:
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-01T00:00:00.000Z&scheduled_start_max=2025-09-30T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044" | \
jq '.jobs[0].assigned_employees[0]'
```

### Debug: Show full job structure:
```bash
curl -X GET "https://api.housecallpro.com/jobs?scheduled_start_min=2025-09-01T00:00:00.000Z&scheduled_start_max=2025-09-30T23:59:59.999Z&page_size=100&work_status[]=scheduled&work_status[]=in_progress&work_status[]=completed" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer e679ef85591e40f1a76e89e2200e1044" | \
jq '.jobs[0]'
```

## Available Query Parameters

- `scheduled_start_min`: Filter jobs scheduled after this date (ISO-8601 format)
- `scheduled_start_max`: Filter jobs scheduled before this date (ISO-8601 format)
- `page_size`: Number of jobs per page (max 100)
- `work_status[]`: Filter by job status (can specify multiple)
  - Available statuses: `scheduled`, `in_progress`, `completed`, `cancelled`
- `page`: Page number for pagination

## JSON Response Structure

### Job Object Structure:
- `id`: Job ID
- `invoice_number`: Invoice number
- `description`: Job description
- `customer`: Customer object (see below)
- `address`: Service address object
- `notes`: Array of note objects
- `work_status`: Current job status
- `work_timestamps`: Start/completion timestamps
- `schedule`: Scheduling information
- `total_amount`: Total job amount (in cents)
- `outstanding_balance`: Outstanding balance
- `assigned_employees`: Array of assigned employee objects

### Customer Object Structure:
- `id`: Customer ID
- `first_name`, `last_name`: Customer name
- `email`: Email address
- `mobile_number`, `home_number`, `work_number`: Phone numbers
- `company`: Company name
- `lead_source`: Lead source
- `notes`: Customer notes
- `notifications_enabled`: Boolean for notifications

### Employee Object Structure:
- `id`: Employee ID
- `first_name`, `last_name`: Employee name
- `email`: Email address
- `mobile_number`: Phone number
- `color_hex`: Color for calendar display
- `avatar_url`: Profile picture URL
- `role`: Employee role
- `permissions`: Permission object