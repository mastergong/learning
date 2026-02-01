# บทที่ 6: Primary Key และ Foreign Key

## Primary Key (กุญแจหลัก)

Primary Key เป็นคอลัมน์หรือชุดของคอลัมน์ที่ใช้ระบุข้อมูลแต่ละแถวในตารางอย่างไม่ซ้ำกัน

### 1. Single Primary Key

#### แบบ Identity (Auto Increment)

```csharp
public class Student
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int Id { get; set; }  // Auto increment

    public string FirstName { get; set; }
    public string LastName { get; set; }
}

// หรือใช้ Fluent API
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Student>()
        .HasKey(s => s.Id);
        
    modelBuilder.Entity<Student>()
        .Property(s => s.Id)
        .ValueGeneratedOnAdd(); // Auto increment
}
```

#### แบบ GUID

```csharp
public class Product
{
    [Key]
    public Guid Id { get; set; } = Guid.NewGuid();

    public string Name { get; set; }
    public decimal Price { get; set; }
}

// Fluent API
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Product>()
        .HasKey(p => p.Id);
        
    modelBuilder.Entity<Product>()
        .Property(p => p.Id)
        .HasDefaultValueSql("NEWID()"); // SQL Server GUID generation
}
```

#### แบบ String Primary Key

```csharp
public class Country
{
    [Key]
    [StringLength(3)]
    public string CountryCode { get; set; }  // เช่น "THA", "USA"

    [Required]
    [StringLength(100)]
    public string CountryName { get; set; }

    public string? Region { get; set; }
}

// Fluent API
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Country>()
        .HasKey(c => c.CountryCode);
        
    modelBuilder.Entity<Country>()
        .Property(c => c.CountryCode)
        .ValueGeneratedNever(); // ไม่ auto generate
}
```

### 2. Composite Primary Key

Primary Key ที่ประกอบด้วยหลายคอลัมน์

```csharp
public class StudentCourse
{
    [Key, Column(Order = 0)]
    public int StudentId { get; set; }

    [Key, Column(Order = 1)]
    public int CourseId { get; set; }

    public DateTime EnrollmentDate { get; set; }
    public string? Grade { get; set; }

    // Navigation Properties
    [ForeignKey("StudentId")]
    public virtual Student Student { get; set; }

    [ForeignKey("CourseId")]
    public virtual Course Course { get; set; }
}

// หรือใช้ Fluent API (แนะนำ)
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<StudentCourse>()
        .HasKey(sc => new { sc.StudentId, sc.CourseId });
        
    // กำหนด Foreign Key relationships
    modelBuilder.Entity<StudentCourse>()
        .HasOne(sc => sc.Student)
        .WithMany(s => s.StudentCourses)
        .HasForeignKey(sc => sc.StudentId);
        
    modelBuilder.Entity<StudentCourse>()
        .HasOne(sc => sc.Course)
        .WithMany(c => c.StudentCourses)
        .HasForeignKey(sc => sc.CourseId);
}
```

### 3. ไม่มี Primary Key (Keyless Entities)

สำหรับ Views หรือ Query Results

```csharp
[Keyless]
public class StudentSummaryView
{
    public int StudentId { get; set; }
    public string FullName { get; set; }
    public string DepartmentName { get; set; }
    public int EnrolledCourses { get; set; }
    public decimal? GPA { get; set; }
}

// DbContext
public DbSet<StudentSummaryView> StudentSummaries { get; set; }

// Configuration
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<StudentSummaryView>()
        .HasNoKey()
        .ToView("vw_StudentSummary");  // Map to database view
}
```

## Foreign Key (กุญแจต่างประเทศ)

Foreign Key ใช้สร้างความสัมพันธ์ระหว่างตาราง

### 1. การกำหนด Foreign Key ด้วย Data Annotations

