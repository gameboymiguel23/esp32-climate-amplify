flowchart LR
  A["ESP32 Dev Module<br/>Temp/Hum (simulerad)"] -->|"MQTT over TLS 8883<br/>X.509 cert (mTLS)"| B["AWS IoT Core"]
  B -->|"IoT Rule #1: DynamoDB action"| C[("DynamoDB<br/>IoTClimateData")]
  B -->|"IoT Rule #2: Invoke Lambda"| D["Lambda: DiscordAlert"]
  D -->|"HTTPS POST<br/>Discord Webhook"| E["Discord #iot-alerts"]
  F["Lambda: GetLatestClimate<br/>(Function URL + CORS)"] -->|"Query latest"| C
  G["S3 Static Website<br/>Dashboard (HTML/JS)"] -->|"HTTPS fetch"| F

