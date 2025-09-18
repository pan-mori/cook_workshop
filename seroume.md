# Bug Report - Point 4: Dark Overlay on https://living.para-bellum.com/profile

## Problem
The page https://living.para-bellum.com/profile displays a dark overlay covering the entire page, making it impossible for users to see the content. The page is unusable.

## Identified Console Errors:

### 1. StoreSelector.jsx:20 - TypeError
```
TypeError: Cannot read properties of undefined (reading 'find')
at ot (StoreSelector.jsx:40:18)
```
**Description:** StoreSelector component is trying to call the `find()` method on an undefined object at line 40.

### 2. Failed stores data loading
```
StoreSelector.jsx:24 Failed to load stores data.
```
**Description:** Failed to load store data, likely due to API issues or incorrect URL.

### 3. profile:17 - Undefined remove error
```
Uncaught TypeError: Cannot read properties of undefined (reading 'remove')
at profile:17:48
```
**Description:** Attempting to call `remove()` method on undefined object in profile script.

### 4. Google Maps API issue
```
Google Maps JavaScript API has been loaded directly without loading=async
GET https://maps.googleapis.com/maps/api/mapsjs/gen_204?csp_test=true net::ERR_BLOCKED_BY_CLIENT
```
**Description:** Suboptimal Google Maps API loading and blocked request (likely by ad-blocker).

## Solutions:

### 1. Fix for StoreSelector.jsx
```javascript
// StoreSelector.jsx line 40
// Before:
const store = stores.find(s => s.id === selectedStoreId);

// After (with defensive programming):
const store = stores && Array.isArray(stores) ? stores.find(s => s.id === selectedStoreId) : null;
```

### 2. Fix for stores data loading
```javascript
// Add error handling and fallback
try {
  const storesData = await fetch('/api/stores');
  if (!storesData.ok) {
    throw new Error(`HTTP error! status: ${storesData.status}`);
  }
  const stores = await storesData.json();
  setStores(stores || []);
} catch (error) {
  console.error('Failed to load stores:', error);
  setStores([]); // fallback to empty array
  setError('Failed to load stores. Please refresh the page.');
}
```

### 3. Fix for profile script
```javascript
// profile.js line 17
// Before:
element.remove();

// After:
if (element && typeof element.remove === 'function') {
  element.remove();
}
```

### 4. Fix for Google Maps API
```html
<!-- Before: -->
<script src="https://maps.googleapis.com/maps/api/js?key=API_KEY&libraries=places"></script>

<!-- After: -->
<script async defer src="https://maps.googleapis.com/maps/api/js?key=API_KEY&libraries=places&callback=initMap"></script>
```


**Root Cause of Dark Overlay:** Most likely the JavaScript error in StoreSelector component causes a crash and displays an error overlay instead of proper page content.
