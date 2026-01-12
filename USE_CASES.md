# Use Cases

Comprehensive list of use cases for the Parcel-Safe Smart Top Box delivery system.

---

## 👤 Customer Use Cases (UC-C)

### UC-C01: View Tracking Page via Share Link
**Actor:** Customer
**Precondition:** Customer received tracking link via SMS/Email
**Flow:**
1. Customer clicks on tracking link
2. System validates share token
3. System displays real-time map with rider location
**Postcondition:** Customer sees live tracking with ETA

---

### UC-C02: View OTP Code When Rider Arrives
**Actor:** Customer
**Precondition:** Rider is within 50m geofence
**Flow:**
1. System detects rider within proximity
2. OTP card becomes visible on tracking page
3. Customer reads 6-digit OTP code
**Postcondition:** Customer has OTP ready for box unlock

---

### UC-C03: Receive Arrival Notification
**Actor:** Customer
**Precondition:** Valid push notification token registered
**Flow:**
1. Rider arrives at delivery location
2. System sends push notification "Your rider has arrived"
3. Customer receives notification on mobile device
**Postcondition:** Customer is alerted to meet rider

---

### UC-C04: View Proof of Delivery Photo
**Actor:** Customer
**Precondition:** Delivery completed with photo captured
**Flow:**
1. Customer opens tracking page after delivery
2. System displays delivery completion status
3. Customer clicks on proof photo thumbnail
4. Full-screen photo modal opens
**Postcondition:** Customer can verify who received package

---

### UC-C05: Track Package ETA
**Actor:** Customer
**Precondition:** Delivery in transit
**Flow:**
1. Customer opens tracking page
2. System calculates ETA based on current distance
3. ETA displayed with countdown timer
**Postcondition:** Customer knows approximate arrival time

---

### UC-C06: View Delivery Status Updates
**Actor:** Customer
**Precondition:** Active delivery exists
**Flow:**
1. Customer opens tracking page
2. System displays status timeline (Pending → Picked Up → In Transit → Arrived → Completed)
3. Current status highlighted with timestamp
**Postcondition:** Customer understands delivery progress

---

### UC-C07: Enter OTP on Keypad
**Actor:** Customer
**Precondition:** Box has valid OTP cached, Customer has OTP
**Flow:**
1. Customer approaches smart box
2. Customer enters 6-digit OTP on keypad
3. Box validates OTP locally
4. Box unlocks solenoid
**Postcondition:** Customer can retrieve package

---

### UC-C08: Request OTP Regeneration
**Actor:** Customer
**Precondition:** Original OTP compromised/shared
**Flow:**
1. Customer clicks "Generate New OTP" on tracking page
2. System invalidates old OTP
3. New OTP generated and synced to box
4. Customer sees new OTP code
**Postcondition:** New secure OTP available

---

### UC-C09: View Rider Details
**Actor:** Customer
**Precondition:** Delivery assigned to rider
**Flow:**
1. Customer opens tracking page
2. System displays rider name and photo
3. Customer can see rider rating
**Postcondition:** Customer knows who is delivering

---

### UC-C10: Access Tracking Without Login
**Actor:** Customer (Guest)
**Precondition:** Valid share token in URL
**Flow:**
1. Customer opens tracking link
2. System authenticates via token only
3. No login/signup required
**Postcondition:** Frictionless tracking experience

---

### UC-C11: View Package Description
**Actor:** Customer
**Precondition:** Package description provided at booking
**Flow:**
1. Customer opens tracking page
2. System displays package details (size, description)
**Postcondition:** Customer confirms correct package

---

### UC-C12: Contact Rider via Call
**Actor:** Customer
**Precondition:** Delivery in progress, rider phone available
**Flow:**
1. Customer clicks "Call Rider" button
2. Phone dialer opens with rider number
3. Customer initiates call
**Postcondition:** Direct communication established

---

### UC-C13: Rate Delivery Experience
**Actor:** Customer
**Precondition:** Delivery completed
**Flow:**
1. System prompts for rating after delivery
2. Customer selects 1-5 stars
3. Customer optionally adds comment
4. Rating submitted to backend
**Postcondition:** Feedback recorded for rider

---

### UC-C14: Report Delivery Issue
**Actor:** Customer
**Precondition:** Delivery completed or in progress
**Flow:**
1. Customer clicks "Report Issue" button
2. System displays issue categories (Damaged, Late, Wrong item, etc.)
3. Customer selects issue and describes problem
4. Report submitted to admin
**Postcondition:** Support ticket created

---

