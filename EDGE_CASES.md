# Edge Cases & Failure Scenarios

Complete list of edge cases that must be bulletproofed for a production-ready delivery system.

---

## 🔴 Critical (Must Handle)

### EC-01: No Signal at Delivery Location
**Scenario:** Rider arrives at basement/elevator/rural area with no cellular/WiFi.

| Component | Problem | Solution | Status | Verification |
|-----------|---------|----------|--------|--------------|
| Box | Can't validate OTP online | OTP pre-cached before trip | ✅ Done | **Simulation** (Mock Offline) |
| Box | Photo can't upload | Queue to SPIFFS, retry later | ✅ Done | **Unit Test** (Queue Logic) |
| Phone | Location update fails | Cache locally, sync later | ✅ Done | **Simulation** (Airplane Mode) |
| Web | No live updates | Show "Last seen X ago" | ✅ Done | **Manual** (Visual Check) |

---

### EC-02: Box Never Received Delivery Assignment
**Scenario:** Delivery assigned while box was already offline (underground parking).

| Solution | Implementation | Verification |
|----------|----------------|--------------|
| Primary | Phone sends OTP via Bluetooth to box | **Feature Test** (BLEPairing) |
| Backup | Rider calls support for manual override | **Manual** (Dry Run) |

**Status:** ✅ Done - BLE implementation added
- ESP32 firmware: BLE server with OTP characteristic (`main.cpp`)
- Mobile app: `bleOtpService.ts` for scanning and OTP transfer
- Protocol: `OTP:123456:delivery_id:timestamp` format

---

### EC-03: Box Battery Dies Mid-Delivery
**Scenario:** Box loses power before customer can pick up.

| Solution | Implementation | Verification |
|----------|----------------|--------------|
| Prevention | Low battery warning at 20%, alert at 10% | **Unit Test** (Threshold Logic) |
| Recovery | OTP remains valid; box unlocks when power restored | **Hardware Test** (Pull Plug) |
| Fallback | Support can mark delivery complete manually | **Manual** (Admin Panel) |

**Status:** ✅ Done - Prevention(battery UI), Recovery(Firebase sync), Fallback(admin override)

---

### EC-04: Customer Enters Wrong OTP 5 Times
**Scenario:** Potential unauthorized access attempt.

| Solution | Implementation | Verification |
|----------|----------------|--------------|
| Lockout | 5 min lockout after 5 failures | **Unit Test** (Counter Logic) |
| Alert | Push notification to rider + admin | **Integration** (Firebase Func) |
| Photo | Capture failed attempts for audit | **Hardware Test** (Trigger Cam) |
| Override | Admin can reset lockout remotely | **Manual** (Admin Panel) |

**Status:** ✅ Done - OTP lockout implemented
- ESP32 firmware: `incrementOtpAttempt()`, `checkOtpLockout()`, `reportLockoutToFirebase()`
- Firebase: `LockoutState` interface, `subscribeToLockout()`, `resetLockout()`
- Tests: `test_ec04_*` in `test_edge_cases.h`, `otpEdgeCases.test.ts`
- Debounce: 1s cooldown between attempts to prevent brute force

---

### EC-05: Rider's Phone Dies
**Scenario:** Only GPS source lost; box still has signal.

| Impact | Severity | Verification |
|--------|----------|--------------|
| GPS tracking | ✅ Box continues normally | **Manual** (Turn off Phone) |
| OTP display | ⚠️ Customer can't see OTP | **Manual** (Check Web Link) |
| Photo proof | ✅ Box captures independently | **Hardware Test** |

**Solution:** OTP is in tracking link (web) - customer can access from any device.

**Status:** ✅ Already handled

---

### EC-06: Both Box AND Phone Offline
**Scenario:** Complete communication blackout.

| What Still Works | Why | Verification |
|------------------|-----|--------------|
| OTP Validation | Pre-cached locally on box | **Hardware Test** (Faraday Cage/Sim) |
| Box Unlock | Local comparison, no network | **Hardware Test** |
| Photo Capture | Saved to SPIFFS | **Unit Test** (Storage Logic) |
| Solenoid Control | Hardware-level operation | **Hardware Test** |

| What Fails | Mitigation |
|------------|------------|
| Live tracking | Customer sees "Last seen" timestamp |
| Status updates | Sync when either reconnects |
| Photo upload | Auto-retry queue (up to 5 attempts) |

**Status:** ✅ Handled by offline implementation

---

## 🟡 High Priority

### EC-07: Stale OTP (Delivery Cancelled/Reassigned)
**Scenario:** Delivery cancelled but box still has old OTP cached.

| Solution | Implementation | Verification |
|----------|----------------|--------------|
| Expiry | OTP valid for 4 hours max | **Unit Test** (Time Logic) |
| Revocation | Box polls Firebase; clears OTP on cancellation | **Integration** (Sync Test) |
| Offline | If offline when cancelled, OTP works until box reconnects | **Edge Case Acceptance** |

**Status:** ✅ Done - OTP expiry & revocation implemented
- ESP32 firmware: `isOtpValid()`, `clearExpiredOtp()`, `otpIssuedAt` tracking
- Firebase: `OtpStatus` interface, `subscribeToOtpStatus()`, `revokeOtp()`, `regenerateOtp()`
- Constants: `OTP_VALIDITY_DURATION = 4 hours` (14400000ms)
- Tests: `test_ec07_*` in `test_edge_cases.h`, `otpEdgeCases.test.ts`
- Revocation: `otp_revoked` flag in Firebase triggers immediate invalidation

---

### EC-08: GPS Spoofing Attack
**Scenario:** Malicious rider fakes location to skip deliveries.

| Detection | Implementation | Verification |
|-----------|----------------|--------------|
| Velocity check | Flag if speed > 200 km/h | **Unit Test** (Math Logic) |
| Distance jump | Flag if position jumps > 10km in 1 second | **Unit Test** (Math Logic) |
| Source mismatch | Compare phone vs box GPS for major discrepancies | **Integration** (Data Analysis) |

