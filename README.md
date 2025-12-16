# IoT Climate Monitoring med ESP32 och AWS

## Innehållsförteckning
- [Översikt](#översikt)
- [Introduktion](#introduktion)
- [Komponenter och verktyg](#komponenter-och-verktyg)
- [Systemarkitektur](#systemarkitektur)
- [Steg-för-steg](#steg-för-steg)
  - [1. Förberedelser](#1-förberedelser)
  - [2. ESP32 → AWS IoT Core (MQTT/TLS)](#2-esp32--aws-iot-core-mqtttls)
  - [3. Lagring i DynamoDB](#3-lagring-i-dynamodb)
  - [4. API för webben (Lambda Function URL)](#4-api-för-webben-lambda-function-url)
  - [5. Webbdashboard (S3 Static Website)](#5-webbdashboard-s3-static-website)
  - [6. Externt API – Discord-notifieringar](#6-externt-api--discord-notifieringar)
- [Säkerhet och skalbarhet](#säkerhet-och-skalbarhet)
- [Kravcheck (kurskrav)](#kravcheck-kurskrav)
- [Slutsats](#slutsats)
- [Screenshots (valfritt)](#screenshots-valfritt)

---

## Översikt

Detta projekt är en komplett IoT-lösning byggd i AWS där en **ESP32** skickar klimatdata (temperatur och luftfuktighet) till molnet via **MQTT över TLS**.  
Datan lagras i **DynamoDB**, visualiseras via en **webbdashboard hostad i S3**, och används även för att skicka **notifieringar till Discord** genom ett externt API.

Projektet är färdigt, testat och uppfyller samtliga kurskrav.

---

## Introduktion

**Scenario:**  
En ESP32-enhet simulerar temperatur- och luftfuktighetsvärden och skickar dessa säkert till AWS IoT Core.  
AWS hanterar datan helt serverlöst och vidarebefordrar den till olika tjänster beroende på användningsområde.

**Funktioner:**
- Säker IoT-kommunikation (MQTT + TLS)
- Lagring i DynamoDB
- Serverlöst API via AWS Lambda
- Webbdashboard via S3 Static Website
- Externa notifieringar via Discord webhook

---

## Komponenter och verktyg

### Hårdvara
- ESP32 Dev Module

### Programvara & molntjänster
- Arduino IDE 2.x
- Serial Monitor (115200 baud)
- AWS IoT Core
- AWS DynamoDB
- AWS Lambda
- Amazon S3 (Static Website)
- Discord (Webhook – externt API)

---

## Systemarkitektur

