⚡ 5. Performance System (เพอร์ฟอร์แมนซ์ ซิสเต็ม)

Performance System = ระบบและแนวทางที่ทำให้ Application (แอปพลิเคชัน) ทำงานได้เร็ว ใช้ทรัพยากรเหมาะสม และรองรับ Concurrent Users (คอนเคอร์เรนต์ ยูสเซอร์) ได้ดี

                                 ⚡ PERFORMANCE
                                        │
          ┌────────────────┼────────────────┐
          ↓                             ↓                             ↓
      🚀 Speed                📦 Resource               🔀 Concurrency
         ความเร็ว                    การใช้ทรัพยากร                การทำงานพร้อมกัน


🚀 1. Caching (แคชชิง)
เก็บข้อมูลที่เรียกบ่อยไว้ชั่วคราว เพื่อไม่ต้องประมวลผลใหม่ทุกครั้ง

Request
   ↓
Cache มีข้อมูลไหม?
   │
 ┌─┴─┐
มี   ไม่มี
│      │
↓      ↓
คืนค่า  Database
       ↓
      Cache
       ↓
     Response

ตัวอย่าง:

GET /api/products

ครั้งแรก
API → Database → Cache → User

ครั้งต่อไป
API → Cache → User

ช่วยลด:

Database Load (ดาต้าเบส โหลด)
API Processing (เอพีไอ โพรเซสซิง)
Response Time (รีสปอนส์ ไทม์)

แต่ต้องเข้าใจ Cache Invalidation (แคช อินแวลลิเดชัน) ด้วย เพราะข้อมูลใน Cache อาจเก่า

🟥 2. Redis (เรดิส)

Redis เป็นระบบจัดเก็บข้อมูลในหน่วยความจำที่มีความเร็วสูง และนิยมใช้เป็น Distributed Cache (ดิสทริบิวเต็ด แคช)

ใช้ได้กับ:

🟥 Cache
🔐 Session
🚦 Rate Limiting
🔑 Distributed Lock
📨 Queue / Stream
📊 Counter

ต้องเข้าใจเพิ่มเติม:

TTL (ทีทีแอล)
Key / Value (คีย์ / แวลู)
Eviction (อีวิกชัน)
Persistence (เพอร์ซิสเทนซ์)
Distributed Cache (ดิสทริบิวเต็ด แคช)
Cache Stampede (แคช สแตมพีด)
🗄️ 3. Database Cache (ดาต้าเบส แคช)

Database (ดาต้าเบส) เองก็มีการใช้ Memory (เมมโมรี) เพื่อเก็บข้อมูลและโครงสร้างที่ถูกใช้งานบ่อย

สำหรับ SQL Server (เอสคิวแอล เซิร์ฟเวอร์) ควรเข้าใจ:

Buffer Pool
   ↓
Data Pages
   ↓
Memory

ถ้าข้อมูลที่ต้องการอยู่ใน Memory แล้ว Database อาจไม่จำเป็นต้องอ่านจาก Disk (ดิสก์) ทุกครั้ง

ดังนั้น Performance ของ Database ไม่ได้มีแค่ Index (อินเด็กซ์) แต่เกี่ยวข้องกับ:

Query
 ↓
Execution Plan
 ↓
Index
 ↓
Memory
 ↓
Disk I/O
🌐 4. HTTP Cache (เอชทีทีพี แคช)

ให้ Browser (เบราว์เซอร์) หรือ Proxy (พร็อกซี) เก็บ Response (รีสปอนส์) ไว้

ตัวอย่าง Header (เฮดเดอร์):

Cache-Control: max-age=3600

เหมาะกับข้อมูลที่ไม่ได้เปลี่ยนบ่อย เช่น:

🖼️ Images
📦 Static Files
📄 CSS
📜 JavaScript

ต้องเข้าใจ:

Cache-Control
ETag
Last-Modified
Browser Cache
CDN Cache
🌍 5. CDN (ซีดีเอ็น)

Content Delivery Network (คอนเทนต์ เดลิเวอรี เน็ตเวิร์ก)

เอาข้อมูลไปกระจายไว้ตาม Server (เซิร์ฟเวอร์) หลายพื้นที่ เพื่อให้ User (ยูสเซอร์) โหลดจากจุดที่ใกล้กว่า

             🌍 CDN
          ┌────┼────┐
          ↓    ↓    ↓
        Asia Europe USA
          │
          ↓
        User

