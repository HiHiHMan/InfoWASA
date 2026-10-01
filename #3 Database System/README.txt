Database System 
🔌 Database Connection (ดาต้าเบส คอนเนกชัน)	
การเชื่อมต่อ API (เอพีไอ) กับ SQL Server, Connection String (คอนเนกชัน สตริง), Timeout (ไทม์เอาต์), Connection Lifecycle (วงจรชีวิตการเชื่อมต่อ)

♻️ Connection Pool (คอนเนกชัน พูล)	
การนำ Connection (คอนเนกชัน) กลับมาใช้แทนการสร้างใหม่ทุก Request (รีเควสต์), ขนาด Pool, Timeout, Pool Exhaustion (พูลเต็ม)

💳 Transaction (ทรานแซกชัน)	
ทำหลาย SQL (เอสคิวแอล) ให้สำเร็จทั้งหมดหรือยกเลิกทั้งหมด เช่น ตัด Stock (สต็อก) + สร้าง Order (ออร์เดอร์)

⚡ Index (อินเด็กซ์)	
ช่วยค้นข้อมูลเร็วขึ้น แต่เพิ่มภาระตอน INSERT / UPDATE / DELETE ต้องเข้าใจ Clustered (คลัสเตอร์ด) / Nonclustered (นอน-คลัสเตอร์ด) / Composite Index (คอมโพสิต อินเด็กซ์)

🛡️ Constraint (คอนสเทรนต์)	
กฎบังคับข้อมูล เช่น PRIMARY KEY, UNIQUE, CHECK, DEFAULT, FOREIGN KEY

🔗 Foreign Key (ฟอเรน คีย์)	
ควบคุมความสัมพันธ์ระหว่าง Table (เทเบิล) และ Referential Integrity (รีเฟอเรนเชียล อินทิกริตี)

🔒 Deadlock Handling (เดดล็อก แฮนดลิง)	
เข้าใจว่า Transaction สองตัวรอกันได้อย่างไร, SQL Server เลือก Victim (วิกทิม) อย่างไร และระบบควร Retry (รีทราย) อย่างไร

🔐 Isolation Level (ไอโซเลชัน เลเวล)	
ควบคุมว่าหลาย Transaction อ่าน/เขียนข้อมูลพร้อมกันอย่างไร เช่น READ COMMITTED, SNAPSHOT, SERIALIZABLE

🚀 Query Optimization (คิวรี ออปทิไมเซชัน)	
Execution Plan (เอ็กซิคิวชัน แพลน), Index Seek (อินเด็กซ์ ซีค), Index Scan (อินเด็กซ์ สแกน), Table Scan (เทเบิล สแกน), Join (จอยน์), Statistics (สแตทิสติกส์)

📄 Pagination (เพจิเนชัน)	
ดึงข้อมูลทีละหน้า ไม่โหลดข้อมูลเป็นแสน/ล้านแถวในครั้งเดียว เช่น OFFSET/FETCH หรือ Keyset Pagination (คีย์เซ็ต เพจิเนชัน)

🗑️ Soft Delete (ซอฟต์ ดีลีต)	
ไม่ลบข้อมูลจริง แต่ใช้ IsDeleted, DeletedAt ฯลฯ ต้องออกแบบ Index และ Query ให้เหมาะสม

💾 Data Backup (ดาต้า แบ็กอัป)	
Full Backup (ฟูล แบ็กอัป), Differential Backup (ดิฟเฟอเรนเชียล แบ็กอัป), Transaction Log Backup (ทรานแซกชัน ล็อก แบ็กอัป)

♻️ Data Recovery (ดาต้า รีคัฟเวอรี)	
Restore (รีสโตร์), Point-in-Time Recovery (พอยต์-อิน-ไทม์ รีคัฟเวอรี), Recovery Model (รีคัฟเวอรี โมเดล) และแผนกู้ระบบเมื่อ DB (ดีบี) เสีย


------------------------------------------------------------------------------------------------------------

🗄️ DATABASE SYSTEM
│
├── 🔌 Connection
│   ├── Database Connection
│   ├── Connection Pool
│   ├── Timeout
│   └── Connection Pool Starvation
│
├── 🔐 Transaction & Concurrency
│   ├── Transaction
│   ├── ACID
│   ├── Isolation Level
│   ├── Lock
│   ├── Blocking
│   ├── Deadlock
│   └── Race Condition
│
├── ⚡ Performance
│   ├── Index
│   ├── Execution Plan
│   ├── Query Optimization
│   ├── Statistics
│   ├── Join Optimization
│   └── Pagination
│
├── 🧱 Data Integrity
│   ├── Primary Key
│   ├── Foreign Key
│   ├── Unique
│   ├── Check
│   ├── Default
│   └── Referential Integrity
│
└── 💾 Disaster Recovery
    ├── Full Backup
    ├── Differential Backup
    ├── Transaction Log Backup
    ├── Restore
    ├── Point-in-Time Recovery
    └── Recovery Model