# ESP32 Climate Monitor (AWS IoT → DynamoDB → Dashboard + Discord)

## Case
En ESP32-enhet skickar temperatur och luftfuktighet till AWS. Data lagras och visualiseras i en webbsida. Vid larm skickas notiser till Discord (extern API).

## Systemskiss (komponenter och kopplingar)

flowchart LR
  A[ESP32 Dev Module\nTemp/Hum (simulerad)] -->|MQTT over TLS 8883\nX.509 cert| B[AWS IoT Core]
  B -->|IoT Rule #1: DynamoDB action| C[(DynamoDB\nIoTClimateData)]
  B -->|IoT Rule #2: Invoke Lambda| D[Lambda: DiscordAlert]
  D -->|HTTPS POST\nDiscord Webhook| E[Discord #iot-alerts]

  F[Lambda: GetLatestClimate\n(Function URL + CORS)] -->|Query latest| C
  G[S3 Static Website\nDashboard (HTML/JS)] -->|HTTPS fetch| F
