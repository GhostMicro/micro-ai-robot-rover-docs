# Next.js Dashboard Development Guide

## ภาพรวม (Overview)

Dashboard นี้ทำหน้าที่เป็นศูนย์ควบคุมหลัก (Command Center) สำหรับหุ่นยนต์กู้ภัย โดยแสดงข้อมูลแบบ Real-time จากเซนเซอร์ทั้งหมด พร้อมระบบควบคุมแบบ Manual Override

---

## 1. ระบบรับข้อมูล (Data Acquisition)

### API Routes
สร้าง Endpoint สำหรับรับข้อมูลจาก ESP32

**ไฟล์:** `src/app/api/rover/telemetry/route.ts`
```typescript
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const data = await request.json();
  
  // บันทึกข้อมูลลง Database หรือ In-Memory Store
  // เช่น Redis, SQLite, หรือ Global State
  
  console.log('Telemetry received:', data);
  
  return NextResponse.json({ status: 'ok' });
}
```

### WebSocket Setup
ใช้ **Socket.io** สำหรับการส่งข้อมูลแบบ Real-time

**ติดตั้ง Dependencies:**
```bash
npm install socket.io socket.io-client
```

**ไฟล์:** `src/lib/socket-server.ts`
```typescript
import { Server } from 'socket.io';

export function initSocketServer(httpServer: any) {
  const io = new Server(httpServer, {
    cors: {
      origin: '*',
    },
  });

  io.on('connection', (socket) => {
    console.log('Client connected:', socket.id);
    
    socket.on('disconnect', () => {
      console.log('Client disconnected:', socket.id);
    });
  });

  return io;
}
```

### Video Streaming Component
แสดงผล MJPEG Stream จาก ESP32-CAM

**ไฟล์:** `src/components/VideoFeed.tsx`
```typescript
'use client';

import { useState, useEffect } from 'react';

interface VideoFeedProps {
  cameraIP: string;
}

export default function VideoFeed({ cameraIP }: VideoFeedProps) {
  const [isOnline, setIsOnline] = useState(false);

  useEffect(() => {
    // ตรวจสอบว่ากล้องออนไลน์หรือไม่
    const checkCamera = async () => {
      try {
        const response = await fetch(`http://${cameraIP}/status`);
        setIsOnline(response.ok);
      } catch {
        setIsOnline(false);
      }
    };

    checkCamera();
    const interval = setInterval(checkCamera, 5000);
    return () => clearInterval(interval);
  }, [cameraIP]);

  return (
    <div className="relative w-full aspect-video bg-gray-950 rounded-lg overflow-hidden border-2 border-orange-500/30">
      {isOnline ? (
        <img 
          src={`http://${cameraIP}/stream`}
          alt="Rover Camera Feed"
          className="w-full h-full object-cover"
        />
      ) : (
        <div className="flex items-center justify-center h-full">
          <p className="text-gray-500">Camera Offline</p>
        </div>
      )}
      <div className="absolute top-2 right-2">
        <div className={`w-3 h-3 rounded-full ${isOnline ? 'bg-green-500' : 'bg-red-500'} animate-pulse`} />
      </div>
    </div>
  );
}
```

---

## 2. หน้าจอแสดงผล (UI Components)

### Sensor Dashboard
แสดงข้อมูลจากเซนเซอร์ทั้งหมด

**ไฟล์:** `src/components/SensorDashboard.tsx`
```typescript
'use client';

import { useEffect, useState } from 'react';

interface TelemetryData {
  timestamp: string;
  dist: {
    front: number;
    back: number;
    left: number;
    right: number;
  };
  imu: {
    pitch: number;
    roll: number;
  };
  batt: number;
  mode: string;
}

