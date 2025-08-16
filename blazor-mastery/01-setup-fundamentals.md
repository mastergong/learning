# 01 — เริ่มต้นโปรเจกต์และเข้าใจโครงสร้าง

## สร้างโปรเจกต์ Blazor Web App
แนวคิด: ใช้เทมเพลต Blazor Web App เพื่อได้ SSR + Interactive

องค์ประกอบหลัก:
- Program.cs: กำหนดบริการ DI, Routing, RenderMode
- App.razor: โครงสร้าง Router และ Layout
- MainLayout.razor: กรอบหน้าเว็บ (NavMenu, Body)

ตัวอย่าง Router ใน App.razor:
- ใช้ <Router> เพื่อแมปเส้นทางไปยังคอมโพเนนต์
- เพิ่ม NotFound/Loading UI

## โครงสร้างไฟล์คอมโพเนนต์
- .razor: Markup + C# (ใช้ @code)
- .razor.cs: Partial class แยก C# ออกจาก UI เพื่อความเป็นระเบียบ
- @inject สำหรับ DI ภายในคอมโพเนนต์

## Render Modes
- @rendermode InteractiveServer / InteractiveWebAssembly / Auto
- Prerender='true' เพื่อ SSR แรกและค่อยเชื่อมต่อ interactivity

## แบบฝึก
- เพิ่มหน้าที่มี @page "/counterx" พร้อมปุ่ม +1 และรีเซ็ต
- ย้ายโค้ด C# ไปไว้ใน .razor.cs และทำให้ทดสอบง่ายขึ้น
