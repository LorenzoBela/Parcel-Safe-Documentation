# Phase Changes Detailed Report

Date: 2026-04-03
Scope: End-to-end reliability hardening, offline/resume recovery, notification and navigation resilience, selective read-layer modernization, backend idempotency safeguards, plus security/auth/session hardening and UX quality-of-life improvements.

## Executive Summary

This implementation moved from planning to actual code changes across six phases:

1. Phase 1: Reliability Foundation (Network Policy + Queue Identity)
2. Phase 2: Foreground Resume Reconciliation and Notification Dedup
3. Phase 3: Navigation Recovery and Query Persistence Modernization
4. Phase 4: Security Foundation and Session Hardening
5. Phase 5: Critical Action UX and Optional UX Integrations
6. Phase 6: Backend Idempotency and Final Validation

The result is a more resilient app under poor/no network conditions, safer replay behavior for critical writes, stronger auth/session security, better notification and deep-link recovery, and improved UX in high-traffic screens.

---

## Phase 1: Reliability Foundation (Network Policy + Queue Identity)

### 1. Shared network policy introduced

New file:

- mobile/src/services/networkPolicy.ts

Implemented:

- Central timeout values for Firebase writes and resume pipeline stages
- Shared retry/backoff constants and helper methods
- Optional jitter helper for retry spreading

Benefits:

- Removes duplicated timeout/retry constants across services
- Makes retry behavior consistent and easier to tune

### 2. Queue identity helper introduced

New file:

- mobile/src/services/queueIdentity.ts

Implemented:

- Stable queue UUID generator with safe prefixes

Benefits:

- Enables end-to-end traceability of queued operations
- Supports observability tags and idempotency diagnostics

### 3. Status queue hardening

Updated file:

- mobile/src/services/statusUpdateService.ts

Implemented:

- Queue entries now include queueId
- Retry delay now uses shared network policy helpers
- Added retry queue metadata field (status_retry_queue_id) on writes

Benefits:

- Better replay diagnostics for EC35 status recovery
- Cleaner, centrally managed retry strategy

### 4. Offline location queue hardening

Updated file:

- mobile/src/services/offlineQueueService.ts

Implemented:

- Location queue entries now include queueId
- Shared Firebase timeout policy adopted
- Ordered online-direct vs buffered flush behavior retained and instrumented

Benefits:

- More deterministic offline-to-online drain behavior
- Better debuggability for location write ordering

### 5. Delivery sync policy alignment

Updated file:

- mobile/src/services/deliverySyncService.ts

Implemented:

- Retry attempts and timeout sourced from shared policy
- Backoff constants unified with other services

Benefits:

- Sync behavior now aligned with app-wide reliability policy

---

## Phase 2: Foreground Resume Reconciliation and Notification Dedup

### 1. Resume pipeline expanded to staged reconciliation

Updated file:

- mobile/src/services/foregroundResumePipelineService.ts

Implemented:

- Ordered stages: gps warmup, auth refresh, status flush, box-command flush, location flush, delivery sync, listener probe
- Timeboxed stage execution via shared timeout values

Benefits:

- Faster, safer recovery after long background and weak-network periods
- Prioritizes state consistency before user-facing interactions

### 2. Reconnect-triggered reconciliation from app lifecycle

Updated file:

- mobile/App.js

Implemented:

- Connectivity listener triggers resume reconciliation when app is active and network returns

Benefits:

- Reduces stale UI/data after offline windows

### 3. Order listener health probe for rider flow

Updated file:

- mobile/src/services/orderListenerService.ts

Implemented:

- ensureOrderListenerHealthy helper to reattach listener if missing

Benefits:

- Prevents silent listener drop after long sleep/background

### 4. Notification dedup service and integrations

New file:

- mobile/src/services/notificationDedupService.ts

Updated files:

- mobile/src/services/pushNotificationService.ts
- mobile/src/services/backgroundServiceManager.ts

Implemented:

- Persistent TTL-based dedup map for processed notifications
- Dedup checks in foreground/background FCM paths

Benefits:

- Prevents duplicate user alerts and duplicate handler side effects

---

## Phase 3: Navigation Recovery and Query Persistence Modernization

### 1. Navigation readiness and cold-start recovery

New file:

- mobile/src/navigation/navigationService.ts

