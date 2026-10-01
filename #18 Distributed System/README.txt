
 │
 └── Message → Queue

ปัญหาคือ Network ไม่ได้ reliable 100%

อาจเกิด:

Timeout
Connection Failed
Packet Loss
High Latency
DNS Failure
Service Unavailable

ดังนั้นต้องออกแบบโดยคิดเสมอว่า

Network สามารถพังได้ตลอดเวลา

3️⃣ Service-to-Service Communication

ถ้ามีหลาย Service:

Order Service
     │
     ▼
Payment Service
     │
     ▼
Notification Service

ต้องจัดการเรื่อง:

Timeout
Retry
Authentication
Authorization
Error Handling
Request ID
Correlation ID
Circuit Breaker

เช่น

Order API
   │
   │ request
   ▼
Payment API
   │
   │ timeout
   ▼
Order API

ห้ามปล่อยให้ Order API รอ Payment API แบบไม่จำกัดเวลา

4️⃣ Timeout ⏱️

ทุก Network Call ควรมี Timeout

ตัวอย่าง:

API → Payment Service
Timeout = 3 sec

ถ้าเกิน 3 วินาที:

Payment Service
       ↓
   ไม่ตอบสนอง
       ↓
    Timeout
       ↓
Order API จัดการ Error

ถ้าไม่มี Timeout:

Request
   ↓
รอ
   ↓
รอ
   ↓
รอ
   ↓
Connection ค้าง
   ↓
Pool เต็ม
   ↓
ระบบเริ่มพัง

นี่เชื่อมโดยตรงกับ Connection Pool Starvation

5️⃣ Retry 🔄

ถ้า Service ปลายทางล้มชั่วคราว อาจ Retry

Request
   ↓
Failed
   ↓
Retry #1
   ↓
Failed
   ↓
Retry #2
   ↓
Success

แต่ Retry แบบมั่ว ๆ อันตรายมาก

เช่น

100 Requests
     ↓
Retry
     ↓
300 Requests

ระบบที่กำลังมีปัญหาอาจถูกยิงหนักกว่าเดิม

จึงควรใช้:

Exponential Backoff
+
Jitter
+
Maximum Retry
+
Retry Budget
+
Idempotency
6️⃣ Circuit Breaker ⚡

ถ้า Service ปลายทางพัง อย่ายิงซ้ำตลอด

ตัวอย่าง:

Order API
    │
    ▼
Payment API ❌

ถ้า Fail ต่อเนื่อง:

CLOSED
  ↓
Failures เพิ่ม
  ↓
OPEN

เมื่อ OPEN

Order API
   │
   X
   │
Payment API

ไม่ส่ง Request ไปปลายทางชั่วคราว

เมื่อเวลาผ่านไป:

OPEN
 ↓
HALF OPEN
 ↓
ทดลอง Request
 ↓
Success
 ↓
CLOSED

ช่วยป้องกัน Cascading Failure (ระบบล้มต่อเนื่องเป็นลูกโซ่)

7️⃣ Load Balancing ⚖️

เมื่อมี API หลายเครื่อง:

             ┌─ API #1
User → LB ───┼─ API #2
             └─ API #3

Load Balancer กระจาย Request

เช่น:

Request 1 → API #1
Request 2 → API #2
Request 3 → API #3
Request 4 → API #1

ทำให้รองรับ Load ได้มากขึ้น

8️⃣ Horizontal Scaling

เพิ่มจำนวน Server

ก่อน

API #1


หลัง

API #1
API #2
API #3
API #4

ต่างจาก Vertical Scaling:

Vertical
CPU 4 → 16 Core
RAM 16 → 64 GB

Distributed System มักใช้ Horizontal Scaling มาก

แต่ต้องระวัง:

เพิ่ม API 10 เครื่อง ไม่ได้แปลว่า Database รับ Load ได้ 10 เท่า

เช่น

10 API Servers
      ↓
10 × Connection Pool
      ↓
SQL Server
      ↓
DB รับไม่ไหว
9️⃣ Stateless

Distributed API ควรพยายามเป็น Stateless (ไม่มี State สำคัญอยู่ในเครื่องใดเครื่องหนึ่ง)

ไม่ควร:

User
 ↓
API #1
 ↓
Session อยู่ RAM API #1

เพราะ Request ต่อไปอาจไป API #2

User
 ↓
API #2
 ↓
หา Session ไม่เจอ

แนวทาง:

API #1 ─┐
API #2 ─┼── Redis
API #3 ─┘

