# 🦾 Module: Body (ระบบขับเคลื่อนและเซนเซอร์รอบตัว)

โมดูลนี้ทำหน้าที่เป็น "กล้ามเนื้อ" และ "สัญชาตญาณ" พื้นฐานของหุ่นยนต์

- **Movement & Power:** ควบคุมมอเตอร์ (Movement) และระบบไฟส่องสว่างกำลังสูง (High-Intensity LED / Searchlight)
- **SOS Actuator:** ทำหน้าที่ส่งสัญญาณฉุกเฉินผ่านเสียง (Buzzer) และไฟกะพริบ (Strobe Light) เมื่อได้รับคำสั่งหรือตรวจพบผู้รอดชีวิต
- **Boundary & Patrol:** รองรับโหมด "Area Patrol" เพื่อลาดตระเวนในรัศมี และ "Auto RTH" เมื่อหลุดเขต
- **Local Initiative (Emergency):** หากสูญเสียการติดต่อกับ Header หรือตรวจพบวัตถุประชิดในที่มืด Nano สามารถตัดสินใจเปิดไฟเองได้ทันที (Local Override)
- จัดการโหมดการควบคุมระหว่าง Standalone (Bluetooth) และ System (Header)

## 🛠️ Hardware Requirements
- **MCU:** Arduino Nano (V3)
- **Drivers:** L298N หรือ MOSFET Modules
- **Sensors:** 4-6x Ultrasonic (HC-SR04), Bluetooth HC-05 (Optional)

## ⚙️ Control Priority Logic
1. **Standalone Mode:** ฟังคำสั่งจาก Bluetooth (HC-05) เมื่อไม่มีการเชื่อมต่อกับ Header
2. **Override Mode:** เมื่อมีสัญญาณจาก **Signal Check Pin** (Header เสียบอยู่) ให้ตัดการทำงาน Bluetooth ทันทีและรับคำสั่งจาก UART (Header) เท่านั้น

## 📍 Reserved Pins (Arduino Nano)
| Function           | Nano Pin         | Note                            |
| :----------------- | :--------------- | :------------------------------ |
| **Motor L PWM**    | D5, D6           | คุมความเร็ว/ทิศทาง                 |
| **Motor R PWM**    | D9, D10          | คุมความเร็ว/ทิศทาง                 |
| **UART (Header)**  | D0 (RX), D1 (TX) | รับคำสั่งจาก STM32                  |
| **Header Check**   | D2               | High = Header Attached          |
| **Lipo Monitor**   | A0               | สำหรับวัดไฟแบตเตอรี่ (Propulsion)    |
| **Searchlight**    | D3               | **Illumination:** ต่อผ่าน MOSFET  |
| **Buzzer**         | D4               | **SOS Audio:** ส่งสัญญาณเสียงกู้ภัย   |
| **Digital Strobe** | D7               | **SOS Visual:** ไฟกะพริบแจ้งตำแหน่ง |
| **Light FX**       | D11              | ระบบไฟสัญญาณ/ไฟเท่                |

---

## 💻 Latest Firmware: Arduino Nano Actuator

โค้ดนี้ใช้สำหรับ Arduino Nano (Body) เพื่อควบคุมการขับเคลื่อนและระบบสัญญาณไฟกู้ภัย

```cpp
#include <Arduino.h>

// Motor Driver Pins
const int ENA = 5;  const int IN1 = 7;  const int IN2 = 8;
const int ENB = 6;  const int IN3 = 9;  const int IN4 = 10;

// SOS & FX Pins
const int PIN_BUZZER = 4;
const int PIN_STROBE = 7;
const int PIN_LIGHTS = 3;

void setup() {
  Serial.begin(115200);
  pinMode(ENA, OUTPUT); pinMode(IN1, OUTPUT); pinMode(IN2, OUTPUT);
  pinMode(ENB, OUTPUT); pinMode(IN3, OUTPUT); pinMode(IN4, OUTPUT);
  pinMode(PIN_BUZZER, OUTPUT); pinMode(PIN_STROBE, OUTPUT);
  pinMode(PIN_LIGHTS, OUTPUT);
  
  Serial.println("Ghost Micro V5: Body Actuator Ready");
}

void loop() {
  if (Serial.available() >= 6) { // Updated to 6 bytes for flags
    if (Serial.read() == 0xAF) { // Signature
      int8_t throttle = Serial.read();
      int8_t steering = Serial.read();
      byte flags = Serial.read(); // Bit 0: RTH, Bit 1: Patrol
      byte sos = Serial.read();
      byte lights = Serial.read();

      // Handle Flags & Failsafe
      bool isRTH = flags & 0x01;
      bool isPatrol = flags & 0x02;

      if (isRTH) {
        // Execute Home-coming logic (e.g. follow path back)
        executeRTH();
      } else if (isPatrol) {
        // Execute Area Patrol logic within radius
        executePatrol();
      } else {
        // Simple Differential Drive
        int left = throttle + steering;
        int right = throttle - steering;
        driveMotor(ENA, IN1, IN2, left);
        driveMotor(ENB, IN3, IN4, right);
      }

      digitalWrite(PIN_BUZZER, (sos || isRTH) ? HIGH : LOW); // Buzz on RTH
      digitalWrite(PIN_STROBE, (sos && (millis() % 200 < 100)) ? HIGH : LOW);
      digitalWrite(PIN_LIGHTS, lights ? HIGH : LOW);
    }
  }
}

void executeRTH() {
  // TODO: Implement sensor-based return or reversal
  driveMotor(ENA, IN1, IN2, -50); // Dummy RTH (Backup slow)
  driveMotor(ENB, IN3, IN4, -50);
}

void executePatrol() {
  // TODO: Implement zig-zag or boundary-bounce logic
}

void driveMotor(int en, int i1, int i2, int speed) {
  int pwm = map(abs(speed), 0, 100, 0, 255);
  digitalWrite(i1, speed >= 0 ? HIGH : LOW);
  digitalWrite(i2, speed >= 0 ? LOW : HIGH);
  analogWrite(en, pwm);
}
```

---

---

> [!TIP]
> สำหรับนักพัฒนา: ให้เน้นความเร็วในการตอบสนองของ Ultrasonic และการทำ Safe-Stop เมื่อเจอสิ่งกีดขวางในระยะอันตรายครับ
