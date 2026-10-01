Payment / Billing System 💳

ระบบชำระเงินและเรียกเก็บเงิน

ควรแยก 2 คำนี้ออกจากกันก่อน:

Payment (เพย์เมนต์)

การรับ/จ่ายเงินจริง

ลูกค้า
 ↓
ชำระเงิน
 ↓
Payment Provider
 ↓
Payment Result
Billing (บิลลิง)

การคำนวณว่า ต้องจ่ายเท่าไร และเพราะอะไร

เช่น:

สินค้า        1,000
ค่าจัดส่ง       100
ส่วนลด         -50
VAT             73.50
--------------------
Total         1,123.50

ดังนั้น:

Billing = คำนวณยอด
Payment = จัดการการชำระเงิน
🧠 Architecture หลัก
Customer
   ↓
Order
   ↓
Billing
   ↓
Invoice
   ↓
Payment
   ↓
Payment Provider
   ↓
Webhook
   ↓
Payment Result
   ↓
Order Status

ตัวอย่าง:

Order #10001
     ↓
Total = 1,500
     ↓
Invoice
     ↓
Payment
     ↓
Provider
     ↓
SUCCESS
     ↓
Order = PAID
1. Pricing / Calculation 💰

ระบบต้องคำนวณราคาอย่างถูกต้อง

ตัวอย่าง:

Subtotal       1,000
Discount        -100
Shipping          50
VAT               66.50
---------------------
Total          1,016.50

ต้องระวัง:

ส่วนลด
ภาษี
ค่าจัดส่ง
ค่าธรรมเนียม
Currency (เคอร์เรนซี)
Decimal Precision (เดซิมัล พรีซิชัน)
Rounding (เรานดิง)

⚠️ เรื่องเงิน ไม่ควรใช้ Floating Point (โฟลททิง พอยต์) แบบที่อาจเกิดความคลาดเคลื่อน

เช่นแนวคิด:

0.1 + 0.2

ในคอมพิวเตอร์อาจไม่ได้แทนค่าเป็น 0.3 แบบตรง ๆ

ระบบการเงินจึงมักใช้ Decimal หรือเก็บเป็นหน่วยย่อย เช่น:

100.50 บาท
↓
10050 สตางค์
2. Invoice 🧾

Invoice (อินวอยซ์) คือเอกสาร/ข้อมูลที่ระบุว่า

ลูกค้าต้องชำระเงินเท่าไร และมาจากรายการอะไร

ตัวอย่าง:

Invoice
────────────────
INV-2026-00001

Product A       1,000
Shipping          100
Discount          -50
VAT                73.50
────────────────
Total           1,123.50

ข้อมูลสำคัญ:

InvoiceId
InvoiceNumber
CustomerId
OrderId
Subtotal
Discount
Tax
Total
Currency
Status
DueDate
CreatedAt
3. Payment Transaction 💳

อย่าใช้แค่:

Order.IsPaid = true

เพราะ Payment จริงมีรายละเอียดมากกว่านั้น

ควรมี Transaction:

Payment
├── PaymentId
├── OrderId
├── Amount
├── Currency
├── Provider
├── ProviderTransactionId
├── Status
├── CreatedAt
├── PaidAt
└── FailureReason

Status เช่น:

PENDING
PROCESSING
SUCCEEDED
FAILED
CANCELLED
REFUNDED
4. Payment State Machine 🔄

Payment ควรมี State (สเตต) ที่ชัดเจน

PENDING
   ↓
PROCESSING
   ↓
SUCCEEDED

หรือ:

PENDING
   ↓
PROCESSING
   ↓
FAILED

หรือ:

SUCCEEDED
   ↓
REFUND_REQUESTED
   ↓
REFUNDED

ไม่ควรให้ Code เปลี่ยนสถานะมั่ว ๆ

เช่น:

FAILED → SUCCEEDED

ต้องกำหนดว่าการเปลี่ยน State ไหนอนุญาตบ้าง

