# Nano Body Driver Rover

## V.1.0

### PlatformIO

```ini
[env:nanoatmega328]
platform = atmelavr
board = nanoatmega328
framework = arduino

; ตั้งค่าความเร็ว Serial Monitor ให้ตรงกับในโค้ด (115200)
monitor_speed = 115200

; หากคุณใช้บอร์ด Nano งานจีนส่วนใหญ่จะเป็น Bootloader แบบใหม่ 
; แต่ถ้าอัปโหลดไม่เข้า ให้ลองเปลี่ยน board เป็น nanoatmega328new
```

### main.cpp

```cpp
#include <Arduino.h>
#include <stdint.h>

// --- PIN DEFINITIONS ---
// Motor Driver (L298N)
const int ENA = 5;
const int IN1 = 7;
const int IN2 = 8;
const int ENB = 6;
const int IN3 = 9;
const int IN4 = 10;

// Peripheral Control
const int PIN_HEADLIGHT = A3;
const int PIN_CAM_POWER = A4;
const int PIN_L_SIGNAL = A0; 
const int PIN_R_SIGNAL = A1; 
const int PIN_BRAKE = 13;
const int PIN_REVERSE = A2;

// Ultrasonic Hub (V4.2)
struct ObstacleMap {
  uint8_t header;      // 0xCC
  uint16_t front_cm;
  uint16_t left_cm;
  uint16_t right_cm;
  uint16_t rear_cm;
  uint8_t checksum;
};
const int US_F_TRIG = 2, US_F_ECHO = 3;
const int US_L_TRIG = 4, US_L_ECHO = 11;
const int US_R_TRIG = 12, US_R_ECHO = A5; 

// --- STATE ---
uint8_t prevLights = 0;

void stopMotors();
void driveMotorLeft(int speed);
void driveMotorRight(int speed);

void setup() {
  Serial.begin(115200); // Link from STM32

  // Motor & Logic Pins
  pinMode(ENA, OUTPUT); pinMode(IN1, OUTPUT); pinMode(IN2, OUTPUT);
  pinMode(ENB, OUTPUT); pinMode(IN3, OUTPUT); pinMode(IN4, OUTPUT);
  pinMode(PIN_HEADLIGHT, OUTPUT); pinMode(PIN_CAM_POWER, OUTPUT);
  pinMode(PIN_L_SIGNAL, OUTPUT); pinMode(PIN_R_SIGNAL, OUTPUT);
  pinMode(PIN_BRAKE, OUTPUT); pinMode(PIN_REVERSE, OUTPUT);

  // Ultrasonic Pins
  pinMode(US_F_TRIG, OUTPUT); pinMode(US_F_ECHO, INPUT);
  pinMode(US_L_TRIG, OUTPUT); pinMode(US_L_ECHO, INPUT);
  pinMode(US_R_TRIG, OUTPUT); pinMode(US_R_ECHO, INPUT);

  digitalWrite(PIN_CAM_POWER, HIGH);
  stopMotors();
}

uint16_t readUS(int trig, int echo) {
  digitalWrite(trig, LOW); delayMicroseconds(2);
  digitalWrite(trig, HIGH); delayMicroseconds(10);
  digitalWrite(trig, LOW);
  long duration = pulseIn(echo, HIGH, 30000); 
  return (duration == 0) ? 999 : (duration * 0.034 / 2);
}

void loop() {
  // 1. COMMAND LISTENER (Serial from STM32)
  if (Serial.available() >= 5) {
    if (Serial.read() == 0xFF) {
      int8_t leftSpeed = (int8_t)Serial.read();
      int8_t rightSpeed = (int8_t)Serial.read();
      uint8_t lights = (uint8_t)Serial.read();
      uint8_t audioId = (uint8_t)Serial.read();

      driveMotorLeft(leftSpeed);
      driveMotorRight(rightSpeed);

      digitalWrite(PIN_HEADLIGHT, (lights & 0x01) ? HIGH : LOW);
      digitalWrite(PIN_BRAKE, (leftSpeed == 0 && rightSpeed == 0) ? HIGH : LOW);
      digitalWrite(PIN_REVERSE, (leftSpeed < 0 || rightSpeed < 0) ? HIGH : LOW);
      
      prevLights = lights;
    }
  }

  // 2. SENSOR HUB LOOP (5Hz)
  static uint32_t lastUS = 0;
  if (millis() - lastUS > 200) {
    lastUS = millis();
    ObstacleMap mapData;
    mapData.header = 0xCC;
    mapData.front_cm = readUS(US_F_TRIG, US_F_ECHO);
    mapData.left_cm = readUS(US_L_TRIG, US_L_ECHO);
    mapData.right_cm = readUS(US_R_TRIG, US_R_ECHO);
    mapData.rear_cm = 0;

    uint8_t *ptr = (uint8_t *)&mapData;
    uint8_t sum = 0;
    for (int i = 0; i < sizeof(mapData) - 1; i++) sum ^= ptr[i];
    mapData.checksum = sum;
    Serial.write((uint8_t *)&mapData, sizeof(mapData));
  }
}

void driveMotorLeft(int speed) {
  int pwm = map(abs(speed), 0, 100, 0, 255);
  digitalWrite(IN1, (speed >= 0) ? HIGH : LOW);
  digitalWrite(IN2, (speed >= 0) ? LOW : HIGH);
  analogWrite(ENA, pwm);
}

void driveMotorRight(int speed) {
  int pwm = map(abs(speed), 0, 100, 0, 255);
  digitalWrite(IN3, (speed >= 0) ? HIGH : LOW);
  digitalWrite(IN4, (speed >= 0) ? LOW : HIGH);
  analogWrite(ENB, pwm);
}

void stopMotors() {
  analogWrite(ENA, 0); analogWrite(ENB, 0);
  digitalWrite(IN1, LOW); digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW); digitalWrite(IN4, LOW);
}
```
