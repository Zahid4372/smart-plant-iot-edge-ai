/*
  ✅ FINAL COMPLETE PROJECT (UNO R4 WiFi)

  - Sensors: DHT22 + Soil Moisture
  - Edge AI: Manual NN inference (TinyML-style)
  - Decision Logic: HARD soil rule + ML confirmation
  - BLE: Live data to phone (nRF Connect)
  - Cloud: ThingSpeak (Fields 1–4)
  - Email: ThingSpeak React (Field 4 == 1)
*/

#include <Arduino.h>
#include <WiFiS3.h>
#include <ThingSpeak.h>
#include <ArduinoBLE.h>
#include <DHT.h>
#include "model_weights.h"

// =====================
// WIFI / THINGSPEAK
// =====================
const char* WIFI_SSID = "Galaxy A14 FE30";
const char* WIFI_PASS = "11111111";

WiFiClient client;

unsigned long CHANNEL_ID = 3225753;
const char* WRITE_API_KEY = "NFV5FVC0DFB58AYC";

const unsigned long TS_INTERVAL_MS = 20000;
unsigned long lastTsMs = 0;

// =====================
// SENSORS
// =====================
#define DHTPIN 2
#define DHTTYPE DHT22
DHT dht(DHTPIN, DHTTYPE);

const int SOIL_PIN = A0;

// =====================
// DECISION THRESHOLDS
// =====================
const int   SOIL_DRY_THRESHOLD = 620;
const float ML_PROB_THRESHOLD  = 0.00025f;

// =====================
// BLE
// =====================
BLEService plantService("19B10000-E8F2-537E-4F6C-D104768A1214");

BLEIntCharacteristic   soilChar      ("19B10001-E8F2-537E-4F6C-D104768A1214", BLERead | BLENotify);
BLEFloatCharacteristic tempChar      ("19B10002-E8F2-537E-4F6C-D104768A1214", BLERead | BLENotify);
BLEFloatCharacteristic humChar       ("19B10003-E8F2-537E-4F6C-D104768A1214", BLERead | BLENotify);
BLEByteCharacteristic  needsWaterChar("19B10004-E8F2-537E-4F6C-D104768A1214", BLERead | BLENotify);
BLEFloatCharacteristic probChar      ("19B10005-E8F2-537E-4F6C-D104768A1214", BLERead | BLENotify);

// =====================
// EDGE AI (MANUAL NN)
// =====================
static inline float relu(float x) { return x > 0 ? x : 0; }
static inline float sigmoid(float x) { return 1.0f / (1.0f + expf(-x)); }

float modelProb(int soil, float temp, float hum) {
  float x0 = (soil - FEATURE_MEANS[0]) / FEATURE_STDS[0];
  float x1 = (temp - FEATURE_MEANS[1]) / FEATURE_STDS[1];
  float x2 = (hum  - FEATURE_MEANS[2]) / FEATURE_STDS[2];

  float h[8];
  for (int j = 0; j < 8; j++) {
    float sum = B1_ML[j];
    sum += x0 * W1[0 * 8 + j];
    sum += x1 * W1[1 * 8 + j];
    sum += x2 * W1[2 * 8 + j];
    h[j] = relu(sum);
  }

  float out = B2_ML[0];
  for (int j = 0; j < 8; j++) out += h[j] * W2[j];

  return sigmoid(out);
}

// =====================
// ✅ FIXED HYBRID DECISION LOGIC
// =====================
int computeNeedsWater(int soil, float prob) {
  // HARD physical rule first
  if (soil < SOIL_DRY_THRESHOLD) {
    return 0;  // ❗ NEVER water if soil < threshold
  }

  // ML only used when soil is already dry
  return (prob >= ML_PROB_THRESHOLD) ? 1 : 0;
}

// =====================
// WIFI CONNECT
// =====================
void connectWiFi() {
  Serial.print("Connecting WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    WiFi.begin(WIFI_SSID, WIFI_PASS);
    delay(2000);
    Serial.print(".");
  }
  Serial.println("\n✅ WiFi Connected");
}

// =====================
// SETUP
// =====================
void setup() {
  Serial.begin(9600);
  delay(1500);

  dht.begin();
  connectWiFi();
  ThingSpeak.begin(client);

  BLE.begin();
  BLE.setLocalName("UNO_R4_BLE");
  BLE.setAdvertisedService(plantService);

  plantService.addCharacteristic(soilChar);
  plantService.addCharacteristic(tempChar);
  plantService.addCharacteristic(humChar);
  plantService.addCharacteristic(needsWaterChar);
  plantService.addCharacteristic(probChar);

  BLE.addService(plantService);
  BLE.advertise();

  Serial.println("✅ System Ready (Sensors + Edge AI + BLE + ThingSpeak)");
}

// =====================
// LOOP
// =====================
void loop() {
  BLE.poll();

  int soil = analogRead(SOIL_PIN);
  float hum = dht.readHumidity();
  float temp = dht.readTemperature();

  if (isnan(hum) || isnan(temp)) return;

  float prob = modelProb(soil, temp, hum);
  int needsWater = computeNeedsWater(soil, prob);

  int soilRule = (soil >= SOIL_DRY_THRESHOLD);
  int mlRule   = (prob >= ML_PROB_THRESHOLD);

  Serial.print("Soil=");
  Serial.print(soil);
  Serial.print(" Temp=");
  Serial.print(temp,2);
  Serial.print(" Hum=");
  Serial.print(hum,2);
  Serial.print(" | ML prob=");
  Serial.print(prob,6);
  Serial.print(" | soilRule=");
  Serial.print(soilRule);
  Serial.print(" | mlRule=");
  Serial.print(mlRule);
  Serial.print(" => needsWater=");
  Serial.println(needsWater);

  soilChar.writeValue(soil);
  tempChar.writeValue(temp);
  humChar.writeValue(hum);
  needsWaterChar.writeValue((byte)needsWater);
  probChar.writeValue(prob);

  unsigned long now = millis();
  if (now - lastTsMs >= TS_INTERVAL_MS) {
    lastTsMs = now;

    ThingSpeak.setField(1, temp);
    ThingSpeak.setField(2, hum);
    ThingSpeak.setField(3, soil);
    ThingSpeak.setField(4, needsWater);

    int resp = ThingSpeak.writeFields(CHANNEL_ID, WRITE_API_KEY);
    if (resp == 200) {
      Serial.print("✅ ThingSpeak OK | EmailTrigger=");
      Serial.println(needsWater);
    }
  }

  delay(2000);
}
