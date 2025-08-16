# 07 — การยืนยันตัวตนและกำหนดสิทธิ์

## ตัวเลือกการพิสูจน์ตัวตน
- ASP.NET Core Identity + Cookie (Blazor Server/SSR)
- OpenID Connect/OAuth2 (Microsoft/Google/Keycloak)
- JWT + API Backend (สำหรับ WASM/แยก Front/Back)

## แนวทาง Blazor Web App
- ใช้ AuthenticationStateProvider เพื่อตรวจสอบผู้ใช้ปัจจุบัน
- @attribute [Authorize] บนเพจ/คอมโพเนนต์
- AuthorizeView แสดง/ซ่อนส่วน UI ตามสิทธิ์

## บทเรียนสำคัญ
- SSR + Interactive Server ใช้คุกกี้/เซสชันได้ง่าย
- Interactive WASM นิยมใช้ JWT เข้าถึง API

## แบบฝึก
- เพิ่มหน้าจัดการโปรไฟล์ที่เข้าถึงได้เฉพาะผู้ใช้ที่ล็อกอิน และหน้า Admin สำหรับ ROLE=Admin
