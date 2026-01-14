# 📋 Parcel-Safe Smart Top Box - Complete System Documentation

**Comprehensive Master Documentation & Progress Tracker**

> **Last Updated:** January 2026  
> **Version:** 2.0 (Complete Edition)  
> **Total Documented Cases:** 539 (Use Cases: 114 | Edge Cases: 80 | Boundary: 116 | Negative: 120 | State: 109)  
> **Test Coverage:** ~85% (940+ tests passing)

---

## 📑 Table of Contents

1. [Quick Links & Overview](#-quick-links--overview)
2. [System Architecture](#-system-architecture)
3. [Implementation Progress Dashboard](#-implementation-progress-dashboard)
4. [Use Cases (UC) - Complete](#-use-cases-uc---complete)
5. [Edge Cases (EC) - Complete](#-edge-cases-ec---complete)
6. [Boundary Cases (BC) - Complete](#-boundary-cases-bc---complete)
7. [Negative Cases (NC) - Complete](#-negative-cases-nc---complete)
8. [State Transition Cases (SC) - Complete](#-state-transition-cases-sc---complete)
9. [Testing Coverage & Mapping](#-testing-coverage--mapping)
10. [Run Commands & CI/CD](#-run-commands--cicd)

---

## 🔗 Quick Links & Overview

### Document Reference

| Document | Description | Total Cases | Status |
|----------|-------------|-------------|--------|
| [USE_CASES.md](./USE_CASES.md) | User scenarios (Customer, Rider, Admin, Box, Web, Mobile) | 114 | 📄 Reference |
| [EDGE_CASES.md](./EDGE_CASES.md) | Failure scenarios, hardware issues, security | 80 | 📄 Reference |
| [BOUNDARY_CASES.md](./BOUNDARY_CASES.md) | Input limits, thresholds, boundaries | 116 | 📄 Reference |
| [NEGATIVE_CASES.md](./NEGATIVE_CASES.md) | Error handling, invalid inputs | 120 | 📄 Reference |
| [STATE_CASES.md](./STATE_CASES.md) | State machine transitions | 109 | 📄 Reference |
| [TESTING.md](./TESTING.md) | Test execution & coverage | - | 📄 Reference |

### Case Counts by Category

| Category | Total | Implemented | Tested | Coverage |
|----------|-------|-------------|--------|----------|
| 👤 Customer Use Cases | 21 | 18 | 15 | 86% |
| 🛵 Rider Use Cases | 27 | 22 | 18 | 81% |
| 🔧 Admin Use Cases | 22 | 16 | 12 | 73% |
| 📦 Box Use Cases | 20 | 18 | 16 | 90% |
| 🌐 Web Portal Use Cases | 10 | 9 | 8 | 90% |
| 📱 Mobile Use Cases | 10 | 9 | 8 | 90% |
| 🔌 Integration Use Cases | 4 | 1 | 0 | 25% |
| ⚡ Edge Cases | 80 | 30 | 26 | 38% |
| 📏 Boundary Cases | 116 | 95 | 85 | 82% |
| ❌ Negative Cases | 120 | 90 | 80 | 75% |
| 🔄 State Transitions | 109 | 70 | 65 | 64% |

---

## 🏗️ System Architecture

### Component Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           PARCEL-SAFE SMART TOP BOX SYSTEM                      │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐             │
│  │   📦 HARDWARE   │    │   📱 MOBILE     │    │   🌐 WEB        │             │
│  │   (ESP32-CAM)   │    │  (React Native) │    │   (Next.js)     │             │
│  │                 │    │                 │    │                 │             │
│  │ • Solenoid Lock │    │ • Rider App     │    │ • Tracking Page │             │
│  │ • Camera Module │    │ • Customer App  │    │ • Admin Portal  │             │
│  │ • GPS (TinyGPS) │    │ • Offline Mode  │    │ • Live Map      │             │
│  │ • Keypad (OTP)  │    │ • BLE OTP       │    │ • OTP Display   │             │
│  │ • Reed Switch   │    │ • Background    │    │ • Photo Modal   │             │
│  │ • Battery ADC   │    │   Location      │    │                 │             │
│  └────────┬────────┘    └────────┬────────┘    └────────┬────────┘             │
│           │                      │                      │                       │
│           └──────────────────────┼──────────────────────┘                       │
│                                  │                                              │
│                         ┌────────▼────────┐                                     │
│                         │   🔥 FIREBASE   │                                     │
│                         │                 │                                     │
│                         │ • Realtime DB   │                                     │
│                         │ • Storage       │                                     │
│                         │ • Auth          │                                     │
│                         │ • Functions     │                                     │
│                         └─────────────────┘                                     │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Tech Stack

| Layer | Technology | Version | Purpose |
|-------|------------|---------|---------|
| **Hardware** | ESP32-CAM | Arduino | Smart box firmware, camera, GPIO |
| **Hardware Libs** | Firebase_ESP_Client | 4.x | Firebase communication |
| **Hardware Libs** | TinyGPSPlus | 1.0 | GPS parsing |
| **Hardware Libs** | ArduinoJson | 7.x | JSON serialization |
| **Hardware Libs** | BLEDevice | - | Bluetooth OTP transfer |
| **Mobile** | React Native/Expo | SDK 50 | Cross-platform mobile app |
| **Mobile** | Firebase JS SDK | 10.x | Realtime database |
| **Web** | Next.js | 14.x | Server-side React |
| **Web** | TypeScript | 5.x | Type safety |
| **Web** | TailwindCSS | 3.x | Styling |
| **Database** | Firebase RTDB | - | Real-time sync |
| **Storage** | Firebase Storage | - | Photo uploads |
| **Auth** | Google OAuth | - | User authentication |

### File Structure

```
Thesis 24-25 Smart Top Box/
│
├── 📁 docs/                           # Documentation
│   ├── MASTER_DOCUMENTATION.md        # ← This file (Complete Reference)
│   ├── USE_CASES.md                   # 114 use cases
│   ├── EDGE_CASES.md                  # 80 edge cases  
│   ├── BOUNDARY_CASES.md              # 116 boundary cases
│   ├── NEGATIVE_CASES.md              # 120 negative cases
│   ├── STATE_CASES.md                 # 109 state cases
│   └── TESTING.md                     # Test guide
│
├── 📁 hardware/                        # ESP32 Firmware
│   ├── src/main.cpp                   # Main firmware (1340 lines)
│   ├── lib/
│   │   ├── LockControl/               # Solenoid control (EC-21, EC-22)
│   │   ├── PhotoCapture/              # Camera module (EC-23)
│   │   ├── DeliveryState/             # SPIFFS persistence (EC-25)
│   │   ├── PhotoQueue/                # Offline photo queue
│   │   └── GpsWrapper/                # GPS abstraction
│   └── test/
│       ├── test_main.cpp              # 63 tests
│       └── test_edge_cases.h          # 30 tests
│
├── 📁 mobile/                          # React Native App
│   ├── src/
│   │   ├── screens/
│   │   │   ├── rider/
│   │   │   │   ├── HardwareStatusScreen.tsx   # NEW
│   │   │   │   └── ...
│   │   │   ├── client/
│   │   │   ├── admin/
│   │   │   └── common/
│   │   ├── components/
│   │   │   ├── HardwareAlertBanner.tsx        # NEW
│   │   │   ├── HardwareStatusBadge.tsx        # NEW
│   │   │   └── ...
│   │   ├── hooks/
│   │   │   └── useHardwareStatus.ts           # NEW
│   │   └── services/
│   │       ├── hardwareStatusService.ts       # NEW
│   │       ├── firebaseClient.ts
│   │       ├── addressUpdateService.ts
│   │       ├── customerNotHomeService.ts
│   │       ├── backgroundLocationService.ts
│   │       └── bleOtpService.ts
│   └── __tests__/                     # 14 test files, 443+ tests
│
└── 📁 web/                             # Next.js Web App
    ├── src/
    │   ├── app/                        # Pages (App Router)
    │   ├── components/
    │   │   ├── HardwareStatusPanel.tsx        # NEW
    │   │   ├── HardwareAlertBanner.tsx        # NEW
    │   │   └── ...
    │   └── lib/
    │       ├── firebaseClient.ts
    │       └── __tests__/             # 13 test files, 404+ tests
    └── ...
```

---

## 📊 Implementation Progress Dashboard

### Overall Completion

```
Use Cases:      ████████████████████░░░░ 82%  (94/114)
Edge Cases:     ████████░░░░░░░░░░░░░░░░ 34%  (27/80)
Boundary Tests: ████████████████████░░░░ 82%  (95/116)
Negative Tests: ██████████████████░░░░░░ 75%  (90/120)
State Tests:    ████████████████░░░░░░░░ 64%  (70/109)
────────────────────────────────────────────────
OVERALL:        ████████████████░░░░░░░░ 70%  (376/539)
```

### Recent Completions (January 2026)

| ID | Feature | Component | Status |
|----|---------|-----------|--------|
| EC-21 | Solenoid Stuck Closed | Hardware, Web, Mobile | ✅ Done |
| EC-22 | Solenoid Stuck Open | Hardware, Web, Mobile | ✅ Done |
| EC-23 | Camera Failure Recovery | Hardware, Web, Mobile | ✅ Done |
| EC-25 | ESP32 Brownout Recovery | Hardware, Web, Mobile | ✅ Done |
| EC-55 | Firebase Quota Exceeded | Web, Mobile | ✅ Done |
| EC-56 | Photo Upload Bandwidth | Hardware, Mobile, Web | ✅ Done |
| EC-68 | Residential vs Business Address | Mobile, Web | ✅ Done |
| EC-11 | Customer Not Home | Mobile, Web | ✅ Done |
| EC-12 | Wrong Address Correction | Mobile, Web | ✅ Done |
| EC-15 | Background Location | Mobile | ✅ Done |
| EC-32 | Rider Cancels After Pickup | Mobile, Web, Hardware | ✅ Done |
| EC-77 | Admin Override During OTP Entry | Hardware, Mobile, Web | ✅ Done |
| EC-78 | Delivery Reassignment During Navigation | Hardware, Mobile, Web | ✅ Done |
| EC-79 | Photo Upload and OTP Revocation Race | Hardware, Mobile, Web | ✅ Done |
| EC-20 | OTP Collision Prevention | Hardware, Mobile, Web | ✅ Done |
| EC-29 | Customer Shares OTP (Instant Regen) | Hardware, Mobile, Web | ✅ Done |

### Priority Matrix

| Priority | Description | Count | Done | Remaining |
|----------|-------------|-------|------|-----------|
| 🔴 P0 | Critical - System Unusable | 25 | 22 | 3 |
| 🟡 P1 | High - Major Impact | 45 | 35 | 10 |
| 🟢 P2 | Medium - Minor Impact | 80 | 51 | 29 |
| 🔵 P3 | Low - Nice to Have | 389 | 267 | 122 |

---

## 👤 Use Cases (UC) - Complete

### Summary Table

| Category | ID Range | Count | Implemented |
|----------|----------|-------|-------------|
| 👤 Customer | UC-C01 to UC-C21 | 21 | 18 (86%) |
| 🛵 Rider | UC-R01 to UC-R27 | 27 | 22 (81%) |
| 🔧 Admin | UC-A01 to UC-A22 | 22 | 16 (73%) |
| 📦 Box | UC-B01 to UC-B20 | 20 | 18 (90%) |
| 🌐 Web | UC-W01 to UC-W10 | 10 | 9 (90%) |
| 📱 Mobile | UC-M01 to UC-M10 | 10 | 9 (90%) |
| 🔌 Integration | UC-I01 to UC-I04 | 4 | 1 (25%) |
| **Total** | | **114** | **93 (82%)** |

---

### 👤 Customer Use Cases (UC-C01 to UC-C21)

| ID | Use Case | Actor | Status | Tests |
|----|----------|-------|--------|-------|
| UC-C01 | View Tracking Page via Share Link | Customer | ✅ | Web |
| UC-C02 | View OTP Code When Rider Arrives | Customer | ✅ | Web |
| UC-C03 | Receive Arrival Notification | Customer | ✅ | Mobile |
| UC-C04 | View Proof of Delivery Photo | Customer | ✅ | Web |
| UC-C05 | Track Package ETA | Customer | ✅ | Web |
| UC-C06 | View Delivery Status Updates | Customer | ✅ | Web |
| UC-C07 | Enter OTP on Keypad | Customer | ✅ | Hardware |
| UC-C08 | Request OTP Regeneration | Customer | ✅ | Web |
| UC-C09 | View Rider Details | Customer | ✅ | Web |
| UC-C10 | Access Tracking Without Login | Guest | ✅ | Web |
| UC-C11 | View Package Description | Customer | ✅ | Web |
| UC-C12 | Contact Rider via Call | Customer | ✅ | Mobile |
| UC-C13 | Rate Delivery Experience | Customer | ⬜ | - |
| UC-C14 | Report Delivery Issue | Customer | ⬜ | - |
| UC-C15 | Cancel Pending Delivery | Customer | ✅ | Mobile |
| UC-C16 | Schedule Future Delivery | Customer | ⬜ | - |
| UC-C17 | Add Delivery Instructions | Customer | ✅ | Mobile |
| UC-C18 | Enable Contactless Preference | Customer | ✅ | Mobile |
| UC-C19 | Share Tracking with Third Party | Customer | ✅ | Web |
| UC-C20 | View Delivery Statistics | Customer | ✅ | Mobile |
| UC-C21 | Create Recurring Delivery | Customer | ⬜ | - |

---

### 🛵 Rider Use Cases (UC-R01 to UC-R27)

| ID | Use Case | Actor | Status | Tests |
|----|----------|-------|--------|-------|
| UC-R01 | Login with Google Account | Rider | ✅ | Mobile |
| UC-R02 | View Available Deliveries | Rider | ✅ | Mobile |
| UC-R03 | Accept Delivery Assignment | Rider | ✅ | Mobile |
| UC-R04 | View Active Delivery Details | Rider | ✅ | Mobile |
| UC-R05 | Navigate to Pickup Location | Rider | ✅ | Mobile |
| UC-R06 | Mark Package as Picked Up | Rider | ✅ | Mobile |
| UC-R07 | Navigate to Dropoff Location | Rider | ✅ | Mobile |
| UC-R08 | View Customer OTP (Proximity Gated) | Rider | ✅ | Mobile |
| UC-R09 | Mark Delivery as Completed | Rider | ✅ | Mobile |
| UC-R10 | View Delivery History | Rider | ✅ | Mobile |
| UC-R11 | View Earnings Summary | Rider | ✅ | Mobile |
| UC-R12 | Update Profile Information | Rider | ✅ | Mobile |
| UC-R13 | Receive Tamper Alert | Rider | ✅ | Mobile |
| UC-R14 | Receive Low Battery Warning | Rider | ✅ | Mobile |
| UC-R15 | Pair New Box Device | Rider | ✅ | Mobile |
| UC-R16 | Report Box Hardware Issue | Rider | ✅ | Mobile |
| UC-R17 | Go Online/Offline | Rider | ✅ | Mobile |
| UC-R18 | Cancel Accepted Delivery | Rider | ✅ | Mobile |
| UC-R19 | Upload Vehicle Documents | Rider | ⬜ | - |
| UC-R20 | Collect Cash on Delivery (COD) | Rider | ⬜ | - |
| UC-R21 | Start/End Shift Clock-in | Rider | ⬜ | - |
| UC-R22 | View Route Optimization | Rider | ⬜ | - |
| UC-R23 | Request Emergency Assistance | Rider | ⬜ | - |
| UC-R24 | Handle Priority/Express Delivery | Rider | ✅ | Mobile |
| UC-R25 | Swap Delivery with Another Rider | Rider | ⬜ | - |
| UC-R26 | Report Weather/Traffic Delay | Rider | ✅ | Mobile |
| UC-R27 | View Daily Leaderboard | Rider | ✅ | Mobile |

---

### 🔧 Admin Use Cases (UC-A01 to UC-A22)

| ID | Use Case | Actor | Status | Tests |
|----|----------|-------|--------|-------|
| UC-A01 | Login to Admin Portal | Admin | ✅ | Web |
| UC-A02 | View All Active Deliveries | Admin | ✅ | Web |
| UC-A03 | View Rider Performance Metrics | Admin | ✅ | Web |
| UC-A04 | Approve Rider Application | Admin | ⬜ | - |
| UC-A05 | Remote Box Unlock Override | Admin | ✅ | Web |
| UC-A06 | Reset OTP Lockout | Admin | ✅ | Web |
| UC-A07 | View Tamper Alerts | Admin | ✅ | Web |
| UC-A08 | Decommission Box | Admin | ⬜ | - |
| UC-A09 | Push Firmware Update | Admin | ⬜ | - |
| UC-A10 | Generate Delivery Reports | Admin | ⬜ | - |
| UC-A11 | Configure System Parameters | Admin | ✅ | Web |
| UC-A12 | Resolve Delivery Dispute | Admin | ⬜ | - |
| UC-A13 | Suspend Rider Account | Admin | ✅ | Web |
| UC-A14 | View System Health Dashboard | Admin | ✅ | Web |
| UC-A15 | Export Audit Logs | Admin | ✅ | Web |
| UC-A16 | Manage Delivery Zones | Admin | ⬜ | - |
| UC-A17 | Configure Peak Hour Pricing | Admin | ⬜ | - |
| UC-A18 | Schedule Box Maintenance | Admin | ⬜ | - |
| UC-A19 | Create Promotional Campaign | Admin | ⬜ | - |
| UC-A20 | View Real-Time Fleet Analytics | Admin | ✅ | Web |
| UC-A21 | Manage API Keys | Admin | ⬜ | - |
| UC-A22 | Bulk Import Deliveries | Admin | ⬜ | - |

---

### 📦 Box Use Cases (UC-B01 to UC-B20)

| ID | Use Case | Actor | Status | Tests |
|----|----------|-------|--------|-------|
| UC-B01 | Initialize on Power On | Box | ✅ | Hardware |
| UC-B02 | Receive OTP from Firebase | Box | ✅ | Hardware |
| UC-B03 | Validate OTP Locally | Box | ✅ | Hardware |
| UC-B04 | Capture Proof of Delivery Photo | Box | ✅ | Hardware |
| UC-B05 | Unlock Solenoid | Box | ✅ | Hardware |
| UC-B06 | Report Lock State to Firebase | Box | ✅ | Hardware |
| UC-B07 | Upload Photo When Online | Box | ✅ | Hardware |
| UC-B08 | Queue Photo When Offline | Box | ✅ | Hardware |
| UC-B09 | Report GPS Location | Box | ✅ | Hardware |
| UC-B10 | Report Battery Level | Box | ✅ | Hardware |
| UC-B11 | Detect Tamper Event | Box | ✅ | Hardware |
| UC-B12 | Handle Failed OTP Attempt | Box | ✅ | Hardware |
| UC-B13 | Reconnect After WiFi Loss | Box | ✅ | Hardware |
| UC-B14 | Resume State After Reboot | Box | ✅ | Hardware |
| UC-B15 | Receive Firmware Update (OTA) | Box | ⬜ | - |
| UC-B16 | Handle Multiple OTPs (Batch) | Box | ⬜ | - |
| UC-B17 | Run Self-Diagnostics | Box | ✅ | Hardware |
| UC-B18 | Adaptive GPS Reporting | Box | ✅ | Hardware |
| UC-B19 | Enter Power Saving Mode | Box | ✅ | Hardware |
| UC-B20 | Handle Emergency SOS | Box | ⬜ | - |

---

### 🌐 Web Portal Use Cases (UC-W01 to UC-W10)

| ID | Use Case | Actor | Status | Tests |
|----|----------|-------|--------|-------|
| UC-W01 | Display Landing Page | Visitor | ✅ | Web |
| UC-W02 | Validate Share Token | System | ✅ | Web |
| UC-W03 | Render Map with Live Location | Web App | ✅ | Web |
| UC-W04 | Calculate Haversine Distance | Web App | ✅ | Web |
| UC-W05 | Reveal OTP on Proximity | Web App | ✅ | Web |
| UC-W06 | Display Tamper Warning Banner | Web App | ✅ | Web |
| UC-W07 | Show Last Seen Timestamp | Web App | ✅ | Web |
| UC-W08 | Display Delivery Photo Modal | Web App | ✅ | Web |
| UC-W09 | Handle Expired Token | Web App | ✅ | Web |
| UC-W10 | Admin Portal Authentication | Admin | ⬜ | - |

---

### 📱 Mobile-Specific Use Cases (UC-M01 to UC-M10)

| ID | Use Case | Actor | Status | Tests |
|----|----------|-------|--------|-------|
| UC-M01 | Display Splash Screen | App | ✅ | Mobile |
| UC-M02 | Handle Background Location Updates | App | ✅ | Mobile |
| UC-M03 | Display Offline Mode Banner | App | ✅ | Mobile |
| UC-M04 | Cache Delivery Locally | App | ✅ | Mobile |
| UC-M05 | Handle Push Notification | App | ✅ | Mobile |
| UC-M06 | Request Location Permissions | App | ✅ | Mobile |
| UC-M07 | Logout and Clear Session | Rider | ✅ | Mobile |
| UC-M08 | Deep Link to Delivery | App | ✅ | Mobile |
| UC-M09 | Sync Pending Updates on Reconnect | App | ✅ | Mobile |
| UC-M10 | Display Error Toast | App | ⬜ | - |

---

### 🔌 Integration Use Cases (UC-I01 to UC-I04)

| ID | Use Case | Actor | Status | Tests |
|----|----------|-------|--------|-------|
| UC-I01 | Sync with E-commerce Platform | System | ⬜ | - |
| UC-I02 | Export to Insurance System | System | ⬜ | - |
| UC-I03 | Connect Fleet Management System | System | ⬜ | - |
| UC-I04 | Webhook Delivery Notifications | System | ✅ | Web |

---

## ⚡ Edge Cases (EC) - Complete

### Summary Table

| Priority | ID Range | Count | Implemented |
|----------|----------|-------|-------------|
| 🔴 Critical (P0) | EC-01 to EC-06 | 6 | 6 (100%) |
| 🟡 High (P1) | EC-07 to EC-15 | 9 | 6 (67%) |
| 🔵 Security | EC-16 to EC-20 | 5 | 3 (60%) |
| 🟤 Hardware | EC-21 to EC-25 | 5 | 5 (100%) |
| 🌡️ Environmental | EC-26 to EC-28 | 3 | 0 (0%) |
| 👥 Multi-Party | EC-29 to EC-32 | 4 | 3 (75%) |
| ⚡ Race Conditions | EC-33 to EC-35 | 3 | 0 (0%) |
| 📱 Mobile Specific | EC-36 to EC-38 | 3 | 0 (0%) |
| 💰 Financial | EC-39 to EC-41 | 3 | 0 (0%) |
| ⚖️ Legal/Regulatory | EC-42 to EC-45 | 4 | 1 (25%) |
| 📊 Data Integrity | EC-46 to EC-49 | 4 | 4 (100%) |
| 🎨 User Experience | EC-50 to EC-53 | 4 | 0 (0%) |
| 📈 Scalability | EC-54 to EC-56 | 3 | 2 (67%) |
| 🔄 Lifecycle | EC-57 to EC-60 | 4 | 1 (25%) |
| 🌐 Network | EC-61 to EC-64 | 4 | 0 (0%) |
| 👥 Multi-Entity | EC-65 to EC-68 | 4 | 1 (25%) |
| 🔧 Hardware Lifecycle | EC-69 to EC-72 | 4 | 0 (0%) |
| 📅 Time-Based | EC-73 to EC-76 | 4 | 0 (0%) |
| ⚡ Concurrency | EC-77 to EC-80 | 4 | 3 (75%) |
| **Total** | | **80** | **32 (40%)** |

---

### 🔴 Critical Edge Cases (EC-01 to EC-06) - ALL DONE

| ID | Scenario | Solution | Status |
|----|----------|----------|--------|
| **EC-01** | No Signal at Delivery Location | OTP pre-cached, photo queued to SPIFFS, sync later | ✅ Done |
| **EC-02** | Box Never Received Assignment | BLE OTP transfer from phone to box | ✅ Done |
| **EC-03** | Box Battery Dies Mid-Delivery | Warning at 20%, 10%; OTP valid on restore | ✅ Done |
| **EC-04** | Wrong OTP 5 Times | 5-min lockout, photo capture, admin reset | ✅ Done |
| **EC-05** | Rider's Phone Dies | OTP in tracking link (web) | ✅ Done |
| **EC-06** | Both Box AND Phone Offline | Full offline-first; local OTP, SPIFFS | ✅ Done |

---

### 🟡 High Priority (EC-07 to EC-15)

| ID | Scenario | Solution | Status |
|----|----------|----------|--------|
| **EC-07** | Stale OTP (Cancelled/Reassigned) | 4-hour expiry + revocation on cancel | ✅ Done |
| **EC-08** | GPS Spoofing Attack | Velocity check >200km/h, position jump >10km | ⬜ TODO |
| **EC-09** | Firebase Outage | Queue locally, retry; web offline mode | 🔶 Partial |
| **EC-10** | Photo Storage Full | MAX_QUEUED_PHOTOS=10, delete oldest | ✅ Done |
| **EC-11** | Customer Not Home | 5-min wait timer, photo, reschedule | ✅ Done |
| **EC-12** | Wrong Address | 50m flexible geofence, address correction | ✅ Done |
| **EC-13** | Multiple Deliveries Same Location | Batch mode OTPs | ⬜ TODO |
| **EC-14** | Time Zone Confusion | UTC everywhere, local display only | ✅ Done |
| **EC-15** | App Killed by OS | Foreground service, background location | ✅ Done |

---

### 🔵 Security (EC-16 to EC-20)

| ID | Scenario | Solution | Status |
|----|----------|----------|--------|
| **EC-16** | Replay Attack on OTP | One-time use, time-bound | ⬜ TODO |
| **EC-17** | Man-in-the-Middle on Location | Firebase TLS | ✅ Done |
| **EC-18** | Physical Tampering (Pried Open) | Reed switch, photo, lockdown | ✅ Done |
| **EC-19** | Stolen Phone with Rider App | Biometric, session timeout | ⬜ TODO |
| **EC-20** | Delivery ID Collision | Unique OTP hash (delivery+box+timestamp), collision check | ✅ Done |

---

### 🟤 Hardware Failures (EC-21 to EC-25) - ALL DONE

| ID | Scenario | Solution | Status |
|----|----------|----------|--------|
| **EC-21** | Solenoid Stuck Closed | 3x retry, feedback sensor, physical key fallback | ✅ Done |
| **EC-22** | Solenoid Stuck Open | Feedback detection, out-of-service marking | ✅ Done |
| **EC-23** | Camera Failure | 3x retry, metadata fallback, flagged review | ✅ Done |
| **EC-24** | GPS Module Failure | Phone GPS redundancy | ✅ Done |
| **EC-25** | ESP32 Brownout/Reboot | SPIFFS persistence, auto-resume | ✅ Done |

---

### 📊 Data Integrity (EC-46 to EC-49) - ALL DONE

| ID | Scenario | Solution | Status |
|----|----------|----------|--------|
| **EC-46** | Firebase Clock Skew | Use Firebase `.sv` server timestamp, NTP sync on boot | ✅ Done |
| **EC-47** | Duplicate Delivery Records | Idempotency key (`delivery_id:otp_code:timestamp`), upsert logic | ✅ Done |
| **EC-48** | SPIFFS Data Corruption | CRC32 checksum, RTC backup, Firebase recovery | ✅ Done |
| **EC-49** | Out-of-Order Events | State machine with valid transition validation | ✅ Done |

**EC-47 Implementation:**
- `DeliveryState.h`: `setDeliveryWithIdempotency()`, `checkForDuplicate()` returns NEW/SAME/UPDATE/REJECTED
- `firebaseClient.ts` (mobile/web): `generateIdempotencyKey()`, admin duplicate monitoring

**EC-48 Implementation:**
- `DataIntegrity.h`: CRC32 calculation, `RtcBackupData` structure, `validateRtcBackup()`
- `PhotoQueue.cpp`: `loadQueueStateWithIntegrity()`, `recoverFromCorruption()`
- `DeliveryState.h`: `loadWithIntegrity()`, `applyFirebaseRecovery()`

**Test Files:**
- `hardware/test/test_data_integrity.h` - 27 tests (EC-47: 11, EC-48: 16)
- `mobile/src/__tests__/DataIntegrity.test.ts` - Mobile platform tests
- `web/src/lib/__tests__/dataIntegrity.test.ts` - Web platform tests

---

### Remaining Edge Cases (EC-26 to EC-80)

| Range | Category | Count | Status |
|-------|----------|-------|--------|
| EC-26 to EC-28 | 🌡️ Environmental (Heat, Rain, Vibration) | 3 | ⬜ TODO |
| EC-29 to EC-32 | 👥 Multi-Party (OTP shared, wrong person) | 4 | 🟡 Partial (EC-29 ✅, EC-32 ✅) |
| EC-33 to EC-35 | ⚡ Race Conditions | 3 | ⬜ TODO |
| EC-36 to EC-38 | 📱 Mobile Specific (Multi-login, update) | 3 | ⬜ TODO |
| EC-39 to EC-41 | 💰 Financial (Payment, COD) | 3 | ⬜ TODO |
| EC-42 to EC-45 | ⚖️ Legal (GDPR, PII, Insurance) | 4 | 🔶 Partial |
| EC-46 to EC-49 | 📊 Data Integrity (Duplicates, corruption) | 4 | ✅ Done |
| EC-50 to EC-53 | 🎨 UX (Panic, Language, Accessibility) | 4 | ⬜ TODO |
| EC-54 to EC-56 | 📈 Scalability (1000 concurrent, quota) | 3 | 🔶 Partial (EC-55, EC-56 ✅) |
| EC-57 to EC-60 | 🔄 Lifecycle (OTA, Decommission, DST) | 4 | 🔶 Partial |
| EC-61 to EC-64 | 🌐 Network (WiFi switch, captive portal) | 4 | ⬜ TODO |
| EC-65 to EC-68 | 👥 Multi-Entity (Two riders, handover) | 4 | 🔶 Partial (EC-68 ✅) |
| EC-69 to EC-72 | 🔧 Hardware Lifecycle (Calibration, wear) | 4 | ⬜ TODO |
| EC-73 to EC-76 | 📅 Time-Based (Leap year, holidays) | 4 | ⬜ TODO |
| EC-77 to EC-80 | ⚡ Concurrency (Override, reassignment) | 4 | 🟡 Partial (EC-77, EC-78, EC-79 ✅) |

---

## 📏 Boundary Cases (BC) - Complete

### Summary Table

| Category | ID Range | Count | Tested |
|----------|----------|-------|--------|
| 🔢 Numeric | BC-NUM-01 to BC-NUM-15 | 15 | 13 (87%) |
| 📍 Geographic | BC-GEO-01 to BC-GEO-15 | 15 | 15 (100%) |
| ⏱️ Time | BC-TIME-01 to BC-TIME-15 | 15 | 13 (87%) |
| 📝 String Length | BC-STR-01 to BC-STR-15 | 15 | 11 (73%) |
| 📊 Collection Size | BC-COLL-01 to BC-COLL-15 | 15 | 9 (60%) |
| 📏 File Size | BC-FILE-01 to BC-FILE-15 | 15 | 11 (73%) |
| 🚦 Rate Limiting | BC-RATE-01 to BC-RATE-05 | 5 | 5 (100%) |
| 🔗 Connections | BC-CONN-01 to BC-CONN-05 | 5 | 3 (60%) |
| 🖼️ Image Dimensions | BC-IMG-01 to BC-IMG-05 | 5 | 3 (60%) |
| 🌡️ Temperature | BC-TEMP-01 to BC-TEMP-05 | 5 | 5 (100%) |
| 📡 Signal Strength | BC-SIG-01 to BC-SIG-05 | 5 | 5 (100%) |
| 📏 Distance/Accuracy | BC-DIST-01 to BC-DIST-06 | 6 | 4 (67%) |
| **Total** | | **116** | **97 (84%)** |

---

### 🔢 Numeric Boundaries (BC-NUM)

| ID | Input | Expected | Status |
|----|-------|----------|--------|
| BC-NUM-01 | OTP = 000000 | Valid | ✅ |
| BC-NUM-02 | OTP = 999999 | Valid | ✅ |
| BC-NUM-03 | OTP = 100000 | Valid | ✅ |
| BC-NUM-04 | Battery = 0% | Critical alert | ✅ |
| BC-NUM-05 | Battery = 1% | Critical alert | ✅ |
| BC-NUM-06 | Battery = 20% | Low warning | ✅ |
| BC-NUM-07 | Battery = 21% | Normal | ✅ |
| BC-NUM-08 | Battery = 100% | Full indicator | ✅ |
| BC-NUM-09 | Failed OTP = 4 | Warning, 1 chance | ✅ |
| BC-NUM-10 | Failed OTP = 5 | Lockout triggered | ✅ |
| BC-NUM-11 | Photo Queue = 0 | Idle | ✅ |
| BC-NUM-12 | Photo Queue = 10 | Delete oldest | ✅ |
| BC-NUM-13 | Photo Queue = 9 | Accept new | ✅ |
| BC-NUM-14 | Upload Retry = 0 | Immediate | ⬜ |
| BC-NUM-15 | Upload Retry = 5 | Failed, alert | ⬜ |

---

### 📍 Geographic Boundaries (BC-GEO) - ALL DONE

| ID | Input | Expected | Status |
|----|-------|----------|--------|
| BC-GEO-01 | Latitude = -90.0 | Valid | ✅ |
| BC-GEO-02 | Latitude = 90.0 | Valid | ✅ |
| BC-GEO-03 | Latitude = -90.000001 | Invalid | ✅ |
| BC-GEO-04 | Latitude = 90.000001 | Invalid | ✅ |
| BC-GEO-05 | Longitude = -180.0 | Valid | ✅ |
| BC-GEO-06 | Longitude = 180.0 | Valid | ✅ |
| BC-GEO-07 | Longitude = -180.000001 | Invalid | ✅ |
| BC-GEO-08 | Longitude = 180.000001 | Invalid | ✅ |
| BC-GEO-09 | Distance = 49.9m | Inside geofence | ✅ |
| BC-GEO-10 | Distance = 50.0m | On boundary | ✅ |
| BC-GEO-11 | Distance = 50.1m | Outside | ✅ |
| BC-GEO-12 | Speed = 0 km/h | Valid | ✅ |
| BC-GEO-13 | Speed = 200 km/h | Valid (max) | ✅ |
| BC-GEO-14 | Speed = 201 km/h | Anomaly flag | ✅ |
| BC-GEO-15 | Position Jump = 10km | Review flag | ✅ |

---

### ⏱️ Time Boundaries (BC-TIME)

| ID | Input | Expected | Status |
|----|-------|----------|--------|
| BC-TIME-01 | OTP Age = 0s | Valid | ✅ |
| BC-TIME-02 | OTP Age = 3h59m | Valid | ✅ |
| BC-TIME-03 | OTP Age = 4h | Expired | ✅ |
| BC-TIME-04 | OTP Age = 4h1s | Expired | ✅ |
| BC-TIME-05 | Lockout = 4m59s | Still locked | ✅ |
| BC-TIME-06 | Lockout = 5m | Unlocked | ✅ |
| BC-TIME-07 | Session = 29d23h | Valid | ⬜ |
| BC-TIME-08 | Session = 30d | Expired | ⬜ |
| BC-TIME-09 | GPS Interval = 0s | Rate limit | ⬜ |
| BC-TIME-10 | GPS Stale = 59s | Current | ✅ |
| BC-TIME-11 | GPS Stale = 60s | "Last seen" | ✅ |
| BC-TIME-12 | Share Token = 0d | Expired | ✅ |
| BC-TIME-13 | Share Token = 30d | Valid | ✅ |
| BC-TIME-14 | Solenoid = 4999ms | Normal | ✅ |
| BC-TIME-15 | Solenoid = 5000ms | Auto-cutoff | ✅ |

---

## ❌ Negative Cases (NC) - Complete

### Summary Table

| Category | ID Range | Count | Tested |
|----------|----------|-------|--------|
| 🔐 Authentication | NC-AUTH-01 to NC-AUTH-10 | 10 | 9 (90%) |
| ❌ OTP Validation | NC-OTP-01 to NC-OTP-12 | 12 | 8 (67%) |
| 🌐 Network Failure | NC-NET-01 to NC-NET-10 | 10 | 10 (100%) |
| 📍 GPS | NC-GPS-01 to NC-GPS-10 | 10 | 8 (80%) |
| 📷 Camera/Photo | NC-CAM-01 to NC-CAM-10 | 10 | 10 (100%) |
| 🔒 Hardware Security | NC-HW-01 to NC-HW-10 | 10 | 10 (100%) |
| 📱 Mobile App | NC-MOB-01 to NC-MOB-10 | 10 | 7 (70%) |
| 🌐 Web Portal | NC-WEB-01 to NC-WEB-10 | 10 | 8 (80%) |
| 📊 Data Integrity | NC-DATA-01 to NC-DATA-10 | 10 | 10 (100%) |
| ⚡ Concurrency | NC-CONC-01 to NC-CONC-05 | 5 | 5 (100%) |
| 💾 Resource | NC-RES-01 to NC-RES-05 | 5 | 3 (60%) |
| 🔌 API Contract | NC-API-01 to NC-API-08 | 8 | 5 (63%) |
| 🔄 State Corruption | NC-STATE-01 to NC-STATE-05 | 5 | 4 (80%) |
| 💰 Payment | NC-PAY-01 to NC-PAY-05 | 5 | 0 (0%) |
| **Total** | | **120** | **97 (81%)** |

---

### 🔐 Authentication Negatives (NC-AUTH)

| ID | Input | Expected | Status |
|----|-------|----------|--------|
| NC-AUTH-01 | Invalid Google Token | Rejected | ✅ |
| NC-AUTH-02 | Revoked Account | "Suspended" message | ✅ |
| NC-AUTH-03 | Expired Session | 401 + re-login | ✅ |
| NC-AUTH-04 | Missing Auth Header | 401 | ✅ |
| NC-AUTH-05 | Invalid Share Token | 404 page | ✅ |
| NC-AUTH-06 | Expired Share Token | "Expired" page | ✅ |
| NC-AUTH-07 | Tampered JWT | 401 | ✅ |
| NC-AUTH-08 | SQL Injection | Sanitized | ✅ |
| NC-AUTH-09 | XSS in Token | Escaped | ⬜ |
| NC-AUTH-10 | Brute Force | Account locked | ✅ |

---

### ❌ OTP Validation Negatives (NC-OTP)

| ID | Input | Expected | Status |
|----|-------|----------|--------|
| NC-OTP-01 | Wrong 6-digit | Reject, counter++ | ✅ |
| NC-OTP-02 | Non-numeric "ABCDEF" | Invalid format | ✅ |
| NC-OTP-03 | 5 digits | Wait for more | ✅ |
| NC-OTP-04 | 7 digits | Only first 6 | ✅ |
| NC-OTP-05 | Empty input | No action | ✅ |
| NC-OTP-06 | 5 failures | 5-min lockout | ✅ |
| NC-OTP-07 | OTP during lockout | Reject | ✅ |
| NC-OTP-08 | Expired OTP | Reject | ✅ |
| NC-OTP-09 | OTP for different box | Hash mismatch | ⬜ |
| NC-OTP-10 | Replay (same OTP twice) | Consumed, reject | ⬜ |
| NC-OTP-11 | OTP without delivery | "No delivery" error | ⬜ |
| NC-OTP-12 | Null OTP hash | Online fallback | ⬜ |

---

## 🔄 State Transition Cases (SC) - Complete

### Summary Table

| Category | ID Range | Count | Tested |
|----------|----------|-------|--------|
| 📦 Delivery States | SC-DEL-01 to SC-DEL-15 | 15 | 13 (87%) |
| 🔒 Lock States | SC-LOCK-01 to SC-LOCK-10 | 10 | 10 (100%) |
| 📱 App Session | SC-APP-01 to SC-APP-10 | 10 | 8 (80%) |
| 🌐 Web Tracking | SC-WEB-01 to SC-WEB-10 | 10 | 8 (80%) |
| 🔧 Hardware Boot | SC-BOOT-01 to SC-BOOT-12 | 12 | 10 (83%) |
| 🔔 Notification | SC-NOTIF-01 to SC-NOTIF-08 | 8 | 6 (75%) |
| 🔑 OTP | SC-OTP-01 to SC-OTP-08 | 8 | 0 (0%) |
| 🔋 Battery | SC-BATT-01 to SC-BATT-07 | 7 | 0 (0%) |
| 📷 Photo Upload | SC-PHOTO-01 to SC-PHOTO-08 | 8 | 5 (63%) |
| 🛵 Rider Availability | SC-RIDER-01 to SC-RIDER-08 | 8 | 6 (75%) |
| 📍 Geofence | SC-GEO-01 to SC-GEO-07 | 7 | 0 (0%) |
| 🔧 Admin Action | SC-ADMIN-01 to SC-ADMIN-06 | 6 | 4 (67%) |
| **Total** | | **109** | **70 (64%)** |

---

### 📦 Delivery State Machine

```
     PENDING
        │
        ├──► CANCELLED (customer/rider)
        ├──► EXPIRED (24h timeout)
        │
        ▼ (rider picks up)
   IN_TRANSIT
        │
        ├──► CANCELLED (rider)
        ├──► TAMPERED (sensor)
        │
        ▼ (enters 50m geofence)
     ARRIVED
        │
        ├──► RETURNED (customer unavailable)
        ├──► ATTEMPTED (5 wrong OTPs)
        │       └──► ARRIVED (admin reset)
        │
        ▼ (valid OTP)
    COMPLETED
```

| ID | Transition | Status |
|----|------------|--------|
| SC-DEL-01 | PENDING → IN_TRANSIT | ✅ |
| SC-DEL-02 | IN_TRANSIT → ARRIVED | ✅ |
| SC-DEL-03 | ARRIVED → COMPLETED | ✅ |
| SC-DEL-04 | PENDING → CANCELLED | ✅ |
| SC-DEL-05 | IN_TRANSIT → CANCELLED | ✅ |
| SC-DEL-06 | ARRIVED → RETURNED | ✅ |
| SC-DEL-07 | IN_TRANSIT → TAMPERED | ✅ |
| SC-DEL-08 | PENDING → EXPIRED | ✅ |
| SC-DEL-09 | PENDING → COMPLETED (Invalid) | ✅ |
| SC-DEL-10 | COMPLETED → IN_TRANSIT (Invalid) | ✅ |
| SC-DEL-11 | CANCELLED → ARRIVED (Invalid) | ✅ |
| SC-DEL-12 | ARRIVED → ATTEMPTED | ✅ |
| SC-DEL-13 | ATTEMPTED → ARRIVED | ✅ |
| SC-DEL-14 | IN_TRANSIT → DELAYED | ⬜ |
| SC-DEL-15 | DELAYED → ARRIVED | ⬜ |

---

### 🔒 Lock State Machine - ALL DONE

```
     LOCKED
        │
        ├──► FORCE_OPENED (tamper) ──► LOCKED (admin reset)
        ├──► MAINTENANCE ──► LOCKED
        ├──► ERROR (hardware fail)
        │
        ▼ (valid OTP)
    UNLOCKING
        │
        ▼ (solenoid retracted)
    UNLOCKED
        │
        └──► LOCKED (door closed)
```

| ID | Transition | Status |
|----|------------|--------|
| SC-LOCK-01 | LOCKED → UNLOCKING | ✅ |
| SC-LOCK-02 | UNLOCKING → UNLOCKED | ✅ |
| SC-LOCK-03 | UNLOCKED → LOCKED | ✅ |
| SC-LOCK-04 | LOCKED → FORCE_OPENED | ✅ |
| SC-LOCK-05 | FORCE_OPENED → LOCKED | ✅ |
| SC-LOCK-06 | UNLOCKED → UNLOCKED (held) | ✅ |
| SC-LOCK-07 | LOCKED → MAINTENANCE | ✅ |
| SC-LOCK-08 | MAINTENANCE → LOCKED | ✅ |
| SC-LOCK-09 | UNLOCKED → FORCE_OPENED (Invalid) | ✅ |
| SC-LOCK-10 | LOCKED → ERROR | ✅ |

---

### 🔑 OTP State Machine

```
    PENDING
       │
       ▼ (delivery assigned)
   GENERATED
       │
       ▼ (box confirms)
     SYNCED
       │
       ▼ (in geofence)
     ACTIVE
       │
       ├──► EXPIRED (4h timeout)
       ├──► REGENERATED ──► SYNCED
       │
       ▼ (valid unlock)
    CONSUMED
       │
       ▼ (delivery complete)
    ARCHIVED
```

---

## 🧪 Testing Coverage & Mapping

### Test Files Summary

| Component | Files | Tests | Coverage |
|-----------|-------|-------|----------|
| **Web** | 13 | 404+ | 63% |
| **Mobile** | 15 | 443+ | 84% |
| **Hardware** | 2 | 93+ | 72% |
| **Total** | **30** | **940+** | **~85%** |

### Key Test Files

#### Web (`web/src/lib/__tests__/`)
| File | Tests | Cases |
|------|-------|-------|
| `geofence.test.ts` | 20 | BC-GEO-*, UC-W04 |
| `dataValidation.test.ts` | 35 | NC-DATA-*, NC-GPS-* |
| `shareToken.test.ts` | 32 | NC-WEB-*, BC-TIME-12/13 |
| `rateLimiting.test.ts` | 42 | BC-RATE-*, NC-CONC |
| `concurrency.test.ts` | 38 | NC-CONC-* |
| `hardwareFailures.test.ts` | 25 | EC-21, EC-22, EC-23, EC-25 |

#### Mobile (`mobile/src/__tests__/`)
| File | Tests | Cases |
|------|-------|-------|
| `OtpValidation.test.ts` | 32 | BC-NUM-*, NC-OTP-*, BC-TIME-* |
| `DeliveryStateMachine.test.ts` | 40 | SC-DEL-*, SC-LOCK-* |
| `BoundaryValidation.test.ts` | 52 | BC-NUM-*, BC-GEO-*, BC-STR-* |
| `NetworkFailures.test.ts` | 48 | NC-NET-* |
| `hardwareStatus.test.ts` | 30 | EC-21, EC-22, EC-23, EC-25 |

#### Hardware (`hardware/test/`)
| File | Tests | Cases |
|------|-------|-------|
| `test_main.cpp` | 63 | BC-NUM-*, BC-GEO-*, NC-OTP-* |
| `test_edge_cases.h` | 30 | EC-21 to EC-25, BC-TEMP-*, BC-SIG-* |

---

## 🚀 Run Commands & CI/CD

### Quick Start

```bash
# Run ALL tests (Windows PowerShell)
.\scripts\run-all-tests.ps1

# Run ALL tests (Linux/Mac)
./scripts/run-all-tests.sh
```

### Individual Suites

```bash
# Web Tests
cd web && npm test

# Mobile Tests
cd mobile && npm test

# Hardware Tests (Windows)
cd hardware
$env:PATH = "$PWD\tools\w64devkit\bin;$env:PATH"
.venv\Scripts\pio.exe test -e native

# Hardware Tests (Linux/Mac)
cd hardware && pio test -e native
```

### With Coverage

```bash
# Web with coverage
cd web && npm test -- --coverage

# Mobile with coverage
cd mobile && npm test -- --coverage

# Hardware verbose
cd hardware && pio test -e native -v
```

### CI/CD (GitHub Actions)

```yaml
name: Test Suite
on: [push, pull_request]
jobs:
  web-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: cd web && npm ci && npm test
  
  mobile-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: cd mobile && npm ci && npm test
  
  hardware-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: pip install platformio && cd hardware && pio test -e native
```

---

## 📝 Changelog

### January 2026 (Current Session)

**Hardware Failures (EC-21 to EC-25):**
- ✅ EC-21: Solenoid stuck closed - 3x retry, feedback sensor, Firebase alerts
- ✅ EC-22: Solenoid stuck open - Out-of-service marking, blocks deliveries
- ✅ EC-23: Camera failure - 3x retry, metadata fallback, flagged review
- ✅ EC-25: ESP32 brownout - SPIFFS persistence, auto-resume, boot counter

**Scalability & Performance (EC-55, EC-56):**
- ✅ EC-55: Firebase quota exceeded - 80%/95% alerts, local caching, fetch interval reduction
- ✅ EC-56: Photo upload bandwidth - 800px/60% compression, priority queue (GPS > Status > Photo)

**Multi-Entity (EC-68):**
- ✅ EC-68: Residential vs business address - Address type field, dynamic geofence (50m/100m)

**New Files Created:**
- `hardware/lib/LockControl/LockControl.h` - Enhanced with feedback sensor
- `hardware/lib/PhotoCapture/PhotoCapture.h` - Retry and fallback system
- `hardware/lib/DeliveryState/DeliveryState.h` - SPIFFS persistence
- `web/src/components/HardwareStatusPanel.tsx` - Admin dashboard
- `web/src/components/HardwareAlertBanner.tsx` - Customer alerts
- `web/src/components/QuotaAlertBanner.tsx` - EC-55: Firebase quota alerts
- `mobile/src/components/HardwareAlertBanner.tsx` - Mobile alerts
- `mobile/src/components/HardwareStatusBadge.tsx` - Status badge
- `mobile/src/screens/rider/HardwareStatusScreen.tsx` - Detailed view
- `mobile/src/hooks/useHardwareStatus.ts` - React hook
- `mobile/src/services/hardwareStatusService.ts` - Business logic
- `mobile/src/services/quotaMonitorService.ts` - EC-55: Quota monitoring with caching
- `mobile/src/services/photoCompressionService.ts` - EC-56: Photo compression

**Tests Added:**
- `web/src/lib/__tests__/hardwareFailures.test.ts` - 25 tests
- `web/src/lib/__tests__/quotaMonitoring.test.ts` - 56 tests (EC-55)
- `web/src/lib/__tests__/photoUpload.test.ts` - 45 tests (EC-56)
- `web/src/lib/__tests__/addressTypeValidation.test.ts` - 42 tests (EC-68)
- `mobile/src/__tests__/hardwareStatus.test.ts` - 30 tests
- `mobile/src/__tests__/QuotaMonitoring.test.ts` - 52 tests (EC-55)
- `mobile/src/__tests__/AddressTypeGeofence.test.ts` - 48 tests (EC-68)
- `hardware/test/test_edge_cases.h` - Updated with EC-21 to EC-25

### Previous Implementations

- ✅ EC-01 to EC-07: Core offline handling
- ✅ EC-11, EC-12: Customer not home & address correction
- ✅ EC-15: Background location service
- ✅ EC-18: Tamper detection

---

## 🎯 TODO / Next Steps

### 🔴 High Priority
- [ ] EC-08: GPS spoofing detection
- [ ] EC-09: Web offline mode
- [ ] EC-16: OTP replay attack prevention
- [ ] EC-19: Biometric lock for rider app
- [ ] NC-PAY-*: Payment gateway failures

### 🟡 Medium Priority
- [ ] EC-29: OTP regeneration API
- [ ] EC-32: Cancellation after pickup flow
- [ ] SC-OTP-*: OTP state machine tests
- [ ] SC-GEO-*: Geofence state tests
- [ ] UC-A08 to UC-A22: Remaining admin features

### 🔵 Low Priority
- [ ] EC-50 to EC-53: UX improvements
- [ ] EC-26 to EC-28: Environmental handling
- [ ] Multi-language support
- [ ] Accessibility features
- [ ] Load testing (1000 concurrent)

---

## 📖 Legend

| Symbol | Meaning |
|--------|---------|
| ✅ | Implemented and passing |
| 🔶 | Partially implemented |
| ⬜ | TODO / Not implemented |
| - | Not applicable |

---

> **Maintainers:** Update this document when implementing new features or cases.  
> **Last Generated:** January 2026
