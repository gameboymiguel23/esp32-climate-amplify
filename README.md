# ESP32 Climate Monitor (AWS IoT → DynamoDB → Dashboard + Discord)

## Case
En ESP32-enhet skickar temperatur och luftfuktighet till AWS. Data lagras och visualiseras i en webbsida. Vid larm skickas notiser till Discord (extern API).

## Systemskiss (komponenter och kopplingar)

flowchart LR
  A["ESP32 Dev Module<br/>Temp/Hum (simulerad)"] -->|"MQTT over TLS 8883<br/>X.509 cert"| B["AWS IoT Core"]
  B -->|"IoT Rule #1: DynamoDB action"| C[("DynamoDB<br/>IoTClimateData")]
  B -->|"IoT Rule #2: Invoke Lambda"| D["Lambda: DiscordAlert"]
  D -->|"HTTPS POST<br/>Discord Webhook"| E["Discord #iot-alerts"]

  F["Lambda: GetLatestClimate<br/>(Function URL + CORS)"] -->|"Query latest"| C
  G["S3 Static Website<br/>Dashboard (HTML/JS)"] -->|"HTTPS fetch"| F



## Säkerhet (IoT Security)
- **Krypterad kommunikation:** ESP32 publicerar via **MQTT över TLS (port 8883)** till AWS IoT Core.
- **Certifikatsbaserad autentisering:** Enheten använder **X.509 certifikat + privat nyckel** (mTLS) för att autentisera sig mot AWS IoT.
- **Åtkomstkontroll:** AWS IoT Policy är kopplad till certifikatet/Thing och ger bara nödvändiga rättigheter (least privilege).
- **Tidsynk:** NTP används i ESP32 för att TLS-certifikat ska kunna valideras korrekt.

## Kommunikation mellan sensor och gateway
- **Sensor/device:** ESP32 Dev Module (temperatur/fukt simuleras i koden).
- **Kommunikation:** WiFi → MQTT.
- **Gateway/ingång till molnet:** AWS IoT Core tar emot MQTT-meddelanden på topic `esp32/climate`.

Exempel payload som skickas:
```json
{"temperature": 22.30, "humidity": 54.80}
