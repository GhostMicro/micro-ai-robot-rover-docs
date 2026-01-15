# 📦 Ghost Micro Inventory (User's Armory)

This document serves as a database of available components to ensure future designs utilize existing resources efficiently.

## 🧠 Integrated Circuits (ICs)
| Component | Description | Potential Uses |
| :--- | :--- | :--- |
| **NE555 (Classic)** | Precision Timer | **Must Use:** Hardware Watchdog (Reset STM32 if frozen), Backup Siren (Police Tone), PWM generator independent of MCU. |
| **LM7805** | 5V Voltage Regulator | Stable 5V for Logic/Sensors from higher voltage battery. |
| **LM7812** | 12V Voltage Regulator | 12V rail for high-power LEDs or solenoid (if applicable). |

## 🕹️ Transistors & Drivers
| Type | Part Number | Description | Application |
| :--- | :--- | :--- | :--- |
| **NPN** | **2N3904** | General Purpose | Signal switching, LED driver (Low current), Buzzer driver. |
| **NPN** | **BC547** | General Purpose | Logic inverter, Level Shifter (Discrete). |
| **NPN** | **C9013** | High Current Audio | Speaker driver, Small motor driver (~500mA). |
| **PNP** | **2N3906** | General Purpose | High-side switch, Complementary pair with 3904. |
| **PNP** | **BC557** | General Purpose | High-side logic switch. |
| **NPN** | **S8050** | High Current | Medium power switching (Relays, High brightness LEDs). |
| **?** | **CTBC5478** | *(Need Verification)* | Likely general purpose variant. |
| **?** | **F708** | *(Need Verification)* | Special function? |

## 🧠 Microcontrollers & Isolation
| Component | Description | Potential Uses |
| :--- | :--- | :--- |
| **Arduino Nano** | **Current Body Controller** | Handles Motor I/O and Encoders (5V logic). Keep utilizing this! |
| **ESP8266 (D1 Mini)** | WiFi / IoT Node | Excellent for **Wireless Remote Sensors** (e.g., place in room corner to send temp/activity data to Robot via ESP-NOW). |
| **Arduino Pro Mini** | **Priority Use** (Have many) | **3.3V Version:** Perfect Slave for STM32 (Direct Connect). **5V Version:** Great for driving Servos/LEDs via Optocoupler. Use as dedicated "Sensor Hubs". |
| **ATtiny85** | Mini MCU (8-pin) | Dedicated sensor processor, Smart LED controller (Neopixel), Independent watchdog. |
| **ATmega8-16PU** | Classic AVR MCU | Slave I/O expander, Standalone subsystem controller. |
| **PC817** | Optocoupler | **High Priority:** 100% Electrical Isolation between Motor Power and STM32 Logic. |

## 📐 Logic & Analog ICs
| Component | Description | Application |
| :--- | :--- | :--- |
| **CD4051BE** | 8-Channel Analog Multiplexer | Expand ADC pins (Read 8 sensors with 1 STM32 pin). |
| **CD4511BE** | BCD to 7-Segment Latch/Decoder | Drive a retro numeric display (0-9) using only 4 pins. |
| **CD4017BE** | Decade Counter | Running light effects (Knight Rider style) without coding. |
| **CD4027BE** | Dual JK Flip-Flop | Toggle switch logic, Hardware state memory. |
| **MC14011BCP** | Quad 2-Input NAND Gate | Basic logic glue, Debounce circuit, Oscillator. |
| **MC14029BCP** | Presettable Up/Down Counter | Hardware event counting. |

## ⚡ Diodes & Protection
| Part Number | Type | Max Current | Use Case |
| :--- | :--- | :--- | :--- |
| **1N4001** | Rectifier | 1A | Reverse polarity protection (Main input), Bridge rectifier. |
| **1N4148** | Signal / Switching | 300mA | Back-EMF protection (Relays/Buzzers), Logic gates (Diode logic). |
| **1N914** | Signal | 300mA | Similar to 4148, High speed switching. |

## 🔋 Passive Components
| Category | Value / Code | Description | Use Case |
| :--- | :--- | :--- | :--- |
| **Potentiometer** | **VR504** (500kΩ) | Trim Pot | Adjustable sensitivity, Analog input calibration. |
| **Capacitor** | **105** (1µF) | Ceramic / Mylar | Audio coupling, Timing (555 Timer). |
| **Capacitor** | **Electrolytic** | (Various) | Power rail smoothing, Bulk energy storage. |
| **Resistor** | **2W - 4W** | High Power | Current sensing, Dummy load, High power LED limiters. |

## 🕵️ Recommended Scavenger Hunt (Things to look for!)
Since you have 100+ modules, please look for these specific ones to upgrade Ghost Micro:

| Function | Recommended Module | Why? |
| :--- | :--- | :--- |
| **Gyro/IMU** | **MPU6050** or **BNO055** | Essential for self-balancing or precise turning (90/180 degrees). |
| **Distance** | **VL53L0X** (Laser) or **HC-SR04** (Ultrasonic) | Laser is faster/smaller. Ultrasonic is cheaper/robust. |
| **Display** | **OLED 0.96 (I2C)** | You likely use this on the remote, but one on the robot face would be cool. |
| **Environment** | **DHT11 / DHT22** | Temp/Humidity monitoring. |
| **Light** | **LDR (Photoresistor)** | Auto-headlights when dark. |

## 🔮 Future Potentials (NFC & Cheap Sensors)
*   **NFC/RFID Readers (RC522):** You mentioned having many. We can use these for:
    *   **Robot Identity:** Tap card to switch robot "Personality" (Aggressive/Passive).
    *   **Security Key:** Unlock controls only with your Master Card.
*   **Budget Sensors:** (IR Obstacle Avoidance, Flame Sensors, Tilt Switches)
    *   Great for swarm robotics or "expendable" scout units.

---
**📝 Agent Note:** I will check this list before suggesting new hardware purchase. We will prioritize using these components in all future schematic designs.