5. Payment Provider 🌐

ระบบจริงมักไม่ได้ประมวลผลบัตรเองทั้งหมด แต่เชื่อมกับ Payment Provider (เพย์เมนต์ โพรไวเดอร์)

แนวคิด:

Your API
   ↓
Payment Provider
   ↓
Bank / Card / Wallet

เช่นช่องทางอาจมี:

Credit Card
QR Payment
Bank Transfer
Wallet

แต่ละ Provider มี API และข้อกำหนดต่างกัน

จึงควรสร้าง Abstraction (แอบสแทรกชัน):

PaymentService
      ↓
PaymentProvider
   ├── Provider A
   ├── Provider B
   └── Provider C

ไม่ควรเอา Logic ของ Provider เจ้าเดียวไปปนกับ Business Logic ทั้งระบบ

6. Webhook 🔔

นี่คือเรื่องที่ ต้องเข้าใจให้ลึกมาก

อย่าพึ่งพาแค่:

POST /payment
 ↓
Provider Response
 ↓
SUCCESS

เพราะหลังจากนั้นอาจเกิด:

Network Timeout
API ล่ม
Provider ประมวลผลช้า
Browser ปิด
Response หาย

Provider จึงมักส่ง Webhook (เว็บฮุก) กลับมาที่ระบบเรา

Customer
 ↓
Payment
 ↓
Provider
 ↓
Webhook
 ↓
Your API
 ↓
Update Payment

เช่น:

POST /webhooks/payment
7. Webhook ต้อง Idempotent 🔐

Provider อาจส่ง Webhook เดิมซ้ำ

Webhook #1 → SUCCESS
Webhook #2 → SUCCESS
Webhook #3 → SUCCESS

ถ้าเราเขียนแบบไม่ระวัง อาจ:

เพิ่มยอดเงิน 3 ครั้ง

จึงต้องมี Event ID หรือ Transaction ID

ProviderEventId

แล้วตรวจสอบ:

เคย Process Event นี้หรือยัง?
        │
    ┌───┴───┐
    │       │
   Yes      No
    │       │
 Ignore   Process
8. Double Payment 🚨

ปัญหาที่ต้องป้องกัน:

User กด Pay
     ↓
Request 1
     ↓
Request 2

ถ้า API สร้าง Payment สองรายการ:

Payment 100 บาท
Payment 100 บาท

ลูกค้าอาจถูกเรียกเก็บ 200 บาท

ต้องใช้:

Idempotency Key

เช่น:

ORDER-10001-PAYMENT

Request ซ้ำด้วย Key เดิม:

Request 1 → Create Payment
Request 2 → Return Existing Payment
9. Transaction + Payment 💾

ต้องระวัง Transaction ระหว่าง SQL Server กับ External Payment Provider

เช่น:

BEGIN TRANSACTION

Create Payment
      ↓
Call Payment Provider
      ↓
Provider Success
      ↓
Commit

ดูเหมือนดี แต่มีปัญหา:

Database Transaction กำลังถือ Lock แล้วต้องรอ External API

ถ้า Provider ใช้เวลา 10 วินาที:

Transaction
   ↓
Lock
   ↓
Wait 10 sec

อาจทำให้:

Blocking
Connection Pool Starvation
Timeout

ดังนั้น อย่าถือ Database Transaction ค้างไว้ระหว่างรอ External Payment API โดยไม่จำเป็น

10. Payment Saga / State-Based Flow 🔄

แทนที่จะพยายามทำทุกอย่างใน Transaction เดียว:

DB + Payment Provider

สามารถออกแบบเป็น State:

Order
 ↓
PAYMENT_PENDING
 ↓
Call Provider
 ↓
Provider Success
 ↓
PAYMENT_SUCCEEDED
 ↓
Order = PAID

ถ้าล้ม:

PAYMENT_PENDING
 ↓
Provider Failed
 ↓
