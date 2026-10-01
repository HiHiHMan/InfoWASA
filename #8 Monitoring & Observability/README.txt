Monitoring (มอนิเทอริง)

คือการติดตามสถานะและ Metrics (เมตริกส์) ของระบบอย่างต่อเนื่อง

ตัวอย่าง:

CPU สูงไหม?
Memory เต็มไหม?
API ช้าไหม?
Error เพิ่มไหม?
Database ช้าไหม?
Connection Pool เต็มไหม?
Disk ใกล้เต็มไหม?

ตัวอย่าง Metric:

CPU Usage          72%
Memory Usage       81%
API Response       230ms
Error Rate         1.2%
Requests/sec       850
DB Connections     18/20



Observability (ออบเซอร์เวบิลิตี)
Observability คือความสามารถในการใช้ข้อมูลจากระบบเพื่อทำความเข้าใจ สถานะภายในของระบบ

โดยทั่วไปจะพูดถึง 3 เสาหลัก:

             🔍 OBSERVABILITY
                    │
       ┌────────────┼────────────┐
       ↓                 ↓                 ↓
    📋 Logs        📊 Metrics       🔗 Traces
      ล็อก              เมตริกส์             เทรซ

1️⃣ Logs (ล็อก)

บอกว่า เกิดเหตุการณ์อะไร

ERROR
Database connection timeout

เชื่อมกับระบบ Logging & Audit ที่เราคุยกันก่อนหน้านี้

2️⃣ Metrics (เมตริกส์)

บอกว่า ระบบมีพฤติกรรมอย่างไรในภาพรวม

เช่น:

Requests / sec
Error Rate
Latency
CPU
Memory
Database Connections
Queue Length

ตัวอย่าง:

10:00 → 500 req/s
10:05 → 550 req/s
10:10 → 900 req/s
10:15 → 1,800 req/s

ถ้า Error Rate เพิ่มขึ้นพร้อมกัน ก็เป็นสัญญาณว่าระบบอาจกำลังรับโหลดสูงผิดปกติ

🔗 3. Distributed Tracing (ดิสทริบิวเต็ด เทรซิง)

อันนี้สำคัญมากเมื่อระบบมีหลาย Service (เซอร์วิส)

สมมติ Request หนึ่งตัว:

User
 ↓
API Gateway
 ↓ 50ms
Order API
 ↓ 100ms
Payment API
 ↓ 800ms
Database
 ↓ 50ms
Response

Tracing ช่วยให้เห็นว่า:

Total = 1000ms

Gateway     50ms
Order API   100ms
Payment     800ms  ← 🔥 ช้าตรงนี้
Database    50ms

ทำให้ไม่ต้องเดาว่า API ช้าตรงไหน

⏱️ Latency (เลเทนซี)

คือเวลาที่ระบบใช้ในการตอบสนอง

เช่น:

GET /api/products

Response Time = 120ms

แต่ไม่ควรดูแค่ Average (แอเวอเรจ)

ควรเข้าใจ:

P50
P90
P95
P99

ตัวอย่าง:

P50 = 100ms
P95 = 300ms
P99 = 2,000ms

แปลว่า User ส่วนใหญ่ไม่ได้ช้ามาก แต่บาง Request ช้ามาก

โดยเฉพาะ P95 / P99 สำคัญสำหรับระบบ Production

🚨 Error Rate

ดูว่า Request มี Error กี่เปอร์เซ็นต์

Total Request = 100,000

Error = 500

Error Rate = 0.5%

ควรแยกประเภทด้วย เช่น:

4xx
5xx
Database Error
Timeout
External API Error

เพราะ 4xx กับ 5xx มีความหมายต่างกัน

📈 Throughput (ทรูพุต)

คือปริมาณงานที่ระบบประมวลผลได้ในช่วงเวลาหนึ่ง

เช่น:

Requests/sec = 2,000

Orders/minute = 500

Messages/sec = 10,000

ใช้ดูว่า System Capacity (ซิสเต็ม แคพาซิตี) อยู่ตรงไหน

🖥️ Infrastructure Monitoring

นอกจาก API ต้องดู Server (เซิร์ฟเวอร์) ด้วย

🖥️ Server
├── CPU
├── Memory
├── Disk
├── Disk I/O
├── Network
└── Process

ตัวอย่าง:

CPU       90% 🔴
Memory    85% 🟠
Disk      92% 🔴
Network   40%
🗄️ Database Monitoring

สำหรับ SQL Server (เอสคิวแอล เซิร์ฟเวอร์) ควร Monitor (มอนิเตอร์) อย่างน้อย:

🗄️ SQL SERVER
│
├── CPU
├── Memory
├── Disk I/O
├── Query Duration
├── Slow Query
├── Blocking
├── Deadlock
├── Lock
├── Connection
├── Connection Pool
├── Transaction
├── Wait Statistics
└── Database Size

โดยเฉพาะ:

🔒 Blocking
Transaction A
      ↓
ถือ Lock
      ↓
Transaction B
      ↓
รอ
      ↓
⏱️ Blocking
💥 Deadlock
Transaction A → รอ B
Transaction B → รอ A

💥 DEADLOCK

ควรสามารถตรวจพบเหตุการณ์เหล่านี้ได้ ไม่ใช่รอ User แจ้ง

🚦 Queue Monitoring

