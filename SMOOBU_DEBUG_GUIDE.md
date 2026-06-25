# Smoobu Integration Debugging Guide

## What Was Added

I've added comprehensive debug logging at every level of the request flow to help identify where the Smoobu API calls are failing:

### 1. Frontend Logging (Browser Console)
- **Blue circles** 🔵 for Preview requests
- **Green circles** 🟢 for Import requests
- Shows: Request URL, body, token presence, response status and data

### 2. Backend Route Logging
- **Blue markers** 🔵 [SMOOBU ROUTES] when request reaches the routes
- Shows: Method, path, full URL, headers, request body

### 3. Backend Service Logging
- **Globe markers** 🌐 [SMOOBU API] when making external calls to Smoobu
- Shows: Date range, API key presence, full Smoobu URL, response status

### 4. App Registration Logging
- Confirms Smoobu routes are registered at `/api/admin/smoobu`

## Testing Steps

### Step 1: Restart the Backend Server

```bash
cd /home/rarnouxp/primeshine/primeshine-back

# Stop the current server (Ctrl+C if running)
# Then restart:
npm run dev
```

**Look for this line on startup:**
```
✅ [APP] Registered Smoobu routes at: /api/admin/smoobu
```

### Step 2: Open the Frontend Application

1. Open your browser
2. Navigate to your admin dashboard
3. **Open Browser DevTools** (F12 or right-click > Inspect)
4. Go to the **Console** tab
5. Keep the console open

### Step 3: Test with the UI

1. Click the "Import Reservations" button on the Smoobu card
2. Select a date range (e.g., Today to +30 days)
3. Click **"Preview"** button

### Step 4: Observe the Logs

#### In Browser Console, you should see:
```
🔵 [FRONTEND] Making Smoobu preview request
🔵 [FRONTEND] URL: http://localhost:3001/api/admin/smoobu/preview-import
🔵 [FRONTEND] Body: { startDate: "...", endDate: "...", tenantId: "1", cityCode: "MCO" }
🔵 [FRONTEND] Token present: true
🔵 [FRONTEND] Response status: 200
🔵 [FRONTEND] Response OK: true
🔵 [FRONTEND] Response data: { success: true, preview: {...} }
```

#### In Backend Terminal, you should see:
```
🔵 [SMOOBU ROUTES] POST /preview-import
🔵 [SMOOBU ROUTES] Full URL: http://localhost:3001/api/admin/smoobu/preview-import
🔵 [SMOOBU ROUTES] Headers: { auth: 'PRESENT', contentType: 'application/json' }
🔵 [SMOOBU ROUTES] Body: { startDate: "...", endDate: "...", ... }
[Smoobu Import Controller] previewImport called
[Smoobu Import Controller] Previewing import for date range: ...
[Smoobu Import] Starting reservation import from Smoobu

🌐 [SMOOBU API] fetchReservationsFromSmoobu called
🌐 [SMOOBU API] Start Date: ...
🌐 [SMOOBU API] End Date: ...
🌐 [SMOOBU API] API Key Present: YES
🌐 [SMOOBU API] Base URL: https://login.smoobu.com/api
🌐 [SMOOBU API] Making external call to: https://login.smoobu.com/api/reservations?page_size=100&page=1&arrivalFrom=...
✅ [SMOOBU API] Response received - Status: 200
✅ [SMOOBU API] Response data keys: ['bookings', 'total', ...]
```

## Troubleshooting Guide

### Issue 1: No Frontend Logs Appear

**Problem:** No blue/green console logs in browser

**Possible Causes:**
- Frontend code not rebuilt after changes
- Browser cache issue
- Wrong page loaded

**Solution:**
```bash
cd /home/rarnouxp/primeshine/primeshine-front
npm run build
# Or if using dev server:
npm run dev
```
Then hard refresh browser (Ctrl+Shift+R)

---

### Issue 2: Frontend Logs Show, But No Backend Route Logs

**Problem:** Browser shows request made, but no 🔵 [SMOOBU ROUTES] logs in backend

**Possible Causes:**
- Wrong API URL in frontend (check REACT_APP_API_URL in .env)
- Backend not running
- Network/firewall blocking request
- CORS issue

**Solution:**
1. Check frontend .env file:
```bash
cat /home/rarnouxp/primeshine/primeshine-front/.env
# Should show: REACT_APP_API_URL=http://localhost:3001
```

