# Boundary Test Cases

Comprehensive list of boundary condition test cases for the Parcel-Safe Smart Top Box delivery system. These cases validate system behavior at the limits of valid input ranges.

---

## 🔢 Numeric Boundaries (BC-NUM)

### BC-NUM-01: OTP = 000000 (Minimum)
**Input:** 6 zeros
**Expected:** Valid OTP format, processed normally
**Component:** Box (ESP32)

---

### BC-NUM-02: OTP = 999999 (Maximum)
**Input:** 6 nines
**Expected:** Valid OTP format, processed normally
**Component:** Box (ESP32)

---

### BC-NUM-03: OTP = 100000 (Just Above Min)
**Input:** Smallest 6-digit number
**Expected:** Valid format, processed normally
**Component:** Box (ESP32)

---

### BC-NUM-04: Battery = 0%
**Input:** ADC reads minimum voltage
**Expected:** Critical alert, shutdown warning
**Component:** Box (ESP32)

---

### BC-NUM-05: Battery = 1%
**Input:** Just above complete depletion
**Expected:** Critical alert displayed
**Component:** Box (ESP32)

---

### BC-NUM-06: Battery = 20% (Low Threshold)
**Input:** Exactly on warning threshold
**Expected:** Low battery warning triggered
**Component:** Box (ESP32)

---

### BC-NUM-07: Battery = 21% (Above Low)
**Input:** Just above warning threshold
**Expected:** Normal operation, no warning
**Component:** Box (ESP32)

---

### BC-NUM-08: Battery = 100%
**Input:** Full charge
**Expected:** Show full indicator
**Component:** Box (ESP32)

---

### BC-NUM-09: Failed OTP Attempts = 4
**Input:** One attempt remaining
**Expected:** Warning shown, one more chance
**Component:** Box (ESP32)

---

### BC-NUM-10: Failed OTP Attempts = 5
**Input:** Exactly at lockout threshold
**Expected:** Lockout triggered immediately
**Component:** Box (ESP32)

---

### BC-NUM-11: Photo Queue = 0
**Input:** Nothing to upload
**Expected:** Queue processor idle
**Component:** Box (ESP32)

---

### BC-NUM-12: Photo Queue = 10 (Max)
**Input:** Queue at capacity
**Expected:** Oldest deleted if new photo needed
**Component:** Box (ESP32)

---

### BC-NUM-13: Photo Queue = 9
**Input:** One slot remaining
**Expected:** Accept new photo normally
**Component:** Box (ESP32)

---

### BC-NUM-14: Upload Retry Count = 0
**Input:** First attempt
**Expected:** Immediate upload attempt
**Component:** Box (ESP32)

---

### BC-NUM-15: Upload Retry Count = 5 (Max)
**Input:** Final attempt reached
**Expected:** Mark as failed, alert admin
**Component:** Box (ESP32)

---

## 📍 Geographic Boundaries (BC-GEO)

### BC-GEO-01: Latitude = -90.0 (South Pole)
**Input:** Minimum valid latitude
**Expected:** Accept as valid coordinates
**Component:** All Components

---

### BC-GEO-02: Latitude = 90.0 (North Pole)
**Input:** Maximum valid latitude
**Expected:** Accept as valid coordinates
**Component:** All Components

---

### BC-GEO-03: Latitude = -90.000001
**Input:** Just below valid range
**Expected:** Reject as invalid
**Component:** All Components

---

### BC-GEO-04: Latitude = 90.000001
**Input:** Just above valid range
**Expected:** Reject as invalid
**Component:** All Components

---

### BC-GEO-05: Longitude = -180.0 (Date Line West)
**Input:** Minimum valid longitude
**Expected:** Accept as valid coordinates
**Component:** All Components

---

### BC-GEO-06: Longitude = 180.0 (Date Line East)
**Input:** Maximum valid longitude
**Expected:** Accept as valid coordinates
**Component:** All Components

---

### BC-GEO-07: Longitude = -180.000001
**Input:** Just below valid range
**Expected:** Reject as invalid
**Component:** All Components

---

### BC-GEO-08: Longitude = 180.000001
**Input:** Just above valid range
**Expected:** Reject as invalid
**Component:** All Components

