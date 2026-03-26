# Implementation Plan: Women's Safety App

## Overview

Build a single self-contained `index.html` file implementing a mobile-first safety app. All HTML, CSS, and JavaScript are embedded in one file. External dependencies (Leaflet.js, fast-check) are loaded from CDN. Persistence uses localStorage. Device capabilities use standard browser APIs.

## Tasks

- [x] 1. Scaffold the HTML shell and app structure
  - Create `index.html` with embedded `<style>` and `<script>` blocks
  - Define the three screen panels: Home, Contacts, Safe Map
  - Add bottom tab navigation bar with Home / Contacts / Map tabs
  - Implement `showScreen(name)` to toggle panel visibility via CSS display
  - Load Leaflet.js CSS and JS from CDN in `<head>`
  - Define `AppState` in-memory object with `currentScreen`, `liveTracking`, `watchId`, `trackingInterval`, `lastSosResult`
  - _Requirements: 1.1, 4.1_

- [x] 2. Implement StorageAPI and TrustedContact data model
  - [x] 2.1 Implement `StorageAPI` module
    - Write `getContacts()`, `saveContacts(arr)`, `addContact(c)`, `removeContact(id)` using `localStorage` key `"wsa_contacts"`
    - Throw a `DuplicateError` when a phone number already exists
    - Catch `QuotaExceededError` and surface a toast message
    - _Requirements: 2.1, 2.3, 2.7_

  - [ ]* 2.2 Write property test for StorageAPI round-trip (Property 7)
    - **Property 7: Contact persistence round-trip**
    - **Validates: Requirements 2.1, 2.7**

  - [ ]* 2.3 Write property test for duplicate rejection (Property 6)
    - **Property 6: Duplicate contact rejection**
    - **Validates: Requirements 2.3**

  - [ ]* 2.4 Write property test for contact removal (Property 8)
    - **Property 8: Contact removal round-trip**
    - **Validates: Requirements 2.4, 2.5**

- [x] 3. Implement GpsAPI
  - [x] 3.1 Implement `GpsAPI` module
    - Write `getCurrentPosition(timeoutMs)` wrapping `navigator.geolocation.getCurrentPosition` as a Promise with a 5-second timeout
    - Maintain `GpsAPI.lastKnown` cache updated on every successful fix
    - Write `watchPosition(callback)` and `clearWatch(watchId)`
    - _Requirements: 1.2, 5.1, 5.2, 5.3, 5.5_

- [x] 4. Implement SmsAPI and alert message builder
  - [x] 4.1 Implement `SmsAPI` module
    - Write `buildAlertMessage(name, coords, timestamp)` producing a string with name, lat/lon, ISO timestamp, and a `https://maps.google.com/?q={lat},{lon}` link
    - Write `sendAlert(contacts, message)` dispatching via `sms:` URI scheme (multi-recipient body); fall back to `navigator.share`; fall back to clipboard copy
    - _Requirements: 1.3, 6.4_

  - [ ]* 4.2 Write property test for alert message completeness (Property 1)
    - **Property 1: Alert message completeness**
    - **Validates: Requirements 1.3, 6.4**

- [x] 5. Implement SOSManager and Home screen SOS button
  - [x] 5.1 Implement `SOSManager.trigger()`
    - Call `GpsAPI.getCurrentPosition(5000)`; on timeout use `GpsAPI.lastKnown` with `locationStale: true`
    - If no contacts saved, block send and show inline prompt
    - Call `SmsAPI.sendAlert` for each contact; collect failures
    - Return `SOSResult` with `timestamp`, `coords`, `locationStale`, `notified`
    - _Requirements: 1.2, 1.3, 1.4, 1.5, 1.6_

  - [x] 5.2 Wire SOS button on Home screen
    - Add large SOS button element to Home screen HTML
    - On click, call `SOSManager.trigger()` and render confirmation screen overlay with timestamp and notified contacts list
    - Show failure notification if any contacts were not reached
    - _Requirements: 1.1, 1.7_

  - [ ]* 5.3 Write property test for GPS timeout fallback (Property 2)
    - **Property 2: GPS timeout fallback uses last-known location**
    - **Validates: Requirements 1.2, 1.5**

  - [ ]* 5.4 Write property test for failed-contact notification (Property 3)
    - **Property 3: Failed-contact notification completeness**
    - **Validates: Requirements 1.6**

  - [ ]* 5.5 Write property test for SOS confirmation screen content (Property 4)
    - **Property 4: SOS confirmation screen content**
    - **Validates: Requirements 1.7**

- [x] 6. Checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [x] 7. Implement Contacts screen UI and contact management
  - [x] 7.1 Build Contacts screen HTML and render loop
    - Render saved contacts list from `StorageAPI.getContacts()`
    - Add "Add Contact" button that opens an inline add-contact form
    - Each contact row has a remove button wired to `StorageAPI.removeContact(id)` followed by re-render
    - _Requirements: 2.1, 2.4, 2.5_

  - [x] 7.2 Implement phone number validation and add-contact form
    - Write `validatePhone(str)` accepting E.164 (`+` followed by 7–15 digits) or common local formats
    - On form submit: validate phone, check for duplicate via `StorageAPI.addContact`, show inline field errors for invalid format or duplicate
    - _Requirements: 2.2, 2.3_

  - [x] 7.3 Implement native contact import via Contact Picker API
    - Add "Import from Contacts" button; call `navigator.contacts.select(['name','tel'])` when available
    - Pre-fill add-contact form with selected contact's name and first phone number
    - Hide button if `navigator.contacts` is not available
    - _Requirements: 2.6_

  - [ ]* 7.4 Write property test for phone number validation (Property 5)
    - **Property 5: Phone number validation**
    - **Validates: Requirements 2.2**

