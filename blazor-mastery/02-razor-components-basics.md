# 02 — Razor & Component พื้นฐาน

## Razor Syntax รวบรัด
- @{} โค้ดบล็อก C#
- @variable, @if/@for
- แทรกค่า/ฟอร์แมตด้วย string interpolation

## พารามิเตอร์คอมโพเนนต์
- [Parameter] public string? Title { get; set; }
- [EditorRequired] บังคับให้ผู้ใช้คอมโพเนนต์ใส่ค่า
- EventCallback และ EventCallback<T>

## วงจรชีวิตคอมโพเนนต์
- OnInitialized{Async}, OnParametersSet{Async}, OnAfterRender{Async}
- ป้องกันการเรียก StateHasChanged เกินจำเป็น

## การประกอบคอมโพเนนต์
- RenderFragment / RenderFragment<T>
- ChildContent สำหรับคอนเทนต์ภายใน

## แบบฝึก
- สร้าง CardComponent ที่รองรับ Header/Footer ผ่าน RenderFragment และแสดง Slot แบบกำหนดเอง
