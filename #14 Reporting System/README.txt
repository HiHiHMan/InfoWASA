Reporting System 📊
1. Reporting System คืออะไร?

Reporting System (รีพอร์ตทิง ซิสเท็ม) คือระบบที่ใช้ดึงข้อมูลจาก Database มาประมวลผลและแสดงเป็นรายงาน เช่น

รายงานยอดขาย
รายงาน Stock
รายงานการตรวจสอบ
รายงานประจำวัน
รายงาน KPI
รายงาน Error
รายงานการทำงานของพนักงาน
Excel Export
PDF Report
Dashboard

ตัวอย่างง่าย ๆ

User
 ↓
Report API
 ↓
Query / Aggregation
 ↓
Database
 ↓
Result
 ↓
Table / Chart / Excel / PDF

แต่ถ้าเป็นระบบใหญ่ ไม่ควรให้ Report ยิง Database หลักหนัก ๆ โดยตรงทุกครั้ง

2. Core Components
REPORTING SYSTEM
├── Report Definition
├── Report API
├── Report Query
├── Filter
├── Sorting
├── Aggregation
├── Pagination
├── Export
│   ├── Excel
│   ├── CSV
│   └── PDF
├── Report Scheduling
├── Background Job
├── Queue
├── Report Storage
├── Cache
├── Permission
├── Audit
├── Logging
└── Monitoring
3. Report Definition

ต้องกำหนดว่า Report แต่ละตัวคืออะไร

เช่น

Report: Sales Daily Report

Parameters:
- StartDate
- EndDate
- Branch
- Product
- Employee

Columns:
- Date
- Branch
- Product
- Quantity
- Amount

ไม่ควรเขียน Report แบบกระจายไปทั่ว Code

ควรมีแนวคิด

Report
 ├── ReportId
 ├── ReportName
 ├── Description
 ├── Query
 ├── Parameters
 ├── Permissions
 └── Export Types
4. Report Query

นี่คือส่วนที่สำคัญที่สุดส่วนหนึ่ง

ตัวอย่าง

SELECT
    OrderDate,
    COUNT(*) AS OrderCount,
    SUM(TotalAmount) AS TotalAmount
FROM Orders
WHERE OrderDate >= @StartDate
  AND OrderDate < @EndDate
GROUP BY OrderDate
ORDER BY OrderDate;

ปัญหาคือ Report มักมี

JOIN
JOIN
JOIN
GROUP BY
SUM
COUNT
ORDER BY
SUBQUERY

พร้อมกัน

ดังนั้น Report Query มีโอกาสหนักกว่าการ Query API ปกติมาก

5. Filtering

Report มักต้องให้ User เลือกเงื่อนไข

เช่น

Date:
01/09/2026 - 30/09/2026

Branch:
Bangkok

Department:
IT

Status:
Completed

แล้ว API ส่ง

{
  "startDate": "2026-09-01",
  "endDate": "2026-09-30",
  "branch": "BKK",
  "status": "COMPLETED"
}

Database จึง Query ตาม Filter

จุดสำคัญ

ต้องป้องกัน

User ขอข้อมูลทั้ง Database

เช่น

SELECT * FROM Orders

หลายล้าน Record

เพราะจะทำให้

DB CPU ↑
DB I/O ↑
Connection ถูกใช้นาน
Connection Pool ↓
API ช้าลง
Request Timeout
6. Aggregation

Report ส่วนใหญ่ไม่ได้ต้องการข้อมูลดิบ

แต่ต้องการ Aggregation (แอกกริเกชัน)

เช่น

COUNT()
SUM()
AVG()
MIN()
MAX()
GROUP BY

ตัวอย่าง

SELECT
    Department,
    COUNT(*) AS EmployeeCount,
    AVG(Salary) AS AverageSalary
FROM Employee
GROUP BY Department;

ผลลัพธ์

IT        120
HR         45
Finance    60
Warehouse 300
7. Pagination

ถ้า Report มีข้อมูล 10 ล้านแถว

อย่าดึงทั้งหมดทีเดียว

10,000,000 rows
       ↓
API
       ↓
RAM 💥

ควรใช้

Page
Limit
Cursor

เช่น

Page 1 → 100 rows
Page 2 → 100 rows
Page 3 → 100 rows

