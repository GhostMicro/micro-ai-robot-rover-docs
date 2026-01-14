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

// 1. กำหนดขาเชื่อมต่อ
const int ENA = 5;  
const int IN1 = 7;  
const int IN2 = 8;  
const int ENB = 6;  
const int IN3 = 9;  
const int IN4 = 10; 

// 2. ประกาศฟังก์ชัน (Function Prototype) เพื่อให้ Compiler รู้จักล่วงหน้า
void stopMotors();
void driveMotorLeft(int speed);
void driveMotorRight(int speed);

void setup() {
    Serial.begin(115200); 
    
    pinMode(ENA, OUTPUT);
    pinMode(IN1, OUTPUT);
    pinMode(IN2, OUTPUT);
    pinMode(ENB, OUTPUT);
    pinMode(IN3, OUTPUT);
    pinMode(IN4, OUTPUT);
    
    stopMotors(); // เรียกใช้งานได้แล้วเพราะมี Prototype ด้านบน
}

void loop() {
    if (Serial.available() >= 4) {
        if (Serial.read() == 0xFF) {
            int8_t leftSpeed = (int8_t)Serial.read();
            int8_t rightSpeed = (int8_t)Serial.read();
            uint8_t btn = (uint8_t)Serial.read(); // รับค่าปุ่มมาเก็บไว้ (แม้ยังไม่ได้ใช้)

            driveMotorLeft(leftSpeed);
            driveMotorRight(rightSpeed);
        }
    }
}

// 3. ส่วนรายละเอียดของฟังก์ชัน
void driveMotorLeft(int speed) {
    int pwm = map(abs(speed), 0, 100, 0, 255);
    if (speed > 0) {
        digitalWrite(IN1, HIGH);
        digitalWrite(IN2, LOW);
    } else if (speed < 0) {
        digitalWrite(IN1, LOW);
        digitalWrite(IN2, HIGH);
    } else {
        pwm = 0;
    }
    analogWrite(ENA, pwm);
}

void driveMotorRight(int speed) {
    int pwm = map(abs(speed), 0, 100, 0, 255);
    if (speed > 0) {
        digitalWrite(IN3, HIGH);
        digitalWrite(IN4, LOW);
    } else if (speed < 0) {
        digitalWrite(IN3, LOW);
        digitalWrite(IN4, HIGH);
    } else {
        pwm = 0;
    }
    analogWrite(ENB, pwm);
}

void stopMotors() {
    analogWrite(ENA, 0);
    analogWrite(ENB, 0);
    digitalWrite(IN1, LOW);
    digitalWrite(IN2, LOW);
    digitalWrite(IN3, LOW);
    digitalWrite(IN4, LOW);
}
```