```csharp
public class Order
{
    public int Id { get; set; }
    public DateTime OrderDate { get; set; }
    public decimal TotalAmount { get; set; }

    // Foreign Key
    [ForeignKey("Customer")]
    public int CustomerId { get; set; }

    [ForeignKey("Employee")]
    public int? EmployeeId { get; set; }  // Nullable FK

    // Navigation Properties
    public virtual Customer Customer { get; set; }
    public virtual Employee? Employee { get; set; }

    // Collection navigation
    public virtual ICollection<OrderItem> OrderItems { get; set; } = new List<OrderItem>();
}

public class Customer
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }

    // Reverse navigation
    public virtual ICollection<Order> Orders { get; set; } = new List<Order>();
}

public class OrderItem
{
    public int Id { get; set; }
    
    [ForeignKey("Order")]
    public int OrderId { get; set; }
    
    [ForeignKey("Product")]
    public int ProductId { get; set; }
    
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }

    // Navigation Properties
    public virtual Order Order { get; set; }
    public virtual Product Product { get; set; }
}
```

### 2. การกำหนดด้วย Fluent API

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Order -> Customer (Required relationship)
    modelBuilder.Entity<Order>()
        .HasOne(o => o.Customer)
        .WithMany(c => c.Orders)
        .HasForeignKey(o => o.CustomerId)
        .OnDelete(DeleteBehavior.Restrict); // ป้องกันการลบ Customer ที่มี Order

    // Order -> Employee (Optional relationship)
    modelBuilder.Entity<Order>()
        .HasOne(o => o.Employee)
        .WithMany(e => e.Orders)
        .HasForeignKey(o => o.EmployeeId)
        .OnDelete(DeleteBehavior.SetNull); // ตั้งค่า null เมื่อลบ Employee

    // OrderItem -> Order
    modelBuilder.Entity<OrderItem>()
        .HasOne(oi => oi.Order)
        .WithMany(o => o.OrderItems)
        .HasForeignKey(oi => oi.OrderId)
        .OnDelete(DeleteBehavior.Cascade); // ลบ OrderItem เมื่อลบ Order

    // OrderItem -> Product
    modelBuilder.Entity<OrderItem>()
        .HasOne(oi => oi.Product)
        .WithMany(p => p.OrderItems)
        .HasForeignKey(oi => oi.ProductId)
        .OnDelete(DeleteBehavior.Restrict);
}
```

### 3. Multiple Foreign Keys ไปยังตารางเดียวกัน

```csharp
public class Match
{
    public int Id { get; set; }
    public DateTime MatchDate { get; set; }
    public int HomeTeamScore { get; set; }
    public int AwayTeamScore { get; set; }

    // Multiple FKs to same table
    [ForeignKey("HomeTeam")]
    public int HomeTeamId { get; set; }

    [ForeignKey("AwayTeam")]
    public int AwayTeamId { get; set; }

    // Navigation Properties
    public virtual Team HomeTeam { get; set; }
    public virtual Team AwayTeam { get; set; }
}

public class Team
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string City { get; set; }

    // Reverse navigation - ต้องตั้งชื่อให้ต่างกัน
    [InverseProperty("HomeTeam")]
    public virtual ICollection<Match> HomeMatches { get; set; } = new List<Match>();

    [InverseProperty("AwayTeam")]
    public virtual ICollection<Match> AwayMatches { get; set; } = new List<Match>();
}

// Fluent API Configuration
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Match>()
        .HasOne(m => m.HomeTeam)
        .WithMany(t => t.HomeMatches)
        .HasForeignKey(m => m.HomeTeamId)
        .OnDelete(DeleteBehavior.Restrict);

    modelBuilder.Entity<Match>()
        .HasOne(m => m.AwayTeam)
        .WithMany(t => t.AwayMatches)
        .HasForeignKey(m => m.AwayTeamId)
        .OnDelete(DeleteBehavior.Restrict);
}
```

## การจัดการ Delete Behavior

### ประเภทของ Delete Behavior

```csharp
public enum DeleteBehavior
{
    Cascade,      // ลบ Parent จะลบ Child ด้วย
    Restrict,     // ป้องกันการลบ Parent ถ้ามี Child
    SetNull,      // ตั้งค่า FK เป็น null เมื่อลบ Parent
    NoAction      // ไม่ทำอะไร (ให้ Database จัดการ)
}
```

### ตัวอย่างการใช้งาน

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Cascade: ลบ Student จะลบ Enrollments ด้วย
    modelBuilder.Entity<Student>()
        .HasMany(s => s.Enrollments)
        .WithOne(e => e.Student)
        .HasForeignKey(e => e.StudentId)
        .OnDelete(DeleteBehavior.Cascade);

    // Restrict: ป้องกันการลบ Course ถ้ามี Enrollment
    modelBuilder.Entity<Course>()
        .HasMany(c => c.Enrollments)
        .WithOne(e => e.Course)
        .HasForeignKey(e => e.CourseId)
        .OnDelete(DeleteBehavior.Restrict);

    // SetNull: ลบ Department ตั้งค่า DepartmentId เป็น null
    modelBuilder.Entity<Department>()
        .HasMany(d => d.Students)
        .WithOne(s => s.Department)
        .HasForeignKey(s => s.DepartmentId)
        .OnDelete(DeleteBehavior.SetNull);
}
```

