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
| Function | GPIO Pin | Note |
| :--- | :--- | :--- |
| **I2C SDA** | 21 | OLED & Sensors |
| **I2C SCL** | 22 | OLED & Sensors |
| **Joy X1** | 34 (ADC1) | Analog Input |
| **Joy Y1** | 35 (ADC1) | Analog Input |
| **ESP-NOW Pairing** | 0 (Boot) | กดค้างเพื่อ Pair |

---

> [!TIP]
> สำหรับนักพัฒนา: ให้โฟกัสที่การทำ Smoothing ค่า Analog และการจัดการ UI บนหน้าจอ OLED เป็นหลักครับ
