# บทที่ 3: การสร้าง DbContext และ Models

## การสร้าง DbContext

DbContext เป็นหัวใจสำคัญของ EF Core ที่ทำหน้าที่เป็นตัวกลางระหว่างโค้ดและฐานข้อมูล

### DbContext พื้นฐาน

```csharp
using Microsoft.EntityFrameworkCore;
using EFCoreDemo.Models;

public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    // DbSet สำหรับแต่ละ Entity
    public DbSet<Student> Students { get; set; }
    public DbSet<Course> Courses { get; set; }
    public DbSet<Enrollment> Enrollments { get; set; }
    public DbSet<Department> Departments { get; set; }
    public DbSet<Instructor> Instructors { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // กำหนดค่า Entity ต่างๆ ที่นี่
        ConfigureStudent(modelBuilder);
        ConfigureCourse(modelBuilder);
        ConfigureEnrollment(modelBuilder);
        
        base.OnModelCreating(modelBuilder);
    }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        // ใช้สำหรับกำหนดค่าเพิ่มเติม เช่น Logging
        if (!optionsBuilder.IsConfigured)
        {
            optionsBuilder.UseSqlServer("Server=.;Database=SchoolDB;Trusted_Connection=true;");
        }
        
        // เปิดใช้งาน Sensitive Data Logging สำหรับ Development
        optionsBuilder.EnableSensitiveDataLogging();
        optionsBuilder.EnableDetailedErrors();
    }

    private void ConfigureStudent(ModelBuilder modelBuilder)
    {
        var entity = modelBuilder.Entity<Student>();
        
        entity.ToTable("Students", "School");
        entity.HasKey(s => s.Id);
        entity.Property(s => s.StudentNumber).HasMaxLength(20).IsRequired();
        entity.HasIndex(s => s.StudentNumber).IsUnique();
    }

    private void ConfigureCourse(ModelBuilder modelBuilder)
    {
        var entity = modelBuilder.Entity<Course>();
        
        entity.ToTable("Courses", "School");
        entity.Property(c => c.Credits).HasColumnType("decimal(3,1)");
    }

    private void ConfigureEnrollment(ModelBuilder modelBuilder)
    {
        var entity = modelBuilder.Entity<Enrollment>();
        
        entity.HasKey(e => new { e.StudentId, e.CourseId }); // Composite Key
    }
}
```

### การใช้ DbContext Factory Pattern

```csharp
public interface IDbContextFactory<out TContext> where TContext : DbContext
{
    TContext CreateDbContext();
}

public class ApplicationDbContextFactory : IDbContextFactory<ApplicationDbContext>
{
    private readonly DbContextOptions<ApplicationDbContext> _options;

    public ApplicationDbContextFactory(DbContextOptions<ApplicationDbContext> options)
    {
        _options = options;
    }

    public ApplicationDbContext CreateDbContext()
    {
        return new ApplicationDbContext(_options);
    }
}
```

## Models (Entity Classes)

### Student Model

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

[Table("Students", Schema = "School")]
[Index(nameof(StudentNumber), IsUnique = true)]
[Index(nameof(Email), IsUnique = true)]
public class Student
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int Id { get; set; }

    [Required]
    [StringLength(20)]
    [Display(Name = "รหัสนักเรียน")]
    public string StudentNumber { get; set; }

    [Required]
    [StringLength(50)]
    [Display(Name = "ชื่อ")]
    public string FirstName { get; set; }

    [Required]
    [StringLength(50)]
    [Display(Name = "นามสกุล")]
    public string LastName { get; set; }

    [StringLength(50)]
    [Display(Name = "ชื่อเล่น")]
    public string? NickName { get; set; }

    [Required]
    [EmailAddress]
    [StringLength(100)]
    public string Email { get; set; }

    [Phone]
    [StringLength(15)]
    public string? PhoneNumber { get; set; }

    [DataType(DataType.Date)]
    [Display(Name = "วันเกิด")]
    public DateTime DateOfBirth { get; set; }

    [Required]
    [Display(Name = "เพศ")]
    public Gender Gender { get; set; }

    [StringLength(500)]
    [Display(Name = "ที่อยู่")]
    public string? Address { get; set; }

    [ForeignKey("Department")]
    public int? DepartmentId { get; set; }

    [Column(TypeName = "decimal(3,2)")]
    [Range(0, 4)]
    [Display(Name = "เกรดเฉลี่ย")]
    public decimal? GPA { get; set; }

    [Display(Name = "วันที่เข้าเรียน")]
    public DateTime EnrollmentDate { get; set; } = DateTime.Now;

    [Display(Name = "สถานะ")]
    public StudentStatus Status { get; set; } = StudentStatus.Active;

    // Navigation Properties
    public virtual Department? Department { get; set; }
    public virtual ICollection<Enrollment> Enrollments { get; set; } = new List<Enrollment>();
    public virtual StudentProfile? Profile { get; set; } // One-to-One

    // Computed Properties
    [NotMapped]
    [Display(Name = "ชื่อเต็ม")]
    public string FullName => $"{FirstName} {LastName}";

    [NotMapped]
    [Display(Name = "อายุ")]
    public int Age
    {
        get
        {
            var today = DateTime.Today;
            var age = today.Year - DateOfBirth.Year;
            if (DateOfBirth > today.AddYears(-age)) age--;
            return age;
        }
    }

    [NotMapped]
    public bool IsAdult => Age >= 18;
}

