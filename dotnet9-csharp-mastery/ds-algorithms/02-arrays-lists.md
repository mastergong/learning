# DS&A 02 — Arrays, List<T>, และ Span<T>

## Arrays vs List<T>
- Array: ขนาดคงที่, index O(1), insert/delete กลาง O(n)
- List<T>: ขยายได้, amortized append O(1), insert/delete กลาง O(n)

## ตัวอย่าง C#
```csharp
// Array
int[] a = new int[5];
for (int i = 0; i < a.Length; i++) a[i] = i * i;

// List<T>
var list = new List<int>();
for (int i = 0; i < 5; i++) list.Add(i * i);

Console.WriteLine(string.Join(",", a));
Console.WriteLine(string.Join(",", list));
```

## Span<T> (ขั้นสูง)
- ทำงานบน stack/managed memory โดยไม่ allocate ใหม่
- ใช้เมื่อต้องการ performance สูงในการประมวลผล slice ของ buffer

```csharp
Span<int> s = stackalloc int[5];
for (int i = 0; i < s.Length; i++) s[i] = i * i;
var slice = s.Slice(1, 3);
foreach (var x in slice) Console.WriteLine(x);
```

## แนวปฏิบัติ
- ใช้ `List<T>` โดยค่าเริ่มต้น หากรู้ขนาดแน่นอนใช้ `new List<T>(capacity)`
- ประหยัด allocation โดยใช้ `Span<T>`/`ReadOnlySpan<T>` ในงานประมวลผลข้อมูลดิบ
