# Nano Core Interface

## V.1.0

### PlatformIO

```ini
[env:nanoatmega328]
platform = atmelavr
board = nanoatmega328
framework = arduino
monitor_speed = 115200
lib_deps = 
    adafruit/Adafruit SSD1306 @ ^2.5.7
    adafruit/Adafruit GFX Library @ ^1.11.5
```

### main.cpp

```cpp
#include <Arduino.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

// ตัวแปรรับค่าจาก STM32
int8_t leftSpeed = 0;
int8_t rightSpeed = 0;
uint8_t currentMode = 0; // 0: Manual, 1: Auto

void updateDisplay();

void setup() {
    Serial.begin(115200);
    
    // เริ่มต้นจอ OLED
    if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
        for(;;); // ถ้าเชื่อมต่อจอไม่ได้ ให้หยุดรอตรงนี้
    }
    
    display.clearDisplay();
    display.setTextColor(WHITE);
    display.setTextSize(1);
    display.setCursor(20, 20);
    display.println("ROBOT READY");
    display.display();
}

void loop() {
    // รับ Packet จาก STM32: Header(0xEE) + Mode + Left + Right
    if (Serial.available() >= 4) {
        if (Serial.read() == 0xEE) {
            currentMode = Serial.read();
            leftSpeed = (int8_t)Serial.read();
            rightSpeed = (int8_t)Serial.read();
            
            updateDisplay();
        }
    }
}

void updateDisplay() {
    display.clearDisplay();
    
    // บรรทัดบนสุด: โหมดการทำงาน
    display.setTextSize(1);
    display.setCursor(0, 0);
    display.print("MODE: ");
    display.println(currentMode == 0 ? "MANUAL" : "AUTO");
    
    display.drawLine(0, 12, 128, 12, WHITE); // ขีดเส้นคั่น

    // แสดงค่าความเร็วล้อ
    display.setCursor(0, 25);
    display.print("L Speed: "); display.println(leftSpeed);
    display.print("R Speed: "); display.println(rightSpeed);

    // ทำ Progress Bar เล็กๆ ให้ดูสวยงาม
    int barL = map(abs(leftSpeed), 0, 100, 0, 50);
    display.drawRect(70, 25, 52, 7, WHITE);
    display.fillRect(71, 26, barL, 5, WHITE);

    display.display();
}
```