Updated files:

- mobile/src/navigation/AppNavigator.tsx
- mobile/App.js

Implemented:

- Global nav ref with pending navigation queue
- Linking routes configured for deep links
- Navigation state persistence/restore via AsyncStorage
- Last notification response read on cold start and routed

Benefits:

- Reliable navigation from notification taps even before navigator is ready
- Better app continuity across restarts and process kills

### 2. TanStack query foundation and persistence

New file:

- mobile/src/services/queryClient.ts

Updated files:

- mobile/App.js
- mobile/package.json

Implemented:

- QueryClient + AsyncStorage persister setup
- PersistQueryClientProvider integrated at app root

Benefits:

- Read cache survives app restarts and reconnects
- Reduces unnecessary refetch churn in poor-network conditions

### 3. Read-path migrations (partial strategy)

Updated files:

- mobile/src/screens/client/DeliveryLogScreen.tsx
- mobile/src/screens/rider/AssignedDeliveriesScreen.tsx
- mobile/src/screens/admin/AdminRecordsScreen.tsx

Implemented:

- DeliveryLog migrated to useQuery
- AssignedDeliveries migrated to useQuery with refetch-based refresh
- AdminRecords migrated to useInfiniteQuery pagination

Benefits:

- More stable read behavior with cache/refetch semantics
- Lower risk migration by targeting read-heavy screens first

---

## Phase 4: Security Foundation and Session Hardening

### 1. Core packages installed

Added to mobile dependencies:

- expo-secure-store
- expo-device
- expo-network
- @sentry/react-native

Files:

- mobile/package.json
- mobile/app.json

What this enables:

- Secure key/value storage for sensitive auth secrets
- Device/network risk context collection for session security
- Crash telemetry bootstrap path for production diagnostics

### 2. SecureStore guard layer

New file:

- mobile/src/services/security/secureStoreService.ts

Implemented:

- Size guard for secure entries (small-secret policy)
- Dedicated invalidation error type for biometric key invalidation cases
- Helper methods for read/write/remove with normalized error handling

Benefits:

- Prevents accidental large payload writes to secure storage
- Makes biometric invalidation edge cases deterministic and testable

### 3. Auth secret policy service

New file:

- mobile/src/services/security/authSecretStore.ts

Implemented:

- Persist and clear only sensitive secrets (access token, refresh token)
- Biometric-protected storage for hashed fallback PIN
- Validation routine that detects key invalidation and signals hard re-login

Benefits:

- Enforces small-secret-only policy in one place
- Protects fallback PIN hash behind device biometric authentication

### 4. Supabase auth lifecycle integration

Updated file:

- mobile/src/services/supabaseClient.ts

Implemented:

- On SIGNED_IN and TOKEN_REFRESHED: mirror auth tokens into secure storage
- On SIGNED_OUT: clear secure auth secrets

Benefits:

- Keeps secure secret store synchronized with real auth session lifecycle
- Reduces stale secret risk after sign-out

### 5. Sentry bootstrap (timeboxed setup)

New file:

- mobile/src/services/observability/sentryService.ts

Updated file:

- mobile/App.js

Implemented:

- DSN-gated Sentry init (safe no-op when DSN is missing)
- Lightweight initialization without deep CI/source-map scope creep

Benefits:

- Immediate crash visibility path for production builds
- Stays aligned with thesis time constraints

---

## Phase 5: Biometric, Notification Actions, and Optional UX Integrations

### 1. Biometric invalidation hard re-login enforcement

Updated file:

- mobile/src/screens/auth/AuthLoadingScreen.tsx

Implemented:

- On app auth restore, run biometric-bound key validation
- If invalidated key is detected, force hard re-login (sign out + go to Login)

Security rationale:

- Handles OS-level biometric changes (for example, new fingerprint added)
- Prevents continued access when biometric trust chain is invalidated

### 2. Device ID secure persistence

Updated file:

- mobile/src/services/sessionService.ts

Implemented:

- Device ID retrieval now prefers secure storage
- Legacy fallback to AsyncStorage retained for backward compatibility
- Migration path writes legacy ID into secure storage

Benefits:

- Hardens session identity persistence
- Avoids breaking existing installs

### 3. Device and network risk context collection

New file:

- mobile/src/services/security/deviceRiskService.ts

Implemented:

- Collects root/jailbreak signal
- Collects connection type, internet reachability, airplane mode
- Includes key device context fields

Benefits:

- Builds risk-aware session telemetry foundation
- Supports later security analytics and step-up auth decisions

### 4. Session registration now carries device risk snapshot

Updated file:

- mobile/src/services/sessionService.ts

Implemented:

- registerSession now captures and stores deviceRisk alongside session metadata

Benefits:

- Every rider session registration can now include risk context

### 5. Rider session registration wired into role flow

Updated file:

- mobile/src/screens/auth/RoleSelectionScreen.tsx

Implemented:

- Before entering Rider flow, app registers rider session with platform/app version metadata

Benefits:

- Session policy is activated by actual UI navigation path, not just as a standalone service

### 6. Interactive security notification actions

Updated files:

- mobile/src/services/pushNotificationService.ts
- mobile/App.js

Implemented:

- Security action category registration (Re-authenticate / Dismiss)
- App-level notification action handling for REAUTH_NOW
- REAUTH_NOW action forces sign-out and auth reset path

Benefits:

- Security alerts become actionable, not just informative
- Fast path to force user re-authentication from notification action

### 7. Notification role filtering hardening

Updated file:

- mobile/src/screens/common/NotificationListScreen.tsx

Implemented:

- Unknown roles are denied in client-side visibility checks
- Explicit role narrowing for allowed role list checks

Benefits:

- Reduces accidental notification exposure due to malformed/unknown role states

### 8. Notification category typing hardening

Updated file:

- mobile/src/services/notificationService.ts

Implemented:

- Strong typing for normalized notification category values

Benefits:

- Less category drift and safer downstream filtering behavior

### 9. Critical action policy for hardware commands

New file:

- mobile/src/services/actionCriticality.ts

Updated file:

- mobile/src/screens/rider/BoxControlsScreen.tsx

Implemented:

- Action criticality classifier (critical hardware, critical state, non-critical metadata)
- Critical hardware actions now log explicit pending/ack behavior path

Benefits:

- Reduces risky optimistic UX for critical lock/unlock flows
- Improves operator clarity during command-in-flight windows

---

## Phase 6: Backend Idempotency and Final Validation

### 1. Cancel-booking endpoint replay safety

Updated file:

- web/src/app/api/cancel-booking/route.ts

Implemented:

- requestId accepted and persisted
- Existing cancellation short-circuit for duplicate requests

Benefits:

- Duplicate retries no longer create duplicate cancellation effects

### 2. No-show endpoint replay safety

Updated file:

- web/src/app/api/deliveries/[id]/no-show/route.ts

Implemented:

- Returns idempotent success if already canceled for CUSTOMER_NO_SHOW

Benefits:

- Retries become safe and client behavior is simplified

### 3. Reliability observability tags for queue lifecycle

Updated files:

- mobile/src/services/statusUpdateService.ts
- mobile/src/services/offlineQueueService.ts
- mobile/src/services/boxCommandQueueService.ts
- mobile/src/services/foregroundResumePipelineService.ts

Implemented:

- Structured telemetry tags: queue_uuid, action_type, flush_stage, idempotency_result
- Error and success events for enqueue, flush, retry, ack, and resume stages

Benefits:

- Faster production triage and postmortem traceability

---

## Supplemental Slice: Optional UX Package Integrations

#### Optional UX packages installed

Added to mobile dependencies:

- react-native-progress
- react-native-mask-text
- react-native-keyboard-aware-scroll-view

File:

- mobile/package.json

#### Booking form keyboard and phone input improvements

Updated file:

- mobile/src/screens/client/BookServiceScreen.tsx

Implemented:

- Contacts step switched to keyboard-aware scrolling container
- Sender and recipient phone fields now use masked inputs
- Existing normalize/validate logic retained

Benefits:

- Better mobile form ergonomics and visibility with keyboard open
- Clearer phone entry format and fewer input mistakes

#### Unlock progress visualization improvement

Updated file:

- mobile/src/screens/rider/BoxControlsScreen.tsx

Implemented:

- Unlock progress UI switched to react-native-progress bar

Benefits:

- Clearer state feedback during critical unlock flow

## Testing and Validation Performed