**Status:** ⬜ TODO - Add anomaly detection

---

### EC-09: Firebase Outage
**Scenario:** Google Cloud has an outage.

| Component | Fallback | Verification |
|-----------|----------|--------------|
| Box | Queue all data locally; retry | **Unit Test** (Queue Rejection) |
| Phone | Use offline cache | **Simulation** (Block Traffic) |
| Web | Show cached data with "Updating..." banner | **Simulation** (Offline Mode) |

**Status:** ⬜ Partial - Need web offline mode

---

### EC-10: Photo Storage Full
**Scenario:** SPIFFS (4MB) filled with unuploaded photos.

| Solution | Implementation |
|----------|----------------|
| Limit | Max 10 queued photos (configurable) |
| Oldest-first delete | If queue full, delete oldest failed upload |
| Alert | Warn admin if queue reaches 80% |

**Status:** ✅ MAX_QUEUED_PHOTOS = 10

---

## 🟢 Medium Priority

### EC-11: Customer Not Home
**Scenario:** Rider arrives but nobody to accept delivery.

| Solution | Implementation |
|----------|----------------|
| Wait timer | 5 min countdown before return option |
| Photo proof | Capture photo showing arrival |
| Notification | Push to customer "Driver is waiting" |
| Reschedule | Web form to pick new time |

**Status:** ✅ Done - Full implementation
- Mobile: `customerNotHomeService.ts` - Wait timer state machine, notification logic
- Web: `customerNotHome.ts` - Reschedule form, customer notifications
- Firebase: Real-time wait timer sync, reschedule request storage
- Tests: `ec11CustomerNotHome.test.ts` - Timer expiry, notifications, reschedule validation
- Features:
  - 5-minute wait timer with formatted countdown (MM:SS)
  - Arrival photo capture with Firebase Storage
  - Push notification to customer on arrival
  - Reschedule with date picker (1-7 days ahead) and time slots
  - Return option available after timer expires

---

### EC-12: Wrong Address
**Scenario:** GPS shows rider at location, but it's incorrect address.

| Solution | Implementation |
|----------|----------------|
| Rider correction | App allows address update |
| Customer correction | Tracking page has "Update Address" |
| Geofence flexibility | 50m radius accommodates GPS errors |

**Status:** ✅ Done - Full implementation
- Mobile: `addressUpdateService.ts` - Geofence calculations, address validation
- Web: `addressUpdate.ts` - Customer address correction form
- Tests: `ec12AddressUpdate.test.ts` - Distance calc, geofence, validation
- Features:
  - Haversine distance calculation for accuracy
  - Dynamic geofence expansion (50m default, 200m max)
  - GPS accuracy compensation in boundary checks
  - Address update with 1km maximum correction distance
  - 60-second cooldown between updates
  - Validation for coordinates, address length, reason required
  - Support for RIDER, CUSTOMER, ADMIN address sources

---

### EC-13: Multiple Deliveries to Same Location
**Scenario:** Rider has 3 deliveries to same building.

| Solution | Implementation |
|----------|----------------|
| Batch mode | Box can store multiple OTPs |
| Clear UI | Rider sees list of pending deliveries |

**Status:** ⬜ TODO - Expand OTP storage

---

### EC-14: Time Zone Confusion
**Scenario:** Server, box, and phone in different timezones.

| Solution | Implementation |
|----------|----------------|
| UTC everywhere | All timestamps in UTC |
| Local display | Convert to local only in UI |
| Firebase server time | Use .sv timestamp for accuracy |

**Status:** ✅ Using server timestamps

---

### EC-15: App Killed by OS
**Scenario:** Android/iOS kills background app; phone GPS stops.

| Solution | Implementation |
|----------|----------------|
| Foreground service | Android notification keeps app alive |
| Location background | iOS background location permission |
| Failover | Box GPS continues independently |

**Status:** ✅ Done - Background location service implemented
- Mobile: `backgroundLocationService.ts` - Expo TaskManager & Location APIs
- Tests: `ec15BackgroundLocation.test.ts` - Service lifecycle, failover logic
- Features:
  - Android foreground service with persistent notification
  - iOS background location mode with significant change monitoring
  - Expo TaskManager for cross-platform background task
  - Automatic service restart on OS termination (health check)
  - 2-minute staleness threshold for location updates
  - Failover detection: alerts when both phone AND box GPS unavailable
  - Power-efficient: balanced accuracy, 10m distance filter
  - App state monitoring for foreground updates

---

## Implementation Priority Matrix

| Priority | Edge Case | Effort | Risk if Unhandled |
|----------|-----------|--------|-------------------|
| P0 | EC-01 (No Signal) | Done | Delivery fails |
| P0 | EC-06 (Both Offline) | Done | Complete failure |
| P1 | EC-04 (Wrong OTP) | Medium | Security breach |
| P1 | EC-07 (Stale OTP) | Low | Wrong person opens |
| P1 | EC-03 (Battery Dies) | Medium | Stuck delivery |
| P2 | EC-02 (Missed Assignment) | High (BLE) | Rare scenario |
| P2 | EC-08 (GPS Spoof) | High | Fraud |
| P3 | Others | Varies | Inconvenience |

---

## Testing Checklist

- [ ] Airplane mode during delivery (EC-01)
- [ ] Cancel delivery while box offline (EC-07)
- [ ] 6 wrong OTP attempts (EC-04)
- [ ] Fill photo queue to limit (EC-10)
- [ ] Simulate box power loss (EC-03)
- [x] Kill app during active tracking (EC-15) ✅ Unit tests pass
- [x] Customer not home wait timer (EC-11) ✅ Unit tests pass
- [x] Wrong address geofence expansion (EC-12) ✅ Unit tests pass
- [x] Solenoid stuck closed retry (EC-21) ✅ Unit tests pass
- [x] Solenoid stuck open detection (EC-22) ✅ Unit tests pass
- [x] Camera failure with retry (EC-23) ✅ Unit tests pass
- [x] ESP32 brownout recovery (EC-25) ✅ Unit tests pass
- [ ] Network switch during upload (EC-61)
- [ ] Captive portal WiFi detection (EC-62)
- [ ] Two riders at same location (EC-65)
- [ ] Shift handover with active delivery (EC-67)
- [ ] Admin override during OTP entry (EC-77)
- [ ] Firmware update during delivery (EC-80)
- [ ] Year-end transition test (EC-74)
- [ ] Battery degradation simulation (EC-72)

