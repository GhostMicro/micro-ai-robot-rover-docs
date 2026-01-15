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

#### 2. Communication with Body (Nano)
| Component | Pin | STM32 Pin | Note |
| :--- | :--- | :--- | :--- |
| **LLC (Level Shift)** | HV | 5V | To Nano 5V |
| (3.3V <-> 5V) | LV | 3.3V | To STM32 3.3V |
| | GND | GND | Common Ground |
| **Nano Link** | TX2 | PA2 | **Serial 2 TX** -> LLC -> Nano RX |
| (via LLC) | RX2 | PA3 | **Serial 2 RX** -> LLC -> Nano TX |

#### 3. Motion & Effect (Direct)
| Component | Pin | STM32 Pin | Note |
| :--- | :--- | :--- | :--- |
| **Buzzer** | I/O | PB0 | PWM Speaker (Sound FX) |
| **NeoPixel** | DIN | PA8 | Status LED Eye (WS2812) |
| **Head Servo** | SIG | PB1 | Pan Servo (Scanner) |

#### 4. Perception (Sensors)
| Component | Pin | STM32 Pin | Note |
| :--- | :--- | :--- | :--- |
| **IMU (Gyro)** | SCL | PB6 | **I2C1** MPU6050 (Balance/Tilt) |
| | SDA | PB7 | **I2C1** MPU6050 |
| | INT | PB5 | **[NEW]** Interrupt |
| **Ultrasonic** | TRIG | PB12 | **[NEW]** Distance Trig |
| | ECHO | PB13 | **[NEW]** Distance Echo |
| **Battery** | V-DIV | PB8 | **[NEW]** Voltage Sense (ADC) |
| **TOF Lidar** | XSHUT | PB14 | **[NEW]** (Optional VL53L0X Control) |

---

### C. Advanced Electronics & Safety (Professional Grade)
To ensure long-term stability and protection, add these discrete components:

**1. Power Control & Monitoring (The "Cockpit")**
*   **Main Switch:** Connect in **Series** with Battery (+) before everything else.
*   **V/A Meter (Panel):**
    *   **Voltmeter:** Connect after Main Switch (Parallel).
    *   **Ammeter:** Connect in Series (after Switch, before Load).
    *   *Effect:* You can see battery health and motor load instantly without checking the PC.

**2. Power Conditioning (Filter Noise)**
*   **Electrolytic Capacitor (100µF - 470µF 16V):** Place across valid `+5V` and `GND` rails (at entry point).
*   **Ceramic Capacitor (0.1µF / Code 104):** Place close to **every** module's VCC/GND (ESP32, STM32, OLED) to filter high-frequency noise.
*   **Diode (1N4001):** Series with Main Battery (+) for Reverse Polarity Protection.

**3. Buzzer Driver Circuit (Loud & Safe)**
Do not connect Buzzer directly to MCU Pin. Use a Transistor switch:
*   **NPN Transistor (BC547 / 2N3904):**
    *   **Collector (C):** To Buzzer (-) -> Buzzer (+) to 5V.
    *   **Emitter (E):** To GND.
    *   **Base (B):** Via **1kΩ Resistor** to STM32 Pin PB0.
*   **Flyback Diode (1N4148):** Connect across Buzzer terminals (Cathode to +5V) to absorb spikes.

**4. Motor Noise Suppression**
*   **Ceramic Cap (0.1µF / 104):** Solder directly across the terminals of each DC motor to reduce RF interference.

**4. Logic Level Converter (Discrete Option)**
If not using a module, build a voltage divider for **5V TX -> 3.3V RX**:
*   **R1 (2.2kΩ):** Signal 5V -> STM32 RX.
*   **R2 (3.3kΩ):** STM32 RX -> GND.
*(Note: For 3.3V TX -> 5V RX, direct connection is usually safe enough for logic high, or use a 2N7000 MOSFET).*