---

### BC-GEO-09: Distance = 49.9 meters
**Input:** Just inside geofence (50m)
**Expected:** Arrival triggered, OTP revealed
**Component:** Web/Mobile

---

### BC-GEO-10: Distance = 50.0 meters
**Input:** Exactly on geofence boundary
**Expected:** Arrival triggered (inclusive)
**Component:** Web/Mobile

---

### BC-GEO-11: Distance = 50.1 meters
**Input:** Just outside geofence
**Expected:** Still "In Transit", OTP hidden
**Component:** Web/Mobile

---

### BC-GEO-12: Speed = 0 km/h
**Input:** Stationary
**Expected:** Valid, no anomaly flag
**Component:** Backend

---

### BC-GEO-13: Speed = 200 km/h
**Input:** Max realistic speed (highway)
**Expected:** Valid, no anomaly flag
**Component:** Backend

---

### BC-GEO-14: Speed = 201 km/h
**Input:** Just above anomaly threshold
**Expected:** Flag for GPS spoofing review
**Component:** Backend

---

### BC-GEO-15: Position Jump = 10 km
**Input:** Exactly on anomaly threshold
**Expected:** Flag for review (inclusive)
**Component:** Backend

---

## ⏱️ Time Boundaries (BC-TIME)

### BC-TIME-01: OTP Age = 0 seconds
**Input:** Just generated
**Expected:** Valid, accept OTP
**Component:** Box (ESP32)

---

### BC-TIME-02: OTP Age = 3 hours 59 minutes
**Input:** Just before 4-hour expiry
**Expected:** Valid, accept OTP
**Component:** Box (ESP32)

---

### BC-TIME-03: OTP Age = 4 hours exactly
**Input:** Exactly at expiry
**Expected:** Expired, reject OTP
**Component:** Box (ESP32)

---

### BC-TIME-04: OTP Age = 4 hours 1 second
**Input:** Just after expiry
**Expected:** Expired, reject OTP
**Component:** Box (ESP32)

---

### BC-TIME-05: Lockout Duration = 4 minutes 59 seconds
**Input:** Almost through lockout
**Expected:** Still locked, reject attempts
**Component:** Box (ESP32)

---

### BC-TIME-06: Lockout Duration = 5 minutes
**Input:** Exactly at unlock time
**Expected:** Lockout cleared, attempts allowed
**Component:** Box (ESP32)

---

### BC-TIME-07: Session Token Age = 29 days 23 hours
**Input:** Just before 30-day expiry
**Expected:** Valid session
**Component:** Mobile App

---

### BC-TIME-08: Session Token Age = 30 days
**Input:** Exactly at expiry
**Expected:** Session expired, prompt re-login
**Component:** Mobile App

---

### BC-TIME-09: GPS Update Interval = 0 seconds
**Input:** Instant update
**Expected:** Rate limit applied (min 1s)
**Component:** Box (ESP32)

---

### BC-TIME-10: GPS Staleness = 59 seconds
**Input:** Almost stale
**Expected:** Show current location
**Component:** Web Portal

---

### BC-TIME-11: GPS Staleness = 60 seconds
**Input:** Exactly at stale threshold
**Expected:** Show "Last seen X ago"
**Component:** Web Portal

---

### BC-TIME-12: Share Token Validity = 0 days
**Input:** Expired at creation (edge case)
**Expected:** Immediate "expired" message
**Component:** Web Portal

---

### BC-TIME-13: Share Token Validity = 30 days (Max)
**Input:** Maximum allowed validity
**Expected:** Accept within 30 days
**Component:** Web Portal

---

### BC-TIME-14: Solenoid Active = 4999 ms
**Input:** Just under safety limit
**Expected:** Normal operation
**Component:** Box (ESP32)

---

### BC-TIME-15: Solenoid Active = 5000 ms
**Input:** Exactly at safety limit
**Expected:** Auto-cutoff triggered
**Component:** Box (ESP32)

---

## 📝 String Length Boundaries (BC-STR)

### BC-STR-01: Tracking Number = "" (Empty)
**Input:** Empty string
**Expected:** Validation fails, required field
**Component:** Backend

