# Full Briefer: Firmware Set 1/2/3 vs 4/5/6

This document is a detailed handoff brief for anyone comparing the original
Smart Top Box firmware set `1/2/3` against the newer production/demo set
`4/5/6`.

It explains what each set is, what changed, why it changed, what behavior to
expect, what benefits 4/5/6 adds, what risks it introduces, and how to validate
the new architecture.

The short version:

- **1/2/3 is the original autonomous SoftAP baseline.**
- **4/5/6 is the newer layered production/demo variant.**
- **4/5/6 is not just "use a phone hotspot."**
- **4/5/6 changes the priority stack to UART first, hotspot WiFi second, BLE
  third, LTE last for cloud fallback.**

---

## 1. Folder Map

The repository now has two main firmware sets:

| Board role | Original set | New production/demo set |
| --- | --- | --- |
| Main controller ESP32 | `Ard IDE/1_Controller_ESP32` | `Ard IDE/4_Controller_ESP32_Hotspot` |
| LilyGO proxy / GPS / LTE board | `Ard IDE/2_Proxy_LilyGO` | `Ard IDE/5_Proxy_LilyGO_Hotspot` |
| ESP32-CAM eye board | `Ard IDE/3_Eye_ESP32CAM` | `Ard IDE/6_Eye_ESP32CAM_Hotspot` |

The original `1/2/3` sketches were intentionally left unchanged as the current
baseline. The new work lives in duplicated folders `4/5/6`.

That matters because it gives us a rollback path:

- If the new architecture has a problem, flash `1/2/3`.
- If the demo needs faster sync and better fallback layering, flash `4/5/6`.

---

## 2. System Roles

The Smart Top Box is a three-board distributed system.

### Controller ESP32

The controller board owns the local physical interaction:

- Keypad input.
- I2C LCD output.
- OTP entry flow.
- Personal PIN flow.
- Solenoid lock control.
- Reed switch lock state.
- Tamper detection.
- Lockout handling.
- Admin override handling.
- Local delivery context cache.
- Controller-side route selection to LilyGO.

In `1_Controller_ESP32`, the controller mainly talks to LilyGO through the
LilyGO SoftAP over WiFi HTTP.

In `4_Controller_ESP32_Hotspot`, the controller now uses a layered local
communication strategy:

1. Dedicated controller-to-LilyGO UART.
2. Hotspot LAN HTTP after discovering the LilyGO IP.
3. BLE fallback for small control messages.
4. Cached/offline behavior where the existing logic supports it.

### LilyGO Proxy Board

The LilyGO board is the system backbone:

- Local API server for controller requests.
- Local API server for camera upload/status.
- GPS/geofence logic.
- Firebase/Supabase cloud sync.
- LTE modem control.
- Delivery context cache.
- Command polling.
- Event/tamper/ack sync.
- Image relay from ESP32-CAM to cloud.
- In `4/5/6`, WiFi hotspot internet is preferred over LTE for cloud operations.

In `2_Proxy_LilyGO`, LilyGO acts as the SoftAP and LTE-based cloud gateway.

In `5_Proxy_LilyGO_Hotspot`, LilyGO joins one of the configured hotspots as a
station and uses hotspot WiFi internet first. LTE remains as a fallback for cloud
operations.

### ESP32-CAM Eye Board

The camera board owns:

- Face/status HTTP endpoint.
- Face detection.
- Proof-of-delivery photo capture.
- Image upload to LilyGO.
- Existing controller-to-camera UART fallback for face checking, if wired.

In `3_Eye_ESP32CAM`, the camera joins the LilyGO SoftAP.

In `6_Eye_ESP32CAM_Hotspot`, the camera joins the same configured hotspot as the
other boards and discovers LilyGO over UDP. It remains WiFi-only for image
upload. BLE is intentionally not used for photo transfer.

---

## 3. Original 1/2/3 Architecture

The original architecture is a LilyGO-owned local network.

```text
Controller ESP32
  |
  | WiFi station
  v
LilyGO SoftAP, fixed local IP, local HTTP API
  ^
  | WiFi station
  |
ESP32-CAM

LilyGO -> LTE modem -> Firebase/Supabase
```

