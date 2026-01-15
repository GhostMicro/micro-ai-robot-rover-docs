# Ghost Micro Robot - Wiring Guide v4.0 (Full Armor Edition)

## 1. ESP32 Transmitter (Remote)
| Component | Pin | ESP32 GPIO | Note |
| :--- | :--- | :--- | :--- |
| **OLED I2C** | SDA | 21 | Display |
| | SCL | 22 | |
| **Joystick** | VRx | 34 | Analog X |
| | VRy | 35 | Analog Y |
| | SW | 32 | Button |
| **Buzzer** | I/O | 25 | Active Buzzer (Boot Sound) |
| **Power** | VIN | 5V | From USB/Battery |

---

## 2. Rover System (Receiver + Brain)

### A. ESP32 Receiver (Gateway)
No change in wiring. Just acts as a bridge.
| Component | Pin | ESP32 GPIO | Note |
| :--- | :--- | :--- | :--- |
| **STM32 Link** | RX2 | 16 | Connect to STM32 TX (PB10) |
| | TX2 | 17 | Connect to STM32 RX (PB11) |
| **Power** | VIN | 5V | Shared Power Bus |

### B. STM32 Brain (Main Controller) - FULL SENSORS

#### 1. Communication & Storage
| Component | Pin | STM32 Pin | Note |
| :--- | :--- | :--- | :--- |
| **ESP32 Link** | TX2 | PB10 | **Serial 3 TX** (To ESP32 RX) |
| | RX2 | PB11 | **Serial 3 RX** (To ESP32 TX) |
| **Micro SD** | CS | PA4 | **SPI1** Chip Select |
| | SCK | PA5 | **SPI1** Clock |
| | MISO | PA6 | **SPI1** Data In |
| | MOSI | PA7 | **SPI1** Data Out |

#### 2. Motion & Effect
| Component | Pin | STM32 Pin | Note |
| :--- | :--- | :--- | :--- |
| **Motors** | L-PWM | PA0 | Left Motor Speed |
| | L-DIR | PA1 | Left Motor Dir |
| | R-PWM | PA2 | Right Motor Speed |
| | R-DIR | PA3 | Right Motor Dir |
| **Buzzer** | I/O | PB0 | PWM Speaker (Sound FX) |
| **NeoPixel** | DIN | PA8 | **[NEW]** Status LED Eye (WS2812) |
| **Head Servo** | SIG | PB1 | **[NEW]** Pan Servo (Scanner) |

#### 3. Perception (Sensors)
| Component | Pin | STM32 Pin | Note |
| :--- | :--- | :--- | :--- |
| **IMU (Gyro)** | SCL | PB6 | **I2C1** MPU6050 (Balance/Tilt) |
| | SDA | PB7 | **I2C1** MPU6050 |
| | INT | PB5 | **[NEW]** Interrupt |
| **Ultrasonic** | TRIG | PB12 | **[NEW]** Distance Trig |
| | ECHO | PB13 | **[NEW]** Distance Echo |
| **Battery** | V-DIV | PB8 | **[NEW]** Voltage Sense (ADC) |
| **TOF Lidar** | XSHUT | PB14 | **[NEW]** (Optional VL53L0X Control) |