แต่สำหรับ Report ที่ต้อง Export ทั้งหมด อาจไม่สามารถใช้ Pagination แบบ UI ได้ จึงต้องเปลี่ยนเป็น Background Job

8. Excel Export 📗

นี่เป็นจุดที่ระบบ Enterprise เจอบ่อยมาก

User กด

Export Excel

แล้วมีข้อมูล

1,000,000 rows

ถ้า API ทำแบบนี้

Request
 ↓
Query 1M rows
 ↓
Generate Excel
 ↓
Return File

จะมีปัญหา

CPU ↑
RAM ↑
Request ใช้นาน
Connection ถูกถือไว้นาน
Timeout

ดังนั้นควรใช้

User
 ↓
POST /reports/export
 ↓
Create Job
 ↓
Queue
 ↓
Worker
 ↓
Query Database
 ↓
Generate Excel
 ↓
Save File
 ↓
Update Job = COMPLETED

แล้ว User ได้

{
  "jobId": "REPORT-10001",
  "status": "PROCESSING"
}

จากนั้น Frontend เช็กสถานะ

PROCESSING
     ↓
COMPLETED
     ↓
Download

นี่เชื่อมกับ #10 Queue / Background Job System โดยตรง

9. PDF Report

PDF มีขั้นตอนเพิ่มขึ้น

Database
 ↓
Report Data
 ↓
Template
 ↓
PDF Generator
 ↓
PDF File

ต้องระวัง Report ที่มีข้อมูลเยอะมาก เพราะ PDF generation อาจใช้ CPU/RAM สูง

ดังนั้น PDF ขนาดใหญ่ก็เหมาะกับ

Queue
+
Worker
10. Report Scheduling ⏰

บาง Report ไม่ได้ให้ User กดเอง

แต่ต้องสร้างอัตโนมัติ

เช่น

ทุกวัน 08:00
→ Daily Sales Report

ทุกวันจันทร์
→ Weekly Report

วันที่ 1 ของเดือน
→ Monthly Report

Architecture

Scheduler
 ↓
Create Job
 ↓
Queue
 ↓
Worker
 ↓
Generate Report
 ↓
Storage
 ↓
Email / Notification

ตรงนี้เชื่อมกับ

#9 Notification System

11. Report Storage

หลัง Generate Report แล้ว ไม่จำเป็นต้องเก็บไฟล์ไว้ใน RAM

ควรเก็บใน Storage

Report
 ↓
Object Storage / File Storage

Database เก็บ Metadata เช่น

ReportId
ReportName
FileName
FilePath
FileType
FileSize
CreatedBy
CreatedAt
ExpiredAt
Status

เช่น

ReportId: RPT-10001
File: sales_2026_09.xlsx
Size: 25 MB
Status: COMPLETED
ExpiredAt: 2026-10-30
12. Report Cache ⚡

Report บางตัวถูกเรียกซ้ำบ่อยมาก

เช่น

Dashboard
Today's Sales
Monthly Summary
KPI

แทนที่จะ Query Database ทุกครั้ง

User
 ↓
Redis
 ↓
มีข้อมูล → Return

ถ้าไม่มี

User
 ↓
Redis
 ↓
MISS
 ↓
Database
 ↓
Cache
 ↓
Return

เชื่อมกับ #5 Performance System

13. Permission 🔐

Report ต้องมี Authorization เช่นกัน

สมมติ

Admin
→ ดูทุกสาขา

Manager
→ ดูเฉพาะสาขาตัวเอง

Employee
→ ดูเฉพาะข้อมูลของตัวเอง

อย่าคิดว่า

User เข้า Report ได้ = เห็นข้อมูลทั้งหมดได้

ต้องแยก

Can View Report
+
Data Scope

ตัวอย่าง

Report Permission
        ↓
Sales Report
        ↓
User = Branch A
        ↓
WHERE BranchId = 'A'

นี่เชื่อมกับ #1 Authentication & Authorization

14. Report Security

Report มีข้อมูลจำนวนมาก จึงต้องระวังเป็นพิเศษ

SQL Injection

อย่าสร้าง SQL แบบ

"SELECT * FROM Orders WHERE Branch = '" + userInput

ใช้ Parameter

WHERE Branch = @Branch
Data Leakage

เช่น

Employee A