---

### BC-STR-02: Tracking Number = 1 char
**Input:** Minimum non-empty
**Expected:** Accept if format valid
**Component:** Backend

---

### BC-STR-03: Tracking Number = 50 chars (Max)
**Input:** Maximum allowed length
**Expected:** Accept
**Component:** Backend

---

### BC-STR-04: Tracking Number = 51 chars
**Input:** Just over max
**Expected:** Reject, too long
**Component:** Backend

---

### BC-STR-05: Recipient Name = "" (Empty)
**Input:** Empty string
**Expected:** Accept (optional field) or reject
**Component:** Backend

---

### BC-STR-06: Recipient Name = 100 chars
**Input:** Very long name
**Expected:** Accept (names can be long)
**Component:** Backend

---

### BC-STR-07: Recipient Name = 256 chars (Max)
**Input:** Maximum database field
**Expected:** Accept
**Component:** Backend

---

### BC-STR-08: Address = 1 char
**Input:** Minimal address
**Expected:** Accept but flag for review
**Component:** Backend

---

### BC-STR-09: Address = 500 chars (Max)
**Input:** Very detailed address
**Expected:** Accept
**Component:** Backend

---

### BC-STR-10: Address = 501 chars
**Input:** Just over max
**Expected:** Truncate or reject
**Component:** Backend

---

### BC-STR-11: Package Description = ""
**Input:** Empty description
**Expected:** Accept (optional field)
**Component:** Backend

---

### BC-STR-12: Package Description = 1000 chars
**Input:** Maximum allowed
**Expected:** Accept
**Component:** Backend

---

### BC-STR-13: Rider Note = 0 chars
**Input:** No notes
**Expected:** Accept, null handling
**Component:** Mobile App

---

### BC-STR-14: Rider Note = 500 chars
**Input:** Maximum note length
**Expected:** Accept
**Component:** Mobile App

---

### BC-STR-15: OTP Display = "000000"
**Input:** Leading zeros
**Expected:** Display all 6 digits
**Component:** Web/Mobile

---

## 📊 Collection Size Boundaries (BC-COLL)

### BC-COLL-01: Active Deliveries = 0
**Input:** No deliveries assigned
**Expected:** Show "No deliveries" message
**Component:** Mobile App

---

### BC-COLL-02: Active Deliveries = 1
**Input:** Single delivery
**Expected:** Show delivery details
**Component:** Mobile App

---

### BC-COLL-03: Active Deliveries = 10 (Max)
**Input:** Maximum concurrent deliveries
**Expected:** All displayed, scrollable list
**Component:** Mobile App

---

### BC-COLL-04: Active Deliveries = 11
**Input:** Over maximum
**Expected:** Reject new assignment
**Component:** Backend

---

### BC-COLL-05: Delivery History = 0
**Input:** New rider
**Expected:** Show "No history yet" message
**Component:** Mobile App

---

### BC-COLL-06: Delivery History = 10,000+
**Input:** Very active rider
**Expected:** Paginate results (50/page)
**Component:** Mobile App

---

### BC-COLL-07: Paired Boxes = 0
**Input:** No devices
**Expected:** Show "Add device" prompt
**Component:** Mobile App

---

### BC-COLL-08: Paired Boxes = 1
**Input:** Single box
**Expected:** Auto-select for deliveries
**Component:** Mobile App

---

### BC-COLL-09: Paired Boxes = 5 (Max)
**Input:** Maximum devices
**Expected:** All listed, select for delivery
**Component:** Mobile App

---

### BC-COLL-10: Pending Notifications = 0
**Input:** Nothing queued
**Expected:** Notification processor idle
**Component:** Backend

---

### BC-COLL-11: Pending Notifications = 1000+
**Input:** Burst of events
**Expected:** Rate limit, process in batches
**Component:** Backend

---

### BC-COLL-12: Search Results = 0
**Input:** No matching deliveries
**Expected:** Show "No results found"
**Component:** Admin Portal

---

### BC-COLL-13: Search Results = 500+
**Input:** Many matches
**Expected:** Paginate, show count
**Component:** Admin Portal

---

