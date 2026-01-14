# Nano Joystrick

## V.1.0

### PlatformIO

```ini
[env:nanoatmega328]
platform = atmelavr
board = nanoatmega328
framework = arduino
monitor_speed = 115200
```

### main.cpp

```cpp
#include <Arduino.h>

// --- 1. Data Structure ---
// ใช้ struct เพื่อแพ็กข้อมูลให้เป็นระเบียบ
struct Payload {
    int8_t x;     // ใช้ int8_t เพื่อเก็บค่า -100 ถึง 100 (ใช้พื้นที่แค่ 1 byte)
    int8_t y;
    uint8_t btn;  // เก็บสถานะปุ่ม 0 หรือ 1
};

Payload joystickData;

// --- 2. Pin Configuration ---
const uint8_t pinX = A0;
const uint8_t pinY = A1;
const uint8_t pinSW = 2;

// --- 3. Parameters ---
const int deadzone = 15;
const int samples = 64; 

void setup() {
    // Nano มี Serial เดียว ซึ่งใช้ทั้งส่งข้อมูลข้ามบอร์ดและ Debug ลงคอม
    // เราจะใช้ความเร็ว 115200 เป็นมาตรฐาน
    Serial.begin(115200);
    
    pinMode(pinSW, INPUT_PULLUP);
    
    // หน่วงเวลารอระบบไฟฟ้าเสถียร
    delay(500);
}

void loop() {
    // --- 4. Data Processing (จาก Phase 1) ---
    long rawX = 0, rawY = 0;
    for(int i = 0; i < samples; i++) {
        rawX += analogRead(pinX);
        rawY += analogRead(pinY);
    }
    
    int avgX = rawX / samples;
    int avgY = rawY / samples;

    // Map เป็น -100 ถึง 100 เพื่อความง่ายและประหยัดพื้นที่ส่งข้อมูล
    int mapX = map(avgX, 0, 1023, -100, 100);
    int mapY = map(avgY, 0, 1023, -100, 100);

    if (abs(mapX) < deadzone) mapX = 0;
    if (abs(mapY) < deadzone) mapY = 0;

    // เตรียมข้อมูลลงใน Struct
    joystickData.x = (int8_t)mapX;
    joystickData.y = (int8_t)mapY;
    joystickData.btn = !digitalRead(pinSW); // กด=1, ปล่อย=0

    // --- 5. Transmission (Phase 2) ---
    // ส่งแบบ Binary เพื่อความเร็วสูงสุดและไม่มีการโค้งของสัญญาณ
    // เราจะส่ง Header 0xAA เพื่อให้ฝั่งรับ (ESP32) รู้ว่าเป็นจุดเริ่มต้น
    
    Serial.write(0xAA); 
    Serial.write((uint8_t*)&joystickData, sizeof(joystickData));
    

    // ส่ง Debug ออกไปดูในคอมด้วย (ทางเทคนิคจะปนไปกับ Binary แต่ Serial Monitor ยังพออ่านออก)
    // หรือจะคอมเมนต์ออกถ้าจะเชื่อมต่อกับ ESP32 จริงๆ
    
    Serial.print(" X:"); Serial.print(joystickData.x);
    Serial.print(" Y:"); Serial.print(joystickData.y);
    Serial.print(" B:"); Serial.println(joystickData.btn);
    
    delay(20); // 50Hz Update rate
}
```