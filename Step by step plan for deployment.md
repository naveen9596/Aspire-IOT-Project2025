Nice — below is a complete, practical **Purchase Order (BOM)** + **architecture** + **step-by-step build & deployment plan** for your **5 × NodeMCU temperature/humidity + relay** devices. I’ll also cover the DHT11 vs DHT22 choice, relay type guidance, safety & org approvals you *must* follow, wiring, firmware outline and test checklist.

I’m assuming:

* You will build **5 identical nodes** (one per bay/room).
* Each node will measure Temp & Humidity and has **relay(s)** to switch a load (e.g., control an adapter / secondary circuit).
* You will use **NodeMCU (ESP8266)**.

---

# 1) Quick recommendation (short)

* Sensor: **DHT22 (AM2302)** — better accuracy and range vs DHT11. Use DHT11 only for very low-cost, low-accuracy needs.
* Relay: Use **1-channel relay module (optocoupled) per device** unless you need to switch two independent circuits — then 2-channel. For AC mains switching of large loads **do not** directly control an AC split-phase/heavy AC unit with cheap relays — use contactors/SSR and a qualified electrician.
* Power: Use **dedicated 5V USB adapter (1A)** per NodeMCU; keep adapter separate if you plan to supply a controlled AC adapter in parallel.

---

# 2) Full Purchase Order (BOM) — for 5 devices (quantities & approximate INR prices)

> Prices are approximate ranges (India retail). Buy slightly extra spares (1–2 of items).

**Per node (×5)**

1. NodeMCU ESP8266 Dev Board — Qty 5 — **₹300–₹450 each**
2. DHT22 (AM2302) sensor — Qty 5 — **₹150–₹250 each** *(recommended)*

   * *Alternative:* DHT11 — **₹50–₹80** (but less accurate)
3. Relay module, 1-channel 5V (optocoupled recommended) — Qty 5 — **₹120–₹220 each**

   * *If you want one module to do 2 circuits:* 2-channel relay module — Qty 5 — **₹180–₹350 each**
4. 5V USB Adapter (5V, 1A or 2A) — Qty 5 — **₹120–₹250 each**
5. Female-male jumper wires / Dupont cables — kit — Qty 1 (spares) — **₹100–₹150**
6. Small perfboard or PCB shield (optional) for tidy soldering — Qty 5 — **₹50–₹150 each**
7. Screw terminal blocks (2-pin/3-pin) — for connecting relay load — Qty 10 — **₹6–₹20 each**
8. 10KΩ resistor (for DHT data line pull-up) — pack — **₹20–₹50**
9. Enclosure / project box (ABS) with wall-mount tabs — Qty 5 — **₹80–₹250 each**
10. Heat-shrink tubing, cable ties, mounting screws — small kit — **₹100–₹200**
11. Optional: 2A USB power splitter or centralized PSU (if you prefer one PSU) — Qty as needed — **₹400–₹1,200**
12. Optional safety: MCB (miniature circuit breaker) / fuse holder / quick disconnect — purchase per site — **₹150–₹500**
13. Optional: Opto-isolated SSR (for silent switching / AC loads) — per device (if required) — **₹700–₹1,500**
14. Labels / QR asset stickers — Qty 5 — **₹50–₹150**

**Spare/bench items**

* USB-to-TTL programmer (if required) — **₹150–₹350**
* Multimeter, insulation tape — if you don’t have — **₹400–₹1,200**

**Estimated total per node:** ~**₹900–₹1,700** (using DHT22 + 1-ch relay + adapter + enclosure)
**Total for 5 nodes:** ~**₹4,500–₹8,500** (plus optional safety hardware & spares)

---

# 3) Why DHT22 vs DHT11? (Short pros/cons)

* **DHT22 (AM2302)**

  * Temp range: −40 to +80°C, accuracy ±0.5°C (typical)
  * Humidity range: 0–100% RH, ±2–5% (better)
  * Slower sampling (~2s) but adequate for room HVAC
  * **Recommended** for AC monitoring
* **DHT11**

  * Temp: 0–50°C, ±2°C — poorer accuracy
  * Humidity: 20–80% ±5%
  * Cheaper, but not recommended when you care about accuracy or slightly wider temperature ranges.

=> **Pick DHT22**.

---

# 4) Relay selection notes & safety

* For switching **low-voltage DC** or control signal to an adapter: a **5V relay module** is fine.
* For switching **AC mains loads** (especially air conditioners or heavy equipment):

  * **Do not** use small cheap PCB relays to control mains compressors directly. Use a **proper contactor** sized for the AC current and have a certified electrician wire it.
  * For silent switching of resistive loads, SSRs are better; for inductive/compressor loads use mechanical contactor/relay with appropriate inrush rating.
