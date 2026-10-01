Queue / Background Job System ⚙️

ระบบคิวและงานเบื้องหลัง

หน้าที่หลักคือ เอางานที่ไม่จำเป็นต้องทำทันที ออกจาก Request หลัก แล้วให้ Worker (เวิร์กเกอร์) ทำภายหลัง

ตัวอย่างงาน:

ส่ง Email
ส่ง SMS
ส่ง Notification
Generate Report
Export Excel
ประมวลผลไฟล์
Resize Image
Import ข้อมูลจำนวนมาก
Sync ข้อมูลระหว่างระบบ
เรียก External API (เอ็กซ์เทอร์นัล เอพีไอ)
ล้างข้อมูลเก่า
ประมวลผลข้อมูลเป็น Batch (แบตช์)
🧠 ทำไมต้องมี Queue?

สมมติ API มีงานแบบนี้

POST /orders
        ↓
Create Order
        ↓
Generate PDF
        ↓
Send Email
        ↓
Send LINE
        ↓
Sync ERP
        ↓
Response

ถ้าแต่ละงานใช้เวลา:

Create Order = 100ms
PDF          = 2s
Email        = 1s
LINE         = 500ms
ERP          = 3s

ผู้ใช้ต้องรอประมาณ 6.6 วินาที

เปลี่ยนเป็น:

POST /orders
        ↓
Create Order
        ↓
Create Jobs
        ↓
Response
        ↓
   Queue
      ↓
   Worker
      ↓
 ┌────┼────┬────┐
 PDF Email LINE ERP

API สามารถตอบกลับได้เร็วขึ้น และงานหนักถูกแยกออกไป

1. Queue 📦

Queue (คิว) คือพื้นที่สำหรับเก็บงานที่รอให้ Worker ประมวลผล

Producer
(ผู้สร้างงาน)
     ↓
   Queue
     ↓
  Worker
(ผู้ทำงาน)

ตัวอย่าง:

Queue
│
├── Job #1001 → Send Email
├── Job #1002 → Generate PDF
├── Job #1003 → Sync ERP
├── Job #1004 → Resize Image
└── Job #1005 → Send Notification
2. Producer 🏭

Producer (โพรดิวเซอร์) คือส่วนที่สร้าง Job

เช่น API:

POST /orders
       ↓
Create Order
       ↓
Queue.Add(
    GenerateInvoice
)

Producer ไม่จำเป็นต้องทำงานเอง

มันแค่บอกว่า:

"มีงานนี้นะ เอาไปทำทีหลัง"

3. Consumer / Worker 👷

Consumer (คอนซูมเมอร์) หรือ Worker คือผู้หยิบงานจาก Queue ไปทำ

Queue
 ↓
Worker
 ↓
Get Job
 ↓
Process
 ↓
Success

ตัวอย่าง:

Job
{
    type: "SEND_EMAIL",
    orderId: 10001
}

Worker:

Get Job
   ↓
Load Order
   ↓
Create Email
   ↓
Send Email
   ↓
Success
4. Background Job 🌙

Background Job (แบ็กกราวด์ จ็อบ) คือ Job ที่ไม่จำเป็นต้องทำใน Request หลัก

ตัวอย่าง:

User
 ↓
Upload Excel
 ↓
API Response
 ↓
"กำลังประมวลผล"
       ↓
    Queue
       ↓
    Worker
       ↓
Import 100,000 rows

เหมาะมากกับงานที่ใช้เวลานาน

5. Job Status 📊

ควรมีสถานะของ Job

PENDING
   ↓
PROCESSING
   ↓
COMPLETED

ถ้าเกิด Error:

PENDING
   ↓
PROCESSING
   ↓
FAILED
   ↓
RETRY
   ↓
PROCESSING

เช่น:

Job ID: 10001
Type: IMPORT_EXCEL
Status: PROCESSING
Attempt: 2
CreatedAt: ...
StartedAt: ...
CompletedAt: ...
6. Retry 🔄

Worker ทำงานแล้วล้ม ไม่ควรทิ้ง Job ทันที

Job
 ↓
Attempt 1 ❌
 ↓
Wait
 ↓
Attempt 2 ❌
 ↓
Wait
 ↓
Attempt 3 ✅

ต้องกำหนด:

Max Retry
Retry Delay
Backoff
Jitter

เช่น:

1st retry → 1 sec
2nd retry → 5 sec
3rd retry → 30 sec
7. Dead Letter Queue ☠️

Dead Letter Queue หรือ DLQ (ดีแอลคิว)

ใช้เก็บ Job ที่ล้มเหลวซ้ำจนเกินจำนวน Retry ที่กำหนด

Queue
 ↓
Worker
 ↓
❌
 ↓
Retry 1
 ↓