export default function SensorDashboard() {
  const [data, setData] = useState<TelemetryData | null>(null);

  useEffect(() => {
    // เชื่อมต่อ WebSocket
    const ws = new WebSocket('ws://localhost:3001');
    
    ws.onmessage = (event) => {
      const telemetry = JSON.parse(event.data);
      setData(telemetry);
    };

    return () => ws.close();
  }, []);

  if (!data) return <div>Waiting for data...</div>;

  return (
    <div className="grid grid-cols-2 gap-4">
      {/* Distance Grid */}
      <div className="bg-gray-900 p-4 rounded-lg border border-orange-500/30">
        <h3 className="text-orange-500 font-bold mb-2">Distance Sensors</h3>
        <div className="grid grid-cols-3 gap-2 text-center">
          <div></div>
          <div className="bg-gray-800 p-2 rounded">
            <div className="text-xs text-gray-400">Front</div>
            <div className="text-lg font-bold text-pink-500">{data.dist.front}cm</div>
          </div>
          <div></div>
          
          <div className="bg-gray-800 p-2 rounded">
            <div className="text-xs text-gray-400">Left</div>
            <div className="text-lg font-bold text-pink-500">{data.dist.left}cm</div>
          </div>
          <div className="bg-gray-800 p-2 rounded flex items-center justify-center">
            <div className="text-2xl">🤖</div>
          </div>
          <div className="bg-gray-800 p-2 rounded">
            <div className="text-xs text-gray-400">Right</div>
            <div className="text-lg font-bold text-pink-500">{data.dist.right}cm</div>
          </div>
          
          <div></div>
          <div className="bg-gray-800 p-2 rounded">
            <div className="text-xs text-gray-400">Back</div>
            <div className="text-lg font-bold text-pink-500">{data.dist.back}cm</div>
          </div>
          <div></div>
        </div>
      </div>

      {/* IMU & Status */}
      <div className="bg-gray-900 p-4 rounded-lg border border-orange-500/30">
        <h3 className="text-orange-500 font-bold mb-2">System Status</h3>
        <div className="space-y-2">
          <div className="flex justify-between">
            <span className="text-gray-400">Pitch:</span>
            <span className="text-pink-500 font-mono">{data.imu.pitch.toFixed(1)}°</span>
          </div>
          <div className="flex justify-between">
            <span className="text-gray-400">Roll:</span>
            <span className="text-pink-500 font-mono">{data.imu.roll.toFixed(1)}°</span>
          </div>
          <div className="flex justify-between">
            <span className="text-gray-400">Battery:</span>
            <span className="text-green-500 font-mono">{data.batt.toFixed(1)}V</span>
          </div>
          <div className="flex justify-between">
            <span className="text-gray-400">Mode:</span>
            <span className="text-orange-500 font-bold">{data.mode}</span>
          </div>
          <div className="flex justify-between">
            <span className="text-gray-400">Time:</span>
            <span className="text-gray-300 font-mono text-sm">{data.timestamp}</span>
          </div>
        </div>
      </div>
    </div>
  );
}
```

---

## 3. ระบบควบคุม (Control Interface)

### Control Panel
ปุ่มควบคุมหุ่นยนต์

**ไฟล์:** `src/components/ControlPanel.tsx`
```typescript
'use client';

import { useState } from 'react';
import { ArrowUp, ArrowDown, ArrowLeft, ArrowRight, Camera, Zap } from 'lucide-react';

