Deployment & Infrastructure คืออะไร?
Deployment (ดีพลอยเมนต์)

คือกระบวนการนำ Application จากเครื่อง Developer ไปให้ผู้ใช้งานจริง

Developer
   ↓
Git
   ↓
Build
   ↓
Test
   ↓
Deploy
   ↓
Server
   ↓
Production
Infrastructure (อินฟราสตรักเชอร์)

คือทรัพยากรและสภาพแวดล้อมที่ Application ต้องใช้

เช่น

Server
CPU
Memory
Disk
Network
Database
Redis
Storage
Load Balancer
DNS
Firewall
Container

ดังนั้น

Deployment = เอา Software ไปใช้งาน

Infrastructure = สิ่งแวดล้อมที่ Software ใช้งานอยู่

2. Core Components
DEPLOYMENT & INFRASTRUCTURE
├── Server
├── Operating System
├── Network
├── DNS
├── Firewall
├── Load Balancer
├── Reverse Proxy
├── Application Runtime
├── Process Manager
├── Container
├── Docker
├── Docker Compose
├── CI/CD
├── Build
├── Deployment Strategy
├── Environment
├── Configuration
├── Secret Management
├── Health Check
├── Scaling
├── Backup
├── Disaster Recovery
├── Monitoring
└── Logging
3. Server 🖥️

Application ต้องมีเครื่องสำหรับ Run

ตัวอย่าง

Server
├── CPU
├── RAM
├── Disk
└── Network

เช่น

CPU    8 Core
RAM    16 GB
Disk   500 GB
OS     Windows Server / Linux

Application

Go API
Next.js
Worker

อาจอยู่บน Server เดียวกัน หรือแยก Server

4. Operating System

Application ต้องทำงานบน OS

Windows Server
Linux

ในโลก Server ต้องเข้าใจอย่างน้อย

Process
Service
Port
File System
Permission
Environment Variable
Network
CPU
Memory
Disk

เช่น Go API

go-api.exe
    ↓
Port 8080

หรือ Linux

go-api
    ↓
:8080
5. Port 🌐

Application มัก Listen Port

Next.js → 3000
Go API  → 8080
Redis   → 6379
SQL     → 1433

ตัวอย่าง

Browser
   ↓
:443
   ↓
Reverse Proxy
   ↓
Go API :8080

ไม่จำเป็นต้องเปิดทุก Port ออก Internet

6. DNS

DNS (ดีเอ็นเอส) แปลง Domain → IP

เช่น

api.example.com
       ↓
192.168.1.100

ผู้ใช้จึงไม่ต้องจำ IP

https://api.example.com

Architecture

User
 ↓
DNS
 ↓
Server IP
 ↓
Load Balancer / Reverse Proxy
 ↓
Application
7. HTTPS / TLS 🔐

Production ไม่ควรให้ User ใช้

HTTP

ควรใช้

HTTPS
User
 ↓
HTTPS :443
 ↓
Reverse Proxy
 ↓
HTTP :8080
 ↓
Application

TLS ช่วยเข้ารหัสข้อมูลระหว่าง Client ↔ Server

เชื่อมกับ #2 Security System

8. Firewall 🧱

Firewall ควบคุมว่า Traffic ไหนสามารถเข้า Server ได้

ตัวอย่าง

Internet
   ↓
Firewall
   │
   ├── 443 → Allow
   ├── 80  → Redirect/Allow
   ├── 1433 → Block ❌
   └── 6379 → Block ❌

ไม่ควรเปิด Database ออก Internet โดยไม่จำเป็น

แนวคิดสำคัญคือ

เปิดเฉพาะ Port ที่จำเป็น

9. Reverse Proxy

Reverse Proxy (รีเวิร์ส พร็อกซี) อยู่หน้าตัว Application

ตัวอย่าง

User
 ↓
Nginx / IIS
 ↓
Next.js

หรือ

User
 ↓
Nginx
 ↓
Go API

หน้าที่อาจรวมถึง

HTTPS
Routing
Load Balancing
Compression
Caching
Security Headers
Rate Limiting
10. Load Balancer ⚖️

ถ้ามี Application หลาย Instance

             Load Balancer
              /    |    \
             ↓     ↓     ↓
           API1   API2   API3

Request ถูกกระจาย

Request 1 → API1
Request 2 → API2
Request 3 → API3
Request 4 → API1

ช่วย

Scale
Availability
Traffic Distribution
11. Stateless Application

ถ้าจะ Scale API หลาย Instance ได้ง่าย Application ควรเป็น