หรือใช้ Token-based Authentication

🔟 Distributed Lock 🔒

สมมติ Worker 2 ตัวทำงานพร้อมกัน:

Worker #1 ──┐
            ├── Order #100
Worker #2 ──┘

ทั้งสองตัวเห็นว่า:

Order = PENDING

แล้วทำงานพร้อมกัน

อาจเกิด:

Worker #1 → Process
Worker #2 → Process

เกิด Duplicate Processing

Distributed Lock ใช้สำหรับบอกว่า:

"ตอนนี้มี Worker #1 กำลังทำงานอยู่
Worker อื่นห้ามทำ"

Redis สามารถนำมาใช้ทำ Distributed Lock ได้ แต่ต้องออกแบบเรื่อง TTL, lock ownership, expiry และ failure ให้ถูกต้อง

1️⃣1️⃣ Idempotency

Distributed System ต้องคิดเรื่องนี้หนักมาก

เช่น

POST /payment

User กด 2 ครั้ง

หรือ Network timeout:

Client → Payment API
             ↓
          Payment สำเร็จ
             ↓
        Response หาย
             ↓
Client คิดว่า Failed
             ↓
Retry

ถ้าไม่มี Idempotency:

จ่ายเงิน 2 ครั้ง 💸💸

ถ้ามี:

Idempotency-Key = PAY-10001

ครั้งที่ 1 → Process
ครั้งที่ 2 → พบ Key เดิม
           → ไม่ Process ซ้ำ

นี่เป็นหนึ่งในหัวใจของ Distributed System

1️⃣2️⃣ Consistency

เมื่อมีหลาย Node / Database / Cache ข้อมูลอาจไม่ตรงกันทันที

เช่น:

SQL Server
Stock = 10

Redis
Stock = 10

มีการซื้อสินค้า:

SQL Server
Stock = 9

Redis
Stock = 10

ช่วงเวลาหนึ่งข้อมูลไม่ตรงกัน

เรียกว่า

Eventual Consistency (ความสอดคล้องในที่สุด)

1️⃣3️⃣ Strong Consistency

ระบบต้องการให้ทุกคนเห็นข้อมูลล่าสุดทันทีตามขอบเขตที่ระบบรับประกัน

เหมาะกับข้อมูลบางประเภท เช่น:

เงิน
ยอดบัญชี
Stock ที่ต้องป้องกัน Overselling
สิทธิ์สำคัญ

ต้องแลกกับ Complexity / Latency / Availability บางส่วน

1️⃣4️⃣ Eventual Consistency

ตัวอย่าง:

SQL Server
   ↓
Update Product
   ↓
Event
   ↓
Queue
   ↓
Worker
   ↓
Search Index

ช่วงหนึ่ง:

Database = ใหม่
Search Index = เก่า

จากนั้น Worker ทำงาน:

Search Index = ใหม่

เหมาะกับ:

Search
Notification
Analytics
Report
Cache
Data Synchronization
1️⃣5️⃣ Replication

ข้อมูลอาจมีหลาย Copy

Primary DB
    │
    ├── Replica #1
    ├── Replica #2
    └── Replica #3

ข้อดี:

Read Scale
High Availability
Failover
Disaster Recovery

แต่ต้องเข้าใจว่า Replica อาจมี Replication Lag

1️⃣6️⃣ Partition / Sharding

เมื่อข้อมูลใหญ่มาก อาจแบ่งข้อมูลออกเป็นส่วน

ตัวอย่าง:

Customer ID

1 - 1,000,000
     ↓
Shard #1

1,000,001 - 2,000,000
     ↓
Shard #2

หรือแบ่งตาม Region:

Thailand → DB #1
Japan    → DB #2
USA      → DB #3

ช่วย Scale แต่ทำให้ Query / Transaction / Data Management ซับซ้อนขึ้นมาก

1️⃣7️⃣ Leader Election 👑

บางงานต้องมี Node ที่เป็น Leader

Node #1 → Leader
Node #2 → Follower
Node #3 → Follower

ถ้า Leader ตาย:

Node #1 ❌

Node #2
   ↓
Leader

เรียกว่า Leader Election

พบในระบบ Distributed หลายประเภท เช่น cluster และ distributed coordination systems

1️⃣8️⃣ Consensus

Consensus (ฉันทามติ) = ทำให้หลาย Node ตกลงกันได้ว่า State ไหนคือสิ่งที่ถูกต้อง

แนวคิดนี้ลึกขึ้นไปถึง:

Raft
Paxos
Leader Election
Replication
Quorum

ไม่จำเป็นต้องเริ่มจากการเขียน Consensus เอง แต่ควรเข้าใจ Concept

1️⃣9️⃣ Distributed Transaction

อันนี้สำคัญมาก

สมมติ:

Order Service
      ↓
Order DB

Payment Service
      ↓
Payment DB

เราต้องการ:

Order สำเร็จ
+
Payment สำเร็จ

แต่ถ้า:

Order DB → SUCCESS
Payment DB → FAILED

จะเกิดอะไรขึ้น?

นี่คือปัญหา Distributed Transaction

แนวทางหนึ่งคือ Saga Pattern

Create Order
    ↓
Reserve Stock
    ↓
Payment
    ↓
Confirm Order

ถ้าขั้นตอนกลางล้ม:

Payment Failed
    ↓
Cancel Stock
    ↓
Cancel Order

เรียกว่า Compensating Transaction

2️⃣0️⃣ Message Queue

Distributed System มักใช้ Queue เพื่อแยก Service

Order Service
      ↓
    Queue
      ↓
Notification Worker

ข้อดี:

ลดการรอ
รองรับ Burst Load
Retry
Async Processing
Decouple Services

แต่ต้องจัดการ:

Duplicate Message
Ordering
Retry
DLQ
Backpressure
Idempotency

ซึ่งเชื่อมกับ #10 Queue / Background Job โดยตรง

2️⃣1️⃣ Distributed Tracing 🔍

เวลามี Request หนึ่งตัววิ่งผ่านหลาย Service:

User
 ↓
Next.js
 ↓
Go API
 ↓
Order Service
 ↓
Payment Service
 ↓
SQL Server

ต้องรู้ว่า Request เดียวกันใช้เวลาเท่าไรในแต่ละจุด

จึงมี:

Trace ID
Span
Correlation ID

ตัวอย่าง:

Trace ID: ABC123

Next.js       20ms
Go API        50ms
Order Service 30ms
Payment API   800ms  ← ช้า
SQL Server    20ms

ทำให้รู้ว่า Bottleneck อยู่ตรงไหน

นี่เชื่อมกับ #8 Monitoring & Observability

2️⃣2️⃣ Fault Tolerance

ระบบต้องออกแบบให้บางส่วนพังแล้วระบบส่วนอื่นยังทำงานได้

เช่น:

Notification Service ❌

แต่:

Order Service
Payment Service
Database

ยังทำงานได้

Notification ค่อย Retry ภายหลัง

ไม่ควร:

Notification พัง
      ↓
Order พัง
      ↓
Payment พัง
      ↓
ทั้งระบบพัง
2️⃣3️⃣ Failover

ถ้า Node หลักพัง:

Primary ❌
   ↓
Secondary
   ↓
รับงานต่อ

ต้องคิดเรื่อง:

Detection
Health Check
Failover
Recovery
Data Consistency
Split Brain
2️⃣4️⃣ Split Brain 🧠

เป็นปัญหาที่ Node หลายตัวเข้าใจว่าตัวเองเป็น Leader พร้อมกัน

เช่น:

Network Partition

     X
     X
────────────

Node A → คิดว่าตัวเองเป็น Leader
Node B → คิดว่าตัวเองเป็น Leader

แล้วทั้งสองตัวเขียนข้อมูล

A → Write
B → Write

อาจเกิด Data Conflict / Corruption ได้

นี่เป็นเหตุผลว่าทำไม Distributed Coordination ถึงซับซ้อน

2️⃣5️⃣ Cascading Failure

ปัญหาที่สำคัญมากสำหรับ Production

Payment Service ช้า
       ↓
Order API รอ
       ↓
Connection ค้าง
       ↓
API Connection Pool เต็ม
       ↓
Request ใหม่รอ
       ↓
Timeout
       ↓
Retry
       ↓
Load เพิ่ม
       ↓
ระบบล้มหนักกว่าเดิม

นี่คือเหตุผลที่ Distributed System ต้องมี:

Timeout
Retry Limit
Circuit Breaker
Bulkhead
Rate Limiting
Backpressure
Queue
2️⃣6️⃣ Bulkhead Pattern

แบ่ง Resource ออกจากกัน

เช่น:

API Server
│
├── Payment Pool
│   └── 20 connections
│
├── Report Pool
│   └── 10 connections
│
└── Normal API Pool
    └── 50 connections

ถ้า Report หนัก:

Report Pool เต็ม

แต่ API ปกติยังสามารถทำงานได้

