# ขั้นตอนการสร้างส่วนสมอง (Brain Implementation)

ส่วนสมอง (Brain) คือกล่องควบคุมหลักที่ประกอบไปด้วย ESP32 และเซนเซอร์ต่างๆ ขั้นตอนนี้จะอธิบายวงจรและการเขียนโปรแกรมสื่อสาร

## 1. วงจรสมอง (Brain Circuit)

หัวใจสำคัญคือการเชื่อมต่อ **ESP32 (3.3V)** เข้ากับ **Arduino Mega (5V)** ผ่านตัวแปลงระดับแรงดัน (**Logic Level Converter**) เพื่อป้องกันไม่ให้ ESP32 เสียหาย

### ผังการต่อสาย (Wiring Diagram)

**การเชื่อมต่อระหว่าง Controller:**
```
[ESP32]             [Level Converter]            [Arduino Mega]
GPIO 17 (TX2) ----> LV1 (Low) -> HV1 (High) ----> D19 (RX1)  (ส่งคำสั่ง)
GPIO 16 (RX2) <---- LV2 (Low) <- HV2 (High) <---- D18 (TX1)  (รับค่ากลับ)
3.3V          ----> LV          HV          ----> 5V
GND           ----> GND         GND         ----> GND
```

**การต่อระบบบันทึกข้อมูล (SD Card SPI):**
ต่อเข้ากับ **ESP32** โดยตรงเพื่อความเร็วในการเขียนข้อมูล
*   **CS:** GPIO 5
*   **SCK:** GPIO 18
*   **MOSI:** GPIO 23
*   **MISO:** GPIO 19
*   **VCC:** 5V (จาก Logic Power)
*   **GND:** GND

**การต่อระบบแสดงผลและเวลา (I2C Bus):**
ต่อขนานกันเข้าที่ **ESP32** (Pin 21, 22)
*   **SDA (GPIO 21):** ต่อไปยัง SDA ของ **OLED** และ SDA ของ **RTC Module (DS3231)**
*   **SCL (GPIO 22):** ต่อไปยัง SCL ของ **OLED** และ SCL ของ **RTC Module (DS3231)**
*   **VCC:** 3.3V (สำหรับเซนเซอร์ไอซีกระแสต่ำ) หรือ 5V ตามรุ่นโมดูล

## 2. โค้ดสื่อสาร (Communication Code)

เราใช้โปรโตคอล **Serial UART** ในการส่งคำสั่งรูปแบบ Text String ที่อ่านง่าย (Human Readable) เช่น `MOVE:100:100`

### ระบบ Priority Control (ลำดับความสำคัญ)
1. **Priority 1:** คำสั่งจาก Notebook (ฉุกเฉิน/Manual Override)
2. **Priority 2:** ปุ่มกดที่ตัวกล่อง ESP32
3. **Priority 3:** โหมดอัตโนมัติตาม Body ID

---

### ฝั่ง Arduino Mega (Muscle & Body Sensor)
รับผิดชอบ: ตรวจจับ Body ID, อ่านค่า Ultrasonic Sensors, รอรับคำสั่งจาก ESP32, และควบคุมมอเตอร์

**Pin Assignment:**
- **Serial1 (RX1=19, TX1=18):** ต่อไปที่ ESP32 ผ่าน Level Converter
- **Pin 30, 31, 32:** Input สำหรับอ่าน ID ของบอดี้รถ (ต่อลง GND เพื่อเลือก ID)
- **Pin 22-29:** Ultrasonic Sensors (Trig/Echo pairs)
- **Pin 2-7:** PWM สำหรับ Motor Driver

