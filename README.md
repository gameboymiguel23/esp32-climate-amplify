# ESP32 Climate Monitor (AWS IoT Core → DynamoDB → Dashboard + Discord)

## Innehållsförteckning
- [Översikt](#översikt)
- [Introduktion](#introduktion)
- [Komponenter](#komponenter)
- [Systemskiss](#systemskiss)
- [Steg-för-steg instruktioner](#steg-för-steg-instruktioner)
  - [Steg 1: Förberedelser](#steg-1-förberedelser)
  - [Steg 2: ESP32 till AWS IoT Core](#steg-2-esp32-till-aws-iot-core)
  - [Steg 3: Lagring i DynamoDB](#steg-3-lagring-i-dynamodb)
  - [Steg 4: API för webben](#steg-4-api-för-webben)
  - [Steg 5: Visualisering](#steg-5-visualisering)
  - [Steg 6: Extern API-integration Discord](#steg-6-extern-api-integration-discord)
- [Säkerhet och skalbarhet](#säkerhet-och-skalbarhet)
- [Kravcheck](#kravcheck)
- [Slutsats](#slutsats)

---

## Översikt
Detta projekt demonstrerar en IoT-lösning där en ESP32 publicerar temperatur- och luftfuktighetsdata via MQTT över TLS till AWS IoT Core. Data lagras i DynamoDB och visualiseras i en webbdashboard. För att uppfylla kravet på extern API-integration skickas larmnotiser till Discord via webhook.

---

## Introduktion
Målet är att bygga en enkel men säker IoT-arkitektur som innehåller:
- Kommunikation mellan device och gateway (ESP32 → AWS IoT Core).
- Molnlagring (DynamoDB).
- Visualisering (S3/Amplify dashboard).
- Säkerhet (TLS + X.509 certifikat + IoT policy).
- Extern API-integration (Discord webhook).

---

## Komponenter

### Hårdvara
- ESP32 Dev Module

### Programvara
- Arduino IDE 2.x (Serial Monitor 115200)
- Bibliotek: `WiFi.h`, `WiFiClientSecure.h`, `PubSubClient.h`

### AWS
- AWS IoT Core
- IoT Rules
- DynamoDB: `IoTClimateData`
- Lambda: `GetLatestClimate`, `DiscordAlert`
- S3 Static Website (eller Amplify)

### Externt API
- Discord Webhook

---

## Systemskiss
```text
ESP32 Dev Module
  └─ WiFi
  └─ MQTT över TLS (8883) + X.509 cert (mTLS)
        │
        ▼
AWS IoT Core (topic: esp32/climate)
  ├─ IoT Rule #1 → DynamoDB (IoTClimateData) [lagring]
  └─ IoT Rule #2 → Lambda (DiscordAlert) → Discord Webhook → #iot-alerts [extern API]

Webbvisualisering:
Dashboard (S3/Amplify) → HTTPS fetch → Lambda Function URL (GetLatestClimate + CORS) → Query → DynamoDB