public enum Gender
{
    [Display(Name = "ชาย")]
    Male = 1,
    
    [Display(Name = "หญิง")]
    Female = 2,
    
    [Display(Name = "อื่นๆ")]
    Other = 3
}

public enum StudentStatus
{
    [Display(Name = "กำลังเรียน")]
    Active = 1,
    
    [Display(Name = "พักการเรียน")]
    Suspended = 2,
    
    [Display(Name = "จบการศึกษา")]
    Graduated = 3,
    
    [Display(Name = "ถูกไล่ออก")]
    Expelled = 4
}
```

### Course Model

```csharp
[Table("Courses", Schema = "School")]
[Index(nameof(CourseCode), IsUnique = true)]
public class Course
{
    [Key]
    public int Id { get; set; }

    [Required]
    [StringLength(10)]
    [Display(Name = "รหัสวิชา")]
    public string CourseCode { get; set; }

    [Required]
    [StringLength(200)]
    [Display(Name = "ชื่อวิชา")]
    public string Title { get; set; }

    [Column(TypeName = "decimal(3,1)")]
    [Range(0, 10)]
    [Display(Name = "หน่วยกิต")]
    public decimal Credits { get; set; }

    [StringLength(1000)]
    [Display(Name = "รายละเอียดวิชา")]
    public string? Description { get; set; }

    [ForeignKey("Department")]
    public int DepartmentId { get; set; }

    [ForeignKey("Instructor")]
    public int? InstructorId { get; set; }

    [Display(Name = "จำนวนชั่วโมงเรียน")]
    public int LectureHours { get; set; }

    [Display(Name = "จำนวนชั่วโมงปฏิบัติ")]
    public int LabHours { get; set; }

    [Display(Name = "ภาคการศึกษา")]
    public Semester Semester { get; set; }

    [Display(Name = "ปีการศึกษา")]
    public int AcademicYear { get; set; }

    [Display(Name = "จำนวนนักเรียนสูงสุด")]
    public int MaxStudents { get; set; } = 50;

    [Display(Name = "วันเวลาเรียน")]
    [StringLength(200)]
    public string? Schedule { get; set; }

    [Display(Name = "ห้องเรียน")]
    [StringLength(50)]
    public string? Classroom { get; set; }

    // Navigation Properties
    public virtual Department Department { get; set; }
    public virtual Instructor? Instructor { get; set; }
    public virtual ICollection<Enrollment> Enrollments { get; set; } = new List<Enrollment>();
    public virtual ICollection<Prerequisite> Prerequisites { get; set; } = new List<Prerequisite>();

    // Computed Properties
    [NotMapped]
    public int EnrolledStudents => Enrollments?.Count ?? 0;

    [NotMapped]
    public bool IsFull => EnrolledStudents >= MaxStudents;

    [NotMapped]
    public int TotalHours => LectureHours + LabHours;
}

public enum Semester
{
    [Display(Name = "ภาคต้น")]
    First = 1,
    
    [Display(Name = "ภาคปลาย")]
    Second = 2,
    
    [Display(Name = "ภาคฤดูร้อน")]
    Summer = 3
}
```

### Department Model

```csharp
[Table("Departments", Schema = "School")]
public class Department
{
    [Key]
    public int Id { get; set; }