---

## 🔵 Security & Attack Vectors

### EC-16: Replay Attack on OTP
**Scenario:** Attacker captures OTP transmission, replays later.

| Solution | Implementation |
|----------|----------------|
| One-time use | OTP invalidated after successful unlock |
| Time-bound | OTP only valid during assigned delivery window |
| Hash verification | OTP hashed with timestamp, can't replay |

**Status:** ⬜ TODO

---

### EC-17: Man-in-the-Middle on Location
**Scenario:** Attacker intercepts Firebase updates, sends fake location.

| Solution | Implementation |
|----------|----------------|
| TLS only | All Firebase uses HTTPS |
| Auth tokens | Only authenticated devices can write |
| Source verification | Cross-check box vs phone GPS |

**Status:** ✅ Firebase uses TLS

---

### EC-18: Physical Tampering - Box Pried Open
**Scenario:** Someone forces the box open without OTP.

| Solution | Implementation |
|----------|----------------|
| Tamper switch | Magnetic reed switch on door (Pin 27) |
| Alert | Immediate push (Mobile) + Banner (Web) |
| Photo | Auto-capture on interrupt |
| Lockdown | Disable all further unlocks until reset |

**Status:** ✅ Done - Hardware logic added, Mobile/Web alerts active

---

### EC-19: Stolen Phone with Rider App
**Scenario:** Thief steals rider's phone, has access to delivery app.

| Solution | Implementation |
|----------|----------------|
| Biometric lock | Require FaceID/Fingerprint for sensitive actions |
| Session timeout | Auto-logout after 30 min inactive |
| Remote wipe | Admin can revoke device access |
| PIN for OTP reveal | Secondary PIN to view customer OTP |

**Status:** ⬜ TODO

---

### EC-20: Delivery ID Collision
**Scenario:** Two deliveries accidentally get same OTP.

| Solution | Implementation |
|----------|----------------|
| Unique generation | OTP tied to delivery_id + box_id + timestamp |
| Collision check | Backend rejects duplicate OTPs |

**Status:** ⬜ Check backend logic

---

## 🟤 Hardware Failure Modes

### EC-21: Solenoid Stuck Closed
**Scenario:** Mechanical failure - lock won't open.

| Solution | Implementation |
|----------|----------------|
| Manual override | Physical key slot (emergency) |
| Power cycle | Retry solenoid 3x with delays |
| Alert | Ping support automatically |

**Status:** ✅ Done - Full implementation
- ESP32 firmware: `LockControl.h` - Retry logic with configurable attempts
- `SOLENOID_MAX_RETRIES = 3`, `SOLENOID_RETRY_DELAY_MS = 500`
- Feedback sensor support for state verification
- Firebase reporting: `hardware/{boxId}/solenoid` with status, retry_count, severity
- Tests: `test_ec21_*` in `test_edge_cases.h`, `hardwareFailures.test.ts`
- Features:
  - 3 automatic retry attempts with 500ms delays
  - Lock feedback sensor verification after each attempt
  - Safety timeout (5s max energized) to prevent coil damage
  - Detailed Firebase alerts with severity: HIGH
  - Audit trail logging

---

### EC-22: Solenoid Stuck Open
**Scenario:** Lock won't engage - box stays unlocked.

| Solution | Implementation |
|----------|----------------|
| Detection | Check lock feedback sensor |
| Alert | Immediate notification |
| Disable deliveries | Mark box as "Out of Service" |

**Status:** ✅ Done - Full implementation
- ESP32 firmware: `LockControl.h` - `markOutOfService()`, `isOutOfService()`
- Feedback sensor detection on lock() attempt
- Firebase reporting: `hardware/{boxId}/solenoid` with severity: CRITICAL
- Web: `subscribeToSolenoid()` for real-time monitoring
- Tests: `test_ec22_*` in `test_edge_cases.h`, `hardwareFailures.test.ts`
- Features:
  - Lock state verification via feedback sensor (Pin 32)
  - Automatic out-of-service marking when lock fails
  - Blocks all unlock attempts when out of service
  - Immediate Firebase notification with "unsecured" warning
  - `shouldBlockDeliveries()` helper for UI decisions

---

### EC-23: Camera Failure
**Scenario:** Photo capture fails - no proof of delivery.

| Solution | Implementation |
|----------|----------------|
| Retry | 3 capture attempts |
| Fallback | Allow delivery without photo (flag for review) |
| Placeholder | Save metadata even if image fails |
| Alert | Notify admin of camera issues |

**Status:** ✅ Done - Full implementation
- ESP32 firmware: `PhotoCapture.h` - Complete retry and fallback system
- `CAMERA_MAX_RETRIES = 3`, `CAMERA_RETRY_DELAY_MS = 500`
- Firebase reporting: `hardware/{boxId}/camera` with status, attempts, failure_reason
- Web: `subscribeToCamera()`, `PhotoMetadata` interface
- Tests: `test_ec23_*` in `test_edge_cases.h`, `hardwareFailures.test.ts`
- Features:
  - 3 retry attempts with 500ms delays
  - `RETRY_SUCCESS` status when succeeds after retry
  - Metadata always saved for audit (even on failure)
  - `flagged_for_review` flag when delivery proceeds without photo
  - Hardware error detection after 3 consecutive complete failures
  - Delivery proceeds without photo (compliance with customer expectation)
  - `canProceedWithoutPhoto()` helper returns true (with flag)

