# Negative Test Cases

Comprehensive list of negative test cases for the Parcel-Safe Smart Top Box delivery system. These cases validate error handling, failure recovery, and system resilience.

---

## 🔐 Authentication & Authorization Negatives (NC-AUTH)

### NC-AUTH-01: Invalid Google Token
**Input:** Malformed/expired Google OAuth token
**Expected:** Auth rejected, error message displayed, no session created
**Component:** Mobile App

---

### NC-AUTH-02: Revoked Account Access
**Input:** User attempts login with suspended account
**Expected:** "Account suspended" message, no access granted
**Component:** Mobile App

---

### NC-AUTH-03: Expired Session Token
**Input:** Request with expired JWT token
**Expected:** 401 Unauthorized, prompt to re-login
**Component:** All APIs

---

### NC-AUTH-04: Missing Authorization Header
**Input:** API request without auth header
**Expected:** 401 Unauthorized response
**Component:** Backend

---

### NC-AUTH-05: Invalid Share Token
**Input:** Non-existent or malformed tracking token
**Expected:** 404 page with "Token not found" message
**Component:** Web Portal

---

### NC-AUTH-06: Expired Share Token
**Input:** Token past expiration timestamp
**Expected:** "Link expired" page with contact support CTA
**Component:** Web Portal

---

### NC-AUTH-07: Tampered JWT Signature
**Input:** JWT with modified payload but original signature
**Expected:** 401 Unauthorized, signature validation failure
**Component:** Backend

---

### NC-AUTH-08: SQL Injection in Login
**Input:** Email field with `'; DROP TABLE users;--`
**Expected:** Sanitized input, normal auth failure message
**Component:** Admin Portal

---

### NC-AUTH-09: XSS Attack in Token Parameter
**Input:** URL with `<script>alert('xss')</script>` in token
**Expected:** Escaped output, no script execution
**Component:** Web Portal

---

### NC-AUTH-10: Brute Force Login Attempt
**Input:** 20+ failed login attempts in 1 minute
**Expected:** Account temporarily locked, cooldown message
**Component:** Admin Portal

---

## ❌ OTP Validation Negatives (NC-OTP)

### NC-OTP-01: Incorrect 6-Digit Code
**Input:** Valid format but wrong OTP (e.g., 123456)
**Expected:** Reject, LED flash red, attempt counter ++
**Component:** Box (ESP32)

---

### NC-OTP-02: Non-Numeric Characters
**Input:** "ABCDEF" or "12AB34"
**Expected:** Reject immediately, invalid format
**Component:** Box (ESP32)

---

### NC-OTP-03: Too Few Digits
**Input:** "12345" (5 digits)
**Expected:** Wait for more input or timeout
**Component:** Box (ESP32)

---

### NC-OTP-04: Too Many Digits
**Input:** "1234567" (7 digits)
**Expected:** Only first 6 processed, or reject
**Component:** Box (ESP32)

---

### NC-OTP-05: Empty Input Submission
**Input:** Keypad # pressed with no digits
**Expected:** No action, wait for input
**Component:** Box (ESP32)

---

### NC-OTP-06: Lockout After 5 Failures
**Input:** 5 consecutive wrong OTPs
**Expected:** 5-minute lockout, keypad disabled, alert sent
**Component:** Box (ESP32)

---

### NC-OTP-07: OTP Attempt During Lockout
**Input:** Correct OTP entered during lockout period
**Expected:** Reject with "Try again later" indication
**Component:** Box (ESP32)

---

### NC-OTP-08: Expired OTP Code
**Input:** Valid OTP but delivery cancelled/past 4 hours
**Expected:** Reject, "OTP expired" status
**Component:** Box (ESP32)

---

### NC-OTP-09: OTP for Different Box
**Input:** OTP generated for Box A entered on Box B
**Expected:** Hash mismatch, reject
**Component:** Box (ESP32)

---

### NC-OTP-10: Replay Attack - Same OTP Twice
**Input:** Correct OTP entered second time after successful unlock
**Expected:** OTP already consumed, reject
**Component:** Box (ESP32)

---

### NC-OTP-11: OTP Without Active Delivery
**Input:** Any 6-digit code when no delivery assigned
**Expected:** "No active delivery" error
**Component:** Box (ESP32)

---

### NC-OTP-12: Null OTP Hash in Cache
**Input:** Box cache corrupted, OTP hash missing
**Expected:** Graceful fallback, online validation attempt
**Component:** Box (ESP32)

---

## 🌐 Network Failure Negatives (NC-NET)

