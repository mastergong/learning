# 06 — การจัดการสถานะ (State Management)

## ระดับของสถานะ
- ภายในคอมโพเนนต์: ตัวแปรภายใน
- ระดับเพจ/ฟีเจอร์: State Container (Scoped Service)
- ระดับแอป: DI Singleton, LocalStorage/SessionStorage

## เทคนิคสำคัญ
- CascadingValue/Parameter สำหรับ Context/Theme/User
- ProtectedBrowserStorage/LocalStorage ผ่าน JS
- Flux/Redux ด้วย Fluxor สำหรับแอปขนาดใหญ่

## ปัญหาที่พบบ่อย
- State หายเมื่อรีเฟรช (WASM) — Persist ลง Storage
- การ Notify UI — ใช้ INotifyPropertyChanged/StateHasChanged อย่างเหมาะสม

## แบบฝึก
- สร้าง CartState ที่คงอยู่ข้ามเพจและ Persist ลง LocalStorage
