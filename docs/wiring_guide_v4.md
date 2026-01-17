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
**CORE UPDATE:** Using **H969-U V2.0 (The Green Power Bank Board)**.
*   **Battery:** Use **18650 Li-Ion** (3.7V) connected in Parallel.
*   **Connection:** Connect Battery (+) to **B+** and (-) to **B-**.
*   **Output:** Take 5V from the **USB Output** (solder to pads or use USB cable) to system `VIN/5V`.

```mermaid
graph LR
    BAT[Li-Ion 3.7V (18650)] -->|Parallel| BOARD[H969-U V2.0]
    BOARD -->|B+| BAT_P(+)
    BOARD -->|B-| BAT_N(-)
    
    BOARD -->|USB Out 5V| SW[Main Switch]
    SW -->|5V Regulated| SYS[System Power Bus (5V)]
    
    SYS -->|Red| RX[ESP32 Receiver]
    SYS -->|Red| BR[STM32 Brain]
    
    subgraph "Built-in Features"
    BOARD --> LCD[LCD Info Display %]
    BOARD --> CHG[Micro USB Charge Port]
    end
```
*   **Main Switch:** Connect wires from the H969-U 5V Output -> Switch -> System.
*   **No Regulator Needed:** The H969-U does the 5V Boost automatically!

**2. Power Conditioning (Filter Noise)**
*   **Electrolytic Capacitor (100µF - 470µF 16V):** Place across valid `+5V` and `GND` rails (at entry point).
*   **Ceramic Capacitor (0.1µF / Code 104):** Place close to **every** module's VCC/GND (ESP32, STM32, OLED).

**3. Buzzer Driver Circuit (Loud & Safe)**
*Use this because the Buzzer draws too much current for the STM32 pin.*
```mermaid
graph TD
    P[STM32 Pin PB0] -->|1k Resistor| B(Base)
    B --> T[NPN Transistor]
    
    VCC[5V] -->|Red| BUZ[Buzzer (+)]
    BUZ -->|Black| C(Collector)
    C --> T
    
    T --> E(Emitter)
    E --> GND[Ground]
```
*   **Check:** Transistor Emitter -> GND, Collector -> Buzzer -.

**4. Motor Noise Suppression**
*   **Ceramic Cap (0.1µF / 104):** Solder directly across the terminals of each DC motor.

**5. Logic Level Converter (Voltage Divider)**
*Protecting STM32 (3.3V) from Nano (5V)*
```mermaid
graph LR
    N[Nano TX (5V)] -->|R1 2.2k| M[Middle Point]
    M -->|Wire| S[STM32 RX (3.3V)]
    M -->|R2 3.3k| G[Ground]
```
If not using a module, build a **Voltage Divider** for 5V TX -> 3.3V RX.