    [Required]
    [StringLength(100)]
    [Display(Name = "ชื่อภาควิชา")]
    public string Name { get; set; }

    [Required]
    [StringLength(20)]
    [Display(Name = "รหัสภาควิชา")]
    public string Code { get; set; }

    [StringLength(500)]
    [Display(Name = "รายละเอียด")]
    public string? Description { get; set; }

    [Display(Name = "วันที่ก่อตั้ง")]
    public DateTime? EstablishedDate { get; set; }

    [StringLength(200)]
    [Display(Name = "อาคาร")]
    public string? Building { get; set; }

    [StringLength(50)]
    [Display(Name = "ชั้น/ห้อง")]
    public string? Floor { get; set; }

    [Phone]
    [StringLength(15)]
    [Display(Name = "เบอร์โทรศัพท์")]
    public string? PhoneNumber { get; set; }

    [EmailAddress]
    [StringLength(100)]
    [Display(Name = "อีเมล")]
    public string? Email { get; set; }

    [Url]
    [StringLength(200)]
    [Display(Name = "เว็บไซต์")]
    public string? Website { get; set; }

    [ForeignKey("Dean")]
    public int? DeanId { get; set; }

    // Navigation Properties
    public virtual Instructor? Dean { get; set; }
    public virtual ICollection<Student> Students { get; set; } = new List<Student>();
    public virtual ICollection<Course> Courses { get; set; } = new List<Course>();
    public virtual ICollection<Instructor> Instructors { get; set; } = new List<Instructor>();

    // Computed Properties
    [NotMapped]
    public int TotalStudents => Students?.Count ?? 0;

    [NotMapped]
    public int TotalCourses => Courses?.Count ?? 0;

    [NotMapped]
    public int TotalInstructors => Instructors?.Count ?? 0;
}
```

### StudentProfile Model (One-to-One)

```csharp
[Table("StudentProfiles", Schema = "School")]
public class StudentProfile
{
    [Key]
    [ForeignKey("Student")]
    public int StudentId { get; set; }

    [StringLength(100)]
    [Display(Name = "ชื่อผู้ปกครอง")]
    public string? ParentName { get; set; }

    [Phone]
    [StringLength(15)]
    [Display(Name = "เบอร์ติดต่อผู้ปกครอง")]
    public string? ParentPhone { get; set; }

    [StringLength(100)]
    [Display(Name = "โรงเรียนเดิม")]
    public string? PreviousSchool { get; set; }

    [Column(TypeName = "decimal(3,2)")]
    [Range(0, 4)]
    [Display(Name = "เกรดเฉลี่ยเดิม")]
    public decimal? PreviousGPA { get; set; }

    [StringLength(200)]
    [Display(Name = "โรคประจำตัว")]
    public string? MedicalCondition { get; set; }

    [StringLength(200)]
    [Display(Name = "ยาที่แพ้")]
    public string? Allergies { get; set; }

    [StringLength(500)]
    [Display(Name = "หมายเหตุ")]
    public string? Notes { get; set; }

    [Display(Name = "วันที่สร้างโปรไฟล์")]
    public DateTime CreatedDate { get; set; } = DateTime.Now;

    [Display(Name = "วันที่อัปเดต")]
    public DateTime? UpdatedDate { get; set; }

    // Navigation Properties
    public virtual Student Student { get; set; }
}
```

## การกำหนดค่า DbContext ในโปรเจค

### Program.cs (Minimal API)

```csharp
using Microsoft.EntityFrameworkCore;
using EFCoreDemo.Data;

var builder = WebApplication.CreateBuilder(args);

// เพิ่ม DbContext
builder.Services.AddDbContext<ApplicationDbContext>(options =>
{
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection"));
    
    // กำหนดค่าเพิ่มเติม
    if (builder.Environment.IsDevelopment())
    {
        options.EnableSensitiveDataLogging();
        options.EnableDetailedErrors();
    }
});

// เพิ่ม DbContext Factory (สำหรับ Background Services)
builder.Services.AddDbContextFactory<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// สร้างฐานข้อมูลและ Seed Data
using (var scope = app.Services.CreateScope())
{
    var context = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
    await context.Database.EnsureCreatedAsync();
    
    // หรือใช้ Migration
    // await context.Database.MigrateAsync();
}

