# Admin Parity TODO Tracker

Last Updated: 2026-04-06
Owner: Mobile + Web Platform Team
Source Plan: /memories/session/plan.md

## Required Fields
Each item tracks: stream/pod, owner, priority, dependencies, status, test evidence, merged commit/PR reference.

## In Progress

| ID | Stream/Pod | Owner | Priority | Dependencies | Status | Test Evidence | Commit/PR |
|---|---|---|---|---|---|---|---|
| AP-018 | Stream A / 1.4 Firebase RTDB rules hardening | Backend | P0 | Admin auth contract stable | In Progress | Manual rules review pending | local-only |
| AP-019 | Stream A / 1.4 Supabase RLS expansion for admin-sensitive tables | Backend | P0 | Policy draft | In Progress | SQL policy dry run pending | local-only |
| AP-027 | Stream D / 4.1 reconnect sync telemetry for admin writes | Mobile | P1 | queue instrumentation points | In Progress | Pending reconnect scenario validation | local-only |

## Backlog

| ID | Stream/Pod | Owner | Priority | Dependencies | Status | Test Evidence | Commit/PR |
|---|---|---|---|---|---|---|---|
| AP-030 | Stream D / 4.2 idempotent command handling for all queue-driven admin actions | Mobile | P1 | AP-027 | Backlog | Not started | - |
| AP-031 | Stream E / 5.5 daily update cadence automation | Project | P2 | Tracker adoption | Backlog | Not started | - |
| AP-032 | Stream F / 6.1 full regression suite (web+mobile) | QA | P1 | AP-018, AP-019 | Backlog | Not started | - |
| AP-033 | Stream F / 6.2 high-risk manual Android+iOS run | QA | P1 | AP-032 | Backlog | Not started | - |

## Blocked

| ID | Stream/Pod | Owner | Priority | Dependencies | Status | Test Evidence | Commit/PR |
|---|---|---|---|---|---|---|---|
| AP-034 | Stream F / 6.3 staged feature-flag rollout percentages | Release | P1 | Production metrics dashboards + rollout windows | Blocked | N/A | - |

## Done

| ID | Stream/Pod | Owner | Priority | Dependencies | Status | Test Evidence | Commit/PR |
|---|---|---|---|---|---|---|---|
| AP-001 | Stream A / 1.1 bearer+cookie compatibility across admin APIs | Backend | P0 | None | Done | Manual API verification + mobile compile | local-only |
| AP-002 | Stream A / 1.2 admin role checks for weak admin endpoints | Backend | P0 | AP-001 | Done | Route-level auth checks + diagnostics clean | local-only |
| AP-003 | Stream A / 1.3 send-receipt auth hardening | Backend | P0 | AP-001 | Done | Endpoint rejects unauthorized paths | local-only |
| AP-004 | Stream A / 1.3 send-receipt idempotency + rate-limit | Backend | P0 | AP-003 | Done | Idempotency + 429 guard added | local-only |
| AP-005 | Stream A / 1.5 bearer contract for tracking history | Backend | P0 | AP-001 | Done | Route auth + mobile request compatibility | local-only |
| AP-010 | Stream B / 2.1 admin bottom navigation redesign | Mobile | P0 | None | Done | TS compile pass, routes verified | local-only |
| AP-011 | Stream B / 2.2 nested admin stacks | Mobile | P0 | AP-010 | Done | Navigator ids + tabs verified | local-only |
| AP-012 | Stream B / 2.3 destination mapping (Ops/Security/Insights/More) | Mobile | P0 | AP-011 | Done | Route navigation updates verified | local-only |
| AP-013 | Stream C / C1 account management parity | Mobile | P0 | AP-001 | Done | users + settings screens wired | local-only |
| AP-014 | Stream C / C2 receipts parity | Mobile | P0 | AP-001 | Done | list + send action wired | local-only |
| AP-015 | Stream C / C3 hardware diagnostics parity | Mobile | P1 | AP-011 | Done | realtime screen wired | local-only |
| AP-016 | Stream C / C4 stolen box parity | Mobile | P1 | AP-011 | Done | lockdown + clear controls wired | local-only |
| AP-017 | Stream C / C5 edge case parity | Mobile | P1 | AP-011 | Done | waits + reschedules actions wired | local-only |
| AP-020 | Stream C / C7 tracking history parity | Mobile | P1 | AP-005 | Done | history screen + backend route gated | local-only |

## Blocker Log

| Date | Blocker | Owner | ETA | Mitigation |
|---|---|---|---|---|
| 2026-04-06 | Unauthorized errors on mobile admin APIs due cookie-only routes | Backend | Resolved | Added bearer+cookie dual-auth support and mobile 401 refresh/retry |
| 2026-04-06 | Missing Settings/Profile access from admin tabs | Mobile | Resolved | Added More tab and menu entry stack |

## Completion Criteria
A task moves to Done only when all are true:
1. Code complete and merged in workspace.
2. Type/lint checks pass for touched files.
3. Manual sanity check of the flow is completed.
4. Evidence line updated in this tracker.