❌
 ↓
Retry 2
 ↓
❌
 ↓
Retry 3
 ↓
❌
 ↓
DLQ

จากนั้นทีมสามารถตรวจสอบ:

Why failed?
What data?
Which error?
How many attempts?
8. Idempotency 🔐

สำคัญมาก

สมมติ Worker ประมวลผล:

Create Payment

ทำสำเร็จแล้ว แต่ก่อน Worker จะบันทึกว่า COMPLETED Server ดันล่ม

เมื่อระบบกลับมา:

Job = PENDING

Worker อาจหยิบไปทำอีกครั้ง

อาจเกิด:

Payment #1
Payment #2

ดังนั้น Job สำคัญต้องออกแบบให้ ทำซ้ำแล้วไม่สร้างผลลัพธ์ซ้ำ

เช่น:

IdempotencyKey
=
ORDER-10001-PAYMENT

แล้วตรวจสอบก่อนทำงาน

9. At-Least-Once / At-Most-Once / Exactly-Once

เรื่องนี้ควรเข้าใจให้ดี

At-Most-Once

ทำงาน ไม่เกิน 1 ครั้ง

0 หรือ 1 ครั้ง

ข้อเสียคือ Job อาจหาย

At-Least-Once

ทำงาน อย่างน้อย 1 ครั้ง

1 หรือหลายครั้ง

ข้อดีคือ Job มีโอกาสไม่หาย

ข้อเสียคืออาจทำซ้ำ

ดังนั้นต้องใช้:

At-Least-Once
+
Idempotency

เป็นแนวทางที่ใช้จริงได้ดี

Exactly-Once

ตั้งใจให้:

ทำเพียงครั้งเดียว

แต่ในระบบ Distributed System (ดิสทริบิวเต็ด ซิสเต็ม) จริง ๆ การรับประกัน Exactly-Once ตั้งแต่ต้นทางถึงปลายทางทำได้ยากมาก

ดังนั้นอย่าออกแบบโดยคิดว่า:

"Job จะไม่มีวันถูกทำซ้ำ"

ให้คิดว่า:

Job อาจถูกทำซ้ำได้ จึงต้องทำให้การทำซ้ำปลอดภัย

10. Queue Ordering 📋

บางงานต้องรักษาลำดับ

ตัวอย่าง:

Order Created
      ↓
Order Paid
      ↓
Order Shipped

ห้ามเกิด:

Order Shipped
      ↓
Order Created

จึงต้องพิจารณา:

FIFO (ไฟโฟ)
Partition (พาร์ทิชัน)
Ordering Key (ออร์เดอริง คีย์)

เช่นใช้:

OrderId = 10001

เป็น Key เพื่อให้ Job ของ Order เดียวกันถูกประมวลผลตามลำดับ

11. Priority Queue 🚨

งานบางอย่างสำคัญกว่า

HIGH
MEDIUM
LOW

เช่น:

HIGH
├── Payment
├── Password Reset
└── Security Alert

LOW
├── Marketing Email
├── Report
└── Analytics

ถ้า Queue มีงาน 1 ล้านรายการ

ไม่ควรปล่อยให้ Marketing Job กิน Worker ทั้งหมดจน Payment Job ต้องรอ

12. Concurrency 👥

สามารถมี Worker หลายตัว

             Queue
               │
       ┌───────┼───────┐
       ↓       ↓       ↓
    Worker1 Worker2 Worker3
       ↓       ↓       ↓
      Job     Job     Job

ทำให้ประมวลผลพร้อมกันได้

แต่ต้องระวัง:

Concurrency
+
Database
+
External API

เช่นมี Worker 100 ตัว แต่ SQL Server รับงานพร้อมกันไม่ไหว

ผลอาจกลายเป็น:

100 Workers
     ↓
100 DB Connections
     ↓
Connection Pool เต็ม
     ↓
Blocking
     ↓
Timeout

ดังนั้น เพิ่ม Worker ไม่ได้แปลว่าระบบจะเร็วขึ้นเสมอ

13. Worker Scaling 📈

ถ้า Queue เพิ่มขึ้นเรื่อย ๆ:

Queue
100
 ↓
1,000
 ↓
10,000
 ↓
100,000

อาจต้องเพิ่ม Worker

Worker × 2
Worker × 5
Worker × 10

แต่ต้องดู Bottleneck (บอตเทิลเน็ก) ด้วย

Queue
 ↓
Worker
 ↓
Database ← Bottleneck

ถ้า DB เป็นคอขวด ต่อให้เพิ่ม Worker จาก 10 → 100 ก็อาจทำให้ DB แย่ลง

14. Queue Backpressure 🛑

Backpressure (แบ็กเพรสเชอร์) คือการควบคุมเมื่อระบบผลิตงานเร็วเกินกว่าที่ Worker จะประมวลผลได้