### 1. New security tests added

New file:

- mobile/src/services/security/__tests__/authSecretStore.test.ts

Covers:

- Auth secret persistence behavior
- Secret cleanup behavior
- Biometric invalidation to hard re-login signal
- Biometric-protected fallback PIN write options

Result:

- 4 passed, 0 failed

### 2. Existing authentication security suite re-run

File:

- mobile/src/__tests__/AuthenticationSecurity.test.ts

Result:

- 42 passed, 0 failed

### 3. Targeted reliability regression suites re-run

Files:

- mobile/src/__tests__/ec35StatusUpdateLost.test.ts
- mobile/src/__tests__/OfflineScenarios.test.ts

Result:

- 44 passed, 0 failed

### 4. Compile/error checks on changed files

Result:

- No new compile errors in touched files

### 5. Full-suite checkpoints

Result:

- Web full suite: 27 passed, 0 failed
- Mobile full suite: reliability-targeted changes validated; remaining failures are in broader pre-existing/unrelated groups (native module mocks and legacy expectation drift)

Note on lint:

- Repository still contains unrelated existing lint warnings/errors in other modules/screens. The implemented changes were validated and corrected where necessary for touched areas.

---

## Files Added

- mobile/src/services/networkPolicy.ts
- mobile/src/services/queueIdentity.ts
- mobile/src/services/notificationDedupService.ts
- mobile/src/navigation/navigationService.ts
- mobile/src/services/queryClient.ts
- mobile/src/services/actionCriticality.ts
- mobile/src/services/security/secureStoreService.ts
- mobile/src/services/security/authSecretStore.ts
- mobile/src/services/security/deviceRiskService.ts
- mobile/src/services/observability/sentryService.ts
- mobile/src/services/security/__tests__/authSecretStore.test.ts

## Key Files Updated

- mobile/app.json
- mobile/package.json
- mobile/App.js
- mobile/src/services/supabaseClient.ts
- mobile/src/services/sessionService.ts
- mobile/src/screens/auth/AuthLoadingScreen.tsx
- mobile/src/screens/auth/RoleSelectionScreen.tsx
- mobile/src/services/pushNotificationService.ts
- mobile/src/services/backgroundServiceManager.ts
- mobile/src/services/statusUpdateService.ts
- mobile/src/services/offlineQueueService.ts
- mobile/src/services/boxCommandQueueService.ts
- mobile/src/services/foregroundResumePipelineService.ts
- mobile/src/services/deliverySyncService.ts
- mobile/src/services/orderListenerService.ts
- mobile/src/services/notificationService.ts
- mobile/src/navigation/AppNavigator.tsx
- mobile/src/screens/common/NotificationListScreen.tsx
- mobile/src/screens/client/DeliveryLogScreen.tsx
- mobile/src/screens/rider/AssignedDeliveriesScreen.tsx
- mobile/src/screens/admin/AdminRecordsScreen.tsx
- mobile/src/screens/client/BookServiceScreen.tsx
- mobile/src/screens/rider/BoxControlsScreen.tsx
- web/src/app/api/cancel-booking/route.ts
- web/src/app/api/deliveries/[id]/no-show/route.ts

---

## Practical Benefits for Thesis Demo

1. Stronger resilience story:
   - Shared retry/timeout policy across services
   - Resume-first staged reconciliation after background/offline
   - Notification dedup and listener health recovery

2. Stronger data consistency story:
   - Queue IDs and replay diagnostics across status/location/box-command flows
   - Backend idempotency for cancellation and no-show retries

3. Stronger security story:
   - Secure token storage policy
   - Biometric invalidation handling with forced re-auth
   - Device-risk-aware sessions

4. Better observability story:
   - Tagged queue lifecycle telemetry in Sentry
   - Better traceability of flush/retry/ack behavior

5. Better UX story:
   - Clearer unlock progress feedback
   - Better keyboard handling and phone input masking
   - More reliable notification-to-screen navigation on cold/warm start

6. Better verification story:
   - New targeted security unit tests
   - Existing auth security suite still passing
   - Reliability regression suites passing

---

## Remaining Recommended Hardening (outside this completed slice)

- Tighten Firebase Realtime Database rules in database.rules.json to enforce role-based access server-side (current permissive rules remain a security risk if unchanged).
