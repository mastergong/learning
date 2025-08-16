# DS&A 01 — Big-O และการวิเคราะห์เวลา/หน่วยความจำ

## ทำไมต้อง Big-O
อธิบายการเติบโตของเวลา/หน่วยความจำตามขนาดข้อมูล n เพื่อเปรียบเทียบอัลกอริทึมโดยไม่ขึ้นกับฮาร์ดแวร์

## ตารางชั้นเชิงความซับซ้อน (โดยสังเขป)
- O(1): เวลาคงที่
- O(log n): Binary Search
- O(n): Linear scan
- O(n log n): Merge/Quick Sort โดยเฉลี่ย
- O(n^2): Nested loops
- O(2^n)/O(n!): บางปัญหาแบบ exhaustive

## ตัวอย่าง C# วัดเวลาแบบง่าย
```csharp
using System.Diagnostics;

static long Measure(Action action)
{
    var sw = Stopwatch.StartNew();
    action();
    sw.Stop();
    return sw.ElapsedMilliseconds;
}

var n = 1_000_000;
var arr = Enumerable.Range(0, n).ToArray();

var t1 = Measure(() => {
    // O(n)
    var sum = 0L;
    for (int i = 0; i < arr.Length; i++) sum += arr[i];
});

Console.WriteLine($"O(n) took: {t1} ms");
```

## เคล็ดลับ
- เลือกโครงสร้างข้อมูลให้เหมาะสม ลดความซับซ้อนจาก O(n) → O(1)/O(log n)
- ระวัง Boxing/Allocations ไม่จำเป็นใน .NET (ใช้ struct/Span<T> เมื่อเหมาะสม)
