# IoT Data Visualization with ESP32, AWS, and Grafana

## Table of Contents
- [Overview](#overview)
- [Requirements](#requirements)
- [1. ESP32 and DHT11 Sensor Setup](#1-esp32-and-dht11-sensor-setup)
- [2. AWS IoT Core Setup](#2-aws-iot-core-setup)
- [3. Publish Data to AWS IoT Core](#3-publish-data-to-aws-iot-core)
- [4. Store Data in DynamoDB](#4-store-data-in-dynamodb)
- [5. Data Processing and Transformation with AWS Glue](#5-data-processing-and-transformation-with-aws-glue)
- [6. Visualize Data with AWS Athena and Grafana](#6-visualize-data-with-aws-athena-and-grafana)
- [Conclusion](#conclusion)
- [Troubleshooting and Optimization](#troubleshooting-and-optimization)
- [Security Enhancements](#security-enhancements)

---

## Overview
![System Overview](Images/Diagram.png)

*System overview of the IoT data visualization setup.*

This project demonstrates how to gather environmental data using an ESP32 microcontroller and a DHT11 sensor, transmit this data to AWS IoT Core, store it in DynamoDB, and visualize it using AWS Athena and Grafana.

---

## Requirements
- **Hardware**: ESP32 microcontroller, DHT11 temperature and humidity sensor.  
- **Software**: Arduino IDE, AWS account, Grafana account.  
- **Libraries**: ESP32 and DHT sensor libraries for Arduino IDE.

---

## 1. ESP32 and DHT11 Sensor Setup
![DHT11 Wiring](Images/Esp32dht11.png)

*Wiring diagram of DHT11 sensor with ESP32.*

The ESP32 is connected to a DHT11 sensor to collect temperature and humidity data. The sketch ([awsdht11.ino](Esp32/awsdht11.ino)) handles sensor data reading and prepares it for transmission.

### Wiring the Sensor
1. **VCC to ESP32**: Connect VCC pin of DHT11 to 3.3V.  
2. **GND to ESP32**: Connect GND pin to GND.  
3. **Data to ESP32**: Connect Data pin to GPIO 2 (D2).

Ensure secure connections to avoid wiring mistakes.

---

## 2. AWS IoT Core Setup
#### Thing Creation and Security
![Thing](Images/Thing.png)

Create a 'Thing' in AWS IoT Core representing the ESP32. Generate certificates and keys for secure connection.

#### Policy Attachment
Attach a policy to the certificate, allowing actions like publishing to MQTT topics.
![Policy](Images/Policy.png)

---

## 3. Publish Data to AWS IoT Core
The ESP32 uses MQTT protocol to send sensor data to AWS IoT Core. The sketch connects to Wi-Fi and publishes data to a designated topic.  
[View the ESP32 Sketch](Esp32/awsdht11.ino)

---

## 4. Store Data in DynamoDB
![DynamoDB Table](Images/DynamoDBTable.png)

Create a DynamoDB table to store sensor data. Use an AWS IoT Rule to trigger a Lambda function to insert new data into the table.  
![Lambda Setup](Images/Lambdasetup.png)

---

## 5. Data Processing and Transformation with AWS Glue
AWS Glue prepares and transforms IoT data for analysis.

### Setting Up AWS Glue
#### Data Catalog
Scan the DynamoDB table with a Glue Crawler to create a table in the Data Catalog.  
![GlueTable](Images/GlueTable.png)

#### ETL Jobs
Create ETL jobs to extract data from DynamoDB, transform it, and load it into S3.  
![GlueFlow](Images/GlueFlow.png)

#### AWS Glue Workflow
Automate crawling and ETL jobs with a Glue Workflow.

---

## 6. Visualize Data with AWS Athena and Grafana
![Grafana Dashboard](Images/Dashboard.png)

### Setting Up AWS Athena
Configure Athena to query DynamoDB data. Create a database and table reflecting the DynamoDB structure.  
![Athena Query](/Images/AthenaQuery.png)

### Integrating Athena with Grafana
Add Athena as a Grafana data source. Configure IAM roles to allow access.  
![Athena-Grafana Integration](/Images/Athena-Grafana.png)

### Creating a Grafana Dashboard
Build dashboards using queries from Athena. Create panels like Time Series graphs, Gauges, and Tables for temperature and humidity.  
![Grafana Setup](/Images/GrafanaSetup.png)

---

## Conclusion
This project demonstrates a complete IoT data pipeline from sensor to visualization. It highlights AWS capabilities for IoT and Grafana’s flexibility for dashboards.

---

## Troubleshooting and Optimization
- Use Serial Monitor for ESP32 debugging.  
- Check AWS permissions for IoT Core, DynamoDB, and Athena.  
- Optimize Athena queries for large datasets.  
- Verify data types and queries in Grafana.

---

## Security Enhancements
#### 1. Fine-Grained IoT Policies
Limit devices to minimum necessary actions on MQTT topics.

#### 2. Secure Device Provisioning
Use AWS IoT Device Defender and Just-in-Time Registration for secure registration.

#### 3. Encrypt Data
Use TLS for transit, AWS KMS for key management, and enable encryption at rest for DynamoDB and S3.

#### 4. Updates and Patching
Keep device firmware and software updated.

#### 5. Network Security
Use VPCs, security groups, and AWS WAF to restrict access.
