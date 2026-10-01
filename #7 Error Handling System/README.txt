Request
   ↓
Application
   ↓
เกิด Error ?
   │
   ├── ❌ Validation Error
   ├── 🔐 Authentication Error
   ├── 🚫 Authorization Error
   ├── 💼 Business Error
   ├── 🗄️ Database Error
   ├── 🌐 Client Error
   └── 💥 Server Error
            ↓
      Global Error Handler
            ↓
       Error Response
            +
       Error Logging
🧩 ประเภทของ Error
👤 1. Client Error (ไคลเอนต์ เออเรอร์)

เกิดจาก Request (รีเควสต์) ที่ Client (ไคลเอนต์) ส่งมาไม่ถูกต้อง หรือไม่สามารถดำเนินการตาม Request นั้นได้

ตัวอย่าง:

ส่ง ID ไม่ถูกต้อง
ส่ง Parameter (พารามิเตอร์) ไม่ครบ
เรียก Resource (รีซอร์ส) ที่ไม่มี
ไม่มีสิทธิ์เข้าถึง

มักเกี่ยวข้องกับ HTTP Status (เอชทีทีพี สเตตัส):

400
401
403
404
409
422
429
✅ 2. Validation Error (วาลิเดชัน เออเรอร์)

ข้อมูลที่ส่งมาไม่ผ่านกฎของระบบ

{
  "email": "abc",
  "age": -5
}

ระบบอาจตอบ:

{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "ข้อมูลไม่ถูกต้อง",
    "fields": {
      "email": "รูปแบบ Email ไม่ถูกต้อง",
      "age": "อายุต้องมากกว่า 0"
    }
  }
}

โดยทั่วไปใช้ 400 หรือ 422 ตามมาตรฐานที่ระบบกำหนด

🔐 3. Authentication Error (ออเธนทิเคชัน เออเรอร์)

ระบบไม่สามารถยืนยันตัวตนได้

เช่น:

ไม่มี Token (โทเคน)
Token หมดอายุ
Token ไม่ถูกต้อง
Session หมดอายุ

โดยทั่วไป:

401 Unauthorized
🚫 4. Authorization Error (ออธอไรเซชัน เออเรอร์)

ยืนยันตัวตนได้แล้ว แต่ ไม่มีสิทธิ์ทำสิ่งนั้น

User
 ↓
Login สำเร็จ
 ↓
ต้องการ DELETE Employee
 ↓
ไม่มี Permission
 ↓
403 Forbidden

จำง่าย ๆ:

Authentication
= คุณคือใคร?

Authorization
= คุณมีสิทธิ์ทำอะไร?
💼 5. Business Error (บิสซิเนส เออเรอร์)

Request ถูกต้องตามรูปแบบ แต่ ผิดกฎทางธุรกิจ

ตัวอย่าง:

สินค้าเหลือ 0
 ↓
User สั่งซื้อ
 ↓
❌ OUT_OF_STOCK

หรือ:

Order ถูกปิดแล้ว
 ↓
User พยายามแก้ไข
 ↓
❌ ORDER_ALREADY_COMPLETED

นี่สำคัญมาก เพราะไม่ควรเอา Business Error ไปปนกับ Server Error

ตัวอย่าง:

{
  "success": false,
  "error": {
    "code": "OUT_OF_STOCK",
    "message": "สินค้าไม่เพียงพอ"
  }
}
🗄️ 6. Database Error (ดาต้าเบส เออเรอร์)

เกิดจาก Database (ดาต้าเบส) หรือการสื่อสารกับ Database

เช่น:

Connection Timeout
Deadlock
Constraint Violation
Database Unavailable
Query Timeout
Connection Pool เต็ม

⚠️ ไม่ควรส่ง Error ของ Database ออกไปให้ Client ตรง ๆ

ไม่ควร:

{
  "error": "SQL Error: Cannot insert duplicate key..."
}

เพราะอาจเปิดเผย:

Database Structure
Table Name
Column Name
SQL Query
Internal Information

ควร:

{
  "success": false,
  "error": {
    "code": "DATABASE_ERROR",
    "message": "เกิดข้อผิดพลาดในการประมวลผล"
  }
}

แล้วเก็บรายละเอียดไว้ใน Server Log (เซิร์ฟเวอร์ ล็อก)

💥 7. Server Error (เซิร์ฟเวอร์ เออเรอร์)

ปัญหาที่ระบบไม่สามารถจัดการได้ตามปกติ

เช่น:

Unhandled Exception
Memory Problem
Unexpected Error
Internal Service Failure

มักใช้:

500 Internal Server Error

Client ควรได้รับข้อมูลเท่าที่จำเป็น ไม่ควรเห็น Stack Trace (สแต็ก เทรซ)

🌐 Global Error Handler

Global Error Handler (โกลบอล เออเรอร์ แฮนดเลอร์) คือจุดกลางที่รับ Error จากระบบ

แทนที่จะเขียน:

try/catch
try/catch
try/catch
try/catch

แล้วแต่ละจุดตอบ Error ไม่เหมือนกัน

ให้มี:

Controller
    ↓
Service
    ↓
Repository
    ↓
❌ Error
    ↓
Global Error Handler
    ↓
Standard Error Response

ข้อดี:

Response เป็นมาตรฐานเดียวกัน
ลด Code ซ้ำ
Logging ทำจากจุดกลางได้
ซ่อนรายละเอียดภายใน
แปลง Error → HTTP Status ได้ง่าย
🏷️ Error Code

ควรมี Error Code (เออเรอร์ โค้ด) ที่เป็นค่าคงที่

เช่น:

VALIDATION_ERROR
INVALID_EMAIL
USER_NOT_FOUND
DUPLICATE_USER
OUT_OF_STOCK
ORDER_ALREADY_COMPLETED
DATABASE_ERROR
INTERNAL_ERROR

ข้อดีคือ Frontend ไม่ต้องเอา message มาตีความ

❌ ไม่ควร:

if message == "ไม่พบผู้ใช้งาน"

✅ ควร:

if error.code == "USER_NOT_FOUND"

เพราะ Message (เมสเสจ) สามารถเปลี่ยนภาษาได้ แต่ Code ควรคงที่

💬 Error Message

Message ควรเหมาะกับผู้รับ

Client
ไม่พบข้อมูลผู้ใช้งาน
Developer Log
User lookup failed:
userId=1024
repository=UserRepository
database=SQL Server
duration=5230ms

ดังนั้น:

Client Message
≠
Internal Error Detail
📝 Error Logging

เมื่อเกิด Error ต้องบันทึกข้อมูลที่ช่วยตรวจสอบปัญหา

เช่น:

Time: 2026-09-30 08:10:20
Level: ERROR
RequestId: req-12345
UserId: 1024
Method: POST
Endpoint: /api/orders
ErrorCode: DATABASE_TIMEOUT
Duration: 5002ms

แต่ต้องระวังไม่ Log (ล็อก) ข้อมูลลับ เช่น:

❌ Password
❌ Access Token
❌ Refresh Token
❌ API Key
❌ Secret
🔄 Retry

Retry (รีทราย) = ลอง Request หรือ Operation (โอเปอเรชัน) ใหม่เมื่อเกิด Temporary Error (เทมโพรารี เออเรอร์)

เช่น:

API A
 ↓
Database
 ↓
Timeout
 ↓
Retry
 ↓
สำเร็จ ✅

แต่ ไม่ใช่ทุก Error ที่ควร Retry

Retry ได้ในบางกรณี
Temporary Network Error
Temporary Service Unavailable
Transient Database Error
ไม่ควร Retry
Invalid Input
Authentication Failed
Authorization Failed
User Not Found
Business Rule Violation

และควรศึกษา:

Exponential Backoff
Jitter
Maximum Retry
Retry Budget
Idempotency

เพราะ Retry ที่ออกแบบไม่ดีสามารถทำให้ระบบหนักกว่าเดิมได้

Server ช้า
 ↓
Client Retry 100 ครั้ง
 ↓
Server หนักขึ้น
 ↓
ช้ากว่าเดิม
 ↓
Retry เพิ่ม
 ↓
💥 ระบบล่ม
⏱️ Timeout

Timeout (ไทม์เอาต์) = กำหนดเวลาสูงสุดที่ระบบจะรอ

ตัวอย่าง:

API Request
   ↓
Database Query
   ↓
รอ 5 วินาที
   ↓
ยังไม่ตอบ
   ↓
⏱️ Timeout

ควรมี Timeout หลายระดับ เช่น:

Client Timeout
      ↓
API Timeout
      ↓
Service Timeout
      ↓
Database Timeout
      ↓
External API Timeout

และควรออกแบบให้ Timeout ของระบบด้านใน ไม่ยาวกว่าระบบด้านนอกแบบไร้เหตุผล

🔥 สิ่งที่ควรเพิ่ม

จากรายการของคุณ ถ้าจะทำ Error Handling System สำหรับ Production ผมแนะนำเพิ่ม:

❌ ERROR HANDLING SYSTEM
│
├── Error Classification
│   ├── Client Error
│   ├── Validation Error
│   ├── Authentication Error
│   ├── Authorization Error
│   ├── Business Error
│   ├── Database Error
│   └── Server Error
│
├── Error Management
│   ├── Global Error Handler
│   ├── Error Code
│   ├── Error Message
│   ├── Error Logging
│   └── Exception Handling
│
├── Reliability
│   ├── Retry
│   ├── Timeout
│   ├── Exponential Backoff
│   ├── Jitter
│   ├── Circuit Breaker
│   └── Fallback
│
└── Debugging
    ├── Request ID
    ├── Correlation ID
    ├── Stack Trace
    └── Error Monitoring

🧠 ภาพรวมที่ควรจำ
                 ❌ ERROR
                     │
          ┌──────────┴──────────┐
          ↓                                 ↓
      คาดการณ์ได้                           คาดการณ์ไม่ได้
          │                                    │
          ↓                                     ↓
  Business / Validation                  Server Error
   Auth / Authorization
          │                                      │
          └──────────┬──────────┘
                              ↓
                         Global Handler
                           ↓
             ┌───────┴───────┐
             ↓                           ↓
       Client Response             Error Log
             ↓                          ↓
          Error Code              Request ID
          Message                Stack Trace
                                 Context

แก่นของระบบนี้คือ:

Client ต้องได้รับ Error ที่เข้าใจได้ ส่วน Developer ต้องได้รับข้อมูลที่เพียงพอในการหาสาเหตุ และ Error หนึ่งตัวต้องไม่ทำให้ทั้งระบบล้มตามกัน