# ESP32-CAM Programming Guide

## ⚠️ Problem: ESP32-CAM Has No USB Port!

**ESP32-CAM ไม่มี USB!** ต่างจาก ESP32 DevKit ที่มี USB ในตัว

```
ESP32 DevKit:  [USB Port] ──> PC (ง่าย!)
ESP32-CAM:     [No USB!] ──> ต้องใช้ตัวช่วย
```

---

## 🔌 Solution: FTDI Programmer (USB-to-Serial)

### What is FTDI?
**FTDI FT232RL** = ตัวแปลง USB → Serial (UART)

```
PC (USB) ──> FTDI ──> ESP32-CAM (Serial RX/TX)
```

### Common FTDI Modules
| Module | Voltage | Price | Notes |
|:-------|:--------|:------|:------|
| **FT232RL** | 3.3V/5V | ~50 THB | Most common |
| **CP2102** | 3.3V | ~30 THB | Cheaper alternative |
| **CH340G** | 3.3V/5V | ~25 THB | Budget option |

> [!WARNING]
> **ต้องใช้ 3.3V!** ESP32-CAM ไม่รองรับ 5V (จะเสียหาย!)

---

## 🔗 Wiring: FTDI → ESP32-CAM

### Pin Connections
```
FTDI Programmer          ESP32-CAM
─────────────────────────────────
3.3V (VCC)       ──>     5V (ใช้ 5V pin!)
GND              ──>     GND
TX               ──>     U0R (RX)
RX               ──>     U0T (TX)
```

### Programming Mode
**ต้อง Short GPIO0 → GND** เพื่อเข้า Boot Mode:
```
ESP32-CAM Pins:
IO0 ──┐
      ├──> ต่อกันตอน Upload
GND ──┘
```

---

## 📸 Step-by-Step Upload

### 1. Connect FTDI
```
┌─────────────┐         ┌──────────────┐
│ FTDI Module │         │  ESP32-CAM   │
│             │         │              │
│ 3.3V    ────┼────────>│ 5V           │
│ GND     ────┼────────>│ GND          │
│ TX      ────┼────────>│ U0R (RX)     │
│ RX      ────┼────────>│ U0T (TX)     │
└─────────────┘         └──────────────┘
```

### 2. Enter Boot Mode
- **Short IO0 to GND** (use jumper wire)
- Press **RESET** button on ESP32-CAM
- Release RESET (keep IO0 shorted)

### 3. Upload Firmware
```bash
~/.platformio/penv/bin/platformio run -e esp32_camera -t upload
```

### 4. Exit Boot Mode
- **Remove IO0-GND jumper**
- Press **RESET** button
- ESP32-CAM will run your code!

---

## 🎯 Alternative: USB-to-TTL Adapter

**ถ้าไม่มี FTDI ก็ใช้อันนี้ได้:**

### Option 1: Arduino Uno as Programmer
```
Arduino Uno          ESP32-CAM
─────────────────────────────
3.3V         ──>     5V
GND          ──>     GND
TX (Pin 1)   ──>     U0R
RX (Pin 0)   ──>     U0T

+ Remove ATmega328P chip from Uno!
```

### Option 2: ESP32 DevKit as Programmer
```
ESP32 DevKit         ESP32-CAM
─────────────────────────────
3.3V         ──>     5V
GND          ──>     GND
TX (GPIO1)   ──>     U0R
RX (GPIO3)   ──>     U0T

+ Hold EN button on DevKit during upload
```

---

## 🛒 Where to Buy FTDI?

**ในไทย:**
- **Shopee/Lazada:** "FT232RL USB to TTL" (~50 THB)
- **ร้านอิเล็กทรอนิกส์:** ตึกพาหุรัด, ตลาดนัดจตุจักร

**ต่างประเทศ:**
- AliExpress: $1-2 USD
- Amazon: $5-10 USD

---

## ⚡ Power Considerations

### FTDI Power Limitation
**FTDI ให้ไฟได้แค่ ~100mA**  
ESP32-CAM ใช้ ~300mA (ไม่พอ!)

**Solution:**
```
FTDI:     TX/RX only (สำหรับ upload)
External: 5V 1A power supply (สำหรับไฟ)

┌──────┐
│ FTDI │──> TX/RX ──> ESP32-CAM
└──────┘                  ↑
                          │
                    5V 1A Power
```

---

## 🔍 Troubleshooting

### "Failed to connect"
**Solutions:**
1. Check TX/RX swap (TX→RX, RX→TX)
2. Verify IO0 is shorted to GND
3. Press RESET while IO0 grounded
4. Check 3.3V power (not 5V on VCC pin!)

### "Brownout detector"
**Cause:** Not enough power  
**Solution:** Use external 5V 1A supply

### "Invalid head of packet"
**Cause:** Wrong baud rate  
**Solution:** Use 115200 (default)

---

## ✅ Summary

**ESP32-CAM Upload Requirements:**
1. ✅ FTDI Programmer (FT232RL ~50 THB)
2. ✅ Short IO0 → GND (boot mode)
3. ✅ External 5V 1A power (optional but recommended)
4. ✅ Press RESET after upload

**After first upload:**
- Can update via **OTA (Over-The-Air)** through WiFi!
- No need to connect FTDI again 🎉

**พี่มี FTDI อยู่แล้วหรือยังครับ?** 🔌