Typical characteristics:

- LilyGO creates the local WiFi network.
- Controller and camera join LilyGO's SoftAP.
- LilyGO usually has a deterministic local IP, commonly `192.168.4.1`.
- Controller can directly target LilyGO without discovery complexity.
- Camera can directly target LilyGO on the local SoftAP.
- Cloud sync relies on LTE.

### Strengths of 1/2/3

1/2/3 is strong because it is simple and self-contained:

- No external hotspot is required for local board-to-board communication.
- No rider phone is required to keep the local network alive.
- The proxy address is stable.
- The topology is easy to reason about.
- There are fewer fallback transports to test.
- Firmware size is lower than 4/5/6.

This is why 1/2/3 remains the autonomous baseline.

### Weaknesses Observed in 1/2/3

The user-visible issue was not that SoftAP is bad. The issue was that local
controller HTTP and cloud modem work could interfere with each other on the
LilyGO.

Observed or suspected pain points:

- LTE HTTP operations can block for seconds.
- Supabase/Firebase calls through modem AT commands can be slow.
- Camera relay and LTE uploads can occupy the proxy for long windows.
- Controller `/otp`, `/refresh-context`, diagnostics, and event requests can
  feel delayed when LilyGO is busy.
- Controller TCP timeouts can trigger reconnect behavior.
- The user experience can feel sluggish even though the local SoftAP topology is
  theoretically deterministic.

Important nuance:

**1/2/3 is architecturally clean, but it can still feel slow if the LilyGO's
local request handling is blocked behind modem/cloud work.**

---

## 4. New 4/5/6 Architecture

The new architecture was created to preserve the box logic while changing the
transport priority.

```text
Controller ESP32
  |
  | Primary local control: dedicated UART
  v
LilyGO Proxy
  |
  | Secondary local control and camera path: shared hotspot LAN
  v
Phone hotspot / mobile router / WiFi AP
  |
  | Primary cloud path when healthy
  v
Firebase/Supabase

LilyGO LTE modem remains available as fallback cloud path.

Controller BLE client -> LilyGO BLE GATT server is a small fallback control
plane when UART and WiFi HTTP are unavailable or unstable.
```

The actual 4/5/6 runtime priority is:

1. **Controller to LilyGO UART**.
2. **Controller to LilyGO WiFi HTTP over hotspot LAN**.
3. **Controller to LilyGO BLE GATT fallback**.
4. **Controller cached/offline behavior where supported**.
5. **LilyGO to cloud over hotspot WiFi internet**.
6. **LilyGO to cloud over LTE fallback**.

This is the core idea: **4/5/6 does not depend on the hotspot for the most
important controller-to-LilyGO path.**

---

## 5. Why 4/5/6 Was Created

The practical goal was demo and testing reliability:

- The phone hotspot or external router can provide faster internet than LTE.
- Firebase/Supabase sync is usually faster over WiFi internet.
- Controller-to-LilyGO local control should not wait on LTE behavior.
- The demo should feel snappier.
- If WiFi cloud fails, LTE should still exist.
- If hotspot LAN is unstable, local controller-to-LilyGO control should still
  exist.

The design also helps with thesis/demo presentation:

- A laptop can join the same hotspot for diagnostics.
- The boards can be demonstrated in a controlled WiFi environment.
- LTE remains available for failure demonstration.
- Transport details can be hidden from the I2C LCD so the user-facing display
  stays clean.

---

## 6. Major Changes Made

### 6.1 Duplicated Firmware Sets

We created new production/demo variants:

- `4_Controller_ESP32_Hotspot`
- `5_Proxy_LilyGO_Hotspot`
- `6_Eye_ESP32CAM_Hotspot`

The original `1/2/3` folders were not modified.

Reason:

- Preserve a stable baseline.
- Allow direct comparison.
- Allow rollback.
- Avoid mixing experimental transport changes into the older firmware.

### 6.2 Multi-Hotspot Support

All three new boards support a list of configured hotspots.

Configured hotspot SSIDs are stored in the new sketch configs. The credentials
are intentionally not repeated in this document to avoid duplicating secrets.

Behavior:

- Scan available WiFi networks.
- Match against configured hotspot list.
- Prefer configured networks.
- Connect to the best available configured hotspot.
- Reconnect/rescan when the current hotspot disappears.
- If no hotspot exists, LilyGO can continue toward LTE fallback for cloud sync.

Expected benefit:

- For demo, you can prepare multiple phones/routers.
- If one hotspot is down, boards can try another.
- The system is not locked to one phone.

Important limitation:

- If all hotspots are down, camera WiFi upload cannot work over hotspot LAN.
- Controller-to-LilyGO UART can still work if wired.
- LilyGO cloud can fall back to LTE.

### 6.3 Dedicated Controller-to-LilyGO UART

A new wired UART link was added between controller and LilyGO.

Controller config:

- `PROXY_UART_RX`
- `PROXY_UART_TX`
- `PROXY_UART_BAUD`
- `PROXY_UART_TIMEOUT_MS`

LilyGO config:

- `PROXY_UART_RX`
- `PROXY_UART_TX`
- `PROXY_UART_BAUD`

Wiring rule:

| Wire | LilyGO Pin | ESP32 Controller Pin |
| --- | --- | --- |
| LilyGO TX -> Controller RX | GPIO21 | GPIO33 |
| LilyGO RX <- Controller TX | GPIO22 | GPIO27 |
| Ground reference | GND | GND |

```text
LilyGO GPIO21 TX -> Controller GPIO33 RX
LilyGO GPIO22 RX <- Controller GPIO27 TX
LilyGO GND       -> Controller GND
```

The pins were deliberately chosen away from existing UART use:

- Controller `Serial2` GPIO 16/17 is already used for ESP32-CAM fallback.
- LilyGO modem UART pins are already used by the LTE modem.
- The new proxy UART must not collide with camera UART or modem UART.

Controller keypad pinout note:

- GPIO 13 and GPIO 14 remain reserved for the existing keypad harness.
- Do not wire controller-to-LilyGO UART to GPIO 13 or GPIO 14 unless the keypad
  harness is redesigned.
- The proxy UART uses controller GPIO 33/27 to avoid disturbing keypad wiring.
- Current controller keypad firmware remains:
  - Rows: GPIO 13, 23, 19, 26.
  - Columns: GPIO 14, 25, 18.

Purpose:

- Make local controller-to-proxy communication independent of WiFi.
- Avoid hotspot client isolation.
- Avoid UDP discovery failures for core requests.
- Avoid WiFi reconnect latency for critical keypad paths.
- Reduce user-visible sync stalls.

### 6.4 Line-Based UART Protocol

The controller and LilyGO communicate with compact line frames over UART.

Expected properties:

- Request IDs.
- Method/path/payload shape similar to HTTP.
- Checksum/guarding to avoid accepting corrupt or stale frames.
- Stale receive buffer flushing before a new request.
- Timeout behavior.
- Small payloads.

The UART request path is used to mirror the same actions that were previously
HTTP-only:

- OTP/context snapshot.
- Refresh context.
- Diagnostics.
- Event write/enqueue.
- Tamper report.
- Command ack.
- Personal PIN verify.
- Camera power command.
- Face-check proxy request.

This was done to preserve the existing logic while changing only the transport
priority.

### 6.5 WiFi Hotspot LAN HTTP Fallback

WiFi HTTP still exists.

It is now the second controller-to-LilyGO route:

1. Try UART first.
2. If UART is unavailable, try WiFi HTTP.
3. If WiFi HTTP is unavailable or unstable, use BLE fallback for small control
   messages.

Because the LilyGO IP is dynamic on a phone/router hotspot, discovery is needed.

Discovery behavior:

- Controller sends a UDP discovery query.
- LilyGO replies with its current IP.
- Camera also discovers LilyGO over UDP.
- LilyGO tracks camera IP when it receives camera discovery messages.

Important limitation:

- UDP broadcast can be blocked or delayed by some phone hotspots.
- Some hotspots isolate clients from each other.
- That is why UART-first and BLE fallback matter.

### 6.6 BLE Reliability Layer

BLE was added only to:

- `4_Controller_ESP32_Hotspot`
- `5_Proxy_LilyGO_Hotspot`

BLE was not added to:

- `6_Eye_ESP32CAM_Hotspot`

Reason:

- BLE is suitable for small control/status messages.
- BLE is not suitable for JPEG photo transfer.
- ESP32-CAM already has tight memory/radio pressure.
- Camera upload needs WiFi bandwidth.

BLE architecture:

- LilyGO is the BLE GATT server.
- Controller is the BLE client.
- The BLE service is a custom SmartTopBox service.
- BLE uses NimBLE-Arduino instead of classic ESP32 BLE to reduce flash/RAM
  pressure.

BLE characteristics include:

- Ping/health.
- OTP/context snapshot.
- Refresh-context request.
- Event/tamper/command-ack enqueue.
- Proxy IP advertisement.
- Future optional hotspot credential update.

BLE priority:

1. UART first.
2. WiFi HTTP second.
3. BLE third.

BLE is not the main data path. It is a low-bandwidth control plane.

Expected benefit:

- If hotspot client isolation blocks controller-to-LilyGO HTTP, BLE can still
  carry small control messages.
- If UDP discovery fails, BLE can expose the LilyGO IP.
- If UART wiring becomes loose, BLE provides another local fallback.

Limitations:

- BLE and WiFi share the ESP32 2.4 GHz radio.
- BLE should stay lightweight.
- BLE range should be enough inside or near the box, but it is not a long-range
  transport.
- BLE does not solve camera image transfer.

### 6.7 LilyGO Cloud Priority Changed

In 1/2/3:

- LilyGO cloud operations are LTE-oriented.

In 4/5/6:

- LilyGO uses hotspot WiFi internet first when healthy.
- LTE is fallback when hotspot WiFi is missing or cloud health fails.
- LilyGO can switch future cloud calls back to WiFi when WiFi becomes healthy
  again.

Expected benefit:

- Faster Firebase/Supabase calls during demo.
- Faster event sync.
- Faster context refresh.
- Potentially faster photo relay to Supabase when WiFi cloud is healthy.
- LTE remains available for field resilience.

Important distinction:

- LTE fallback is for LilyGO cloud operations.
- Controller does not use LTE directly.
- Camera does not use LTE directly.

### 6.8 ESP32-CAM Kept WiFi-Only

The camera behavior remains intentionally WiFi-centered.

In 4/5/6:

- Camera joins one of the configured hotspots.
- Camera discovers LilyGO over UDP.
- Camera uploads images to LilyGO over HTTP.
- LilyGO relays images to Supabase over WiFi cloud or LTE fallback.
- Existing direct controller-to-camera UART fallback for face check can remain if
  that wiring is still used.

What we did not do:

- No BLE image upload.
- No camera LTE.
- No separate wired camera photo pipe.

Reason:

- JPEG transfer over BLE is too slow.
- BLE would add memory and radio pressure to ESP32-CAM.
- WiFi is the correct camera data path.

### 6.9 LCD Transport Hiding

The controller LCD should not expose transport details such as:

- LTE.
- Modem.
- APN.
- Carrier.
- BLE.

LCD should stay generic:

- `Net: CONNECTED`
- `Sync: OK`
- `Offline`
- `Signal`
- `Connecting WiFi`
- Other user/workflow-focused states.

Detailed transport debugging belongs on Serial logs, not the I2C LCD.

Reason:

- Cleaner user experience.
- Avoid confusing demo viewers.
- Avoid exposing implementation details.
- Keep the LCD focused on box state, not radio internals.

### 6.10 Controller Build Trim

The controller build grew after adding BLE and more fallback logic.

Observed compile pressure:

- User reported the controller sketch at about 97 percent flash usage.
- After trimming, compile output reached about 93 percent in one verified pass.

Trimming work applied:

- Added `CONTROLLER_VERBOSE_LOGS 0`.
- Changed `netLog(...)` from a runtime no-op into a compile-time macro when
  verbose logs are disabled.
