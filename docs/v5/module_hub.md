# 🏠 Module: Autonomous Hub (ศูนย์กลางและจุดอ้างอิง)

Hub คือโมดูลที่ถูกพัฒนาแยกออกมาเพื่อทำหน้าที่เป็น "จุดยึดเหนี่ยว" (Stationary Anchor) และ "ผู้จัดการพื้นที่" (Area Manager) สำหรับหุ่นยนต์ Ghost Micro หลายตัว

## 🎯 เป้าหมาย (Objectives)
- เป็นจุดอ้างอิงคงที่ (Home Point) สำหรับระบบ Return Home (RTH)
- กำหนดขอบเขตพื้นที่ปลอดภัย (Boundary Enforcement) ผ่านสัญญาณ Beacon
- ประสานงานระหว่างหุ่นยนต์หลายตัว (Multi-Robot Coordination) โดยไม่ต้องผ่าน Computer ตลอดเวลา
- ทำงานแบบ Standalone เพียงแค่จ่ายไฟก็เริ่มทำงานได้ทันที

## 🛠️ Hardware Requirements
- **MCU Options:**
    - **ESP8266 (NodeMCU/D1 Mini):** เพียงพอสำหรับการทำ Beacon ประจำที่ (ประหยัดพลังงานและราคาถูก)
    - **ESP32 (Standard):** สำหรับงานที่ต้องการการเชื่อมต่อที่ซับซ้อนขึ้น หรือใช้จอ OLED ขนาดใหญ่
- **Status Display:** OLED 0.96" (I2C) - เพื่อแสดงจำนวนหุ่นยนต์ที่เชื่อมต่อและสถานะพื้นที่
- **Indicators:** High-Brightness RGB LED - เพื่อบอกสถานะความปลอดภัยของพื้นที่ (เขียว/แดง)
- **Power:** External 5V Power Supply (เนื่องจากเป็นจุดประจำที่)

## 📡 Communication Rules
- **Mode:** Broadcast (One-to-Many)
- **Protocol:** ESP-NOW (Ch 1)
- **Payload:** Heartbeat Beacon ที่มี Hub ID และสถานะเขตพื้นที่

## 📍 Pin Allocation (Hub)
| Function       | ESP32 Pin | ESP8266 Pin  | Note                 |
| :------------- | :-------- | :----------- | :------------------- |
| **I2C SDA**    | 21        | D2 (GPIO 4)  | OLED Display         |
| **I2C SCL**    | 22        | D1 (GPIO 5)  | OLED Display         |
| **Status LED** | 2         | D4 (GPIO 2)  | Internal LED (Blink) |
| **Alarm Pin**  | 25        | D5 (GPIO 14) | ต่อกับ Siren (ถ้ามี)     |

---

## 💻 Latest Firmware: Hub Beacon (Universal)

โค้ดนี้สามารถปรับใช้ได้กับทั้ง **ESP32** และ **ESP8266** (โดยการเปลี่ยน Library บางตัว) เพื่อทำหน้าที่ปล่อยสัญญาณ Beacon

> [!NOTE]
> หากใช้ ESP8266 ต้องติดตั้ง Library `ESP8266WiFi` และ `espnow.h` สำหรับ 8266 โดยเฉพาะ

```cpp
#include <WiFi.h>
#include <esp_now.h>
#include <Adafruit_SSD1306.h>

Adafruit_SSD1306 display(128, 64, &Wire, -1);

struct HubBeacon {
  byte signature = 0xBB;
  byte hubID = 0x01;
  byte areaStatus = 0x00; // 0=Safe, 1=Alert
} beaconData;

void setup() {
  Serial.begin(115200);
  display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
  
  WiFi.mode(WIFI_STA);
  esp_now_init();
  
  // Register peer as broadcast address
  uint8_t broadcastAddress[] = {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF};
  esp_now_peer_info_t peerInfo = {};
  memcpy(peerInfo.peer_addr, broadcastAddress, 6);
  esp_now_add_peer(&peerInfo);

  Serial.println("Ghost Micro Hub: Operational");
}

void loop() {
  // Broadcast heartbeat every 1 second
  esp_now_send(NULL, (uint8_t *) &beaconData, sizeof(beaconData));
  
  // Update UI
  display.clearDisplay();
  display.setCursor(0,0);
  display.printf("HUB ID: %02X", beaconData.hubID);
  display.setCursor(0,10);
  display.print("Status: STANDALONE");
  display.setCursor(0,30);
  display.print("Broadcasting Beacon...");
  display.display();
  
  delay(1000);
}
```

---

> [!TIP]
> **Stationary Point:** แนะนำให้ติดตั้ง Hub ไว้ในที่สูงหรือจุดกึ่งกลางของพื้นที่ปฏิบัติงาน เพื่อการกระจายสัญญาณที่ครอบคลุมที่สุดครับ
