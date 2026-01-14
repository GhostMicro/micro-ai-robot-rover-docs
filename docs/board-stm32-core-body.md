# STM32 Core Body

## V.1.0

### PlatformIO

```ini
[env:bluepill_f103c8]
platform = ststm32
board = bluepill_f103c8
framework = arduino
upload_protocol = stlink  ; บังคับให้ใช้ ST-Link ในการอัปโหลด
debug_tool = stlink
monitor_speed = 115200

; --- เพิ่ม 2 บรรทัดนี้ ---
upload_flags = -c set CPUTAPID 0x2ba01477
debug_server = 
    ${platformio.packages_dir}/tool-openocd/bin/openocd
    -f
    scripts/interface/stlink.cfg
    -c
    "set CPUTAPID 0x2ba01477"
    -f
    scripts/target/stm32f1x.cfg
```

### main.cpp

```cpp
void setup() {
    Serial.begin(115200);
}

void loop() {
    Serial.println("Hello World");
    delay(1000);
}
```#include <Arduino.h>

// สร้าง Serial2 สำหรับคุยกับ Arduino Mega
HardwareSerial Serial2(PA3, PA2); 

struct __attribute__((packed)) Payload {
    int8_t x;
    int8_t y;
    uint8_t btn;
};

Payload remoteData;
unsigned long lastRecvTime = 0; // สำหรับระบบ Failsafe

void setup() {
    Serial1.begin(115200); // รับจาก ESP32
    Serial2.begin(115200); // ส่งหา Mega
    pinMode(PC13, OUTPUT);
}

void loop() {
    // 1. รับข้อมูลจาก ESP32
    if (Serial1.available() > 0) {
        if (Serial1.read() == 0xAA) {
            Serial1.readBytes((uint8_t*)&remoteData, sizeof(remoteData));
            lastRecvTime = millis(); // อัปเดตเวลาที่ได้รับสัญญาณล่าสุด
            
            // 2. คำนวณ Mixer (ล้อซ้าย-ขวา) 
            // สูตรพื้นฐาน: ซ้าย = Y+X, ขวา = Y-X
            int leftSpeed = remoteData.y + remoteData.x;
            int rightSpeed = remoteData.y - remoteData.x;

            // Constrain ค่าให้อยู่ในกลุ่ม -100 ถึง 100
            leftSpeed = constrain(leftSpeed, -100, 100);
            rightSpeed = constrain(rightSpeed, -100, 100);

            // 3. ส่งคำสั่งที่คำนวณเสร็จแล้วให้ Mega
            // เราจะส่ง Header 0xFF นำหน้าเพื่อให้ Mega แยกแยะว่าเป็นคำสั่งล้อ
            Serial2.write(0xFF); 
            Serial2.write((int8_t)leftSpeed);
            Serial2.write((int8_t)rightSpeed);
            Serial2.write(remoteData.btn); // ส่งสถานะปุ่มไปด้วย
            
            digitalWrite(PC13, LOW); // ไฟติดเมื่อมีสัญญาณ
        }
    }

    // 4. ระบบ Failsafe: ถ้าสัญญาณหายไปเกิน 500ms ให้หยุดรถ
    if (millis() - lastRecvTime > 500) {
        Serial2.write(0xFF); 
        Serial2.write((int8_t)0); // สั่งล้อซ้ายหยุด
        Serial2.write((int8_t)0); // สั่งล้อขวาหยุด
        Serial2.write((int8_t)0); 
        digitalWrite(PC13, HIGH); // ไฟดับเมื่อสัญญาณขาด
    }
}
```