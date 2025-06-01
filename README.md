 #include <SoftwareSerial.h>
#include <DHT.h>

const int gasPin = A0; // Gas sensor pin
const int flamePin = A1; // Flame sensor pin
const int LDR_PIN = 3; // LDR connected to digital pin 3
const int DHTPIN = 2; // DHT sensor connected to digital pin 2
const int LED_PIN = 9; // LED connected to digital pin 9
const int FAN_PIN = 10; // Fan connected to digital pin 10

int gasValue = 0; // Gas sensor value
int flameValue = 0; // Flame sensor value
int ldrValue = 0; // LDR value
float temperature = 0.0; // Temperature in Celsius

const int gasThreshold = 400; // Gas leakage threshold
const int flameThreshold = 400; // Flame detection threshold
const int tempThreshold = 30.0; // Temperature threshold in Celsius

SoftwareSerial mySerial(7, 8); // GSM module connected to digital pin 7 and 8
DHT dht(DHTPIN, DHT11); // DHT11 sensor

void setup() {
  mySerial.begin(9600);
  Serial.begin(9600);
  pinMode(gasPin, INPUT);
  pinMode(flamePin, INPUT);
  pinMode(LDR_PIN, INPUT);
  pinMode(LED_PIN, OUTPUT);
  pinMode(FAN_PIN, OUTPUT);
  pinMode(DHTPIN, INPUT);
  dht.begin();
  delay(100);
}

void loop() {
  // Gas sensor reading
  int sum = 0;
  for (int i = 0; i < 10; i++) {
    sum += analogRead(gasPin);
    delay(10);
  }
  gasValue = sum / 10;
  Serial.print("Gas Value: ");
  Serial.println(gasValue);

  // Flame sensor reading
  sum = 0;
  for (int i = 0; i < 10; i++) {
    sum += analogRead(flamePin);
    delay(10);
  }
  flameValue = sum / 10;
  Serial.print("Flame Value: ");
  Serial.println(flameValue);

  // LDR reading
  ldrValue = digitalRead(LDR_PIN);
  if (ldrValue == LOW) {
    digitalWrite(LED_PIN, LOW); // Turn on LED if it's dark
    Serial.println("Dark Mode: LED is ON");
  } else {
    digitalWrite(LED_PIN, HIGH); // Turn off LED if it's light
    Serial.println("Light Mode: LED is OFF");
  }

  // Temperature reading
  temperature = dht.readTemperature();
  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.println(" °C");

  // Gas leakage detection
  if (gasValue > gasThreshold) {
    Serial.println("Gas Leakage Detected!");
    sendMessage("GAS LEAKAGE ALERT! Immediate action required. Ensure safety precautions.");
    makeCall();
  }

  // Flame detection
  if (flameValue < flameThreshold) {
    Serial.println("Flame Detected!");
    sendMessage("FIRE ALERT! Fire detected in the industry. Danger situation! Take immediate action.");
    makeCall();
  }

  // Temperature based fan control
  if (temperature > tempThreshold) {
    digitalWrite(FAN_PIN, HIGH); // Turn on fan
  } else {
    digitalWrite(FAN_PIN, LOW); // Turn off fan
  }

  delay(1000);
}

void sendMessage(String message) {
  mySerial.println("AT+CMGF=1");
  delay(1000);
  mySerial.println("AT+CMGS=\"+919679841892\"\r");
  delay(1000);
  mySerial.println(message);
  delay(100);
  mySerial.println((char)26);
  delay(1000);
}

void makeCall() {
  Serial.println("Calling through GSM Module");
  delay(1000);
  mySerial.println("ATD9679841892;");
  delay(2000); // Wait for 2 seconds for call to connect
  mySerial.println("ATH"); // Hang up the call after 10 seconds
  delay(10000);
}
