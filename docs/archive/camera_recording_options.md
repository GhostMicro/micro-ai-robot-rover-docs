# ESP32-CAM Recording Options Comparison

## 🎥 Option 1: PC Recording (Recommended ✅)

### Advantages
- **No SD Card Needed:** Save cost and weight (~5g)
- **Unlimited Storage:** PC has TB of space vs 32GB SD
- **Better Quality:** Can record at higher bitrate
- **Easy Playback:** Files directly on PC, no need to remove SD
- **Live Monitoring:** See what's recording in real-time
- **Post-Processing:** Easy to edit/analyze on PC

### Disadvantages
- **WiFi Required:** Must stay in range
- **PC Must Run:** Dashboard needs to be open
- **Network Lag:** Slight delay in recording

### Implementation
```python
# In dashboard.py
import cv2
import requests
import numpy as np

# Record stream from ESP32-CAM
stream_url = "http://192.168.4.1/stream"
cap = cv2.VideoCapture(stream_url)

# Save to file
fourcc = cv2.VideoWriter_fourcc(*'XVID')
out = cv2.VideoWriter('ghost_mission_001.avi', fourcc, 20.0, (320,240))

while recording:
    ret, frame = cap.read()
    out.write(frame)
```

**File Size:** ~10MB/minute @ QVGA

---

## 📹 Option 2: ESP32-CAM SD Card Recording

### Advantages
- **Autonomous:** Records even if WiFi disconnects
- **No PC Needed:** Robot is independent
- **Guaranteed Recording:** No network issues
- **Black Box:** Like airplane recorder

### Disadvantages
- **Limited Storage:** 32GB SD = ~50 hours @ QVGA
- **SD Card Wear:** Limited write cycles (~10,000)
- **Extra Weight:** +5g (SD card)
- **Power Consumption:** +50mA when writing
- **Manual Retrieval:** Must remove SD to view files
- **File Management:** Need to delete old files manually

### Implementation
```cpp
// In esp32_camera/main.cpp
#include "SD_MMC.h"

File videoFile;
unsigned long frameCount = 0;

void startRecording() {
    String filename = "/video_" + String(millis()) + ".mjpeg";
    videoFile = SD_MMC.open(filename, FILE_WRITE);
}

void loop() {
    camera_fb_t *fb = esp_camera_fb_get();
    if (videoFile && fb) {
        videoFile.write(fb->buf, fb->len);
        frameCount++;
    }
    esp_camera_fb_return(fb);
}
```

**File Size:** ~15MB/minute @ QVGA (MJPEG)

---

## 🎯 Recommendation: **PC Recording**

### Reasons
1. **Weight Savings:** No SD card (-5g)
2. **Cost Savings:** No need to buy SD card
3. **Convenience:** Files already on PC
4. **Flexibility:** Can switch recording on/off from dashboard
5. **Better Quality:** PC can handle higher resolution

### When to Use SD Card
- **Long Missions:** >1 hour without PC
- **Outdoor:** Far from WiFi range
- **Autonomous Mode:** Robot navigates alone
- **Data Logging:** Need guaranteed recording

---

## 💡 Hybrid Solution (Best of Both)

**Use PC recording by default, SD card as backup:**

```cpp
// ESP32-CAM: Dual mode
bool pc_connected = false;
bool sd_recording = false;

void loop() {
    camera_fb_t *fb = esp_camera_fb_get();
    
    // Always stream to PC if connected
    if (pc_connected) {
        sendToPC(fb);
    }
    
    // Record to SD only if PC disconnected
    if (!pc_connected && sd_recording) {
        writeToSD(fb);
    }
    
    esp_camera_fb_return(fb);
}
```

**Benefits:**
- Normal: PC records (light, convenient)
- Emergency: SD takes over (autonomous backup)

---

## 📊 Storage Comparison

| Duration | QVGA PC (10MB/min) | QVGA SD (15MB/min) |
|:---------|:-------------------|:-------------------|
| 10 min | 100 MB | 150 MB |
| 1 hour | 600 MB | 900 MB |
| 3 hours | 1.8 GB | 2.7 GB |

**32GB SD Card:** ~35 hours of recording  
**PC (1TB):** ~27,000 hours of recording

---

## ✅ Final Recommendation

**For Ghost Micro v4.0:**
- **Primary:** PC Recording (Dashboard)
- **Optional:** Add SD card later if needed for autonomous missions

**Saves:**
- Weight: 5g
- Cost: ~100 THB
- Power: 50mA
- Complexity: No file management

**พี่ไม่ต้องใส่ SD Card ก็ได้ครับ ให้คอมบันทึกสะดวกกว่า!** 💾✨
