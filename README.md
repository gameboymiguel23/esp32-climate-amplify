# IoT Data Visualization with ESP32, AWS, and S3 Dashboard (+ Discord Alerts)

## Table of Contents
- [Overview](#overview)
- [Requirements](#requirements)
- [1. ESP32 Sensor Setup](#1-esp32-sensor-setup)
- [2. AWS IoT Core Setup](#2-aws-iot-core-setup)
- [3. Publish Data to AWS IoT Core](#3-publish-data-to-aws-iot-core)
- [4. Store Data in DynamoDB](#4-store-data-in-dynamodb)
- [5. Fetch Latest Data with AWS Lambda (Function URL)](#5-fetch-latest-data-with-aws-lambda-function-url)
- [6. Visualize Data with S3 Static Website](#6-visualize-data-with-s3-static-website)
- [7. External API Integration: Discord Webhook Alerts](#7-external-api-integration-discord-webhook-alerts)
- [Conclusion](#conclusion)
- [Troubleshooting and Optimization](#troubleshooting-and-optimization)
- [Security Enhancements](#security-enhancements)

---

## Overview
![image alt](https://github.com/gameboymiguel23/esp32-climate-amplify/blob/7eecfcc46b1260a819e42ff92f902ee97842723d/aws.png)

*System overview of the IoT data pipeline and visualization.*

This project demonstrates a complete IoT pipeline where an **ESP32 Dev Module** publishes **temperature and humidity** readings to **AWS IoT Core** using **MQTT over TLS (port 8883)** with **X.509 certificate authentication (mTLS)**.  
Data is stored in **DynamoDB**, the latest reading is retrieved via **AWS Lambda (Function URL + CORS)**, and a simple **web dashboard hosted in S3 Static Website** visualizes the data.  
To satisfy the “external API” requirement, alerts are also pushed to **Discord** via **Webhook** using a separate IoT Rule + Lambda.

---

## Requirements
- **Hardware**: ESP32 Dev Module  
- **Software**: Arduino IDE 2.x, AWS Account (eu-north-1)  
- **Arduino Libraries**:
  - `WiFi.h`
  - `WiFiClientSecure.h`
  - `PubSubClient.h`

---

## 1. ESP32 Sensor Setup
This project uses **simulated temperature and humidity** values in the ESP32 sketch (values change slightly over time).  
(You can replace simulation with a real sensor later without changing the AWS architecture.)

## Kommunikation mellan sensor och gateway
- **Sensor/device:** ESP32 Dev Module (temperatur/fukt simuleras i koden).
- **Kommunikation:** WiFi → MQTT.
- **Gateway/ingång till molnet:** AWS IoT Core tar emot MQTT-meddelanden på topic `esp32/climate`.

---

## 2. AWS IoT Core Setup (Gateway + säker anslutning)
1. Skapa en **Thing** i AWS IoT Core (t.ex. `ESP32ClimateClient`).
2. Skapa/aktivera **X.509 certifikat** och ladda ner:
   - Device certificate
   - Private key
   - Amazon Root CA
3. Skapa en **IoT Policy** (least privilege) som tillåter t.ex. `iot:Connect` och `iot:Publish` på ditt topic.
4. Koppla **policy → certifikat** och **certifikat → thing**.
5. Testa i AWS IoT “MQTT test client” att du ser messages på `esp32/climate`.

---

## 3. Publish Data to AWS IoT Core
1. ESP32 kopplar upp sig till WiFi.
2. ESP32 synkar tid via **NTP** (krävs för TLS-validering).
3. ESP32 ansluter till AWS IoT Core via MQTT över TLS på port **8883**.
4. Enheten publicerar data till topic `esp32/climate` var ~10:e sekund.
5. Verifiera i MQTT test client att datapaket kommer in.

---

## 4. Store Data in DynamoDB
1. Skapa tabellen **IoTClimateData**
   - Partition key: `deviceId` (String)
   - Sort key: `timestamp` (Number, epoch ms)
2. IoT Rule #1 skriver in i DynamoDB (antingen via DynamoDB action eller via ingest-Lambda).
3. Verifiera att det kommer nya rader var ~10:e sekund när ESP32 kör.

---

## 5. Fetch Latest Data with AWS Lambda (Function URL)
1. Lambda `GetLatestClimate` gör en Query i DynamoDB:
   - `deviceId = "ESP32ClimateClient"`
   - `ScanIndexForward: false`
   - `Limit: 1`
2. Aktivera **Function URL** (Auth = NONE) och **CORS** (Allow-Origin `*` räcker för kursprojekt).
3. Testa URL i webbläsaren → ska returnera senaste datapunkten som JSON.

---

## 6. Visualize Data with S3 Static Website
1. Skapa S3-bucket och slå på **Static website hosting**.
2. Lägg upp dashboard-sidan (HTML/JS) som gör `fetch()` mot Lambda Function URL.
3. Verifiera att sidan visar aktuell temperatur och luftfuktighet.

---

## 7. External API Integration: Discord Webhook Alerts
1. Skapa Discord-server + kanal (t.ex. `#iot-alerts`).
2. Skapa en **webhook** i kanalen och kopiera webhook-URL.
3. Lambda `DiscordAlert` skickar **HTTPS POST** till webhook-URL (lägg URL som env var `DISCORD_WEBHOOK_URL`).
4. IoT Rule #2 triggar `DiscordAlert` när nytt message kommer på `esp32/climate` (eller när larmvillkor uppfylls).
5. Verifiera att meddelanden dyker upp i Discord-kanalen.

---
![image alt](https://github.com/gameboymiguel23/esp32-climate-amplify/blob/7eecfcc46b1260a819e42ff92f902ee97842723d/aws.png)
## Conclusion
Projektet visar en komplett IoT-kedja där en ESP32 skickar sensordata säkert via MQTT/TLS till AWS IoT Core. Data lagras i DynamoDB, visualiseras i en webbdashboard och kan även pushas vidare till ett externt API (Discord) för notifieringar. Arkitekturen är enkel men följer riktiga IoT-principer och går att skala upp med fler enheter.

---

## Troubleshooting and Optimization
- Kontrollera Serial Monitor (115200) för debug på ESP32.
- Verifiera att IoT Policy tillåter `Connect` och `Publish` på rätt topic.
- Om TLS misslyckas: kontrollera NTP-tid och att rätt Root CA används.
- Om webben inte kan hämta data: kontrollera CORS på Lambda Function URL.
- Om DynamoDB är tom: kontrollera att IoT Rule är aktiv och matchar topic.

---

## Security Enhancements
- **Fine-Grained IoT Policies**: Begränsa policy till minsta möjliga (topic-specifikt).
- **TLS Everywhere**: MQTT över TLS samt HTTPS för API + Discord webhook.
- **Least Privilege IAM**: Lambda-rollen ska bara få `dynamodb:Query` (och ev. `dynamodb:PutItem` om ingest används).
- **Secrets Management**: Webhook-URL som Lambda environment variable (inte i GitHub).
- **Device Lifecycle**: Rotera certifikat vid behov och inaktivera certifikat vid förlorad enhet.