```cpp
// --- Arduino Mega: Muscle & Body Sensor ---
#include <NewPing.h>

// --- Configuration: Pins ---
#define BODY_RESCUE_PIN 30
#define BODY_EXPLORE_PIN 31
#define BODY_SENTRY_PIN 32

// Motor Pins (ต่อเข้ากับ MOSFET หรือ Motor Driver)
#define MOTOR_L_PWM 2  // ความเร็วซ้าย
#define MOTOR_L_DIR 24 // ทิศทางซ้าย (HIGH=หน้า, LOW=หลัง)
#define MOTOR_R_PWM 3  // ความเร็วขวา
#define MOTOR_R_DIR 25 // ทิศทางขวา (HIGH=หน้า, LOW=หลัง)

// Ultrasonic Sensors
#define SONAR_NUM 4
#define MAX_DISTANCE 200
#define SAFE_DISTANCE 15 // ระยะหยุดฉุกเฉิน (ซม.)

NewPing sonar[SONAR_NUM] = {
  NewPing(22, 23, MAX_DISTANCE), // 0: Front
  NewPing(28, 29, MAX_DISTANCE), // 1: Back
  NewPing(26, 27, MAX_DISTANCE), // 2: Left
  NewPing(40, 41, MAX_DISTANCE)  // 3: Right (ปรับพินตามความเหมาะสม)
};

int distances[SONAR_NUM];
unsigned long lastBodyCheck = 0;

void setup() {
  Serial.begin(115200); // Debug Console
  Serial1.begin(9600);  // Communication with ESP32 (Main Brain)

  // Setup Body Switch Pins
  pinMode(BODY_RESCUE_PIN, INPUT_PULLUP);
  pinMode(BODY_EXPLORE_PIN, INPUT_PULLUP);
  pinMode(BODY_SENTRY_PIN, INPUT_PULLUP);

  // Setup Motor Pins
  pinMode(MOTOR_L_PWM, OUTPUT);
  pinMode(MOTOR_L_DIR, OUTPUT);
  pinMode(MOTOR_R_PWM, OUTPUT);
  pinMode(MOTOR_R_DIR, OUTPUT);

  stopAllMotors();
  Serial.println("Mega: Muscle System Online - Ready to Roll");
}

void loop() {
  // 1. อ่านค่าเซนเซอร์แบบรวดเร็ว
  updateSensors();

  // 2. ระบบความปลอดภัยอัตโนมัติ (Autonomous Safety)
  if (distances[0] < SAFE_DISTANCE) { 
    stopAllMotors();
    Serial1.println("EVENT:OBSTACLE_FRONT"); 
  }

  // 3. ส่งข้อมูลระยะทางให้ ESP32 ทุก 500ms (ลด Traffic สาย Serial)
  static unsigned long lastUpdate = 0;
  if (millis() - lastUpdate > 500) {
    sendSensorData();
    lastUpdate = millis();
  }

  // 4. ส่งสถานะ Body ID ทุก 5 วินาที
  if (millis() - lastBodyCheck > 5000) {
    sendBodyID();
    lastBodyCheck = millis();
  }

  // 5. รับคำสั่งจาก ESP32 (Dashboard)
  if (Serial1.available()) {
    String cmd = Serial1.readStringUntil('\n');
    cmd.trim();
    processCommand(cmd);
  }
}

// --- ฟังก์ชันเสริม (Helper Functions) ---

void updateSensors() {
  for (int i = 0; i < SONAR_NUM; i++) {
    distances[i] = sonar[i].ping_cm();
    if (distances[i] == 0) distances[i] = MAX_DISTANCE;
    delay(30); // ระยะเวลาที่เหมาะสมสำหรับ NewPing
  }
}

void sendSensorData() {
  Serial1.print("DIST:");
  Serial1.print(distances[0]); Serial1.print(",");
  Serial1.print(distances[1]); Serial1.print(",");
  Serial1.print(distances[2]); Serial1.print(",");
  Serial1.println(distances[3]);
}

void sendBodyID() {
  if (digitalRead(BODY_RESCUE_PIN) == LOW) Serial1.println("BODY:RESCUE");
  else if (digitalRead(BODY_EXPLORE_PIN) == LOW) Serial1.println("BODY:EXPLORE");
  else if (digitalRead(BODY_SENTRY_PIN) == LOW) Serial1.println("BODY:SENTRY");
  else Serial1.println("BODY:DEFAULT");
}

void processCommand(String cmd) {
  Serial.println("Received: " + cmd);
  
  if (cmd == "HALT") {
    stopAllMotors();
    Serial1.println("ACK:STOPPED");
  } 
  else if (cmd.startsWith("MOVE:")) {
    // Format: MOVE:DIRECTION:SPEED (เช่น MOVE:FWD:200)
    int firstColon = cmd.indexOf(':');
    int secondColon = cmd.indexOf(':', firstColon + 1);
    
    String direction = cmd.substring(firstColon + 1, secondColon);
    int speed = cmd.substring(secondColon + 1).toInt();
    
    controlMovement(direction, speed);
  }
}

void controlMovement(String dir, int spd) {
  if (dir == "FWD") {
    digitalWrite(MOTOR_L_DIR, HIGH); digitalWrite(MOTOR_R_DIR, HIGH);
    analogWrite(MOTOR_L_PWM, spd);   analogWrite(MOTOR_R_PWM, spd);
  } else if (dir == "BCK") {
    digitalWrite(MOTOR_L_DIR, LOW);  digitalWrite(MOTOR_R_DIR, LOW);
    analogWrite(MOTOR_L_PWM, spd);   analogWrite(MOTOR_R_PWM, spd);
  } else if (dir == "LFT") {
    digitalWrite(MOTOR_L_DIR, LOW);  digitalWrite(MOTOR_R_DIR, HIGH);
    analogWrite(MOTOR_L_PWM, spd);   analogWrite(MOTOR_R_PWM, spd);
  } else if (dir == "RGT") {
    digitalWrite(MOTOR_L_DIR, HIGH); digitalWrite(MOTOR_R_DIR, LOW);
    analogWrite(MOTOR_L_PWM, spd);   analogWrite(MOTOR_R_PWM, spd);
  }
}

void stopAllMotors() {
  analogWrite(MOTOR_L_PWM, 0);
  analogWrite(MOTOR_R_PWM, 0);
  Serial.println("!!! STOPPED !!!");
}
```

