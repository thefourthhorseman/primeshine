# Job Fetch Sequence Diagram for MaidMapDisplay

## End-to-End Flow: Frontend → Backend → Housecall Pro → Map Display

```mermaid
sequenceDiagram
    participant User as 👤 User
    participant Map as 🗺️ MaidMapDisplay
    participant JobService as 📡 jobService.js
    participant Auth as 🔐 Auth Middleware
    participant Controller as 🎛️ externalJobsController
    participant HCP as 🏢 Housecall Pro API
    participant Geocoder as 🌍 Google Geocoding
    participant Cache as 💾 Google Maps Cache
    participant MapDisplay as 🎯 Map Rendering

    Note over User, MapDisplay: Initial Load & Weekly Navigation
    User->>Map: Navigate to map or change week
    Map->>Map: setSelectedWeekStart(weekStart)
    Map->>Map: fetchJobsAndLocations(weekStart)
    
    Note over Map, JobService: Frontend Service Layer
    Map->>JobService: fetchJobsForMapping({<br/>scheduledStartMin,<br/>scheduledStartMax<br/>})
    JobService->>JobService: fetchJobs({<br/>include: 'booking,assignments',<br/>limit: 100,<br/>scheduledStartMin,<br/>scheduledStartMax<br/>})
    
    Note over JobService, Controller: HTTP Request to Backend
    JobService->>JobService: getApiBase() // REACT_APP_API_URL
    JobService->>JobService: getAuthHeaders() // Bearer token
    JobService->>Controller: GET /api/jobs?<br/>scheduledStartMin=...&<br/>scheduledStartMax=...&<br/>limit=100
    
    Note over Controller, HCP: Backend Processing
    Auth->>Controller: Validate Bearer token
    Controller->>Controller: Validate HCP_API_KEY env var
    Controller->>Controller: Build query params for HCP
    Controller->>Controller: Prepare headers with HCP API key
    
    Note over Controller, HCP: External API Call
    Controller->>HCP: GET https://api.housecallpro.com/jobs?<br/>scheduled_start_min=...&<br/>scheduled_start_max=...&<br/>page_size=100&<br/>expand[]=appointments
    HCP-->>Controller: Response: { jobs: [...] }
    
    Note over Controller, JobService: Data Transformation
    Controller->>Controller: Extract jobs from response
    Controller->>Controller: Deduplicate jobs by ID
    Controller->>Controller: mapHcpJobToJobSchema(job) for each job
    Note right of Controller: Transform HCP format to internal format:<br/>- Map status (completed → COMPLETED)<br/>- Build formatted address<br/>- Extract customer info<br/>- Calculate job values
    Controller-->>JobService: Response: { jobs: [...], pagination: {...} }
    
    Note over JobService, Map: Frontend Processing
    JobService-->>Map: Return mappableJobs (filtered by address)
    Map->>Map: Filter jobs with valid addresses
    
    Note over Map, Geocoder: Address Geocoding
    Map->>Map: For each job with address:
    Map->>Cache: Check cache for address
    alt Address in cache
        Cache-->>Map: Return cached coordinates
    else Address not in cache
        Map->>Geocoder: geocodeAddress(job.address)
        Geocoder-->>Map: Return coordinates
        Map->>Cache: Cache geocoding result
    end
    
    Note over Map, MapDisplay: Map Preparation
    Map->>Map: Calculate job circle size based on totalAmount
    Map->>Map: Get job status color
    Map->>Map: Build job markers with location data
    Map->>Map: setJobs(validJobs)
    
    Note over Map, MapDisplay: Map Rendering
    Map->>MapDisplay: Render job markers on Google Map
    MapDisplay-->>User: Display jobs as colored circles on map
    
    Note over User, MapDisplay: Interactive Features
    User->>Map: Click on job marker
    Map->>Map: setSelectedMarker(job)
    Map->>MapDisplay: Show InfoWindow with job details
    MapDisplay-->>User: Display job information popup
```

## Error Handling Flow