2. Check backend is running:
```bash
curl http://localhost:3001/api-docs
# Should return API documentation
```

3. Check browser Network tab for the request:
   - Look for `/api/admin/smoobu/preview-import` in Network tab
   - Check if request was sent (should show status code)
   - If shows "CORS error", check backend CORS config

---

### Issue 3: Backend Route Logs Show, But No Smoobu API Logs

**Problem:** 🔵 [SMOOBU ROUTES] logs appear, but no 🌐 [SMOOBU API] logs

**Possible Causes:**
- Controller throwing error before reaching service
- Validation failing
- Missing API key

**Solution:**
1. Check for error messages in backend logs right after route logs
2. Verify API key is set:
```bash
cd /home/rarnouxp/primeshine/primeshine-back
grep SMOOBU_API_KEY .env
# Should show: SMOOBU_API_KEY=FHpl4rGjQ74YpOAFrH82sEWhA58r6teoKFu5Djxv3A
```

3. Check if controller logs appear:
```
[Smoobu Import Controller] previewImport called
```
If this doesn't appear, there's a routing issue.

---

### Issue 4: Smoobu API Logs Show, But External Call Fails

**Problem:** 🌐 logs appear, but no ✅ success logs

**Possible Causes:**
- Invalid API key
- Network issue
- Wrong Smoobu API URL
- Smoobu API is down
- Rate limiting

**Solution:**
1. Test Smoobu API directly with curl:
```bash
curl -X GET "https://login.smoobu.com/api/reservations?page_size=10" \
  -H "Api-Key: FHpl4rGjQ74YpOAFrH82sEWhA58r6teoKFu5Djxv3A" \
  -H "Content-Type: application/json"
```

Expected response: JSON with reservations data

If this fails, the issue is with Smoobu API access (key, permissions, etc.)

2. Check for axios error in backend logs:
```
[Smoobu Import] Error fetching reservations page 1: ...
```

---

### Issue 5: Everything Works But No Reservations

**Problem:** All logs show success, but `totalFetched: 0`

**Possible Causes:**
- No reservations in Smoobu for selected date range
- Wrong date format
- Smoobu API returning empty results

**Solution:**
1. Verify reservations exist in Smoobu dashboard for your date range
2. Check the Smoobu API response in logs:
```
✅ [SMOOBU API] Response data keys: ['bookings', 'total']
```
3. Try a wider date range (e.g., last 30 days to next 90 days)

---

## Direct API Testing

Use the provided test script to bypass the frontend entirely:

```bash
cd /home/rarnouxp/primeshine/primeshine-back

# First, get your JWT token from browser:
# 1. Open browser console
# 2. Run: localStorage.getItem('token')
# 3. Copy the token value

# Edit the test script and replace YOUR_JWT_TOKEN_HERE with your actual token:
nano test-smoobu-direct.sh

# Run the test:
./test-smoobu-direct.sh
```

This will show you if the backend API works independently of the frontend.

---

## Success Indicators

When everything is working correctly, you should see this flow:

1. ✅ Frontend logs in browser console (blue/green markers)
2. ✅ Backend route logs (blue [SMOOBU ROUTES] marker)
3. ✅ Backend controller logs
4. ✅ Backend service logs (globe 🌐 [SMOOBU API] marker)
5. ✅ External API call to Smoobu
6. ✅ Response received from Smoobu
7. ✅ Data processed and returned to frontend
8. ✅ Frontend displays preview/import results

---

## Quick Checklist

- [ ] Backend server restarted after code changes
- [ ] See "✅ [APP] Registered Smoobu routes" on backend startup
- [ ] Browser console open and showing logs
- [ ] SMOOBU_API_KEY set in backend .env
- [ ] REACT_APP_API_URL set correctly in frontend .env
- [ ] Network tab shows request being sent
- [ ] Backend logs show route being hit
- [ ] Service logs show external API call
- [ ] Smoobu API responds successfully

---

## Next Steps After Debugging

Once you identify where the request flow stops, we can:
1. Fix authentication issues
2. Correct API configuration
3. Handle Smoobu API errors
4. Adjust date formatting
5. Add error handling

Please run through these steps and share:
1. What logs appear in browser console
2. What logs appear in backend terminal
3. Any error messages at any level
4. Results of the curl test (if you try it)
