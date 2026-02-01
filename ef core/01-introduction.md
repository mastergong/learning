# บทที่ 1: บทนำและการติดตั้ง Entity Framework Core

## 🎯 จุดประสงค์ของบทนี้

หลังจากเรียนจบบทนี้ คุณจะสามารถ:
- เข้าใจว่า EF Core คืออะไรและทำงานอย่างไร
- ติดตั้งและตั้งค่า EF Core ได้ถูกต้อง
- สร้าง Model และ DbContext พื้นฐาน
- เชื่อมต่อและสร้างฐานข้อมูลแรก
- ใช้คำสั่ง CLI พื้นฐานของ EF Core

---

## Entity Framework Core คืออะไร?

**EF Core** เป็น **Object-Relational Mapping (ORM)** framework สำหรับ .NET ที่ทำหน้าที่เป็น "สะพานเชื่อม" ระหว่าง:

- **C# Objects** (ที่เราเขียนโค้ด) ↔ **Database Tables** (ที่เก็บข้อมูล)

### 🤔 ทำไมต้องใช้ EF Core?

**แทนที่จะเขียนแบบนี้ (SQL ดิบ):**
```sql
SELECT Id, FirstName, LastName, Email 
FROM Students 
WHERE Age > 18
```

**เราเขียนแบบนี้ (C# กับ LINQ):**
```csharp
var students = context.Students
    .Where(s => s.Age > 18)
    .ToList();
```

### ✅ ข้อดีของ EF Core

| ข้อดี | คำอธิบาย |
|-------|----------|
| **Type Safety** | ตรวจสอบ syntax ตอน compile time |
| **IntelliSense** | มี auto-complete ช่วยเขียนโค้ด |
| **LINQ Support** | เขียน query ด้วย C# แทน SQL |
| **Migration** | จัดการการเปลี่ยนแปลง schema อัตโนมัติ |
| **Cross-Platform** | ใช้ได้กับ Windows, Linux, macOS |
| **Multiple Databases** | รองรับ SQL Server, SQLite, PostgreSQL, MySQL |

---

## 🛠️ การติดตั้ง EF Core (Step by Step)

### ขั้นตอนที่ 1: สร้าง Project ใหม่

```bash
# เปิด Terminal หรือ Command Prompt
dotnet new console -n MyFirstEFApp
cd MyFirstEFApp

# เปิดด้วย VS Code (ถ้าต้องการ)
code .
```

### ขั้นตอนที่ 2: ติดตั้ง EF Core Packages

```bash
# Package หลักสำหรับ SQL Server
dotnet add package Microsoft.EntityFrameworkCore.SqlServer

# Tools สำหรับ Migration และ CLI
dotnet add package Microsoft.EntityFrameworkCore.Tools

# Design-time tools (สำคัญสำหรับ migrations)
dotnet add package Microsoft.EntityFrameworkCore.Design
```

### ขั้นตอนที่ 3: ตรวจสอบ .csproj file

หลังติดตั้งเรียบร้อย ไฟล์ `.csproj` ควรมีหน้าตาแบบนี้:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="6.0.0" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="6.0.0" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="6.0.0" />
  </ItemGroup>
</Project>
```

### ขั้นตอนที่ 4: เลือก Database Provider (ทางเลือก)

```bash
# สำหรับ SQLite (เหมาะสำหรับเรียนรู้)
dotnet add package Microsoft.EntityFrameworkCore.Sqlite

# สำหรับ PostgreSQL
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL

# สำหรับ MySQL
dotnet add package Pomelo.EntityFrameworkCore.MySql

# สำหรับการทดสอบ (In-Memory Database)
dotnet add package Microsoft.EntityFrameworkCore.InMemory
```

---

## 📁 โครงสร้างโปรเจคที่แนะนำ

```
MyFirstEFApp/
├── Models/           # 🏠 Entity Classes (ตารางในรูปแบบ C# class)
│   ├── Student.cs
│   └── Course.cs
├── Data/            # 🏢 DbContext และ Configuration
│   └── SchoolContext.cs
├── Migrations/      # 📋 Migration files (สร้างอัตโนมัติ)
├── appsettings.json # ⚙️ Configuration (connection strings)
└── Program.cs       # 🚀 Entry point
```

---

## 💻 สร้างแอป EF Core แรก (Tutorial แบบ Step-by-Step)

### Step 1: สร้าง Model แรก

```csharp
// Models/Student.cs
using System.ComponentModel.DataAnnotations;

namespace MyFirstEFApp.Models
{
    public class Student
    {
        // Primary Key (EF จะรู้อัตโนมัติจากชื่อ Id)
        public int Id { get; set; }
        
        // ชื่อจริง (ห้ามเป็น null, สูงสุด 50 ตัวอักษร)
        [Required]
        [MaxLength(50)]
        public string FirstName { get; set; } = string.Empty;
        
        // นามสกุล
        [Required]  
        [MaxLength(50)]
        public string LastName { get; set; } = string.Empty;
        
        // อีเมล (ต้องเป็น format อีเมลที่ถูกต้อง)
        [Required]
        [EmailAddress]
        [MaxLength(100)]
        public string Email { get; set; } = string.Empty;
        
        // วันเกิด
        public DateTime DateOfBirth { get; set; }
        
        // วันที่สร้างข้อมูล (กำหนดค่าเริ่มต้นเป็นเวลาปัจจุบัน)
        public DateTime CreatedDate { get; set; } = DateTime.Now;
        
        // Computed Property (ไม่ได้เก็บในฐานข้อมูล)
        public int Age => DateTime.Now.Year - DateOfBirth.Year;
    }
}
```

### Step 2: สร้าง DbContext

```csharp
// Data/SchoolContext.cs
using Microsoft.EntityFrameworkCore;
using MyFirstEFApp.Models;

namespace MyFirstEFApp.Data
{
    public class SchoolContext : DbContext
    {
        // Constructor สำหรับรับ options จาก DI
        public SchoolContext(DbContextOptions<SchoolContext> options) 
            : base(options)
        {
        }

        // DbSet = Table ในฐานข้อมูล
        public DbSet<Student> Students { get; set; } = null!;

        // Configuration เพิ่มเติมสำหรับ Models
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            // ตั้งค่าเพิ่มเติมสำหรับ Student table
            modelBuilder.Entity<Student>(entity =>
            {
                // กำหนดชื่อตาราง (ถ้าไม่ระบุจะใช้ชื่อ DbSet)
                entity.ToTable("Students");
                
                // Index สำหรับ Email (เพื่อความเร็วในการค้นหา)
                entity.HasIndex(s => s.Email)
                    .IsUnique(); // Email ต้องไม่ซ้ำ
                
                // กำหนด Default Value
                entity.Property(s => s.CreatedDate)
                    .HasDefaultValueSql("GETDATE()"); // SQL Server syntax
            });
            
            base.OnModelCreating(modelBuilder);
        }
    }
}
```

### Step 3: ตั้งค่า Connection String

```json
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=MyFirstSchoolDB;Trusted_Connection=true;MultipleActiveResultSets=true"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "Microsoft.EntityFrameworkCore.Database.Command": "Information"
    }
  },
  "AllowedHosts": "*"
}
```

### Step 4: ตั้งค่าใน Program.cs

```csharp
// Program.cs
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using MyFirstEFApp.Data;
using MyFirstEFApp.Models;

