#define BLYNK_PRINT Serial
#define BLYNK_TEMPLATE_ID "TMPL6ZOr93Q8a"
#define BLYNK_TEMPLATE_NAME "Automatic Environmental Monitoring System"
#define BLYNK_AUTH_TOKEN "OxCxbm5HyMpDd3O5qITobHhDUXieic2t"

#include <Wire.h>
#include <Adafruit_Sensor.h>
#include <Adafruit_BMP280.h>
#include <Adafruit_AHTX0.h>
#include <BlynkSimpleEsp32.h>
#include <WiFi.h>

// Initialize BMP280 and AHT20
Adafruit_BMP280 bmp;
Adafruit_AHTX0 aht;

// Define component pins
#define HUMIDIFIER_PIN 13 // Humidifier pin
#define FAN_PIN 14 // Fan pin
#define VPIN_HUMIDIFIER V2 // Virtual pin for humidifier control
#define VPIN_FAN V3 // Virtual pin for fan control
#define VPIN_MODE V4 // Virtual pin for mode control

BlynkTimer timer;

// Variables to track sensor states
int humidifierState = LOW;
int fanState = LOW;
bool isAutoMode = true; // Variable to track the mode (automatic/manual)

void setup() {
  Serial.begin(115200);

  pinMode(HUMIDIFIER_PIN, OUTPUT);
  pinMode(FAN_PIN, OUTPUT);

  if (!bmp.begin()) { // Check if BMP280 sensor is connected
    Serial.println("Could not find a valid BMP280 sensor, check wiring!");
    while (1);
  }

  if (!aht.begin()) { // Check if AHT20 sensor is connected
    Serial.println("Could not find a valid AHT20 sensor, check wiring!");
    while (1);
  }

  Blynk.begin(BLYNK_AUTH_TOKEN, "Test-Project", "Test-Project");

  // Call functions
  timer.setInterval(1000L, readSensors);
  timer.setInterval(100L, checkHumidifierControl);
  timer.setInterval(100L, checkFanControl);
}

void loop() {
  // Run Blynk
  Blynk.run();
  // Run timer
  timer.run();
}

void readSensors() {
  float temp = bmp.readTemperature();
  sensors_event_t humidityEvent;
  aht.getEvent(NULL, &humidityEvent);

  float humidity = humidityEvent.relative_humidity;

  if (isnan(temp) || isnan(humidity)) {
    Serial.println("Failed to read from sensors!");
    return;
  }

  // Write temperature and humidity to Blynk
  Blynk.virtualWrite(V0, temp);
  Blynk.virtualWrite(V1, humidity);

  Serial.print("Temperature: ");
  Serial.print(temp);
  Serial.print(" *C, Humidity: ");
  Serial.print(humidity);
  Serial.println(" %");
}

void checkHumidifierControl() {
  if (isAutoMode) {
    sensors_event_t humidityEvent;
    aht.getEvent(NULL, &humidityEvent);
    float humidity = humidityEvent.relative_humidity;

    if (humidity < 40.0) {
      digitalWrite(HUMIDIFIER_PIN, HIGH);
      humidifierState = HIGH;
    } else {
      digitalWrite(HUMIDIFIER_PIN, LOW);
      humidifierState = LOW;
    }

    Blynk.virtualWrite(VPIN_HUMIDIFIER, humidifierState);
  }
}

void checkFanControl() {
  if (isAutoMode) {
    float temp = bmp.readTemperature();

    if (temp > 33.0) {
      digitalWrite(FAN_PIN, HIGH);
      fanState = HIGH;
    } else {
      digitalWrite(FAN_PIN, LOW);
      fanState = LOW;
    }

    Blynk.virtualWrite(VPIN_FAN, fanState);
  }
}

BLYNK_WRITE(VPIN_HUMIDIFIER) {
  if (!isAutoMode) {
    humidifierState = param.asInt();
    digitalWrite(HUMIDIFIER_PIN, humidifierState);
  }
}

BLYNK_WRITE(VPIN_FAN) {
  if (!isAutoMode) {
    fanState = param.asInt();
    digitalWrite(FAN_PIN, fanState);
  }
}

BLYNK_WRITE(VPIN_MODE) {
  isAutoMode = param.asInt(); // Read the state of the mode control button
  Serial.print("Auto Mode: ");
  Serial.println(isAutoMode ? "Enabled" : "Disabled");
}