```mermaid
sequenceDiagram
    participant Map as 🗺️ MaidMapDisplay
    participant JobService as 📡 jobService.js
    participant Controller as 🎛️ externalJobsController
    participant HCP as 🏢 Housecall Pro API

    Note over Map, HCP: Error Scenarios
    
    alt HCP API Failure
        Map->>JobService: fetchJobsForMapping()
        JobService->>Controller: GET /api/jobs
        Controller->>HCP: GET /jobs
        HCP-->>Controller: Error: 400/500/Timeout
        Controller->>Controller: Catch error
        Controller-->>JobService: Return empty array []
        JobService-->>Map: Return empty jobs array
        Map->>Map: setJobs([])
        Map->>Map: Show "No jobs found" message
    end
    
    alt Authentication Failure
        Map->>JobService: fetchJobsForMapping()
        JobService->>Controller: GET /api/jobs (invalid token)
        Controller->>Controller: Auth middleware rejects
        Controller-->>JobService: 401 Unauthorized
        JobService->>JobService: Throw "Failed to fetch jobs"
        JobService-->>Map: Error: "Failed to fetch job locations"
        Map->>Map: setError("Failed to load job locations")
        Map->>Map: Show error alert
    end
    
    alt Geocoding Failure
        Map->>Map: fetchJobsAndLocations()
        Map->>Map: geocodeAddress(job.address)
        Note right of Map: Address geocoding fails
        Map->>Map: Filter out job (no location)
        Map->>Map: Continue with remaining jobs
        Map->>Map: Log "No valid address found for job"
    end
```

## Data Transformation Details

```mermaid
flowchart TD
    A[HCP Raw Job] --> B[mapHcpJobToJobSchema]
    B --> C{Extract Customer Info}
    C --> D[Build customerName from first_name + last_name]
    C --> E[Extract email, phone from customer object]
    
    B --> F{Extract Address}
    F --> G[Build formatted address from address object]
    F --> H[Create address structure for frontend]
    
    B --> I{Map Status}
    I --> J[HCP 'completed' → 'COMPLETED']
    I --> K[HCP 'in_progress' → 'IN_PROGRESS']
    I --> L[HCP 'scheduled' → 'NOT_STARTED']
    I --> M[HCP 'cancelled' → 'CANCELLED']
    
    B --> N{Extract Financial Data}
    N --> O[total_amount → totalAmount]
    N --> P[invoice_status → paymentStatus]
    
    B --> Q{Create Internal Job Object}
    Q --> R[Add job identification fields]
    Q --> S[Add scheduling information]
    Q --> T[Add maid assignment data]
    Q --> U[Add HCP metadata]
    
    R --> V[Final Mapped Job]
    S --> V
    T --> V
    U --> V
    D --> V
    E --> V
    G --> V
    H --> V
    J --> V
    K --> V
    L --> V
    M --> V
    O --> V
    P --> V
```

## Key Components and Their Roles

| Component | Role | Key Functions |
|-----------|------|---------------|
| **MaidMapDisplay** | Frontend UI Component | - Initiates job fetching<br>- Handles weekly navigation<br>- Manages map state<br>- Renders job markers |
| **jobService.js** | Frontend Service Layer | - HTTP requests to backend<br>- Data filtering and processing<br>- Error handling<br>- Address extraction |
| **externalJobsController** | Backend Controller | - HCP API integration<br>- Data transformation<br>- Error handling<br>- Response formatting |
| **Housecall Pro API** | External Data Source | - Provides job data<br>- Handles date filtering<br>- Returns paginated results |
| **Google Geocoding** | Address Processing | - Converts addresses to coordinates<br>- Caches results<br>- Handles geocoding errors |
| **Google Maps Cache** | Performance Optimization | - Stores geocoding results<br>- Reduces API calls<br>- Improves response time |

## Environment Variables Used

- `REACT_APP_API_URL` - Backend API base URL
- `HCP_API_KEY` - Housecall Pro API authentication
- `REACT_APP_TENANT_ID` - Multi-tenant filtering
- `REACT_APP_CITY_CODE` - Geographic filtering
- `REACT_APP_GOOGLE_MAPS_API_KEY` - Google Maps/Geocoding API

## Performance Optimizations

1. **Caching**: Google Maps geocoding results cached to reduce API calls
2. **Batch Processing**: Multiple addresses processed in batches
3. **Deduplication**: Removes duplicate jobs from HCP response
4. **Filtering**: Only processes jobs with valid addresses
5. **Pagination**: Limits results to prevent overwhelming the UI 