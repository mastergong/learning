# บทที่ 10: Fluent API

## Fluent API คืออะไร?

Fluent API เป็นอีกหนึ่งวิธีในการกำหนดค่า Entity Framework Core แทนการใช้ Data Annotations โดยใช้ method chaining ใน `OnModelCreating` ซึ่งให้ความยืดหยุ่นและความสามารถในการกำหนดค่าที่มากกว่า Data Annotations

### ข้อดีของ Fluent API

- **ความสามารถครบถ้วน**: สามารถกำหนดค่าได้ทุกอย่างที่ EF Core รองรับ
- **แยกการกำหนดค่าออกจาก Model**: Model classes สะอาดขึ้น
- **Type Safety**: ตรวจจับ error ได้ตอน compile time
- **IntelliSense**: รองรับ auto-complete ได้ดี
- **การกำหนดค่าที่ซับซ้อน**: สามารถทำได้มากกว่า Data Annotations

## การกำหนดค่า Entity พื้นฐาน

### 1. การกำหนดค่า Table และ Schema

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // กำหนดชื่อตารางและ Schema
    modelBuilder.Entity<Student>()
        .ToTable("Students", "Academic");

    modelBuilder.Entity<Course>()
        .ToTable("Courses", "Academic");

    // กำหนด Default Schema สำหรับทั้ง DbContext
    modelBuilder.HasDefaultSchema("School");

    // กำหนด Table Comment
    modelBuilder.Entity<Student>()
        .ToTable("Students", t => t.HasComment("ตารางข้อมูลนักเรียน"));
}
```

### 2. การกำหนดค่า Primary Key

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    var studentEntity = modelBuilder.Entity<Student>();

    // Single Primary Key
    studentEntity.HasKey(s => s.Id);

    // Composite Primary Key
    modelBuilder.Entity<StudentCourse>()
        .HasKey(sc => new { sc.StudentId, sc.CourseId });

    // กำหนดชื่อ Primary Key Constraint
    studentEntity.HasKey(s => s.Id)
        .HasName("PK_Student_Id");

    // Primary Key ที่ไม่ใช่ Identity
    studentEntity.Property(s => s.Id)
        .ValueGeneratedNever();

    // Primary Key แบบ GUID
    modelBuilder.Entity<Product>()
        .HasKey(p => p.Id);
    
    modelBuilder.Entity<Product>()
        .Property(p => p.Id)
        .HasDefaultValueSql("NEWID()");
}
```

### 3. การกำหนดค่า Properties

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    var studentEntity = modelBuilder.Entity<Student>();

    // Property พื้นฐาน
    studentEntity.Property(s => s.FirstName)
        .IsRequired()
        .HasMaxLength(50)
        .HasColumnName("given_name")
        .HasColumnType("nvarchar(50)")
        .HasComment("ชื่อจริงของนักเรียน");

    // Property ที่เป็น Decimal
    studentEntity.Property(s => s.GPA)
        .HasColumnType("decimal(3,2)")
        .HasPrecision(3, 2);

    // Property ที่เป็น DateTime
    studentEntity.Property(s => s.EnrollmentDate)
        .HasColumnType("datetime2")
        .HasDefaultValueSql("GETDATE()");

    // Property ที่เป็น Enum
    studentEntity.Property(s => s.Status)
        .HasConversion<int>() // เก็บเป็น int ในฐานข้อมูล
        .HasDefaultValue(StudentStatus.Active);

    // String Property ที่เป็น Unicode
    studentEntity.Property(s => s.Notes)
        .IsUnicode(true)
        .HasMaxLength(1000);

    // Computed Column
    studentEntity.Property(s => s.Age)
        .HasComputedColumnSql("DATEDIFF(YEAR, [DateOfBirth], GETDATE())");

    // Property ที่ไม่ map กับฐานข้อมูล
    studentEntity.Ignore(s => s.FullName);
}
```

## การกำหนดค่า Relationships

### 1. One-to-One Relationship

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Principal to Dependent
    modelBuilder.Entity<Student>()
        .HasOne(s => s.Profile)
        .WithOne(p => p.Student)
        .HasForeignKey<StudentProfile>(p => p.StudentId)
        .OnDelete(DeleteBehavior.Cascade)
        .HasConstraintName("FK_StudentProfile_Student");

    // กำหนด Required relationship
    modelBuilder.Entity<Student>()
        .HasOne(s => s.Profile)
        .WithOne(p => p.Student)
        .IsRequired();

    // Alternative syntax
    modelBuilder.Entity<StudentProfile>()
        .HasOne(p => p.Student)
        .WithOne(s => s.Profile)
        .HasForeignKey<StudentProfile>(p => p.StudentId);
}
```

