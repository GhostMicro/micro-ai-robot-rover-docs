# 📟 Module: Transmitter (ชุดควบคุมระยะไกล)

โมดูลนี้รับผิดชอบการสร้าง Interface ให้มนุษย์สามารถสั่งการหุ่นยนต์ได้โดยตรง (Emergency & Manual Control)

## 🎯 เป้าหมาย (Objectives)
- รับค่าจาก Analog Joystick และ Switches
- ส่งข้อมูลแบบ Real-time ไปยัง Header โดยมีความหน่วง (Latency) ต่ำที่สุด
- แสดงสถานะ Telemetry พื้นฐานบนจอ OLED

## 🛠️ Hardware Requirements
- **MCU:** ESP32 (WROOM-32)
- **Input:** 2x Dual-Axis Joystick, 4x Toggle/Push Buttons
- **Display:** 0.96" OLED (I2C)
- **Power:** Li-ion 18650 + Charging Circuit

## 📡 Communication Rules
- **Protocol:** ESP-NOW (Point-to-Point) ไปยัง ESP32 Gateway (Header)
- **Data Structure:** ต้องใช้ Standard `struct` ที่กำหนดไว้ใน [Standards & Comms](standards_pins_comms.md)

## 📍 Reserved Pins (ESP32)
| Function            | GPIO Pin  | Note             |
| :------------------ | :-------- | :--------------- |
| **I2C SDA**         | 21        | OLED & Sensors   |
| **I2C SCL**         | 22        | OLED & Sensors   |
| **Joy X1**          | 34 (ADC1) | Analog Input     |
| **Joy Y1**          | 35 (ADC1) | Analog Input     |
| **RTH Button**      | 26        | ปุ่มสั่งกลับฐาน       |
| **Patrol Switch**   | 25        | สวิตช์เริ่มลาดตระเวน |
| **ESP-NOW Pairing** | 0 (Boot)  | กดค้างเพื่อ Pair    |

---

## 💻 Latest Firmware: ESP32 Advanced Transmitter

โค้ดนี้ใช้สำหรับ ESP32 (Transmitter) เพื่ออ่านค่าจอยสติ๊กและส่งผ่าน ESP-NOW ไปยังหุ่นยนต์

```cpp
#include <WiFi.h>
#include <esp_now.h>
#include <Adafruit_SSD1306.h>

Adafruit_SSD1306 display(128, 64, &Wire, -1);
uint8_t robotMAC[] = {0xAA, 0xBB, 0xCC, 0xDD, 0xEE, 0xFF};

struct ControlPacket {
  byte signature = 0xAF;
  int8_t throttle;
  int8_t steering;
  byte command_flags; // Bit 0: RTH, Bit 1: Area Patrol
  byte sos;
  byte lights;
} myData;

const int pinRTH = 26;
const int pinPatrol = 25;

void setup() {
  Serial.begin(115200);
  display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
  
  pinMode(pinRTH, INPUT_PULLUP);
  pinMode(pinPatrol, INPUT_PULLUP);

  WiFi.mode(WIFI_STA);
  esp_now_init();
  
  esp_now_peer_info_t peerInfo = {};
  memcpy(peerInfo.peer_addr, robotMAC, 6);
  esp_now_add_peer(&peerInfo);
}

void loop() {
  // Read Joysticks (ADC 12-bit)
  int rawT = analogRead(35);
  int rawS = analogRead(34);
  
  // Clean Data & Smooth
  myData.throttle = map(rawT, 0, 4095, -100, 100);
  myData.steering = map(rawS, 0, 4095, -100, 100);
  
  if(abs(myData.throttle) < 15) myData.throttle = 0;
  if(abs(myData.steering) < 15) myData.steering = 0;

  // Handle Command Flags
  myData.command_flags = 0;
  if (digitalRead(pinRTH) == LOW) myData.command_flags |= 0x01; // Bit 0: RTH
  if (digitalRead(pinPatrol) == LOW) myData.command_flags |= 0x02; // Bit 1: Patrol

  esp_now_send(robotMAC, (uint8_t *) &myData, sizeof(myData));
  
  // UI Update
  display.clearDisplay();
  display.setCursor(0,0);
  display.printf("T: %d | S: %d", myData.throttle, myData.steering);
  display.display();
  
  delay(50);
}
```

---

---

> [!TIP]
> สำหรับนักพัฒนา: ให้โฟกัสที่การทำ Smoothing ค่า Analog และการจัดการ UI บนหน้าจอ OLED เป็นหลักครับ