เหมาะกับ:

Image (รูปภาพ)
Video (วิดีโอ)
JavaScript
CSS
Static Assets (สแตติก แอสเซ็ต)

ช่วยลดภาระ Origin Server (ออริจิน เซิร์ฟเวอร์)

🔌 6. Connection Pool (คอนเนกชัน พูล)

เกี่ยวข้องกับ Performance โดยตรง

ไม่ควร:

Request
 ↓
สร้าง Database Connection
 ↓
Query
 ↓
ปิด Connection

ทุก Request เพราะการสร้าง Connection มีต้นทุน

ควร:

             Connection Pool
          ┌────┬────┬────┬────┐
Request → │ C1 │ C2 │ C3 │ C4 │
          └────┴────┴────┴────┘

แต่ต้องระวัง Connection Pool Starvation (คอนเนกชัน พูล สตาร์เวชัน)

เช่น:

Pool = 20 Connections

Request 1 → ใช้ C1
Request 2 → ใช้ C2
...
Request 20 → ใช้ C20

Request 21
    ↓
ไม่มี Connection
    ↓
ต้องรอ
    ↓
Timeout
⚡ 7. Query Optimization (คิวรี ออปทิไมเซชัน)

ต้องทำให้ Database ประมวลผล Query (คิวรี) อย่างมีประสิทธิภาพ

ต้องดู:

Execution Plan
├── Index Seek
├── Index Scan
├── Table Scan
├── Key Lookup
├── Join
├── Sort
├── Filter
└── Estimated / Actual Rows

ตัวอย่างง่าย ๆ:

SELECT *
FROM Orders
WHERE CustomerId = 100;

ถ้ามี Index ที่เหมาะสม อาจค้นหาได้เร็วกว่าอ่านทั้ง Table (เทเบิล)

แต่ Index ไม่ได้แปลว่ายิ่งเยอะยิ่งดี เพราะ Index เพิ่มภาระในการเขียนข้อมูลและใช้ Storage (สตอเรจ)

💤 8. Lazy Loading (เลซซี โหลดดิง)

โหลดข้อมูลเมื่อจำเป็น แทนที่จะโหลดทุกอย่างตั้งแต่แรก

เช่น:

เปิดหน้า User
   ↓
โหลด User Profile
   ↓
ยังไม่โหลด Order History
   ↓
User กด "ประวัติการสั่งซื้อ"
   ↓
ค่อยโหลด Order History

ช่วยลด:

Initial Load (อินิเชียล โหลด)
Network (เน็ตเวิร์ก)
Memory
Database Query

แต่ต้องระวัง N+1 Query Problem (เอ็น พลัส วัน คิวรี พร็อบเล็ม)

📦 9. Batch Processing (แบตช์ โพรเซสซิง)

แทนที่จะทำทีละรายการ:

100,000 Records
      ↓
INSERT
INSERT
INSERT
INSERT
...

จัดกลุ่ม:

100,000 Records
      ↓
Batch 1 → 1,000
Batch 2 → 1,000
Batch 3 → 1,000
...

ช่วยลด:

Network Round Trip (เน็ตเวิร์ก ราวด์ ทริป)
Database Calls
Transaction Overhead (ทรานแซกชัน โอเวอร์เฮด)

เหมาะกับ:

📥 Import
📤 Export
📊 Report
🔄 Data Processing
🗜️ 10. Compression (คอมเพรสชัน)

ลดขนาดข้อมูลที่ส่งผ่าน Network (เน็ตเวิร์ก)

เช่น:

JSON
 ↓
Gzip / Brotli
 ↓
Network

ตัวอย่าง:

ข้อมูลเดิม = 1 MB
↓
Compress
↓
ข้อมูลที่ส่ง = 250 KB

ช่วยลด Bandwidth (แบนด์วิดท์) และอาจลดเวลาในการส่งข้อมูล แต่ต้องแลกกับ CPU (ซีพียู) สำหรับการบีบอัด/คลายการบีบอัด

🔀 Concurrency & Performance

ส่วนที่คุณระบุมา ควรแยกเป็นหมวดสำคัญอีกหมวดหนึ่ง เพราะมันเกี่ยวกับทั้ง Performance และ Correctness (คอร์เร็กต์เนส)

