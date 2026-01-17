# ESP32 Commander v4.0 - Setup Guide

## 📦 Required Hardware

### Modules
- **BNO055 IMU** (I2C 9-axis sensor)
- **GPS NEO-M8N** (UART GPS module)
- **SD Card Module** (SPI, optional for Phase 2)

### Wiring

#### BNO055 IMU
```
BNO055          ESP32
VIN     →       3.3V
GND     →       GND
SDA     →       GPIO21
SCL     →       GPIO22
```

#### GPS NEO-M8N
```
GPS             ESP32
VCC     →       3.3V
GND     →       GND
TX      →       GPIO16 (RX2)
RX      →       GPIO17 (TX2)
```

#### STM32 Link
```
ESP32           STM32
GPIO1 (TX)  →   PB11 (RX3)
GPIO3 (RX)  →   PB10 (TX3)
GND         →   GND
```

---

## 🔧 Software Setup

### 1. Install Libraries
The `platformio.ini` already includes:
```ini
lib_deps = 
    adafruit/Adafruit BNO055 @ ^1.6.3
    adafruit/Adafruit Unified Sensor @ ^1.1.14
    mikalhart/TinyGPSPlus @ ^1.0.3
```

### 2. Upload Firmware
```bash
cd /media/devg/Micro-SV7/GitHub/GhostMicro/micro-ai-robot-rover/firmware

# Option 1: Use main_v4.cpp (recommended)
cp src/esp32_receiver/main_v4.cpp src/esp32_receiver/main.cpp

# Option 2: Build with specific file
pio run -e esp32_receiver -t upload
```

### 3. Test I2C Devices
```bash
# Open Serial Monitor
pio device monitor -e esp32_receiver

# Expected output:
# BNO055 OK
# ESP32 Commander Ready!
# Heading: 45.2° | GPS: 13.756300, 100.501800 | Valid: 1
```

---

## 🧪 Testing Checklist

### BNO055 IMU
- [ ] Power LED on BNO055 is lit
- [ ] Serial shows "BNO055 OK"
- [ ] Heading value changes when rotating board (0-360°)
- [ ] Tilt values respond to board movement

### GPS NEO-M8N
- [ ] GPS has clear view of sky (outdoor or near window)
- [ ] Red LED blinks (searching for satellites)
- [ ] Serial shows valid coordinates after 30-60 seconds
- [ ] `GPS: Valid: 1` appears in Serial Monitor

### Communication
- [ ] GoalCommand sent to STM32 every time ESP-NOW data received
- [ ] Serial shows `Goal [Mode:X H:XXX S:XX]`

---

## 🐛 Troubleshooting

### BNO055 Not Detected
**Symptom:** `BNO055 not detected!`  
**Solutions:**
1. Check I2C wiring (SDA/SCL swapped?)
2. Verify 3.3V power (not 5V!)
3. Try I2C scanner:
   ```cpp
   Wire.begin(21, 22);
   Wire.beginTransmission(0x28);
   if (Wire.endTransmission() == 0) Serial.println("BNO055 found!");
   ```

### GPS No Fix
**Symptom:** `GPS: Valid: 0` forever  
**Solutions:**
1. Move to outdoor location or near window
2. Wait 2-5 minutes for cold start
3. Check GPS baud rate (should be 9600)
4. Verify TX/RX not swapped

### No Data to STM32
**Symptom:** STM32 not responding  
**Solutions:**
1. Check Serial1 TX/RX pins (GPIO1/GPIO3)
2. Verify common GND between ESP32 and STM32
3. Test with Serial1.println("test") to confirm UART works

---

## 📊 Expected Performance

| Metric | Value |
|:-------|:------|
| **Heading Update Rate** | 100 Hz |
| **GPS Update Rate** | 1-10 Hz (depends on module) |
| **Command Latency** | < 10ms |
| **I2C Speed** | 400 kHz (Fast Mode) |

---

## 🚀 Next Steps

1. ✅ Wire BNO055 and GPS
2. ✅ Upload `main_v4.cpp`
3. ✅ Test sensors
4. ⏳ **Phase 3:** Add encoders to STM32
5. ⏳ **Phase 4:** Add ultrasonics to Nano

**Status:** Phase 2 in progress! 🔧