### NC-NET-01: WiFi Password Changed
**Input:** Box tries to connect with old credentials
**Expected:** Auth failure, retry with backoff, alert admin
**Component:** Box (ESP32)

---

### NC-NET-02: Firebase Unreachable
**Input:** Firebase server down or blocked
**Expected:** Local operations continue, queue updates
**Component:** All Components

---

### NC-NET-03: DNS Resolution Failure
**Input:** Cannot resolve Firebase hostname
**Expected:** Retry DNS, fallback to IP if available
**Component:** Box (ESP32)

---

### NC-NET-04: SSL/TLS Handshake Failure
**Input:** Certificate validation fails
**Expected:** Connection refused, do not send sensitive data
**Component:** Box (ESP32)

---

### NC-NET-05: Network Timeout (30s)
**Input:** Request takes >30 seconds
**Expected:** Abort request, retry with backoff
**Component:** Mobile App

---

### NC-NET-06: Intermittent Connection (Flapping)
**Input:** WiFi connects/disconnects rapidly
**Expected:** Debounce reconnection, don't reset state
**Component:** Box (ESP32)

---

### NC-NET-07: Slow Upload (Photo >60s)
**Input:** Large photo on 2G network
**Expected:** Timeout handling, resume support
**Component:** Box (ESP32)

---

### NC-NET-08: API Rate Limited
**Input:** Exceed 100 requests/minute
**Expected:** 429 response, exponential backoff
**Component:** Mobile App

---

### NC-NET-09: Partial Data Received
**Input:** Network cuts during Firebase response
**Expected:** Discard partial, retry full request
**Component:** Box (ESP32)

---

### NC-NET-10: CORS Error on Web
**Input:** API endpoint without proper CORS headers
**Expected:** Request blocked, error logged, user notified
**Component:** Web Portal

---

## 📍 GPS Negatives (NC-GPS)

### NC-GPS-01: GPS Module Returns (0, 0)
**Input:** Cold start or no fix
**Expected:** Discard, use last known location
**Component:** Box (ESP32)

---

### NC-GPS-02: GPS Returns Same Coords for 10 Minutes
**Input:** Stuck GPS module
**Expected:** Flag as stale, switch to phone GPS
**Component:** Box (ESP32)

---

### NC-GPS-03: Latitude Out of Range
**Input:** lat = 95.0 (valid: -90 to 90)
**Expected:** Discard invalid data
**Component:** Box (ESP32)

---

### NC-GPS-04: Longitude Out of Range
**Input:** lng = 200.0 (valid: -180 to 180)
**Expected:** Discard invalid data
**Component:** Box (ESP32)

---

### NC-GPS-05: Position Jump >10km in 1 Second
**Input:** Teleport coordinates (spoofing)
**Expected:** Flag anomaly, alert admin
**Component:** Backend

---

### NC-GPS-06: Speed >300 km/h
**Input:** Calculated velocity exceeds possible
**Expected:** Flag as spoofed, use previous location
**Component:** Backend

---

### NC-GPS-07: GPS Coordinates as Strings
**Input:** lat = "invalid", lng = "data"
**Expected:** Parse failure, discard
**Component:** All Components

---

### NC-GPS-08: NaN Coordinates
**Input:** lat = NaN, lng = NaN
**Expected:** Type check fails, discard
**Component:** All Components

---

### NC-GPS-09: Negative Altitude (Underground)
**Input:** altitude = -500
**Expected:** Accept (mines exist), flag for review
**Component:** Backend

---

### NC-GPS-10: Heading = -1 (Invalid)
**Input:** Heading outside 0-360 range
**Expected:** Discard heading, keep coords
**Component:** Box (ESP32)

---

## 📷 Camera & Photo Negatives (NC-CAM)

### NC-CAM-01: Camera Not Connected
**Input:** Camera module unplugged
**Expected:** Camera init fails, alert admin, allow delivery without photo (flagged)
**Component:** Box (ESP32)

---

### NC-CAM-02: Camera Returns Black Image
**Input:** Lens cap on or complete darkness
**Expected:** Detect low entropy, retry with flash
**Component:** Box (ESP32)

---

### NC-CAM-03: Camera Returns Corrupt Data
**Input:** Random bytes instead of JPEG
**Expected:** JPEG validation fails, retry capture
**Component:** Box (ESP32)

---

### NC-CAM-04: SPIFFS Full for Photo
**Input:** Storage at 100% capacity
**Expected:** Delete oldest failed photo, write new
**Component:** Box (ESP32)

---

