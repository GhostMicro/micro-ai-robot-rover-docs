# สรุปการต่อสาย (Complete Wiring Guide)

## ภาพรวมระบบ (System Overview)

ระบบประกอบด้วย 3 บอร์ดหลักที่ทำงานร่วมกัน:
1. **ESP32 Main** - สมองหลัก (Navigator)
2. **Arduino Mega** - กล้ามเนื้อ (Muscle)
3. **ESP32-CAM** - ดวงตา (Vision)

---

## 1. ระบบไฟ (Power System)

### ชุดที่ 1: Logic Power (5V Regulated)
**จ่ายให้:**
- ESP32 Main (ขา VIN หรือ 5V)
- Arduino Mega (ขา 5V)
- Level Converter (ขา HV)
- Sensors ทั้งหมด (OLED, RTC, SD Card)

### ชุดที่ 2: Motor Power (7.4V - 8.4V)
**จ่ายให้:**
- Motor Driver (L298N / MOSFET Module)
- DC Motors

> **⚠️ สำคัญ:** ต้องเชื่อม GND ของทั้ง 2 ชุดเข้าด้วยกัน (Common Ground)

---

## 2. ESP32 Main (Navigator)

### I2C Bus (ขา 21, 22)
| อุปกรณ์        | SDA  | SCL  | VCC  | หมายเหตุ      |
| :----------- | :--- | :--- | :--- | :----------- |
| OLED Display | 21   | 22   | 3.3V | แสดงสถานะ    |
| RTC DS3231   | 21   | 22   | 3.3V | นาฬิกาเวลาจริง |

### SPI Bus (SD Card)
| สัญญาณ | ESP32 Pin | SD Module Pin |
| :---- | :-------- | :------------ |
| CS    | 5         | CS            |
| SCK   | 18        | SCK           |
| MISO  | 19        | MISO          |
| MOSI  | 23        | MOSI          |
| VCC   | 5V        | VCC           |
| GND   | GND       | GND           |

### UART2 (Serial Bridge to Mega)
| สัญญาณ | ESP32 Pin | Level Converter | Mega Pin |
| :---- | :-------- | :-------------- | :------- |
| TX2   | 17        | LV1 → HV1       | 19 (RX1) |
| RX2   | 16        | LV2 ← HV2       | 18 (TX1) |

### อื่นๆ
| ฟังก์ชัน       | Pin  | การต่อ                 |
| :---------- | :--- | :-------------------- |
| Mode Button | 4    | ต่อไปที่ปุ่มกด → GND       |
| USB Serial  | 0, 1 | สำหรับรับคำสั่งจาก Notebook |

---

## 3. Arduino Mega (Muscle)

### Serial Communication
| สัญญาณ | Mega Pin | Level Converter | ESP32 Pin |
| :---- | :------- | :-------------- | :-------- |
| TX1   | 18       | HV2 → LV2       | 16 (RX2)  |
| RX1   | 19       | HV1 ← LV1       | 17 (TX2)  |

### Body ID Detection
| Body Type    | Pin  | การต่อ                  |
| :----------- | :--- | :--------------------- |
| Rescue Mode  | 30   | ต่อลง GND เมื่อต้องการเลือก |
| Explore Mode | 31   | ต่อลง GND เมื่อต้องการเลือก |
| Sentry Mode  | 32   | ต่อลน GND เมื่อต้องการเลือก |

### Ultrasonic Sensors (HC-SR04)
| ตำแหน่ง | Trig Pin | Echo Pin |
| :---- | :------- | :------- |
| Front | 22       | 23       |
| Left  | 24       | 25       |
| Right | 26       | 27       |
| Back  | 28       | 29       |

### Motor Control (PWM)
| Motor       | PWM Pins | หมายเหตุ            |
| :---------- | :------- | :----------------- |
| Left Motor  | 2, 3     | ต่อเข้า Motor Driver |
| Right Motor | 4, 5     | ต่อเข้า Motor Driver |
| Enable Pins | 6, 7     | สำหรับควบคุมความเร็ว   |

---

## 4. ESP32-CAM (Vision)

ESP32-CAM ใช้ขาภายในสำหรับกล้อง ไม่ต้องต่อสายเพิ่ม เพียงแค่:
- **VCC:** 5V (จาก Logic Power)
- **GND:** GND (Common Ground)
- **WiFi:** เชื่อมต่อไร้สายเพื่อส่งภาพไปยัง Notebook

---

## 5. Level Converter (MH)

| ฝั่ง Low Voltage (LV)    | ฝั่ง High Voltage (HV) |
| :--------------------- | :------------------- |
| LV (3.3V) ← ESP32 3.3V | HV (5V) ← Mega 5V    |
| LV1 ← ESP32 Pin 17     | HV1 → Mega Pin 19    |
| LV2 → ESP32 Pin 16     | HV2 ← Mega Pin 18    |
| GND ← Common Ground    | GND ← Common Ground  |

---

## 6. ลำดับการต่อสาย (Assembly Order)

1. **เตรียมแหล่งจ่ายไฟ:** ต่อ Buck Converter และแบตเตอรี่ให้พร้อม
2. **ต่อ Ground ทั้งหมด:** รวม GND ของทุกอุปกรณ์เข้าด้วยกัน
3. **ติดตั้ง Level Converter:** ต่อสาย Serial ระหว่าง ESP32 และ Mega
4. **ต่อ I2C Bus:** เชื่อม OLED และ RTC เข้ากับ ESP32
5. **ต่อ SD Card:** เชื่อม SPI Bus เข้ากับ ESP32
6. **ต่อ Sensors:** ติดตั้ง Ultrasonic บน Mega
7. **ทดสอบทีละส่วน:** ตรวจสอบว่าแต่ละบอร์ดสื่อสารกันได้

---

## 7. การทดสอบ (Testing Checklist)

- [ ] ตรวจสอบแรงดันไฟทุกจุด (ต้องได้ 5V และ 7.4V ตามที่กำหนด)
- [ ] ทดสอบ Serial Communication (ESP32 ↔ Mega)
- [ ] ทดสอบ I2C (OLED แสดงผลได้, RTC บอกเวลาถูกต้อง)
- [ ] ทดสอบ SD Card (เขียนไฟล์ได้)
- [ ] ทดสอบ Body ID Detection (อ่านค่า Pin 30-32 ได้)
- [ ] ทดสอบ Ultrasonic Sensors (วัดระยะได้)
- [ ] ทดสอบ Motor Control (มอเตอร์หมุนได้)
