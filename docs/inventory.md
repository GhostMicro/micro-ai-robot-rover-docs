# 📦 Ghost Micro Inventory (User's Armory)

This document serves as a database of available components to ensure future designs utilize existing resources efficiently.

## 🧠 SBCs & Microcontrollers
| Component | Qty | Description | Potential Uses |
| :--- | :---: | :--- | :--- |
| **ESP32 WROOM** | 10 | High-performance WiFi/BT MCU | Main robot controller, heavy computation, camera tasks. |
| **Arduino Nano** | 1 | Classic 5V MCU | Body controller, Motor I/O, Encoder handling. |
| **Arduino Pro Mini 3.3V** | 5+ | Small form factor 3.3V MCU | Perfect Slave for STM32, dedicated sensor hubs. |
| **Arduino Pro Mini 5V** | 5 | Small form factor 5V MCU | Driving servos/LEDs via optocouplers. |
| **Arduino Mega 2560** | 1 | High I/O MCU | Master controller for complex logic. |
| **STM32 (Blue Pill)** | 5 | High-speed 32-bit MCU | Main processor for Ghost Micro Rover. |
| **Pixhawk 2.4.8** | 🛒 | Advanced Flight Controller | **V5 Upgrade:** Primary stabilization and Nav Brain. |
| **ESP8266 (D1 Mini)** | 3 | WiFi / IoT Node | Wireless remote sensors, ESP-NOW nodes. |
| **Arduino Uno R3 (DIP/SMD)**| 2 | Standard Prototyping Board | Desktop testing, shield compatibility. |
| **Arduino D1** | 1 | ESP8266 in Uno form factor | WiFi projects requiring Uno shields. |
| **ATtiny85** | 1 | Mini MCU (8-pin) | Dedicated sensor processor, Smart LED controller. |
| **Raspberry Pi 4B (4G)** | 1 | High-power SBC | Edge AI, computer vision, server host. |
| **Raspberry Pi Zero** | 1 | Compact Linux SBC | Low-power IoT gateway, small Linux projects. |
| **TTGO (ESP32 w/ Display)** | 4 | ESP32 board with OLED/TFT | Handheld controllers, status monitors. |
| **Air101** | 4 | LuatOS based MCU | Efficient, low-power scripts, specialized tasks. |
| **ESP-01** | 5 | Smallest ESP8266 module | Tiny WiFi bridge, sensor nodes. |
| **NodeMcu Base ver 1.0** | 10 | Breadboard breakout shield | Easy wiring for NodeMcu/ESP8266 projects. |

## 📡 Wireless & Communication
| Component | Qty | Description | Application |
| :--- | :---: | :--- | :--- |
| **NRF24L01** | 2 | 2.4GHz Transceiver | Short-range low power robot-to-remote comms. |
| **NRF24L01 PA LNA** | 2 | Long-range Transceiver | High-power communication with external antenna. |
| **HC-05 (ZS-040)** | 1 | Bluetooth Serial Module | Smartphone control, wireless debugging. |
| **FlySky FS-iA6B** | 🛒 | 2.4GHz 6CH Receiver | **V5 Upgrade:** Manual RC control (iBus/PPM). |
| **PPM Encoder** | 1 | PWM to PPM Signal Converter | **V5 Upgrade:** Connect non-PPM receivers to Pixhawk. |
| **I2C Level Converter** | 1 | 4/8-ch Logic Shifter | Interfacing 5V sensors with 3.3V MCUs (ESP32/STM32). |
| **WCMCU-0108** | 1 | 8-bit Level Shifter | High-speed logic translation. |
| **NFC Reader (RC522)** | 2 | RFID/NFC Module | Security key, personality switching. |

## 🔋 Power & Battery Management
| Component | Qty | Description | Application |
| :--- | :---: | :--- | :--- |
| **H969-U V2.0** | 5 | Power Bank Module (w/ LCD) | **Main Power Hub:** 3.7V to 5V Boost + Charging + Display. |
| **Power Module GM V1.0**| 1 | Pixhawk Power & Monitoring | **V5 Upgrade:** Clean 5.3V for FC + Volt/Curr Sensing. |
| **H913-A V2.0** | 1 | Battery Management | Alternative charging/boost solution. |
| **Dual 18650 Shield V8**| 2 | Battery Holder + 5V/3V Boost | Portable power for hand-held controllers or small robots. |
| **TP4056 Module** | 5 | Li-ion Battery Charger | USB charging for individual 18650 cells. |
| **CN3791 MPPT Solar** | 1 | Solar Charger | Outdoor robot power management. |
| **LM2596 Buck Converter**| 3 | Step-Down (3A Max) | High-current regulated rails (e.g., 5V from 12V). |
| **HW-106 Boost Module** | 1 | Step-Up DC-DC | Boosting low voltage batteries for logic rails. |
| **BMS 1S Protector** | 1 | Battery Protection Circuit | Safety for single Li-ion cells. |
| **HW-313 / HW-613** | 1 | Mini Buck/Boost | Compact power regulation. |
| **2-Port Buck Conv.** | 2 | Step-Down w/ Dual Output | Clean power distribution. |
| **Voltage Sensor** | 1 | Voltage Divider Module | Monitoring battery health via ADC. |

