# 10 — ประสิทธิภาพ: SSR, AOT, Trimming

## ปรับปรุง First Load
- SSR/Static SSR + Prerender
- แยก Interactive เฉพาะคอมโพเนนต์ที่จำเป็น

## ขนาดดาวน์โหลด (WASM)
- IL Trimming, Native AOT (PublishAot)
- Lazy load assemblies และ resource

## Rendering ประสิทธิภาพ
- ใช้ @key, Virtualize, แยกคอมโพเนนต์ย่อย

## แบบฝึก
- เปิด AOT สำหรับหน้า Analytics เฉพาะ และวัดผลก่อน/หลัง
