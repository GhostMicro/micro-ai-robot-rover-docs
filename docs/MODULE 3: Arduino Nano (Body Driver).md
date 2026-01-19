

ระบบ "Shared Trigger" โดยใช้ Pin D2 เป็นตัวยิงสัญญาณ (Trig) ให้กับเซนเซอร์ทั้ง 4 ตัวพร้อมกัน แล้วแยกรับค่าสะท้อน (Echo) ทีละตัว วิธีนี้ช่วยประหยัด Pin ได้มาก และทำให้เราใส่เซนเซอร์ได้ครบ 360 องศาโดยไม่ไปตีกับขามอเตอร์หรือไฟเลี้ยวครับ

รายละเอียด Pin (Module 3):

*    Trigger ร่วม: D2
*    Echo หน้า: D3
*    Echo ซ้าย: D4
*    Echo ขวา: D11
*    Echo หลัง: D12*

MODULE 3: Arduino Nano (Body Driver)
ทำหน้าที่ขับมอเตอร์ 4 ล้อ และเก็บค่าระยะทางจากเซนเซอร์ Ultrasonic 4 ทิศทาง

## 🔌 Pin Mapping (V4.5 Final)
Component	Nano Pin	Note
L298N Motors	D5 - D10	PWM Drive
US Shared TRIG	D2	[NEW] Shared for all 4 sensors
US Front ECHO	D3	
US Left ECHO	D4	
US Right ECHO	D11	
US Rear ECHO	D12	
Signals L/R	A0, A1	Shifted for US
Headlight	A3	White High-Power LED
🛠 Source Code (Refined)

```cpp
// [main.cpp for nano-body-mode1]
#include <Arduino.h>
#include <stdint.h>
const int US_TRIG = 2;
const int US_F_ECHO = 3, US_L_ECHO = 4, US_R_ECHO = 11, US_B_ECHO = 12;
uint16_t readUS(int echo) {
  digitalWrite(US_TRIG, LOW); delayMicroseconds(2);
  digitalWrite(US_TRIG, HIGH); delayMicroseconds(10);
  digitalWrite(US_TRIG, LOW);
  long duration = pulseIn(echo, HIGH, 20000); 
  return (duration == 0) ? 999 : (duration * 0.034 / 2);
}
void setup() {
  Serial.begin(115200);
  pinMode(US_TRIG, OUTPUT);
  pinMode(US_F_ECHO, INPUT); pinMode(US_L_ECHO, INPUT);
  pinMode(US_R_ECHO, INPUT); pinMode(US_B_ECHO, INPUT);
  // ... Motor Init ...
}
void loop() {
  // 5Hz Sensor Hub Loop
  static uint32_t lastUS = 0;
  if (millis() - lastUS > 200) {
    lastUS = millis();
    uint16_t f = readUS(US_F_ECHO); delay(10);
    uint16_t l = readUS(US_L_ECHO); delay(10);
    uint16_t r = readUS(US_R_ECHO); delay(10);
    uint16_t b = readUS(US_B_ECHO);
    // Send ObstacleMap to STM32...
  }
}
```