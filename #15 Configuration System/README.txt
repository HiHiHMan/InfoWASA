Configuration System ⚙️
1. Configuration System คืออะไร?

Configuration System (คอนฟิกกูเรชัน ซิสเท็ม) คือระบบที่ใช้จัดการค่าต่าง ๆ ที่ระบบต้องใช้ แต่ ไม่ควรเขียนตายตัว (Hard-code) อยู่ใน Source Code

เช่น

Database Connection
API URL
Port
JWT Expiration
Redis Address
File Size Limit
Email Server
Feature Toggle
Retry Count
Timeout
Environment

ตัวอย่างที่ไม่ควรทำ ❌

db, err := sql.Open(
    "sqlserver",
    "server=192.168.1.10;user=sa;password=123456",
)

เพราะ Password อยู่ใน Code

ควรเป็น

Environment Variable
        ↓
Configuration
        ↓
Application
2. ทำไม Configuration System สำคัญ?

เพราะ Application เดียวกันต้องทำงานหลาย Environment

Development
      ↓
UAT
      ↓
Production

แต่แต่ละ Environment ใช้ค่าต่างกัน

Development
DB = localhost
Redis = localhost

UAT
DB = UAT-SQL01
Redis = UAT-REDIS

Production
DB = PROD-SQL01
Redis = PROD-REDIS

Code ไม่ควรต้องแก้ตาม Environment

Same Code
   │
   ├── DEV Config
   ├── UAT Config
   └── PROD Config
3. Core Components
CONFIGURATION SYSTEM
├── Environment
├── Configuration Source
│   ├── Environment Variable
│   ├── Config File
│   ├── Secret Manager
│   └── Remote Config
├── Configuration Loader
├── Configuration Validation
├── Configuration Schema
├── Default Value
├── Override
├── Secret Management
├── Feature Flag
├── Configuration Versioning
├── Configuration Reload
├── Environment Separation
├── Access Control
├── Audit
└── Monitoring
4. Environment Configuration 🌍

พื้นฐานที่สุดคือแยก Environment

.env.development
.env.uat
.env.production

เช่น

APP_ENV=development
APP_PORT=8080

DB_HOST=localhost
DB_PORT=1433
DB_NAME=MyApp

Production:

APP_ENV=production
APP_PORT=8080

DB_HOST=PROD-SQL01
DB_PORT=1433
DB_NAME=MyApp

Application ใช้ Code เดิม

แต่ Configuration ต่างกัน

5. Environment Variable

Environment Variable (เอ็นไวรอนเมนต์ แวริอะเบิล) เป็นวิธีที่นิยมมากในการส่ง Configuration ให้ Application

ตัวอย่าง

APP_ENV=production
APP_PORT=8080
DB_HOST=192.168.1.100
DB_NAME=ERP

Go:

port := os.Getenv("APP_PORT")

ข้อดี:

ไม่ต้องแก้ Code
เปลี่ยนตาม Environment ได้
เหมาะกับ Docker
เหมาะกับ CI/CD
ลดการ Hard-code
6. Config File 📄

บาง Configuration สามารถเก็บใน File

เช่น

server:
  port: 8080
  timeout: 30

database:
  maxOpenConnections: 100
  maxIdleConnections: 20

logging:
  level: info

เหมาะกับ Configuration ที่ไม่ใช่ Secret

เช่น

Timeout
Limit
Feature Setting
Logging Level
Worker Count

แต่ไม่ควรเอา Password ไปใส่ตรง ๆ

7. Secret Configuration 🔐

สิ่งสำคัญมาก

Configuration กับ Secret ไม่เหมือนกัน

Configuration
PORT=8080
TIMEOUT=30
MAX_CONNECTION=100
Secret
DB_PASSWORD
JWT_SECRET
API_KEY
SMTP_PASSWORD
ENCRYPTION_KEY

Secret ควรใช้

Secret Management (ซีเคร็ต แมเนจเมนต์)

เช่น

Secret Manager
Vault
Cloud Secret Manager
Kubernetes Secret
Environment Secret

หลักการคือ

Source Code
     ❌
Password

Secret Store
     ↓
Application
     ↓
Password
8. Configuration Loader

