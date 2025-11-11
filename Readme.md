Below is a complete, ready-to-use Blueprint for your Blynk IoT Dashboard + ESP32 sensors + Arduino code, specifically for your 4 prototype locations:

Libra Bay

Hydra Meeting Hall

Cafeteria

Meeting Room Training Bay

This includes:
✔ Architecture Blueprint
✔ Wiring Diagram
✔ ESP32/ESP8266 Arduino Firmware
✔ Recommended Blynk Datastreams
✔ Mobile + Web Dashboard Layout Template
✔ Device Naming + Template Setup

🚀 1. SYSTEM BLUEPRINT (High-Level Architecture)
┌─────────────────────────────┐
│        ESP32 / ESP8266       │
│   + DHT22 Temp/Humidity      │
│ Connect Wi-Fi → Blynk Cloud  │
└──────────────┬──────────────┘
               │ Wi-Fi
               ▼
        ┌──────────────┐
        │  Blynk Cloud │
        └───────┬──────┘
                │
     ┌──────────┴─────────┐
     │                    │
┌─────────────┐    ┌──────────────┐
│Mobile App   │    │Web Dashboard │
│(Android/iOS)│    │(blynk.cloud) │
└─────────────┘    └──────────────┘

🧩 2. WIRING DIAGRAM (ESP32 + DHT22)
DHT22 Sensor           ESP32
------------------------------
VCC  → 3.3V
GND  → GND
DATA → GPIO 4


(For ESP8266 NodeMCU use D4 instead of GPIO 4)

Add a 10K pull-up resistor between DATA ↔ VCC (recommended for stable readings).

⚙️ 3. BLYNK TEMPLATE SETUP
Template Name: TempMonitorV1
Template ID: (Auto generated in Blynk)
Device Name Examples:

Libra Bay

Hydra Meeting Hall

Cafeteria

Training Bay Meeting Room

🎛 Datastreams in Blynk Template
Datastream	Type	Pin	Description
Temp	Double	V0	Temperature °C
Humid	Double	V1	Humidity %
RoomName	String	V2	Location name
LastUpdate	String	V3	Last updated time
HeatAlert	Integer	V4	1/0 (for alerts if temp > limit)
📱 4. DASHBOARD (WEB + MOBILE) DESIGN
Web Dashboard Layout
Widgets:

2-Gauge Widgets

Gauge 1 → Temperature °C (V0)

Gauge 2 → Humidity % (V1)

Label Widget

Room Name (V2)

Chart / History Graph

Plot V0 (Temp) + V1 (Humidity)

LED Widget

Red LED if temperature > 27°C
(linked to V4)

Event Timeline

Logs when temp crosses threshold

Mobile App Dashboard

Widgets (in order):

Styled Value

TEMP (°C) → V0

HUMIDITY (%) → V1

Gauge

Temperature with color zones (20°C–30°C)

Graph

6h / 12h / 24h history

Device Selector (Optional)
Lets you switch between the 4 locations.

🧪 5. ARDUINO CODE (COMPLETE, READY TO UPLOAD)

ESP32 Version (change Wi-Fi + Blynk creds).

#define BLYNK_TEMPLATE_ID "YOUR_TEMPLATE_ID"
#define BLYNK_TEMPLATE_NAME "TempMonitorV1"
#define BLYNK_AUTH_TOKEN "DEVICE_TOKEN"

#include <WiFi.h>
#include <WiFiClient.h>
#include <BlynkSimpleEsp32.h>
#include "DHT.h"

char ssid[] = "YOUR_WIFI_NAME";
char pass[] = "YOUR_WIFI_PASS";

#define DHTPIN 4
#define DHTTYPE DHT22
DHT dht(DHTPIN, DHTTYPE);

BlynkTimer timer;

void sendSensorData() {
  float temp = dht.readTemperature();
  float humid = dht.readHumidity();

  if (isnan(temp) || isnan(humid)) {
    Serial.println("Sensor read error!");
    return;
  }

  // Push to Blynk
  Blynk.virtualWrite(V0, temp);
  Blynk.virtualWrite(V1, humid);

  // Room Name
  Blynk.virtualWrite(V2, "Libra Bay"); // Change per device

  // Timestamp
  Blynk.virtualWrite(V3, String(millis()/1000) + " sec");

  // Alert
  if (temp > 27) {
    Blynk.virtualWrite(V4, 1);
    Blynk.logEvent("overheat", "Temperature High!");
  } else {
    Blynk.virtualWrite(V4, 0);
  }

  Serial.print("Temp: ");
  Serial.print(temp);
  Serial.print(" °C  | Humidity: ");
  Serial.println(humid);
}

void setup() {
  Serial.begin(115200);
  dht.begin();
  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);
  timer.setInterval(30000L, sendSensorData);
}

void loop() {
  Blynk.run();
  timer.run();
}

🏷 6. PER-DEVICE CONFIGURATION
Device 1: Libra Bay

V2 = "Libra Bay"

Token = Device Token #1

Device 2: Hydra Meeting Hall

V2 = "Hydra Meeting Hall"

Token = Device Token #2

Device 3: Cafeteria

V2 = "Cafeteria"

Token = Device Token #3

Device 4: Training Bay Meeting Room

V2 = "Training Bay Meeting Room"

Token = Device Token #4

(Each device must have its own Blynk Device Token.)

📡 7. ALERTS & AUTOMATIONS
In Blynk → Automations:

IF Temperature > 27°C
THEN

Send Push Notification

Switch LED Widget ON

Log Event "overheat"

📦 8. BLUEPRINT DOCUMENT (Summary Text You Can Use in Proposal)

“The AC Temperature Monitoring Prototype uses four ESP32 IoT nodes installed at Libra Bay, Hydra Meeting Hall, Cafeteria, and Meeting Room Training Bay. Each node collects temperature and humidity using DHT22 sensors and transmits securely via Wi-Fi to the Blynk IoT Cloud.
A unified Web and Mobile dashboard visualizes live data, historical trends, and sends alerts when temperature crosses thresholds. The system supports device-wise identity, event logs, and real-time monitoring.”

🎁 Next Output I Can Generate For You

I can create:

✔ Full Visio Architecture Diagram (PNG/PDF)
✔ Blynk Dashboard Screenshots (Template Layout)
✔ 3D Case Recommendation for the 4 devices
✔ BoM (Bill of Materials) for procurement
✔ Project proposal / documentation for your manager
