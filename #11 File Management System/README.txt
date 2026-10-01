File Management System 📁

ระบบจัดการไฟล์

หน้าที่หลักคือจัดการวงจรชีวิตของไฟล์ตั้งแต่

Upload
  ↓
Validate
  ↓
Store
  ↓
Access
  ↓
Download
  ↓
Replace / Version
  ↓
Delete / Archive

ตัวอย่างไฟล์:

รูปภาพ
PDF
Excel
CSV
Video
Document
ZIP
ไฟล์ที่ User อัปโหลด
ไฟล์ที่ระบบ Generate ขึ้นมา
1. File Upload 📤

ระบบรับไฟล์จาก User

User
 ↓
POST /files
 ↓
API
 ↓
Validate
 ↓
Storage

ต้องตรวจสอบอย่างน้อย:

File Size (ขนาดไฟล์)
File Type (ประเภทไฟล์)
Extension (นามสกุล)
MIME Type (ไมม์ ไทป์)
File Name
Content
จำนวนไฟล์
User Permission

เช่น:

Allowed:
.pdf
.xlsx
.jpg
.png

แต่ อย่าเชื่อ Extension อย่างเดียว

เช่น:

virus.exe

อาจเปลี่ยนชื่อเป็น:

document.pdf

ดังนั้นต้องตรวจสอบ MIME Type และถ้าระบบมีความเสี่ยงสูง อาจต้องตรวจสอบเนื้อไฟล์จริงเพิ่มเติม

2. File Size Limit 📏

ต้องกำหนดขนาดสูงสุด

เช่น:

Profile Image → 5 MB
PDF            → 20 MB
Excel          → 50 MB
Video          → 500 MB

ต้องกำหนด Limit หลายชั้น เช่น:

Browser
 ↓
Reverse Proxy
 ↓
API
 ↓
Storage

ไม่ควรปล่อยให้ API รับไฟล์ขนาดไม่จำกัด เพราะอาจเกิด:

Large Upload
     ↓
Memory Usage ↑
     ↓
CPU ↑
     ↓
Disk ↑
     ↓
Server Slow
3. File Storage 💾

ไฟล์สามารถเก็บได้หลายรูปแบบ

File Storage
│
├── Local Disk
├── Network Storage
├── Object Storage
└── Cloud Storage
Local Disk

เช่น:

D:\Uploads

ข้อดี:

ง่าย
เร็ว
เหมาะกับระบบเล็ก/ภายใน

ข้อเสีย:

Server พัง → ไฟล์อาจหาย
Scale หลาย Server ยาก
Backup ต้องออกแบบเอง
Object Storage ☁️

เช่น:

Bucket
 ├── documents
 ├── images
 └── reports

แนวคิดนี้เหมาะกับระบบที่ต้องรองรับไฟล์จำนวนมากและหลาย Server

4. Database vs File Storage 🗄️

เรื่องนี้สำคัญมาก

โดยทั่วไปไม่ควรเอาไฟล์ขนาดใหญ่ยัดลง Database โดยตรง

❌

SQL Server
 └── PDF 50 MB
 └── PDF 100 MB
 └── Excel 30 MB

มักจะเก็บ:

Database
   ↓
File Metadata

Storage
   ↓
Actual File

เช่น Database:

FileId
FileName
StoragePath
MimeType
Size
Hash
UploadedBy
CreatedAt

ส่วนไฟล์จริงอยู่ใน Storage

Storage
└── files/
    └── 2026/
        └── 09/
            └── abc123.pdf
5. File Metadata 🏷️

Metadata (เมทาดาทา) คือข้อมูลเกี่ยวกับไฟล์

ตัวอย่าง:

FileId
OriginalName
StoredName
Extension
MimeType
Size
Hash
StoragePath
OwnerId
UploadedBy
CreatedAt
UpdatedAt
DeletedAt
Status

ตัวอย่าง:

FileId       = 10001
OriginalName = report.xlsx
StoredName   = 8f2a91c4.xlsx
MimeType     = application/xlsx
Size         = 2048000
UploadedBy   = 501
6. อย่าใช้ Original File Name เป็น Storage Name ⚠️

ไม่ควร:

/uploads/report.xlsx

เพราะอาจเกิด:

ชื่อซ้ำ
Path Traversal (พาธ ทราเวอร์ซัล)
Character แปลก ๆ
Unicode ปัญหา
การเดา URL ได้ง่าย

ควร Generate ชื่อใหม่ เช่น:

8f3d1c9a-2c21-4f2d.pdf

หรือใช้:

UUID
Hash
Random ID

ส่วนชื่อเดิมเก็บไว้ใน Metadata

7. Path Traversal 🛡️

ต้องป้องกันการที่ User พยายามส่ง Path เช่น:

../../../../etc/passwd

หรือบน Windows:

..\..\..\secret.txt