**หมายเหตุ:**
- ต้องติดตั้ง Library `NewPing` สำหรับ Ultrasonic Sensors
- ข้อมูลจะถูกส่งในรูปแบบ `DIST:25,80,40,42` (Front, Back, Left, Right)

---

### ฝั่ง ESP32 Main (Navigator & Gateway)
รับผิดชอบ: ตัดสินใจระดับสูง, รับคำสั่งจาก Notebook, บันทึก Log, แสดงผลบน OLED, และส่งข้อมูลไปยัง Dashboard

**หน้าที่หลัก:**
1. รับข้อมูลจาก Mega (ระยะทาง Ultrasonic, สถานะมอเตอร์)
2. อ่านค่าเซนเซอร์ตัวเอง (MPU6050, RTC)
3. บันทึกข้อมูลลง SD Card (Black Box Log)
4. ส่งข้อมูลไปยัง Next.js Dashboard ผ่าน HTTP POST

**Pin Assignment:**
- **I2C (SDA=21, SCL=22):** ต่อ OLED, RTC DS3231, และ MPU6050
- **SPI (CS=5, SCK=18, MISO=19, MOSI=23):** ต่อ Micro SD Card
- **UART2 (RX2=16, TX2=17):** ต่อไปที่ Mega ผ่าน Level Converter
- **Pin 4:** ปุ่มกดเปลี่ยนโหมดที่ตัวกล่อง

#### PlatformIO Configuration
สำหรับใช้งานกับ PlatformIO IDE สร้างไฟล์ `platformio.ini`:

