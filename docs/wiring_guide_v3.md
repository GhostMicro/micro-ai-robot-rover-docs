# Ghost Micro Robot - Wiring Guide v3.0

## 1. ESP32 Transmitter (Remote)
| Component | Pin | ESP32 GPIO | Note |
| :--- | :--- | :--- | :--- |
| **OLED I2C** | SDA | 21 | Display |
| | SCL | 22 | |
| **Joystick** | VRx | 34 | Analog X |
| | VRy | 35 | Analog Y |
| | SW | 32 | Button |
| **Buzzer** | VCC | 3.3V | |
| | GND | GND | |
| | I/O | 25 | **[NEW]** Active Buzzer |
| **Power** | VIN | 5V | From USB/Battery |

---

## 2. Rover System (Receiver + Brain)

### A. ESP32 Receiver (Gateway)
| Component | Pin | ESP32 GPIO | Note |
| :--- | :--- | :--- | :--- |
| **STM32 Link** | RX2 | 16 | Connect to STM32 TX |
| | TX2 | 17 | Connect to STM32 RX |
| **Power** | VIN | 5V | Shared Power Bus |

### B. STM32 Brain (Main Controller)
| Component | Pin | STM32 Pin | Note |
| :--- | :--- | :--- | :--- |
| **ESP32 Link** | TX1 | PA9 | Connect to ESP32 RX2 |
| | RX1 | PA10 | Connect to ESP32 TX2 |
| **Micro SD** | CS | PA4 | **[NEW]** Chip Select |
| (SPI1) | SCK | PA5 | **[NEW]** Clock |
| | MISO | PA6 | **[NEW]** |
| | MOSI | PA7 | **[NEW]** |
| **Buzzer** | I/O | PB0 | **[NEW]** PWM Speaker |
| **Motors** | L-PWM | PA0 | Left Motor Speed |
| | L-DIR | PA1 | Left Motor Dir |
| | R-PWM | PA2 | Right Motor Speed |
| | R-DIR | PA3 | Right Motor Dir |
