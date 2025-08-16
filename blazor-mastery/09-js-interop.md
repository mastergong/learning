# 09 — JavaScript Interop (C# <-> JS)

## พื้นฐาน
- IJSRuntime.InvokeAsync
- JS Modules: IJSObjectReference (import)

## เรียก C# จาก JS
- [JSInvokable] บน static method
- ใช้ DotNetObjectReference เพื่อเรียก instance method

## กรณีใช้งาน
- Clipboard, LocalStorage, Third-party Widgets, Charts

## แบบฝึก
- เขียนโมดูล JS สำหรับ Toast Notification และเรียกใช้จาก C# พร้อม Callback