```ini
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
monitor_speed = 115200
lib_deps =
    adafruit/Adafruit SSD1306 @ ^2.5.7
    adafruit/Adafruit GFX Library @ ^1.11.5
    adafruit/Adafruit MPU6050 @ ^2.2.4
    adafruit/Adafruit Unified Sensor @ ^1.1.9
    adafruit/RTClib @ ^2.1.1
    bblanchon/ArduinoJson @ ^6.21.2
```

#### ESP32 Main Code

#include <WiFi.h>
#include <esp_now.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>
#include <HardwareSerial.h>
#include <Wire.h>
#include <Adafruit_SSD1306.h>
#include <RTClib.h>
#include <SPI.h>
#include <SD.h>
#include <Adafruit_MPU6050.h>
#include <Adafruit_Sensor.h>

// --- ESP-NOW Data Structure (ต้องตรงกับ Transmitter) ---
typedef struct struct_message {
    int speed;
    int steer;
    bool rocket;
    int mode;
} struct_message;

struct_message incomingData;

// --- ตั้งค่า WiFi ---
const char* ssid = "YOUR_WIFI_SSID";
const char* password = "YOUR_WIFI_PASSWORD";
const char* serverUrl = "http://YOUR_PC_IP:3000/api/telemetry";

// --- Pins ---
#define SD_CS 5
#define MODE_BUTTON 4

// --- Objects ---
HardwareSerial MegaSerial(2);
Adafruit_SSD1306 display(128, 64, &Wire, -1);
RTC_DS3231 rtc;
Adafruit_MPU6050 mpu;

// --- Variables ---
String currentMode = "WAITING";
bool manualOverride = false;

struct TelemetryData {
  int distFront = 0, distBack = 0, distLeft = 0, distRight = 0;
  float pitch = 0.0, roll = 0.0;
  int speed = 0, steer = 0;
} telemetry;

// Function Prototypes
void handleNotebookCommand(String cmd);
void parseMegaData(String data);
void sendTelemetryToDashboard();
void updateOLED();

// ESP-NOW Callback
void OnDataRecv(const uint8_t * mac, const uint8_t *incoming, int len) {
  memcpy(&incomingData, incoming, sizeof(incomingData));
  
  // Forward to Mega via Serial
  if (incomingData.mode == 1) { // Manual Mode
      String cmd = "MOVE:";
      if (incomingData.speed > 0) cmd += "FWD:";
      else if (incomingData.speed < 0) cmd += "BCK:";
      else cmd += "STP:"; // Stop if speed is 0
      
      cmd += String(abs(incomingData.speed));
      MegaSerial.println(cmd);
      
      // Handle Steer if speed is 0
      if (incomingData.speed == 0 && abs(incomingData.steer) > 50) {
          if (incomingData.steer > 0) MegaSerial.println("MOVE:RGT:150");
          else MegaSerial.println("MOVE:LFT:150");
      }
  }
}

void setup() {
  Serial.begin(115200);
  MegaSerial.begin(9600, SERIAL_8N1, 16, 17);
  
  pinMode(MODE_BUTTON, INPUT_PULLUP);
  Wire.begin(21, 22);
  
  // 1. ESP-NOW Setup
  WiFi.mode(WIFI_STA);
  if (esp_now_init() != ESP_OK) {
    Serial.println("Error initializing ESP-NOW");
  }
  esp_now_register_recv_cb(esp_now_recv_cb_t(OnDataRecv));

  // 2. Hardware Init
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) Serial.println("OLED Fail");
  if (!rtc.begin()) Serial.println("RTC Fail");
  if (!mpu.begin()) Serial.println("MPU Fail");
  if (!SD.begin(SD_CS)) Serial.println("SD Fail");

  // 3. Connect WiFi (Optional for Telemetry)
  WiFi.begin(ssid, password);
}