### UC-C15: Cancel Pending Delivery
**Actor:** Customer
**Precondition:** Delivery not yet picked up
**Flow:**
1. Customer clicks "Cancel Delivery" button
2. System confirms cancellation
3. Delivery marked as cancelled
4. Refund initiated if payment made
**Postcondition:** Delivery cancelled, resources freed

---

### UC-C16: Schedule Future Delivery
**Actor:** Customer
**Precondition:** Active delivery service in area
**Flow:**
1. Customer selects "Schedule Delivery" option
2. System displays available time slots
3. Customer selects preferred date and time window
4. System confirms scheduling
5. Delivery queued for selected time
**Postcondition:** Delivery scheduled for future execution

---

### UC-C17: Add Delivery Instructions
**Actor:** Customer
**Precondition:** Delivery booked or in progress
**Flow:**
1. Customer opens delivery details
2. Clicks "Add Instructions" button
3. Enters special instructions (gate code, landmarks, etc.)
4. Instructions saved and visible to rider
**Postcondition:** Rider has additional delivery guidance

---

### UC-C18: Enable Contactless Delivery Preference
**Actor:** Customer
**Precondition:** Account settings accessible
**Flow:**
1. Customer navigates to preferences
2. Toggles "Contactless Delivery" option
3. System saves preference
4. Future deliveries default to contactless
**Postcondition:** Contactless delivery is default for customer

---

### UC-C19: Share Tracking Link with Third Party
**Actor:** Customer
**Precondition:** Active delivery with tracking link
**Flow:**
1. Customer clicks "Share Tracking" button
2. System generates shareable link with limited permissions
3. Customer shares via SMS/email/messaging app
4. Third party can view basic tracking info
**Postcondition:** Third party has read-only tracking access

---

### UC-C20: View Delivery Statistics
**Actor:** Customer
**Precondition:** Completed deliveries exist
**Flow:**
1. Customer opens "My Statistics" section
2. System displays total deliveries, on-time rate, etc.
3. Customer can view monthly/yearly breakdown
**Postcondition:** Customer sees delivery history analytics

---

### UC-C21: Create Recurring Delivery
**Actor:** Customer
**Precondition:** Subscription feature enabled
**Flow:**
1. Customer selects "Set Up Recurring" on delivery
2. Chooses frequency (daily/weekly/monthly)
3. Sets start date and end date (optional)
4. System creates recurring schedule
**Postcondition:** Automatic deliveries scheduled

---

## 🛵 Rider Use Cases (UC-R)

### UC-R01: Login with Google Account
**Actor:** Rider
**Precondition:** Google account linked to profile
**Flow:**
1. Rider opens mobile app
2. Clicks "Sign in with Google" button
3. Native Google sign-in popup appears
4. Rider authenticates
5. Session token stored
**Postcondition:** Rider logged into app

---

### UC-R02: View Available Deliveries
**Actor:** Rider
**Precondition:** Logged in, location enabled
**Flow:**
1. Rider navigates to "Available Jobs" screen
2. System displays list of nearby pending deliveries
3. Each delivery shows pickup/dropoff, estimated pay
**Postcondition:** Rider can select delivery to accept

---

### UC-R03: Accept Delivery Assignment
**Actor:** Rider
**Precondition:** Viewing available deliveries
**Flow:**
1. Rider selects delivery from list
2. System displays full details (addresses, package info)
3. Rider clicks "Accept" button
4. Delivery assigned to rider
5. OTP synced to paired box
**Postcondition:** Delivery locked to rider

---

### UC-R04: View Active Delivery Details
**Actor:** Rider
**Precondition:** Has accepted delivery
**Flow:**
1. Rider navigates to "My Deliveries" screen
2. System displays active delivery with full details
3. Map shows route to destination
**Postcondition:** Rider has delivery information

---

### UC-R05: Navigate to Pickup Location
**Actor:** Rider
**Precondition:** Delivery accepted, pickup pending
**Flow:**
1. Rider clicks "Navigate to Pickup"
2. System opens preferred maps app
3. Turn-by-turn navigation begins
**Postcondition:** Rider can navigate to sender

---

### UC-R06: Mark Package as Picked Up
**Actor:** Rider
**Precondition:** At pickup location
**Flow:**
1. Rider collects package from sender
2. Rider opens app, clicks "Confirm Pickup"
3. System updates status to "IN_TRANSIT"
4. Photo captured as pickup proof
**Postcondition:** Delivery tracking begins

---

### UC-R07: Navigate to Dropoff Location
**Actor:** Rider
**Precondition:** Package picked up
**Flow:**
1. Rider clicks "Navigate to Dropoff"
2. System opens maps with customer address
3. Navigation begins
**Postcondition:** Rider heading to customer

---