ไม่ควรให้ทุกส่วนของ Application อ่าน Environment เอง

แบบนี้ไม่ดี ❌

os.Getenv("DB_HOST")
os.Getenv("DB_PORT")
os.Getenv("DB_NAME")

กระจายเต็ม Project

ควรมี

Environment
     ↓
Config Loader
     ↓
Config Object
     ↓
Application

ตัวอย่างแนวคิด

type Config struct {
    App      AppConfig
    Database DatabaseConfig
    Redis    RedisConfig
}

แล้ว Application ใช้

config.Database.Host
config.Database.Port

แทนการอ่าน Environment ทุกที่

9. Configuration Validation ✅

อย่าปล่อยให้ Application Start แล้วค่อยพบว่า Configuration ผิด

เช่น

DB_HOST=
DB_PORT=abc
DB_MAX_CONNECTION=-10

ควรตรวจตอน Startup

Application Start
       ↓
Load Config
       ↓
Validate
       ↓
Valid?
 ┌─────┴─────┐
No          Yes
 ↓            ↓
Stop        Start

ตัวอย่าง

DB_HOST       Required
DB_PORT       Number
DB_MAX_CONN   > 0
JWT_SECRET    Required
APP_ENV       DEV/UAT/PROD
10. Fail Fast 🚨

ถ้า Configuration สำคัญหาย

เช่น

DB_PASSWORD ไม่มี
JWT_SECRET ไม่มี

ไม่ควร

Application Start
      ↓
ทำงานไปก่อน
      ↓
Request เข้ามา
      ↓
💥 Error

ควร

Application Start
      ↓
Validate Config
      ↓
INVALID
      ↓
❌ Application ไม่ Start

นี่เรียกว่า Fail Fast (เฟล ฟาสต์)

11. Default Value

บาง Configuration มี Default

เช่น

APP_PORT
Default = 8080

TIMEOUT
Default = 30 sec

MAX_RETRY
Default = 3

แต่ต้องระวัง

Configuration ที่เกี่ยวกับ Security หรือ Database สำคัญ ๆ ไม่ควร Default แบบอันตราย

เช่น

DB_PASSWORD = ""
JWT_SECRET = "123456"

ไม่ควรเด็ดขาด ❌

12. Configuration Override

ระบบอาจมี Configuration หลาย Layer

เช่น

Default
   ↓
Config File
   ↓
Environment Variable
   ↓
Runtime Override

ตัวอย่าง

Default Timeout = 30

UAT Config = 60

Environment Variable = 120

ค่าที่มี Priority สูงกว่าจะ Override ค่าเก่า

120 ← ใช้งาน

ควรกำหนดลำดับให้ชัดเจน ไม่อย่างนั้น Debug ยากมาก

13. Feature Flag 🚩

เป็นส่วนสำคัญของ Configuration System

Feature Flag (ฟีเจอร์ แฟลก) ใช้เปิด/ปิด Feature โดยไม่ต้อง Deploy Code ใหม่

เช่น

NEW_DASHBOARD=true
NEW_PAYMENT=false
NEW_SEARCH=true

Application:

if NewDashboard {
    ใช้ Dashboard ใหม่
} else {
    ใช้ Dashboard เดิม
}

เหมาะกับ

ทดลอง Feature
ทยอยเปิดระบบ
A/B Testing
Emergency Disable
Release Control
14. Dynamic Configuration

Configuration บางตัวอาจเปลี่ยนได้ขณะ Application กำลังทำงาน

เช่น

Rate Limit
Feature Flag
Cache TTL
Maintenance Mode

ตัวอย่าง

FeatureFlag
      ↓
Redis / Config Service
      ↓
Application

ข้อดีคือไม่ต้อง Restart

แต่มีความซับซ้อนเพิ่มขึ้น

15. Configuration Reload 🔄

มี 2 แนวทาง

Restart
แก้ Config
 ↓
Restart Application
 ↓
โหลด Config ใหม่

ง่ายและปลอดภัยกว่าในหลายกรณี

Hot Reload
แก้ Config
 ↓
Application Detect
 ↓
Reload

เหมาะกับ Configuration บางประเภท

แต่ต้องระวัง Race Condition

เช่น