### 2. One-to-Many Relationship

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Department -> Students
    modelBuilder.Entity<Department>()
        .HasMany(d => d.Students)
        .WithOne(s => s.Department)
        .HasForeignKey(s => s.DepartmentId)
        .OnDelete(DeleteBehavior.SetNull)
        .HasConstraintName("FK_Student_Department");

    // Course -> Enrollments
    modelBuilder.Entity<Course>()
        .HasMany(c => c.Enrollments)
        .WithOne(e => e.Course)
        .HasForeignKey(e => e.CourseId)
        .OnDelete(DeleteBehavior.Cascade);

    // Required relationship
    modelBuilder.Entity<Course>()
        .HasMany(c => c.Enrollments)
        .WithOne(e => e.Course)
        .IsRequired();

    // กำหนด Principal Key ที่ไม่ใช่ Primary Key
    modelBuilder.Entity<Department>()
        .HasMany(d => d.Students)
        .WithOne(s => s.Department)
        .HasPrincipalKey(d => d.Code) // ใช้ Code แทน Id
        .HasForeignKey(s => s.DepartmentCode);
}
```

### 3. Many-to-Many Relationship

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Traditional Many-to-Many with Join Entity
    modelBuilder.Entity<Enrollment>()
        .HasKey(e => new { e.StudentId, e.CourseId });

    modelBuilder.Entity<Enrollment>()
        .HasOne(e => e.Student)
        .WithMany(s => s.Enrollments)
        .HasForeignKey(e => e.StudentId)
        .OnDelete(DeleteBehavior.Cascade);

    modelBuilder.Entity<Enrollment>()
        .HasOne(e => e.Course)
        .WithMany(c => c.Enrollments)
        .HasForeignKey(e => e.CourseId)
        .OnDelete(DeleteBehavior.Cascade);

    // Modern Many-to-Many (EF Core 5+)
    modelBuilder.Entity<Student>()
        .HasMany(s => s.Courses)
        .WithMany(c => c.Students)
        .UsingEntity<StudentCourse>(
            j => j
                .HasOne(sc => sc.Course)
                .WithMany()
                .HasForeignKey(sc => sc.CourseId),
            j => j
                .HasOne(sc => sc.Student)
                .WithMany()
                .HasForeignKey(sc => sc.StudentId),
            j =>
            {
                j.Property(sc => sc.EnrollmentDate).HasDefaultValueSql("GETDATE()");
                j.HasKey(sc => new { sc.StudentId, sc.CourseId });
                j.ToTable("StudentCourses");
            });

    // Simple Many-to-Many without additional properties
    modelBuilder.Entity<Student>()
        .HasMany(s => s.Subjects)
        .WithMany(sub => sub.Students)
        .UsingEntity(j => j.ToTable("StudentSubjects"));
}
```

### 4. Self-Referencing Relationship

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Employee -> Manager relationship
    modelBuilder.Entity<Employee>()
        .HasOne(e => e.Manager)
        .WithMany(m => m.Subordinates)
        .HasForeignKey(e => e.ManagerId)
        .OnDelete(DeleteBehavior.Restrict); // ป้องกัน cascade delete

    // Category Tree
    modelBuilder.Entity<Category>()
        .HasOne(c => c.ParentCategory)
        .WithMany(c => c.SubCategories)
        .HasForeignKey(c => c.ParentCategoryId)
        .OnDelete(DeleteBehavior.Restrict);
}
```

## การกำหนดค่า Index

### 1. Single Column Index

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    var studentEntity = modelBuilder.Entity<Student>();

    // Basic Index
    studentEntity.HasIndex(s => s.Email)
        .HasDatabaseName("IX_Student_Email");

    // Unique Index
    studentEntity.HasIndex(s => s.StudentNumber)
        .IsUnique()
        .HasDatabaseName("IX_Student_Number_Unique");

    // Filtered Index
    studentEntity.HasIndex(s => s.PhoneNumber)
        .HasFilter("PhoneNumber IS NOT NULL")
        .HasDatabaseName("IX_Student_Phone");
}
```

