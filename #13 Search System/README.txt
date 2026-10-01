Search System 🔎

ระบบค้นหาข้อมูล

หน้าที่หลักคือรับคำค้นจาก User แล้วค้นหาข้อมูลที่เกี่ยวข้องอย่างรวดเร็ว

User
 ↓
Search Query
 ↓
Search System
 ↓
Search Engine / Database
 ↓
Ranking
 ↓
Results

ตัวอย่าง:

ค้นหา: "iphone 17"

ผลลัพธ์
├── iPhone 17 Pro
├── iPhone 17
├── iPhone 17 Air
└── iPhone 17 Case
1. Search Query 🔎

Search Query (เสิร์ช คิวรี) คือคำที่ User ใช้ค้นหา

เช่น:

"iphone 17"
"สายชาร์จ type c"
"ABC-001"
"อรรถพล"

ระบบต้องจัดการ:

Empty Query
Query Length
Special Characters
Typo
Case Sensitivity
ภาษาไทย
ภาษาอังกฤษ
ตัวเลข
หลายคำ
Exact Match
2. Basic Search — SQL LIKE

ระบบเล็ก ๆ อาจเริ่มด้วย:

SELECT *
FROM Products
WHERE ProductName LIKE '%iphone%';

ข้อดี:

ง่าย
ไม่ต้องมี Search Engine
เหมาะกับข้อมูลไม่ใหญ่มาก

แต่เมื่อข้อมูลเพิ่มขึ้น:

1,000 rows
   ↓
100,000 rows
   ↓
10,000,000 rows

การ:

LIKE '%keyword%'

อาจทำให้ Database ต้อง Scan ข้อมูลจำนวนมาก

3. Database Index 🔍

ถ้าค้นหาแบบ:

WHERE ProductCode = 'ABC001'

สามารถใช้ Index (อินเด็กซ์)

Index
 ↓
ABC001
 ↓
Row

จึงเร็วมากเมื่อออกแบบถูกต้อง

แต่ต้องแยกให้ออกว่า:

Exact Search

กับ

Text Search

ไม่เหมือนกัน

4. Full-Text Search 📚

Full-Text Search (ฟูลเท็กซ์ เสิร์ช) ถูกออกแบบมาสำหรับค้นหาข้อความ

แทนที่จะคิดแบบ:

LIKE '%database%'

ระบบจะสร้างโครงสร้างสำหรับค้นหาคำ

แนวคิด:

Documents
 ↓
Tokenize
 ↓
Index
 ↓
Search

ตัวอย่าง:

Document:

"SQL Server database performance"

ระบบแยกเป็น:

SQL
Server
database
performance

แล้วสร้าง Search Index

5. Search Engine 🚀

เมื่อระบบใหญ่ขึ้น อาจใช้ Search Engine (เสิร์ช เอนจิน) โดยเฉพาะ

Architecture:

Database
    ↓
Sync
    ↓
Search Index
    ↓
Search Engine
    ↓
Results

ตัวอย่าง Search Engine ที่พบได้บ่อย:

Elasticsearch (อิลาสติคเสิร์ช)
OpenSearch (โอเพนเสิร์ช)
Solr (โซลร์)

ข้อดี:

Full-text Search
Fuzzy Search
Ranking
Filtering
Faceted Search
Highlighting
Autocomplete
6. Search Index 📑

Search Index (เสิร์ช อินเด็กซ์) คือโครงสร้างข้อมูลที่สร้างขึ้นเพื่อให้ค้นหาเร็ว

เช่น:

Product
│
├── Name
├── Description
├── Category
└── Tags

Search Index:

iphone
 ├── Product 101
 ├── Product 205
 └── Product 800

samsung
 ├── Product 102
 └── Product 301

เวลาค้นหา:

iphone
 ↓
Index
 ↓
101,205,800

ไม่จำเป็นต้อง Scan ทุก Product

7. Tokenization ✂️

Tokenization (โทเคนไนเซชัน) คือการแยกข้อความออกเป็นหน่วยสำหรับ Search

ภาษาอังกฤษ:

"database performance"
       ↓
database
performance

แต่ภาษาไทยซับซ้อนกว่า:

"ระบบจัดการฐานข้อมูล"

เพราะภาษาไทยไม่ได้เว้นวรรคระหว่างคำเหมือนภาษาอังกฤษเสมอ

ดังนั้น Thai Search ต้องให้ความสำคัญกับ:

Word Segmentation (เวิร์ด เซกเมนเทชัน)
Thai Dictionary
Tokenization
Search Analyzer
8. Normalization 🧹

Normalization (นอร์มัลไลเซชัน) คือการทำข้อมูลให้รูปแบบสม่ำเสมอก่อนค้นหา

เช่น:

iPhone
IPHONE
iphone

อาจต้องทำให้เป็น:

iphone