Request A
   ↓
Config Version 1

Config Reload

Request B
   ↓
Config Version 2

ต้องออกแบบให้ State เปลี่ยนอย่างปลอดภัย

16. Configuration Versioning

ใน Production ควรรู้ว่า

ตอนเกิดปัญหา ระบบใช้ Configuration Version ไหน?

เช่น

Config v1
DB Pool = 50

Config v2
DB Pool = 100

Config v3
DB Pool = 200

ถ้าเกิดปัญหา

CPU ↑
DB Connection ↑

สามารถตรวจได้ว่า

หลัง Deploy Config v3
↓
ปัญหาเริ่มเกิด

ช่วย Incident Investigation ได้มาก

เชื่อมกับ #8 Monitoring & Observability

17. Configuration Access Control 🔐

ไม่ใช่ทุกคนควรแก้ Configuration

ตัวอย่าง

Developer
→ Read DEV

QA
→ Read UAT

Admin
→ Manage Production

Production Configuration ควรมี Permission เข้มกว่า Development

18. Audit

ถ้า Configuration เปลี่ยน ต้องรู้ว่า

Who
What
When
Before
After
Reason

ตัวอย่าง

User: Admin01
Config: DB_MAX_CONNECTION
Old: 100
New: 200
Time: 10:32

โดยเฉพาะ

Production
Security
Database
Payment
Feature Flag
19. Configuration กับ Docker 🐳

Configuration System สำคัญมากกับ Docker

ไม่ควร Build Image ใหม่เพียงเพราะ Database เปลี่ยน

เช่น

Docker Image
      ↓
Same Image
      │
      ├── DEV Config
      ├── UAT Config
      └── PROD Config

แนวคิดสำคัญคือ

Build once, configure per environment

20. Configuration กับ CI/CD 🚀

Pipeline

Git
 ↓
Build
 ↓
Test
 ↓
Docker Image
 ↓
Deploy
 ↓
Environment Config
 ↓
Application

ไม่ควรเอา Production Secret ใส่ Git Repository

เช่น ❌

GitHub
 └── .env.production
      └── DB_PASSWORD=xxxxx

แม้ Repository จะเป็น Private ก็ไม่ควรถือว่าเป็น Secret Store

21. Configuration กับ Database

Database Configuration มักมี

DB_HOST
DB_PORT
DB_NAME
DB_USER
DB_PASSWORD
DB_MAX_OPEN
DB_MAX_IDLE
DB_CONNECTION_TIMEOUT

และมีความสัมพันธ์กับ #3 Database System

เช่น

DB_MAX_CONNECTION = 100

ไม่ได้หมายความว่า

100 = ดีเสมอ

ถ้า SQL Server รับ Workload ไม่ไหว

Pool = 100
 ↓
Query จำนวนมาก
 ↓
SQL Server CPU สูง
 ↓
Query ช้า
 ↓
Connection ถูกถือไว้นาน

Configuration ที่ตั้งสูงเกินไปสามารถทำให้ระบบแย่ลงได้

22. Configuration กับ Performance

ตัวอย่าง

WORKER_COUNT=100

ดูเหมือนจะเร็ว

แต่จริง ๆ

100 Workers
 ↓
100 DB Requests
 ↓
SQL Server
 ↓
CPU / Lock / Connection Pool
 ↓
ช้า

ดังนั้น Configuration ต้องพิจารณาร่วมกับ

CPU
Memory
Database
Connection Pool
Queue
Network
External API
23. Configuration กับ Security

ห้าม Log Secret

❌

DB_PASSWORD=abc123
JWT_SECRET=xxxx

เช่นตอน Startup

Config loaded:
DB_HOST=...
DB_PASSWORD=...

อันตรายมาก

ควร Mask

DB_HOST=SQL01
DB_PASSWORD=********
24. Configuration กับ Multi-Instance

ถ้ามีหลาย Server

API 1
API 2
API 3
API 4

ต้องระวัง Configuration ไม่ตรงกัน

เช่น

API 1 → Timeout 30
API 2 → Timeout 30
API 3 → Timeout 120 ❌
API 4 → Timeout 30

User จึงเจอพฤติกรรมไม่เหมือนกัน

