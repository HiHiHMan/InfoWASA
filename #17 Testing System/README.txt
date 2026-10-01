Testing System คืออะไร?

Testing System (เทสทิง ซิสเท็ม) คือกระบวนการตรวจสอบ Software ว่า

ทำงานถูกต้องหรือไม่?
ตรงตาม Requirement หรือไม่?
มี Bug หรือไม่?
รองรับ Load หรือไม่?
ปลอดภัยหรือไม่?
แก้ Code แล้วของเดิมพังหรือไม่?
Deploy แล้วทำงานจริงหรือไม่?

Mental Model ง่าย ๆ:

Code
 ↓
Test
 ↓
พบ Bug?
 ├── YES → Fix
 └── NO
      ↓
   Deploy
2. Core Components
TESTING SYSTEM
├── Test Strategy
├── Unit Test
├── Integration Test
├── API Test
├── E2E Test
├── Regression Test
├── Functional Test
├── Non-Functional Test
│   ├── Performance
│   ├── Load
│   ├── Stress
│   ├── Endurance
│   └── Scalability
├── Security Test
├── Database Test
├── Contract Test
├── Mock / Stub
├── Test Data
├── Test Environment
├── UAT
├── Automation
├── CI/CD Test
├── Test Report
└── Bug / Defect Management
3. Unit Test 🔬

Unit Test (ยูนิต เทสต์) คือทดสอบหน่วยเล็ก ๆ ของ Code

เช่น Function

func Add(a int, b int) int {
    return a + b
}

Test:

Add(2, 3)
→ 5

หรือ Business Logic

CalculateDiscount()
CalculateTax()
CalculateTotal()
จุดเด่น
เร็ว
แยกง่าย
Debug ง่าย
ตัวอย่าง
CalculateTotal()
├── 100 + 20 → 120
├── 100 + 0  → 100
└── -100      → Error
4. Integration Test 🔗

Integration Test (อินทิเกรชัน เทสต์) ทดสอบว่า Component หลายตัวทำงานร่วมกันได้หรือไม่

เช่น

Go API
 ↓
Service
 ↓
Repository
 ↓
SQL Server

ตัวอย่าง

POST /orders
 ↓
Order Service
 ↓
INSERT SQL Server
 ↓
ตรวจสอบข้อมูล

ต่างจาก Unit Test ตรงที่ Integration Test สนใจ การเชื่อมต่อระหว่างระบบ

5. API Test 🌐

ทดสอบ API โดยตรง

เช่น

POST /api/orders

ส่ง

{
  "productId": 100,
  "quantity": 2
}

ตรวจ

Status Code
Response
Database
Business Logic
Error

ตัวอย่าง

Valid Request
→ 201

Invalid Request
→ 400

Unauthenticated
→ 401

No Permission
→ 403

Not Found
→ 404

Duplicate
→ 409

เชื่อมกับ #4 API System

6. End-to-End Test

E2E Test (เอนด์-ทู-เอนด์) ทดสอบตั้งแต่ User จนถึง Backend

เช่น Login

Browser
 ↓
Next.js
 ↓
Go API
 ↓
SQL Server
 ↓
Response
 ↓
Next.js
 ↓
Browser

ตัวอย่าง Test

เปิด Login
 ↓
กรอก Username
 ↓
กรอก Password
 ↓
กด Login
 ↓
Dashboard

เป็นการจำลอง User จริง

7. Test Pyramid 🔺

แนวคิดสำคัญ

             E2E
            /   \
          API / Integration
        /             \
          Unit Tests

โดยทั่วไป

Unit
→ เยอะที่สุด
→ เร็ว

Integration
→ ปานกลาง

E2E
→ น้อยกว่า
→ ช้ากว่า
→ ดูแลยากกว่า

ไม่ควรสร้างทุกอย่างเป็น E2E เพราะ Test Suite จะช้าและเปราะ

8. Functional Testing

Functional Test (ฟังก์ชันนัล เทสต์) ตรวจว่า Feature ทำงานตรง Requirement หรือไม่

ตัวอย่าง Order

Quantity = 2
Price = 100

Expected:
Total = 200

Test Case:

TC-001
Create Order

Input:
Product = A
Qty = 2

Expected:
Order Created
Total = 200
9. Regression Testing 🔄

Regression Test (รีเกรสชัน เทสต์) คือการตรวจว่า

แก้ของใหม่แล้วของเก่าพังหรือไม่

ตัวอย่าง

แก้ Login
 ↓
Login ผ่าน

แต่ต้องตรวจด้วยว่า

Register
Logout
Reset Password
Permission
Profile

ยังทำงานเหมือนเดิมหรือไม่

นี่เป็นเหตุผลที่ Automation Test สำคัญ