### BC-COLL-14: Firebase Listeners = 0
**Input:** Nothing subscribed
**Expected:** No realtime updates
**Component:** Web Portal

---

### BC-COLL-15: Firebase Listeners = 100
**Input:** Many open subscriptions
**Expected:** Manage memory, close unused
**Component:** Web Portal

---

## 📏 File Size Boundaries (BC-FILE)

### BC-FILE-01: Photo Size = 1 KB
**Input:** Minimum reasonable photo
**Expected:** Accept upload
**Component:** Box (ESP32)

---

### BC-FILE-02: Photo Size = 100 KB (Target)
**Input:** Optimal compressed size
**Expected:** Fast upload
**Component:** Box (ESP32)

---

### BC-FILE-03: Photo Size = 500 KB
**Input:** High quality image
**Expected:** Accept, may be slow
**Component:** Box (ESP32)

---

### BC-FILE-04: Photo Size = 1 MB (Max)
**Input:** Maximum allowed
**Expected:** Accept with warning
**Component:** Box (ESP32)

---

### BC-FILE-05: Photo Size = 1.1 MB
**Input:** Over maximum
**Expected:** Compress before upload or reject
**Component:** Box (ESP32)

---

### BC-FILE-06: SPIFFS Usage = 0%
**Input:** Empty filesystem
**Expected:** Normal operation
**Component:** Box (ESP32)

---

### BC-FILE-07: SPIFFS Usage = 79%
**Input:** Just below warning
**Expected:** Normal operation
**Component:** Box (ESP32)

---

### BC-FILE-08: SPIFFS Usage = 80%
**Input:** Warning threshold
**Expected:** Alert admin, cleanup old data
**Component:** Box (ESP32)

---

### BC-FILE-09: SPIFFS Usage = 99%
**Input:** Nearly full
**Expected:** Critical alert, aggressive cleanup
**Component:** Box (ESP32)

---

### BC-FILE-10: SPIFFS Usage = 100%
**Input:** Completely full
**Expected:** Delete oldest, write new
**Component:** Box (ESP32)

---

### BC-FILE-11: Firmware Binary = 1 MB (Min)
**Input:** Small firmware
**Expected:** OTA succeeds quickly
**Component:** Box (ESP32)

---

### BC-FILE-12: Firmware Binary = 4 MB (Max)
**Input:** Maximum partition size
**Expected:** OTA succeeds (slower)
**Component:** Box (ESP32)

---

### BC-FILE-13: Firmware Binary = 4.1 MB
**Input:** Over partition limit
**Expected:** OTA rejected, error logged
**Component:** Box (ESP32)

---

### BC-FILE-14: Document Upload = 0 bytes
**Input:** Empty file
**Expected:** Reject with error message
**Component:** Admin Portal

---

### BC-FILE-15: Document Upload = 25 MB (Max)
**Input:** Maximum allowed
**Expected:** Accept with progress bar
**Component:** Admin Portal

---

## Summary

| Category | Count |
|----------|-------|
| 🔢 Numeric Boundaries | 15 |
| 📍 Geographic Boundaries | 15 |
| ⏱️ Time Boundaries | 15 |
| 📝 String Length Boundaries | 15 |
| 📊 Collection Size Boundaries | 15 |
| 📏 File Size Boundaries | 15 |
| **Total** | **90** |

---

## Testing Matrix

| Boundary Type | Min | Min+1 | Normal | Max-1 | Max | Max+1 |
|---------------|-----|-------|--------|-------|-----|-------|
| OTP Value | 000000 | 000001 | 456789 | 999998 | 999999 | N/A |
| Latitude | -90.0 | -89.999 | 14.5995 | 89.999 | 90.0 | 90.001 |
| Battery % | 0 | 1 | 50 | 99 | 100 | N/A |
| String Length | 0 | 1 | typical | max-1 | max | max+1 |
| Queue Size | 0 | 1 | 5 | 9 | 10 | 11 |

---

## Validation Strategy

1. **Input Validation**: All boundaries tested at API level
2. **Database Constraints**: CHECK constraints enforce limits
3. **UI Validation**: Client prevents out-of-range input
4. **Firmware Bounds**: C++ templates for compile-time checks
