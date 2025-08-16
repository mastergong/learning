# Architecture 04 — Clean Architecture Walkthrough

จุดหมาย: เดินจาก Use Case สู่ HTTP แบบ end-to-end โดยแยกเลเยอร์ชัดเจน

## 1) Domain: กฎธุรกิจบริสุทธิ์
- Entities: Behavior + Invariants
- Interfaces: สัญญาเพื่อเข้าถึง resource ภายนอก (Repository, Clock, Email)

## 2) Application: Use Cases
- ไม่อิง EF/HTTP
- รับ DTO/Command, ประสานงาน Domain, เรียก Interfaces

## 3) Infrastructure: Implement Interfaces
- EF Core DbContext + Repository
- Adapters: Email/SMS/External APIs

## 4) Web: Composition Root
- Configure DI
- Map Endpoints/Controllers
- Validation/Mapping/Filters

## การไหลของคำสั่ง (Example: CreateProduct)
- Web รับ ProductDto -> validate -> Map -> call CreateProduct.Handle
- Application สร้าง Entity -> เรียก IProductRepository.AddAsync
- Infrastructure บันทึกผ่าน EF Core
- Web ตอบ 201 Created พร้อม resource URI

Checklist ความบริสุทธิ์:
- Domain ไม่รู้จัก EF/HTTP
- Application ทดแทน Infra ได้ด้วย mocks
- Infra ทดสอบแบบ integration
- Web บาง; ไม่ยัด logic ธุรกิจ