PAYMENT_FAILED

นี่เป็นแนวคิดสำคัญในระบบ Distributed System (ดิสทริบิวเต็ด ซิสเต็ม)

11. Refund 💸

ต้องรองรับการคืนเงิน

Payment
   ↓
Refund Request
   ↓
Provider
   ↓
Refund Success

สถานะ:

REFUND_PENDING
REFUNDED
REFUND_FAILED

ต้องเก็บ:

RefundId
PaymentId
Amount
Reason
ProviderRefundId
Status
CreatedAt
CompletedAt
12. Partial Refund 💰

ไม่จำเป็นต้องคืนเงินทั้งหมดเสมอไป

เช่น:

Payment = 1,000

คืน:

Refund = 300

เหลือ:

700

ต้องตรวจสอบว่า:

Total Refund
≤
Total Payment

ไม่ให้:

Payment = 1,000
Refund = 1,200 ❌
13. Payment Reconciliation 🔍

Reconciliation (เรคอนซิลิเอชัน) คือการตรวจสอบว่า

ระบบเรา กับ Payment Provider มีข้อมูลตรงกันหรือไม่

ตัวอย่าง:

ระบบเรา:

Payment #1001
Status = PENDING

Provider:

Payment #1001
Status = SUCCESS

ข้อมูลไม่ตรงกัน

ต้องมีระบบตรวจสอบ:

Our Database
      ↕
Payment Provider
      ↓
Reconciliation

อาจทำเป็น Scheduled Job:

ทุก 1 ชั่วโมง
 ↓
ค้นหา Payment ที่ PENDING
 ↓
ตรวจสอบ Provider
 ↓
Update Status
14. Payment Timeout ⏱️

Payment ที่ค้างนานเกินไปต้องจัดการ

เช่น:

PENDING
   ↓
5 minutes
   ↓
EXPIRED

ตัวอย่าง:

QR Payment
สร้าง 10:00
หมดอายุ 10:15

Worker สามารถตรวจสอบ:

Expired Payment
 ↓
Mark EXPIRED
15. Security 🔐

Payment System ต้องให้ความสำคัญกับ Security สูงมาก

ต้องป้องกัน:

SQL Injection
Authentication
Authorization
Replay Attack (รีเพลย์ แอทแทก)
Webhook Forgery
Request Tampering
Duplicate Payment
Secret Leakage

Webhook ควรตรวจสอบความถูกต้อง เช่น:

Webhook
 ↓
Verify Signature
 ↓
Verify Event
 ↓
Check Amount
 ↓
Check Order
 ↓
Process

ไม่ควรเชื่อแค่:

status = "success"

ที่ส่งมาจาก Client

16. Audit Log 📜

Payment ทุกการเปลี่ยนแปลงควรตรวจสอบย้อนหลังได้

Payment #10001

10:00 Created
10:01 Processing
10:02 Failed
10:03 Retry
10:04 Success
10:05 Refund Requested
10:06 Refunded

ต้องรู้ว่า:

Who
What
When
Before
After
Reason
RequestId
17. Billing Cycle 🔁

ถ้าเป็น Subscription (ซับสคริปชัน) จะมี Billing Cycle (รอบเรียกเก็บเงิน)

เช่น:

Monthly
Yearly
Weekly

ตัวอย่าง:

Plan = PRO
Price = 299/month

01 Oct
 ↓
Charge 299

01 Nov
 ↓
Charge 299

01 Dec
 ↓
Charge 299

ต้องจัดการ:

Next Billing Date
Failed Payment
Retry
Grace Period
Cancellation
Upgrade
Downgrade
18. Failed Payment 🔴

Payment ไม่สำเร็จไม่ได้แปลว่าต้องยกเลิกทันทีเสมอไป โดยเฉพาะ Subscription

อาจเป็น:

Payment Failed
      ↓
Retry
      ↓
Failed
      ↓
Retry Later
      ↓
Success

หรือ:

Payment Failed
      ↓
Grace Period
      ↓
Account Suspended
19. Money Ledger 📚

ถ้าเป็นระบบการเงินจริงจัง ควรเข้าใจ Ledger (เลดเจอร์)

แทนที่จะเก็บเพียง:

Balance = 1000

ระบบการเงินมักต้องมีรายการเคลื่อนไหว:

Transaction
----------------
+1000 Deposit
-300 Purchase
-100 Fee
+200 Refund
----------------
Balance = 800

ข้อดีคือสามารถตรวจสอบย้อนหลังได้ว่า:

เงิน 800 มาจากไหน?

🧠 Mental Model
PAYMENT / BILLING SYSTEM
│
├── Billing
│   ├── Pricing
│   ├── Discount
│   ├── Tax
│   ├── Fee
│   ├── Currency
│   └── Rounding
│
├── Invoice
│   ├── Invoice Number
│   ├── Items
│   ├── Total
│   └── Due Date
│
├── Payment
│   ├── Transaction
│   ├── Status
│   ├── Provider
│   └── Idempotency
│
├── Payment Provider
│   ├── Card
│   ├── QR
│   ├── Bank
│   └── Wallet
│
├── Webhook
│   ├── Signature Verification
│   ├── Idempotency
│   └── Event Processing
│
├── Refund
│   ├── Full Refund
│   └── Partial Refund
│
├── Reconciliation
│
├── Subscription
│   ├── Billing Cycle
│   ├── Retry
│   ├── Grace Period
│   └── Cancellation
│
├── Ledger
│
├── Security
│
├── Audit
│
└── Monitoring
    ├── Payment Success
    ├── Payment Failure
    ├── Processing Time
    ├── Webhook Failure
    └── Reconciliation Difference
🔥 สิ่งที่ควรเข้าใจให้ลึกที่สุด

สำหรับ Payment System ผมจะให้ความสำคัญตามนี้:

1. Money Calculation
2. Payment State Machine
3. Idempotency
4. Webhook
5. Double Payment Prevention
6. Transaction + External API
7. Retry
8. Refund
9. Reconciliation
10. Ledger
11. Security
12. Audit

และจำ Flow นี้ให้แม่น:

                 ┌──────────────┐
                 │       Order        │
                 └──────┬───────┘
                           ↓
                 ┌──────────────┐
                 │      Billing      │
                 └──────┬───────┘
                           ↓
                 ┌──────────────┐
                 │      Invoice       │
                 └──────┬───────┘
                           ↓
                 ┌──────────────┐
                 │       Payment     │
                 └──────┬───────┘
                           ↓
              ┌────────────────────┐
              │     Payment Provider      │
              └─────────┬──────────┘
                            │
                         Webhook
                            ↓
              ┌────────────────────┐
              │     Verify + Idempotent   │
              └─────────┬──────────┘
                            ↓
                     Update Payment
                            ↓
                      Update Order
                            ↓
                       Audit Log

⚠️ จุดสำคัญที่สุด

Payment System ต้องคิดต่างจาก CRUD (ซีอาร์ยูดี) ทั่วไป

CRUD:

Create
Read
Update
Delete

แต่ Payment ต้องคิดเรื่อง:

Money
+
State
+
Idempotency
+
Concurrency
+
External System
+
Retry
+
Audit
+
Reconciliation

เพราะฉะนั้นโจทย์สำคัญไม่ใช่แค่ "จ่ายเงินได้ไหม?"

แต่ต้องตอบให้ได้ว่า:

ถ้า Request ถูกส่ง 2 ครั้ง, Webhook ถูกส่ง 3 ครั้ง, Provider ตอบช้า, Server ล่มกลางทาง, DB Transaction Rollback หรือระบบเราไม่ตรงกับ Provider แล้วเงินของลูกค้าจะยังถูกต้องหรือไม่?

นี่คือหัวใจของ Payment / Billing System ครับ