---

### EC-24: GPS Module Failure
**Scenario:** Box GPS returns invalid/frozen coordinates.

| Detection | Implementation |
|-----------|----------------|
| Stale data | Same coords for >5 min while moving |
| Invalid range | Lat/lng outside valid bounds |
| Phone fallback | Auto-switch to phone GPS |

**Status:** ✅ Redundancy system handles this

---

### EC-25: ESP32 Brownout/Reboot Mid-Delivery
**Scenario:** Power dip causes ESP32 to restart.

| Solution | Implementation |
|----------|----------------|
| Persistent state | Save delivery state to SPIFFS |
| Auto-resume | Load state on boot, continue delivery |
| Status report | Send "box_rebooted" event to Firebase |

**Status:** ✅ Done - Full implementation
- ESP32 firmware: `DeliveryState.h` - Complete SPIFFS persistence
- State file: `/delivery_state.json` with full delivery context
- Boot counter: `/boot_count.txt` for tracking reboots
- Firebase reporting: `hardware/{boxId}/reboot` with restored_state
- Web: `subscribeToReboot()`, `clearRebootFlag()`, `RebootState` interface
- Tests: `test_ec25_*` in `test_edge_cases.h`, `hardwareFailures.test.ts`
- Features:
  - Auto-save on delivery assignment, arrival, unlock
  - Periodic auto-save (30s interval) during active delivery
  - Full state restoration: delivery_id, OTP, target, arrived, unlocked, power_state
  - `wasRebootedDuringDelivery` flag for detection
  - Boot count tracking for maintenance insights
  - `shouldResumeDelivery()` helper for UI decisions
  - Audit trail with restored_state details

---

## 🌡️ Environmental Factors

### EC-26: Extreme Heat (>60°C)
**Scenario:** Box in direct sun in Philippines summer.

| Solution | Implementation |
|----------|----------------|
| Temp sensor | Monitor internal temperature |
| Throttle | Reduce update frequency to save power |
| Alert | Warn if electronics at risk |

**Status:** ⬜ TODO

---

### EC-27: Water Ingress / Heavy Rain
**Scenario:** Water damages electronics during monsoon.

| Prevention | Implementation |
|------------|----------------|
| IP65 rating | Sealed enclosure |
| Moisture sensor | Detect water ingress early |
| Corrosion check | Periodic health check |

**Status:** ⬜ Hardware design consideration

---

### EC-28: Vibration Damage (Motorcycle)
**Scenario:** Constant vibration loosens connections.

| Prevention | Implementation |
|------------|----------------|
| Secure mounts | Lock-tite on screws |
| Connector strain relief | Proper cable management |
| Vibration dampening | Rubber mounts for boards |

**Status:** ⬜ Hardware design

---

## 👥 Multi-Party Edge Cases

### EC-29: Customer Shares OTP Publicly
**Scenario:** Customer posts OTP on social media accidentally.

| Solution | Implementation |
|----------|----------------|
| OTP regeneration | Allow rider/customer to request new OTP |
| Time limit | OTP expires in 4 hours |
| Delivery cancellation | Customer can cancel if compromised |

**Status:** ⬜ TODO - OTP refresh API

---

### EC-30: Rider Delivers to Wrong Person
**Scenario:** Wrong person receives OTP, opens box.

| Mitigation | Implementation |
|------------|----------------|
| Photo proof | Customer sees who unlocked |
| Time/location logging | Full audit trail |
| Recipient confirmation | Customer marks "I received it" |

**Status:** ⬜ Partial - photo exists

---

### EC-31: Disputed Delivery - "I Never Received It"
**Scenario:** Customer claims non-delivery despite photo.

| Evidence Chain | Implementation |
|----------------|----------------|
| GPS at unlock | Prove box was at location |
| Timestamp | Exact time of unlock |
| Photo | Face/hands of person who unlocked |
| OTP entry log | Prove correct code was entered |

**Status:** ✅ Photo + GPS + timestamp logged

---

### EC-32: Rider Cancels After Pickup
**Scenario:** Rider picks up package, then cancels delivery.

| Solution | Implementation |
|----------|----------------|
| Lock box | OTP changes on status change |
| Alert | Notify sender of cancellation |
| Return flow | Generate "return OTP" for sender |

**Status:** ⬜ TODO - Cancellation flow

---

## ⚡ Race Conditions & Timing

### EC-33: Two People Enter OTP Simultaneously
**Scenario:** Customer types OTP, rider also types (testing).

| Solution | Implementation |
|----------|----------------|
| Debounce | 500ms lockout between attempts |
| Single unlock | Only triggers once per OTP |

**Status:** ⬜ TODO

---

### EC-34: OTP Entered While Box Updating
**Scenario:** Box downloading new OTP from Firebase, old one entered.

| Solution | Implementation |
|----------|----------------|
| Accept both | During update window, accept old OR new |
| Atomic switch | Lock keypad during OTP refresh |

**Status:** ⬜ TODO

---

### EC-35: Delivery Status Update Lost
**Scenario:** Box unlocks, but status update to "COMPLETED" fails.

| Solution | Implementation |
|----------|----------------|
| Retry queue | Queue status updates like photos |
| Reconciliation | Backend marks complete on photo receipt |
| Fallback | Rider can manually mark complete in app |

**Status:** ⬜ TODO

---

## 📱 Mobile App Specific

### EC-36: Multiple Riders Logged Into Same Account
**Scenario:** Rider shares credentials, two phones active.

| Solution | Implementation |
|----------|----------------|
| Single device | Force logout on new login |
| Device binding | Lock account to one device |

**Status:** ⬜ TODO

---

### EC-37: App Update Required Mid-Delivery
**Scenario:** Force update blocks screen during active delivery.

| Solution | Implementation |
|----------|----------------|
| Grace period | Allow completing current delivery |
| Cache delivery | Show even without latest version |