10. Smoke Test 💨

Smoke Test (สโมก เทสต์) เป็นการตรวจแบบเร็ว ๆ ว่า Build นี้ “พอใช้งานได้ไหม”

เช่นหลัง Deploy

GET /health
      ↓
Login
      ↓
Dashboard
      ↓
Create Order

ถ้าสิ่งพื้นฐานพัง

❌ Stop

ไม่ต้องเสียเวลา Run Test ใหญ่ทั้งหมด

11. Sanity Test

Sanity Test (แซนิตี้ เทสต์) คล้าย Smoke Test แต่เน้นตรวจ Feature ที่เพิ่งแก้

เช่นแก้

Payment

ก็ตรวจ

Create Payment
Payment Status
Webhook
Refund

เป็นหลัก

12. Performance Testing ⚡

นี่สำคัญกับระบบที่คุณสนใจมาก

ต้องทดสอบว่า System รับ Load ได้แค่ไหน

100 Users
500 Users
1,000 Users
5,000 Users

ดู

Response Time
Throughput
Error Rate
CPU
Memory
DB
Connection Pool
13. Load Test

Load Test (โหลด เทสต์) ทดสอบภายใต้ Load ที่คาดว่าจะเจอจริง

เช่น

1,000 concurrent users

ตรวจว่า

P50
P95
P99
Error Rate

เป็นเท่าไร

ตัวอย่าง

1,000 Requests

P95 = 800 ms
Error = 0.2%
14. Stress Test 💥

Stress Test (สเตรส เทสต์) เพิ่ม Load เกินกว่าปกติ เพื่อดูว่า System จะพังตรงไหน

1,000
 ↓
2,000
 ↓
5,000
 ↓
10,000
 ↓
20,000

ต้องหา

Breaking Point (เบรกกิง พอยต์)

เช่น

5,000 Users
→ OK

10,000 Users
→ DB CPU 100%

12,000 Users
→ Timeout

ทำให้รู้ว่าคอขวดอยู่ตรงไหน

15. Spike Test 📈

Spike Test (สไปก์ เทสต์) ทดสอบ Load ที่เพิ่มขึ้นอย่างรวดเร็ว

เช่น

100 users
    ↓
    ↓
    ↓
10,000 users

เหมาะกับระบบที่อาจเจอ Traffic Burst

เช่น

Flash Sale
Event
Announcement
Campaign
16. Endurance Test

Endurance Test (เอนดูแรนซ์ เทสต์) หรือ Soak Test

ทดสอบระบบเป็นเวลานาน

เช่น

Load 500 Users
       ↓
       ↓
       ↓
24 Hours

เอาไว้หา

Memory Leak
Connection Leak
Connection Pool Exhaustion
Memory Growth
Disk Growth
Queue Growth

เชื่อมกับ #5 Performance + #8 Monitoring

17. Scalability Test

ทดสอบว่าเพิ่ม Resource แล้วรองรับ Load ได้ดีขึ้นหรือไม่

เช่น

1 API
→ 1,000 req/s

2 API
→ 1,900 req/s

4 API
→ 3,500 req/s

ไม่ได้หมายความว่าเพิ่ม Server 4 เท่าแล้วต้องเร็ว 4 เท่า

เพราะอาจติด

Database
Network
Lock
Connection Pool
External API
18. Security Testing 🔐

ทดสอบเรื่อง Security เช่น

SQL Injection
XSS
CSRF
Authentication
Authorization
IDOR
Rate Limit
Brute Force
File Upload
JWT
Session

ตัวอย่าง

User A
 ↓
GET /orders/999

แต่ Order 999 เป็นของ User B

ต้องได้

403

ไม่ใช่

200

เชื่อมกับ #2 Security System

19. Database Testing 🗄️

ทดสอบ Database ด้วย

Constraint
Foreign Key
Unique
Transaction
Rollback
Deadlock
Concurrency
Index
Stored Procedure
Trigger

ตัวอย่าง

BEGIN TRANSACTION

UPDATE Stock
INSERT Order

เกิด Error

ROLLBACK

ตรวจว่า

Stock กลับเหมือนเดิม
Order ไม่ถูกสร้าง
20. Concurrency Testing 🔥

อันนี้สำคัญมากสำหรับ Production

สมมติ Stock เหลือ

Stock = 1

มี User 2 คนซื้อพร้อมกัน

User A ──┐
         ├── Buy
User B ──┘

ต้องไม่เกิด

A → ซื้อสำเร็จ
B → ซื้อสำเร็จ

Stock = -1 ❌

ต้องทดสอบ

Race Condition
Deadlock
Lock
Isolation
Duplicate Request
Idempotency

เชื่อมกับ #3 Database + #5 Performance

21. Mock / Stub 🎭