### UC-R08: View Customer OTP (Proximity Gated)
**Actor:** Rider
**Precondition:** Within 50m of dropoff location
**Flow:**
1. System detects proximity
2. OTP becomes visible in rider app
3. Rider can verify customer's OTP entry
**Postcondition:** Rider can confirm correct recipient

---

### UC-R09: Mark Delivery as Completed
**Actor:** Rider
**Precondition:** Customer unlocked box successfully
**Flow:**
1. Box sends unlock confirmation to Firebase
2. App receives status update
3. Rider clicks "Confirm Delivery Complete"
4. System marks delivery as COMPLETED
**Postcondition:** Delivery finalized, payment triggered

---

### UC-R10: View Delivery History
**Actor:** Rider
**Precondition:** Completed past deliveries
**Flow:**
1. Rider navigates to "History" screen
2. System displays paginated list of completed deliveries
3. Each entry shows date, earnings, customer rating
**Postcondition:** Rider can review past work

---

### UC-R11: View Earnings Summary
**Actor:** Rider
**Precondition:** Completed paid deliveries
**Flow:**
1. Rider navigates to "Earnings" screen
2. System displays daily/weekly/monthly earnings
3. Breakdown by delivery shown
**Postcondition:** Rider knows total earnings

---

### UC-R12: Update Profile Information
**Actor:** Rider
**Precondition:** Logged in
**Flow:**
1. Rider navigates to "Profile" screen
2. Updates phone number, vehicle info, avatar
3. Clicks "Save"
4. Backend updates profile record
**Postcondition:** Profile information current

---

### UC-R13: Receive Tamper Alert
**Actor:** Rider
**Precondition:** Box tamper sensor triggered
**Flow:**
1. Box detects forced entry attempt
2. Immediate push notification to rider
3. App shows tamper warning banner
4. Photo captured of tampering
**Postcondition:** Rider alerted to investigate

---

### UC-R14: Receive Low Battery Warning
**Actor:** Rider
**Precondition:** Box battery below 20%
**Flow:**
1. Box sends low battery event to Firebase
2. App receives notification
3. Warning banner displayed
**Postcondition:** Rider knows to charge box

---

### UC-R15: Pair New Box Device
**Actor:** Rider
**Precondition:** New box hardware available
**Flow:**
1. Rider navigates to "Devices" screen
2. Clicks "Add New Box"
3. Scans QR code on box or enters MAC address
4. Box paired to rider account
**Postcondition:** Box linked for deliveries

---

### UC-R16: Report Box Hardware Issue
**Actor:** Rider
**Precondition:** Box malfunction detected
**Flow:**
1. Rider clicks "Report Issue" on device screen
2. Selects issue type (Camera, Lock, GPS, etc.)
3. Describes problem
4. Report sent to maintenance team
**Postcondition:** Support ticket created

---

### UC-R17: Go Online/Offline
**Actor:** Rider
**Precondition:** Logged in
**Flow:**
1. Rider toggles availability switch
2. Status updated to ONLINE or OFFLINE
3. When OFFLINE, no new deliveries assigned
**Postcondition:** Rider availability reflected

---

### UC-R18: Cancel Accepted Delivery
**Actor:** Rider
**Precondition:** Delivery accepted, not yet picked up
**Flow:**
1. Rider clicks "Cancel Delivery"
2. Selects cancellation reason
3. Delivery returned to available pool
4. Cancellation logged
**Postcondition:** Delivery unassigned, metrics updated

---

### UC-R19: Upload Vehicle Documents
**Actor:** Rider
**Precondition:** Onboarding or renewal required
**Flow:**
1. Rider navigates to "Documents" screen
2. Uploads driver's license, vehicle registration
3. Documents sent for verification
**Postcondition:** Documents pending admin review

---

### UC-R20: Collect Cash on Delivery (COD)
**Actor:** Rider
**Precondition:** Delivery marked as COD
**Flow:**
1. Rider collects cash from customer
2. Enters amount received in app
3. Customer confirms amount
4. Payment logged
**Postcondition:** COD transaction recorded

---

### UC-R21: Start/End Shift Clock-in
**Actor:** Rider
**Precondition:** Logged in, verified account
**Flow:**
1. Rider taps "Start Shift" button
2. System records shift start time and location
3. Rider is marked available for deliveries
4. At end, rider taps "End Shift"
5. System calculates shift hours and earnings
**Postcondition:** Shift hours tracked for payroll

---

### UC-R22: View Route Optimization
**Actor:** Rider
**Precondition:** Multiple deliveries assigned
**Flow:**
1. Rider opens "My Route" screen
2. System calculates optimal delivery order
3. Map displays optimized route with waypoints
4. Rider can manually reorder if needed
**Postcondition:** Efficient multi-stop route displayed

