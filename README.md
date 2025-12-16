ESP32 Climate Monitor (AWS IoT → DynamoDB → Dashboard + Discord)
Översikt

Detta projekt visar en komplett IoT-lösning där en ESP32 skickar temperatur- och luftfuktighetsdata till AWS via krypterad MQTT (TLS + certifikat). Data lagras i DynamoDB och visualiseras i en webbsida hostad i S3. För att uppfylla kravet på extern API-integration skickas även larmnotiser till Discord via webhook.

IoT Diagram

Översikt på IoT-flödet och hur komponenterna är sammankopplade:

ESP32 Dev Module
  └─ WiFi
  └─ MQTT över TLS (8883) + X.509 cert (mTLS)
        │
        ▼
AWS IoT Core  (topic: esp32/climate)
  ├─ IoT Rule #1 → DynamoDB (IoTClimateData)  [lagring]
  └─ IoT Rule #2 → Lambda (DiscordAlert) → Discord Webhook → #iot-alerts  [extern API]

Webbvisualisering:
S3 Static Website (Dashboard) → HTTPS fetch → Lambda Function URL (GetLatestClimate + CORS) → Query → DynamoDB

Introduktion

Målet med projektet är att skapa en enkel men robust IoT-arkitektur som kan:

upprätta kommunikation mellan sensor/device och gateway,

använda molntjänster för lagring och visualisering,

inkludera tydliga säkerhetsmekanismer (TLS + certifikat),

samt integrera data med ett externt API (Discord).

Den här typen av lösning kan användas för exempelvis miljöövervakning (temperatur/fukt), smarta hem eller enklare driftövervakning.

Komponenter
Hårdvara

ESP32 Dev Module

Programvara

Arduino IDE 2.x (seriell hastighet 115200)

ESP32-bibliotek: WiFi, WiFiClientSecure, PubSubClient

AWS-tjänster

AWS IoT Core (MQTT broker + Rules)

DynamoDB (lagring av mätvärden)

AWS Lambda

GetLatestClimate (hämtar senaste datapunkt till webben)

DiscordAlert (skickar larm till Discord)

S3 Static Website Hosting (dashboard/webb)

Externt API

Discord Webhook (push-notiser)

Dataformat (payload)

ESP32 skickar JSON till topic esp32/climate:

{"temperature": 22.30, "humidity": 54.80}

Databas (DynamoDB)

Tabell: IoTClimateData
Nycklar:

Partition key: deviceId (String)

Sort key: timestamp (Number, epoch ms)

Exempel item:

{
  "deviceId": "ESP32ClimateClient",
  "timestamp": 1765726138311,
  "temperature": 22.3,
  "humidity": 54.8
}

Steg-för-steg Instruktioner (hur jag byggde lösningen)
Steg 1: Programmera ESP32 (device)

ESP32 kopplas upp till WiFi.

TLS aktiveras med NTP-tidsynk (krävs för certifikatvalidering).

ESP32 ansluter till AWS IoT Core via MQTT över TLS (8883).

Enheten publicerar temperatur/fukt var ~10:e sekund till esp32/climate.

Steg 2: AWS IoT Core (gateway + routing)

Skapa en Thing och certifikat.

Koppla certifikat + policy (least privilege).

Verifiera att enheten kan publicera till topic esp32/climate.

Steg 3: Lagring i DynamoDB

Skapa tabellen IoTClimateData med deviceId + timestamp.

Skapa en IoT Rule som tar emot meddelanden från esp32/climate och skriver till DynamoDB.

Steg 4: API för webben (Lambda + Function URL)

Skapa Lambda GetLatestClimate som gör Query på DynamoDB och returnerar senaste mätningen.

Aktivera Function URL (Auth = NONE) och CORS så webbläsaren får hämta data.

Webbsidan gör fetch() mot Function URL och uppdaterar värden.

Steg 5: Visualisering i molnet (S3)

Skapa en S3-bucket och slå på Static Website Hosting.

Ladda upp index.html (dashboard).

Testa att sidan visar senaste temperatur och luftfuktighet.

Steg 6: Externt API (Discord webhook)

Skapa en Discord-server och kanal (t.ex. #iot-alerts).

Skapa en webhook i kanalen och kopiera webhook-URL.

Skapa Lambda DiscordAlert som skickar HTTP POST till webhook-URL när larmvillkor uppfylls.

Skapa IoT Rule #2 som triggar DiscordAlert på esp32/climate.

Säkerhet / IoT Security

MQTT över TLS (8883): Krypterad transport mellan ESP32 och AWS IoT Core.

mTLS med X.509 certifikat: Certifikatsbaserad autentisering (enheten identifieras med cert + private key).

Least privilege policy: IoT-policy ger bara nödvändiga rättigheter (t.ex. publish på ett topic).

NTP-tidsynk: Säkerställer att TLS-certifikat kan valideras korrekt.

Secrets hanteras säkert: Discord webhook-URL läggs som environment variable i Lambda och publiceras inte i GitHub.

Säkerhet / Skalbarhet (kort)

Lösningen kan skalas till fler enheter genom att skapa fler Things/certifikat.

DynamoDB skalar automatiskt och passar tidsseriedata (deviceId + timestamp).

IoT Rules + Lambda gör att man enkelt kan lägga fler integrationer (t.ex. SMS, e-post, fler API:er).

Kravcheck (för godkänt)

✅ Systemskiss som visar komponenter och kopplingar

✅ Säkerhetsmekanismer beskrivna (TLS + certifikat, policy)

✅ Data inhämtas från device (ESP32)

✅ Data skickas till AWS (IoT Core)

✅ Data lagras i databas (DynamoDB)

✅ Data pushas till externt API (Discord webhook)

✅ Data visualiseras i moln (S3 dashboard + Lambda API)

✅ Dokumentation i GitHub (denna README)

Screenshots (rekommenderas i repo)

Lägg gärna in skärmbilder som bevis:

Dashboard (S3) med aktuella värden

DynamoDB items som fylls på

IoT Rule-konfiguration

Discord-kanalen med mottagna larm

Slutsats

Projektet resulterade i en fungerande IoT-arkitektur där en ESP32 säkert kan skicka sensordata till AWS via TLS och certifikatbaserad autentisering. Data lagras i DynamoDB, visualiseras i en webbdashboard och kan även skickas vidare till ett externt API (Discord) för notiser. Lösningen är enkel att förstå men samtidigt byggd på samma principer som används i större IoT-system och kan skalas upp vid behov.

okej ska ja kopiera in allt det här du skicka till mig eller

Ja — kopiera in allt i mitt senaste meddelande (från raden “# ESP32 Climate Monitor…” ända ner till Slutsats).

Gör så här:

GitHub → README.md → Edit

Ctrl+A (markera allt) → Delete

Klistra in hela texten

Preview → ska se bra ut

Commit changes