- Added `CTRL_LOG_PRINT`, `CTRL_LOG_PRINTLN`, and `CTRL_LOG_PRINTF` macros.
- Replaced many controller-side `Serial.print*` debug calls with those macros.
- Added `build_opt.h` for controller NimBLE trim flags.
- Disabled unused NimBLE controller-side roles:
  - Peripheral role off.
  - Broadcaster role off.
- Reduced NimBLE logging:
  - Host log level none.
  - C++ wrapper log level none.
- Reduced BLE resource counts:
  - Max connections set to 1.
  - Max bonds set to 1.
  - Max CCCDs set to 1.
  - Preferred MTU set to 128.

Important compile note:

- We tried the NimBLE mbedTLS crypto stack switch because the NimBLE docs suggest
  it can save flash when mbedTLS is already present.
- On this Arduino ESP32 core, it caused unresolved CMAC link symbols.
- That flag was removed.

Result:

- Full controller feature set remains.
- BLE remains enabled.
- UART/WiFi/BLE priority remains.
- Verbose production debug strings are stripped.
- User-facing LCD behavior is preserved.

---

## 7. Expected Runtime Behavior

### 7.1 Normal Demo With Hotspot Available and UART Wired

Expected path:

```text
Controller -> LilyGO: UART
LilyGO -> Cloud: hotspot WiFi
Camera -> LilyGO: hotspot WiFi HTTP
```

Expected user experience:

- Fast keypad response.
- Fast OTP/context sync.
- Fast event acknowledgement.
- Faster cloud dashboard updates than LTE-only.
- LCD does not say LTE or BLE.

This is the best-case 4/5/6 behavior.

### 7.2 UART Disconnected, Hotspot LAN Working

Expected path:

```text
Controller -> LilyGO: WiFi HTTP after discovery
LilyGO -> Cloud: hotspot WiFi if healthy, LTE if not
Camera -> LilyGO: hotspot WiFi HTTP
```

Expected behavior:

- Slightly more latency than UART.
- Discovery is required because LilyGO IP is dynamic.
- Works if the hotspot allows client-to-client communication.

### 7.3 UART Disconnected, Hotspot Blocks Client-to-Client HTTP

Expected path:

```text
Controller -> LilyGO: BLE for small control messages
LilyGO -> Cloud: hotspot WiFi or LTE
Camera -> LilyGO: may fail if client isolation blocks camera-to-LilyGO traffic
```

Expected behavior:

- Controller may still get health/context through BLE.
- Some controller actions can still enqueue through BLE.
- Camera image upload may not work if the hotspot blocks clients from talking.

Important:

- BLE is not a replacement for camera WiFi upload.

### 7.4 All Hotspots Down, UART Wired

Expected path:

```text
Controller -> LilyGO: UART
LilyGO -> Cloud: LTE fallback
Camera -> LilyGO: no hotspot WiFi path unless another camera path exists
```

Expected behavior:

- Local controller-to-LilyGO control can remain alive.
- LilyGO cloud sync can continue through LTE.
- Camera WiFi upload is unavailable without a WiFi LAN.

### 7.5 All Hotspots Down, UART Disconnected, BLE In Range

Expected path:

```text
Controller -> LilyGO: BLE fallback for small messages
LilyGO -> Cloud: LTE fallback
Camera -> LilyGO: unavailable over hotspot WiFi
```

Expected behavior:

- Minimal control plane can remain.
- Not ideal for full workflow.
- Useful as a last local fallback before cached/offline behavior.

### 7.6 Hotspot Internet Fails but Local Hotspot LAN Stays Up

Expected path:

```text
Controller -> LilyGO: UART first or WiFi HTTP
LilyGO -> Cloud: LTE fallback
Camera -> LilyGO: hotspot LAN still works
```

Expected behavior:

- Local workflow can still work.
- Cloud sync may become slower because LilyGO falls back to LTE.
- LCD should remain generic.

### 7.7 Hotspot Internet Restored

Expected behavior:

- LilyGO should return future cloud calls to WiFi once cloud health is good.
- LTE remains available but no longer preferred.

---

## 8. Comparison Table