void loop() {
  // รับข้อมูลจาก Mega (Sensors)
  if (MegaSerial.available()) {
    String megaData = MegaSerial.readStringUntil('\n');
    parseMegaData(megaData);
  }

  // อ่าน IMU
  sensors_event_t a, g, temp;
  mpu.getEvent(&a, &g, &temp);
  telemetry.pitch = atan2(a.acceleration.y, a.acceleration.z) * 180.0 / PI;
  telemetry.roll = atan2(a.acceleration.x, a.acceleration.z) * 180.0 / PI;

  updateOLED();
  delay(100); 
}

void parseMegaData(String data) {
  if (data.startsWith("DIST:")) {
    // Parse distances and update telemetry struct
  }
}

void updateOLED() {
  display.clearDisplay();
  display.setTextSize(1);
  display.setCursor(0,0);
  display.printf("MODE: %d\n", incomingData.mode);
  display.printf("SPD: %d STR: %d\n", incomingData.speed, incomingData.steer);
  display.printf("F-DIST: %d\n", telemetry.distFront);
  display.display();
}
```

**หมายเหตุสำคัญ:**
1. **PlatformIO:** โค้ดนี้รองรับทั้ง Arduino IDE และ PlatformIO (แนะนำ PlatformIO สำหรับการจัดการ Library ที่ง่ายกว่า)
2. **Libraries Required:** ดูรายการใน `platformio.ini` หรือติดตั้งผ่าน Arduino Library Manager
3. **WiFi Configuration:** เปลี่ยน `YOUR_WIFI_SSID`, `YOUR_WIFI_PASSWORD`, และ `YOUR_PC_IP` ให้ตรงกับระบบของคุณ
4. **Data Format:** Mega ต้องส่งข้อมูลในรูปแบบ `DIST:25,80,40,42` (Front, Back, Left, Right)
5. **Code Optimization:** โค้ดได้รับการปรับปรุงให้กระชับและใช้ Function Prototypes สำหรับ PlatformIO

---

### ฝั่ง ESP32-CAM (Vision System)
รับผิดชอบ: บันทึกภาพลง SD Card เมื่อได้รับคำสั่ง

**ฟีเจอร์:**
- บันทึกภาพลง MicroSD Card เมื่อได้รับคำสั่ง 'C'
- รองรับ Stream ภาพผ่าน WiFi (สำหรับ Dashboard - ขั้นสูง)
- รับคำสั่งผ่าน Serial จาก ESP32 Main

#### PlatformIO Configuration
สำหรับ ESP32-CAM สร้างไฟล์ `platformio.ini`:

```ini
[env:esp32cam]
platform = espressif32
board = esp32cam
framework = arduino
monitor_speed = 115200
```

#### ESP32-CAM Code (Optimized)

```cpp
#include "esp_camera.h"
#include "FS.h"
#include "SD_MMC.h"

// AI-Thinker Pin Map
#define PWDN_GPIO_NUM     32
#define RESET_GPIO_NUM    -1
#define XCLK_GPIO_NUM      0
#define SIOD_GPIO_NUM     26
#define SIOC_GPIO_NUM     27
#define Y9_GPIO_NUM       35
#define Y8_GPIO_NUM       34
#define Y7_GPIO_NUM       39
#define Y6_GPIO_NUM       36
#define Y5_GPIO_NUM       21
#define Y4_GPIO_NUM       19
#define Y3_GPIO_NUM       18
#define Y2_GPIO_NUM        5
#define VSYNC_GPIO_NUM    25
#define HREF_GPIO_NUM     23
#define PCLK_GPIO_NUM     22

int photoCount = 0;

