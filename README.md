# ESP32 Climate Monitor (AWS IoT Core → DynamoDB → Dashboard + Discord)

## Innehållsförteckning
- [Översikt](#översikt)
- [Introduktion](#introduktion)
- [Komponenter](#komponenter)
- [Systemskiss](#systemskiss)
- [Steg-för-steg Instruktioner](#steg-för-steg-instruktioner)
  - [Steg 1: Förberedelser](#steg-1-förberedelser)
  - [Steg 2: ESP32 → AWS IoT Core](#steg-2-esp32--aws-iot-core)
  - [Steg 3: Lagring i DynamoDB](#steg-3-lagring-i-dynamodb)
  - [Steg 4: API för webben (Lambda + Function URL)](#steg-4-api-för-webben-lambda--function-url)
  - [Steg 5: Visualisering (S3/Amplify)](#steg-5-visualisering-s3amplify)
  - [Steg 6: Extern API-integration (Discord)](#steg-6-extern-api-integration-discord)
- [Säkerhet/Skalbarhet](#säkerhetskalbarhet)
- [Kravcheck](#kravcheck)
- [Slutsats](#slutsats)

---

## Översikt
Detta projekt demonstrerar en komplett IoT-arkitektur där en ESP32 publicerar temperatur- och luftfuktighetsdata via MQTT över TLS till AWS IoT Core. Data lagras i DynamoDB och visualiseras i en webbdashboard. Vid larm skickas notiser till Discord via webhook (extern API).

> (Valfritt) Lägg in en bild på din skiss här om du har en:
>
> ![IoT Diagram](./img/iot-diagram.png)

---

## Introduktion
Målet är att bygga en enkel men säker IoT-lösning som uppfyller följande:
- Kommunikationen mellan device och gateway fungerar (ESP32 → AWS IoT Core).
- Molntjänster används för lagring och visualisering (DynamoDB + dashboard).
- Säkerhet ingår (kryptering, certifikatbaserad autentisering, policy/least privilege).
- Data skickas vidare till ett externt API (Discord webhook).

---

## Komponenter

### Hårdvara
- ESP32 Dev Module

### Sensor/data
- Temperatur/fukt **simuleras i koden** (kan bytas till riktig sensor senare)

### AWS
- AWS IoT Core (MQTT broker)
- IoT Rules (routing)
- DynamoDB-tabell: `IoTClimateData`
- Lambda:
  - `GetLatestClimate` (hämtar senaste datapunkt till dashboard)
  - `DiscordAlert` (skickar larm till Discord webhook)
- Hosting:
  - S3 Static Website **eller** Amplify (valfritt)

### Externt API
- Discord Webhook (pushnotiser)

---

## Systemskiss
Nedan är en textbaserad systemskiss (fungerar direkt i GitHub utan Mermaid-problem):

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

S3 Static Website (Dashboard) → HTTPS fetch → Lambda Function URL (GetLatestClimate + CORS) → Query → DynamoDB
