Smart Indoor Plant Monitoring System with Edge AI
Overview

This repository contains the complete implementation of a Smart Indoor Plant Monitoring System developed using the Arduino UNO R4 WiFi. The project was designed and implemented as part of a Level 6 Internet of Things (IoT) module and addresses the real-world problem of monitoring indoor plant conditions and determining when watering is required.

The system combines sensor data acquisition, Edge AI (TinyML-style inference), Machine-to-Machine (M2M) communication, cloud data storage and analytics, and wireless user interaction through both WiFi and Bluetooth Low Energy (BLE).

Problem Statement

Indoor plants are often over-watered or under-watered due to a lack of real-time information about soil and environmental conditions. This project aims to solve this problem by continuously monitoring soil moisture, temperature, and humidity, and by intelligently deciding when a plant requires watering. The decision is made locally on the device to reduce latency and demonstrate Edge AI capabilities.

System Architecture

The system follows a hybrid Edge–Cloud IoT architecture:

Edge Layer
Arduino UNO R4 WiFi collects sensor data and performs on-device inference using a lightweight neural network model.

Communication Layer

WiFi is used to send data to the ThingSpeak cloud platform.

Bluetooth Low Energy (BLE) is used to broadcast live sensor data and system decisions to a mobile device.

Cloud Layer
ThingSpeak is used for data storage, visualisation, and triggering automated email alerts.

(Insert system architecture diagram here)

Hardware Components

I used the following hardware components:

Arduino UNO R4 WiFi

DHT22 temperature and humidity sensor

Soil moisture sensor

Breadboard and jumper wires

USB power supply

The DHT22 sensor is connected to a digital input pin for temperature and humidity readings, while the soil moisture sensor is connected to an analogue input pin to measure soil resistance.

(Insert circuit diagram here)

Software Components

The project software consists of:

Arduino firmware written in C/C++

ThingSpeak cloud platform (data storage and analytics)

BLE communication using the ArduinoBLE library

MATLAB Analysis and React on ThingSpeak for alert logic

Sensor Data Acquisition

The Arduino continuously reads:

Soil moisture (analogue value from the soil sensor)

Temperature (°C) from the DHT22

Humidity (%) from the DHT22

These readings form the input to both the rule-based logic and the Edge AI model.

Edge AI (TinyML-Style Inference)

To achieve Edge AI functionality, I trained a lightweight neural network model using real sensor data collected from the system. The model was trained offline and the resulting weights were exported and embedded directly into the Arduino firmware as a header file.

The neural network takes three inputs:

Soil moisture value

Temperature

Humidity

It outputs a probability indicating whether the plant requires watering.

All inference is performed locally on the Arduino, demonstrating Edge AI by:

Reducing dependency on cloud computation

Minimising latency

Improving reliability if internet connectivity is lost

Hybrid Decision Logic

To improve reliability, I implemented a hybrid decision approach:

Rule-based logic
If the soil moisture value exceeds a predefined threshold (dry soil), watering is required.

Machine learning logic
The neural network predicts the probability that watering is needed based on multiple environmental factors.

The final decision prioritises the soil moisture rule for clearly dry conditions, while the Edge AI model supports decision-making in borderline cases.

Machine-to-Machine (M2M) Communication

The project demonstrates M2M communication using two technologies:

WiFi
The Arduino automatically transmits sensor data and decision flags to the ThingSpeak cloud platform.

Bluetooth Low Energy (BLE)
The Arduino broadcasts live data to a smartphone using BLE. Data can be viewed using the nRF Connect mobile application without human intervention.

This satisfies the requirement for autonomous machine communication.

Cloud Integration (ThingSpeak)

ThingSpeak is used for:

Real-time data logging

Data visualisation using charts

Trigger-based automation

Field mapping:

Field 1 – Temperature

Field 2 – Humidity

Field 3 – Soil moisture

Field 4 – Needs-water flag

When Field 4 is set to 1, it indicates that the plant requires watering.

Automated Alerts

Email alerts are triggered through ThingSpeak React and MATLAB Analysis when the system detects that watering is required. This provides a practical notification mechanism for the user without requiring continuous manual monitoring.

Bluetooth Monitoring

Using BLE, I can monitor:

Soil moisture

Temperature

Humidity

Watering decision

Edge AI probability

The data is transmitted in real time and can be viewed on a mobile phone using the nRF Connect application.

(Insert BLE characteristic screenshot here)

Repository Structure
arduino/
 ├── smart_plant_iot.ino      # Final Arduino firmware
 ├── model_weights.h          # Embedded Edge AI model weights
README.md                     # Project documentation

How to Run the Project

Clone this repository

Open the Arduino sketch in the Arduino IDE

Connect the sensors to the Arduino UNO R4 WiFi

Configure WiFi credentials and ThingSpeak API keys

Upload the sketch to the board

View live data on ThingSpeak and via BLE on a mobile device

Educational Context

This project was developed to meet Level 6 learning outcomes in:

IoT system design

Edge AI integration

M2M communication

Cloud data analytics

It demonstrates both theoretical understanding and practical implementation of modern IoT and Edge AI systems.

Author

Zahid Ul Haq Chaudhary (250259117)
BSc (Hons) Computer Science
Level 6 – Internet of Things
