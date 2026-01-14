# ภาพรวมโครงการและฮาร์ดแวร์ (Project Overview & Hardware)

## โครงสร้างระบบ (System Architecture)

ระบบถูกออกแบบโดยแบ่งออกเป็น 2 ส่วนหลักอย่างชัดเจน คือ **ส่วนสมอง (Brain)** และ **ส่วนร่างกาย (Body)**

![Architecture Diagram](assets/images/architecture_diagram.png)

### 1. ส่วนสมอง (The Brain)
ทำหน้าที่: สั่งการระดับสูง, ประมวลผลภาพ, สื่อสารไร้สาย, และเก็บข้อมูล Log
อุปกรณ์หลัก:
*   **ESP32 (Main Controller):** "กัปตัน" ผู้สั่งการ เชื่อมต่อ WiFi/ESP-NOW และตัดสินใจเส้นทาง
*   **ESP32-CAM:** "ดวงตา" ทำ Vision AI (Face Detection, Object Tracking) และสตรีมภาพ
*   **Sensors:**
    *   **GY-521 (IMU):** รับรู้ทิศทางและความเอียง (Orientation)
    *   **HW-611 (Barometer):** รับรู้ความสูงและอุณหภูมิ
    *   **RTC Module (DS3231):** "นาฬิกา" ให้เวลาที่แม่นยำสำหรับการบันทึก Log (Time-stamping)
    *   **Micro SD Card:** กล่องดำบันทึกข้อมูล (Black Box)

### 2. ส่วนร่างกาย (The Body)
ทำหน้าที่: ขับเคลื่อน, ใช้กำลังงานสูง, และรับรู้สิ่งกีดขวางระยะใกล้
อุปกรณ์หลัก:
*   **Arduino Mega 2560:** "คนขับ" รับคำสั่งจากกัปตันแล้วสั่งมอเตอร์ ควบคุมเซนเซอร์รอบคัน
*   **Motor Driver (L298N / MOSFET):** ขยายสัญญาณเพื่อขับมอเตอร์
*   **Ultrasonic Sensors:** "ผิวกาย" ตรวจจับการชนรอบทิศทาง
*   **Battery System:** แหล่งพลังงานแยกส่วน (Logic vs Motor)

---

## รายการอุปกรณ์ (Bill of Materials)

| ประเภท           | รุ่น/เบอร์               | จำนวน | หน้าที่               |
| :--------------- | :-------------------- | :--- | :----------------- |
| **Brain**        | ESP32 DevKit V1       | 1    | สมองหลัก            |
| **Vision**       | ESP32-CAM             | 1    | กล้อง AI            |
| **Clock/Time**   | DS3231 (RTC)          | 1    | นาฬิกาบอกเวลาจริง    |
| **Body Control** | Arduino Mega 2560     | 1    | คุมมอเตอร์/เซนเซอร์   |
| **IMU**          | GY-521 (MPU6050)      | 1    | เข็มทิศ/Gyro         |
| **Env Sensor**   | HW-611 (BMP280)       | 1    | วัดความสูง           |
| **Distance**     | HC-SR04 / US-016      | 4    | วัดระยะรอบคัน        |
| **Storage**      | Micro SD Module       | 1    | เก็บ Log            |
| **Motor Drive**  | L298N / Module        | 1    | ขับมอเตอร์           |
| **Power**        | 18650 Li-ion          | 2-4  | แบตเตอรี่            |
| **Converter**    | Logic Level Converter | 1    | แปลงไฟ 3.3V <-> 5V |
