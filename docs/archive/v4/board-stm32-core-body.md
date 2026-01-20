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

#include <Adafruit_MPU6050.h>
#include <Adafruit_Sensor.h>
#include <Arduino.h>
#include <Wire.h>

HardwareSerial Serial2(PA3, PA2); // To Nano
HardwareSerial Serial3(PB11, PB10); // From ESP32
Adafruit_MPU6050 mpu;

struct __attribute__((packed)) Telemetry {
  uint8_t header1, header2; // 0xBB, 0xCC
  uint16_t seq;
  float ax, ay, az, gx, gy, gz, temp;
  uint8_t i2c_status, checksum;
};

void initMPU() {
    Wire.beginTransmission(0x68);
    Wire.write(0x6B); Wire.write(0x80); // Reset
    Wire.endTransmission();
    delay(500);
    if (mpu.begin()) {
        mpu.setAccelerometerRange(MPU6050_RANGE_8_G);
        mpu.setFilterBandwidth(MPU6050_BAND_21_HZ);
    }
}

void setup() {
  Serial.begin(115200);
  Serial2.begin(115200); Serial3.begin(9600);
  Wire.begin(); Wire.setClock(50000);
  initMPU();
}

void loop() {
  static uint32_t lastTelem = 0;
  if (millis() - lastTelem > 100) {
    lastTelem = millis();
    sensors_event_t a, g, temp;
    Telemetry t; memset(&t, 0, sizeof(t));
    t.header1 = 0xBB; t.header2 = 0xCC;
    
    if (mpu.getEvent(&a, &g, &temp)) {
      t.ax = a.acceleration.x; t.temp = temp.temperature;
      if (t.ax > -150.0f && t.ax < 150.0f) { // Glitch Filter
        // Calculate checksum and Serial3.write(t)
      } else { initMPU(); }
    }
  }

  if (Serial3.available() >= 7) { // Read Payload from ESP32
     // Differential Mixer -> Serial2.write to Nano
  }
}