app.Run();
```

### appsettings.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=SchoolManagementDB;Trusted_Connection=true;MultipleActiveResultSets=true;TrustServerCertificate=true",
    "BackupConnection": "Server=.\\SQLEXPRESS;Database=SchoolManagementDB;Integrated Security=true;TrustServerCertificate=true"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.EntityFrameworkCore.Database.Command": "Information"
    }
  }
}
```

## การใช้งาน DbContext

### Repository Pattern

```csharp
public interface IStudentRepository
{
    Task<IEnumerable<Student>> GetAllAsync();
    Task<Student?> GetByIdAsync(int id);
    Task<Student> AddAsync(Student student);
    Task UpdateAsync(Student student);
    Task DeleteAsync(int id);
    Task<IEnumerable<Student>> SearchAsync(string searchTerm);
}

public class StudentRepository : IStudentRepository
{
    private readonly ApplicationDbContext _context;

    public StudentRepository(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<IEnumerable<Student>> GetAllAsync()
    {
        return await _context.Students
            .Include(s => s.Department)
            .Include(s => s.Profile)
            .AsNoTracking()
            .ToListAsync();
    }

    public async Task<Student?> GetByIdAsync(int id)
    {
        return await _context.Students
            .Include(s => s.Department)
            .Include(s => s.Profile)
            .Include(s => s.Enrollments)
                .ThenInclude(e => e.Course)
            .FirstOrDefaultAsync(s => s.Id == id);
    }

    public async Task<Student> AddAsync(Student student)
    {
        _context.Students.Add(student);
        await _context.SaveChangesAsync();
        return student;
    }

    public async Task UpdateAsync(Student student)
    {
        _context.Entry(student).State = EntityState.Modified;
        await _context.SaveChangesAsync();
    }

    public async Task DeleteAsync(int id)
    {
        var student = await _context.Students.FindAsync(id);
        if (student != null)
        {
            _context.Students.Remove(student);
            await _context.SaveChangesAsync();
        }
    }

    public async Task<IEnumerable<Student>> SearchAsync(string searchTerm)
    {
        return await _context.Students
            .Where(s => s.FirstName.Contains(searchTerm) || 
                       s.LastName.Contains(searchTerm) ||
                       s.StudentNumber.Contains(searchTerm))
            .Include(s => s.Department)
            .AsNoTracking()
            .ToListAsync();
    }
}
```

## การกำหนดค่าขั้นสูงด้วย Fluent API

### 1. Complex Relationship Configurations

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Self-referencing relationship
    modelBuilder.Entity<Employee>()
        .HasOne(e => e.Manager)
        .WithMany(m => m.Subordinates)
        .HasForeignKey(e => e.ManagerId)
        .OnDelete(DeleteBehavior.Restrict);

    // Multiple relationships ระหว่าง entities เดียวกัน
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

    // Conditional relationships
    modelBuilder.Entity<Student>()
        .HasOne(s => s.Advisor)
        .WithMany(i => i.AdvisedStudents)
        .HasForeignKey(s => s.AdvisorId)
        .IsRequired(false);
}
```

### 2. Value Converters ขั้นสูง

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // JSON Converter
    modelBuilder.Entity<Student>()
        .Property(s => s.ContactInfo)
        .HasConversion(
            v => JsonSerializer.Serialize(v, (JsonSerializerOptions)null),
            v => JsonSerializer.Deserialize<ContactInfo>(v, (JsonSerializerOptions)null)!,
            new ValueComparer<ContactInfo>(
                (c1, c2) => c1!.Equals(c2),
                c => c.GetHashCode(),
                c => new ContactInfo(c.Email, c.PhoneNumber, c.Address)));

    // Enum Array Converter
    modelBuilder.Entity<Student>()
        .Property(s => s.Skills)
        .HasConversion(
            v => string.Join(';', v.Select(e => e.ToString())),
            v => v.Split(';', StringSplitOptions.RemoveEmptyEntries)
                  .Select(e => Enum.Parse<Skill>(e)).ToArray());

    // Money Value Converter
    modelBuilder.Entity<Student>()
        .OwnsOne(s => s.ScholarshipAmount, money =>
        {
            money.Property(m => m.Amount)
                .HasColumnType("decimal(18,2)");
            money.Property(m => m.Currency)
                .HasConversion<string>()
                .HasMaxLength(3);
        });

    // Date Range Converter
    modelBuilder.Entity<Course>()
        .Property(c => c.Duration)
        .HasConversion<DateRangeConverter>();
}

// Custom Value Converters
public class DateRangeConverter : ValueConverter<DateRange, string>
{
    public DateRangeConverter() : base(
        v => $"{v.StartDate:yyyy-MM-dd}|{v.EndDate:yyyy-MM-dd}",
        v => ParseDateRange(v))
    {
    }

    private static DateRange ParseDateRange(string value)
    {
        var parts = value.Split('|');
        return new DateRange(
            DateTime.Parse(parts[0]),
            DateTime.Parse(parts[1]));
    }
}
```