* Always include a fuse / MCB, insulating terminal blocks, and isolate the high-voltage wiring from low-voltage electronics.
* Use **optocoupled** relay modules if you want better isolation between NodeMCU and mains.

---

# 5) High-level architecture (for your 5-node deployment)

```
[ NodeMCU + DHT22 + Relay  ]  (Device, in enclosure)
   - Measures temp/humidity
   - Controls Relay (1ch/2ch)
   - Wi-Fi → sends to Cloud (Blynk / ThingSpeak / Local server)

5 x devices  ──── Wi-Fi ────> Router/Office LAN
                                 |
                        ┌────────┴─────────┐
                        │ Blynk Cloud /    │
                        │ Local Server (MQTT / Node-RED / DB) │
                        └────────┬─────────┘
                                 |
                              Dashboard / Notifications / Admin
```

Per-device responsibilities:

* Read sensor every 30s–60s.
* Publish to cloud/MQTT.
* Evaluate thresholds locally (optional) to flip relay.
* Accept remote commands for relay via MQTT/Blynk.
* OTA updates (recommended).

---

# 6) Wiring & pin mapping (NodeMCU ESP8266 example)

**NodeMCU pin names vs DHT/Relay**

* DHT22 DATA → **D4** (GPIO2)

  * Place a 10K pull-up between DATA and 3.3V.
* DHT22 VCC → **3V3**
* DHT22 GND → **GND**
* Relay VCC → **5V or VIN** (check module voltage; many relay modules need 5V)

  * If relay requires 5V, power relay VCC from separate 5V adapter or from USB 5V (but check current). NodeMCU 5V pin (VIN) can supply limited current — better to drive relay VCC from the same 5V adapter but common ground.
* Relay IN → **D1 or D2 (GPIO5 / GPIO4)** (choose free GPIO)
* Relay GND → **GND**

**Wiring tips**

* Use common ground between NodeMCU and relay power supply.
* If relay module has JD-VCC and VCC jumper (for separate relay power) use the separate JD-VCC to remove coil noise from MCU power.
* Use screw terminals for mains connections; leave high-voltage wires inside the enclosure separated and marked.

---

# 7) Firmware / Software plan

### Required libraries (Arduino IDE)

* `ESP8266WiFi`
* `BlynkSimpleEsp8266` or `PubSubClient` (MQTT)
* `DHT` and `Adafruit_Sensor`
* `Blynk` or `ArduinoJson` (if needed)

### Behavior

* Connect to Wi-Fi on boot (store creds or use provisioning).
* Read sensor every 30 sec (DHT22 allows 2s minimum).
* Send telemetry to cloud (Blynk V0/V1 or MQTT topic).
* If temp > threshold (e.g., 27°C), set relay on/off based on your logic.
* Support remote override (dashboard).
* Implement debouncing (avoid frequent relay switching).
* Implement OTA updates (ESP8266 Arduino OTA or Blynk OTA).

### Example logic (pseudo)

```text
loop:
  if WiFi disconnected -> try connect
  every 30s:
    temp = readDHT()
    hum = readDHT()
    sendTelemetry(temp,hum)
    if localControl == true:
      if temp > HIGH_LIMIT and hysteresis passed:
         switch relay ON (enable cooling action)
      elif temp < LOW_LIMIT:
         switch relay OFF
    apply remote override commands from server
    save last update timestamp
```

### Simple safety in code

* Use **hysteresis** to avoid relay chattering (e.g., turn on at 27°C, turn off at 25°C).
* Minimum relay on/off time (e.g., 60s) to protect actuators.

---

# 8) Enclosure & installation guidance

* Use ABS project box with wall-mount tabs.
* Provide **vent hole** near the sensor so measurement reflects room air (not trapped heat).
* Provide **mains isolation** area (physical barrier) if relay switches mains.
* Label each device with **Asset ID** and room name.
* Place power adapter in accessible location or inside enclosure with ventilation.
* For room aesthetics, paint or pick neutral color and hide cables neatly.

---

# 9) Policies, approvals & safety checklist (must do before live install)

1. **Electrical safety sign-off** from Facilities/Electrical team if relay will touch mains.
2. **IT / Network registration** (MAC whitelisting) for 5 NodeMCU devices.
3. **Device naming & inventory**: record MAC, serial, device token, physical location.
4. **Change management**: maintenance windows for firmware updates.
5. **Data & privacy**: log retention of telemetry & access control for dashboards.
6. **Isolation & fuse**: ensure appropriate fuse/MCB for circuits controlled.
7. **Label high-voltage** inside enclosures.
8. **Insurance / Liability**: confirm with operations if controlling power to equipment is allowed.