🔀 CONCURRENCY
│
├── 🏁 Race Condition
├── 🔒 Deadlock
├── ⛔ Starvation
├── 🔌 Connection Pool Starvation
└── 🐘 Thundering Herd
🏁 Race Condition (เรซ คอนดิชัน)

หลาย Request แก้ข้อมูลเดียวกันพร้อมกัน แล้วผลลัพธ์ขึ้นอยู่กับว่าใครทำงานก่อน

Stock = 1

Request A → อ่าน Stock = 1
Request B → อ่าน Stock = 1

A → ซื้อ
B → ซื้อ

💥 สินค้า 1 ชิ้น แต่ขาย 2 ครั้ง

ต้องแก้ด้วยกลไก เช่น Transaction (ทรานแซกชัน), Lock (ล็อก), Atomic Operation (อะทอมมิก โอเปอเรชัน) หรือ Optimistic Concurrency Control (ออพทิมิสติก คอนเคอร์เรนซี คอนโทรล) ตามกรณี

🔒 Deadlock (เดดล็อก)

Process (โพรเซส) หรือ Transaction (ทรานแซกชัน) รอกันเองจนไม่มีใครไปต่อ

Transaction A
    ↓
Lock A
    ↓
รอ B
    ↑
    │
Transaction B
    ↓
Lock B
    ↓
รอ A

💥 DEADLOCK

สำหรับ SQL Server ควรเข้าใจ Lock Modes (ล็อก โมดส์), Blocking (บล็อกกิง), Deadlock Graph (เดดล็อก กราฟ) และแนวทาง Retry (รีทราย)

⛔ Starvation (สตาร์เวชัน)

งานหนึ่งถูกงานอื่นแซงหรือแย่ง Resource (รีซอร์ส) ซ้ำ ๆ จนแทบไม่ได้ทำงาน

ต่างจาก Deadlock ตรงที่ระบบยังเดินต่อได้ แต่บางงานอาจรอนานผิดปกติ

🔌 Connection Pool Starvation

เป็นกรณีหนึ่งของ Resource Starvation (รีซอร์ส สตาร์เวชัน)

Connection Pool
      ↓
Connections ถูกใช้งานหมด
      ↓
Request ใหม่รอ
      ↓
รอนาน
      ↓
Timeout

สิ่งที่ทำให้เกิดได้ เช่น:

Query ช้า
Transaction เปิดนาน
Connection ไม่ถูกคืน
จำนวน Connection ไม่เหมาะสม
ระบบรับ Request มากเกินกำลัง
🐘 Thundering Herd (ทันเดอริง เฮิร์ด)

เกิดเมื่อ Request จำนวนมากพยายามเข้าถึง Resource เดียวกันพร้อมกัน โดยเฉพาะหลัง Cache (แคช) หมดอายุ

Cache Expire
     ↓
Cache ไม่มีข้อมูล
     ↓
🔥 Request 1 ─┐
🔥 Request 2 ─┤
🔥 Request 3 ─┤→ Database
🔥 Request 4 ─┤
🔥 Request 5 ─┘
     ↓
Database รับโหลดมหาศาล

แนวทางที่ใช้ได้ตามบริบท เช่น:

Cache
  +
TTL
  +
Jitter
  +
Lock / Single Flight
  +
Request Coalescing
🧠 ภาพรวมที่ควรจำ
⚡ PERFORMANCE SYSTEM
│
├── 🚀 Caching
│   ├── Redis
│   ├── Database Cache
│   ├── HTTP Cache
│   └── CDN
│
├── 🗄️ Database Performance
│   ├── Connection Pool
│   ├── Query Optimization
│   ├── Index
│   └── Batch Processing
│
├── 🌐 Network Performance
│   ├── HTTP Cache
│   ├── CDN
│   └── Compression
│
├── 🖥️ Application Performance
│   └── Lazy Loading
│
└── 🔀 Concurrency
    ├── Race Condition
    ├── Deadlock
    ├── Starvation
    ├── Connection Pool Starvation
    └── Thundering Herd
🎯 แก่นของ Performance

จำเป็นภาพเดียว:

ลดงาน → ลดการรอ → ลดการเดินทาง → ใช้ทรัพยากรให้คุ้ม → ควบคุมงานพร้อมกัน