ตัวอย่าง:

Producer
1000 jobs/sec
      ↓
Queue
      ↓
Worker
100 jobs/sec

Queue จะเพิ่มขึ้นเรื่อย ๆ

100
1,000
10,000
100,000

ระบบต้องมีวิธีรับมือ เช่น:

จำกัดจำนวน Job
จำกัด Producer
Rate Limit
Reject งานบางประเภท
เพิ่ม Worker
ลดงานที่ไม่สำคัญ
ใช้ Priority
15. Scheduled Job ⏰

บาง Job ไม่ได้เกิดจาก API แต่ต้องทำตามเวลา

ตัวอย่าง:

ทุก 01:00
 ↓
Backup Check

หรือ:

ทุก 5 นาที
 ↓
Sync ERP

หรือ:

ทุกวัน 08:00
 ↓
ส่ง Report

เรียกว่า Scheduled Job (สเคดจูลด์ จ็อบ)

ต้องระวังเรื่อง:

Server Restart
Time Zone
Duplicate Execution
Missed Schedule
16. Batch Job 📦

บางงานไม่ควรทำทีละ Record

❌

100,000 records
 ↓
100,000 DB calls

ดีกว่า:

100,000 records
 ↓
Batch 1 → 1,000
Batch 2 → 1,000
Batch 3 → 1,000
...

ช่วยลด:

Database Round Trip
Network Overhead
Transaction Overhead
Memory Pressure
17. Queue Monitoring 📊

ต้อง Monitor (มอนิเทอร์) อย่างน้อย:

Queue Length
Processing Rate
Success Rate
Failure Rate
Retry Count
DLQ Count
Job Age
Processing Time
Worker Count
Worker Error

ตัวที่สำคัญมากคือ:

Queue Depth

จำนวน Job ที่รอ

Queue = 100

ปกติ

Queue = 100,000

เริ่มมีปัญหา

Oldest Job Age

Job ที่เก่าที่สุดรอมานานแค่ไหน

เช่น:

Oldest Job = 45 minutes

แสดงว่า Worker ประมวลผลไม่ทัน

🧠 Mental Model
QUEUE / BACKGROUND JOB SYSTEM
│
├── Producer
│
├── Queue
│   ├── FIFO
│   ├── Priority
│   └── Partition
│
├── Worker
│   ├── Concurrency
│   ├── Scaling
│   └── Graceful Shutdown
│
├── Job
│   ├── Pending
│   ├── Processing
│   ├── Completed
│   ├── Failed
│   └── Retry
│
├── Reliability
│   ├── Retry
│   ├── Backoff
│   ├── Jitter
│   ├── Idempotency
│   └── Deduplication
│
├── Failure
│   ├── Dead Letter Queue
│   └── Error Handling
│
├── Performance
│   ├── Batch
│   ├── Concurrency
│   ├── Backpressure
│   └── Scaling
│
├── Scheduling
│   ├── Delayed Job
│   └── Scheduled Job
│
└── Monitoring
    ├── Queue Depth
    ├── Job Age
    ├── Processing Time
    ├── Failure Rate
    ├── Retry Count
    └── DLQ
🔥 เรื่องที่ควรเข้าใจให้ลึก

สำหรับคุณที่สนใจ Go + SQL Server + API ที่รองรับ Concurrent Request จำนวนมาก ผมจะให้ความสำคัญกับ 8 เรื่องนี้:

1. Queue Architecture
2. Worker / Worker Pool
3. Concurrency Control
4. Retry + Exponential Backoff + Jitter
5. Idempotency
6. Dead Letter Queue
7. Backpressure
8. Queue Monitoring

และต้องเข้าใจความสัมพันธ์นี้ให้แม่น:

API
 ↓
Producer
 ↓
Queue
 ↓
Worker Pool
 ↓
Database / External API

เมื่อโหลดสูง:

100,000 Requests
        ↓
     Queue
        ↓
   Worker Pool
        ↓
 ┌──────┼──────┐
 ↓      ↓      ↓
 DB    API    File

แต่ถ้าออกแบบผิด:

100 Workers
     ↓
100 DB Connections
     ↓
SQL Server รับไม่ไหว
     ↓
Blocking
     ↓
Connection Pool เต็ม
     ↓
Timeout
     ↓
Retry
     ↓
Queue เพิ่ม
     ↓
🔥 ระบบยิ่งพัง

นี่คือเหตุผลที่ Queue System ไม่ใช่แค่ "เอางานไปใส่คิว" แต่เกี่ยวข้องโดยตรงกับ Concurrency, Database, Connection Pool, Retry, Error Handling, Performance และ Monitoring ทั้งหมดครับ