### NC-CAM-05: Photo Upload 403 Forbidden
**Input:** Firebase Storage rules reject upload
**Expected:** Log detailed error, alert admin
**Component:** Box (ESP32)

---

### NC-CAM-06: Photo File 0 Bytes
**Input:** Empty file saved
**Expected:** Detect empty file, retry capture
**Component:** Box (ESP32)

---

### NC-CAM-07: Photo >5MB
**Input:** Uncompressed large image
**Expected:** Compress before upload, or reject if can't
**Component:** Box (ESP32)

---

### NC-CAM-08: Camera I2C Communication Error
**Input:** Wiring issue or interference
**Expected:** Retry 3x, then fallback mode
**Component:** Box (ESP32)

---

### NC-CAM-09: Overwritten Photo Before Upload
**Input:** New photo captured before old uploaded
**Expected:** Queue maintains order, both uploaded
**Component:** Box (ESP32)

---

### NC-CAM-10: Photo with PII Not Blurred
**Input:** License plate visible in frame
**Expected:** Auto-blur or flag for admin review
**Component:** Backend (Future)

---

## 🔒 Hardware Security Negatives (NC-HW)

### NC-HW-01: Solenoid Not Responding
**Input:** Unlock command sent, feedback unchanged
**Expected:** Retry 3x, then report hardware failure
**Component:** Box (ESP32)

---

### NC-HW-02: Tamper Switch Always Open
**Input:** Damaged reed switch
**Expected:** Constant tamper state, alert admin immediately
**Component:** Box (ESP32)

---

### NC-HW-03: Tamper Switch Always Closed
**Input:** Switch stuck or bypassed
**Expected:** No tamper detection, manual inspection needed
**Component:** Box (ESP32)

---

### NC-HW-04: Solenoid Overheating
**Input:** Held open >5 seconds
**Expected:** Auto-cutoff at 5s limit, cool down period
**Component:** Box (ESP32)

---

### NC-HW-05: Battery Voltage Reads 0
**Input:** ADC failure or disconnected battery
**Expected:** Default to 100% or alert critical
**Component:** Box (ESP32)

---

### NC-HW-06: Keypad Button Stuck
**Input:** Key '5' continuously pressed
**Expected:** Debounce ignores repeated, timeout reset
**Component:** Box (ESP32)

---

### NC-HW-07: Power Brown-out During Unlock
**Input:** Voltage dip resets ESP32
**Expected:** OTP still valid after reboot, retry allowed
**Component:** Box (ESP32)

---

### NC-HW-08: SPIFFS Corrupted
**Input:** Flash memory damage
**Expected:** Detect corruption, reinit SPIFFS, log loss
**Component:** Box (ESP32)

---

### NC-HW-09: RTC Memory Lost
**Input:** Power loss too long
**Expected:** Refetch state from Firebase on boot
**Component:** Box (ESP32)

---

### NC-HW-10: LED Not Illuminating
**Input:** LED burnt out
**Expected:** Box functions normally, visual feedback lost
**Component:** Box (ESP32)

---

## 📱 Mobile App Negatives (NC-MOB)

### NC-MOB-01: Location Permission Denied
**Input:** User denies location access
**Expected:** Show requirement message, disable delivery features
**Component:** Mobile App

---

### NC-MOB-02: Push Notification Permission Denied
**Input:** User blocks notifications
**Expected:** Show warning, allow app use without push
**Component:** Mobile App

---

### NC-MOB-03: App Killed by OS
**Input:** System kills app for memory
**Expected:** Foreground service keeps tracking alive (Android)
**Component:** Mobile App

---

### NC-MOB-04: Invalid Delivery ID in Cache
**Input:** Delivery ID deleted server-side
**Expected:** Clear cache, show "Delivery not found"
**Component:** Mobile App

---

### NC-MOB-05: Parse Error in API Response
**Input:** Malformed JSON from server
**Expected:** Catch error, show generic failure message
**Component:** Mobile App

---

### NC-MOB-06: Image Load Failure
**Input:** Photo URL returns 404
**Expected:** Show placeholder, offer retry
**Component:** Mobile App

---

### NC-MOB-07: Map Renderer Crash
**Input:** Out of memory for tiles
**Expected:** Graceful degradation, text-only mode
**Component:** Mobile App

---

### NC-MOB-08: Deep Link to Non-Existent Delivery
**Input:** Notification for deleted delivery
**Expected:** Show "Delivery not found" screen
**Component:** Mobile App

---

### NC-MOB-09: Offline Queue Overflow
**Input:** 100+ pending updates while offline
**Expected:** Drop oldest, keep latest 50
**Component:** Mobile App