// สร้าง Host Builder สำหรับ Console App
var host = Host.CreateDefaultBuilder(args)
    .ConfigureServices((context, services) =>
    {
        // อ่าน Connection String จาก appsettings.json
        var connectionString = context.Configuration.GetConnectionString("DefaultConnection");
        
        // ลงทะเบียน DbContext
        services.AddDbContext<SchoolContext>(options =>
        {
            options.UseSqlServer(connectionString);
            
            // เปิด Logging สำหรับดู SQL queries (ใน Development เท่านั้น)
            if (context.HostingEnvironment.IsDevelopment())
            {
                options.EnableSensitiveDataLogging();
                options.EnableDetailedErrors();
            }
        });
    })
    .Build();

// ใช้งาน EF Core
using (var scope = host.Services.CreateScope())
{
    var context = scope.ServiceProvider.GetRequiredService<SchoolContext>();
    
    try
    {
        Console.WriteLine("🚀 เริ่มต้นแอปพลิเคชัน EF Core");
        
        // สร้างฐานข้อมูล (ถ้ายังไม่มี)
        Console.WriteLine("📊 กำลังสร้างฐานข้อมูล...");
        await context.Database.EnsureCreatedAsync();
        Console.WriteLine("✅ สร้างฐานข้อมูลเรียบร้อยแล้ว!");
        
        // ตรวจสอบว่ามีข้อมูลอยู่หรือไม่
        if (!context.Students.Any())
        {
            Console.WriteLine("👤 กำลังเพิ่มข้อมูลนักเรียนตัวอย่าง...");
            
            var students = new List<Student>
            {
                new Student
                {
                    FirstName = "สมชาย",
                    LastName = "ใจดี",
                    Email = "somchai@school.com",
                    DateOfBirth = new DateTime(2000, 5, 15)
                },
                new Student
                {
                    FirstName = "สมหญิง", 
                    LastName = "สุขใจ",
                    Email = "somying@school.com",
                    DateOfBirth = new DateTime(2001, 8, 20)
                },
                new Student
                {
                    FirstName = "วิชัย",
                    LastName = "เก่งเรียน",
                    Email = "vichai@school.com", 
                    DateOfBirth = new DateTime(1999, 12, 3)
                }
            };
            
            context.Students.AddRange(students);
            await context.SaveChangesAsync();
            Console.WriteLine($"✅ เพิ่มข้อมูล {students.Count} คนเรียบร้อยแล้ว!");
        }
        
        // แสดงข้อมูลนักเรียนทั้งหมด
        Console.WriteLine("\n📋 รายชื่อนักเรียนทั้งหมด:");
        Console.WriteLine("".PadRight(60, '='));
        
        var allStudents = await context.Students
            .OrderBy(s => s.FirstName)
            .ToListAsync();
            
        foreach (var student in allStudents)
        {
            Console.WriteLine($"👤 {student.FirstName} {student.LastName}");
            Console.WriteLine($"   📧 {student.Email}");
            Console.WriteLine($"   🎂 อายุ {student.Age} ปี");
            Console.WriteLine($"   📅 สร้างเมื่อ {student.CreatedDate:dd/MM/yyyy HH:mm}");
            Console.WriteLine();
        }
        
        // ตัวอย่างการ Query
        Console.WriteLine("🔍 ค้นหานักเรียนที่อายุมากกว่า 20 ปี:");
        var adultStudents = await context.Students
            .Where(s => s.Age > 20)
            .Select(s => new { s.FirstName, s.LastName, s.Age })
            .ToListAsync();
            
        foreach (var student in adultStudents)
        {
            Console.WriteLine($"   - {student.FirstName} {student.LastName} (อายุ {student.Age} ปี)");
        }
        
        Console.WriteLine("\n🎉 แอปพลิเคชันทำงานเสร็จสิ้น!");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"❌ เกิดข้อผิดพลาด: {ex.Message}");
        if (ex.InnerException != null)
        {
            Console.WriteLine($"   รายละเอียด: {ex.InnerException.Message}");
        }
    }
}