Stateless (สเตตเลส)

หมายถึง API ไม่ควรเก็บ Session สำคัญไว้ใน Memory ของ Instance เดียว

ไม่ควร

User
 ↓
API1
 ↓
Session อยู่ RAM API1

แล้ว Request ถัดไปไป API2

User
 ↓
API2
 ↓
ไม่มี Session ❌

ควรใช้

API1 ─┐
API2 ─┼→ Redis / Database
API3 ─┘

หรือใช้ Token-based Authentication ตามการออกแบบระบบ

เชื่อมกับ #1 Authentication และ #5 Performance

12. Process Manager

Application ต้องมี Process ที่ทำงานต่อเนื่อง

ถ้า Application Crash

Go API
 ↓
Crash 💥

ต้องมีระบบช่วย Restart

Process Manager
       ↓
Detect Crash
       ↓
Restart

บน Windows อาจใช้ Windows Service

บน Linux อาจใช้ systemd

หรือใช้ Container Runtime เช่น Docker

13. Docker 🐳

Docker ทำให้ Application และ Environment ถูก Package เป็น Container

Source Code
   ↓
Docker Build
   ↓
Docker Image
   ↓
Docker Container
   ↓
Application

ตัวอย่าง

Go API
 ↓
Docker Image
 ↓
Container
 ↓
:8080

ข้อดีคือ Environment มีความสม่ำเสมอมากขึ้น

Developer
     ↓
Same Image
     ↓
UAT
     ↓
Same Image
     ↓
Production

เชื่อมโดยตรงกับ #15 Configuration System

14. Docker Image vs Container

จำง่าย ๆ

Image
= Template

Container
= Running Instance

เช่น

Go API Image
      ↓
 ┌──┼───┐
 ↓    ↓    ↓
API1 API2 API3

Image เดียวสามารถสร้างหลาย Container

15. Docker Compose

ถ้ามีหลาย Service

Next.js
Go API
Redis
Worker

สามารถจัดการด้วย Docker Compose

docker-compose
├── frontend
├── api
├── worker
└── redis

Architecture

                Docker Compose
                     │
       ┌─────────┼─────────┐
       ↓             ↓             ↓
    Next.js         Go API        Redis
                      ↓
                    Worker
                      ↓
                  SQL Server

เหมาะมากสำหรับ Development / UAT และระบบขนาดเล็กถึงกลางบางรูปแบบ

16. CI/CD 🚀

CI/CD (ซีไอ/ซีดี) คือ Automation ของ Build → Test → Deploy

ตัวอย่าง

Developer
 ↓
git push
 ↓
CI
 ↓
Build
 ↓
Test
 ↓
Security Check
 ↓
Docker Build
 ↓
Deploy
 ↓
Health Check

แทนที่จะ

Developer
 ↓
Copy File
 ↓
Remote Desktop
 ↓
แก้ไฟล์
 ↓
Run

ซึ่งมีความเสี่ยงสูง

17. Continuous Integration

CI (Continuous Integration)

เมื่อ Developer Push Code

Git Push
 ↓
Build
 ↓
Unit Test
 ↓
Integration Test
 ↓
Lint

ถ้าไม่ผ่าน

❌ Stop

ไม่ควร Deploy ต่อ

18. Continuous Delivery / Deployment

หลัง CI ผ่าน

Build
 ↓
Test
 ↓
Package
 ↓
Deploy
Continuous Delivery

ระบบเตรียมพร้อม Deploy แต่ยังอาจให้คนกด Approve

Continuous Deployment

ผ่าน Pipeline แล้ว Deploy อัตโนมัติ

19. Deployment Strategy

มีหลายรูปแบบ

Rolling Deployment

ค่อย ๆ เปลี่ยน Instance

API1 old
API2 old
API3 old

↓ Deploy

API1 new
API2 old
API3 old

↓
API1 new
API2 new
API3 old

ลด Downtime

Blue-Green Deployment

มีสอง Environment

Blue  = Current
Green = New
User
 ↓
Blue

Deploy Green

User
 ↓
Green

ถ้าเกิดปัญหาสามารถเปลี่ยนกลับได้

Green ❌
 ↓
Blue
Canary Deployment

ปล่อยให้ User บางส่วนใช้ Version ใหม่ก่อน

100 Users

90 → V1
10 → V2

Monitor

Error
Latency
CPU

ถ้าปกติค่อยเพิ่ม Traffic

20. Zero Downtime Deployment

เป้าหมายคือ

Deploy
 ↓