---

### UC-R23: Request Emergency Assistance
**Actor:** Rider
**Precondition:** Active delivery or shift
**Flow:**
1. Rider presses SOS/Emergency button
2. System captures GPS location
3. Alert sent to support team
4. Optional: Auto-call to emergency contacts
**Postcondition:** Support team alerted with rider location

---

### UC-R24: Handle Priority/Express Delivery
**Actor:** Rider
**Precondition:** Express delivery assigned
**Flow:**
1. System assigns express delivery with priority flag
2. Rider receives urgent notification
3. Express indicator shown throughout delivery
4. Bonus payment applied upon completion
**Postcondition:** Express delivery completed with bonus

---

### UC-R25: Swap Delivery with Another Rider
**Actor:** Rider
**Precondition:** Delivery assigned, another rider available
**Flow:**
1. Rider selects "Request Swap" on delivery
2. System notifies nearby available riders
3. Another rider accepts swap request
4. Delivery transferred with OTP re-sync
**Postcondition:** Delivery reassigned, both riders notified

---

### UC-R26: Report Weather/Traffic Delay
**Actor:** Rider
**Precondition:** Active delivery in progress
**Flow:**
1. Rider taps "Report Delay" button
2. Selects reason (traffic, weather, accident, etc.)
3. System updates ETA and notifies customer
4. Delay logged for analytics
**Postcondition:** Customer informed of delay reason

---

### UC-R27: View Daily Leaderboard
**Actor:** Rider
**Precondition:** Logged in
**Flow:**
1. Rider navigates to "Leaderboard" screen
2. System displays top riders by deliveries/rating
3. Rider sees their current rank
4. Achievement badges displayed
**Postcondition:** Rider motivated by gamification

---

## 🔧 Admin Use Cases (UC-A)

### UC-A01: Login to Admin Portal
**Actor:** Admin
**Precondition:** Admin credentials exist
**Flow:**
1. Admin navigates to admin portal URL
2. Enters email and password
3. System validates credentials
4. Admin dashboard displayed
**Postcondition:** Admin has access to management tools

---

### UC-A02: View All Active Deliveries
**Actor:** Admin
**Precondition:** Logged into admin portal
**Flow:**
1. Admin navigates to "Deliveries" dashboard
2. System displays map with all active riders
3. List view shows delivery statuses
**Postcondition:** Admin has operational overview

---

### UC-A03: View Rider Performance Metrics
**Actor:** Admin
**Precondition:** Riders have completed deliveries
**Flow:**
1. Admin navigates to "Riders" section
2. Selects specific rider
3. System displays completed count, rating, on-time %
**Postcondition:** Admin can evaluate rider

---

### UC-A04: Approve Rider Application
**Actor:** Admin
**Precondition:** Rider submitted documents
**Flow:**
1. Admin views pending applications
2. Reviews uploaded documents
3. Clicks "Approve" or "Reject"
4. Rider notified of decision
**Postcondition:** Rider status updated

---

### UC-A05: Remote Box Unlock Override
**Actor:** Admin
**Precondition:** Emergency unlock required
**Flow:**
1. Admin selects box from device list
2. Clicks "Emergency Unlock"
3. Confirmation dialog appears
4. Override command sent to box
5. Solenoid unlocks
**Postcondition:** Box unlocked remotely

---

### UC-A06: Reset OTP Lockout
**Actor:** Admin
**Precondition:** Customer locked out after 5 attempts
**Flow:**
1. Admin receives lockout alert
2. Navigates to delivery details
3. Clicks "Reset Lockout"
4. Lockout counter cleared
**Postcondition:** Customer can try OTP again

---

### UC-A07: View Tamper Alerts
**Actor:** Admin
**Precondition:** Tamper events logged
**Flow:**
1. Admin navigates to "Security Alerts"
2. System displays list of tamper events
3. Each event shows box, time, photo
**Postcondition:** Admin can investigate incidents

---

### UC-A08: Decommission Box
**Actor:** Admin
**Precondition:** Box retired or damaged
**Flow:**
1. Admin selects box from device list
2. Clicks "Decommission"
3. Confirms action
4. Box unlinked from rider, data archived
**Postcondition:** Box removed from active fleet

---

### UC-A09: Push Firmware Update
**Actor:** Admin
**Precondition:** New firmware version available
**Flow:**
1. Admin navigates to "Firmware Management"
2. Uploads new firmware binary
3. Selects target boxes (all or subset)
4. Initiates OTA update
**Postcondition:** Boxes receive firmware update

---

