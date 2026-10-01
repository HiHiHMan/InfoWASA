🌐 API System — ระบบ API

API (เอพีไอ) คือช่องทางที่ให้ระบบหนึ่งสื่อสารกับอีกระบบหนึ่ง เช่น

Frontend (ฟรอนต์เอนด์)
        │
        │ HTTP Request (เอชทีทีพี รีเควสต์)
        ↓
     REST API
        │
        ↓
    Service (เซอร์วิส)
        │
        ↓
Database (ดาต้าเบส)



🌐 REST API (เรสต์ เอพีไอ)	
การออกแบบ API ตาม Resource (รีซอร์ส) เช่น /users, /orders, /products

🔤 HTTP Method (เอชทีทีพี เมธอด)	
GET, POST, PUT, PATCH, DELETE และความหมายของแต่ละตัว

🔢 Status Code (สเตตัส โค้ด)	
200, 201, 204, 400, 401, 403, 404, 409, 422, 429, 500 และเลือกใช้ให้ถูกสถานการณ์

📥 Request / Response (รีเควสต์ / รีสปอนส์)	
Headers (เฮดเดอร์ส), Path Parameter (พาธ พารามิเตอร์), Query Parameter (คิวรี พารามิเตอร์), Body (บอดี) และรูปแบบ Response

✅ Validation (วาลิเดชัน)	
ตรวจข้อมูลก่อนเข้า Business Logic (บิสซิเนส ลอจิก) เช่น Type (ชนิดข้อมูล), Required (จำเป็น), Length (ความยาว), Range (ช่วงค่า), Format (รูปแบบ)

❌ Error Response (เออเรอร์ รีสปอนส์)	
กำหนดรูปแบบ Error (เออเรอร์) ให้เป็นมาตรฐานเดียวกันทั้งระบบ

📄 Pagination (เพจิเนชัน)	
จำกัดจำนวนข้อมูลที่ส่งกลับ เช่น page, limit หรือ Cursor (เคอร์เซอร์)

🔍 Filtering (ฟิลเตอร์ริง)	
ค้นหาเฉพาะข้อมูลที่ต้องการ เช่น status=active&department=IT

↕️ Sorting (ซอร์ติง)	
กำหนดลำดับข้อมูล เช่น sortBy=createdAt&order=desc

🔄 API Versioning (เอพีไอ เวอร์ชันนิง)	
รองรับ API หลายเวอร์ชัน เช่น /api/v1/users, /api/v2/users

🔁 Idempotency (ไอเด็มโพเทนซี)	
ส่ง Request เดิมซ้ำแล้วไม่ทำให้ผลลัพธ์เกิดซ้ำโดยไม่ตั้งใจ สำคัญมากกับการสร้าง Order / Payment

📚 API Documentation (เอพีไอ ด็อกคิวเมนเทชัน)	
เอกสารที่บอกว่า API แต่ละตัวใช้ทำอะไร รับอะไร และคืนอะไร

📖 OpenAPI / Swagger (โอเพนเอพีไอ / สแวกเกอร์)
มาตรฐานและเครื่องมือสำหรับอธิบาย API รวมถึงทดลองเรียก API ผ่านหน้าเว็บ

