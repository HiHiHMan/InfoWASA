📝 6. Logging & Audit System
ระบบบันทึกเหตุการณ์และตรวจสอบย้อนหลัง
📝 LOGGING & AUDIT SYSTEM
│
├── 📋 Logging
│   ├── INFO
│   ├── WARN
│   ├── ERROR
│   └── DEBUG
│
└── 🔍 Audit Log
    ├── Who
    ├── What
    ├── Which Data
    ├── When
    ├── From Where
    └── Result
📋 Logging (ล็อกกิง)

Logging คือการบันทึกว่า ระบบกำลังเกิดอะไรขึ้น

ใช้สำหรับ:

🐛 Debug (ดีบัก)
🔎 ตรวจสอบปัญหา
📊 ดูพฤติกรรมของระบบ
🚨 ตรวจจับ Error (เออเรอร์)
🔧 วิเคราะห์ Performance (เพอร์ฟอร์แมนซ์)
🔄 ติดตาม Request (รีเควสต์)
ระดับ Log (ล็อก)
DEBUG
↓
รายละเอียดสำหรับ Developer (ดีเวลอปเปอร์)

INFO
↓
เหตุการณ์ปกติของระบบ

WARN
↓
สิ่งผิดปกติที่ยังไม่ถึงขั้นระบบพัง

ERROR
↓
เกิดข้อผิดพลาด

FATAL / CRITICAL
↓
ปัญหาร้ายแรงที่กระทบระบบ

ตัวอย่าง:

INFO
User login successfully

INFO
GET /api/orders

WARN
Database connection pool is almost full

ERROR
Database query timeout

ERROR
Payment service unavailable
🧩 Log ที่ดีควรมีอะไรบ้าง?

ไม่ควรมีแค่:

Database Error

ควรมี Context (คอนเท็กซ์) ที่ช่วยหาปัญหาได้

Time: 2026-09-30 07:30:21
Level: ERROR
Service: Order API
RequestId: req-8a92
UserId: 1024
Method: POST
Endpoint: /api/orders
Error: Database timeout
Duration: 5230ms

สิ่งสำคัญมากคือ Request ID / Correlation ID (รีเควสต์ ไอดี / คอร์เรเลชัน ไอดี)

เพราะ Request หนึ่งอาจวิ่งผ่านหลายระบบ:

Client
  ↓
API Gateway
  ↓
Order API
  ↓
Payment Service
  ↓
Database

ถ้าทุกระบบใช้ ID เดียวกัน:

RequestId = ABC123

เราสามารถค้นหา Log (ล็อก) ของ ABC123 แล้วตามเหตุการณ์ได้ทั้ง Chain (เชน)

🔍 Audit Log (ออดิท ล็อก)

Audit Log ไม่ได้เน้นว่า "ระบบทำงานอย่างไร"

แต่เน้นว่า:

"ใครทำอะไรกับข้อมูลอะไร และเกิดอะไรขึ้น"

หลักสำคัญ:

👤 WHO ใคร?

⬇️

🎯 WHAT ทำอะไร?

⬇️

🗄️ WHICH DATA ข้อมูลอะไร?

⬇️

⏰ WHEN เมื่อไหร่?

⬇️

🌐 WHERE มาจากไหน?

⬇️

✅ RESULT ผลลัพธ์เป็นอย่างไร?

ตัวอย่าง:

User: ART
Action: UPDATE
Table: Employee
Record: 1024

Old:
Department = IT

New:
Department = HR

Time:
2026-09-30 07:30

IP:
192.168.1.50

Result:
SUCCESS
🆚 Logging vs Audit Log

จุดนี้สำคัญมาก

				📋 Logging	🔍 Audit Log
จุดประสงค์							แก้ปัญหาระบบ	         ตรวจสอบการกระทำ
ใคร Login			✅		✅
API Error			✅		❌/ไม่จำเป็น
Database Timeout		✅		❌
User เปลี่ยนข้อมูล					อาจมี				✅
ใครแก้ข้อมูล						อาจมี				✅
Old → New			ไม่จำเป็น		✅
Security Investigation		ช่วยได้		✅
Compliance (คอมพลายแอนซ์)	อาจเกี่ยวข้อง	✅

พูดง่าย ๆ:
Logging
= "ระบบเกิดอะไรขึ้น?"

