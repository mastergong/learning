# 00 — ภาพรวม Blazor และการเตรียมสภาพแวดล้อม

## Blazor คืออะไร
Blazor คือเฟรมเวิร์ก UI ของ .NET ที่ให้คุณเขียนเว็บด้วย C# และ Razor Components แทนที่จะพึ่ง JavaScript เป็นหลัก รองรับ 3 โหมดสำคัญ:
- Blazor Web App (แนะนำ): ใช้ SSR + Interactive Render Modes (Server/WASM/Auto)
- Blazor Server: เรนเดอร์บนเซิร์ฟเวอร์ สื่อสารผ่าน SignalR (Latency ต่ำ, โหลดเร็ว)
- Blazor WebAssembly: รัน C# บนเบราว์เซอร์ (ทำงานออฟไลน์ได้, CDN/Static Hosting)

.NET 8/9 ทำให้ Blazor Web App เป็นดีฟอลต์ที่ยืดหยุ่นด้วยโหมดเรนเดอร์:
- SSR/Static SSR: เร็วสำหรับ First Paint และ SEO ดี
- Interactive Server: ลดดาวน์โหลด, Interactivity ผ่าน SignalR
- Interactive WASM: ออฟไลน์/Edge/ไม่มีการเชื่อมต่อเซิร์ฟเวอร์
- Auto: สลับตามสภาพแวดล้อมอัตโนมัติ

## ติดตั้งเครื่องมือ
- .NET 9 SDK
- IDE: Visual Studio 2022 (17.10+) หรือ VS Code + C# Dev Kit
- Browser DevTools, Node.js (ตัวเลือก), Git

ตรวจสอบเวอร์ชัน
- dotnet --info
- ถ้าใช้ VS Code ให้ติดตั้งส่วนเสริม: C#, C# Dev Kit, Razor

## สร้างโปรเจกต์แรก
เทมเพลตแนะนำ: Blazor Web App
- โครงสร้างสำคัญ:
  - App.razor / Program.cs — จุดเริ่มแอป
  - Components/ — คอมโพเนนต์ Razor
  - Pages/ — หน้าเพจที่มี @page
  - wwwroot/ — ไฟล์สาธารณะ (CSS/JS/ภาพ)
  - AppSettings* — ค่ากำหนด

## แนวคิดหลักที่ต้องรู้
- Razor Component: ไฟล์ .razor + C# partial class (ถ้าต้องการ)
- Parameter, EventCallback, CascadingValue
- DI ผ่าน IServiceCollection + [Inject]
- Routing และ Layout
- Render Modes และ Prerendering

## แบบฝึก
1) สร้าง Blazor Web App และเพิ่มหน้าใหม่ /hello ที่รับพารามิเตอร์ name และแสดงเวลาแบบ real-time
2) เพิ่มปุ่มสลับธีม (light/dark) โดยเก็บสถานะไว้ใน localStorage

## คำแนะนำ
- เลือกโหมดเรนเดอร์ตามความต้องการ SEO/Latency/ออฟไลน์
- เริ่มจาก SSR + Interactive Server แล้วเพิ่ม WASM เฉพาะหน้าที่ต้องการออฟไลน์/Edge