เวลาทดสอบไม่จำเป็นต้องเรียก External Service จริงเสมอไป

เช่น

Go API
 ↓
Payment Provider

ในการ Test อาจ Mock

Go API
 ↓
Mock Payment Provider

จำลอง

Success
Failed
Timeout
500
Slow Response

ทำให้ Test สถานการณ์ยาก ๆ ได้ง่ายขึ้น

22. Test Data

ต้องมีข้อมูลสำหรับ Test

Test User
Test Product
Test Order
Test Payment

ควรแยกจาก Production

DEV DB
UAT DB
PROD DB

ห้ามเอา Production Data ที่มี Sensitive Data มาใช้แบบไม่ควบคุม

23. Test Environment

ควรมี Environment สำหรับ Test

Development
 ↓
Testing
 ↓
UAT
 ↓
Production

แต่ Environment ไม่จำเป็นต้องเหมือนกัน 100%

อย่างน้อยควรทำให้ Dependency สำคัญ ๆ ใกล้เคียง Production พอที่จะตรวจพฤติกรรมที่ต้องการ

24. UAT 👨‍💼

UAT (User Acceptance Testing) คือการให้ User/Business ตรวจว่า

ระบบตรงกับการทำงานจริงหรือไม่

ตัวอย่าง

Requirement:
Manager สามารถ Approve Order ได้

UAT

Manager Login
 ↓
Open Order
 ↓
Approve
 ↓
Status = APPROVED

UAT ต่างจาก Unit Test เพราะไม่ได้สนใจแค่ Code ถูก แต่สนใจ Business Requirement

25. Test Automation 🤖

แทนที่จะ Test ด้วยมือทุกครั้ง

Developer
 ↓
Push Code
 ↓
Automated Test
 ↓
Result

เช่น

1,000 Unit Tests
100 API Tests
50 Integration Tests
20 E2E Tests

ให้ CI/CD รันเอง

เชื่อมกับ #16 Deployment & Infrastructure

26. CI/CD Testing

Pipeline สามารถเป็น

Git Push
   ↓
Lint
   ↓
Unit Test
   ↓
Build
   ↓
Integration Test
   ↓
Security Test
   ↓
Docker Build
   ↓
Deploy UAT
   ↓
Smoke Test
   ↓
Deploy Production

ถ้า Test Fail

❌ Pipeline Stop
27. Contract Testing

Contract Test (คอนแทรกต์ เทสต์) ใช้ตรวจว่า Service คุยกันตาม Contract เดิมหรือไม่

เช่น

Frontend
   ↓
Go API

API บอกว่าจะส่ง

{
  "id": 100,
  "name": "Product A"
}

ถ้า Backend เปลี่ยนเป็น

{
  "product_id": 100,
  "product_name": "Product A"
}

Frontend อาจพัง

Contract Test ช่วยจับปัญหานี้ก่อน Deploy

28. Test Coverage 📊

Code Coverage (โค้ด คัฟเวอเรจ) บอกว่า Test ครอบคลุม Code แค่ไหน

เช่น

1000 Lines
Tested 800

Coverage

80%

แต่ต้องจำว่า

Coverage สูง ≠ ระบบถูกต้อง 100%

เพราะ Test อาจ Run Code แต่ไม่ได้ตรวจผลลัพธ์อย่างถูกต้อง

29. Test Case

Test Case ควรระบุ

Test ID
Test Name
Precondition
Input
Steps
Expected Result
Actual Result
Status

ตัวอย่าง

TC-001

Name:
Login Success

Input:
Username = admin
Password = correct

Expected:
Login successful
Token returned

Status:
PASS
30. Negative Testing ❌

อย่าทดสอบเฉพาะกรณีสำเร็จ

ต้องทดสอบกรณีผิดด้วย

Empty Input
Invalid Input
Wrong Password
Expired Token
No Permission
Duplicate Request
Database Error
Timeout
External API Failure

ตัวอย่าง

Quantity = -10

ต้องไม่สร้าง Order

31. Boundary Testing

ทดสอบค่าขอบเขต

เช่น

Quantity: 1 - 100

ควร Test

0
1
2
99
100
101

ไม่ใช่แค่

50

เพราะ Bug มักอยู่ตรง Boundary

32. Bug / Defect Management 🐞

เมื่อเจอ Bug ต้องบันทึก

Bug ID
Title
Environment
Steps
Expected
Actual
Severity
Priority
Screenshot / Log
Request ID
Version

เช่น

BUG-1001

Login returns 500

Environment:
UAT

Version:
v1.8.2

RequestId:
abc-123

เชื่อมกับ #6 Logging และ #8 Monitoring

33. Severity vs Priority

สองคำนี้อย่าสับสน

Severity

