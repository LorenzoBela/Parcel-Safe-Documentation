# Delivery Completion Status Stuck on ARRIVED

## Summary

Rider drop-off completion can leave a delivery displayed as `ARRIVED` after the rider swipes to complete, especially around the rider phone fallback / ML Kit face-detection path.

The issue is not a single UI bug. There are two observed failure modes:

1. **Split-brain completion:** Firebase and audit logs show `COMPLETED`, but Supabase `deliveries.status` remains `ARRIVED`.
2. **No completion evidence:** the delivery remains `ARRIVED` in both Firebase and Supabase because the completion path never produced a delivery proof photo or completion transition.

Because the web tracker and rider dashboard read Supabase in several places, any stale Supabase status can keep showing `ARRIVED` even when Firebase moved forward.

## Observed Evidence

### Split-brain examples

These deliveries had completion evidence but were still shown as `ARRIVED` until manually healed:

- `BK_1779110086001_e5k1awn`
- `BK_1779111019581_ncrwlu4`

For these rows:

- Firebase `/deliveries/{id}/status` was `COMPLETED`.
- Firebase had `delivered_at` and `proof_photo_url`.
- Supabase audit logs had `DELIVERY_COMPLETED` with `fromStatus: ARRIVED`, `toStatus: COMPLETED`.
- Supabase `deliveries.status` stayed `ARRIVED`.

This means the deployed lifecycle endpoint accepted enough of the completion flow to write Firebase/audit, but Supabase did not land on `COMPLETED`.

### Latest different case

`BK_1779111464357_e2n5j3h` was different:

- Firebase status: `ARRIVED`
- Supabase status: `ARRIVED`
- Supabase audit: only `IN_TRANSIT` and `ARRIVED`, no `DELIVERY_COMPLETED`
- Firebase delivery node: pickup photo exists, no delivery proof photo
- Firebase `delivery_proofs/{id}`: `null`
- Firebase `audit_logs/{id}`: `null`
- Box lock event was stale from an older session

This means the swipe did not complete the proof-upload / completion path. A status resolver cannot safely infer `COMPLETED` from this row because there is no delivery proof photo.

## Relevant Code Paths

- Mobile drop-off completion: `mobile/src/screens/rider/components/DropoffVerification.tsx`
- Mobile transition API wrapper: `mobile/src/services/riderMatchingService.ts`
- Web lifecycle transition service: `web/src/lib/deliveryLifecycleService.ts`
- Web tracking status resolver: `web/src/lib/deliveryDetailResolver.ts`
- Web tracking page: `web/src/app/track/[token]/TrackingClient.tsx`
- Firebase-to-Supabase sync route: `web/src/app/api/sync-deliveries/route.ts`

## Why "Both Pics Are Up Means Completed" Helps but Is Not Enough

Using evidence-based status is a good workaround when the data exists.

Safe completion evidence:

- Delivery has `proof_photo_url`, or
- Delivery has `delivered_at` / `completed_at`, or
- Firebase has `status: COMPLETED`, or
- Audit log has a validated `DELIVERY_COMPLETED` transition.

Unsafe evidence:

- Pickup photo only. A pickup photo proves pickup, not delivery completion.
- Stale hardware camera `last_upload_public_url` if it is not tied to the current `deliveryId`.
- Stale lock event from a previous session.

The web resolver already treats `proof_photo_url` as completion evidence. That covers split-brain rows when the page actually receives proof evidence. It does not cover rows where proof upload never happened.

## Current Mitigations Added Locally

### Backend terminal reconciliation

`web/src/lib/deliveryLifecycleService.ts` was updated so terminal transitions reconcile Supabase through an admin write and verify the resulting row status.

Purpose:

- If `COMPLETED` is accepted, Supabase must actually become `COMPLETED`.
- If reconciliation fails, the transition should fail loudly instead of leaving audit/Firebase ahead of Supabase.

Important deployment note:

- The mobile app currently points to `https://parcel-safe.vercel.app`.
- Local backend changes do not affect phone tests until the web app is deployed.

### Mobile mirror sync after completion