### UC-A10: Generate Delivery Reports
**Actor:** Admin
**Precondition:** Delivery data exists
**Flow:**
1. Admin navigates to "Reports" section
2. Selects date range and filters
3. Clicks "Generate Report"
4. System produces CSV/PDF export
**Postcondition:** Report available for download

---

### UC-A11: Configure System Parameters
**Actor:** Admin
**Precondition:** Logged into admin portal
**Flow:**
1. Admin navigates to "Settings"
2. Adjusts parameters (geofence radius, OTP expiry, etc.)
3. Clicks "Save"
4. Settings applied system-wide
**Postcondition:** Configuration updated

---

### UC-A12: Resolve Delivery Dispute
**Actor:** Admin
**Precondition:** Customer reported issue
**Flow:**
1. Admin views dispute ticket
2. Reviews GPS trail, photos, OTP logs
3. Makes resolution decision
4. Updates ticket status, issues refund if needed
**Postcondition:** Dispute resolved

---

### UC-A13: Suspend Rider Account
**Actor:** Admin
**Precondition:** Rider violated policy
**Flow:**
1. Admin selects rider profile
2. Clicks "Suspend Account"
3. Enters suspension reason
4. Rider account disabled
**Postcondition:** Rider cannot login or accept deliveries

---

### UC-A14: View System Health Dashboard
**Actor:** Admin
**Precondition:** Logged into admin portal
**Flow:**
1. Admin navigates to "System Health"
2. System displays Firebase status, API latency, device health
3. Alerts shown for any issues
**Postcondition:** Admin monitors system status

---

### UC-A15: Export Audit Logs
**Actor:** Admin
**Precondition:** Audit logging enabled
**Flow:**
1. Admin navigates to "Audit Logs"
2. Filters by user, action, date
3. Clicks "Export"
4. CSV downloaded
**Postcondition:** Audit trail available for review

---

### UC-A16: Manage Delivery Zones
**Actor:** Admin
**Precondition:** Logged into admin portal
**Flow:**
1. Admin navigates to "Zone Management"
2. Views map with current service boundaries
3. Creates/edits polygon zones
4. Assigns pricing and availability per zone
**Postcondition:** Service areas configured

---

### UC-A17: Configure Peak Hour Pricing
**Actor:** Admin
**Precondition:** Pricing module enabled
**Flow:**
1. Admin navigates to "Surge Pricing"
2. Defines time-based pricing rules
3. Sets multipliers for peak hours
4. Rules activate automatically
**Postcondition:** Dynamic pricing active during peak times

---

### UC-A18: Schedule Box Maintenance
**Actor:** Admin
**Precondition:** Box fleet registered
**Flow:**
1. Admin selects box(es) for maintenance
2. Sets maintenance date and type
3. Box marked unavailable during window
4. Technician notified
**Postcondition:** Maintenance scheduled, box offline

---

### UC-A19: Create Promotional Campaign
**Actor:** Admin
**Precondition:** Marketing module enabled
**Flow:**
1. Admin navigates to "Promotions"
2. Creates new campaign with discount rules
3. Sets validity period and target audience
4. Campaign goes live
**Postcondition:** Promo codes/discounts active

---

### UC-A20: View Real-Time Fleet Analytics
**Actor:** Admin
**Precondition:** Active fleet operations
**Flow:**
1. Admin opens "Fleet Dashboard"
2. Views live map with all active riders/boxes
3. Sees utilization, delivery counts, delays
4. Drill-down into individual metrics
**Postcondition:** Operational insight obtained

---

### UC-A21: Manage API Keys
**Actor:** Admin
**Precondition:** Integration module enabled
**Flow:**
1. Admin navigates to "API Management"
2. Creates/revokes API keys for partners
3. Sets rate limits and permissions
4. Monitors API usage
**Postcondition:** Third-party integrations controlled

---

### UC-A22: Bulk Import Deliveries
**Actor:** Admin
**Precondition:** CSV template available
**Flow:**
1. Admin navigates to "Import Deliveries"
2. Uploads CSV file with delivery data
3. System validates and previews entries
4. Admin confirms import
5. Deliveries created in batch
**Postcondition:** Multiple deliveries created at once

---

## 📦 Box Use Cases (UC-B)

### UC-B01: Initialize on Power On
**Actor:** Box (ESP32)
**Precondition:** Power connected
**Flow:**
1. Box boots up
2. Loads configuration from SPIFFS
3. Connects to WiFi
4. Syncs with Firebase
5. Status LED turns green
**Postcondition:** Box ready for operation

---

### UC-B02: Receive OTP from Firebase
**Actor:** Box
**Precondition:** Delivery assigned
**Flow:**
1. Firebase listener receives new OTP hash
2. Box caches OTP locally
3. Keypad enabled for input
**Postcondition:** Box ready to validate OTP

