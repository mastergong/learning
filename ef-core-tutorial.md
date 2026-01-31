# Entity Framework Core (EF Core) แบบละเอียด

## บทที่ 1: พื้นฐาน EF Core

### EF Core คืออะไร?
Entity Framework Core (EF Core) เป็น ORM (Object-Relational Mapper) เครื่องมือที่ช่วยให้สามารถจัดการฐานข้อมูลใน C# ได้สะดวก โดยทำหน้าที่เป็นตัวกลางระหว่างโปรแกรมและฐานข้อมูล 

**ข้อดีของการใช้ EF Core**
- ลดความซับซ้อนในการเขียน SQL
- ง่ายต่อการบำรุงรักษาโค้ด
- รองรับหลายฐานข้อมูล เช่น SQL Server, SQLite และ PostgreSQL

### การติดตั้ง EF Core
1. ใช้คำสั่ง NuGet Package Manager:
   ```bash
   Install-Package Microsoft.EntityFrameworkCore
   ```
2. หากใช้ .NET CLI:
   ```bash
   dotnet add package Microsoft.EntityFrameworkCore
   ```

### สร้างโปรเจคใหม่
```bash
dotnet new console -n EfCoreDemo
cd EfCoreDemo
```

### การตั้งค่า DbContext เบื้องต้น
สร้างไฟล์ชื่อ `AppDbContext.cs`
```csharp
using Microsoft.EntityFrameworkCore;

public class AppDbContext : DbContext
{
    public DbSet<Student> Students { get; set; } = null!;

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("Server=localhost;Database=EfCoreDemo;Trusted_Connection=True;");
    }
}

public class Student
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
}
```

### เพิ่มการ Migrate สำหรับฐานข้อมูล
1. เพิ่ม Migration:
   ```bash
   dotnet ef migrations add InitialCreate
   ```
2. อัพเดทฐานข้อมูลตาม Migration:
   ```bash
   dotnet ef database update
   ```

---

## บทที่ 2: การ Query ข้อมูลใน EF Core

### การเพิ่มข้อมูล (Insert)
```csharp
using (var context = new AppDbContext())
{
    var student = new Student { Name = "John Doe" };
    context.Students.Add(student);
    context.SaveChanges();
}
```

### การดึงข้อมูล (Read)
```csharp
using (var context = new AppDbContext())
{
    var students = context.Students.ToList();
    foreach (var student in students)
    {
        Console.WriteLine($"ID: {student.Id}, Name: {student.Name}");
    }
}
```

### การอัพเดตข้อมูล (Update)
```csharp
using (var context = new AppDbContext())
{
    var student = context.Students.First();
    student.Name = "Jane Smith";
    context.SaveChanges();
}
```

### การลบข้อมูล (Delete)
```csharp
using (var context = new AppDbContext())
{
    var student = context.Students.First();
    context.Students.Remove(student);
    context.SaveChanges();
}
}
```

---

## บทที่ 3: Advanced EF Core

### การใช้ LINQ Queries
EF Core รองรับการเขียน Query ด้วย LINQ (Language Integrated Query).
```csharp
using (var context = new AppDbContext())
{
    var filteredStudents = context.Students
        .Where(s => s.Name.Contains("John"))
        .OrderBy(s => s.Name)
        .ToList();

    foreach (var student in filteredStudents) {
        Console.WriteLine(student.Name);
    }
}
```

### การใช้ Eager Loading และ Lazy Loading
#### Eager Loading
ดึงข้อมูลที่เกี่ยวข้องมาพร้อมกัน
```csharp
var studentsWithCourses = context.Students.Include(s => s.Courses).ToList();
```

#### Lazy Loading
โหลดข้อมูลสัมพันธ์ตามความจำเป็น
```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    public virtual ICollection<Course> Courses { get; set; }
}
```

---

**เรียนรู้เพิ่มเติม:** ดูรายละเอียดในเอกสาร [EF Core](https://learn.microsoft.com/en-us/ef/core/)