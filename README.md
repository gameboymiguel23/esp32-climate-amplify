# ESP32 Climate Monitor (AWS IoT → DynamoDB → Dashboard + Discord)

## Case
En ESP32-enhet skickar temperatur och luftfuktighet till AWS. Data lagras i DynamoDB och visualiseras i en webbsida. Vid larm skickas notiser till Discord (extern API) via webhook.

---

## Systemskiss (komponenter och kopplingar)

[ESP32 Dev Module]
  - WiFi
  - MQTT över TLS (8883)
  - X.509 cert + privat nyckel (mTLS)
        |
        v
[AWS IoT Core]  (topic: esp32/climate)
   | \
   |  \__ IoT Rule #2 → [Lambda: DiscordAlert] → HTTPS POST → [Discord #iot-alerts]
   |
   \__ IoT Rule #1 → [DynamoDB: IoTClimateData]

[DynamoDB: IoTClimateData] ← Query ← [Lambda: GetLatestClimate (Function URL + CORS)] ← HTTPS fetch ← [S3 Static Website: Dashboard]

---

## Säkerhet (IoT Security)
- **Krypterad kommunikation:** ESP32 publicerar via **MQTT över TLS (port 8883)** till AWS IoT Core.
- **Certifikatsbaserad autentisering:** Enheten använder **X.509 certifikat + privat nyckel** (mTLS) för att autentisera sig mot AWS IoT.
- **Åtkomstkontroll:** AWS IoT Policy är kopplad till certifikatet/Thing och ger bara nödvändiga rättigheter (**least privilege**).
- **Tidsynk:** NTP används i ESP32 för att TLS-certifikat ska kunna valideras korrekt.

> OBS: Certifikat/private key och Discord webhook-URL är hemligheter och ska inte publiceras i GitHub.

---

## Kommunikation mellan sensor och gateway
- **Sensor/device:** ESP32 Dev Module (temperatur/fukt simuleras i koden).
- **Kommunikation:** WiFi → MQTT.
- **Gateway/ingång till molnet:** AWS IoT Core tar emot MQTT-meddelanden på topic `esp32/climate`.

Exempel payload som skickas:
```json
{"temperature": 22.30, "humidity": 54.80}