### 3. Table Splitting และ Entity Splitting

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Table Splitting - หลาย Entity ใช้ตารางเดียวกัน
    modelBuilder.Entity<Student>()
        .ToTable("Students");

    modelBuilder.Entity<StudentProfile>()
        .ToTable("Students") // ใช้ตารางเดียวกัน
        .HasOne<Student>()
        .WithOne()
        .HasForeignKey<StudentProfile>(p => p.Id);

    // Entity Splitting - Entity เดียวแยกเป็นหลายตาราง
    modelBuilder.Entity<Student>()
        .ToTable("Students")
        .SplitToTable("StudentDetails", splitBuilder =>
        {
            splitBuilder.Property(s => s.PersonalNotes);
            splitBuilder.Property(s => s.EmergencyContact);
            splitBuilder.Property(s => s.MedicalInfo);
        });

    // Owned Type Table Splitting
    modelBuilder.Entity<Student>()
        .OwnsOne(s => s.Address, address =>
        {
            address.ToTable("StudentAddresses");
            address.Property(a => a.Street).HasColumnName("Street");
            address.Property(a => a.City).HasColumnName("City");
            address.Property(a => a.PostalCode).HasColumnName("PostalCode");
        });
}
```

### 4. Global Query Filters และ Soft Delete

```csharp
public abstract class BaseEntity
{
    public bool IsDeleted { get; set; } = false;
    public DateTime? DeletedAt { get; set; }
    public string? DeletedBy { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public string? CreatedBy { get; set; }
    public DateTime? UpdatedAt { get; set; }
    public string? UpdatedBy { get; set; }
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Global Query Filter สำหรับ Soft Delete
    foreach (var entityType in modelBuilder.Model.GetEntityTypes())
    {
        if (typeof(BaseEntity).IsAssignableFrom(entityType.ClrType))
        {
            modelBuilder.Entity(entityType.ClrType)
                .HasQueryFilter(GetSoftDeleteFilter(entityType.ClrType));
        }
    }

    // Multi-tenant Global Filter
    modelBuilder.Entity<Student>()
        .HasQueryFilter(s => s.TenantId == GetCurrentTenantId());

    // Active Status Filter
    modelBuilder.Entity<Student>()
        .HasQueryFilter(s => s.Status == StudentStatus.Active);
}

private LambdaExpression GetSoftDeleteFilter(Type entityType)
{
    var parameter = Expression.Parameter(entityType, "e");
    var property = Expression.Property(parameter, "IsDeleted");
    var notDeleted = Expression.Equal(property, Expression.Constant(false));
    return Expression.Lambda(notDeleted, parameter);
}

private int GetCurrentTenantId()
{
    // Get tenant ID from current context
    return 1; // Placeholder
}
```

### 5. Advanced Owned Types และ Complex Types

```csharp
// Complex Owned Type with Collections
public class ContactInformation
{
    public List<PhoneNumber> PhoneNumbers { get; set; } = new();
    public List<EmailAddress> EmailAddresses { get; set; } = new();
    public Address PrimaryAddress { get; set; } = new();
    public Address? AlternateAddress { get; set; }
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Student>()
        .OwnsOne(s => s.ContactInfo, contact =>
        {
            // Nested Owned Collections
            contact.OwnsMany(c => c.PhoneNumbers, phone =>
            {
                phone.ToTable("StudentPhones");
                phone.WithOwner().HasForeignKey("StudentId");
                phone.Property<int>("Id");
                phone.HasKey("Id");
                phone.Property(p => p.Number).HasMaxLength(20);
                phone.Property(p => p.Type).HasConversion<string>();
                phone.Property(p => p.IsPrimary).HasDefaultValue(false);
            });

            contact.OwnsMany(c => c.EmailAddresses, email =>
            {
                email.ToTable("StudentEmails");
                email.WithOwner().HasForeignKey("StudentId");
                email.Property<int>("Id");
                email.HasKey("Id");
                email.Property(e => e.Address).HasMaxLength(100);
                email.Property(e => e.Type).HasConversion<string>();
                email.Property(e => e.IsVerified).HasDefaultValue(false);
            });

            // Nested Owned Types
            contact.OwnsOne(c => c.PrimaryAddress, addr =>
            {
                addr.Property(a => a.Street).HasColumnName("PrimaryAddress_Street");
                addr.Property(a => a.City).HasColumnName("PrimaryAddress_City");
                addr.Property(a => a.Country).HasDefaultValue("Thailand");
            });

            contact.OwnsOne(c => c.AlternateAddress, addr =>
            {
                addr.Property(a => a.Street).HasColumnName("AlternateAddress_Street");
                addr.Property(a => a.City).HasColumnName("AlternateAddress_City");
            });
        });
}
```

### 6. Custom Conventions และ Model Building

```csharp
protected override void ConfigureConventions(ModelConfigurationBuilder configurationBuilder)
{
    // String Properties Convention
    configurationBuilder.Properties<string>()
        .HaveMaxLength(500) // Default max length
        .AreUnicode(); // Default to Unicode

    // Decimal Properties Convention
    configurationBuilder.Properties<decimal>()
        .HavePrecision(18, 2);

    // DateTime Convention
    configurationBuilder.Properties<DateTime>()
        .HaveColumnType("datetime2(0)");

    // Enum Convention
    configurationBuilder.Properties<Enum>()
        .HaveConversion<string>();

    // Custom Naming Convention
    configurationBuilder.Types()
        .HaveTableNameFromDisplayName();
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Apply configurations from assembly
    modelBuilder.ApplyConfigurationsFromAssembly(typeof(ApplicationDbContext).Assembly);

    // Custom naming strategy
    foreach (var entity in modelBuilder.Model.GetEntityTypes())
    {
        // Snake case table names
        var tableName = entity.GetTableName();
        if (!string.IsNullOrEmpty(tableName))
        {
            entity.SetTableName(ToSnakeCase(tableName));
        }

        // Snake case column names
        foreach (var property in entity.GetProperties())
        {
            var columnName = property.GetColumnName();
            if (!string.IsNullOrEmpty(columnName))
            {
                property.SetColumnName(ToSnakeCase(columnName));
            }
        }
    }

    // Configure Shadow Properties
    ConfigureShadowProperties(modelBuilder);

    // Configure Indexes
    ConfigureIndexes(modelBuilder);
}

private void ConfigureShadowProperties(ModelBuilder modelBuilder)
{
    foreach (var entityType in modelBuilder.Model.GetEntityTypes())
    {
        // Add audit shadow properties
        modelBuilder.Entity(entityType.ClrType)
            .Property<DateTime>("CreatedDate")
            .HasDefaultValueSql("GETDATE()");

        modelBuilder.Entity(entityType.ClrType)
            .Property<string>("CreatedBy")
            .HasMaxLength(100);

        modelBuilder.Entity(entityType.ClrType)
            .Property<DateTime?>("UpdatedDate");

        modelBuilder.Entity(entityType.ClrType)
            .Property<string>("UpdatedBy")
            .HasMaxLength(100);

        // Add concurrency token
        modelBuilder.Entity(entityType.ClrType)
            .Property<byte[]>("RowVersion")
            .IsRowVersion();
    }
}

private void ConfigureIndexes(ModelBuilder modelBuilder)
{
    // Auto-create indexes for foreign keys
    foreach (var entityType in modelBuilder.Model.GetEntityTypes())
    {
        foreach (var foreignKey in entityType.GetForeignKeys())
        {
            var indexName = $"IX_{entityType.GetTableName()}_{string.Join("_", foreignKey.Properties.Select(p => p.Name))}";
            
            modelBuilder.Entity(entityType.ClrType)
                .HasIndex(foreignKey.Properties.Select(p => p.Name).ToArray())
                .HasDatabaseName(indexName);
        }
    }

    // Auto-create indexes for common search properties
    var searchableProperties = new[] { "Name", "Title", "Email", "Code" };
    
    foreach (var entityType in modelBuilder.Model.GetEntityTypes())
    {
        foreach (var property in entityType.GetProperties())
        {
            if (searchableProperties.Contains(property.Name) && property.ClrType == typeof(string))
            {
                var indexName = $"IX_{entityType.GetTableName()}_{property.Name}";
                
                modelBuilder.Entity(entityType.ClrType)
                    .HasIndex(property.Name)
                    .HasDatabaseName(indexName);
            }
        }
    }
}

private static string ToSnakeCase(string input)
{
    if (string.IsNullOrEmpty(input))
        return input;

    return Regex.Replace(input, "(?<!^)([A-Z])", "_$1").ToLower();
}
```

### 7. Temporal Tables (SQL Server 2016+)

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Configure Temporal Table
    modelBuilder.Entity<Student>()
        .ToTable("Students", b => b.IsTemporal(temporal =>
        {
            temporal.HasPeriodStart("ValidFrom")
                   .HasColumnName("ValidFrom");
            temporal.HasPeriodEnd("ValidTo")
                   .HasColumnName("ValidTo");
            temporal.UseHistoryTable("StudentHistory", "History");
        }));

    // Query historical data
    // var historicalData = context.Students.TemporalAll().Where(...);
}
```

### 8. Advanced Query Configuration

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Define Views
    modelBuilder.Entity<StudentSummaryView>()
        .HasNoKey()
        .ToView("vw_StudentSummary")
        .HasAnnotation("Relational:ViewDefinition", @"
            SELECT 
                s.Id,
                s.FirstName + ' ' + s.LastName AS FullName,
                s.Email,
                d.Name AS DepartmentName,
                COUNT(e.CourseId) AS EnrolledCourses
            FROM Students s
            LEFT JOIN Departments d ON s.DepartmentId = d.Id  
            LEFT JOIN Enrollments e ON s.Id = e.StudentId
            GROUP BY s.Id, s.FirstName, s.LastName, s.Email, d.Name");

    // Define Stored Procedure Results
    modelBuilder.Entity<StoredProcedureResult>()
        .HasNoKey();

    // Configure Function Mappings
    modelBuilder.HasDbFunction(typeof(ApplicationDbContext).GetMethod(nameof(GetStudentAge))!)
        .HasName("fn_GetStudentAge");
}

[DbFunction]
public static int GetStudentAge(DateTime dateOfBirth)
{
    throw new NotSupportedException("This method is for use with Entity Framework Core only.");
}
```

## Tips และ Best Practices

### 1. การจัดการ Connection String

```csharp
public class DatabaseConfiguration
{
    public static string GetConnectionString(IConfiguration configuration)
    {
        var connectionString = configuration.GetConnectionString("DefaultConnection");
        
        // แทนที่ค่าจาก Environment Variables
        connectionString = connectionString?.Replace("{DB_SERVER}", 
            Environment.GetEnvironmentVariable("DB_SERVER") ?? "localhost");
        connectionString = connectionString?.Replace("{DB_NAME}", 
            Environment.GetEnvironmentVariable("DB_NAME") ?? "SchoolDB");
        
        return connectionString ?? throw new InvalidOperationException("Connection string not found");
    }
}
```

### 2. การใช้ Multiple DbContext

```csharp
// สำหรับ Business Data
public class BusinessDbContext : DbContext
{
    public DbSet<Student> Students { get; set; }
    public DbSet<Course> Courses { get; set; }
}

// สำหรับ Logging และ Audit
public class LoggingDbContext : DbContext
{
    public DbSet<AuditLog> AuditLogs { get; set; }
    public DbSet<ErrorLog> ErrorLogs { get; set; }
}

// Registration
builder.Services.AddDbContext<BusinessDbContext>(options =>
    options.UseSqlServer(connectionString));
    
builder.Services.AddDbContext<LoggingDbContext>(options =>
    options.UseSqlServer(loggingConnectionString));
```

---

**หมายเหตุ**: การออกแบบ Models และ DbContext ที่ดีจะส่งผลต่อประสิทธิภาพและความง่ายในการบำรุงรักษาโค้ดในระยะยาว