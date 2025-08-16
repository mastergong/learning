# 03 — Routing, Layout และ Navigation

## Routing
- @page "/products"
- พารามิเตอร์เส้นทาง: @page "/products/{id:int}"
- Query string: NavigationManager.ToAbsoluteUri(Uri).Query

## Layout
- สร้าง Shared/MainLayout.razor ใช้ @Body สำหรับเนื้อหาเพจ
- Nested Layout และ Section-like ด้วย RenderFragment

## Navigation
- NavigationManager.NavigateTo("/cart")
- ป้องกันออกจากหน้าเมื่อมีการแก้ไข: ใช้ JS Interop หรือ ConfirmDialog

## แบบฝึก
- สร้างโครงสร้าง Layout พร้อมเมนูด้านข้างและ Breadcrumbs
- ทำหน้ารายการสินค้า + หน้ารายละเอียดที่แมปเส้นทางด้วย id
