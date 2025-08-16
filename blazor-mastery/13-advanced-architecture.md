# 13 — สถาปัตยกรรมและแนวปฏิบัติขั้นสูง

## การแยกเลเยอร์และโดเมน
- UI (Blazor) แยกจาก Application/Domain/Infrastructure
- ใช้ Mediator/CQRS เพื่อลด Coupling กับ UI

## Modularization
- แยกฟีเจอร์เป็นโฟลเดอร์/ไลบรารีคอมโพเนนต์
- คอมโพเนนต์ที่นำกลับใช้ได้ (Design System)

## การจัดการ Error/Telemetry
- Global Error Boundary, Logging, OpenTelemetry

## แบบฝึก
- รีแฟกเตอร์แอปเป็นโมดูล Catalog/Ordering/Identity พร้อมเลเยอร์ชัดเจน