ความรุนแรงของปัญหา

Critical
High
Medium
Low
Priority

ความเร่งด่วนในการแก้

P1
P2
P3

Bug อาจ

Severity = High
Priority = Low

ได้ ถ้าปัญหารุนแรงแต่เกิดใน Feature ที่แทบไม่มีคนใช้

34. Production Verification

หลัง Deploy Production ควรมี Test อีกครั้ง

Deploy
 ↓
Health Check
 ↓
Smoke Test
 ↓
Critical API Test
 ↓
Monitoring

ตัวอย่าง

Login          ✅
Dashboard      ✅
Create Order   ✅
Search         ✅
Report         ✅

นี่ช่วยจับปัญหาที่เกิดเฉพาะ Production Environment

35. Testing กับระบบที่เราเรียนมา

Testing จริง ๆ เชื่อมแทบทุกระบบ

Authentication
       ↓
Security Test

Database
       ↓
Integration / Concurrency Test

API
       ↓
API Test

Performance
       ↓
Load / Stress Test

Queue
       ↓
Retry / Duplicate / Failure Test

File
       ↓
Upload / Download / Security Test

Payment
       ↓
Idempotency / Webhook / Refund Test

Reporting
       ↓
Large Data / Export / Performance Test

Configuration
       ↓
Environment / Secret / Validation Test

Deployment
       ↓
Smoke / Rollback / Health Test
36. Mental Model 🧠
TESTING SYSTEM
│
├── Functional
│   ├── Unit Test
│   ├── Integration Test
│   ├── API Test
│   ├── E2E Test
│   └── Regression Test
│
├── Validation
│   ├── Positive Test
│   ├── Negative Test
│   ├── Boundary Test
│   └── Business Rule Test
│
├── Performance
│   ├── Load Test
│   ├── Stress Test
│   ├── Spike Test
│   ├── Endurance Test
│   └── Scalability Test
│
├── Security
│   ├── Authentication
│   ├── Authorization
│   ├── SQL Injection
│   ├── XSS
│   ├── CSRF
│   └── IDOR
│
├── Reliability
│   ├── Timeout
│   ├── Retry
│   ├── Failure
│   ├── Concurrency
│   └── Recovery
│
├── Test Infrastructure
│   ├── Test Data
│   ├── Test Environment
│   ├── Mock
│   └── Stub
│
├── Automation
│   ├── CI
│   ├── CD
│   └── Automated Test
│
├── UAT
│
├── Test Report
│
└── Defect Management
37. Deep Topics ที่ควรรู้ 🔥

ถ้าจะเรียน Testing แบบ Production จริง ๆ:

1. Unit Testing
2. Integration Testing
3. API Testing
4. E2E Testing
5. Test Pyramid
6. Mock / Stub
7. Test Data
8. Regression Testing
9. Smoke Testing
10. Functional Testing
11. Negative Testing
12. Boundary Testing
13. Contract Testing
14. Code Coverage
15. Load Testing
16. Stress Testing
17. Spike Testing
18. Endurance Testing
19. Scalability Testing
20. Concurrency Testing
21. Security Testing
22. Database Testing
23. UAT
24. Test Automation
25. CI/CD Testing
26. Bug / Defect Management
27. Production Verification
🔥 สิ่งที่คุณควรจำที่สุด

Testing ที่ดีไม่ได้ถามแค่

"มันทำงานไหม?"

แต่ต้องถามหลายระดับ:

                    TESTING
                       │
        ┌──────────┼──────────┐
        ↓              ↓              ↓
    ถูกต้องไหม?     รับ Load ไหม?    ปลอดภัยไหม?
        │             │             │
        ↓              ↓              ↓
   Functional      Performance      Security
        │             │              │
        └──────────┼──────────┘
                       ↓
                  Production
                       ↓
                ยังทำงานจริงไหม?

และสำหรับ Architecture ของคุณ:

Developer
   ↓
Git
   ↓
CI
   ├── Unit Test
   ├── Integration Test
   ├── Security Test
   └── Build
        ↓
Docker Image
        ↓
UAT
   ├── API Test
   ├── E2E Test
   ├── Load Test
   └── Smoke Test
        ↓
Production
        ↓
Health Check
        ↓
Monitoring
⭐ จุดที่ควรลงลึกเป็นพิเศษสำหรับสาย Backend ของคุณ
Unit Test
    ↓
Integration Test
    ↓
API Test
    ↓
Database Test
    ↓
Concurrency Test
    ↓
Load Test
    ↓
Stress Test
    ↓
Endurance Test
    ↓
CI/CD Automation

โดยเฉพาะ Concurrency Test + Load Test + Endurance Test เพราะมันจะทำให้เห็นปัญหาที่ Test แบบ