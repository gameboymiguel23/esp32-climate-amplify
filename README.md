# ESP32 Climate Monitor (AWS IoT Core + DynamoDB + Dashboard + Discord)

## Innehållsförteckning
- [Översikt](#översikt)
- [Introduktion](#introduktion)
- [Komponenter](#komponenter)
- [IoT Diagram](#iot-diagram)
- [Steg-för-steg Instruktioner](#steg-för-steg-instruktioner)
  - [Steg 1: Förberedelser](#steg-1-förberedelser)
  - [Steg 2: ESP32 till AWS IoT Core](#steg-2-esp32-till-aws-iot-core)
  - [Steg 3: Lagring i DynamoDB](#steg-3-lagring-i-dynamodb)
  - [Steg 4: API för webben Lambda + Function URL](#steg-4-api-för-webben-lambda--function-url)
  - [Steg 5: Visualisering S3 eller Amplify](#steg-5-visualisering-s3-eller-amplify)
  - [Steg 6: Extern API integration Discord](#steg-6-extern-api-integration-discord)
- [Säkerhet och Skalbarhet](#säkerhet-och-skalbarhet)
- [Kravcheck](#kravcheck)
- [Slutsats](#slutsats)

---

## Översikt
Detta projekt visar en komplett IoT-lösning där en **ESP32** skickar temperatur- och luftfuktighetsdata till **AWS** via **MQTT över TLS** (port 8883) med **X.509 certifikat** (mTLS).  
Data lagras i **DynamoDB** och visualiseras i en webbdashboard (S3/Amplify).  
För att uppfylla kravet på extern API-integration skickas även notiser till **Discord** via **webhook**.

---

## Introduktion
Målet med projektet är att bygga en IoT-arkitektur som kan:
- upprätta kommunikation mellan sensor/device och gateway (ESP32 → AWS IoT Core)
- lagra data i en molndatabas (DynamoDB)
- visualisera data via molntjänst (S3/Amplify dashboard)
- inkludera säkerhet (TLS + certifikat + least privilege)
- skicka data/notiser till ett externt API (Discord webhook)

---

## Komponenter

### Hårdvara
- ESP32 Dev Module

### Programvara
- Arduino IDE 2.x (Serial Monitor 115200)
- Bibliotek: `WiFi.h`, `WiFiClientSecure.h`, `PubSubClient.h`

### AWS tjänster
- AWS IoT Core (MQTT broker + cert/policy)
- IoT Rules (routing)
- DynamoDB (tabell: `IoTClimateData`)
- AWS Lambda:
  - `GetLatestClimate` (hämtar senaste värde till webben)
  - `DiscordAlert` (skickar notiser till Discord)
- S3 Static Website Hosting (eller Amplify)

### Extern tjänst
- Discord Webhook (extern API)

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
  ├─ IoT Rule #1 → DynamoDB (IoTClimateData) [lagring]
  └─ IoT Rule #2 → Lambda (DiscordAlert) → Discord Webhook → #iot-alerts [extern API]

Webbvisualisering:
Dashboard (S3/Amplify) → HTTPS fetch → Lambda Function URL (GetLatestClimate + CORS) → Query → DynamoDB

Dashboard (S3/Amplify) → HTTPS fetch → Lambda Function URL (GetLatestClimate + CORS) → Query → DynamoDB
