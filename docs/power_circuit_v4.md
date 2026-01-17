# Ghost Micro v4.0 - Power Circuit Diagram ⚡

## 🔋 Overview
This guide details the **Power Distribution System** for Ghost Micro v4.0.
We use a **3x 18650 Parallel Pack** to power 3 separate zones:
1.  **Brain Zone (5V):** ESP32, STM32, Nano, Sensors (Critical)
2.  **Camera Zone (5V):** ESP32-CAM (Isolated to prevent brownout)
3.  **Motor Zone (3.7V):** L298N + TT Motors (High current)

---

## 🔌 Master Wiring Diagram

```mermaid
graph TD
    %% styles
    classDef bat fill:#ffeb3b,stroke:#fbc02d,stroke-width:2px,color:black
    classDef mod fill:#e3f2fd,stroke:#2196f3,stroke-width:2px,color:black
    classDef mot fill:#ffebee,stroke:#ef5350,stroke-width:2px,color:black
    classDef gnd fill:#000,stroke:#000,stroke-width:1px,color:white

    subgraph BATTERY_PACK [Main Power Source]
        BAT[3x 18650 Parallel<br/>3.7V 7500mAh]:::bat
        SW[Main Switch]
        FUSE[Fuse 10A]
    end

    subgraph BRAIN_ZONE [Zone 1: Brain & Logic]
        H969[H969-U Boost Module<br/>IN: 3.7V -> OUT: 5V 2A]:::mod
        ESP_MAIN[ESP32 Commander]
        STM[STM32 Controller]
        NANO[Arduino Nano]
        SENS[Sensors / GPS / IMU]
    end

    subgraph CAMERA_ZONE [Zone 2: Vision]
        BUCK[LM2596 Buck Module<br/>IN: 3.7V -> OUT: 5V 3A]:::mod
        CAM[ESP32-CAM]
    end

    subgraph MOTOR_ZONE [Zone 3: Traction]
        L298[L298N Driver]:::mot
        M1[Motor FL]:::mot
        M2[Motor FR]:::mot
        M3[Motor RL]:::mot
        M4[Motor RR]:::mot
    end

    %% Wiring Connections
    BAT --> SW --> FUSE
    
    %% Distribution
    FUSE -->|3.7V| H969
    FUSE -->|3.7V| BUCK
    FUSE -->|3.7V High Current| L298

    %% Zone 1 Logic
    H969 -->|5V Stable| ESP_MAIN
    H969 -->|5V Stable| STM
    H969 -->|5V Stable| NANO
    H969 -->|5V Stable| SENS

    %% Zone 2 Logic
    BUCK -->|5V Isolated| CAM

    %% Zone 3 Logic
    L298 --> M1 & M2 & M3 & M4

    %% Common Ground
    GND[COMMON GROUND POINT]:::gnd
    BAT --- GND
    H969 --- GND
    BUCK --- GND
    L298 --- GND
    ESP_MAIN --- GND
    STM --- GND
    NANO --- GND
    CAM --- GND
```

---

## 🛠️ Step-by-Step Wiring

### 1. Battery Pack Assembly
*   **Components:** 3x 18650 Li-ion cells (3.7V each).
*   **Configuration:** **Parallel (+ to +, - to -)**.
*   **Output:** 3.7V - 4.2V, ~7500mAh.
*   **Safety:** Add a **Master Switch** and **10A Fuse** on the positive (`+`) line immediately after the battery holder.

### 2. Zone 1: Brain Power (H969-U)
This rail powers the intelligent components. It must be stable 5V.
*   **Input:** Connect Battery `+` to H969-U `IN+`, Battery `-` to H969-U `IN-`.
*   **Output:** Connect H969-U `OUT+` (5V) to the `5V` pins of ESP32, STM32, Nano.
*   **Ground:** Connect H969-U `OUT-` to the `GND` pins of all MCUs.

### 3. Zone 2: Camera Power (LM2596)
**CRITICAL:** The camera is isolated to prevent it from crashing the brain or brownout.
*   **Input:** Connect Battery `+` to LM2596 `IN+`, Battery `-` to LM2596 `IN-`.
*   **Adjust:** <span style="color:red">**IMPORTANT:** Rotate the trimmer screw on LM2596 until output is exactly 5V BEFORE connecting the camera!</span>
*   **Output:** Connect LM2596 `OUT+` to ESP32-CAM `5V` pin.
*   **Ground:** Connect LM2596 `OUT-` to ESP32-CAM `GND`.

### 4. Zone 3: Motor Power (L298N)
Motors take raw battery power.
*   **Power:** Connect Battery `+` to L298N `12V` input (it accepts 3.7V-35V).
*   **Ground:** Connect Battery `-` to L298N `GND`.
*   **Note:** <span style="color:red">**REMOVE** the 5V regulator jumper on L298N if supplying < 7V (keep it on for 3.7V operation usually, but do NOT use the 5V output of L298N to power other things).</span>

---

## ⚠️ Important Safety Checks
1.  **Common Ground:** ensure the negative (`-`) terminal of the battery connects to the GND of **ALL** modules (H969, LM2596, L298N, ESP32, etc.).
2.  **Wire Thickness:** Use thicker wires (20-18 AWG) for the **Battery -> L298N** path, as motors draw high current.
3.  **Voltage Check:** Use a multimeter to verify 5V outputs on H969 and LM2596 before connecting expensive microcontrollers.

---

## 📝 Part List
| Component | Function | Status |
| :--- | :--- | :--- |
| **3x 18650 Batteries** | Main Power Source | ✅ Ready |
| **Battery Holder (3-slot)** | Holds batteries in parallel | ✅ Ready |
| **H969-U (Green Board)** | 5V Boost for Brain | ✅ Ready |
| **LM2596 (Blue Board)** | 5V Buck/Boost for Camera | 🛒 Need |
| **SPST Switch** | Master ON/OFF | ✅ Ready |
| **Wires (Red/Black)** | Power distribution | ✅ Ready |