| Area | 1/2/3 Original SoftAP | 4/5/6 New UART-First Hotspot |
| --- | --- | --- |
| Main purpose | Autonomous baseline | Production/demo performance variant |
| Local network owner | LilyGO SoftAP | External hotspot/router |
| Controller critical path | WiFi HTTP to LilyGO | Dedicated UART first |
| Controller fallback path | WiFi reconnect/cache behavior | WiFi HTTP, BLE, cache/offline |
| LilyGO cloud path | LTE-focused | WiFi internet first, LTE fallback |
| Camera path | WiFi to LilyGO SoftAP | WiFi to LilyGO through hotspot LAN |
| Proxy IP | Fixed/deterministic | Dynamic, discovered by UDP/BLE |
| Client isolation risk | Low | Present for WiFi, bypassed by UART/BLE for controller |
| Phone/router dependency | None for local LAN | Needed for hotspot LAN/camera WiFi, not for UART local control |
| Demo speed | LTE-bound cloud sync | Faster when hotspot WiFi internet is healthy |
| Firmware complexity | Lower | Higher |
| Flash pressure | Lower | Higher |
| Wiring complexity | Lower | Higher due controller-to-LilyGO UART |
| Best use | Field baseline, simple autonomous mode | Demo, testing, faster sync, layered fallback |

---

## 9. Reliability Interpretation

The key comparison is not:

```text
SoftAP vs phone hotspot
```

The real comparison is:

```text
1/2/3: WiFi SoftAP local control + LTE cloud
4/5/6: UART local control + hotspot WiFi cloud + LTE fallback + BLE fallback
```

If 4/5/6 were hotspot-only, then the reliability criticism would be correct:

- Phone hotspots can sleep.
- Client isolation can break local HTTP.
- UDP discovery can fail.
- Dynamic IP adds uncertainty.

But current 4/5/6 is not hotspot-only. The controller tries UART first.

That means:

- Client isolation does not break controller-to-LilyGO UART.
- UDP discovery failure does not break controller-to-LilyGO UART.
- Phone hotspot screen lock does not break controller-to-LilyGO UART.
- Hotspot cloud failure can fall back to LTE.

So the honest conclusion is:

- **1/2/3 is simpler and more self-contained.**
- **4/5/6 is more layered and usually snappier when wired correctly.**
- **4/5/6 is better for demo/testing and fast cloud sync.**
- **1/2/3 remains better if the goal is minimum complexity with no external AP.**

---

## 10. Speed Expectations

### Controller-to-LilyGO

Expected fastest path in 4/5/6:

```text
UART
```

Why:

- No WiFi association delay.
- No DHCP dependency.
- No UDP discovery.
- No hotspot client isolation.
- No TCP socket overhead.
- Short line-based frames.

Expected user-visible result:

- Snappier OTP/context checks.
- Faster event acknowledgement.
- Fewer "sync lag" moments caused by WiFi route instability.

### LilyGO-to-Cloud

Expected fastest path in 4/5/6:

```text
Hotspot WiFi internet
```

Why:

- Typically lower latency than LTE.
- Typically higher bandwidth than LTE.
- Less AT-command overhead than modem HTTP.
- Better for Firebase/Supabase during demos.

LTE is still valuable, but it is no longer the preferred cloud path when good
WiFi exists.

### Camera Upload

Camera upload remains WiFi-based.

Expected fastest path in 4/5/6:

```text
ESP32-CAM -> hotspot LAN -> LilyGO -> hotspot WiFi cloud
```

Fallback:

```text
ESP32-CAM -> hotspot LAN -> LilyGO -> LTE cloud
```

No BLE photo upload.

---

## 11. What Could Still Go Wrong

4/5/6 improves responsiveness and fallback layering, but it adds complexity.

Main risks:

### UART Wiring Risk

If TX/RX are not crossed correctly, UART will fail.

Correct wiring:

```text
LilyGO GPIO21 TX -> Controller GPIO33 RX
LilyGO GPIO22 RX <- Controller GPIO27 TX
GND shared
```

If UART fails, controller should fall back to WiFi HTTP or BLE.

### Pin Conflict Risk

Do not reuse pins already dedicated to:

- Controller ESP32-CAM fallback UART.
- LilyGO modem UART.
- Boot strap pins that can prevent boot.

### Hotspot Client Isolation Risk