เปรียบเหมือนเรือที่มีผนังกั้นหลายห้อง หากห้องหนึ่งน้ำเข้า ไม่จำเป็นต้องจมทั้งลำ

🔥 ปัญหาหลักที่ต้องจำ

Distributed System ต้องคิดเรื่องนี้เสมอ:

Network Failure
Timeout
Retry
Duplicate Request
Duplicate Message
Race Condition
Deadlock
Distributed Lock
Data Consistency
Replication Lag
Service Failure
Node Failure
Database Failure
Cache Failure
Queue Failure
Clock Difference
Split Brain
Cascading Failure
🔗 เชื่อมกับระบบที่เรียนมาก่อนหน้า

นี่คือจุดที่ #18 Distributed System เอาทุกระบบก่อนหน้ามารวมกัน

                    Distributed System
                           │
        ┌─────────────┼──────────────┐
        ▼                  ▼                  ▼
      API              Database             Queue
        │                  │                  │
        ▼                  ▼                  ▼
   Timeout             Transaction          Worker
   Retry               Lock                 Retry
   Circuit             Deadlock             DLQ
   Breaker             Isolation            Idempotency
        │                  │                  │
        └─────────────┼─────────────┘
                           ▼
                         Redis
                           │
                    Cache / Lock
                           │
                           ▼
                    Observability
                    Logs / Metrics
                    Tracing
🧩 สำหรับ Architecture ของคุณ

ถ้าใช้ Architecture ที่คุณวางไว้:

                 User
                   │
                   ▼
              Next.js
                   │
                   ▼
              Load Balancer
                   │
          ┌─────┴──────┐
          ▼                ▼
       Go API #1         Go API #2
          │                 │
          └─────┬──────┘
                   │
        ┌───────┼─────────┐
        ▼          ▼           ▼
      Redis      Queue      SQL Server
        │          │
        │       ┌─┴─┐
        │       ▼    ▼
        │   Worker1 Worker2
        │
        ▼
     Cache/Lock

สิ่งที่ต้องออกแบบให้ดีคือ:

API
→ Stateless + Timeout + Rate Limit

Database
→ Connection Pool + Transaction + Isolation + Deadlock

Redis
→ Cache + Distributed Lock + Rate Limit

Queue
→ Retry + DLQ + Idempotency + Backpressure

Worker
→ Concurrency + Graceful Shutdown + Idempotency

Infrastructure
→ Load Balancer + Health Check + Failover

Observability
→ Log + Metrics + Trace ID

📚 Deep Topics ที่ควรศึกษา

ถ้าจะเข้าใจ Distributed System จริง ๆ ฉันแนะนำลำดับนี้:

1. Client / Server
2. Network Communication
3. Timeout
4. Retry
5. Idempotency
6. Stateless Service
7. Load Balancing
8. Horizontal Scaling
9. Cache
10. Message Queue
11. Eventual Consistency
12. Distributed Lock
13. Distributed Transaction
14. Saga Pattern
15. Replication
16. Partitioning
17. Failover
18. Fault Tolerance
19. Circuit Breaker
20. Bulkhead
21. Backpressure
22. Leader Election
23. Consensus
24. Split Brain
25. Distributed Tracing
26. CAP Theorem
27. Consistency Models
28. Distributed Systems Failure Model
⭐ 5 เรื่องที่อยากให้จำให้แม่นที่สุด
1. Network สามารถล้มได้
2. Request สามารถถูกส่งซ้ำได้
3. Message สามารถถูกประมวลผลซ้ำได้
4. Node สามารถล้มได้ทุกเมื่อ
5. ข้อมูลหลาย Node อาจไม่ตรงกันทันที

และถ้าเอา Distributed System + ระบบที่เรียนมาก่อน มารวมกัน จะเห็นภาพใหญ่แบบนี้:

User
 ↓
Next.js
 ↓
Load Balancer
 ↓
Go API
 ↓
┌───────────────┬───────────────┬───────────────┐
│               │               │
Redis          Queue         SQL Server
│               │               │
Cache/Lock      Worker        Transaction
│               │               │
└───────────────┴───────────────┘
        ↓
Monitoring / Logging / Tracing

หัวใจของ #18 ไม่ใช่การมีหลาย Server แต่คือการทำให้หลายส่วนที่อาจล้ม ช้า หรือทำงานพร้อมกัน สามารถทำงานร่วมกันได้โดยระบบยังคงความถูกต้องและความน่าเชื่อถือไว้ได้.