Audit Log
= "ใครทำอะไรกับระบบ?"
🗄️ Audit Log สำหรับ Database

ถ้าระบบองค์กรมีข้อมูลสำคัญ ควรสามารถตรวจสอบย้อนหลังได้ เช่น

Employee
Order
Customer
Payment
Stock
Asset
Permission
Configuration

ตัวอย่าง:

┌─────────────────────────────────────────┐
│ AuditLog                                		    │
├─────────────────────────────────────────┤
│ Id                                      		    │
│ UserId                                  		    │
│ Action                                  		    │
│ Entity                                  		    │
│ EntityId                                		    │
│ OldValue                                		    │
│ NewValue                                		    │
│ IPAddress                               		    │
│ UserAgent                               		    │
│ RequestId                               		    │
│ CreatedAt                               		    │
│ Result                                  		    │
└─────────────────────────────────────────┘

อาจเก็บข้อมูลเป็น JSON (เจสัน) เช่น:

{
  "Department": {
    "old": "IT",
    "new": "HR"
  }
}
🔐 Audit Log ต้องระวังเรื่อง Security

Audit Log เองก็เป็นข้อมูลสำคัญ

จึงไม่ควรให้ User ทั่วไปสามารถ:

❌ UPDATE Audit Log
❌ DELETE Audit Log
❌ แก้ Old Value
❌ แก้ UserId
❌ แก้ Timestamp

ควรออกแบบให้:

Application
     ↓
Audit Service
     ↓
Audit Storage
     ↓
🔒 Restricted Access

และต้องระวัง Sensitive Data (เซนซิทีฟ ดาต้า) เช่น:

❌ Password
❌ Access Token
❌ Refresh Token
❌ API Key
❌ Secret
❌ ข้อมูลส่วนตัวที่ไม่จำเป็น

ไม่ควรโยนทุกอย่างลง Log เพราะ Log อาจถูกเก็บไว้นานและมีผู้ดูแลหลายระดับเข้าถึงได้

🚨 Logging + Monitoring

Logging ไม่ควรอยู่เดี่ยว ๆ

ควรเชื่อมกับ Monitoring (มอนิเทอริง)

เช่น:

Application
    ↓
Logging
    ↓
Log Storage
    ↓
Monitoring
    ↓
Alert

ตัวอย่าง:

ERROR เพิ่มขึ้นผิดปกติ
        ↓
Monitoring ตรวจพบ
        ↓
🚨 Alert
        ↓
Developer / Admin
🔥 สิ่งที่ควรเพิ่มใน Logging System

จากรายการเดิม ผมแนะนำให้เพิ่ม:

📝 LOGGING SYSTEM
│
├── Log Level
│   ├── DEBUG
│   ├── INFO
│   ├── WARN
│   ├── ERROR
│   └── CRITICAL
│
├── Context
│   ├── Timestamp
│   ├── Request ID
│   ├── User ID
│   ├── Service
│   ├── Endpoint
│   └── Duration
│
├── Structured Logging
├── Log Rotation
├── Log Retention
├── Centralized Logging
├── Sensitive Data Masking
└── Log Monitoring

และฝั่ง Audit:

🔍 AUDIT LOG
│
├── Who
├── Action
├── Entity
├── Entity ID
├── Old Value
├── New Value
├── Timestamp
├── IP Address
├── User Agent
├── Request ID
├── Result
│
├── Audit Retention
├── Audit Security
└── Audit Access Control
🧠 จำง่าย ๆ
📋 LOGGING
"ระบบเกิดอะไรขึ้น?"

       +

🔍 AUDIT
"ใครทำอะไรกับข้อมูล?"

       +

📊 MONITORING
"ตอนนี้ระบบเป็นอย่างไร?"

       +

🚨 ALERT
"มีอะไรผิดปกติหรือไม่?"

ทั้ง 4 ตัวนี้เมื่อเอามารวมกัน จะกลายเป็นพื้นฐานของ Observability (ออบเซอร์เวบิลิตี) ที่ดีของระบบ Production (โปรดักชัน) โดยเฉพาะระบบองค์กรที่ต้องสามารถ ตรวจสอบย้อนหลัง + หา Root Cause (รูต คอส) + ตรวจเหตุการณ์ผิดปกติ + ตรวจสอบการเปลี่ยนแปลงข้อมูล ได้ครับ