Console.WriteLine("\nกด Enter เพื่อปิดแอป...");
Console.ReadLine();
```

---

## 🎮 ทดลองรันแอปครั้งแรก

### 1. รันแอปพลิเคชัน

```bash
# รันแอป
dotnet run
```

### 2. ผลลัพธ์ที่ควรเห็น

```
🚀 เริ่มต้นแอปพลิเคชัน EF Core
📊 กำลังสร้างฐานข้อมูล...
✅ สร้างฐานข้อมูลเรียบร้อยแล้ว!
👤 กำลังเพิ่มข้อมูลนักเรียนตัวอย่าง...
✅ เพิ่มข้อมูล 3 คนเรียบร้อยแล้ว!

📋 รายชื่อนักเรียนทั้งหมด:
============================================================
👤 สมชาย ใจดี
   📧 somchai@school.com
   🎂 อายุ 23 ปี
   📅 สร้างเมื่อ 01/02/2026 14:30

👤 สมหญิง สุขใจ
   📧 somying@school.com
   🎂 อายุ 22 ปี
   📅 สร้างเมื่อ 01/02/2026 14:30

👤 วิชัย เก่งเรียน
   📧 vichai@school.com
   🎂 อายุ 24 ปี
   📅 สร้างเมื่อ 01/02/2026 14:30

🔍 ค้นหานักเรียนที่อายุมากกว่า 20 ปี:
   - สมชาย ใจดี (อายุ 23 ปี)
   - สมหญิง สุขใจ (อายุ 22 ปี)
   - วิชัย เก่งเรียน (อายุ 24 ปี)

🎉 แอปพลิเคชันทำงานเสร็จสิ้น!
```

---

## 🔧 คำสั่ง EF Core CLI พื้นฐาน

เมื่อติดตั้งเรียบร้อยแล้ว ลองใช้คำสั่งเหล่านี้:

```bash
# ดูเวอร์ชัน EF Core CLI
dotnet ef --version

# ดูคำสั่งทั้งหมด
dotnet ef --help

# สร้าง Migration แรก (สำหรับบทต่อไป)
dotnet ef migrations add InitialCreate

# อัปเดตฐานข้อมูลจาก Migration
dotnet ef database update

# ดูข้อมูล DbContext
dotnet ef dbcontext info

