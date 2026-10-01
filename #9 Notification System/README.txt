Notification System 🔔

ระบบแจ้งเตือน

ระบบที่รับผิดชอบการส่งข้อความหรือแจ้งเตือนไปยังผู้ใช้ผ่านช่องทางต่าง ๆ เช่น Email (อีเมล), SMS (เอสเอ็มเอส), Push Notification (พุช นอทิฟิเคชัน), LINE, In-App Notification (อินแอป นอทิฟิเคชัน)

จุดสำคัญคือ อย่าส่ง Notification (นอทิฟิเคชัน) แบบผูกติดกับ Request หลักโดยตรง เพราะถ้าระบบส่งช้า ระบบหลักก็ช้าตาม

🔔 สิ่งที่ระบบ Notification ควรมี
Notification Type (ประเภทการแจ้งเตือน)
Notification Template (เทมเพลต)
Notification Channel (ช่องทาง)
Notification Queue (คิว)
Background Worker (แบ็กกราวด์ เวิร์กเกอร์)
Retry (รีไทร)
Delay / Scheduling (ดีเลย์ / การตั้งเวลา)
Priority (ไพรออริตี)
Delivery Status (สถานะการส่ง)
Idempotency (ไอเด็มโพเทนซี)
Deduplication (ดีดูพลิเคชัน)
Rate Limiting (เรต ลิมิตทิง)
User Preference (การตั้งค่าของผู้ใช้)
Failure Handling (การจัดการเมื่อส่งไม่สำเร็จ)
Notification History (ประวัติการแจ้งเตือน)
Monitoring (มอนิเทอริง)
1. Notification Type 🔔

กำหนดว่า Notification นี้มีไว้ทำอะไร

ตัวอย่าง:

ORDER_CREATED
ORDER_COMPLETED
PAYMENT_SUCCESS
PAYMENT_FAILED
PASSWORD_RESET
ACCOUNT_LOCKED
SYSTEM_MAINTENANCE

ไม่ควรเขียนข้อความกระจายอยู่ทั่ว Code (โค้ด)

❌

"สั่งซื้อสินค้าสำเร็จแล้ว"

กระจายอยู่หลายไฟล์

ควรมี Type กลาง:

ORDER_COMPLETED

แล้วให้ระบบเลือก Template (เทมเพลต) ที่เหมาะสม

2. Notification Channel 📢

ช่องทางที่ใช้ส่ง

Notification
│
├── In-App
├── Email
├── SMS
├── Push Notification
├── LINE
└── Webhook

ตัวอย่าง User (ยูสเซอร์) คนหนึ่งอาจตั้งไว้ว่า

ORDER_COMPLETED

├── In-App ✅
├── Email ✅
├── SMS ❌
└── Push ✅

ดังนั้น Notification Type กับ Channel ควรแยกจากกัน

3. Notification Template 📝

เก็บรูปแบบข้อความไว้เป็น Template

ตัวอย่าง:

Order {{OrderNo}} completed successfully.

ข้อมูล:

OrderNo = ORD-20260930-001

ผลลัพธ์:

Order ORD-20260930-001 completed successfully.

ข้อดีคือสามารถเปลี่ยนข้อความได้โดยไม่ต้องแก้ Business Logic (บิสซิเนส ลอจิก)

และสามารถมีหลายภาษาได้:

TH
EN
JP
4. Notification Queue 📦

นี่เป็นส่วนที่ สำคัญมากสำหรับระบบจริง

แทนที่จะทำแบบนี้:

API
 ↓
Create Order
 ↓
Send Email
 ↓
Send LINE
 ↓
Send Push
 ↓
Response

ถ้า Email ใช้เวลา 3 วินาที

API ก็ต้องรอ 3 วินาที

ควรเป็น:

API
 ↓
Create Order
 ↓
Create Notification Job
 ↓
Response 200
        ↓
      Queue
        ↓
      Worker
        ↓
 ┌──────┼──────┐
 ↓      ↓      ↓
Email  LINE   Push

ระบบหลักจึงไม่ต้องรอ Notification

5. Background Worker ⚙️

Worker (เวิร์กเกอร์) คือ Process (โพรเซส) ที่คอยหยิบงานจาก Queue มาทำ

ตัวอย่าง:

Queue

Job 001 → Email
Job 002 → Push
Job 003 → LINE
Job 004 → SMS

Worker:

Worker
   ↓
Get Job
   ↓
Process
   ↓
Success → Complete
   ↓
Fail → Retry

เหมาะมากกับงานที่:

ส่ง Email
ส่ง SMS
ส่ง Push
Generate Report (เจเนอเรต รีพอร์ต)
ส่งไฟล์
Webhook
งานที่ใช้เวลานาน
6. Retry 🔄

ระบบส่ง Notification ไม่สำเร็จ ไม่ควรล้มทันที

ตัวอย่าง:

Attempt 1 ❌
   ↓
Wait 1s
   ↓
Attempt 2 ❌
   ↓
Wait 5s
   ↓
Attempt 3 ❌
   ↓
Wait 30s
   ↓
Attempt 4 ❌
   ↓
Dead Letter Queue

ควรรู้เรื่อง:

Maximum Retry (จำนวนครั้งสูงสุด)
Exponential Backoff (เอ็กซ์โพเนนเชียล แบ็กออฟ)
Jitter (จิตเตอร์)
Retryable Error (เออเรอร์ที่ควร Retry)
Non-Retryable Error (เออเรอร์ที่ไม่ควร Retry)

เช่น

Network Timeout → Retry ✅
Provider 500 → Retry ✅
Invalid Email → Retry ❌
Invalid Phone → Retry ❌
Permission Error → Retry ❌
7. Idempotency 🔐

สำคัญมาก

สมมติระบบส่ง Email สำเร็จแล้ว แต่ตอนบันทึกสถานะเกิด Error

ระบบคิดว่า:

ส่งไม่สำเร็จ

แล้ว Retry

อาจเกิด:

📧 Email 1
📧 Email 2

ผู้ใช้ได้รับ Email ซ้ำ

จึงควรมี IdempotencyKey

เช่น

ORDER_COMPLETED:ORD-10001

ก่อนส่งตรวจสอบว่า Job นี้ถูกดำเนินการไปแล้วหรือยัง

8. Deduplication 🔄

ป้องกัน Notification ซ้ำ

เช่นระบบมี Bug:

Order Updated
Order Updated
Order Updated
Order Updated

อาจเกิด Notification 4 ครั้ง

ระบบอาจกำหนด:

UserId
+
NotificationType
+
EntityId
+
Time Window

เพื่อป้องกันการส่งซ้ำ

9. Delivery Status 📊

ควรเก็บสถานะการส่ง

PENDING
PROCESSING
SENT
FAILED
RETRYING
CANCELLED

ตัวอย่าง:

NotificationId: 10001
UserId: 500
Type: ORDER_COMPLETED
Channel: EMAIL
Status: SENT
Attempt: 2
SentAt: ...

ทำให้ตรวจสอบย้อนหลังได้ว่า

"ทำไม User คนนี้ไม่ได้รับ Email?"

10. User Preference ⚙️

ผู้ใช้ควรกำหนดได้ว่าจะรับ Notification แบบไหน

ตัวอย่าง:

User Notification Settings

Order
├── Email ✅
├── Push ✅
└── SMS ❌

Marketing
├── Email ❌
├── Push ❌
└── SMS ❌

Security
├── Email ✅
├── Push ✅
└── SMS ✅

⚠️ แต่ Notification สำคัญด้าน Security (ซีเคียวริตี) เช่น Password Reset อาจไม่ควรให้ User ปิดได้ทั้งหมด

11. Priority 🚨

Notification บางอย่างสำคัญไม่เท่ากัน

CRITICAL
HIGH
NORMAL
LOW

ตัวอย่าง:

Password Reset       → CRITICAL
Payment Failed       → HIGH
Order Completed      → NORMAL
Promotion            → LOW

Queue อาจจัดลำดับ:

CRITICAL
   ↓
HIGH
   ↓
NORMAL
   ↓
LOW

เพื่อไม่ให้ Notification จำนวนมหาศาลจาก Promotion ขัดขวาง Notification สำคัญ

12. Rate Limiting 🚦

ต้องป้องกันการส่ง Notification จำนวนมหาศาล

เช่น Bug ทำให้:

1 User
↓
10,000 Notifications

หรือ

1 นาที
↓
1,000,000 Emails

อาจทำให้ Provider (โพรไวเดอร์) block ระบบ

จึงควรมี Limit เช่น:

User → 10 notifications/min
Email → 1000/min
SMS → 100/min