## 📡 Sensors & Perception
| Component | Qty | Description | Application |
| :--- | :---: | :--- | :--- |
| **HC-SR04** | 10 | Ultrasonic Distance | Basic obstacle avoidance. |
| **US-016** | 1 | Analog Ultrasonic Sensor | High-precision distance with analog output. |
| **HC-SR501** | 10 | PIR Motion Sensor | Security features, human detection. |
| **BME280 / BMP280** | 2 | Temp/Pressure/Humidity | Environmental scanning, altitude sensing. |
| **HW-611 / BMP180** | 1 | Barometric Pressure | Basic altitude/weather monitoring. |
| **GY-87 (10DOF)** | 1 | IMU + Mag + Baro | Complete navigation sensor (MPU6050 + HMC5883L + BMP180). |
| **Pixhawk GPS (M8N)**| 🛒 | High-gain GPS + Compass | **V5 Upgrade:** Autonomous navigation & RTH. |
| **Crius I2C Hub**   | 1 | I2C Splitter / Expansion | **V5 Upgrade:** Multiplex Compass, OLED, and MCUs. |
| **GY-521 (MPU6050)** | 1 | 6-Axis Gyro/Accel | Self-balancing, orientation tracking. |
| **LDR Photoresistor** | 2 | Light Dependent Resistor | Automatic headlights, light seeking behavior. |
| **GRAYSCALE HY-S126** | 1 | Line/Edge Tracking | Floor color detection, cliff detection. |
| **SW-420** | 1 | Vibration Sensor | Impact detection, anti-tamper. |
| **Soil Moisture** | 1 | Capacitive/Resistive | Terrestrial environment sensing. |
| **DHT11** | 10 | Temp & Humidity Sensor | Basic environmental monitoring. |
| **DHT22** | 1 | High-precision Temp/Hum | Precise environmental data. |
| **INMP441 (I2S Mic)** | 🛒 | MEMS Digital Microphone | **V4.1:** Hearing/Streaming audio to Dashboard. |

## ⚙️ Motor Drivers & Control
| Component | Qty | Description | Application |
| :--- | :---: | :--- | :--- |
| **PCA9685 16CH PWM** | 1 | I2C Servo/PWM Driver | **Priority:** Control up to 16 servos for robot legs/limbs. |
| **Servo SG92R** | 2 | High Torque Micro Servo | Robot arm or heavy linkage control. |
| **Servo SG90** | 6 | Standard Micro Servo | Basic joint movement, pan/tilt. |
| **L293D Shield** | 3 | Quad Motor Driver Shield | Driving DC motors from Arduino Uno/Mega. |
| **Mosfet SW-M211** | 10 | High-current Switch | PWM control for high power LEDs or pumps. |
| **PWM Speed Controller**| 1 | DC Motor Dimmer | Manual motor speed adjustment. |
| **HW-070 Mini PWM** | 1 | Micro PWM Driver | Compact motor/LED control. |
| **DFPlayer Mini** | 🛒 | MP3/Audio Player | **V4.1:** Speech/Playing clips from SD Card. |
| **Speaker (8Ω 3W)** | 🛒 | Mini Speaker | Output for DFPlayer. |

## 📟 Human Interface & Displays
| Component | Qty | Description | Application |
| :--- | :---: | :--- | :--- |
| **Joystick XY Module** | 10 | Dual Axis Analog + Button | Building various handheld remote controllers. |
| **4x4 Membrane Key** | 1 | Matrix Keyboard | PIN entry, mode selection. |
| **Tach Switch** | 50 | Momentary Push Buttons | Reset buttons, bumper switches, limit switches. |
| **4-Digit 7-Segment** | 1 | Numeric Display | Error codes, battery %, or timers. |
| **LCD 1602 / 1604** | 1 | Character LCD | Scrolling status logs and system info. |
| **V/A Panel Meter** | 1 | Analog/Digital Meter | Real-time hardware monitoring. |

## 📐 Logic & Analog ICs
| Component | Description | Application |
| :--- | :--- | :--- |
| **NE555** | Precision Timer | Hardware watchdog, pulse generation. |
| **CD4051BE** | 8-Channel Analog Mux | Expand ADC pins for multiple sensors. |
| **CD4511BE** | BCD to 7-Segment | Hardware-driven numeric display. |
| **CD4017BE** | Decade Counter | LED chaser effects, hardware step sequencing. |
| **CD4027BE** | Dual JK Flip-Flop | State memory, toggle logic. |
| **MC14011 / 14029** | NAND Gates / Counter | Custom hardware logic glue. |
| **Transistors** | 2N3904, BC547, S8050, etc. | Signal switching, small drivers. |

## 🧱 Passives & Protection
| Component | Description | Use Case |
| :--- | :--- | :--- |
| **PC817** | Optocoupler | **Safety:** Isolate logic from noisy motor power. |
| **Diodes** | 1N4001, 1N4148, 1N914 | Reverse polarity & Back-EMF protection. |
| **Potentiometers** | VR504 (500kΩ) | Sensitivity/Analog adjustment. |
| **Capacitors** | Ceramic, Electrolytic | Power filtering, timing circuits. |
| **Resistors** | 2W - 4W, Standard | Current limiting, pull-ups/downs. |

---
**📝 Agent Note:** This inventory is updated frequently. Prioritize using existing modules before suggesting new purchases.