## การสร้าง Custom Primary Key Generator

### 1. Custom String Key Generator

```csharp
public class Invoice
{
    [Key]
    [StringLength(20)]
    public string InvoiceNumber { get; set; }  // เช่น "INV-2024-001"

    public DateTime InvoiceDate { get; set; }
    public decimal Amount { get; set; }

    public Invoice()
    {
        InvoiceNumber = GenerateInvoiceNumber();
    }

    private static string GenerateInvoiceNumber()
    {
        var year = DateTime.Now.Year;
        var randomPart = Random.Shared.Next(1000, 9999);
        return $"INV-{year}-{randomPart}";
    }
}
```

### 2. Sequential Custom Key

```csharp
public class CustomKeyGenerator
{
    private readonly ApplicationDbContext _context;

    public CustomKeyGenerator(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<string> GenerateStudentNumberAsync()
    {
        var year = DateTime.Now.Year;
        var yearPrefix = year.ToString().Substring(2); // เอา 2 หลักสุดท้าย

        // หา Student Number สูงสุดในปีนี้
        var lastStudentNumber = await _context.Students
            .Where(s => s.StudentNumber.StartsWith(yearPrefix))
            .OrderByDescending(s => s.StudentNumber)
            .Select(s => s.StudentNumber)
            .FirstOrDefaultAsync();

        int nextNumber = 1;
        if (lastStudentNumber != null)
        {
            var lastNumber = int.Parse(lastStudentNumber.Substring(2));
            nextNumber = lastNumber + 1;
        }

        return $"{yearPrefix}{nextNumber:D4}"; // เช่น "240001"
    }

    public async Task<string> GenerateOrderNumberAsync()
    {
        var today = DateTime.Today;
        var datePrefix = today.ToString("yyyyMMdd");

        var lastOrderNumber = await _context.Orders
            .Where(o => o.OrderNumber.StartsWith(datePrefix))
            .OrderByDescending(o => o.OrderNumber)
            .Select(o => o.OrderNumber)
            .FirstOrDefaultAsync();

        int nextSequence = 1;
        if (lastOrderNumber != null)
        {
            var sequencePart = lastOrderNumber.Substring(8);
            nextSequence = int.Parse(sequencePart) + 1;
        }

        return $"{datePrefix}{nextSequence:D4}"; // เช่น "202401150001"
    }
}

// การใช้งาน
public class OrderService
{
    private readonly ApplicationDbContext _context;
    private readonly CustomKeyGenerator _keyGenerator;

    public OrderService(ApplicationDbContext context, CustomKeyGenerator keyGenerator)
    {
        _context = context;
        _keyGenerator = keyGenerator;
    }

    public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
    {
        var order = new Order
        {
            OrderNumber = await _keyGenerator.GenerateOrderNumberAsync(),
            CustomerId = request.CustomerId,
            OrderDate = DateTime.Now,
            TotalAmount = request.TotalAmount
        };

        _context.Orders.Add(order);
        await _context.SaveChangesAsync();

        return order;
    }
}
```

## การตรวจสอบ Referential Integrity

### 1. Custom Validation