---

### NC-MOB-10: Background Location Update Failure
**Input:** OS restricts background activity
**Expected:** Warn user, request battery optimization exception
**Component:** Mobile App

---

## 🌐 Web Portal Negatives (NC-WEB)

### NC-WEB-01: JavaScript Disabled
**Input:** User has JS disabled in browser
**Expected:** Show noscript message, basic info available
**Component:** Web Portal

---

### NC-WEB-02: Unsupported Browser
**Input:** Internet Explorer 11
**Expected:** Show upgrade browser message
**Component:** Web Portal

---

### NC-WEB-03: Map API Key Invalid
**Input:** Google Maps API key revoked
**Expected:** Map shows error state, text location shown
**Component:** Web Portal

---

### NC-WEB-04: Firebase Subscription Error
**Input:** onValue() fails to connect
**Expected:** Show "Connecting..." with retry
**Component:** Web Portal

---

### NC-WEB-05: Hydration Mismatch
**Input:** Server/client render differently
**Expected:** Client takes over, no visible error
**Component:** Web Portal

---

### NC-WEB-06: Cookie Blocked
**Input:** Third-party cookies disabled
**Expected:** Session works via localStorage fallback
**Component:** Web Portal

---

### NC-WEB-07: localStorage Quota Exceeded
**Input:** Too much cached data
**Expected:** Clear old data, continue
**Component:** Web Portal

---

### NC-WEB-08: WebSocket Connection Failed
**Input:** Corporate firewall blocks WSS
**Expected:** Fall back to long-polling
**Component:** Web Portal

---

### NC-WEB-09: Image CDN Blocked
**Input:** Firebase Storage blocked in region
**Expected:** Proxy through API, show fallback
**Component:** Web Portal

---

### NC-WEB-10: Server Error 500
**Input:** Backend exception
**Expected:** Show friendly error page with retry
**Component:** Web Portal

---

## 📊 Data Integrity Negatives (NC-DATA)

### NC-DATA-01: Duplicate Delivery Insert
**Input:** Same delivery_id submitted twice
**Expected:** Upsert or reject duplicate
**Component:** Backend

---

### NC-DATA-02: Foreign Key Violation
**Input:** Rider ID doesn't exist
**Expected:** Reject insert, return clear error
**Component:** Backend

---

### NC-DATA-03: Required Field Null
**Input:** delivery.dropoff_address = null
**Expected:** Validation fails, 400 response
**Component:** Backend

---

### NC-DATA-04: Invalid Enum Value
**Input:** status = "FLYING" (not in enum)
**Expected:** Validation fails
**Component:** Backend

---

### NC-DATA-05: Out-of-Order Status Update
**Input:** COMPLETED before IN_TRANSIT
**Expected:** State machine rejects
**Component:** Backend

---

### NC-DATA-06: Timestamp in Future
**Input:** Event timestamp = +1 year
**Expected:** Reject or flag for review
**Component:** Backend

---

### NC-DATA-07: Negative Delivery ID
**Input:** delivery_id = -5
**Expected:** Validation fails
**Component:** Backend

---

### NC-DATA-08: OTP Hash Too Short
**Input:** 10 character hash (should be 64)
**Expected:** Validation fails
**Component:** Backend

---

### NC-DATA-09: Unicode Emoji in Address
**Input:** "123 Main St 🏠"
**Expected:** Accepted if valid UTF-8
**Component:** Backend

---

### NC-DATA-10: SQL Injection in Search
**Input:** search = "'; DROP TABLE--"
**Expected:** Parameterized query, no injection
**Component:** Backend

---

## Summary

| Category | Count |
|----------|-------|
| 🔐 Authentication & Authorization | 10 |
| ❌ OTP Validation | 12 |
| 🌐 Network Failure | 10 |
| 📍 GPS | 10 |
| 📷 Camera & Photo | 10 |
| 🔒 Hardware Security | 10 |
| 📱 Mobile App | 10 |
| 🌐 Web Portal | 10 |
| 📊 Data Integrity | 10 |
| **Total** | **92** |

---

## Testing Strategy

1. **Unit Tests**: NC-OTP-*, NC-DATA-*, NC-GPS-* logic validation
2. **Integration Tests**: NC-NET-*, NC-AUTH-* with mocked services
3. **Hardware Tests**: NC-HW-*, NC-CAM-* with actual modules
4. **E2E Tests**: NC-MOB-*, NC-WEB-* full flow testing
5. **Security Tests**: NC-AUTH-08, NC-AUTH-09, NC-DATA-10 penetration testing