ระบบใหญ่จึงนิยม Centralized Configuration

             Config Service
              /    |    \
             ↓     ↓     ↓
          API1   API2   API3
25. Configuration Drift

Configuration Drift (คอนฟิกกูเรชัน ดริฟต์) คือ Configuration ของแต่ละ Server ค่อย ๆ ไม่เหมือนกัน

เช่น

Server A
MAX_WORKER=50

Server B
MAX_WORKER=50

Server C
MAX_WORKER=100

Server D
MAX_WORKER=20

ระบบทำงานไม่เหมือนกัน

นี่เป็นปัญหาที่พบได้ในระบบที่มีหลาย Server และหลาย Environment

26. Production Configuration Checklist

ก่อน Deploy Production ควรตรวจ

☑ APP_ENV ถูกต้อง
☑ DB Host ถูกต้อง
☑ DB Credential ถูกต้อง
☑ DB Pool ถูกต้อง
☑ Redis ถูกต้อง
☑ API URL ถูกต้อง
☑ Timeout ถูกต้อง
☑ Retry ถูกต้อง
☑ JWT Configuration ถูกต้อง
☑ Encryption Key ถูกต้อง
☑ File Storage ถูกต้อง
☑ Log Level ถูกต้อง
☑ Feature Flag ถูกต้อง
☑ Secret ไม่อยู่ใน Git
☑ Secret ไม่ถูก Log
☑ Config Validation ผ่าน
☑ Config Version ถูกบันทึก
☑ Permission ถูกต้อง
27. Mental Model 🧠
CONFIGURATION SYSTEM
│
├── Environment
│   ├── Development
│   ├── UAT
│   └── Production
│
├── Configuration Source
│   ├── Default
│   ├── Config File
│   ├── Environment Variable
│   ├── Secret Manager
│   └── Remote Config
│
├── Config Loader
│
├── Validation
│   ├── Required
│   ├── Type
│   ├── Range
│   └── Format
│
├── Override
│
├── Secret
│   ├── DB Password
│   ├── API Key
│   ├── JWT Secret
│   └── Encryption Key
│
├── Feature Flag
│
├── Runtime
│   ├── Reload
│   └── Restart
│
├── Versioning
│
├── Access Control
│
├── Audit
│
└── Monitoring
28. Deep Topics ที่ควรรู้ 🔥

ถ้าจะเข้าใจ Configuration System แบบ Production จริง ๆ ให้ลงลึกตามลำดับนี้

1. Environment Variable
2. Config File
3. Configuration Loader
4. Configuration Validation
5. Fail Fast
6. Default / Override
7. Secret Management
8. Feature Flag
9. Dynamic Configuration
10. Configuration Reload
11. Configuration Versioning
12. Configuration Drift
13. Centralized Configuration
14. Configuration Security
15. Configuration Audit
16. Configuration + Docker
17. Configuration + CI/CD
18. Configuration + Kubernetes
🔥 หลักที่ควรจำ

Configuration System มีหน้าที่ตอบคำถาม 5 ข้อนี้:

1. ระบบใช้ค่าอะไร?
2. ค่านี้มาจากไหน?
3. Environment ไหนกำลังใช้?
4. ใครสามารถเปลี่ยนได้?
5. ถ้าเปลี่ยนแล้ว เรารู้ได้อย่างไร?

และ Architecture ที่เหมาะกับ Stack ของคุณจะประมาณนี้:

                 Configuration
                       │
          ┌────────┼─────────┐
          ↓            ↓            ↓
        Go API       Worker       Scheduler
          │            │            │
          └────────┼─────────┘
                       ↓
              Database / Redis
                       │
          ┌────────┴─────────┐
          ↓                         ↓
       DEV Config               PROD Config

จุดที่ต้องระวังที่สุด: Configuration ไม่ใช่แค่เรื่อง .env แต่เกี่ยวข้องกับ Security + Database + Performance + Docker + CI/CD + Feature Flag + Production Operations โดยตรง

โดยเฉพาะระบบของคุณที่มี หลาย Server / หลาย Project / API หลายตัว เรื่อง Configuration Drift, Secret Management และ Environment Separation จะสำคัญมากเป็นพิเศษ.