ถ้าระบบมี Queue (คิว) ต้อง Monitor:

Queue Length
Processing Rate
Failed Jobs
Retry Count
Oldest Message
Consumer Status

เช่น:

Queue
████████████████████ 20,000

ถ้า Queue เพิ่มขึ้นเรื่อย ๆ แต่ Worker (เวิร์กเกอร์) ประมวลผลไม่ทัน อาจหมายถึงระบบกำลังมี Bottleneck (บอตเทิลเน็ก)

🚨 Alerting (อเลิร์ติง)

Monitoring ที่ไม่มี Alert (อเลิร์ต) ก็อาจกลายเป็นแค่ Dashboard (แดชบอร์ด) สวย ๆ 😅

ควรกำหนดเงื่อนไข เช่น:

CPU > 90%
        ↓
🚨 Alert

Error Rate > 5%
        ↓
🚨 Alert

P95 > 2 sec
        ↓
🚨 Alert

Database Connection > 90%
        ↓
🚨 Alert

Disk > 90%
        ↓
🚨 Alert

แต่ต้องระวัง Alert Fatigue (อเลิร์ต แฟทีก)

ถ้าแจ้งเตือนทุกอย่าง:

🚨 Alert
🚨 Alert
🚨 Alert
🚨 Alert
🚨 Alert

สุดท้ายคนดูแลระบบจะเริ่มไม่สนใจ Alert

🎯 SLI / SLO / SLA

ถ้าจะเข้าใจ Monitoring ระดับ Production ควรเพิ่ม 3 เรื่องนี้

SLI (เอสแอลไอ)

ตัวชี้วัดที่ใช้วัดคุณภาพจริง

เช่น:

API Availability
API Latency
Error Rate
SLO (เอสแอลโอ)

เป้าหมายที่ระบบต้องการ

เช่น:

99.9% Availability
P95 Latency < 500ms
SLA (เอสแอลเอ)

ข้อตกลงระดับการให้บริการกับลูกค้า/ผู้ใช้

Availability ≥ 99.9%

โดย SLA อาจมีเงื่อนไขทางธุรกิจหรือการชดเชยตามสัญญา

🩺 Health Check

ระบบควรมี Endpoint (เอนด์พอยต์) สำหรับตรวจสถานะ

เช่น:

GET /health

ตอบ:

{
  "status": "healthy"
}

แต่ระบบจริงอาจแยก:

/health/live
/health/ready
Liveness (ไลฟ์เนส)

ถามว่า:

Application ยังทำงานอยู่ไหม?

Readiness (เรดดิเนส)

ถามว่า:

Application พร้อมรับ Traffic (ทราฟฟิก) หรือยัง?

เช่น Application เปิดได้ แต่ Database ยังเชื่อมไม่ได้:

Liveness  → ✅
Readiness → ❌
🔎 Incident Investigation

เมื่อระบบมีปัญหา ควรสามารถไล่แบบนี้:

🚨 Alert
   ↓
📊 Metrics
   ↓
🔗 Trace
   ↓
📋 Logs
   ↓
🗄️ Database
   ↓
🔍 Root Cause

ตัวอย่าง:

API P95 สูงขึ้น
      ↓
Trace พบ Payment API ช้า
      ↓
Log พบ Database Timeout
      ↓
SQL Server พบ Blocking
      ↓
พบ Transaction เปิดค้าง
      ↓
🎯 Root Cause

นี่คือประโยชน์สำคัญของ Observability

🧠 เพิ่มหัวข้อเหล่านี้ใน Checklist

ถ้าจะทำ Monitoring & Observability System ให้ครบสำหรับ Production ผมแนะนำ:

📊 MONITORING & OBSERVABILITY
│
├── 📈 Monitoring
│   ├── CPU
│   ├── Memory
│   ├── Disk
│   ├── Network
│   └── Process
│
├── 📊 Metrics
│   ├── Request Rate
│   ├── Error Rate
│   ├── Latency
│   ├── Throughput
│   └── Resource Usage
│
├── 📋 Logs
│   ├── Application Log
│   ├── Error Log
│   └── Audit Log
│
├── 🔗 Distributed Tracing
│   ├── Trace ID
│   ├── Span
│   └── Service Dependency
│
├── 🚨 Alerting
│   ├── Threshold
│   ├── Anomaly
│   ├── Alert Routing
│   └── Alert Fatigue
│
├── 🩺 Health Check
│   ├── Liveness
│   └── Readiness
│
├── 🎯 Reliability
│   ├── SLI
│   ├── SLO
│   └── SLA
│
└── 🔍 Troubleshooting
    ├── Root Cause Analysis
    ├── Incident Investigation
    └── Dependency Analysis
🔥 จำ 4 ตัวนี้ให้แม่น
📋 LOG
"เกิดอะไรขึ้น?"

📊 METRIC
"ระบบมีอาการอย่างไร?"

🔗 TRACE
"Request นี้วิ่งผ่านอะไรบ้าง และช้าตรงไหน?"

🚨 ALERT
"ตอนนี้มีอะไรที่ต้องเข้าไปจัดการ?"

และถ้าเอา Logging + Monitoring + Observability + Error Handling มารวมกัน ระบบจะเริ่มมีความสามารถในการ Detect → Investigate → Diagnose → Resolve ปัญหาได้อย่างเป็นระบบครับ