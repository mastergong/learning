# บทที่ 1 — แนะนำ .NET 9 และภาษา C#

เป้าหมาย: เข้าใจแพลตฟอร์ม .NET, แนวคิดภาษา C#, และแนวทางโปรเจกต์พื้นฐานที่ใช้ในคอร์สนี้

## ภาพรวม .NET
- Cross-platform: Windows, Linux, macOS
- ประเภทแอป: Console, Web (ASP.NET Core), APIs, Desktop (WinUI/WPF), Cloud/Serverless, Workers
- SDK & CLI: `dotnet new`, `dotnet build`, `dotnet run`, `dotnet test`

## ภาษา C# สาระสำคัญที่ใช้บ่อย
- Strong typing, Generics, LINQ, async/await, Records, Pattern Matching
- Collections: List<T>, Dictionary<TKey,TValue>, HashSet<T>, Span<T> (ขั้นสูง)

## โครงสร้างโปรเจกต์ Console เริ่มต้น
```csharp
// Program.cs
using System;

Console.WriteLine("Hello, C# on .NET 9!");
```

รัน:
```
dotnet new console -n HelloDotNet
cd HelloDotNet
dotnet run
```

## ตัวอย่าง Unit Test พื้นฐาน (xUnit)
```csharp
// ExampleTests.cs
using Xunit;

public class ExampleTests
{
    [Fact]
    public void Add_TwoNumbers_ReturnsSum()
    {
        int Add(int a, int b) => a + b;
        Assert.Equal(5, Add(2, 3));
    }
}
```

## ต่อไป
- โครงสร้างข้อมูลและอัลกอริทึม → `ds-algorithms/01-big-o.md`
- หลักการ SOLID และดีไซน์แพทเทิร์น → `design-patterns-solid/01-solid-principles.md`
- MVC และ Clean Architecture → `architecture/01-mvc.md`