### 2. Composite Index

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    var studentEntity = modelBuilder.Entity<Student>();

    // Multi-column Index
    studentEntity.HasIndex(s => new { s.LastName, s.FirstName })
        .HasDatabaseName("IX_Student_FullName");

    // Unique Composite Index
    studentEntity.HasIndex(s => new { s.DepartmentId, s.StudentNumber })
        .IsUnique()
        .HasDatabaseName("IX_Student_Dept_Number_Unique");

    // Composite Index with Include columns (SQL Server)
    studentEntity.HasIndex(s => new { s.DepartmentId, s.Status })
        .IncludeProperties(s => new { s.FirstName, s.LastName, s.Email })
        .HasDatabaseName("IX_Student_Dept_Status_Covering");
}
```

### 3. Descending Index

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Descending Index (SQL Server 2022+)
    modelBuilder.Entity<Student>()
        .HasIndex(s => new { s.DepartmentId, s.GPA })
        .HasDatabaseName("IX_Student_Dept_GPA_Desc")
        .IsDescending(false, true); // DepartmentId ASC, GPA DESC
}
```

## Value Conversions

### 1. Enum Conversions

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    var studentEntity = modelBuilder.Entity<Student>();

    // Enum to String
    studentEntity.Property(s => s.Status)
        .HasConversion<string>();

    // Enum to Int with custom mapping
    studentEntity.Property(s => s.Gender)
        .HasConversion(
            v => v.ToString(),
            v => (Gender)Enum.Parse(typeof(Gender), v));

    // Multiple Enum values (Flags)
    studentEntity.Property(s => s.Permissions)
        .HasConversion(
            v => string.Join(',', v.ToString().Split(',')),
            v => (Permissions)Enum.Parse(typeof(Permissions), v));
}
```

### 2. Complex Type Conversions

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    var studentEntity = modelBuilder.Entity<Student>();

    // JSON Conversion
    studentEntity.Property(s => s.Address)
        .HasConversion(
            v => JsonSerializer.Serialize(v, (JsonSerializerOptions)null),
            v => JsonSerializer.Deserialize<Address>(v, (JsonSerializerOptions)null));

    // DateTime to Unix Timestamp
    studentEntity.Property(s => s.CreatedTimestamp)
        .HasConversion(
            v => v.ToUnixTimeSeconds(),
            v => DateTimeOffset.FromUnixTimeSeconds(v));

    // Money Type
    studentEntity.Property(s => s.ScholarshipAmount)
        .HasConversion(
            v => v.Amount,
            v => new Money(v, Currency.THB))
        .HasColumnType("decimal(18,2)");
}

// Supporting Types
public record Address(string Street, string City, string PostalCode);
public record Money(decimal Amount, Currency Currency);
public enum Currency { THB, USD, EUR }
```

### 3. Collection Conversions

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    var studentEntity = modelBuilder.Entity<Student>();

    // Array/List to JSON
    studentEntity.Property(s => s.Tags)
        .HasConversion(
            v => JsonSerializer.Serialize(v, (JsonSerializerOptions)null),
            v => JsonSerializer.Deserialize<List<string>>(v, (JsonSerializerOptions)null) ?? new List<string>());

    // Array to CSV
    studentEntity.Property(s => s.Skills)
        .HasConversion(
            v => string.Join(';', v),
            v => v.Split(';', StringSplitOptions.RemoveEmptyEntries));
}
```

## Owned Types

### 1. Simple Owned Type

```csharp
// Owned Type Definition
public class Address
{
    public string Street { get; set; }
    public string City { get; set; }
    public string PostalCode { get; set; }
    public string Country { get; set; } = "Thailand";
}

// Configuration
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Student>()
        .OwnsOne(s => s.Address, address =>
        {
            address.Property(a => a.Street)
                .HasColumnName("Address_Street")
                .HasMaxLength(200)
                .IsRequired();

            address.Property(a => a.City)
                .HasColumnName("Address_City")
                .HasMaxLength(100)
                .IsRequired();

            address.Property(a => a.PostalCode)
                .HasColumnName("Address_PostalCode")
                .HasMaxLength(10);

            address.Property(a => a.Country)
                .HasColumnName("Address_Country")
                .HasMaxLength(50)
                .HasDefaultValue("Thailand");
        });
}
```

### 2. Collection of Owned Types

```csharp
// Owned Type
public class PhoneNumber
{
    public string Number { get; set; }
    public PhoneType Type { get; set; }
    public bool IsPrimary { get; set; }
}

public enum PhoneType { Mobile, Home, Work }

