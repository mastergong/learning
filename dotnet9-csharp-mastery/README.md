# C# for .NET 9 — Zero to Hero

เรียน C# และ .NET 9 อย่างเป็นระบบ ครอบคลุม Data Structures & Algorithms, Design Patterns / SOLID, และ MVC + Clean Architecture พร้อมตัวอย่างโค้ด C# และคำอธิบายละเอียด

## โครงสร้างหลักของคอร์ส

- Data Structures & Algorithms (`ds-algorithms/`)
  - Big-O, Arrays/Lists, Stacks/Queues, Linked Lists, Hash Tables
  - Trees/BST, Heaps/PriorityQueue, Graphs (BFS/DFS), Sorting, Searching, Dynamic Programming (พื้นฐาน)
- Design Patterns / SOLID (`design-patterns-solid/`)
  - SOLID Principles
  - Creational, Structural, Behavioral Patterns
  - Practical patterns ใน .NET (DI, Options, Logging, Pipelines)
- MVC & Clean Architecture (`architecture/`)
  - ASP.NET Core MVC (Minimal Hosting, Controllers, Views, Model Binding)
  - Clean Architecture (Domain, Application, Infrastructure, Web)
  - Dependency Injection และ Testing Strategy

เริ่มที่บทนำ: `01-intro-dotnet-and-csharp.md` และตามด้วยหัวข้อย่อยในแต่ละโฟลเดอร์

## เครื่องมือที่ใช้ (แนะนำ)
- .NET SDK 9.x
- C# (12/13 ตาม SDK) + Visual Studio 2022 / VS Code + C# Dev Kit
- xUnit / NUnit + FluentAssertions

## วิธีรันตัวอย่าง (ตัวอย่างโปรเจกต์จะอ้างอิง .NET 9)
- บทเรียนส่วนใหญ่เป็น .md + โค้ดตัวอย่างที่คอมไพล์ได้ คุณสามารถคัดลอกไปใส่ในโปรเจกต์ Console/MVC ตามบทเรียน
- โปรเจกต์ตัวอย่าง MVC ดูที่ `architecture/01-mvc.md`

> หมายเหตุ: โค้ดตัวอย่างพยายามใช้ API มาตรฐานของ .NET เพื่อให้ทำงานได้ข้ามเวอร์ชัน หากมีฟีเจอร์ใหม่ของ .NET 9 จะมีหมายเหตุประกอบ