---

### UC-B03: Validate OTP Locally
**Actor:** Box
**Precondition:** OTP cached, keypad entry received
**Flow:**
1. User enters 6-digit code on keypad
2. Box hashes input
3. Compares with cached hash
4. Match confirmed
**Postcondition:** Unlock sequence initiated

---

### UC-B04: Capture Proof of Delivery Photo
**Actor:** Box
**Precondition:** Valid OTP entered
**Flow:**
1. Box triggers camera
2. Image captured to buffer
3. Photo saved to SPIFFS
4. Upload queued
**Postcondition:** Photo recorded before unlock

---

### UC-B05: Unlock Solenoid
**Actor:** Box
**Precondition:** Photo captured successfully
**Flow:**
1. Box activates solenoid relay
2. Lock mechanism retracts
3. Timer starts (5s max)
4. LED turns blue
**Postcondition:** Lid can be opened

---

### UC-B06: Report Lock State to Firebase
**Actor:** Box
**Precondition:** Lock state changed
**Flow:**
1. Box updates /boxes/{id}/lock/is_locked
2. Firebase receives new state
3. Web/Mobile apps reflect change
**Postcondition:** Lock state synchronized

---

### UC-B07: Upload Photo When Online
**Actor:** Box
**Precondition:** Photo in queue, WiFi available
**Flow:**
1. Box checks upload queue
2. Reads photo from SPIFFS
3. Uploads to Firebase Storage
4. On success, removes from queue
**Postcondition:** Photo in cloud storage

---

### UC-B08: Queue Photo When Offline
**Actor:** Box
**Precondition:** No WiFi connection
**Flow:**
1. Photo capture completes
2. Box attempts upload, fails
3. Photo added to SPIFFS queue
4. Retry counter initialized
**Postcondition:** Photo preserved for later upload

---

### UC-B09: Report GPS Location
**Actor:** Box
**Precondition:** GPS module functioning
**Flow:**
1. Box reads GPS coordinates every N seconds
2. Validates coordinates (within bounds)
3. Updates /boxes/{id}/location
**Postcondition:** Live location available

---

### UC-B10: Report Battery Level
**Actor:** Box
**Precondition:** Battery monitoring circuit active
**Flow:**
1. Box reads ADC voltage
2. Calculates percentage
3. Updates /boxes/{id}/battery
4. Triggers alert if low
**Postcondition:** Battery status visible

---

### UC-B11: Detect Tamper Event
**Actor:** Box
**Precondition:** Reed switch on door
**Flow:**
1. Door opened without OTP
2. Interrupt triggers
3. Camera captures photo
4. "TAMPERED" event sent to Firebase
5. Box enters lockdown mode
**Postcondition:** Tamper recorded, alerts sent

---

### UC-B12: Handle Failed OTP Attempt
**Actor:** Box
**Precondition:** Incorrect OTP entered
**Flow:**
1. Box receives keypad input
2. Hash comparison fails
3. Attempt counter incremented
4. LED flashes red
5. If 5 failures, lockout initiated
**Postcondition:** Failed attempt logged

---

### UC-B13: Reconnect After WiFi Loss
**Actor:** Box
**Precondition:** WiFi connection lost
**Flow:**
1. Box detects disconnect
2. LED turns blinking blue
3. Exponential backoff reconnection
4. On success, resume Firebase sync
**Postcondition:** Connection restored

---

### UC-B14: Resume State After Reboot
**Actor:** Box
**Precondition:** Power restored after outage
**Flow:**
1. Box boots
2. Reads state from SPIFFS
3. Loads cached OTP and delivery ID
4. Checks for pending uploads
5. Resumes operation
**Postcondition:** No data lost from reboot

---

### UC-B15: Receive Firmware Update (OTA)
**Actor:** Box
**Precondition:** Admin pushed update
**Flow:**
1. Box receives OTA notification
2. Downloads firmware binary
3. Verifies checksum
4. Flashes new firmware
5. Reboots
**Postcondition:** Running updated firmware

---

### UC-B16: Handle Multiple OTPs (Batch Mode)
**Actor:** Box
**Precondition:** Multiple deliveries assigned to same box
**Flow:**
1. Firebase syncs multiple OTP hashes
2. Box caches all OTPs with delivery IDs
3. Any valid OTP triggers unlock
4. Consumed OTP removed from cache
**Postcondition:** Box can serve multiple deliveries

---

### UC-B17: Run Self-Diagnostics
**Actor:** Box
**Precondition:** Scheduled or triggered by admin
**Flow:**
1. Box initiates health check sequence
2. Tests camera, GPS, solenoid, sensors
3. Checks storage, memory, battery
4. Reports results to Firebase
**Postcondition:** Health status updated in dashboard

