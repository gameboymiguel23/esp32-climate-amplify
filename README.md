# Konfigurera AWS IoT Core för ESP32 (temperatur/fukt) + DynamoDB + Dashboard + Discord

## Innehållsförteckning
- [Översikt](#översikt)
- [Introduktion](#introduktion)
- [Komponenter](#komponenter)
- [IoT Diagram](#iot-diagram)
- [Steg-för-steg Instruktioner](#steg-för-steg-instruktioner)
  - [Steg 1: Nedladdningar](#steg-1-nedladdningar)
  - [Steg 2: Uppkoppling](#steg-2-uppkoppling)
  - [Steg 3: Lagring i databas (DynamoDB)](#steg-3-lagring-i-databas-dynamodb)
  - [Steg 4: API och hämtning av data (Lambda + Function URL)](#steg-4-api-och-hämtning-av-data-lambda--function-url)
  - [Steg 5: Visualisering (S3/Amplify)](#steg-5-visualisering-s3amplify)
  - [Steg 6: Externt API (Discord Webhook)](#steg-6-externt-api-discord-webhook)
- [Säkerhet/Skalbarhet](#säkerhetskalbarhet)
- [Slutsats](#slutsats)

---

## Översikt
![IoT Diagram](./img/iot-diagram.png)

*Översikt på diagrammet för IoT-flödet i AWS: ESP32 → AWS IoT Core → DynamoDB → Dashboard, samt Discord-notiser via webhook.*

---

## Introduktion
Detta projekt beskriver hur man bygger en säker och fungerande IoT-lösning med en **ESP32 Dev Module** som skickar temperatur- och luftfuktighetsdata till **AWS IoT Core** via **MQTT över TLS (8883)** med **certifikatsbaserad autentisering (X.509 / mTLS)**.  
Data lagras i **DynamoDB** och presenteras i en **webbdashboard** hostad i **S3 (Static Website)**.  
För att uppfylla kravet på extern API-integration pushas larm/meddelanden vidare till **Discord** via webhook.

Den här lösningen kan användas för att övervaka miljöförhållanden i realtid, t.ex. i rum, förråd eller serverutrymmen.

---

## Komponenter
- AWS-konto (region: eu-north-1 / Stockholm).
- **AWS IoT Core** (MQTT broker + Things/certifikat + IoT Rules).
- **ESP32 Dev Module** (Arduino IDE 2.x).
- (Sensor-data): Temperatur och luftfuktighet **simuleras** i koden (kan ersättas med riktig sensor).
- **DynamoDB** tabell: `IoTClimateData`.
- **AWS Lambda**:
  - `GetLatestClimate` (hämtar senaste datapunkt från DynamoDB).
  - `DiscordAlert` (skickar notiser till Discord).
- **S3 Static Website** (dashboard / index.html).
- **Discord Webhook** (extern API).

![ESP32 Image](./img/esp32.png)

*ESP32 setup*

---

## IoT Diagram
*Hur komponenterna integreras med molnet:*

```text
ESP32 Dev Module
  └─ WiFi
  └─ MQTT över TLS (8883) + X.509 cert (mTLS)
        │
        ▼
AWS IoT Core (topic: esp32/climate)
  ├─ IoT Rule #1 → DynamoDB (IoTClimateData) [lagring]
  └─ IoT Rule #2 → Lambda (DiscordAlert) → Discord Webhook → #iot-alerts [extern API]

Dashboard:
S3 Static Website → HTTPS fetch → Lambda Function URL (GetLatestClimate + CORS) → Query → DynamoDB