User ไม่ควรสามารถกำหนด Path ของ Storage เองได้

❌

/uploads/{userInput}

ควรให้ Server เป็นคนสร้าง Storage Path

Storage
 ↓
Generate Path
 ↓
Save File
8. File Access Control 🔐

แค่มี File ID ไม่ได้หมายความว่า User มีสิทธิ์อ่านไฟล์

เช่น:

GET /files/10001

ต้องตรวจสอบ:

User
 ↓
Authentication
 ↓
Authorization
 ↓
File Ownership / Permission
 ↓
Download

ตัวอย่าง:

User A
 └── File 10001 ✅

User B
 └── File 10001 ❌

ต้องป้องกัน IDOR (ไอดอร์) หรือการที่ User เปลี่ยน ID แล้วเข้าถึงข้อมูลของคนอื่น

9. Download 📥

มี 2 แนวทางหลัก

API ส่งไฟล์เอง
User
 ↓
API
 ↓
Storage
 ↓
API
 ↓
User

ข้อเสียคือ API ต้องรับภาระส่งไฟล์

Signed URL

API ตรวจ Permission ก่อน:

User
 ↓
API
 ↓
Check Permission
 ↓
Generate Signed URL
 ↓
User
 ↓
Storage

ข้อดี:

API
 ↓
Authorization

แล้วให้ Storage จัดการ Data Transfer

เหมาะกับไฟล์ขนาดใหญ่

10. File Upload แบบ Streaming 🌊

ไฟล์ใหญ่ไม่ควรโหลดทั้งหมดเข้า Memory

❌

500 MB File
 ↓
RAM
 ↓
Save

ถ้ามีหลาย User พร้อมกัน:

100 Users
×
500 MB
=
50 GB

อาจทำให้ Memory พุ่ง

ควรใช้ Streaming (สตรีมมิง):

Upload
 ↓
Stream
 ↓
Storage

ข้อมูลไหลไปเรื่อย ๆ โดยไม่ต้องเก็บไฟล์ทั้งหมดไว้ใน RAM

11. Chunked / Multipart Upload 🧩

สำหรับไฟล์ขนาดใหญ่มาก สามารถแบ่งเป็น Chunk (ชังก์)

เช่น:

1 GB File

Chunk 1 → 10 MB
Chunk 2 → 10 MB
Chunk 3 → 10 MB
...
Chunk 100 → 10 MB

ถ้า Chunk 53 ล้ม:

Chunk 53 ❌

ไม่จำเป็นต้อง Upload ใหม่ทั้งหมด

Retry Chunk 53

เหมาะกับ:

Video
Backup
Large Dataset
Large Archive
12. File Integrity 🔍

ต้องตรวจสอบว่าไฟล์เสียหรือถูกเปลี่ยนระหว่างทางหรือไม่

ใช้ Hash (แฮช) เช่น:

SHA-256

ตัวอย่าง:

Original
 ↓
SHA-256
 ↓
ABC123...

หลัง Upload:

Stored File
 ↓
SHA-256
 ↓
ABC123...

ถ้าตรงกัน:

✅ File Integrity OK
13. Virus / Malware Scanning 🦠

ถ้าระบบเปิดให้ User Upload ไฟล์ ต้องพิจารณา Malware Scanning (มัลแวร์ สแกนนิง)

Flow:

Upload
 ↓
Temporary Storage
 ↓
Virus Scan
 ↓
Safe?
 ├── YES → Permanent Storage
 └── NO  → Reject / Quarantine

โดยเฉพาะระบบที่รับ:

.exe
.zip
.docx
.xls
PDF

ต้องระวังมากขึ้น

14. File Status 📊

ไฟล์อาจมี Lifecycle (ไลฟ์ไซเคิล)

UPLOADING
    ↓
UPLOADED
    ↓
SCANNING
    ↓
AVAILABLE

ถ้าตรวจพบปัญหา:

SCANNING
    ↓
REJECTED

ถ้าถูกลบ:

AVAILABLE
    ↓
DELETED
15. Soft Delete 🗑️

แทนที่จะลบไฟล์ทันที

DELETE

อาจเปลี่ยนเป็น:

DeletedAt
DeletedBy
Status = DELETED

ทำให้สามารถ:

Restore

ได้

แต่ต้องระวังว่า Soft Delete ไม่ได้แปลว่าไฟล์ยังควรอยู่ตลอดไป

อาจมี:

Deleted
 ↓
Retention Period
 ↓
Permanent Delete
16. File Versioning 📚

ถ้า User Upload ไฟล์ชื่อเดิมหลายครั้ง:

report.xlsx

อาจต้องเก็บ Version (เวอร์ชัน)

report.xlsx
 ├── v1
 ├── v2
 ├── v3
 └── v4

Metadata:

FileId
DocumentId
Version
StoragePath
UploadedBy
CreatedAt

เหมาะกับ:

Contract
Document
SOP
Specification
Report
17. Retention Policy 🗑️

Retention Policy (รีเทนชัน พอลิซี) คือกฎว่าไฟล์ควรเก็บไว้นานเท่าไร

เช่น:

Temporary File → 1 day
Report          → 30 days
Audit Document  → 7 years

Worker สามารถทำงาน:

Every Night
 ↓
Find Expired Files
 ↓
Delete
 ↓
Record Audit

ช่วยไม่ให้ Storage โตไม่สิ้นสุด

18. Duplicate File Detection ♻️

ไฟล์เดียวกันอาจถูก Upload หลายครั้ง

เช่น:

report.pdf
report-copy.pdf
report-final.pdf
report-final2.pdf

แต่เนื้อหาเหมือนกัน

สามารถใช้:

SHA-256 Hash

ตรวจสอบ

File A → ABC123
File B → ABC123

แสดงว่า Content เหมือนกัน

อาจเลือก:

Reuse Existing File

เพื่อลด Storage

19. Image Processing 🖼️

ถ้าระบบรับรูปภาพ อาจมี Processing Pipeline:

Upload
 ↓
Validate
 ↓
Virus Scan
 ↓
Resize
 ↓
Generate Thumbnail
 ↓
Compress
 ↓
Store

เช่น:

Original
4000 × 3000

สร้าง:

Thumbnail
300 × 225

Medium
1200 × 900

ทำให้ Frontend ไม่ต้องโหลดรูป Original ขนาดใหญ่ตลอดเวลา

20. Queue + File System ⚙️

ตรงนี้เชื่อมกับ #10 Queue / Background Job System โดยตรง

เช่น User Upload Excel 100,000 rows:

Upload
 ↓
Store File
 ↓
Create Job
 ↓
Queue
 ↓
Worker
 ↓
Read Excel
 ↓
Validate
 ↓
Batch Insert
 ↓
Complete

API ไม่ต้องรอ Import ทั้งหมด

🧠 Mental Model
FILE MANAGEMENT SYSTEM
│
├── Upload
│   ├── Validation
│   ├── Size Limit
│   ├── MIME Check
│   └── Streaming
│
├── Storage
│   ├── Local Disk
│   ├── Network Storage
│   └── Object Storage
│
├── Metadata
│   ├── File ID
│   ├── File Name
│   ├── MIME Type
│   ├── Size
│   ├── Hash
│   └── Storage Path
│
├── Security
│   ├── Authentication
│   ├── Authorization
│   ├── IDOR Prevention
│   ├── Path Traversal Prevention
│   ├── Malware Scan
│   └── Access Control
│
├── Download
│   ├── API Download
│   └── Signed URL
│
├── Large File
│   ├── Streaming
│   └── Chunked Upload
│
├── Lifecycle
│   ├── Upload
│   ├── Available
│   ├── Version
│   ├── Soft Delete
│   ├── Archive
│   └── Permanent Delete
│
├── Processing
│   ├── Resize
│   ├── Compress
│   ├── Thumbnail
│   └── File Import
│
└── Monitoring
    ├── Storage Usage
    ├── Upload Failure
    ├── Download Count
    ├── Processing Time
    └── Failed Jobs
🔥 สิ่งที่ควรเข้าใจให้ลึก

สำหรับระบบ Production ผมให้ความสำคัญกับ 9 เรื่อง:

1. File Upload Security
2. File Storage Architecture
3. Database Metadata vs Actual File
4. Authentication + Authorization
5. Streaming
6. Large File / Chunked Upload
7. File Integrity / Hash
8. File Lifecycle + Retention
9. Queue + Background Processing

และต้องจำ Architecture (อาร์คิเทกเชอร์) นี้:

                User
                  │
                  ▼
                API
                  │
          ┌───────┴────────┐
          │                │
          ▼                ▼
      SQL Server        File Storage
          │                │
      Metadata          Actual File
          │                │
          └───────┬────────┘
                  │
                  ▼
                Queue
                  │
                  ▼
               Worker
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
     Scan      Resize     Import
⚠️ จุดที่มักพลาด

อย่าออกแบบแบบนี้

API
 ↓
รับไฟล์ 1 GB
 ↓
โหลดเข้า RAM
 ↓
เก็บ Binary ลง SQL Server
 ↓
ส่งไฟล์กลับผ่าน API

แต่ควรคิดเป็น:

                API
                 │
       Validate + Permission
                 │
                 ▼
           File Storage
                 │
                 ▼
             Metadata
             SQL Server
                 │
                 ▼
               Queue
                 │
                 ▼
              Worker
        ┌────────┼────────┐
        ▼        ▼        ▼
      Scan     Resize    Import

แบบนี้จะแยก Database, Storage, API และ Background Processing ออกจากกัน ทำให้รองรับไฟล์ใหญ่และโหลดสูงได้ง่ายกว่ามากครับ.