`mobile/src/screens/rider/components/DropoffVerification.tsx` was updated to call a Firebase-to-Supabase mirror sync after writing Firebase `COMPLETED`.

Purpose:

- If the deployed transition endpoint leaves Supabase stale, the sync route can mirror Firebase `COMPLETED` back into Supabase.

Limit:

- This only helps after the mobile app is rebuilt/reloaded with the patched code.
- It still requires Firebase to have `COMPLETED` and proof evidence.

### Rider dashboard first-snapshot terminal handling

`mobile/src/screens/rider/RiderDashboard.tsx` was updated so a first Firebase snapshot of `COMPLETED`, `CANCELLED`, or `RETURNED` clears the active delivery immediately.

Purpose:

- Avoid showing an active `ARRIVED` job after Firebase already says terminal.

Limit:

- This does not fix Supabase rows that never completed.

## Recommended Robust Fix

Implement completion as an idempotent, evidence-aware operation owned by the backend.

### Proposed API behavior

Create or harden a backend endpoint such as:

`POST /api/deliveries/{id}/complete`

Request body:

- `boxId`
- `proofPhotoUrl`
- `proofPhotoUploadedAt`
- `proofSource`: `smart_box` or `rider_phone_fallback`
- `fallbackPhotoUsed`

Server responsibilities:

1. Verify authenticated rider is assigned to the delivery.
2. Verify delivery is in `ARRIVED` or `IN_TRANSIT`.
3. Require proof evidence for fallback completion.
4. Write Supabase `status = COMPLETED`, `delivered_at`, `proof_photo_url`, `proof_photo_uploaded_at`.
5. Write Firebase `/deliveries/{id}` with the same terminal fields.
6. Clear hardware context only after the row is terminal.
7. Return the fresh delivery row.

The mobile app should call this endpoint after upload and then update UI from the returned row.

## Safer Status Resolution Rule

For display only, status can be resolved as:

1. `RETURNED` if return evidence exists.
2. `COMPLETED` if completion evidence exists.
3. Otherwise use highest-priority status from Firebase live status, Supabase status, initial status.

Completion evidence should include:

- `proof_photo_url`
- `delivered_at`
- `completed_at`
- Firebase live `status === COMPLETED`

Completion evidence should not include:

- `pickup_photo_url` alone
- Generic hardware camera URL not tied to current delivery
- Stale lock events

## Immediate Debug Checklist for Next Reproduction

When a swipe still shows `ARRIVED`, inspect the same delivery id in this order:

1. Supabase `deliveries.status`
2. Firebase `/deliveries/{id}/status`
3. Supabase `audit_logs` for `DELIVERY_COMPLETED`
4. Supabase `proof_photo_url` and `proof_photo_uploaded_at`
5. Firebase `/delivery_proofs/{id}`
6. Firebase `/audit_logs/{id}`
7. Firebase `/photo_uploads/{boxId}`
8. Current lock event timestamp versus delivery `created_at`

Interpretation:

- Firebase `COMPLETED` + Supabase `ARRIVED`: mirror/reconciliation failure.
- Both `ARRIVED` + no proof photo: mobile completion did not execute or proof upload failed.
- Proof photo exists + both `ARRIVED`: transition call failed after upload.

## Current High-Probability Root Causes

1. The deployed backend is behind the local lifecycle-service fix.
2. Mobile fallback completion depends on proof upload completing; if upload fails or the swipe is not fired, no completion evidence exists.
3. Some hardware nodes contain stale camera/lock data from old deliveries, so completion gating must only trust evidence tied to the current delivery id.
4. The UI can show stale active jobs if it loads Supabase `ARRIVED` before seeing a terminal Firebase snapshot.

## Next Actions

1. Deploy the backend lifecycle reconciliation fix.
2. Rebuild/reload the mobile app so the mirror-sync fallback is active.
3. Add a dedicated backend `complete` endpoint for rider drop-off completion.
4. Add logging around the mobile fallback swipe:
   - before proof upload
   - after proof upload
   - before transition API call
   - after transition API response
   - after Firebase sync
5. Add a repair job:
   - Find rows where Firebase says `COMPLETED` but Supabase says non-terminal.
   - Copy terminal evidence from Firebase to Supabase.

