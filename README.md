# ESP32 Climate Monitor (AWS IoT → DynamoDB → Dashboard + Discord)

## Översikt
Detta projekt visar en komplett IoT-lösning där en ESP32 skickar temperatur- och luftfuktighetsdata till AWS via krypterad MQTT (TLS + certifikat). Data lagras i DynamoDB och visualiseras i en webbsida hostad i S3. För att uppfylla kravet på extern API-integration skickas även larmnotiser till Discord via webhook.

---

## IoT Diagram
Översikt på IoT-flödet och hur komponenterna är sammankopplade:

```text
ESP32 Dev Module
  └─ WiFi
  └─ MQTT över TLS (8883) + X.509 cert (mTLS)
        │
        ▼
AWS IoT Core (topic: esp32/climate)
  ├─ IoT Rule #1 → DynamoDB (IoTClimateData)  [lagring]
  └─ IoT Rule #2 → Lambda (DiscordAlert) → Discord Webhook → #iot-alerts  [extern API]

Webbvisualisering:
S3 Static Website (Dashboard) → HTTPS fetch → Lambda Function URL (GetLatestClimate + CORS) → Query → DynamoDB