# ดูรายการ Migrations
dotnet ef migrations list
```

---

## ❗ การแก้ปัญหาเบื้องต้น

### ปัญหาที่เจอบ่อย:

#### 1. Connection String ผิด
```
❌ Error: Cannot connect to database
```
**แก้ไข:** ตรวจสอบ Connection String ใน `appsettings.json`

#### 2. Missing Package
```
❌ Error: The type or namespace name 'DbContext' could not be found
```
**แก้ไข:** ติดตั้ง package ที่ขาด:
```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
```

#### 3. Missing appsettings.json
```
❌ Error: Configuration section 'ConnectionStrings' not found
```
**แก้ไข:** สร้างไฟล์ `appsettings.json` และตั้งค่า Connection String

---

## ✅ สิ่งที่คุณได้เรียนรู้ในบทนี้

- [x] เข้าใจหลักการทำงานของ EF Core  
- [x] ติดตั้งและตั้งค่า EF Core ในโปรเจค
- [x] สร้าง Model class พื้นฐานพร้อม Data Annotations
- [x] สร้าง DbContext และตั้งค่า Entity Configuration
- [x] เชื่อมต่อฐานข้อมูลและสร้างตารางอัตโนมัติ
- [x] เพิ่ม/แสดง/ค้นหาข้อมูลพื้นฐาน
- [x] ใช้คำสั่ง CLI พื้นฐาน

---

## 🚀 เตรียมพร้อมสำหรับบทถัดไป

ในบทถัดไป เราจะเรียนรู้เกี่ยวกับ:
- **Data Annotations** แบบละเอียด
- **ชนิดข้อมูลต่างๆ** ที่ EF Core รองรับ  
- **การตั้งค่า Validation** ที่ซับซ้อนขึ้น
- **Custom Attributes** และการใช้งาน

💡 **เคล็ดลับ:** ลองแก้ไขโค้ดและรันใหม่เพื่อเข้าใจการทำงาน เช่น เปลี่ยนชื่อ property หรือเพิ่มข้อมูลใหม่

---

**หัวข้อถัดไป:** [02 - ชนิดข้อมูลและ Data Annotations](02-data-types.md)
using EFCoreDemo.Models;

namespace EFCoreDemo.Data
{
    public class SchoolContext : DbContext
    {
        public SchoolContext(DbContextOptions<SchoolContext> options) 
            : base(options)
        {
        }

        // DbSet properties
        public DbSet<Student> Students { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            // Configuration จะเขียนที่นี่
            base.OnModelCreating(modelBuilder);
        }
    }
}
```

### 3. ตั้งค่า Connection String

```csharp
// Program.cs
using Microsoft.EntityFrameworkCore;
using EFCoreDemo.Data;

var builder = WebApplication.CreateBuilder(args);

// เพิ่ม DbContext
builder.Services.AddDbContext<SchoolContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// ตัวอย่างการใช้งานพื้นฐาน
using (var scope = app.Services.CreateScope())
{
    var context = scope.ServiceProvider.GetRequiredService<SchoolContext>();
    
    // สร้างฐานข้อมูลหากไม่มี
    context.Database.EnsureCreated();
    
    // ตัวอย่างการเพิ่มข้อมูล
    if (!context.Students.Any())
    {
        context.Students.Add(new Student
        {
            FirstName = "สมชาย",
            LastName = "ใจดี",
            Email = "somchai@example.com",
            DateOfBirth = new DateTime(2000, 1, 1)
        });
        
        context.SaveChanges();
    }
}

app.Run();
```

### 4. ไฟล์ appsettings.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=SchoolDB;Trusted_Connection=true;MultipleActiveResultSets=true"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

## คำสั่งพื้นฐานสำหรับ EF Core CLI

```bash
# ดู CLI version
dotnet ef --version

# เพิ่ม Migration ใหม่
dotnet ef migrations add InitialCreate

# อัปเดตฐานข้อมูล
dotnet ef database update

# ดู Migration list
dotnet ef migrations list

# สร้าง SQL Script จาก Migration
dotnet ef migrations script

# ลบ Migration ล่าสุด
dotnet ef migrations remove
```

## การตรวจสอบการติดตั้ง

เพื่อตรวจสอบว่าการติดตั้งสำเร็จ ให้รันคำสั่งต่อไปนี้:

```bash
dotnet ef --version
```

หากแสดงหมายเลข version แสดงว่าติดตั้งสำเร็จแล้ว

## สิ่งที่ต้องเรียนรู้ต่อไป

หลังจากติดตั้งเรียบร้อยแล้ว ให้ศึกษาหัวข้อถัดไป:
- ชนิดข้อมูลและ Data Annotations
- การสร้าง Models ขั้นสูง
- การจัดการ Relationships

---

**หมายเหตุ**: ตัวอย่างโค้ดทั้งหมดในคู่มือนี้ใช้ .NET 6+ และ EF Core 6+