หรือจัดการ:

Whitespace
Case
Unicode
Special Characters

เพื่อให้ Search ทำงานสม่ำเสมอ

9. Fuzzy Search 🧠

Fuzzy Search (ฟัซซี เสิร์ช) ช่วยค้นหาแม้ User พิมพ์ผิด

เช่น:

User:
iphnoe

แต่ระบบสามารถหา:

iphone

ได้

เหมาะกับ:

Product Search
Customer Search
Employee Search
Document Search
10. Autocomplete ⚡

เมื่อ User พิมพ์:

iph

ระบบแนะนำ:

iphone
iphone 17
iphone 17 pro
iphone case

Flow:

User Typing
 ↓
Autocomplete API
 ↓
Cache / Search Index
 ↓
Suggestions

ต้องระวังจำนวน Request

เพราะ User พิมพ์:

i
ip
iph
ipho
iphon
iphone

อาจกลายเป็น 6 Requests

ถ้ามี User 10,000 คนพร้อมกัน:

10,000 × 6
=
60,000 requests

จึงควรใช้:

Debounce (ดีบาวซ์)
Cache
Rate Limiting
Lightweight Query
11. Ranking 🎯

Search ไม่ใช่แค่:

"เจอหรือไม่เจอ"

แต่ต้องตอบ:

"ผลลัพธ์ไหนควรอยู่ก่อน?"

เช่นค้นหา:

iphone

มี 100,000 รายการ

ระบบต้องเรียง:

1. iPhone 17 Pro
2. iPhone 17
3. iPhone 16 Pro
4. iPhone Case
...

Ranking (แรงกิง) อาจพิจารณา:

Text Relevance
+
Popularity
+
Freshness
+
Sales
+
User Behavior
12. Relevance 🧠

Relevance (เรเลเวินซ์) คือระดับความเกี่ยวข้องของผลลัพธ์กับคำค้น

เช่น User ค้น:

"red shoes"

ผล:

Red Running Shoes

ควรมีคะแนนสูงกว่า:

Blue Shirt

ระบบ Search Engine อาจใช้ Algorithm (อัลกอริทึม) เช่น:

TF-IDF
BM25
Vector Similarity

สำหรับ Keyword Search ทั่วไป BM25 เป็นแนวคิดที่ควรรู้จัก

13. Filter 🔧

Search มักมาคู่กับ Filter (ฟิลเตอร์)

เช่น:

Search: iPhone

Filter
├── Price < 30,000
├── Brand = Apple
├── Storage = 256GB
└── Color = Black

Flow:

Query
 ↓
Search
 ↓
Filter
 ↓
Ranking
 ↓
Results
14. Faceted Search 📊

Faceted Search (ฟาซิเต็ด เสิร์ช) คือการแสดงตัวเลือก Filter พร้อมจำนวนผลลัพธ์

เช่น:

Apple       (120)
Samsung      (80)
Google       (30)

หรือ:

0-10,000      (50)
10,000-20,000 (80)
20,000+       (30)

ช่วยให้ User ค่อย ๆ จำกัดผลลัพธ์

15. Pagination 📄

Search Result อาจมีจำนวนมหาศาล

ไม่ควร:

SELECT 10,000,000 rows

ควร:

Page 1 → 20
Page 2 → 20
Page 3 → 20

หรือใช้ Cursor / Search After สำหรับระบบขนาดใหญ่

16. Search Cache ⚡

คำค้นยอดนิยมสามารถ Cache (แคช)

เช่น:

"iphone"

ค้นซ้ำหลายพันครั้ง

แทนที่จะ:

ทุก Request
 ↓
Search Engine

ใช้:

Request
 ↓
Redis
 ↓
Cache Hit
 ↓
Result

ช่วยลด Search Engine Load

17. Search Index Synchronization 🔄

นี่เป็นเรื่องสำคัญมาก

Database:

Product Name
=
iPhone 17

แต่ Search Index ยังเป็น:

iPhone 16

เกิดข้อมูลไม่ตรงกัน

จึงต้องมีระบบ Sync

Database
 ↓
Event
 ↓
Queue
 ↓
Worker
 ↓
Search Index

เช่น:

Product Updated
      ↓
Queue
      ↓
Worker
      ↓
Update Search Index

ตรงนี้เชื่อมกับ #10 Queue / Background Job โดยตรง

18. Eventual Consistency 🔄

Search Index อาจไม่ได้ Update พร้อม Database 100%

เช่น:

10:00:00
DB Updated

10:00:01
Queue

10:00:02
Search Index Updated

ช่วง 1–2 วินาที:

Database = ใหม่
Search = เก่า

เรียกว่า Eventual Consistency (อีเวนชวล คอนซิสเทนซี)

ต้องออกแบบให้ระบบยอมรับได้ว่าข้อมูล Search อาจล่าช้าเล็กน้อย

