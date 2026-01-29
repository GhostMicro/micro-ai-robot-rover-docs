# ESP32 Core Reciver

## V.1.0

### PlatformIO

```ini
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
monitor_speed = 115200

; ปรับระดับความแรงของสัญญาณ WiFi (เผื่อกรณีต้องการระยะไกลขึ้น)
; build_flags = -DCORE_DEBUG_LEVEL=0

; หากต้องการระบุพอร์ตเพื่อไม่ให้สับสนเวลาเสียบพร้อมกันหลายบอร์ด
; upload_port = /dev/ttyUSB0  ; เปลี่ยนเป็นพอร์ตที่คุณใช้งานจริง
```

### main.cpp

```cpp
#include <Arduino.h>
#include <esp_now.h>
#include <WiFi.h>

// 1. โครงสร้างข้อมูลแบบบีบอัด (Packed) เพื่อให้ STM32 อ่านได้ถูกต้อง
// ต้องแน่ใจว่า STM32 ใช้ Struct รูปแบบเดียวกันเป๊ะๆ
struct __attribute__((packed)) Payload {
    int8_t x;
    int8_t y;
    uint8_t btn;
};

Payload incomingData;

// 2. ฟังก์ชัน Callback เมื่อได้รับสัญญาณวิทยุ
void OnDataRecv(const uint8_t * mac, const uint8_t *incoming, int len) {
    // นำข้อมูลที่ได้รับมาใส่ใน Struct
    memcpy(&incomingData, incoming, sizeof(incomingData));
    
    // 3. ส่งต่อข้อมูลดิบให้ STM32 ผ่าน Hardware Serial 2
    // เราจะส่ง Header 0xAA นำหน้าเพื่อให้ STM32 ตรวจสอบจุดเริ่มต้นได้
    Serial2.write(0xAA); 
    Serial2.write((uint8_t*)&incomingData, sizeof(incomingData));

    // (Optional) สำหรับ Monitor ดูผ่านคอมพิวเตอร์เพื่อตรวจเช็ค
    // หากระบบจริงเสถียรแล้ว สามารถ Comment บรรทัดนี้ออกเพื่อเพิ่มความเร็ว
    Serial.printf("Radio Recv -> Forwarding to STM32 [X:%d Y:%d B:%d]\n", 
                  incomingData.x, incomingData.y, incomingData.btn);
}

void setup() {
    // Serial สำหรับต่อคอมพิวเตอร์ (Debug)
    Serial.begin(115200);
    
    // Serial 2 สำหรับต่อกับ STM32 (TX=GPIO 17, RX=GPIO 16)
    // ใช้ Baud rate สูงเพื่อให้ส่งข้อมูลเข้าสมองหลักได้รวดเร็ว
    Serial2.begin(115200, SERIAL_8N1, 16, 17);
    
    WiFi.mode(WIFI_STA);
    WiFi.disconnect(); // ไม่เชื่อมต่อ AP ใดๆ เพื่อลดการกวนสัญญาณ

    // เริ่มต้น ESP-NOW
    if (esp_now_init() != ESP_OK) {
        Serial.println("Error initializing ESP-NOW");
        return;
    }
    
    // ลงทะเบียนฟังก์ชันรับข้อมูล
    esp_now_register_recv_cb(esp_now_recv_cb_t(OnDataRecv));

    Serial.println("ESP32 Gateway Ready. Forwarding to STM32...");
}

void loop() {
    // ในโหมด Gateway เราไม่เขียนอะไรใน loop เพื่อให้ CPU สแตนด์บายรอรับ Interrupt จาก WiFi
}
```
