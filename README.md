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
- **Software**: Arduino IDE for programming the ESP32, AWS account for cloud services, and a Grafana account for data visualization.

---

## 1. ESP32 and DHT11 Sensor Setup
![DHT11 Wiring](Images/Esp32dht11.png)

*Wiring diagram of DHT11 sensor with ESP32.*

The ESP32 is connected to a DHT11 sensor to collect temperature and humidity data. The sketch ([awsdht11.ino](Esp32/awsdht11.ino)), ([awsconnection.h](Esp32/awsconnection.h)) handles sensor data reading and prepares it for transmission. It includes necessary libraries and defines connection parameters for AWS IoT Core.

### Wiring the Sensor
To wire the DHT11 sensor to the ESP32, follow these steps:
1. **VCC to ESP32**: Connect the VCC pin of the DHT11 sensor to a 3.3V pin on the ESP32.
2. **GND to ESP32**: Connect the GND pin of the DHT11 sensor to a GND pin on the ESP32.
3. **Data to ESP32**: Connect the data pin (out) of the DHT11 sensor to a GPIO pin on the ESP32. In this project, we use GPIO 2 (D2 pin in many ESP32 boards).

Ensure the connections are secure and double-check each connection to avoid any short circuits or wiring mistakes.

---

## 2. AWS IoT Core Setup
#### Thing Creation and Security
![Thing](Images/Thing.png)

In AWS IoT Core, create a 'Thing' representing the ESP32. This 'Thing' acts as a digital twin and helps manage device-specific settings and states. Generate security certificates and keys for the ESP32. These are essential for authenticating and establishing a secure connection with AWS IoT Core.

#### Policy Attachment
Attach a policy to the device's certificate. This policy specifies the permissions, like publishing to specific MQTT topics, ensuring secure and authorized communication.
![Policy](Images/Policy.png)

---

## 3. Publish Data to AWS IoT Core
The ESP32 uses MQTT protocol to send sensor data to AWS IoT Core. The sketch includes functionality to connect to Wi-Fi, establish a secure connection with AWS IoT Core, and publish data to a designated MQTT topic. [View the ESP32 Sketch](Esp32/awsdht11.ino)

---

## 4. Store Data in DynamoDB
![DynamoDB Table](Images/DynamoDBTable.png)

*Configuration of the DynamoDB table for storing IoT data.*

Create a table in DynamoDB with a suitable schema to store sensor data. DynamoDB provides a fast, flexible NoSQL database solution that scales with the amount of IoT data.
![Create Table](/Images/CreateTable.png)

#### Data Insertion via AWS Lambda
![Lambda Setup](Images/Lambdasetup.png)

Use an AWS IoT Rule to trigger a Lambda function whenever new data is published to the MQTT topic. The Lambda function ([Function.py](Lambda/Function.py)) processes this data and inserts it into the DynamoDB table.

---

## 5. Data Processing and Transformation with AWS Glue
AWS Glue plays a pivotal role in this project by preparing and transforming the IoT data for efficient analysis. 

### Setting Up AWS Glue
#### Data Catalog
AWS Glue Data Catalog stores metadata and schema definitions for the data in DynamoDB, making it accessible and queryable. Configure a Glue Crawler to scan the DynamoDB table and create a corresponding table in the Data Catalog.
![GlueTable](Images/GlueTable.png)

#### ETL Jobs
Create an ETL (Extract, Transform, Load) job in AWS Glue to process and transform the raw IoT data. The ETL job will extract data from DynamoDB, transform it into a query-optimized format, and load it into an S3 bucket.
![GlueFlow](Images/GlueFlow.png)

#### AWS Glue Workflow
Set up a workflow in AWS Glue that automates the crawling and ETL processes, ensuring that the data is regularly updated and transformed for analysis.


---

## 6. Visualize Data with AWS Athena and Grafana
![Grafana Dashboard](Images/Dashboard.png)

*Sample Grafana dashboard visualizing IoT data.*

### Setting Up AWS Athena
Configure AWS Athena to query the data stored in DynamoDB. This involves setting up a database and a table in Athena that mirrors the structure of the DynamoDB table.
![Athena Query](/Images/AthenaQuery.png)

### Integrating Athena with Grafana
Add Athena as a data source in Grafana. This requires configuring IAM roles and permissions to allow Grafana to access Athena and fetch the data.
![Athena Integration with Grafana](/Images/Athena-Grafana.png)

### Creating a Grafana Dashboard
Build a Grafana dashboard to visualize the data. Use SQL queries to retrieve data from Athena and create panels like Time Series graphs, Gauge, Tables to display temperature and humidity over time.
![Grafana Setup](/Images/GrafanaSetup.png)

---

## Conclusion
This project provides a comprehensive example of integrating various technologies for IoT data collection, storage, and visualization. It demonstrates the capabilities of AWS in handling IoT applications and the flexibility of Grafana for data analysis and dashboard creation.

---

## Troubleshooting and Optimization
- **Debugging Connectivity**: Use Serial Monitor for ESP32 troubleshooting.
- **AWS Permissions**: Ensure correct configurations for IoT Core, DynamoDB, and Athena.
- **Athena Query Optimization**: Consider data partitioning for large datasets.
- **Grafana Visualization**: Verify data types and query formats for visualizations.

---

### Security Enhancements

#### 1. **Fine-Grained IoT Policies**
- Implement more specific AWS IoT Core policies. Limit devices to the minimum necessary actions, such as restricting each device to only its MQTT topics.

#### 2. **Secure Device Provisioning**
- Use AWS IoT Device Defender to continuously monitor and audit your IoT configurations.
- Consider using Just-in-Time Registration (JITR) for dynamic and secure device registration.

#### 3. **Encrypt Data**
- Ensure all data in transit is encrypted using TLS.
- Use AWS KMS (Key Management Service) for encryption keys management and enable encryption at rest for DynamoDB and S3.

#### 4. **Regular Updates and Patching**
- Keep the device firmware and software up-to-date with the latest security patches.

#### 5. **Network Security**
- Employ VPCs, security groups, and network ACLs to restrict network access.
- Use AWS WAF (Web Application

