# ESP32 Transmitter

## V.1.0

### PlatformIO

```ini
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
monitor_speed = 115200
```

### main.cpp

```cpp
#include <Arduino.h>
#include <WiFi.h>
#include <esp_now.h>

// 1. โครงสร้างข้อมูล (ต้องตรงกันทั้ง 3 บอร์ด)
struct Payload {
    int8_t x;
    int8_t y;
    uint8_t btn;
};
Payload data;

// 2. ใส่ MAC Address ของ ESP32 "ตัวรับ" (ที่หุ่นยนต์) 
// ถ้ายังไม่มี ให้ใส่เลขหลอกไว้ก่อนเพื่อคอมไพล์
uint8_t receiverAddress[] = {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF}; 

void setup() {
    Serial.begin(115200);
    WiFi.mode(WIFI_STA);
    
    // แสดง MAC ของตัวเอง (Transmitter)
    Serial.print("Transmitter MAC: ");
    Serial.println(WiFi.macAddress());

    // เริ่มต้น Serial2 รับจาก Nano
    Serial2.begin(115200, SERIAL_8N1, 16, 17);

    // เริ่มต้น ESP-NOW
    if (esp_now_init() != ESP_OK) {
        Serial.println("Error initializing ESP-NOW");
        return;
    }
}

void loop() {
    if (Serial2.available() > 0) {
        // ดักฟัง Header 0xAA เพื่อความแม่นยำ
        if (Serial2.read() == 0xAA) {
            Serial2.readBytes((uint8_t*)&data, sizeof(data));

            // แสดงผล Debug ออกจอคอม (สะอาดแล้ว)
            Serial.printf("JOYSTICK -> X:%d Y:%d BTN:%d\n", data.x, data.y, data.btn);

            // ส่งข้อมูลออกอากาศไปยังหุ่นยนต์
            esp_now_send(receiverAddress, (uint8_t *) &data, sizeof(data));
        }
    }
}

```