- [x] 8. Implement LiveTracker and home screen toggle
  - [x] 8.1 Implement `LiveTracker` module
    - Write `start(contacts)`: call `GpsAPI.watchPosition`, set `setInterval` at 30 s, send location SMS each tick, set `AppState.liveTracking = true`
    - Write `stop(contacts)`: clear interval, call `GpsAPI.clearWatch`, send session-ended SMS, set `AppState.liveTracking = false`
    - If GPS becomes unavailable mid-session, send "location temporarily unavailable" SMS and resume on next fix
    - _Requirements: 3.2, 3.3, 3.5, 3.6_

  - [x] 8.2 Wire Live Tracking toggle on Home screen
    - Add toggle switch element to Home screen
    - On enable: guard for empty contacts list (show prompt); call `LiveTracker.start`; show active indicator
    - On disable: call `LiveTracker.stop`; hide active indicator
    - _Requirements: 3.1, 3.4, 3.7_

  - [ ]* 8.3 Write property test for live tracking start/stop round-trip (Property 9)
    - **Property 9: Live tracking start/stop round-trip**
    - **Validates: Requirements 3.2, 3.3**

- [x] 9. Implement OverpassAPI
  - [x] 9.1 Implement `OverpassAPI.fetchNearby(lat, lon, radiusMeters)`
    - Build Overpass QL query requesting `amenity=police` and `amenity=hospital` nodes within `radiusMeters` of `(lat, lon)`
    - Fetch from `https://overpass-api.de/api/interpreter`
    - Normalize response elements into `SafePlace[]` objects
    - On network error or timeout, throw so callers can show the retry toast
    - _Requirements: 4.3_

  - [ ]* 9.2 Write property test for Overpass query radius and amenity types (Property 10)
    - **Property 10: Overpass query uses correct radius**
    - **Validates: Requirements 4.3**

- [x] 10. Implement MapView and Safe Places Map screen
  - [x] 10.1 Implement `MapView` module
    - Write `init(containerId)`: initialise Leaflet map with OpenStreetMap tile layer
    - Write `setUserLocation(coords)`: place/update a user marker and pan to it
    - Write `renderPlaces(places)`: add colour-coded markers (blue = police, red = hospital) for each `SafePlace`
    - Write `showPlaceDetail(place, userCoords)`: open a Leaflet popup with name, address, computed straight-line distance, and a "Get Directions" link to `https://www.google.com/maps/dir/?api=1&destination={lat},{lon}`
    - _Requirements: 4.2, 4.4, 4.5, 4.6_

  - [x] 10.2 Wire Safe Places Map screen
    - On tab open: call `GpsAPI.getCurrentPosition`, then `OverpassAPI.fetchNearby(lat, lon, 5000)`, then `MapView.init` + `setUserLocation` + `renderPlaces`
    - On empty results: show "No safe places found" message with "Expand to 10 km" button that re-calls `fetchNearby` with `10000`
    - On GPS denied: show permission-explanation card with link to device settings
    - On Overpass error: show toast with retry button
    - _Requirements: 4.1, 4.2, 4.3, 4.7, 4.8_

  - [ ]* 10.3 Write property test for map marker category correctness (Property 11)
    - **Property 11: Map marker category correctness**
    - **Validates: Requirements 4.4**

  - [ ]* 10.4 Write property test for place detail popup content (Property 12)
    - **Property 12: Place detail popup content**
    - **Validates: Requirements 4.5**

  - [ ]* 10.5 Write property test for navigation URL construction (Property 13)
    - **Property 13: Navigation URL construction**
    - **Validates: Requirements 4.6**

- [x] 11. Checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [x] 12. Implement permission gate and first-launch flow
  - Add a permission-gate overlay shown on first launch
  - Request `navigator.geolocation` permission; on grant proceed to Home screen; on deny show explanation card with settings deep-link
  - Show SMS permission explanation card (browser cannot request SMS permission directly; explain the `sms:` URI approach)
  - _Requirements: 5.1, 5.2, 5.3, 6.1, 6.2, 6.3_

- [x] 13. Apply mobile-first CSS and native-app styling
  - Embed all styles in a single `<style>` block
  - Use CSS custom properties for the colour palette (primary pink/red, neutral grays)
  - Style the SOS button as a large circular tap target (min 80 px diameter)
  - Style bottom tab bar with active-state indicators
  - Ensure all interactive elements meet minimum 44 × 44 px touch target size
  - Apply `viewport` meta tag and `box-sizing: border-box` reset
  - _Requirements: 1.1, 3.1, 3.4_

- [ ] 14. Final integration and wiring
  - [ ] 14.1 Wire all modules together inside a single `DOMContentLoaded` listener
    - Initialise `AppState`, call `StorageAPI.getContacts()` to hydrate contacts list, attach all event listeners
    - Ensure tab navigation, SOS button, live tracking toggle, contacts CRUD, and map screen all function end-to-end within the single HTML file
    - _Requirements: 1.1, 2.1, 3.1, 4.1_

  - [ ]* 14.2 Write integration tests for SOSManager end-to-end flow
    - Mock `GpsAPI` and `SmsAPI`; test trigger with 1 contact, 3 contacts, and empty contact list
    - _Requirements: 1.2, 1.3, 1.4, 1.5, 1.6, 1.7_

- [ ] 15. Final checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for a faster MVP
- All code lives in a single `index.html` — no separate JS or CSS files
- Tests live in `tests/` and are run with Vitest (`vitest --run`) using fast-check for property tests
- Each task references specific requirements for traceability
- Property tests validate universal correctness properties; unit tests cover specific examples and edge cases