export default function ControlPanel() {
  const [mode, setMode] = useState<'AUTO' | 'MANUAL'>('AUTO');

  const sendCommand = async (command: string) => {
    try {
      await fetch('/api/rover/command', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ command }),
      });
    } catch (error) {
      console.error('Failed to send command:', error);
    }
  };

  return (
    <div className="bg-gray-900 p-6 rounded-lg border border-orange-500/30">
      <h3 className="text-orange-500 font-bold mb-4">Control Panel</h3>
      
      {/* Mode Toggle */}
      <div className="flex gap-2 mb-4">
        <button
          onClick={() => {
            setMode('AUTO');
            sendCommand('SET_MODE:AUTO');
          }}
          className={`flex-1 py-2 rounded ${
            mode === 'AUTO' 
              ? 'bg-orange-500 text-white' 
              : 'bg-gray-800 text-gray-400'
          }`}
        >
          Auto Pilot
        </button>
        <button
          onClick={() => {
            setMode('MANUAL');
            sendCommand('SET_MODE:MANUAL');
          }}
          className={`flex-1 py-2 rounded ${
            mode === 'MANUAL' 
              ? 'bg-pink-500 text-white' 
              : 'bg-gray-800 text-gray-400'
          }`}
        >
          Manual
        </button>
      </div>

      {/* D-Pad Controls */}
      {mode === 'MANUAL' && (
        <div className="grid grid-cols-3 gap-2 mb-4">
          <div></div>
          <button
            onClick={() => sendCommand('MOVE:FWD:80')}
            className="bg-gray-800 hover:bg-orange-500 p-4 rounded transition-colors"
          >
            <ArrowUp className="mx-auto" />
          </button>
          <div></div>
          
          <button
            onClick={() => sendCommand('MOVE:LEFT:80')}
            className="bg-gray-800 hover:bg-orange-500 p-4 rounded transition-colors"
          >
            <ArrowLeft className="mx-auto" />
          </button>
          <button
            onClick={() => sendCommand('HALT')}
            className="bg-red-600 hover:bg-red-700 p-4 rounded transition-colors font-bold"
          >
            STOP
          </button>
          <button
            onClick={() => sendCommand('MOVE:RIGHT:80')}
            className="bg-gray-800 hover:bg-orange-500 p-4 rounded transition-colors"
          >
            <ArrowRight className="mx-auto" />
          </button>
          
          <div></div>
          <button
            onClick={() => sendCommand('MOVE:BACK:80')}
            className="bg-gray-800 hover:bg-orange-500 p-4 rounded transition-colors"
          >
            <ArrowDown className="mx-auto" />
          </button>
          <div></div>
        </div>
      )}

      {/* Action Buttons */}
      <div className="grid grid-cols-2 gap-2">
        <button
          onClick={() => sendCommand('CAM:CAPTURE')}
          className="bg-gray-800 hover:bg-pink-500 py-3 rounded transition-colors flex items-center justify-center gap-2"
        >
          <Camera size={20} />
          Take Photo
        </button>
        <button
          onClick={() => sendCommand('EMERGENCY')}
          className="bg-gray-800 hover:bg-yellow-500 py-3 rounded transition-colors flex items-center justify-center gap-2"
        >
          <Zap size={20} />
          Emergency
        </button>
      </div>
    </div>
  );
}
```

---

## 4. ข้อมูลทางเทคนิคสำหรับเชื่อมต่อ

### Telemetry Data Format (JSON)
ESP32 จะส่งข้อมูลมาในรูปแบบนี้:

```json
{
  "timestamp": "2024-05-20 14:30:05",
  "dist": {
    "front": 25,
    "back": 80,
    "left": 40,
    "right": 42
  },
  "imu": {
    "pitch": 2.5,
    "roll": -1.2
  },
  "batt": 12.4,
  "mode": "MANUAL"
}
```

### Command Format
Dashboard ส่งคำสั่งไปยัง ESP32 ในรูปแบบ:

```json
{
  "command": "MOVE:FWD:80"
}
```

**คำสั่งที่รองรับ:**
- `MOVE:FWD:80` - เดินหน้าความเร็ว 80%
- `MOVE:BACK:80` - ถอยหลัง
- `MOVE:LEFT:80` - เลี้ยวซ้าย
- `MOVE:RIGHT:80` - เลี้ยวขวา
- `HALT` - หยุดฉุกเฉิน
- `SET_MODE:AUTO` - เปลี่ยนเป็นโหมดอัตโนมัติ
- `SET_MODE:MANUAL` - เปลี่ยนเป็นโหมดควบคุมมือ
- `CAM:CAPTURE` - สั่งถ่ายภาพ (ส่ง 'C' ไปยัง ESP32-CAM)

---

## 5. การติดตั้งและรัน Dashboard

```bash
# ติดตั้ง Dependencies
npm install socket.io socket.io-client lucide-react

# รัน Development Server
npm run dev
```

**เปิดดูที่:** `http://localhost:3000/dashboard`

---

## 6. โครงสร้างโปรเจกต์ที่แนะนำ

```
micro-ai-robot-rover/
├── src/
│   ├── app/
│   │   ├── api/
│   │   │   └── rover/
│   │   │       ├── command/route.ts
│   │   │       └── telemetry/route.ts
│   │   ├── dashboard/
│   │   │   └── page.tsx
│   ├── components/
│   │   ├── VideoFeed.tsx
│   │   ├── SensorDashboard.tsx
│   │   ├── ControlPanel.tsx
│   │   └── LogViewer.tsx
│   ├── lib/
│   │   └── socket-server.ts
```