สามารถเปลี่ยน

branchId=999

แล้วเห็นข้อมูล Branch อื่น

ต้องตรวจ Authorization ที่ Server

15. Report Versioning

Report เปลี่ยนบ่อย

เช่น

Sales Report v1
Sales Report v2
Sales Report v3

เพราะ Business อาจเปลี่ยน

สูตร
Column
VAT
Discount
KPI

จึงควรระวังไม่ให้แก้ Query แล้ว Report เก่าย้อนหลังเปลี่ยนความหมายโดยไม่ตั้งใจ

16. Snapshot Report

บาง Report ต้องการข้อมูล ณ เวลาหนึ่ง

เช่น

Monthly Stock Report
September 30, 2026

ถ้าวันนี้ข้อมูลใน Database เปลี่ยน

Report ย้อนหลังอาจเปลี่ยนตาม

ดังนั้นบางระบบต้องสร้าง

Snapshot (สแนปช็อต)

30 Sep
 ↓
Generate Report
 ↓
Save Result

ผลคือ

September Report

ยังคงเป็นข้อมูลของวันที่ 30 Sep

17. Report Data Source

ระบบเล็ก

API
 ↓
SQL Server

ระบบใหญ่สามารถเป็น

Application DB
       ↓
Read Replica
       ↓
Reporting DB
       ↓
Data Warehouse
       ↓
BI / Reporting

แนวคิดสำคัญคือ

อย่าให้ Report หนัก ๆ ไปฆ่า Production Database

18. Reporting Database

ถ้า Report มี Query หนักมาก อาจแยก Database

Production DB
      │
      │ Sync
      ↓
Reporting DB
      │
      ↓
Report

Production ใช้สำหรับ

INSERT
UPDATE
DELETE
Transaction

Reporting DB ใช้สำหรับ

SELECT
GROUP BY
Aggregation
Analysis

ช่วยแยก Workload

19. Data Warehouse 🏢

เมื่อข้อมูลเยอะและต้องวิเคราะห์หลายปี อาจมี

Data Warehouse (ดาต้าแวร์เฮาส์)

Application DB
      ↓
ETL / ELT
      ↓
Data Warehouse
      ↓
Reporting / BI

เช่น

FactSales
FactInventory
FactOrders

และ

DimDate
DimProduct
DimBranch
DimEmployee

แนวคิดนี้จะเริ่มเข้าสู่ Data Engineering / BI

20. Report Job Status

สำหรับ Report แบบ Async ควรมีสถานะ

PENDING
   ↓
PROCESSING
   ↓
COMPLETED

ถ้าเกิดปัญหา

PROCESSING
   ↓
FAILED
   ↓
RETRY

หรือ

FAILED
   ↓
DEAD LETTER

เชื่อมกับ #10 Queue System และ #7 Error Handling

21. Report Monitoring 📈

ควร Monitor เช่น

Report Count
Report Duration
Report Failure Rate
Export Count
Export Duration
Queue Length
Worker Count
Database CPU
Database Query Duration
File Size
Storage Usage

ตัวอย่าง

Sales Report
Average: 2 sec

Inventory Report
Average: 15 sec ⚠️

Monthly Report
Average: 180 sec 🚨

ทำให้รู้ว่า Report ตัวไหนกำลังสร้างปัญหา

22. Logging

ควร Log

ReportId
ReportName
UserId
RequestId
StartTime
EndTime
Duration
Filter
RowCount
FileSize
Status
Error

เช่น

Report: Inventory
User: 10025
Rows: 850,000
Duration: 72 sec
Status: FAILED
Error: Timeout

แต่ต้องระวังไม่ Log ข้อมูล Sensitive โดยไม่จำเป็น

เชื่อมกับ #6 Logging & Audit System

23. Audit

Report บางประเภทต้องรู้ว่าใครเปิดข้อมูล

เช่น

User A
→ Export Salary Report
→ 10:32
→ 850,000 rows

Audit สามารถเก็บ

UserId
ReportId
Action
Filter
CreatedAt
IPAddress
RequestId

โดยเฉพาะ Report ที่มี