Some phone hotspots do not allow clients to talk to each other.

Effect:

- Controller WiFi HTTP may fail.
- Camera-to-LilyGO upload may fail.
- UART still saves controller-to-LilyGO local control.
- BLE may save small control messages.

### UDP Discovery Risk

Some hotspots delay or drop broadcast/multicast traffic.

Effect:

- Controller or camera may not discover LilyGO IP.
- BLE proxy IP advertisement can help controller.
- Camera still needs WiFi discovery or a known IP strategy.

### BLE/Radio Contention

BLE and WiFi share the 2.4 GHz radio.

Effect:

- BLE must remain low bandwidth.
- Do not use BLE for camera images.
- Do not poll BLE aggressively.

### Flash Size Risk

Controller is near the ESP32 app partition limit.

Mitigation already done:

- Verbose logs stripped.
- NimBLE roles trimmed.
- NimBLE resource counts reduced.

Future caution:

- Avoid adding large libraries to controller.
- Avoid verbose string logs in production.
- Prefer compact helpers and shared parsers.
- Keep BLE payloads small.

### Testing Burden

4/5/6 has more states:

- UART healthy.
- UART failed.
- WiFi healthy.
- WiFi local but no internet.
- WiFi client isolation.
- BLE reachable.
- BLE unreachable.
- LTE healthy.
- LTE failed.
- Camera discovered.
- Camera undiscovered.

This requires a broader test plan than 1/2/3.

---

## 12. Build and Compile Notes

### Controller 4

Controller 4 uses:

- WiFi.
- HTTPClient.
- WiFiUDP.
- NimBLE-Arduino.
- Preferences.
- Keypad.
- LiquidCrystal I2C.
- UART to LilyGO.
- UART to camera fallback.

Build trim added:

- `CONTROLLER_VERBOSE_LOGS 0`
- Compile-time `netLog(...)` removal.
- `CTRL_LOG_*` macros.
- `build_opt.h` NimBLE options.

Important:

- BLE remains enabled.
- Full feature set remains.
- The trim removes production debug text, not core behavior.

### LilyGO 5

LilyGO 5 uses:

- WiFi station mode.
- WiFi cloud health checks.
- LTE fallback.
- UDP discovery server.
- HTTP local API.
- UART protocol handler.
- NimBLE GATT server.
- Camera relay.
- Firebase/Supabase logic.

Important:

- It still has LTE.
- It should prefer WiFi cloud when healthy.
- It should fall back to LTE when WiFi cloud is unavailable.

### Camera 6

Camera 6 uses:

- WiFi station mode.
- Multi-hotspot scan/connect.
- UDP discovery for LilyGO.
- HTTP face-status endpoint.
- HTTP image upload to LilyGO.
- Existing UART face command fallback.

Important:

- Camera does not use BLE.
- Camera does not use LTE.
- Camera still needs a WiFi LAN path for image upload.

---

## 13. What to Tell a Reviewer

If someone asks, "Is 4/5/6 more reliable than 1/2/3?", answer carefully:

**For controller-to-LilyGO local control, yes, if UART is wired correctly.**

Why:

- UART is more deterministic than WiFi.
- UART bypasses hotspot issues.
- UART avoids SoftAP/client reconnect problems.

**For total system simplicity, no. 1/2/3 is simpler.**

Why:

- Fewer transports.
- Fixed IP.
- No external hotspot.
- Less code.

**For demo and cloud sync speed, 4/5/6 should be better.**

Why:

- Hotspot WiFi internet is typically faster than LTE.
- Controller local path is UART first.
- LTE remains fallback instead of primary.

**For camera upload, 4/5/6 depends on hotspot LAN quality.**

Why:

- Camera still uses WiFi.
- Client isolation can affect camera-to-LilyGO.
- BLE is not a camera data path.

---

## 14. Recommended Deployment Guidance

### Use 1/2/3 when:

- You want the simplest autonomous baseline.
- You do not want external AP dependency.
- You want fixed local addressing.
- You want fewer things to test.
- You are not prioritizing fastest cloud sync.

### Use 4/5/6 when:

- You want the snappiest controller-to-LilyGO behavior.
- You can wire controller-to-LilyGO UART correctly.
- You want hotspot WiFi for faster Firebase/Supabase.
- You want LTE as a fallback instead of the first cloud path.
- You are preparing for demo/testing.
- You accept higher complexity.

---

## 15. Validation Checklist

Before calling 4/5/6 production-ready, validate every layer.

### Compile

- Compile `4_Controller_ESP32_Hotspot`.
- Compile `5_Proxy_LilyGO_Hotspot`.
- Compile `6_Eye_ESP32CAM_Hotspot`.
- Confirm controller flash usage has acceptable headroom.

### UART

- Wire controller TX to LilyGO RX.
- Wire controller RX to LilyGO TX.
- Share GND.
- Verify UART health/ping request.
- Verify UART OTP/context request.
- Verify UART refresh-context request.
- Verify UART diagnostics request.
- Verify UART event enqueue.
- Verify UART tamper report.
- Verify UART command ack.
- Verify UART personal PIN verify.
- Verify stale response flushing by restarting one board mid-test.
- Verify bad checksum frames are ignored.

### WiFi Hotspot

- Test each configured hotspot.
- Test best available hotspot selection.
- Test reconnect after turning off current hotspot.
- Test fallback to another hotspot.
- Test all-hotspots-down behavior.
- Verify WiFi sleep is disabled on ESP32 station code where needed.

### UDP Discovery

- Verify controller discovers LilyGO IP.
- Verify camera discovers LilyGO IP.
- Verify LilyGO receives camera presence.
- Test after LilyGO DHCP IP changes.
- Test after hotspot restart.

### BLE

- Verify LilyGO advertises the SmartTopBox BLE service.
- Verify controller scans and connects.
- Verify BLE health/ping.
- Verify BLE context snapshot.
- Verify BLE refresh request.
- Verify BLE event/tamper/ack enqueue where implemented.
- Verify BLE proxy IP characteristic.
- Disconnect UART and block WiFi HTTP, then verify BLE fallback.

### Cloud Priority

- With hotspot internet working, verify LilyGO cloud calls use WiFi path.
- Disable hotspot internet while keeping local LAN alive.
- Verify LilyGO falls back to LTE.
- Restore hotspot internet.
- Verify LilyGO returns to WiFi cloud path.

### Camera

- Verify camera joins configured hotspot.
- Verify camera discovers LilyGO.
- Verify `/face-status` works.
- Verify photo upload to LilyGO works.
- Verify LilyGO relays photo to Supabase over WiFi cloud.
- Disable WiFi cloud internet and verify LilyGO LTE relay fallback.
- Confirm BLE is not used for photo upload.

### LCD/User Experience

- Confirm LCD does not show `LTE`.
- Confirm LCD does not show modem/APN/carrier wording.
- Confirm LCD does not expose BLE transport details.
- Confirm LCD uses generic states such as connected, syncing, offline, signal,
  or workflow-specific states.

### Failure Tests

- UART disconnected.
- Hotspot LAN working.
- Hotspot LAN isolated.
- Hotspot internet down.
- All hotspots down.
- LTE unavailable.
- Camera missing.
- LilyGO reboot while controller is running.
- Controller reboot while LilyGO is running.
- ESP32-CAM reboot while LilyGO is running.

---

## 16. Final Verdict

The new 4/5/6 set should be understood as a layered transport upgrade, not a
simple hotspot rewrite.

The old `1/2/3` set remains valuable because it is simpler, autonomous, and
deterministic.

The new `4/5/6` set is better aligned with the current goal:

- Demo-ready behavior.
- Faster cloud sync when hotspot WiFi is available.
- Snappier controller-to-LilyGO communication through UART.
- BLE fallback for small local control messages.
- LTE fallback when all configured hotspots are down or WiFi cloud fails.
- Cleaner LCD presentation.
- Original baseline preserved.

The correct operating expectation is:

```text
4/5/6 should feel faster and more responsive than 1/2/3 when UART is wired and
hotspot WiFi is healthy.

1/2/3 should remain simpler and more self-contained when no external hotspot or
extra wiring should be trusted.
```

That is the real tradeoff.