```csharp
public class ReferentialIntegrityService
{
    private readonly ApplicationDbContext _context;

    public ReferentialIntegrityService(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<bool> CanDeleteStudentAsync(int studentId)
    {
        // ตรวจสอบว่ามี Enrollment หรือไม่
        var hasEnrollments = await _context.Enrollments
            .AnyAsync(e => e.StudentId == studentId);

        return !hasEnrollments;
    }

    public async Task<bool> CanDeleteDepartmentAsync(int departmentId)
    {
        // ตรวจสอบว่ามี Student หรือ Course หรือไม่
        var hasStudents = await _context.Students
            .AnyAsync(s => s.DepartmentId == departmentId);

        var hasCourses = await _context.Courses
            .AnyAsync(c => c.DepartmentId == departmentId);

        return !hasStudents && !hasCourses;
    }

    public async Task<List<string>> GetDependenciesAsync(string tableName, int id)
    {
        var dependencies = new List<string>();

        switch (tableName.ToLower())
        {
            case "student":
                if (await _context.Enrollments.AnyAsync(e => e.StudentId == id))
                    dependencies.Add("Enrollments");
                break;

            case "course":
                if (await _context.Enrollments.AnyAsync(e => e.CourseId == id))
                    dependencies.Add("Enrollments");
                break;

            case "department":
                if (await _context.Students.AnyAsync(s => s.DepartmentId == id))
                    dependencies.Add("Students");
                if (await _context.Courses.AnyAsync(c => c.DepartmentId == id))
                    dependencies.Add("Courses");
                break;
        }

        return dependencies;
    }
}
```

### 2. Soft Delete Implementation

```csharp
public abstract class SoftDeletableEntity
{
    public bool IsDeleted { get; set; } = false;
    public DateTime? DeletedDate { get; set; }
    public string? DeletedBy { get; set; }
}

public class Student : SoftDeletableEntity
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    // ... other properties
}

// DbContext with Global Query Filter
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Global filter สำหรับ Soft Delete
    modelBuilder.Entity<Student>()
        .HasQueryFilter(s => !s.IsDeleted);

    modelBuilder.Entity<Course>()
        .HasQueryFilter(c => !c.IsDeleted);
}

// Soft Delete Service
public class SoftDeleteService
{
    private readonly ApplicationDbContext _context;

    public SoftDeleteService(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task SoftDeleteStudentAsync(int studentId, string deletedBy)
    {
        var student = await _context.Students
            .IgnoreQueryFilters() // ไม่ใช้ Global Filter
            .FirstOrDefaultAsync(s => s.Id == studentId);

        if (student != null && !student.IsDeleted)
        {
            student.IsDeleted = true;
            student.DeletedDate = DateTime.Now;
            student.DeletedBy = deletedBy;

            await _context.SaveChangesAsync();
        }
    }

    public async Task RestoreStudentAsync(int studentId)
    {
        var student = await _context.Students
            .IgnoreQueryFilters()
            .FirstOrDefaultAsync(s => s.Id == studentId);

        if (student != null && student.IsDeleted)
        {
            student.IsDeleted = false;
            student.DeletedDate = null;
            student.DeletedBy = null;

            await _context.SaveChangesAsync();
        }
    }
}
```

## การใช้งาน Shadow Properties สำหรับ Keys

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Shadow Property สำหรับ Audit
    modelBuilder.Entity<Student>()
        .Property<DateTime>("CreatedDate")
        .HasDefaultValueSql("GETDATE()");

    modelBuilder.Entity<Student>()
        .Property<string>("CreatedBy")
        .HasMaxLength(100);

    modelBuilder.Entity<Student>()
        .Property<DateTime?>("UpdatedDate");

    modelBuilder.Entity<Student>()
        .Property<string>("UpdatedBy")
        .HasMaxLength(100);

    // Index บน Shadow Property
    modelBuilder.Entity<Student>()
        .HasIndex("CreatedDate");
}

// การใช้งาน Shadow Properties
public async Task<Student> CreateStudentAsync(Student student, string createdBy)
{
    _context.Students.Add(student);
    
    // ตั้งค่า Shadow Properties
    _context.Entry(student).Property("CreatedBy").CurrentValue = createdBy;
    
    await _context.SaveChangesAsync();
    
    return student;
}
```

---

**หมายเหตุ**: การออกแบบ Primary Key และ Foreign Key ที่ดีเป็นรากฐานสำคัญของฐานข้อมูล ควรพิจารณาการใช้งานและประสิทธิภาพอย่างรอบคอบ