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
**CRITICAL:** Pins 16/17 caused Flash errors. Updated to Safe Pins.

| Component | Pin | ESP32 GPIO | Note |
| :--- | :--- | :--- | :--- |
| **STM32 Link** | RX2 | 34 | Connect to STM32 TX (PB10) |
| | TX2 | 27 | Connect to STM32 RX (PB11) |
| **I2C Bus** | SDA | 21 | MPU6050 & OLED |
| | SCL | 22 | |
| **Power** | VIN | 5V | Shared Power Bus |

### B. STM32 Brain (Main Controller)
*No changes in wiring for V4.5.*

### C. Nano Body Driver (Actuators & Sensors)
| Component | Pin | Nano Pin | Note |
| :--- | :--- | :--- | :--- |
| **Motors L** | IN1/IN2 | D7, D8 | Left Side Wheels |
| **Motors R** | IN3/IN4 | D9, D10 | Right Side Wheels |
| **Headlight** | SIG | A3 | White High-Power LED |
| **Signals** | L-SIG | A0 | **[NEW]** Shifted for US compatibility |
| | R-SIG | A1 | **[NEW]** Shifted for US compatibility |
| **Ultrasonic F** | TRIG/ECHO| D2, D3 | Front Clearance |
| **Ultrasonic L** | TRIG/ECHO| D4, D11 | Left Clearance |
| **Ultrasonic R** | TRIG/ECHO| D12, A5 | Right Clearance |
| **Ultrasonic B** | TRIG/ECHO| TBD | Rear Clearance |

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
