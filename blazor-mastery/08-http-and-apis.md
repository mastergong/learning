# 08 — การเชื่อมต่อ Backend: HTTP/REST/gRPC

## HttpClient
- ลงทะเบียน Typed Client/Named Client
- นโยบาย Resilience ด้วย Polly (Retry/Timeout/Circuit Breaker)

## REST API
- จัดการโทเค็น (Bearer) สำหรับ WASM
- จัดรูปแบบ DTO/ViewModel แยกจาก Entity

## gRPC-Web (WASM)
- ใช้ gRPC-Web Proxy/Service; เหมาะงานแบบสตรีม/ประสิทธิภาพ

## แบบฝึก
- ผูกตารางสินค้าเข้ากับ REST API พร้อมโหลดแบบ Lazy, Error/Empty State ชัดเจน