เงินเดือน
ข้อมูลลูกค้า
ข้อมูลส่วนบุคคล
ข้อมูลการเงิน
ข้อมูลภายในบริษัท
24. ปัญหาที่เจอบ่อยมาก ⚠️
❌ 1. Report Query หนัก
JOIN 10 tables
+
GROUP BY
+
ORDER BY
+
10M rows

→ Database หนัก

❌ 2. Export Excel ใน API
API
 ↓
Query 1M rows
 ↓
Generate Excel

→ Timeout / RAM สูง

❌ 3. Report ใช้ Production DB
Report Query
 ↓
Production DB 💥

กระทบ Transaction ของระบบหลัก

❌ 4. ไม่มี Limit

User เลือก

01/01/2010 → 30/09/2026

แล้ว Query ทุกอย่าง

❌ 5. ไม่มี Index

Report Query

WHERE
JOIN
GROUP BY
ORDER BY

แต่ไม่มี Index ที่เหมาะสม

❌ 6. Worker เยอะเกินไป
100 Workers
 ↓
100 DB Connections
 ↓
SQL Server
 ↓
CPU 100%
 ↓
Blocking
 ↓
Timeout

นี่เชื่อมกับ

Connection Pool + Performance + Database + Queue

25. Mental Model 🧠
REPORTING SYSTEM
│
├── Report
│   ├── Definition
│   ├── Parameters
│   ├── Query
│   └── Version
│
├── Data
│   ├── Production DB
│   ├── Reporting DB
│   └── Data Warehouse
│
├── Query
│   ├── Filter
│   ├── Join
│   ├── Aggregation
│   ├── Sorting
│   └── Pagination
│
├── Export
│   ├── Excel
│   ├── CSV
│   └── PDF
│
├── Background Processing
│   ├── Queue
│   ├── Worker
│   ├── Retry
│   └── Job Status
│
├── Storage
│   ├── File
│   ├── Metadata
│   └── Retention
│
├── Scheduling
│   ├── Daily
│   ├── Weekly
│   └── Monthly
│
├── Performance
│   ├── Index
│   ├── Query Optimization
│   ├── Cache
│   └── Read Database
│
├── Security
│   ├── Authentication
│   ├── Authorization
│   ├── Data Scope
│   └── Sensitive Data
│
├── Audit
│
├── Logging
│
└── Monitoring
26. Deep Topics ที่ควรรู้ 🔥

ถ้าจะเข้าใจ Reporting System แบบ Production จริง ๆ ฉันแนะนำให้ลงลึกตามนี้

1. SQL Query Optimization
2. Index
3. Execution Plan
4. JOIN Optimization
5. Aggregation
6. Pagination
7. Large Data Processing
8. Streaming
9. Excel Export
10. Background Job
11. Queue
12. Worker Concurrency
13. Report Scheduling
14. Reporting Database
15. Data Warehouse
16. Cache
17. Authorization / Data Scope
18. Snapshot
19. Report Versioning
20. Monitoring
21. Audit
22. Retention
🔥 จุดที่สำคัญที่สุด

Reporting System ไม่ใช่แค่

"เอาข้อมูลจาก SQL มาแสดงเป็นตาราง"

แต่ควรมองเป็น

DATA
 ↓
QUERY
 ↓
AGGREGATION
 ↓
PERFORMANCE
 ↓
SECURITY
 ↓
BACKGROUND PROCESSING
 ↓
FILE GENERATION
 ↓
STORAGE
 ↓
MONITORING

และสำหรับ Architecture ที่คุณกำลังเรียนอยู่ จุดที่ควรจำมากที่สุดคือ:

User
 ↓
Next.js
 ↓
Go API
 ↓
Report Service
 ↓
 ┌───────────────┐
 │                     │
Simple Report     Large Report
 │                     │
 ↓                      ↓
SQL Server            Queue
                        ↓
                      Worker
                        ↓
                   Reporting DB
                        ↓
                    Excel/PDF
                        ↓
                     Storage

กฎทอง:

Report ที่ใช้เวลานาน อย่าผูกชีวิตมันไว้กับ HTTP Request และอย่าให้ Report หนัก ๆ ไปแย่ง Resource กับ Transaction ของระบบหลัก

#14 Reporting System จึงเชื่อมกับระบบที่เรียนมาก่อนแทบทั้งหมด โดยเฉพาะ Database → Performance → Queue → File → Error Handling → Logging → Monitoring → Authorization.