**Status:** ⬜ TODO

---

### EC-38: Push Notification Not Received
**Scenario:** Customer doesn't know rider arrived.

| Fallback | Implementation |
|----------|----------------|
| SMS | Send SMS as backup |
| Email | Send email notification |
| Retry | 3 push attempts |

**Status:** ⬜ TODO

## 💰 Financial & Payment Edge Cases

### EC-39: Payment Declined After Pickup
**Scenario:** COD payment declined, but package already in box.

| Solution | Implementation |
|----------|----------------|
| Pre-auth | Verify payment before pickup |
| Return flow | Generate return OTP for rider |
| Hold status | Lock box until payment resolved |

**Status:** ⬜ TODO

---

### EC-40: Refund Requested After Delivery
**Scenario:** Customer wants refund but package already delivered.

| Solution | Implementation |
|----------|----------------|
| Evidence | Show photo + GPS + OTP log |
| Return pickup | Schedule return delivery |
| Dispute flow | Admin panel for resolution |

**Status:** ⬜ TODO

---

### EC-41: Rider Pocketing COD Payment
**Scenario:** Rider collects cash but doesn't report it.

| Solution | Implementation |
|----------|----------------|
| Digital only | Prefer GCash/PayMaya |
| Amount confirmation | Customer confirms amount in app |
| Random audits | Admin spotchecks |

**Status:** ⬜ TODO - Audit system

---

## ⚖️ Legal & Regulatory

### EC-42: GDPR/Data Privacy - Location History
**Scenario:** Customer requests deletion of all their data.

| Compliance | Implementation |
|------------|----------------|
| Data export | Download all delivery history |
| Right to delete | Anonymize, don't hard delete |
| Retention policy | Auto-purge after 1 year |

**Status:** ⬜ TODO

---

### EC-43: Photo Contains PII
**Scenario:** Photo accidentally captures license plate, face of bystander.

| Solution | Implementation |
|----------|----------------|
| Blur detection | Auto-blur faces except recipient |
| Access control | Photo only visible to parties |
| Time-limited | Auto-delete after 30 days |

**Status:** ⬜ TODO

---

### EC-44: Illegal Content in Package
**Scenario:** Package contains contraband.

| Prevention | Implementation |
|------------|----------------|
| No liability clause | ToS coverage |
| Rider doesn't verify contents | Not liable |
| Cooperation with authorities | Clear process |

**Status:** ⬜ Legal/ToS

---

### EC-45: Insurance Claim for Lost Package
**Scenario:** Package lost, customer claims insurance.

| Evidence Required | Implementation |
|-------------------|----------------|
| Pickup photo | Timestamp + GPS |
| Transit history | All location updates |
| Delivery attempt log | What happened |

**Status:** ✅ Partial - audit trail exists

---

## 📊 Data Integrity

### EC-46: Firebase Clock Skew
**Scenario:** Server timestamp differs from box/phone by minutes.

| Solution | Implementation |
|----------|----------------|
| Server time only | Use Firebase .sv timestamp |
| NTP sync | Box syncs on boot |
| Tolerance | Accept ±5 min variance |

**Status:** ✅ Using server timestamps

---

### EC-47: Duplicate Delivery Records
**Scenario:** Same delivery inserted twice due to retry.

| Solution | Implementation |
|----------|----------------|
| Idempotency key | `delivery_id:otp_code:issued_at` combined key |
| Upsert | `setDeliveryWithIdempotency()` updates if exists |
| Deduplication | `checkForDuplicate()` returns NEW/SAME/UPDATE/REJECTED |

**Implementation:**
- Hardware: `DeliveryState.h` - `DuplicateCheckResult` enum, idempotency key tracking
- Mobile: `firebaseClient.ts` - `generateIdempotencyKey()`, `assignDeliveryWithIdempotency()`
- Web: `firebaseClient.ts` - Admin monitoring with `subscribeToDuplicateEvents()`

**Files Modified:**
- `hardware/lib/DeliveryState/DeliveryState.h` - Added EC-47 idempotency methods
- `hardware/src/main.cpp` - Uses `setDeliveryWithIdempotency()` for OTP assignment
- `mobile/src/services/firebaseClient.ts` - Added idempotency interfaces
- `web/src/lib/firebaseClient.ts` - Added admin duplicate monitoring

**Test Coverage:**
- `hardware/test/test_data_integrity.h` - 11 EC-47 unit tests
- `mobile/src/__tests__/DataIntegrity.test.ts` - Idempotency scenario tests
- `web/src/lib/__tests__/dataIntegrity.test.ts` - Admin dashboard tests

**Status:** ✅ Implemented

---

### EC-48: Data Corruption in SPIFFS
**Scenario:** Flash memory corrupted, queue data lost.

| Solution | Implementation |
|----------|----------------|
| Checksums | CRC32 validation via `DataIntegrity.h` |
| Backup | `RTC_DATA_ATTR RtcBackupData` survives soft reset |
| Recovery | `loadWithIntegrity()` → RTC → Firebase cascade |

**Implementation:**
- Hardware: `DataIntegrity.h` - CRC32 calculation, RTC backup, JSON checksum wrapper
- Hardware: `PhotoQueue.cpp` - `loadQueueStateWithIntegrity()`, `recoverFromCorruption()`
- Hardware: `DeliveryState.h` - `validateIntegrity()`, `applyFirebaseRecovery()`
- Mobile/Web: `firebaseClient.ts` - `DataIntegrityState`, recovery status monitoring

**Recovery Flow:**
1. `loadJsonWithChecksum()` validates CRC32
2. On failure → `validateRtcBackup()` attempts RTC recovery
3. On RTC failure → `needsFirebaseRecovery` flag triggers Firebase re-fetch
4. `reportIntegrityStatusToFirebase()` logs corruption events

