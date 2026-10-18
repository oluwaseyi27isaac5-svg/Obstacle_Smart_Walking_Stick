#include <LiquidCrystal.h>

// Ultrasonic sensor
const int trigPin = 9;
const int echoPin = 10;

// Updated LED pins
const int redLED = 6;
const int yellowLED = 5;
const int greenLED = 4;

// Buzzer
const int buzzerPin = 8;

// LCD: RS, E, D4, D5, D6, D7
LiquidCrystal lcd(12, 13, 2, 3, 7, 11);

unsigned long previousBeep = 0;
bool buzzerOn = false;

float getDistance() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  unsigned long duration =
      pulseIn(echoPin, HIGH, 30000UL);

  if (duration == 0) {
    return -1;
  }

  return duration * 0.0343 / 2.0;
}

void allLEDsOff() {
  digitalWrite(redLED, LOW);
  digitalWrite(yellowLED, LOW);
  digitalWrite(greenLED, LOW);
}

void displayStatus(const char* message, float cm) {
  lcd.setCursor(0, 0);
  lcd.print("                ");
  lcd.setCursor(0, 0);
  lcd.print(message);

  lcd.setCursor(0, 1);
  lcd.print("                ");
  lcd.setCursor(0, 1);

  if (cm >= 0) {
    lcd.print("Distance: ");
    lcd.print((int)cm);
    lcd.print(" cm");
  } else {
    lcd.print("Sensor check!");
  }
}

void beepPattern(unsigned long interval) {
  unsigned long now = millis();

  if (now - previousBeep >= interval) {
    previousBeep = now;
    buzzerOn = !buzzerOn;

    if (buzzerOn) {
      tone(buzzerPin, 2200);
    } else {
      noTone(buzzerPin);
    }
  }
}

void setup() {
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  pinMode(redLED, OUTPUT);
  pinMode(yellowLED, OUTPUT);
  pinMode(greenLED, OUTPUT);
  pinMode(buzzerPin, OUTPUT);

  lcd.begin(16, 2);

  lcd.setCursor(0, 0);
  lcd.print("SMART WALKING");
  lcd.setCursor(0, 1);
  lcd.print("STICK STARTING");

  Serial.begin(9600);
  delay(1500);
  lcd.clear();
}

void loop() {
  float distance = getDistance();

  allLEDsOff();

  if (distance < 0) {
    noTone(buzzerPin);
    buzzerOn = false;

    displayStatus("CHECK SENSOR", -1);
    Serial.println("No valid echo");
  }
  else if (distance <= 20) {
    digitalWrite(redLED, HIGH);

    tone(buzzerPin, 2500);
    buzzerOn = true;

    displayStatus("STOP! TOO CLOSE", distance);
  }
  else if (distance <= 50) {
    digitalWrite(redLED, HIGH);

    displayStatus("OBSTACLE NEAR", distance);
    beepPattern(150);
  }
  else if (distance <= 100) {
    digitalWrite(yellowLED, HIGH);

    displayStatus("CAUTION AHEAD", distance);
    beepPattern(500);
  }
  else {
    digitalWrite(greenLED, HIGH);

    noTone(buzzerPin);
    buzzerOn = false;

    displayStatus("PATH CLEAR", distance);
  }
  
  
  <img width="857" height="537" alt="Screenshot 2026-10-18 154004" src="https://github.com/user-attachments/assets/cc1632a8-7c6f-4c33-b4f1-cdbc13a742d6" />




  delay(60);
}