// Configuration
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Student>()
        .OwnsMany(s => s.PhoneNumbers, phoneNumber =>
        {
            phoneNumber.WithOwner().HasForeignKey("StudentId");
            phoneNumber.Property<int>("Id");
            phoneNumber.HasKey("Id");

            phoneNumber.Property(p => p.Number)
                .HasMaxLength(20)
                .IsRequired();

            phoneNumber.Property(p => p.Type)
                .HasConversion<string>();

            phoneNumber.Property(p => p.IsPrimary)
                .HasDefaultValue(false);

            phoneNumber.ToTable("StudentPhoneNumbers");
        });
}
```

### 3. Nested Owned Types

```csharp
// Nested Owned Types
public class ContactInfo
{
    public Address Address { get; set; }
    public List<PhoneNumber> PhoneNumbers { get; set; } = new();
    public string Email { get; set; }
}

// Configuration
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Student>()
        .OwnsOne(s => s.ContactInfo, contact =>
        {
            // Owned Address within ContactInfo
            contact.OwnsOne(c => c.Address, address =>
            {
                address.Property(a => a.Street).HasColumnName("Contact_Address_Street");
                address.Property(a => a.City).HasColumnName("Contact_Address_City");
                // ... other address properties
            });

            // Email property
            contact.Property(c => c.Email)
                .HasColumnName("Contact_Email")
                .HasMaxLength(100);

            // Owned collection within ContactInfo
            contact.OwnsMany(c => c.PhoneNumbers, phone =>
            {
                phone.ToTable("StudentContactPhones");
                phone.WithOwner().HasForeignKey("StudentId");
                phone.Property<int>("Id");
                phone.HasKey("Id");
                // ... phone configuration
            });
        });
}
```

## Global Configurations

### 1. Global Query Filters

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Soft Delete Filter
    modelBuilder.Entity<Student>()
        .HasQueryFilter(s => !s.IsDeleted);

    modelBuilder.Entity<Course>()
        .HasQueryFilter(c => !c.IsDeleted);

    // Tenant Filter
    modelBuilder.Entity<Student>()
        .HasQueryFilter(s => s.TenantId == CurrentTenantId);

    // Active Only Filter
    modelBuilder.Entity<Student>()
        .HasQueryFilter(s => s.Status == StudentStatus.Active);
}

// ใช้งานกับ Query
public async Task<List<Student>> GetActiveStudentsAsync()
{
    // Global filter จะทำงานอัตโนมัติ
    return await _context.Students.ToListAsync();
}

public async Task<List<Student>> GetAllStudentsIncludingDeletedAsync()
{
    // ข้าม Global filter
    return await _context.Students
        .IgnoreQueryFilters()
        .ToListAsync();
}
```

### 2. Conventions

```csharp
protected override void ConfigureConventions(ModelConfigurationBuilder configurationBuilder)
{
    // String length convention
    configurationBuilder.Properties<string>()
        .HaveMaxLength(100);

    // Decimal precision convention
    configurationBuilder.Properties<decimal>()
        .HavePrecision(18, 2);

    // DateTime convention
    configurationBuilder.Properties<DateTime>()
        .HaveColumnType("datetime2");

    // Enum to string convention
    configurationBuilder.Properties<Enum>()
        .HaveConversion<string>();
}
```

### 3. Model Configuration Classes

```csharp
// แยก Configuration ออกเป็นคลาสต่างหาก
public class StudentConfiguration : IEntityTypeConfiguration<Student>
{
    public void Configure(EntityTypeBuilder<Student> builder)
    {
        builder.ToTable("Students", "Academic");
        
        builder.HasKey(s => s.Id);
        
        builder.Property(s => s.StudentNumber)
            .IsRequired()
            .HasMaxLength(20);
            
        builder.HasIndex(s => s.StudentNumber)
            .IsUnique();
            
        // ... other configurations
    }
}

public class CourseConfiguration : IEntityTypeConfiguration<Course>
{
    public void Configure(EntityTypeBuilder<Course> builder)
    {
        builder.ToTable("Courses", "Academic");
        
        builder.HasKey(c => c.Id);
        
        // ... configurations
    }
}

// Apply configurations
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.ApplyConfiguration(new StudentConfiguration());
    modelBuilder.ApplyConfiguration(new CourseConfiguration());
    
    // หรือ Apply ทั้งหมดใน Assembly
    modelBuilder.ApplyConfigurationsFromAssembly(Assembly.GetExecutingAssembly());
}
```

---

**หมายเหตุ**: Fluent API ให้ความยืดหยุ่นมากกว่า Data Annotations แต่อาจทำให้โค้ดในส่วน OnModelCreating ยาวขึ้น แนะนำให้แยก Configuration ออกเป็นคลาสต่างหากเพื่อความเป็นระเบียบ