void setup() {
  Serial.begin(115200);
  
  camera_config_t config;
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer = LEDC_TIMER_0;
  config.pin_d0 = Y2_GPIO_NUM;
  config.pin_d1 = Y3_GPIO_NUM;
  config.pin_d2 = Y4_GPIO_NUM;
  config.pin_d3 = Y5_GPIO_NUM;
  config.pin_d4 = Y6_GPIO_NUM;
  config.pin_d5 = Y7_GPIO_NUM;
  config.pin_d6 = Y8_GPIO_NUM;
  config.pin_d7 = Y9_GPIO_NUM;
  config.pin_xclk = XCLK_GPIO_NUM;
  config.pin_pclk = PCLK_GPIO_NUM;
  config.pin_vsync = VSYNC_GPIO_NUM;
  config.pin_href = HREF_GPIO_NUM;
  config.pin_sscb_sda = SIOD_GPIO_NUM;
  config.pin_sscb_scl = SIOC_GPIO_NUM;
  config.pin_pwdn = PWDN_GPIO_NUM;
  config.pin_reset = RESET_GPIO_NUM;
  config.xclk_freq_hz = 20000000;
  config.pixel_format = PIXFORMAT_JPEG;
  config.frame_size = FRAMESIZE_VGA;  // 640x480
  config.jpeg_quality = 12;
  config.fb_count = 1;

  if (esp_camera_init(&config) != ESP_OK) return;
  if (!SD_MMC.begin()) Serial.println("SD Fail");
}

void loop() {
  if (Serial.available()) {
    char cmd = Serial.read();
    if (cmd == 'C') {
      camera_fb_t * fb = esp_camera_fb_get();
      if (!fb) return;
      String path = "/photo_" + String(++photoCount) + ".jpg";
      File file = SD_MMC.open(path.c_str(), FILE_WRITE);
      if (file) {
        file.write(fb->buf, fb->len);
        file.close();
      }
      esp_camera_fb_return(fb);
    }
  }
}
```

**หมายเหตุ:**
- โค้ดนี้เน้นความเรียบง่ายและประสิทธิภาพสูงสุด
- ใช้ VGA resolution (640x480) เพื่อความเร็วในการบันทึก
- รับคำสั่ง 'C' จาก ESP32 Main ผ่าน Serial เพื่อถ่ายภาพ
- ภาพจะถูกบันทึกเป็น `/photo_1.jpg`, `/photo_2.jpg`, ...

---

## 3. การเชื่อมต่อกับ Next.js Dashboard

### แนวคิดการทำงาน
1. **ESP32-CAM** ทำหน้าที่เป็น HTTP Server ส่ง MJPEG Stream
2. **Next.js Dashboard** ดึงภาพจาก ESP32-CAM ผ่าน `<img src="http://[ESP32-CAM-IP]/stream" />`
3. **ESP32 Main** ส่งข้อมูลเซนเซอร์ไปยัง Next.js ผ่าน WebSocket หรือ HTTP POST

### ตัวอย่าง Dashboard Structure (Next.js)
```
micro-ai-robot-rover/
├── src/
│   ├── app/
│   │   ├── api/
│   │   │   └── rover/
│   │   │       ├── command/route.ts    # รับคำสั่งจาก UI
│   │   │       └── telemetry/route.ts  # รับข้อมูลจาก ESP32
│   │   ├── dashboard/
│   │   │   └── page.tsx                # หน้า Dashboard หลัก
│   ├── components/
│   │   ├── VideoFeed.tsx               # แสดงภาพจาก ESP32-CAM
│   │   ├── ControlPanel.tsx            # ปุ่มควบคุม
│   │   └── StatusMonitor.tsx           # แสดงค่าเซนเซอร์
```

### ตัวอย่าง Component: VideoFeed
```typescript
// src/components/VideoFeed.tsx
'use client';

export default function VideoFeed({ cameraIP }: { cameraIP: string }) {
  return (
    <div className="relative w-full aspect-video bg-gray-900 rounded-lg overflow-hidden">
      <img 
        src={`http://${cameraIP}/stream`}
        alt="Rover Camera Feed"
        className="w-full h-full object-cover"
      />
    </div>
  );
}
```