---

# 10) Step-by-step build & deployment plan (milestones)

**Phase 0 — Procurement**

* Order components (BOM). Buy 1–2 spares of NodeMCU and relays.

**Phase 1 — Prototype (1 unit)**

* Assemble on breadboard/perfboard: NodeMCU + DHT22 + relay.
* Upload firmware: Wi-Fi connect, read DHT, relay toggle.
* Test relay switching with a low-voltage lamp or test load (not AC compressor).
* Implement hysteresis and safe timing.

**Phase 2 — Lab test & Safety**

* Replace test load with intended adapter. If switching AC mains, have electrician inspect wiring and test insulation.
* Test power stability for long runs (24–72 hrs).

**Phase 3 — Cloud/Dashboard**

* Create Blynk (free) template or MQTT + Node-RED + Grafana local stack.
* Add widgets to control/monitor each device (on/off, temp, hum, last seen).
* Configure alerts (push/email) on thresholds.

**Phase 4 — Pilot Deployment (2–3 devices)**

* Install 2–3 nodes in selected rooms.
* Monitor for 1–2 weeks: connectivity, false triggers, sensor accuracy.
* Refine thresholds and mounting positions.

**Phase 5 — Full Deployment (5 devices)**

* Install remaining units.
* Document installation & handover to operations team.
* Schedule routine audit/maintenance.

---

# 11) Example minimal Arduino code snippet (NodeMCU + DHT22 + Relay)

*(Sketch skeleton — change SSID, Blynk token or MQTT credentials as needed)*

```cpp
#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include "DHT.h"

#define DHTPIN D4
#define DHTTYPE DHT22
#define RELAY_PIN D1

DHT dht(DHTPIN, DHTTYPE);
char ssid[] = "YOUR_SSID";
char pass[] = "YOUR_PASS";
char auth[] = "BLYNK_DEVICE_TOKEN"; // if using Blynk

BlynkTimer timer;
float lastTemp = -100;
unsigned long lastSwitch = 0;
const int MIN_RELAY_INTERVAL = 60000; // 60s
const float ON_THRESHOLD = 27.0;
const float OFF_THRESHOLD = 25.0;

void sendSensor() {
  float t = dht.readTemperature();
  float h = dht.readHumidity();
  if (isnan(t) || isnan(h)) return;
  Blynk.virtualWrite(V0, t);
  Blynk.virtualWrite(V1, h);

  // Local hysteresis control
  unsigned long now = millis();
  if ( (t >= ON_THRESHOLD) && digitalRead(RELAY_PIN)==LOW && (now - lastSwitch) > MIN_RELAY_INTERVAL ){
    digitalWrite(RELAY_PIN, HIGH); // activate relay
    lastSwitch = now;
  } else if ( (t <= OFF_THRESHOLD) && digitalRead(RELAY_PIN)==HIGH && (now - lastSwitch) > MIN_RELAY_INTERVAL ){
    digitalWrite(RELAY_PIN, LOW);
    lastSwitch = now;
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(RELAY_PIN, OUTPUT);
  digitalWrite(RELAY_PIN, LOW);
  dht.begin();
  Blynk.begin(auth, ssid, pass);
  timer.setInterval(30000L, sendSensor); // every 30s
}

void loop() {
  Blynk.run();
  timer.run();
}
```

> NOTE: adapt relay logic for active-low modules (many relays are active LOW). Test with multimeter & a dummy load before connecting mains.

---

# 12) Final safety & regulatory notes (critical)

* **If relay switches mains, involve Facilities or licensed electrician** for wiring and final installation.
* Use enclosures with **sufficient creepage/clearance** between mains and low-voltage electronics.
* Add fuses / MCBs. **Do not** rely on the relay as the only safety device for mains.
* For permanent installation controlling HVAC / AC units, prefer integration with building management (BMS) or use the device only to send a control signal to the BMS, rather than cutting mains directly.

---

If you want, I can immediately provide the following next artifacts (pick any / multiple) and I’ll produce them right away:

A. A ready-to-purchase **shopping list** with item links (India / Amazon / Robu / DigiKey) for all BOM items.
B. A **wiring diagram PDF** (clear pinouts + relay wiring + safety notes).
C. A **production PCB / perfboard layout** and KiCad / Eagle starter file for cleaner builds.
D. A **full Arduino sketch** with Blynk + OTA + safe relay logic and configuration per device.
E. A **site safety checklist & sign-off template** for Facilities/IT/Legal.

Which one would you like first?