19. Search Security 🔐

Search ก็ต้องมี Authorization (ออธอไรเซชัน)

สมมติ:

Employee A

มีสิทธิ์เห็น:

Document A
Document B

แต่:

Document C

เป็นข้อมูลลับ

Search ต้องไม่ส่ง Document C ออกมา

⚠️ ไม่ใช่แค่หน้า Detail ที่ต้อง Check Permission

Search Result เองก็ต้อง Filter สิทธิ์ด้วย

20. Search Injection / Query Safety 🛡️

Search Query จาก User เป็น Input (อินพุต)

ต้อง Validate และป้องกัน:

Very Long Query
Special Characters
Malformed Query
Query Injection

ถ้า Search Engine มี Query Language ของตัวเอง ต้องไม่เอา User Input ไปประกอบ Query แบบไม่ปลอดภัย

21. Search Logging 📜

ควรเก็บข้อมูลเช่น:

Search Query
User
Result Count
Duration
Timestamp

เช่น:

Query = "iphone"
Results = 1200
Duration = 85ms

ข้อมูลนี้ช่วยดูว่า:

User ค้นหาอะไร
คำค้นไหนนิยม
คำค้นไหนไม่มีผลลัพธ์
Search ช้าเมื่อไร
22. Zero Result Analysis 🔎

คำค้นที่ไม่มีผลลัพธ์มีประโยชน์มาก

เช่น:

"iphone 18"
Result = 0

สามารถนำไปวิเคราะห์ว่า:

User ต้องการอะไร แต่ระบบยังไม่มีข้อมูล?

ช่วยพัฒนา Search และ Catalog ได้

🧠 Mental Model
SEARCH SYSTEM
│
├── Query
│   ├── Validation
│   ├── Normalization
│   └── Tokenization
│
├── Search Engine
│   ├── Database Search
│   ├── Full-Text Search
│   └── Search Index
│
├── Matching
│   ├── Exact Match
│   ├── Partial Match
│   └── Fuzzy Search
│
├── Ranking
│   ├── Relevance
│   ├── BM25
│   ├── Popularity
│   └── Freshness
│
├── Filtering
│   ├── Category
│   ├── Price
│   ├── Status
│   └── Facets
│
├── User Experience
│   ├── Autocomplete
│   ├── Highlighting
│   └── Pagination
│
├── Performance
│   ├── Cache
│   ├── Index
│   ├── Debounce
│   └── Rate Limiting
│
├── Synchronization
│   ├── Event
│   ├── Queue
│   ├── Worker
│   └── Eventual Consistency
│
├── Security
│   ├── Authorization
│   ├── Input Validation
│   └── Query Safety
│
└── Monitoring
    ├── Search Latency
    ├── Error Rate
    ├── Query Volume
    ├── Zero Result
    └── Index Health
🔥 สิ่งที่ควรเข้าใจให้ลึก

ถ้าจะทำ Search ระดับ Production ผมแนะนำให้เข้าใจ:

1. Database Index
2. Full-Text Search
3. Search Index
4. Tokenization
5. Fuzzy Search
6. Ranking / Relevance
7. Filtering
8. Autocomplete
9. Cache
10. Index Synchronization
11. Eventual Consistency
12. Search Security
13. Search Performance

และจำ Architecture นี้:

                      User
                       ↓
                   Search API
                       ↓
          ┌────────┴────────┐
          ↓                        ↓
         Redis                Search Engine
          │                       │
          │                  Search Index
          │                       │
          └────────┬────────┘
                      ↓
                   Results

ส่วนการ Update ข้อมูล:

SQL Server
    ↓
Product Updated
    ↓
Queue
    ↓
Worker
    ↓
Search Index
⚠️ จุดที่ต้องจำ

Database เป็น Source of Truth (ซอร์ส ออฟ ทรูธ)
Search Index เป็นข้อมูลสำหรับการค้นหา

ดังนั้นถ้า:

SQL Server = ถูกต้อง
Search Index = ผิด

เราควรมีระบบ Rebuild / Re-sync Index ไม่ใช่ถือว่า Search Index เป็นข้อมูลหลัก

และสำหรับ Stack (สแตก) ของคุณที่เป็น Go API + SQL Server แนวทางเริ่มต้นที่ดีคือ:

เริ่มเล็ก
↓
SQL Server + Index
↓
Full-Text Search
↓
Cache
↓
Queue สำหรับ Sync
↓
เมื่อข้อมูล/Traffic ใหญ่จริง
↓
พิจารณา Search Engine

ไม่จำเป็นต้องกระโดดไปใช้ Search Engine ตั้งแต่วันแรก เพราะ Search Engine เพิ่มความซับซ้อนด้าน Infrastructure (อินฟราสตรักเชอร์), Synchronization และ Monitoring ด้วยครับ