---

### UC-B18: Adaptive GPS Reporting
**Actor:** Box
**Precondition:** GPS module active
**Flow:**
1. Box detects movement speed
2. If stationary: report every 60s
3. If moving slowly: report every 10s
4. If moving fast: report every 3s
**Postcondition:** Optimized GPS update frequency

---

### UC-B19: Enter Power Saving Mode
**Actor:** Box
**Precondition:** Battery < 15% or idle > 30 minutes
**Flow:**
1. Box detects low battery or extended idle
2. Reduces GPS frequency to every 5 minutes
3. Disables camera preview
4. LED dimmed
5. Core unlock function remains active
**Postcondition:** Extended battery life in degraded mode

---

### UC-B20: Handle Emergency SOS
**Actor:** Box
**Precondition:** SOS button installed (optional hardware)
**Flow:**
1. Rider presses physical SOS button on box
2. Box captures photo and GPS
3. Emergency alert sent to Firebase
4. Admin and emergency contacts notified
**Postcondition:** Emergency response triggered

---

## 🌐 Web Portal Use Cases (UC-W)

### UC-W01: Display Landing Page
**Actor:** Visitor
**Precondition:** User navigates to main URL
**Flow:**
1. Server renders landing page
2. Hero section, features displayed
3. CTA buttons visible
**Postcondition:** User understands product

---

### UC-W02: Validate Share Token
**Actor:** System
**Precondition:** Token in URL path
**Flow:**
1. Server receives /track/[token] request
2. Queries database for token
3. Returns delivery data if valid
4. Returns 404 if expired/invalid
**Postcondition:** Token validated for access

---

### UC-W03: Render Map with Live Location
**Actor:** Web App
**Precondition:** Valid token, rider in transit
**Flow:**
1. Client subscribes to Firebase GPS
2. Map component initialized
3. Rider marker placed
4. Marker animates on updates
**Postcondition:** Real-time tracking visible

---

### UC-W04: Calculate Haversine Distance
**Actor:** Web App
**Precondition:** Rider and dropoff coordinates available
**Flow:**
1. Client receives GPS update
2. Calculates distance using Haversine formula
3. Updates ETA display
4. Triggers OTP reveal if <50m
**Postcondition:** Distance-based features work

---

### UC-W05: Reveal OTP on Proximity
**Actor:** Web App
**Precondition:** Distance < 50m
**Flow:**
1. Distance calculation triggers threshold
2. OTP card state changes to visible
3. 6-digit code rendered
4. UI theme changes to "Actionable"
**Postcondition:** Customer can read OTP

---

### UC-W06: Display Tamper Warning Banner
**Actor:** Web App
**Precondition:** Tamper event received
**Flow:**
1. Firebase delivers tamper status
2. Warning banner component rendered
3. Security alert message displayed
**Postcondition:** Customer aware of security issue

---

### UC-W07: Show Last Seen Timestamp (Offline)
**Actor:** Web App
**Precondition:** GPS updates stopped
**Flow:**
1. Client detects stale data (>60s old)
2. "Last seen X minutes ago" displayed
3. Offline indicator shown
**Postcondition:** User not confused by stale data

---

### UC-W08: Display Delivery Photo Modal
**Actor:** Web App
**Precondition:** Delivery completed, photo exists
**Flow:**
1. User clicks photo thumbnail
2. Full-screen modal opens
3. Photo zooms to fit
4. Close button or backdrop click closes
**Postcondition:** User viewed proof photo

---

### UC-W09: Handle Expired Token
**Actor:** Web App
**Precondition:** Token past expiration date
**Flow:**
1. Server validates token timestamp
2. Expiration detected
3. "Link Expired" page rendered
4. Contact support CTA shown
**Postcondition:** User guided to resolution

---

### UC-W10: Admin Portal Authentication
**Actor:** Admin
**Precondition:** Valid admin credentials
**Flow:**
1. Admin enters credentials
2. Server validates against auth provider
3. JWT issued
4. Dashboard rendered
**Postcondition:** Admin session active

---

## 📱 Mobile-Specific Use Cases (UC-M)

### UC-M01: Display Splash Screen
**Actor:** App
**Precondition:** App launched
**Flow:**
1. Splash screen rendered
2. Session token validated
3. Navigation to appropriate screen
**Postcondition:** Smooth app launch experience

---

### UC-M02: Handle Background Location Updates
**Actor:** App
**Precondition:** Location permissions granted, foreground service running
**Flow:**
1. OS sends location updates
2. App processes in background
3. GPS pushed to Firebase
**Postcondition:** Continuous tracking even when app minimized