**Files Modified:**
- `hardware/lib/DataIntegrity/DataIntegrity.h` - NEW: CRC32, RTC backup, recovery
- `hardware/lib/PhotoQueue/PhotoQueue.cpp` - Added checksum validation
- `hardware/lib/DeliveryState/DeliveryState.h` - Added integrity validation
- `hardware/src/main.cpp` - RTC_DATA_ATTR backup, integrity checks
- `mobile/src/services/firebaseClient.ts` - Added integrity interfaces
- `web/src/lib/firebaseClient.ts` - Added corruption alert monitoring

**Test Coverage:**
- `hardware/test/test_data_integrity.h` - 16 EC-48 unit tests (CRC32, RTC, recovery)
- `mobile/src/__tests__/DataIntegrity.test.ts` - Integrity state tests
- `web/src/lib/__tests__/dataIntegrity.test.ts` - Admin severity assessment tests

**Status:** ✅ Implemented

---

### EC-49: Out-of-Order Events
**Scenario:** "COMPLETED" received before "IN_TRANSIT".

| Solution | Implementation |
|----------|----------------|
| State machine | Only valid transitions |
| Last-write-wins | Use timestamps |
| Validation | Reject invalid sequences |

**Status:** ⬜ TODO

---

## 🎨 User Experience

### EC-50: Customer Panic - "Where's My Package?"
**Scenario:** Customer checks every 10 seconds, anxious.

| Solution | Implementation |
|----------|----------------|
| Progress indicators | Step-by-step status |
| ETA countdown | Live estimated time |
| Calm UI | No error messages unless real |

**Status:** ⬜ Partial

---

### EC-51: Language Barrier
**Scenario:** Rider speaks Tagalog, customer speaks Mandarin.

| Solution | Implementation |
|----------|----------------|
| Multi-language | App + web in multiple languages |
| Pre-written messages | "I'm at the door" buttons |
| Translation | Chat translate feature |

**Status:** ⬜ TODO - i18n

---

### EC-52: Accessibility - Blind Customer
**Scenario:** Customer can't read OTP on screen.

| Solution | Implementation |
|----------|----------------|
| Voice readout | TTS for OTP |
| Large font | Accessibility mode |
| Phone call | Rider calls to give OTP |

**Status:** ⬜ TODO

---

### EC-53: First-Time User Confusion
**Scenario:** Customer doesn't understand what OTP is.

| Solution | Implementation |
|----------|----------------|
| Onboarding | Explain on first delivery |
| In-context help | "What's this?" tooltips |
| SMS instruction | Include in arrival SMS |

**Status:** ⬜ TODO

---

## 📈 Scalability & Performance

### EC-54: 1000 Concurrent Deliveries
**Scenario:** Peak hour, system under load.

| Solution | Implementation |
|----------|----------------|
| Connection pooling | Firebase SDK handles |
| Rate limiting | Max 5 req/sec per box |
| Load balancing | Multiple regions |

**Status:** ⬜ Load testing needed

---

### EC-55: Firebase Quota Exceeded
**Scenario:** Hit daily read/write limits.

| Solution | Implementation |
|----------|----------------|
| Quota monitoring | Alert at 80%, critical at 95% |
| Local caching | Reduce read frequency when near limit |
| Blaze plan | Pay-as-you-go for prod |

**Status:** ✅ Done - Full implementation
- Web: `firebaseClient.ts` - Quota state types, `subscribeToQuotaState()`, alert level calculation
- Web: `QuotaAlertBanner.tsx` - Admin UI component with expandable details
- Mobile: `quotaMonitorService.ts` - Local caching, quota tracking, graceful degradation
- Tests: `quotaMonitoring.test.ts` (Web), `QuotaMonitoring.test.ts` (Mobile)
- Features:
  - Alert thresholds: WARNING at 80%, CRITICAL at 95%, EXCEEDED at 100%
  - Automatic cache TTL extension when approaching limits (5 min vs 1 min)
  - Fetch interval reduction (2x at 80%, 5x at 95%)
  - Local operation counters with daily reset
  - Bytes formatting utilities for UI display

---

### EC-56: Photo Upload Bandwidth
**Scenario:** Large photos slow down other operations.

| Solution | Implementation |
|----------|----------------|
| Compression | 800px max, 60% quality |
| Chunked upload | Resume on failure |
| Priority queue | GPS > Status > Photo |

**Status:** ✅ Done - Full implementation
- Hardware: `PhotoQueue.h/.cpp` - Compression config, priority queue, chunked uploads
- Mobile: `photoCompressionService.ts` - Client-side compression, priority management
- Web: `firebaseClient.ts` - `PhotoUploadState` interface for monitoring
- Tests: `photoUpload.test.ts` (Web)
- Features:
  - Max dimension: 800px (longest edge)
  - JPEG quality: 60%
  - Priority levels: GPS (0) > Status (1) > Photo (2)
  - Upload progress tracking with byte-level accuracy
  - Resumable uploads with chunk validation (4KB chunks)
  - Bandwidth estimation with exponential moving average
  - Compression statistics (ratio, bytes saved)

---

## 🔄 Lifecycle & Maintenance

### EC-57: Firmware Update Required
**Scenario:** Security patch must be deployed to all boxes.

| Solution | Implementation |
|----------|----------------|
| OTA updates | ESP32 supports OTA |
| Staged rollout | 10% → 50% → 100% |
| Rollback | Keep previous version |

**Status:** ⬜ TODO

---

### EC-58: Box Decommissioned
**Scenario:** Box retired, data needs cleanup.

| Solution | Implementation |
|----------|----------------|
| Archive | Move to cold storage |
| Unlink | Remove from rider account |
| Wipe | Factory reset device |

**Status:** ⬜ TODO

---

### EC-59: Rider Quits Mid-Shift
**Scenario:** Rider goes offline with packages in box.

| Solution | Implementation |
|----------|----------------|
| Alert | Ping supervisor |
| Reassign | Transfer deliveries |
| Return | GPS shows box location |

**Status:** ⬜ TODO

---

