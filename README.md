# FogGuard-Ai-2026
FogGuard AI - SIH 2026
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

// =====================================================
// FOGGUARD AI - FINAL PROTOTYPE
// ESP32 + L298N + 4WD + HC-SR04 + OLED + LEDs + Buzzer
// =====================================================


// ---------------- MOTOR PINS ----------------

// LEFT SIDE
#define LEFT_IN1 26
#define LEFT_IN2 27

// RIGHT SIDE
#define RIGHT_IN1 14
#define RIGHT_IN2 12


// ---------------- ULTRASONIC ----------------

#define TRIG_PIN 5
#define ECHO_PIN 18


// ---------------- LEDs ----------------

#define GREEN_LED 25
#define YELLOW_LED 33
#define RED_LED 32


// ---------------- BUZZER ----------------

#define BUZZER 23


// ---------------- OLED ----------------

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64

Adafruit_SSD1306 display(
  SCREEN_WIDTH,
  SCREEN_HEIGHT,
  &Wire,
  -1
);


// ---------------- DISTANCE LIMITS ----------------

#define STOP_DISTANCE 20
#define CAUTION_DISTANCE 50


// =====================================================
// SETUP
// =====================================================

void setup() {

  Serial.begin(115200);

  // Motor pins
  pinMode(LEFT_IN1, OUTPUT);
  pinMode(LEFT_IN2, OUTPUT);

  pinMode(RIGHT_IN1, OUTPUT);
  pinMode(RIGHT_IN2, OUTPUT);


  // Ultrasonic
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);


  // LEDs
  pinMode(GREEN_LED, OUTPUT);
  pinMode(YELLOW_LED, OUTPUT);
  pinMode(RED_LED, OUTPUT);


  // Buzzer
  pinMode(BUZZER, OUTPUT);


  // Stop motors when ESP32 starts
  stopMotors();


  // OLED
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {

    Serial.println("OLED NOT FOUND");

  } else {

    display.clearDisplay();

    display.setTextColor(SSD1306_WHITE);

    display.setTextSize(2);
    display.setCursor(5, 5);
    display.println("FOGGUARD");

    display.setTextSize(1);
    display.setCursor(30, 35);
    display.println("AI SYSTEM");

    display.display();

    delay(2000);
  }


  Serial.println();
  Serial.println("==============================");
  Serial.println("       FOGGUARD AI");
  Serial.println("==============================");
  Serial.println("SYSTEM READY");
  Serial.println();
}


// =====================================================
// MAIN LOOP
// =====================================================

void loop() {

  // Read obstacle distance
  float distance = getDistance();


  // Print distance
  Serial.print("Obstacle Distance: ");

  if (distance > 0) {

    Serial.print(distance);
    Serial.println(" cm");

  } else {

    Serial.println("No object detected");
  }


  // ===================================================
  // DANGER
  // ===================================================

  if (distance > 0 && distance <= STOP_DISTANCE) {

    stopMotors();

    digitalWrite(GREEN_LED, LOW);
    digitalWrite(YELLOW_LED, LOW);
    digitalWrite(RED_LED, HIGH);

    digitalWrite(BUZZER, HIGH);

    showOLED(
      "DANGER",
      "STOP",
      distance
    );

    Serial.println("!!! COLLISION RISK !!!");
    Serial.println("!!! VEHICLE STOPPED !!!");
  }


  // ===================================================
  // CAUTION
  // ===================================================

  else if (
    distance > STOP_DISTANCE &&
    distance <= CAUTION_DISTANCE
  ) {

    // Move forward
    forward();

    digitalWrite(GREEN_LED, LOW);
    digitalWrite(YELLOW_LED, HIGH);
    digitalWrite(RED_LED, LOW);

    digitalWrite(BUZZER, LOW);

    showOLED(
      "CAUTION",
      "SLOW",
      distance
    );

    Serial.println("CAUTION - OBJECT NEAR");
  }


  // ===================================================
  // SAFE
  // ===================================================

  else {

    forward();

    digitalWrite(GREEN_LED, HIGH);
    digitalWrite(YELLOW_LED, LOW);
    digitalWrite(RED_LED, LOW);

    digitalWrite(BUZZER, LOW);

    showOLED(
      "SAFE",
      "PATH CLEAR",
      distance
    );

    Serial.println("PATH CLEAR - MOVING");
  }


  delay(300);
}


// =====================================================
// FORWARD
// =====================================================

void forward() {

  // LEFT SIDE FORWARD
  digitalWrite(LEFT_IN1, HIGH);
  digitalWrite(LEFT_IN2, LOW);


  // RIGHT SIDE FORWARD
  digitalWrite(RIGHT_IN1, HIGH);
  digitalWrite(RIGHT_IN2, LOW);
}


// =====================================================
// BACKWARD
// =====================================================

void backward() {

  // LEFT SIDE BACKWARD
  digitalWrite(LEFT_IN1, LOW);
  digitalWrite(LEFT_IN2, HIGH);


  // RIGHT SIDE BACKWARD
  digitalWrite(RIGHT_IN1, LOW);
  digitalWrite(RIGHT_IN2, HIGH);
}


// =====================================================
// TURN LEFT
// =====================================================

void turnLeft() {

  // LEFT SIDE BACKWARD
  digitalWrite(LEFT_IN1, LOW);
  digitalWrite(LEFT_IN2, HIGH);


  // RIGHT SIDE FORWARD
  digitalWrite(RIGHT_IN1, HIGH);
  digitalWrite(RIGHT_IN2, LOW);
}


// =====================================================
// TURN RIGHT
// =====================================================

void turnRight() {

  // LEFT SIDE FORWARD
  digitalWrite(LEFT_IN1, HIGH);
  digitalWrite(LEFT_IN2, LOW);


  // RIGHT SIDE BACKWARD
  digitalWrite(RIGHT_IN1, LOW);
  digitalWrite(RIGHT_IN2, HIGH);
}


// =====================================================
// STOP
// =====================================================

void stopMotors() {

  digitalWrite(LEFT_IN1, LOW);
  digitalWrite(LEFT_IN2, LOW);

  digitalWrite(RIGHT_IN1, LOW);
  digitalWrite(RIGHT_IN2, LOW);
}


// =====================================================
// ULTRASONIC DISTANCE
// =====================================================

float getDistance() {

  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);


  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);


  digitalWrite(TRIG_PIN, LOW);


  long duration = pulseIn(
    ECHO_PIN,
    HIGH,
    30000
  );


  // No echo
  if (duration == 0) {

    return -1;
  }


  // Calculate distance
  float distance =
    duration * 0.0343 / 2;


  return distance;
}


// =====================================================
// OLED DISPLAY
// =====================================================

void showOLED(
  String status,
  String action,
  float distance
) {

  display.clearDisplay();

  display.setTextColor(SSD1306_WHITE);


  // Status
  display.setTextSize(2);

  display.setCursor(0, 0);

  display.println(status);


  // Distance
  display.setTextSize(1);

  display.setCursor(0, 30);

  display.print("Distance: ");


  if (distance > 0) {

    display.print(distance, 1);
    display.println(" cm");

  } else {

    display.println("--");
  }


  // Action
  display.setCursor(0, 48);

  display.print("Action: ");

  display.println(action);


  display.display();
}




