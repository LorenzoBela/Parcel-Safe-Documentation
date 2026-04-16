# Reliability Remaining To-Do

Updated: 2026-04-10
Source: session plan + 10 parallel code audits

## Already Implemented
- Proxy atomic delivery context persistence and durable command ACK lifecycle.
- Controller offline OTP policy (uptime TTL + reboot token) and peer liveness handling.
- Proxy diagnostics expansion (cam/controller health + command stage fields).
- ESP32-CAM pending payload persistence/replay with bounded retry/backoff.
- Reliability mode in test runners and canary guidance in testing docs.

## Remaining Work (Execution Order)

### P0 - Firmware blockers before bench canary
- [x] Remove blocking reconnect loops in CAM WiFi maintenance and face-status request waiting paths.
  - Target: `Ard IDE/3_Eye_ESP32CAM/3_Eye_ESP32CAM.ino`
- [x] Refactor Proxy LTE reconnect/init paths that block for long windows into non-blocking scheduler/state-machine stages.
  - Target: `Ard IDE/2_Proxy_LilyGO/2_Proxy_LilyGO.ino`
- [x] Finish partial-connectivity matrix behavior for all degraded combinations.
  - Must define explicit behavior for `Controller_DOWN + CAM_UP` and `Proxy_DOWN` peer behavior.
  - Targets: `Ard IDE/2_Proxy_LilyGO/2_Proxy_LilyGO.ino`, `Ard IDE/1_Controller_ESP32/1_Controller_ESP32.ino`
- [x] Harden idempotency on retryable backend writes (command ACK, tamper/event writes, camera upload replay path).
  - Targets: `Ard IDE/2_Proxy_LilyGO/2_Proxy_LilyGO.ino`, `Ard IDE/3_Eye_ESP32CAM/3_Eye_ESP32CAM.ino`

### P1 - Wear budget + telemetry gates
- [x] Add flash write counters and hourly export metrics (NVS/SPIFFS writes per hour).
  - Targets: `Ard IDE/2_Proxy_LilyGO/DeliveryPersist.cpp`, `Ard IDE/2_Proxy_LilyGO/2_Proxy_LilyGO.ino`, `Ard IDE/3_Eye_ESP32CAM/3_Eye_ESP32CAM.ino`
- [x] Reduce unnecessary repeat persistence writes by adding change-detection/batching where applicable.
  - Targets: `Ard IDE/2_Proxy_LilyGO/2_Proxy_LilyGO.ino`, `Ard IDE/2_Proxy_LilyGO/DeliveryPersist.cpp`
- [x] Add timing instrumentation for unlock and reconnect latency (P95/P99 extraction-ready).
  - Targets: controller/proxy lock/reconnect paths + telemetry export

### P0/P1 Delta Notes (Current Pass)
- CAM: non-blocking reconnect scheduler, deferred face client intake (no blocking wait loop), upload idempotency header, hourly write metrics log.
- Proxy: async modem recovery task, runtime reconnect scheduling, connectivity matrix state + `/diag` fields, short-window idempotency dedupe for command ACK/tamper/lock events.
- Persistence: NVS write counters in `DeliveryPersist` and hash-based change detection for delivery + personal PIN context saves.
- Controller: unlock latency metric forwarded in event payload; proxy stores `unlock_latency_ms` in lock event data.

### P2 - Bench canary tooling
- [x] Add bench canary orchestration scripts for Windows/Linux.
  - Targets: `scripts/bench-canary.ps1`, `scripts/bench-canary.sh`
- [x] Add chaos/fault injection test modules (brownout, AP flap, LTE outage, Firebase timeout).
  - Target folder: `hardware/test/chaos/`
- [x] Add metrics aggregation parser and pass/fail report generation for canary runs.
  - Suggested targets: `hardware/tools/`, `scripts/`

### P3 - Run and gate
- [x] Execute canary scenarios with repeated cycles and capture evidence for all promotion gates.
  - Scenarios: brownout mid-unlock, dead-zone traversal, CAM unreachable, controller unreachable, LTE/Firebase outage
- [x] Evaluate gates:
  - recovery success rate
  - all-nodes-up availability
  - duplicate command rate
  - queue/payload loss rate
  - unlock/reconnect latency
  - flash write rate
- [x] Prepare staged rollout decision report (bench canary -> limited field canary -> full rollout) with rollback triggers.

### P3 Execution Evidence (2026-04-10)
- Command: `./scripts/bench-canary.ps1 -Cycles 3 -SkipBaseline -DryRun`
- Output folder: `scripts/outputs/canary/20260410-134703/`
- Gate report: `scripts/outputs/canary/20260410-134703/canary-gates.json`
- Rollout report: `scripts/outputs/canary/20260410-134703/rollout-decision.md`
- Gate summary: 18/18 runs, all gates PASS in simulation mode.
- Rollout decision: `HOLD_FOR_REAL_BENCH_CANARY` (expected for dry-run synthetic metrics).

## Quick Next Run (Today)
1. Run `./scripts/bench-canary.ps1 -Cycles 3` on live bench hardware (no `-DryRun`) to collect real telemetry.
2. Promote to limited field canary only if rollout decision moves from `HOLD_FOR_REAL_BENCH_CANARY` to `PROMOTE_TO_LIMITED_FIELD_CANARY`.
3. Keep rollback triggers in the generated rollout report as hard promotion gates.