### EC-60: Daylight Saving Time Change
**Scenario:** Clocks shift, scheduled deliveries off.

| Solution | Implementation |
|----------|----------------|
| UTC storage | All times in UTC |
| Timezone DB | Use proper tz library |
| User's local time | Display only |

**Status:** ✅ UTC everywhere

---

## 🌐 Network & Connectivity Edge Cases

### EC-61: Network Switch Mid-Upload (WiFi → Cellular)
**Scenario:** Photo upload starts on WiFi, phone switches to cellular mid-transfer.

| Solution | Implementation |
|----------|----------------|
| Resumable uploads | Firebase Storage resumable sessions |
| Connection monitoring | Detect network change, pause/resume |
| Chunk validation | Verify each chunk before continuing |

**Status:** ⬜ TODO

---

### EC-62: Captive Portal WiFi
**Scenario:** Box connects to hotel/airport WiFi that requires browser login.

| Solution | Implementation |
|----------|----------------|
| Detection | HTTP check to known endpoint |
| Fallback | Mark WiFi as unusable, alert rider |
| Alternative | Use phone hotspot instead |

**Status:** ⬜ TODO

---

### EC-63: IPv6-Only Network
**Scenario:** Network only supports IPv6, Firebase connectivity issues.

| Solution | Implementation |
|----------|----------------|
| Dual stack | Ensure code works with IPv4 and IPv6 |
| Firebase SDK | Use latest SDK with IPv6 support |
| Testing | Test in IPv6-only environment |

**Status:** ⬜ TODO

---

### EC-64: Slow DNS Resolution
**Scenario:** DNS lookup takes >10 seconds, causing timeouts.

| Solution | Implementation |
|----------|----------------|
| DNS caching | Cache resolved IPs locally |
| Multiple DNS | Try Google DNS (8.8.8.8) as fallback |
| Timeout handling | Longer timeout for initial connection |

**Status:** ⬜ TODO

---

## 👥 Multi-Entity Edge Cases

### EC-65: Two Riders Arrive Simultaneously
**Scenario:** Two riders arrive at same location for different deliveries.

| Solution | Implementation |
|----------|----------------|
| Distinct OTPs | Each delivery has unique OTP |
| Customer clarity | Show which rider is for which package |
| Queue display | Ordered list if same customer |

**Status:** ⬜ TODO

---

### EC-66: Customer Orders from Two Riders
**Scenario:** Same customer has two active deliveries from different riders.

| Solution | Implementation |
|----------|----------------|
| Multi-delivery view | Tracking page shows both |
| Separate OTPs | Each delivery independent |
| Combined notifications | Group notifications logically |

**Status:** ⬜ TODO

---

### EC-67: Box Shared Between Riders (Shift Handover)
**Scenario:** Rider A ends shift, Rider B takes over same box with pending deliveries.

| Solution | Implementation |
|----------|----------------|
| Box-centric OTP | OTP tied to box, not rider |
| Handover protocol | Formal transfer in app |
| Audit trail | Log shift changes |

**Status:** ⬜ TODO

---

### EC-68: Residential vs Business Address
**Scenario:** GPS matches both residential and business at same coordinates.

| Solution | Implementation |
|----------|----------------|
| Address type field | Customer specifies type (RESIDENTIAL/BUSINESS/OTHER) |
| Instructions | Prompt for building name/unit for business |
| Geofence size | 50m residential, 100m business complexes |

**Status:** ✅ Done - Full implementation
- Mobile: `addressUpdateService.ts` - `AddressType` enum, `validateBusinessAddress()`, geofence helpers
- Web: `addressUpdate.ts` - Address type validation, formatting, geofence creation
- Tests: `addressTypeValidation.test.ts` (Web), `AddressTypeGeofence.test.ts` (Mobile)
- Features:
  - Address types: RESIDENTIAL (50m), BUSINESS (100m), OTHER (50m)
  - Business addresses require building name OR unit number
  - `suggestAddressType()` heuristic based on address keywords
  - `needsBusinessDetails()` prompt helper for UI
  - `formatAddressWithDetails()` with building/floor/unit formatting
  - Dynamic geofence creation: `createGeofenceForAddressType()`
  - Maximum geofence radius capped at 200m

---

## 🔧 Hardware Lifecycle Edge Cases

### EC-69: Hardware Calibration Drift
**Scenario:** Sensors become less accurate over time.

| Solution | Implementation |
|----------|----------------|
| Periodic calibration | Monthly self-test routine |
| Drift detection | Compare against baseline |
| Alert | Flag boxes needing recalibration |

**Status:** ⬜ TODO

---

### EC-70: Flash Memory Wear Exhaustion
**Scenario:** SPIFFS write cycles approaching limit.

| Solution | Implementation |
|----------|----------------|
| Wear leveling | Built into ESP32 SPIFFS |
| Write reduction | Batch writes, reduce frequency |
| Monitoring | Track write counts, alert at 80% life |

**Status:** ⬜ TODO

---

### EC-71: Keypad Key Wear
**Scenario:** Frequently used keys (1, 2, 3) become unresponsive.

| Solution | Implementation |
|----------|----------------|
| Key health check | Test all keys in diagnostics |
| Redundant entry | Alternative input method |
| Maintenance alert | Flag worn keys |

**Status:** ⬜ TODO

---

### EC-72: Battery Degradation
**Scenario:** Battery capacity reduced >30% from original.

| Solution | Implementation |
|----------|----------------|
| Capacity tracking | Compare actual vs rated capacity |
| Performance mode | Adjust power usage for degraded battery |
| Replacement alert | Notify when replacement needed |

**Status:** ⬜ TODO

---

## 📅 Time-Based Edge Cases

### EC-73: Leap Year Date Calculations
**Scenario:** Feb 29 causes date math errors.

| Solution | Implementation |
|----------|----------------|
| Standard library | Use proven date libraries |
| Testing | Test specifically for leap years |
| Duration calc | Use day-agnostic duration math |