ค่าจริงต้องออกแบบตามระบบและข้อจำกัดของ Provider

13. Scheduled Notification ⏰

รองรับการส่งตามเวลา

เช่น:

2026-10-01 08:00
       ↓
Send Notification

ตัวอย่าง:

นัดหมาย
แจ้งเตือนงาน
แจ้งเตือนหมดอายุ
แจ้งเตือนชำระเงิน
Reminder (รีไมน์เดอร์)

ต้องระวังเรื่อง:

Time Zone (ไทม์โซน)
Duplicate Job
Server Restart
Retry
Missed Schedule
14. Notification History 📜

ควรมีประวัติ

Notification History

User
Type
Channel
Message
Status
CreatedAt
SentAt
ReadAt
Error

สำหรับ In-App Notification อาจมี:

isRead
readAt

ทำให้ระบบแสดง:

🔔 Notifications (3)

ได้

15. Failure Handling 💥

Provider อาจมีปัญหา

Application
     ↓
Email Provider ❌

ไม่ควรทำให้ระบบหลักล่ม

ควร:

Provider Error
      ↓
Retry
      ↓
Still Failed
      ↓
Dead Letter Queue
      ↓
Alert

Dead Letter Queue (เดด เลทเทอร์ คิว) คือพื้นที่เก็บ Job ที่พยายามส่งแล้วไม่สำเร็จตามจำนวนที่กำหนด เพื่อให้ตรวจสอบหรือดำเนินการภายหลัง

16. Monitoring 📊

Notification System ต้อง Monitor อย่างน้อย:

Notification
├── Queue Length
├── Processing Rate
├── Success Rate
├── Failure Rate
├── Retry Count
├── Processing Time
├── Provider Response Time
└── Dead Letter Count

ตัวอย่าง:

Queue = 100,000
Worker = 10
Processing = 100/sec

แสดงว่า Queue กำลังสะสมเร็ว

ต้องเพิ่ม Worker หรือหาสาเหตุที่ Provider ช้า

🧠 โครงสร้างที่ควรจำ
NOTIFICATION SYSTEM
│
├── Notification Type
├── Template
├── Channel
│   ├── Email
│   ├── SMS
│   ├── Push
│   ├── LINE
│   └── In-App
│
├── Queue
├── Background Worker
│
├── Retry
│   ├── Backoff
│   ├── Jitter
│   └── Max Retry
│
├── Idempotency
├── Deduplication
├── Priority
├── Rate Limiting
│
├── User Preference
├── Scheduling
│
├── Delivery Status
├── Notification History
│
├── Dead Letter Queue
│
└── Monitoring
    ├── Queue
    ├── Success
    ├── Failure
    ├── Retry
    └── Processing Time
🔥 สิ่งที่ควรเข้าใจให้ลึก

ถ้าจะทำระบบระดับ Production (โพรดักชัน) ผมแนะนำให้เข้าใจ 7 เรื่องนี้เป็นพิเศษ:

1. Queue + Worker
เข้าใจว่าทำไมงาน Notification ไม่ควรทำใน Request หลัก

2. Retry + Backoff + Jitter
เข้าใจว่า Retry ที่ออกแบบผิดสามารถทำให้ระบบพังหนักกว่าเดิม

3. Idempotency
ป้องกันการส่ง Notification ซ้ำ

4. Deduplication
ป้องกัน Event เดิมสร้าง Notification ซ้ำ

5. Failure Handling
Provider ล่มต้องไม่ทำให้ API หลักล่ม

6. Priority + Rate Limiting
Notification สำคัญต้องไม่ถูกงานจำนวนมหาศาลกลบ

7. Monitoring Queue
ต้องรู้ว่า Queue กำลังสะสมหรือ Worker กำลังตามงานไม่ทัน

🔗 เชื่อมกับระบบที่เรียนมาก่อน
API
 ↓
Business Logic
 ↓
Database Transaction
 ↓
Create Notification Job
 ↓
Queue
 ↓
Worker
 ↓
Provider
 ↓
Email / SMS / Push / LINE

และถ้า Provider ล่ม:

Provider ❌
    ↓
Retry
    ↓
Backoff
    ↓
Retry Again
    ↓
Dead Letter Queue
    ↓
Alert

ดังนั้น Notification System จะเชื่อมกับ API + Database + Queue + Error Handling + Performance + Logging + Monitoring แทบทั้งหมดเลยครับ