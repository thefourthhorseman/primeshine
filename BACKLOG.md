# PrimeShine Development Backlog

## Server Fallback & Error Handling Improvements

### Current Issues
- **No pricing fallback values** - If API fails, users see no pricing information
- **Limited offline capability** - Application relies heavily on server responses
- **No retry mechanisms** - Single failure stops the entire process
- **No graceful degradation** - Features become completely unusable rather than providing limited functionality

### Proposed Improvements

#### 1. Enhanced Pricing Fallback System
- Implement default pricing estimates when `/api/bookings/calculate-price` fails
- Store last known pricing in localStorage for offline estimation
- Add configurable fallback rates for main services and extras
- Show pricing disclaimers when using fallback data

#### 2. Service Information Resilience
- Expand hardcoded service catalog for complete offline operation
- Implement service catalog caching with periodic refresh
- Add service availability fallbacks when catalog API is unreachable
- Graceful handling of missing service descriptions/images

#### 3. Request Retry Mechanisms
- Add exponential backoff retry for failed API requests
- Implement circuit breaker pattern for repeated failures
- Queue failed requests for retry when connection restored
- Show retry progress to users during network issues

#### 4. Offline-First Capabilities
- Cache critical application data in localStorage/IndexedDB
- Enable basic booking flow completion offline
- Sync pending operations when connection restored
- Provide clear offline/online status indicators

#### 5. Error Recovery Workflows
- Implement "Try Again" buttons with specific retry logic
- Add manual refresh options for failed data loads
- Provide alternative pathways when primary flows fail
- Clear error state management and user messaging

#### 6. Network Status Integration
- Detect online/offline status changes
- Adapt UI behavior based on connection quality
- Show appropriate messaging for network-related errors
- Enable background sync when connection improves

### Technical Implementation Notes
- Consider using React Query or SWR for better caching and retry logic
- Implement service worker for offline functionality
- Add comprehensive error boundary components
- Create fallback data constants and validation schemas

### Priority
- Medium-High priority for production reliability
- Should be implemented before major production rollout
- Critical for mobile users with unstable connections

## Frontend Static Asset Consolidation (`public/images` vs `public/media`)

### Current State
There is no architectural rule separating the two directories — it's a historical
split. `public/images/` is the original asset folder (Jul 2025); `public/media/`
was introduced with the landing-page redesign (Dec 2025, commits `924569b`,
`4381c8c`). Both are plain CRA static assets served from the site root and
referenced by absolute URL string; no config, helper, or build step treats them
differently.

**`public/images/` — legacy / non-landing surfaces**
| Subfolder | Used by |
|---|---|
| root (`logo2.jpg`, `og-image.jpg`, `cleaning-hero.jpg`) | `Seo.js`, `siteInfo.js`, `TopBar.js`, `Login.js`, `ResetPasswordPage.js`, `MaidPortalLayout.js` |
| `review-photos/` | `CustomerReviews.js` |
| `services/` | `servicesData.json`, `Home.js` |
| `tips/` | `content/cleaning-tips/*.json` |

**`public/media/` — the landing-page redesign**
| Subfolder | Used by |
|---|---|
| `gallery/` | `Landing.js` media gallery + background videos |
| `features/` | `Landing.js` feature cards |
| `services/` | `ServicesGrid.js`, `OurServices.js`, `ServiceCarousel.js` |
| `promo/` | promo section |

The one genuine design difference: `media/services/` is keyed to ServiceCatalog
IDs (`svc_<serviceId>.webp`) and resolved by convention in
`OurServices.js:176-179`, whereas `images/services/` is hand-referenced by
filename. `media/` is also format-modern (`.webp` stills, `.webm` video).

### Broken References (migration never finished)
- `ServiceCarousel.js:19-49` — all 7 entries point at `/media/services/deep-cleaning.jpg`,
  `move-cleaning.png`, `pool-cleaning.jpg`, etc. Those files live in
  `images/services/`, not `media/services/`. Looks like a `/images/` → `/media/`
  find-and-replace without moving the files.
- `HeroTestimonialSection.js` — `/media/avatars/*.png` (directory does not exist)
  and `/media/services/persona_maid_[1-4].png` are all missing; the persona image
  actually exists as `/images/persona_maid1.webp`.
- Also referenced but absent: `/images/quotation-hero.png`, `/images/tips/step*.jpg`,
  `/images/services/deep-cleaning.jpg`, `s2.jpg`, `s3.jpg`.
- Both trees contain WSL `:Zone.Identifier` files that ship into the build.

### Proposed Rule
- `media/` = marketing/landing content (webp/webm, service-ID-keyed)
- `images/` = app chrome, brand, and SEO assets
- Move stragglers to match, fix the broken paths above, and gitignore/strip
  `:Zone.Identifier` files from the build

### Priority
- Low — cosmetic/organizational, except the broken references, which are
  user-visible missing images on the landing page and service carousel