**Status:** ⬜ TODO

---

### EC-74: Year-End Transition
**Scenario:** Dec 31 23:59 → Jan 1 00:00 causes year rollover bugs.

| Solution | Implementation |
|----------|----------------|
| UTC timestamps | Avoid local time manipulation |
| Epoch time | Use Unix timestamps internally |
| Year-agnostic | Don't hardcode year assumptions |

**Status:** ⬜ TODO

---

### EC-75: Maintenance During Active Delivery
**Scenario:** Scheduled maintenance starts while delivery in progress.

| Solution | Implementation |
|----------|----------------|
| Grace period | Allow completion of active deliveries |
| Block new assignments | Only prevent new deliveries |
| Priority override | Critical deliveries can continue |

**Status:** ⬜ TODO

---

### EC-76: Public Holiday Operations
**Scenario:** Reduced staff/riders during holidays.

| Solution | Implementation |
|----------|----------------|
| Holiday calendar | System-wide holiday awareness |
| Capacity planning | Adjust max deliveries |
| Customer notice | Show extended ETAs |

**Status:** ⬜ TODO

---

## ⚡ Concurrency Edge Cases

### EC-77: Admin Override During OTP Entry
**Scenario:** Admin remotely unlocks box while customer is entering OTP.

| Solution | Implementation |
|----------|----------------|
| Lock state sync | Real-time lock status |
| Input cancellation | Clear keypad buffer on remote unlock |
| Notification | Inform customer box was remotely opened |

**Status:** ⬜ TODO

---

### EC-78: Delivery Reassignment During Navigation
**Scenario:** Delivery reassigned while rider is en route.

| Solution | Implementation |
|----------|----------------|
| Active notification | Alert rider immediately |
| Route update | Navigation adjusts automatically |
| Confirmation | Require rider acknowledgment |

**Status:** ⬜ TODO

---

### EC-79: Photo Upload and OTP Revocation Race
**Scenario:** Photo uploading when OTP is revoked (delivery cancelled).

| Solution | Implementation |
|----------|----------------|
| Upload completion | Finish upload regardless |
| Metadata flag | Mark photo as "cancelled delivery" |
| No block | Don't fail upload due to status |

**Status:** ⬜ TODO

---

### EC-80: Firmware Update During Active Delivery
**Scenario:** OTA update triggered while delivery is active.

| Solution | Implementation |
|----------|----------------|
| Defer update | Queue until delivery complete |
| Critical only | Only security patches interrupt |
| State preservation | Save state before reboot |

**Status:** ⬜ TODO

---

## Summary: Priority Matrix (Final)

| Priority | Count | Edge Cases |
|----------|-------|------------|
| 🔴 P0 (Critical) | 6 | EC-01, EC-06, EC-18, EC-31, EC-77, EC-80 |
| 🟡 P1 (High) | 16 | EC-02, EC-03, EC-04, EC-07, EC-19, ~~EC-21~~✅, ~~EC-22~~✅, EC-39, EC-41, EC-45, ~~EC-48~~✅, EC-59, EC-61, EC-67, EC-70, EC-78 |
| 🟢 P2 (Medium) | 20 | EC-08, EC-16, ~~EC-23~~✅, ~~EC-25~~✅, EC-29, EC-32, EC-35, EC-42, EC-46, ~~EC-47~~✅, EC-49, EC-54, ~~EC-55~~✅, ~~EC-56~~✅, EC-57, EC-62, ~~EC-68~~✅, EC-69, EC-72, EC-75, EC-79 |
| 🔵 P3 (Low) | 38+ | All others |

**Completed:** EC-21, EC-22, EC-23, EC-25, EC-47, EC-48, EC-55, EC-56, EC-68

---

## ✅ Already Handled

| Edge Case | How |
|-----------|-----|
| EC-01 (No Signal) | Offline OTP + photo queue |
| EC-02 (Missed Assignment) | BLE OTP transfer from phone to box |
| EC-04 (Wrong OTP 5x) | 5min lockout, photo capture, admin reset |
| EC-06 (Both Offline) | Full offline-first design |
| EC-07 (Stale OTP) | 4-hour expiry + revocation on cancellation |
| EC-10 (Queue Full) | MAX_QUEUED_PHOTOS limit |
| EC-11 (Not Home) | 5min wait timer + photo + notification + reschedule |
| EC-12 (Wrong Address) | Rider/customer correction + 50m flexible geofence |
| EC-14 (Timezones) | UTC + server timestamps |
| EC-15 (App Killed) | Foreground service (Android) + background location (iOS) + box GPS failover |
| EC-17 (MITM) | Firebase TLS |
| EC-18 (Tamper) | Reed switch + photo + lockdown |
| EC-21 (Solenoid Closed) | 3x retry + feedback sensor + alerts + physical key fallback |
| EC-22 (Solenoid Open) | Feedback sensor + out-of-service marking + blocks deliveries |
| EC-23 (Camera Fail) | 3x retry + metadata fallback + flagged for review |
| EC-24 (GPS Fail) | Phone GPS redundancy |
| EC-25 (Brownout) | SPIFFS state persistence + auto-resume + reboot event |
| EC-31 (Disputed) | Photo + GPS + OTP log |
| EC-46 (Clock Skew) | Firebase server time |
| EC-47 (Duplicate Records) | Idempotency key + upsert logic + duplicate detection |
| EC-48 (SPIFFS Corruption) | CRC32 checksum + RTC backup + Firebase recovery |
| EC-55 (Firebase Quota) | 80%/95% alerts + local caching + fetch interval reduction |
| EC-56 (Photo Bandwidth) | 800px/60% compression + priority queue + resumable uploads |
| EC-60 (DST) | UTC everywhere |
| EC-68 (Res/Bus Address) | Address type field + dynamic geofence (50m/100m) + building details |

**Total Edge Cases Documented: 80**
**Total Edge Cases Implemented: 27**