---

### UC-M03: Display Offline Mode Banner
**Actor:** App
**Precondition:** No network connection
**Flow:**
1. App detects offline
2. Banner displayed: "Offline - Using cached data"
3. Core functions remain available
**Postcondition:** User knows connectivity status

---

### UC-M04: Cache Delivery Locally
**Actor:** App
**Precondition:** Active delivery assigned
**Flow:**
1. Delivery data received
2. Saved to AsyncStorage/MMKV
3. Accessible without network
**Postcondition:** Offline access to delivery info

---

### UC-M05: Handle Push Notification
**Actor:** App
**Precondition:** FCM token registered
**Flow:**
1. FCM delivers notification payload
2. Custom notification sound plays
3. Notification displayed in system tray
4. Tap opens relevant screen
**Postcondition:** User notified of event

---

### UC-M06: Request Location Permissions
**Actor:** App
**Precondition:** First launch or permissions revoked
**Flow:**
1. Permission dialog shown
2. User grants/denies
3. App adjusts features accordingly
4. Background location requested if foreground granted
**Postcondition:** Permissions configured

---

### UC-M07: Logout and Clear Session
**Actor:** Rider
**Precondition:** Logged in
**Flow:**
1. User taps "Logout"
2. Confirmation dialog shown
3. Session tokens cleared
4. Local cache purged
5. Navigate to login screen
**Postcondition:** User logged out securely

---

### UC-M08: Deep Link to Delivery
**Actor:** App
**Precondition:** Push notification contains delivery ID
**Flow:**
1. User taps notification
2. App parses deep link
3. Navigates directly to delivery details
**Postcondition:** Contextual navigation

---

### UC-M09: Sync Pending Updates on Reconnect
**Actor:** App
**Precondition:** Updates queued while offline
**Flow:**
1. Network connectivity restored
2. Queue processor activates
3. Pending updates pushed to backend
4. Queue cleared on success
**Postcondition:** Data synchronized

---

### UC-M10: Display Error Toast
**Actor:** App
**Precondition:** API error occurred
**Flow:**
1. Error caught by handler
2. User-friendly message composed
3. Toast displayed with message
4. Auto-dismiss after 4 seconds
**Postcondition:** User informed of issue

---

## 🔌 Integration Use Cases (UC-I)

### UC-I01: Sync with E-commerce Platform
**Actor:** System
**Precondition:** E-commerce integration configured
**Flow:**
1. E-commerce platform sends order webhook
2. System validates and parses order data
3. Delivery created automatically
4. Status updates sent back to platform
**Postcondition:** Seamless order-to-delivery flow

---

### UC-I02: Export to Insurance System
**Actor:** System
**Precondition:** Insurance integration enabled
**Flow:**
1. Delivery completed with incidents
2. System compiles delivery audit trail
3. Data exported to insurance partner API
4. Claim reference returned
**Postcondition:** Insurance records synchronized

---

### UC-I03: Connect Fleet Management System
**Actor:** System
**Precondition:** Fleet management integration active
**Flow:**
1. Box location updates streamed
2. Fleet system receives real-time telemetry
3. Maintenance alerts forwarded
4. Utilization reports generated
**Postcondition:** Centralized fleet visibility

---

### UC-I04: Webhook Delivery Notifications
**Actor:** System
**Precondition:** Webhook endpoint configured by partner
**Flow:**
1. Delivery status changes
2. System constructs webhook payload
3. HTTP POST sent to partner endpoint
4. Retry on failure (3 attempts)
**Postcondition:** Partner systems notified in real-time

---

## Summary

| Category | Count |
|----------|-------|
| 👤 Customer Use Cases | 21 |
| 🛵 Rider Use Cases | 27 |
| 🔧 Admin Use Cases | 22 |
| 📦 Box Use Cases | 20 |
| 🌐 Web Portal Use Cases | 10 |
| 📱 Mobile-Specific Use Cases | 10 |
| 🔌 Integration Use Cases | 4 |
| **Total** | **114** |

---

## Testing Priority

| Priority | Use Cases | Rationale |
|----------|-----------|-----------|
| P0 (Critical) | UC-C07, UC-B03, UC-B04, UC-B05, UC-B16 | Core unlock flow |
| P1 (High) | UC-R03, UC-R09, UC-B07, UC-B08, UC-R23, UC-B20 | Delivery lifecycle & safety |
| P2 (Medium) | UC-A05, UC-A06, UC-B11, UC-B17, UC-I01, UC-I04 | Security & integration |
| P3 (Low) | UC-C13, UC-R11, UC-M01, UC-C20, UC-R27, UC-A19 | Nice-to-have features |
