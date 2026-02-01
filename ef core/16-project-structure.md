# บทที่ 16: โครงสร้าง Project และ Configuration ครบชุด

## 🎯 ภาพรวมของบทนี้

บทนี้จะแสดงตัวอย่างการสร้าง Project EF Core แบบครบชุด พร้อมไฟล์และการตั้งค่าที่จำเป็นทั้งหมด รวมถึงตัวอย่างการใช้งาน 1 Database และหลาย Database

### สิ่งที่คุณจะได้เรียนรู้:
- โครงสร้าง Project ที่เป็นมาตรฐาน
- ไฟล์และ Configuration ที่จำเป็น
- ตัวอย่าง Single Database Project
- ตัวอย่าง Multiple Database Project
- Best Practices สำหรับการจัดการ

---

## 📁 โครงสร้าง Project มาตรฐาน

### โครงสร้างพื้นฐานที่แนะนำ

```
EFCoreProject/
├── 📁 src/
│   └── 📁 EFCoreProject/
│       ├── 📁 Data/                    # DbContext และ Configuration
│       │   ├── ApplicationDbContext.cs
│       │   ├── Configurations/         # Entity Configurations
│       │   │   ├── StudentConfiguration.cs
│       │   │   └── CourseConfiguration.cs
│       │   └── Seed/                   # Seed Data
│       │       └── SeedData.cs
│       ├── 📁 Models/                  # Entity Models
│       │   ├── Student.cs
│       │   ├── Course.cs
│       │   ├── Enrollment.cs
│       │   └── Common/                 # Base Classes
│       │       └── BaseEntity.cs
│       ├── 📁 Services/                # Business Logic
│       │   ├── Interfaces/
│       │   │   └── IStudentService.cs
│       │   └── StudentService.cs
│       ├── 📁 Repositories/            # Data Access Layer
│       │   ├── Interfaces/
│       │   │   ├── IGenericRepository.cs
│       │   │   └── IStudentRepository.cs
│       │   ├── GenericRepository.cs
│       │   └── StudentRepository.cs
│       ├── 📁 DTOs/                    # Data Transfer Objects
│       │   ├── StudentDto.cs
│       │   └── CourseDto.cs
│       ├── 📁 Migrations/              # EF Migrations (สร้างอัตโนมัติ)
│       ├── appsettings.json
│       ├── appsettings.Development.json
│       ├── EFCoreProject.csproj
│       └── Program.cs
├── 📁 tests/
│   └── 📁 EFCoreProject.Tests/
│       ├── UnitTests/
│       ├── IntegrationTests/
│       └── EFCoreProject.Tests.csproj
├── README.md
├── .gitignore
└── EFCoreProject.sln
```

---

## 🗂️ ตัวอย่าง Project 1: Single Database

### 1. ไฟล์ .csproj

```xml
<!-- EFCoreProject.csproj -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

  <ItemGroup>
    <!-- EF Core Packages -->
    <PackageReference Include="Microsoft.EntityFrameworkCore" Version="6.0.25" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="6.0.25" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="6.0.25" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="6.0.25" />
    
    <!-- Configuration และ Logging -->
    <PackageReference Include="Microsoft.Extensions.Hosting" Version="6.0.1" />
    <PackageReference Include="Microsoft.Extensions.Configuration.Json" Version="6.0.0" />
    <PackageReference Include="Serilog.Extensions.Hosting" Version="5.0.1" />
    <PackageReference Include="Serilog.Sinks.Console" Version="4.1.0" />
    
    <!-- Utilities -->
    <PackageReference Include="AutoMapper.Extensions.Microsoft.DependencyInjection" Version="12.0.1" />
    <PackageReference Include="FluentValidation.DependencyInjectionExtensions" Version="11.8.0" />
  </ItemGroup>

  <ItemGroup>
    <None Update="appsettings.json">
      <CopyToOutputDirectory>Always</CopyToOutputDirectory>
    </None>
    <None Update="appsettings.Development.json">
      <CopyToOutputDirectory>Always</CopyToOutputDirectory>
    </None>
  </ItemGroup>
</Project>
```

### 2. appsettings.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=SchoolManagement;Trusted_Connection=true;MultipleActiveResultSets=true"
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

### 3. appsettings.Development.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=SchoolManagement_Dev;Trusted_Connection=true;MultipleActiveResultSets=true"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "System": "Information",
      "Microsoft": "Information",
      "Microsoft.EntityFrameworkCore.Database.Command": "Information"
    }
  }
}
```

### 4. Base Entity Class

```csharp
// Models/Common/BaseEntity.cs
namespace EFCoreProject.Models.Common;

public abstract class BaseEntity
{
    public int Id { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.Now;
    public DateTime? UpdatedAt { get; set; }
    public string? CreatedBy { get; set; }
    public string? UpdatedBy { get; set; }
    public bool IsDeleted { get; set; } = false;
}
```

### 5. Entity Models

```csharp
// Models/Student.cs
using EFCoreProject.Models.Common;
using System.ComponentModel.DataAnnotations;

namespace EFCoreProject.Models;

public class Student : BaseEntity
{
    [Required]
    [MaxLength(50)]
    public string FirstName { get; set; } = string.Empty;
    
    [Required]
    [MaxLength(50)]
    public string LastName { get; set; } = string.Empty;
    
    [Required]
    [EmailAddress]
    [MaxLength(100)]
    public string Email { get; set; } = string.Empty;
    
    [MaxLength(15)]
    public string? Phone { get; set; }
    
    public DateTime DateOfBirth { get; set; }
    
    [MaxLength(20)]
    public string StudentNumber { get; set; } = string.Empty;
    
    public int? DepartmentId { get; set; }
    public virtual Department? Department { get; set; }
    
    public virtual ICollection<Enrollment> Enrollments { get; set; } = new List<Enrollment>();
    
    // Computed Properties
    public int Age => DateTime.Now.Year - DateOfBirth.Year;
    public string FullName => $"{FirstName} {LastName}";
}

// Models/Course.cs
using EFCoreProject.Models.Common;

namespace EFCoreProject.Models;

public class Course : BaseEntity
{
    [Required]
    [MaxLength(100)]
    public string Title { get; set; } = string.Empty;
    
    [MaxLength(10)]
    public string CourseCode { get; set; } = string.Empty;
    
    public int Credits { get; set; }
    
    [MaxLength(500)]
    public string? Description { get; set; }
    
    public int DepartmentId { get; set; }
    public virtual Department Department { get; set; } = null!;
    
    public virtual ICollection<Enrollment> Enrollments { get; set; } = new List<Enrollment>();
}

// Models/Department.cs
using EFCoreProject.Models.Common;

namespace EFCoreProject.Models;

public class Department : BaseEntity
{
    [Required]
    [MaxLength(100)]
    public string Name { get; set; } = string.Empty;
    
    [MaxLength(10)]
    public string Code { get; set; } = string.Empty;
    
    [MaxLength(200)]
    public string? Description { get; set; }
    
    public virtual ICollection<Student> Students { get; set; } = new List<Student>();
    public virtual ICollection<Course> Courses { get; set; } = new List<Course>();
}

// Models/Enrollment.cs
using EFCoreProject.Models.Common;

namespace EFCoreProject.Models;

public class Enrollment : BaseEntity
{
    public int StudentId { get; set; }
    public virtual Student Student { get; set; } = null!;
    
    public int CourseId { get; set; }
    public virtual Course Course { get; set; } = null!;
    
    public DateTime EnrollmentDate { get; set; } = DateTime.Now;
    
    [Range(0, 4.0)]
    public double? Grade { get; set; }
    
    public EnrollmentStatus Status { get; set; } = EnrollmentStatus.Enrolled;
}

public enum EnrollmentStatus
{
    Enrolled = 1,
    Completed = 2,
    Dropped = 3,
    Failed = 4
}
```

### 6. Entity Configurations

```csharp
// Data/Configurations/StudentConfiguration.cs
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;
using EFCoreProject.Models;

namespace EFCoreProject.Data.Configurations;

public class StudentConfiguration : IEntityTypeConfiguration<Student>
{
    public void Configure(EntityTypeBuilder<Student> builder)
    {
        builder.ToTable("Students");
        
        builder.HasKey(s => s.Id);
        
        builder.Property(s => s.FirstName)
            .IsRequired()
            .HasMaxLength(50);
            
        builder.Property(s => s.LastName)
            .IsRequired()
            .HasMaxLength(50);
            
        builder.Property(s => s.Email)
            .IsRequired()
            .HasMaxLength(100);
            
        builder.Property(s => s.StudentNumber)
            .IsRequired()
            .HasMaxLength(20);
            
        builder.Property(s => s.Phone)
            .HasMaxLength(15);
            
        // Indexes
        builder.HasIndex(s => s.Email)
            .IsUnique()
            .HasDatabaseName("IX_Students_Email");
            
        builder.HasIndex(s => s.StudentNumber)
            .IsUnique()
            .HasDatabaseName("IX_Students_StudentNumber");
            
        // Relationships
        builder.HasOne(s => s.Department)
            .WithMany(d => d.Students)
            .HasForeignKey(s => s.DepartmentId)
            .OnDelete(DeleteBehavior.SetNull);
            
        // ⚠️ Computed Columns (SQL Server specific)
        // หมายเหตุ: ใช้ได้เฉพาะ SQL Server, ถ้าใช้ SQLite ให้เอาส่วนนี้ออก
        builder.Property(s => s.FullName)
            .HasComputedColumnSql("[FirstName] + ' ' + [LastName]");
            
        // Default Values
        builder.Property(s => s.CreatedAt)
            .HasDefaultValueSql("GETDATE()");
            
        builder.Property(s => s.IsDeleted)
            .HasDefaultValue(false);
            
        // Query Filter for Soft Delete
        builder.HasQueryFilter(s => !s.IsDeleted);
    }
}

// Data/Configurations/CourseConfiguration.cs
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;
using EFCoreProject.Models;

namespace EFCoreProject.Data.Configurations;

public class CourseConfiguration : IEntityTypeConfiguration<Course>
{
    public void Configure(EntityTypeBuilder<Course> builder)
    {
        builder.ToTable("Courses");
        
        builder.HasKey(c => c.Id);
        
        builder.Property(c => c.Title)
            .IsRequired()
            .HasMaxLength(100);
            
        builder.Property(c => c.CourseCode)
            .IsRequired()
            .HasMaxLength(10);
            
        builder.Property(c => c.Description)
            .HasMaxLength(500);
            
        // Indexes
        builder.HasIndex(c => c.CourseCode)
            .IsUnique()
            .HasDatabaseName("IX_Courses_CourseCode");
            
        // Relationships
        builder.HasOne(c => c.Department)
            .WithMany(d => d.Courses)
            .HasForeignKey(c => c.DepartmentId)
            .OnDelete(DeleteBehavior.Cascade);
            
        // Query Filter
        builder.HasQueryFilter(c => !c.IsDeleted);
    }
}

// Data/Configurations/EnrollmentConfiguration.cs
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;
using EFCoreProject.Models;

namespace EFCoreProject.Data.Configurations;

public class EnrollmentConfiguration : IEntityTypeConfiguration<Enrollment>
{
    public void Configure(EntityTypeBuilder<Enrollment> builder)
    {
        builder.ToTable("Enrollments");
        
        builder.HasKey(e => e.Id);
        
        // Relationships
        builder.HasOne(e => e.Student)
            .WithMany(s => s.Enrollments)
            .HasForeignKey(e => e.StudentId)
            .OnDelete(DeleteBehavior.Cascade);
            
        builder.HasOne(e => e.Course)
            .WithMany(c => c.Enrollments)
            .HasForeignKey(e => e.CourseId)
            .OnDelete(DeleteBehavior.Cascade);
            
        // Composite Index
        builder.HasIndex(e => new { e.StudentId, e.CourseId })
            .IsUnique()
            .HasDatabaseName("IX_Enrollments_Student_Course");
            
        // Default Values
        builder.Property(e => e.EnrollmentDate)
            .HasDefaultValueSql("GETDATE()");
            
        builder.Property(e => e.Status)
            .HasDefaultValue(EnrollmentStatus.Enrolled);
            
        // Query Filter
        builder.HasQueryFilter(e => !e.IsDeleted);
    }
}
```

### 7. DbContext

```csharp
// Data/ApplicationDbContext.cs
using Microsoft.EntityFrameworkCore;
using System.Linq.Expressions;
using EFCoreProject.Models;
using EFCoreProject.Models.Common;
using EFCoreProject.Data.Configurations;

namespace EFCoreProject.Data;

public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options)
    {
    }

    public DbSet<Student> Students { get; set; } = null!;
    public DbSet<Course> Courses { get; set; } = null!;
    public DbSet<Department> Departments { get; set; } = null!;
    public DbSet<Enrollment> Enrollments { get; set; } = null!;

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Apply all configurations from assembly
        // จะค้นหาและ apply configurations ทั้งหมดที่ implement IEntityTypeConfiguration
        modelBuilder.ApplyConfigurationsFromAssembly(typeof(ApplicationDbContext).Assembly);
        
        // Global Query Filters for Soft Delete
        // Loop ผ่าน entity types ทั้งหมดและเพิ่ม query filter สำหรับ soft delete
        foreach (var entityType in modelBuilder.Model.GetEntityTypes())
        {
            if (typeof(BaseEntity).IsAssignableFrom(entityType.ClrType))
            {
                // สร้าง filter expression แบบ dynamic สำหรับแต่ละ entity type
                modelBuilder.Entity(entityType.ClrType)
                    .HasQueryFilter(GetSoftDeleteFilter(entityType.ClrType));
            }
        }
        
        base.OnModelCreating(modelBuilder);
    }
    
    // สร้าง Lambda Expression สำหรับ Soft Delete Filter แบบ Dynamic
    // ตัวอย่าง: entity => entity.IsDeleted == false
    private static LambdaExpression GetSoftDeleteFilter(Type entityType)
    {
        var parameter = Expression.Parameter(entityType, "e"); // e =>
        var property = Expression.Property(parameter, "IsDeleted"); // e.IsDeleted
        var condition = Expression.Equal(property, Expression.Constant(false)); // e.IsDeleted == false
        return Expression.Lambda(condition, parameter); // e => e.IsDeleted == false
    }

    public override async Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
    {
        foreach (var entry in ChangeTracker.Entries<Common.BaseEntity>())
        {
            switch (entry.State)
            {
                case EntityState.Added:
                    entry.Entity.CreatedAt = DateTime.Now;
                    break;
                case EntityState.Modified:
                    entry.Entity.UpdatedAt = DateTime.Now;
                    break;
                case EntityState.Deleted:
                    entry.State = EntityState.Modified;
                    entry.Entity.IsDeleted = true;
                    entry.Entity.UpdatedAt = DateTime.Now;
                    break;
            }
        }

        return await base.SaveChangesAsync(cancellationToken);
    }
}
```

### 8. Repository Pattern

```csharp
// Repositories/Interfaces/IGenericRepository.cs
using System.Linq.Expressions;

namespace EFCoreProject.Repositories.Interfaces;

public interface IGenericRepository<T> where T : class
{
    Task<T?> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate);
    Task<T?> FirstOrDefaultAsync(Expression<Func<T, bool>> predicate);
    Task AddAsync(T entity);
    Task AddRangeAsync(IEnumerable<T> entities);
    void Update(T entity);
    void Remove(T entity);
    void RemoveRange(IEnumerable<T> entities);
    Task<int> CountAsync();
    Task<bool> ExistsAsync(Expression<Func<T, bool>> predicate);
}

// Repositories/GenericRepository.cs
using Microsoft.EntityFrameworkCore;
using EFCoreProject.Data;
using EFCoreProject.Repositories.Interfaces;
using System.Linq.Expressions;

namespace EFCoreProject.Repositories;

public class GenericRepository<T> : IGenericRepository<T> where T : class
{
    protected readonly ApplicationDbContext _context;
    protected readonly DbSet<T> _dbSet;

    public GenericRepository(ApplicationDbContext context)
    {
        _context = context;
        _dbSet = _context.Set<T>();
    }

    public virtual async Task<T?> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }

    public virtual async Task<IEnumerable<T>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }

    public virtual async Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate)
    {
        return await _dbSet.Where(predicate).ToListAsync();
    }

    public virtual async Task<T?> FirstOrDefaultAsync(Expression<Func<T, bool>> predicate)
    {
        return await _dbSet.FirstOrDefaultAsync(predicate);
    }

    public virtual async Task AddAsync(T entity)
    {
        await _dbSet.AddAsync(entity);
    }

    public virtual async Task AddRangeAsync(IEnumerable<T> entities)
    {
        await _dbSet.AddRangeAsync(entities);
    }

    public virtual void Update(T entity)
    {
        _dbSet.Update(entity);
    }

    public virtual void Remove(T entity)
    {
        _dbSet.Remove(entity);
    }

    public virtual void RemoveRange(IEnumerable<T> entities)
    {
        _dbSet.RemoveRange(entities);
    }

    public virtual async Task<int> CountAsync()
    {
        return await _dbSet.CountAsync();
    }

    public virtual async Task<bool> ExistsAsync(Expression<Func<T, bool>> predicate)
    {
        return await _dbSet.AnyAsync(predicate);
    }
}

// Repositories/Interfaces/IStudentRepository.cs
using EFCoreProject.Models;

namespace EFCoreProject.Repositories.Interfaces;

public interface IStudentRepository : IGenericRepository<Student>
{
    Task<Student?> GetByEmailAsync(string email);
    Task<Student?> GetByStudentNumberAsync(string studentNumber);
    Task<IEnumerable<Student>> GetByDepartmentAsync(int departmentId);
    Task<IEnumerable<Student>> GetStudentsWithEnrollmentsAsync();
}

// Repositories/StudentRepository.cs
using Microsoft.EntityFrameworkCore;
using EFCoreProject.Data;
using EFCoreProject.Models;
using EFCoreProject.Repositories.Interfaces;

namespace EFCoreProject.Repositories;

public class StudentRepository : GenericRepository<Student>, IStudentRepository
{
    public StudentRepository(ApplicationDbContext context) : base(context)
    {
    }

    public async Task<Student?> GetByEmailAsync(string email)
    {
        return await _dbSet
            .Include(s => s.Department)
            .FirstOrDefaultAsync(s => s.Email == email);
    }

    public async Task<Student?> GetByStudentNumberAsync(string studentNumber)
    {
        return await _dbSet
            .Include(s => s.Department)
            .FirstOrDefaultAsync(s => s.StudentNumber == studentNumber);
    }

    public async Task<IEnumerable<Student>> GetByDepartmentAsync(int departmentId)
    {
        return await _dbSet
            .Where(s => s.DepartmentId == departmentId)
            .OrderBy(s => s.LastName)
            .ThenBy(s => s.FirstName)
            .ToListAsync();
    }

    public async Task<IEnumerable<Student>> GetStudentsWithEnrollmentsAsync()
    {
        return await _dbSet
            .Include(s => s.Department)
            .Include(s => s.Enrollments)
                .ThenInclude(e => e.Course)
            .ToListAsync();
    }
}
```

### 9. Unit of Work Pattern

```csharp
// Repositories/Interfaces/IUnitOfWork.cs
using EFCoreProject.Repositories.Interfaces;

namespace EFCoreProject.Repositories.Interfaces;

public interface IUnitOfWork : IDisposable
{
    IStudentRepository Students { get; }
    IGenericRepository<Course> Courses { get; }
    IGenericRepository<Department> Departments { get; }
    IGenericRepository<Enrollment> Enrollments { get; }
    
    Task<int> SaveChangesAsync();
    Task BeginTransactionAsync();
    Task CommitAsync();
    Task RollbackAsync();
}

// Repositories/UnitOfWork.cs
using Microsoft.EntityFrameworkCore.Storage;
using EFCoreProject.Data;
using EFCoreProject.Models;
using EFCoreProject.Repositories.Interfaces;

namespace EFCoreProject.Repositories;

public class UnitOfWork : IUnitOfWork
{
    private readonly ApplicationDbContext _context;
    private IDbContextTransaction? _transaction;

    public UnitOfWork(ApplicationDbContext context)
    {
        _context = context;
        Students = new StudentRepository(_context);
        Courses = new GenericRepository<Course>(_context);
        Departments = new GenericRepository<Department>(_context);
        Enrollments = new GenericRepository<Enrollment>(_context);
    }

    public IStudentRepository Students { get; private set; }
    public IGenericRepository<Course> Courses { get; private set; }
    public IGenericRepository<Department> Departments { get; private set; }
    public IGenericRepository<Enrollment> Enrollments { get; private set; }

    public async Task<int> SaveChangesAsync()
    {
        return await _context.SaveChangesAsync();
    }

    public async Task BeginTransactionAsync()
    {
        _transaction = await _context.Database.BeginTransactionAsync();
    }

    public async Task CommitAsync()
    {
        try
        {
            await _context.SaveChangesAsync();
            if (_transaction != null)
            {
                await _transaction.CommitAsync();
            }
        }
        catch
        {
            await RollbackAsync();
            throw;
        }
        finally
        {
            if (_transaction != null)
            {
                await _transaction.DisposeAsync();
                _transaction = null;
            }
        }
    }

    public async Task RollbackAsync()
    {
        if (_transaction != null)
        {
            await _transaction.RollbackAsync();
            await _transaction.DisposeAsync();
            _transaction = null;
        }
    }

    public void Dispose()
    {
        _transaction?.Dispose();
        _context.Dispose();
    }
}
```

### 10. Seed Data

```csharp
// Data/Seed/SeedData.cs
using EFCoreProject.Data;
using EFCoreProject.Models;
using Microsoft.EntityFrameworkCore;

namespace EFCoreProject.Data.Seed;

public static class SeedData
{
    public static async Task SeedAsync(ApplicationDbContext context)
    {
        if (await context.Departments.AnyAsync())
            return; // Database has been seeded

        // Add Departments
        var departments = new[]
        {
            new Department { Name = "Computer Science", Code = "CS", Description = "Department of Computer Science" },
            new Department { Name = "Mathematics", Code = "MATH", Description = "Department of Mathematics" },
            new Department { Name = "Physics", Code = "PHYS", Description = "Department of Physics" },
            new Department { Name = "Chemistry", Code = "CHEM", Description = "Department of Chemistry" }
        };

        context.Departments.AddRange(departments);
        await context.SaveChangesAsync();

        // Add Courses
        var courses = new[]
        {
            new Course { Title = "Data Structures", CourseCode = "CS101", Credits = 3, DepartmentId = departments[0].Id, Description = "Introduction to Data Structures and Algorithms" },
            new Course { Title = "Database Systems", CourseCode = "CS201", Credits = 4, DepartmentId = departments[0].Id, Description = "Database Design and Implementation" },
            new Course { Title = "Calculus I", CourseCode = "MATH101", Credits = 4, DepartmentId = departments[1].Id, Description = "Differential Calculus" },
            new Course { Title = "Linear Algebra", CourseCode = "MATH201", Credits = 3, DepartmentId = departments[1].Id, Description = "Vectors, Matrices, and Linear Transformations" },
            new Course { Title = "General Physics", CourseCode = "PHYS101", Credits = 4, DepartmentId = departments[2].Id, Description = "Introduction to Mechanics and Thermodynamics" }
        };

        context.Courses.AddRange(courses);
        await context.SaveChangesAsync();

        // Add Students
        var students = new[]
        {
            new Student { FirstName = "สมชาย", LastName = "ใจดี", Email = "somchai@university.com", StudentNumber = "6501001", DateOfBirth = new DateTime(2000, 5, 15), DepartmentId = departments[0].Id },
            new Student { FirstName = "สมหญิง", LastName = "สุขใจ", Email = "somying@university.com", StudentNumber = "6501002", DateOfBirth = new DateTime(2001, 3, 22), DepartmentId = departments[0].Id },
            new Student { FirstName = "วิชัย", LastName = "เก่งเรียน", Email = "vichai@university.com", StudentNumber = "6502001", DateOfBirth = new DateTime(1999, 11, 8), DepartmentId = departments[1].Id },
            new Student { FirstName = "มาลี", LastName = "รักเรียน", Email = "malee@university.com", StudentNumber = "6502002", DateOfBirth = new DateTime(2000, 7, 30), DepartmentId = departments[1].Id },
            new Student { FirstName = "ปิติ", LastName = "สุขสบาย", Email = "piti@university.com", StudentNumber = "6503001", DateOfBirth = new DateTime(2001, 1, 12), DepartmentId = departments[2].Id }
        };

        context.Students.AddRange(students);
        await context.SaveChangesAsync();

        // Add Enrollments
        var enrollments = new[]
        {
            new Enrollment { StudentId = students[0].Id, CourseId = courses[0].Id, EnrollmentDate = DateTime.Now.AddDays(-30), Status = EnrollmentStatus.Enrolled },
            new Enrollment { StudentId = students[0].Id, CourseId = courses[1].Id, EnrollmentDate = DateTime.Now.AddDays(-25), Status = EnrollmentStatus.Enrolled },
            new Enrollment { StudentId = students[1].Id, CourseId = courses[0].Id, EnrollmentDate = DateTime.Now.AddDays(-28), Status = EnrollmentStatus.Enrolled },
            new Enrollment { StudentId = students[2].Id, CourseId = courses[2].Id, EnrollmentDate = DateTime.Now.AddDays(-35), Status = EnrollmentStatus.Completed, Grade = 3.5 },
            new Enrollment { StudentId = students[2].Id, CourseId = courses[3].Id, EnrollmentDate = DateTime.Now.AddDays(-20), Status = EnrollmentStatus.Enrolled },
            new Enrollment { StudentId = students[3].Id, CourseId = courses[2].Id, EnrollmentDate = DateTime.Now.AddDays(-32), Status = EnrollmentStatus.Enrolled },
            new Enrollment { StudentId = students[4].Id, CourseId = courses[4].Id, EnrollmentDate = DateTime.Now.AddDays(-15), Status = EnrollmentStatus.Enrolled }
        };

        context.Enrollments.AddRange(enrollments);
        await context.SaveChangesAsync();
    }
}
```

### 11. Program.cs

```csharp
// Program.cs
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
using EFCoreProject.Data;
using EFCoreProject.Data.Seed;
using EFCoreProject.Repositories;
using EFCoreProject.Repositories.Interfaces;

var host = Host.CreateDefaultBuilder(args)
    .ConfigureServices((context, services) =>
    {
        // Database Configuration
        var connectionString = context.Configuration.GetConnectionString("DefaultConnection");
        services.AddDbContext<ApplicationDbContext>(options =>
        {
            options.UseSqlServer(connectionString);
            
            if (context.HostingEnvironment.IsDevelopment())
            {
                options.EnableSensitiveDataLogging();
                options.EnableDetailedErrors();
                options.LogTo(Console.WriteLine, LogLevel.Information);
            }
        });

        // Repository Registration
        services.AddScoped<IUnitOfWork, UnitOfWork>();
        services.AddScoped<IStudentRepository, StudentRepository>();
        services.AddScoped(typeof(IGenericRepository<>), typeof(GenericRepository<>));

        // Add AutoMapper if needed
        // services.AddAutoMapper(typeof(Program));
    })
    .Build();

// Application Logic
using (var scope = host.Services.CreateScope())
{
    var logger = scope.ServiceProvider.GetRequiredService<ILogger<Program>>();
    var context = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
    var unitOfWork = scope.ServiceProvider.GetRequiredService<IUnitOfWork>();

    try
    {
        logger.LogInformation("🚀 Starting EF Core Project Application");

        // Ensure database is created
        logger.LogInformation("📊 Ensuring database exists...");
        await context.Database.EnsureCreatedAsync();

        // Seed data
        logger.LogInformation("🌱 Seeding initial data...");
        await SeedData.SeedAsync(context);

        // Example Usage
        logger.LogInformation("👥 Fetching all students...");
        var students = await unitOfWork.Students.GetStudentsWithEnrollmentsAsync();
        
        logger.LogInformation($"📋 Found {students.Count()} students:");
        foreach (var student in students)
        {
            logger.LogInformation($"   - {student.FullName} ({student.StudentNumber}) - {student.Department?.Name}");
            foreach (var enrollment in student.Enrollments)
            {
                logger.LogInformation($"     📚 {enrollment.Course.Title} - {enrollment.Status}");
            }
        }

        // Example: Add new student
        logger.LogInformation("➕ Adding new student...");
        await unitOfWork.BeginTransactionAsync();
        
        try
        {
            var newStudent = new Models.Student
            {
                FirstName = "ทดสอบ",
                LastName = "ใหม่",
                Email = "test@university.com",
                StudentNumber = "6504001",
                DateOfBirth = new DateTime(2002, 6, 15),
                DepartmentId = (await unitOfWork.Departments.FirstOrDefaultAsync(d => d.Code == "CS"))?.Id
            };

            await unitOfWork.Students.AddAsync(newStudent);
            await unitOfWork.CommitAsync();
            
            logger.LogInformation($"✅ Successfully added student: {newStudent.FullName}");
        }
        catch (Exception ex)
        {
            await unitOfWork.RollbackAsync();
            logger.LogError(ex, "❌ Failed to add new student");
        }

        logger.LogInformation("🎉 Application completed successfully!");
    }
    catch (Exception ex)
    {
        logger.LogError(ex, "💥 Application failed with error: {Message}", ex.Message);
    }
}

Console.WriteLine("\nPress any key to exit...");
Console.ReadKey();
```

---

## 🗂️ ตัวอย่าง Project 2: Multiple Databases

### Project Structure สำหรับหลาย Database

```
MultiDbProject/
├── 📁 src/
│   └── 📁 MultiDbProject/
│       ├── 📁 Data/
│       │   ├── 📁 HumanResources/         # HR Database
│       │   │   ├── HRDbContext.cs
│       │   │   ├── Models/
│       │   │   │   ├── Employee.cs
│       │   │   │   └── Department.cs
│       │   │   └── Configurations/
│       │   ├── 📁 Finance/               # Finance Database
│       │   │   ├── FinanceDbContext.cs
│       │   │   ├── Models/
│       │   │   │   ├── Account.cs
│       │   │   │   └── Transaction.cs
│       │   │   └── Configurations/
│       │   └── 📁 Inventory/             # Inventory Database
│       │       ├── InventoryDbContext.cs
│       │       ├── Models/
│       │       │   ├── Product.cs
│       │       │   └── Category.cs
│       │       └── Configurations/
│       ├── 📁 Services/
│       │   ├── CrossDatabaseService.cs
│       │   └── Interfaces/
│       ├── appsettings.json
│       └── Program.cs
```

### Multiple Database Configuration

```json
// appsettings.json
{
  "ConnectionStrings": {
    "HRConnection": "Server=(localdb)\\mssqllocaldb;Database=CompanyHR;Trusted_Connection=true;",
    "FinanceConnection": "Server=(localdb)\\mssqllocaldb;Database=CompanyFinance;Trusted_Connection=true;",
    "InventoryConnection": "Server=(localdb)\\mssqllocaldb;Database=CompanyInventory;Trusted_Connection=true;"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.EntityFrameworkCore": "Information"
    }
  }
}
```

### HR Database Context

```csharp
// Data/HumanResources/Models/Employee.cs
namespace MultiDbProject.Data.HumanResources.Models;

public class Employee
{
    public int Id { get; set; }
    public string FirstName { get; set; } = string.Empty;
    public string LastName { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public string EmployeeNumber { get; set; } = string.Empty;
    public DateTime HireDate { get; set; }
    public decimal Salary { get; set; }
    public int DepartmentId { get; set; }
    public virtual Department Department { get; set; } = null!;
    public bool IsActive { get; set; } = true;
    public DateTime CreatedAt { get; set; } = DateTime.Now;
}

public class Department
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Code { get; set; } = string.Empty;
    public string? ManagerEmployeeNumber { get; set; }
    public decimal Budget { get; set; }
    public virtual ICollection<Employee> Employees { get; set; } = new List<Employee>();
    public DateTime CreatedAt { get; set; } = DateTime.Now;
}

// Data/HumanResources/HRDbContext.cs
using Microsoft.EntityFrameworkCore;
using MultiDbProject.Data.HumanResources.Models;

namespace MultiDbProject.Data.HumanResources;

public class HRDbContext : DbContext
{
    public HRDbContext(DbContextOptions<HRDbContext> options) : base(options)
    {
    }

    public DbSet<Employee> Employees { get; set; } = null!;
    public DbSet<Department> Departments { get; set; } = null!;

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Employee>(entity =>
        {
            entity.ToTable("Employees", "HR");
            entity.HasKey(e => e.Id);
            entity.Property(e => e.FirstName).IsRequired().HasMaxLength(50);
            entity.Property(e => e.LastName).IsRequired().HasMaxLength(50);
            entity.Property(e => e.Email).IsRequired().HasMaxLength(100);
            entity.Property(e => e.EmployeeNumber).IsRequired().HasMaxLength(20);
            entity.Property(e => e.Salary).HasColumnType("decimal(18,2)");
            
            entity.HasIndex(e => e.Email).IsUnique();
            entity.HasIndex(e => e.EmployeeNumber).IsUnique();
            
            entity.HasOne(e => e.Department)
                .WithMany(d => d.Employees)
                .HasForeignKey(e => e.DepartmentId);
        });

        modelBuilder.Entity<Department>(entity =>
        {
            entity.ToTable("Departments", "HR");
            entity.HasKey(d => d.Id);
            entity.Property(d => d.Name).IsRequired().HasMaxLength(100);
            entity.Property(d => d.Code).IsRequired().HasMaxLength(10);
            entity.Property(d => d.Budget).HasColumnType("decimal(18,2)");
            
            entity.HasIndex(d => d.Code).IsUnique();
        });
    }
}
```

### Finance Database Context

```csharp
// Data/Finance/Models/Account.cs
namespace MultiDbProject.Data.Finance.Models;

public class Account
{
    public int Id { get; set; }
    public string AccountNumber { get; set; } = string.Empty;
    public string AccountName { get; set; } = string.Empty;
    public AccountType AccountType { get; set; }
    public decimal Balance { get; set; }
    public bool IsActive { get; set; } = true;
    public virtual ICollection<Transaction> Transactions { get; set; } = new List<Transaction>();
    public DateTime CreatedAt { get; set; } = DateTime.Now;
}

public class Transaction
{
    public int Id { get; set; }
    public string TransactionNumber { get; set; } = string.Empty;
    public int AccountId { get; set; }
    public virtual Account Account { get; set; } = null!;
    public TransactionType Type { get; set; }
    public decimal Amount { get; set; }
    public string Description { get; set; } = string.Empty;
    public string? ReferenceNumber { get; set; }
    public DateTime TransactionDate { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.Now;
}

public enum AccountType
{
    Asset = 1,
    Liability = 2,
    Equity = 3,
    Revenue = 4,
    Expense = 5
}

public enum TransactionType
{
    Debit = 1,
    Credit = 2
}

// Data/Finance/FinanceDbContext.cs
using Microsoft.EntityFrameworkCore;
using MultiDbProject.Data.Finance.Models;

namespace MultiDbProject.Data.Finance;

public class FinanceDbContext : DbContext
{
    public FinanceDbContext(DbContextOptions<FinanceDbContext> options) : base(options)
    {
    }

    public DbSet<Account> Accounts { get; set; } = null!;
    public DbSet<Transaction> Transactions { get; set; } = null!;

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Account>(entity =>
        {
            entity.ToTable("Accounts", "Finance");
            entity.HasKey(a => a.Id);
            entity.Property(a => a.AccountNumber).IsRequired().HasMaxLength(20);
            entity.Property(a => a.AccountName).IsRequired().HasMaxLength(100);
            entity.Property(a => a.Balance).HasColumnType("decimal(18,2)");
            
            entity.HasIndex(a => a.AccountNumber).IsUnique();
        });

        modelBuilder.Entity<Transaction>(entity =>
        {
            entity.ToTable("Transactions", "Finance");
            entity.HasKey(t => t.Id);
            entity.Property(t => t.TransactionNumber).IsRequired().HasMaxLength(20);
            entity.Property(t => t.Amount).HasColumnType("decimal(18,2)");
            entity.Property(t => t.Description).IsRequired().HasMaxLength(500);
            
            entity.HasIndex(t => t.TransactionNumber).IsUnique();
            entity.HasIndex(t => t.TransactionDate);
            
            entity.HasOne(t => t.Account)
                .WithMany(a => a.Transactions)
                .HasForeignKey(t => t.AccountId);
        });
    }
}
```

### Cross-Database Service

```csharp
// Services/Interfaces/ICrossDatabaseService.cs
namespace MultiDbProject.Services.Interfaces;

public interface ICrossDatabaseService
{
    Task<object> GetEmployeeFinancialSummaryAsync(string employeeNumber);
    Task<bool> ProcessEmployeePayrollAsync(string employeeNumber, decimal amount);
    Task<object> GetDepartmentBudgetReportAsync(int departmentId);
}

// Services/CrossDatabaseService.cs
using Microsoft.EntityFrameworkCore;
using MultiDbProject.Data.HumanResources;
using MultiDbProject.Data.Finance;
using MultiDbProject.Services.Interfaces;

namespace MultiDbProject.Services;

public class CrossDatabaseService : ICrossDatabaseService
{
    private readonly HRDbContext _hrContext;
    private readonly FinanceDbContext _financeContext;
    private readonly ILogger<CrossDatabaseService> _logger;

    public CrossDatabaseService(
        HRDbContext hrContext,
        FinanceDbContext financeContext,
        ILogger<CrossDatabaseService> logger)
    {
        _hrContext = hrContext;
        _financeContext = financeContext;
        _logger = logger;
    }

    public async Task<object> GetEmployeeFinancialSummaryAsync(string employeeNumber)
    {
        // Get employee from HR database
        var employee = await _hrContext.Employees
            .Include(e => e.Department)
            .FirstOrDefaultAsync(e => e.EmployeeNumber == employeeNumber);

        if (employee == null)
        {
            throw new ArgumentException($"Employee {employeeNumber} not found");
        }

        // Get related transactions from Finance database
        var salaryAccount = await _financeContext.Accounts
            .FirstOrDefaultAsync(a => a.AccountNumber == $"SAL-{employeeNumber}");

        var transactions = new List<object>();
        if (salaryAccount != null)
        {
            transactions = await _financeContext.Transactions
                .Where(t => t.AccountId == salaryAccount.Id)
                .OrderByDescending(t => t.TransactionDate)
                .Take(10)
                .Select(t => new
                {
                    t.TransactionNumber,
                    t.Amount,
                    t.Type,
                    t.Description,
                    t.TransactionDate
                })
                .Cast<object>()
                .ToListAsync();
        }

        return new
        {
            Employee = new
            {
                employee.EmployeeNumber,
                employee.FirstName,
                employee.LastName,
                employee.Email,
                employee.Salary,
                Department = employee.Department.Name
            },
            FinancialInfo = new
            {
                SalaryAccountBalance = salaryAccount?.Balance ?? 0,
                RecentTransactions = transactions
            }
        };
    }

    public async Task<bool> ProcessEmployeePayrollAsync(string employeeNumber, decimal amount)
    {
        using var hrTransaction = await _hrContext.Database.BeginTransactionAsync();
        using var financeTransaction = await _financeContext.Database.BeginTransactionAsync();

        try
        {
            // Verify employee exists in HR database
            var employee = await _hrContext.Employees
                .FirstOrDefaultAsync(e => e.EmployeeNumber == employeeNumber);

            if (employee == null)
            {
                throw new ArgumentException($"Employee {employeeNumber} not found");
            }

            // Get or create salary account in Finance database
            var salaryAccount = await _financeContext.Accounts
                .FirstOrDefaultAsync(a => a.AccountNumber == $"SAL-{employeeNumber}");

            if (salaryAccount == null)
            {
                salaryAccount = new Data.Finance.Models.Account
                {
                    AccountNumber = $"SAL-{employeeNumber}",
                    AccountName = $"Salary Account - {employee.FirstName} {employee.LastName}",
                    AccountType = Data.Finance.Models.AccountType.Liability,
                    Balance = 0
                };
                _financeContext.Accounts.Add(salaryAccount);
                await _financeContext.SaveChangesAsync();
            }

            // Create payroll transaction
            var transaction = new Data.Finance.Models.Transaction
            {
                TransactionNumber = $"PAY-{DateTime.Now:yyyyMMdd}-{employeeNumber}",
                AccountId = salaryAccount.Id,
                Type = Data.Finance.Models.TransactionType.Credit,
                Amount = amount,
                Description = $"Salary payment for {employee.FirstName} {employee.LastName}",
                ReferenceNumber = employeeNumber,
                TransactionDate = DateTime.Now
            };

            _financeContext.Transactions.Add(transaction);
            salaryAccount.Balance += amount;

            // Save changes
            await _financeContext.SaveChangesAsync();
            await _hrContext.SaveChangesAsync();

            // Commit both transactions
            await financeTransaction.CommitAsync();
            await hrTransaction.CommitAsync();

            _logger.LogInformation("Successfully processed payroll for employee {EmployeeNumber}, Amount: {Amount}",
                employeeNumber, amount);

            return true;
        }
        catch (Exception ex)
        {
            await financeTransaction.RollbackAsync();
            await hrTransaction.RollbackAsync();
            _logger.LogError(ex, "Failed to process payroll for employee {EmployeeNumber}", employeeNumber);
            throw;
        }
    }

    public async Task<object> GetDepartmentBudgetReportAsync(int departmentId)
    {
        var department = await _hrContext.Departments
            .Include(d => d.Employees)
            .FirstOrDefaultAsync(d => d.Id == departmentId);

        if (department == null)
        {
            throw new ArgumentException($"Department {departmentId} not found");
        }

        var employeeNumbers = department.Employees.Select(e => e.EmployeeNumber).ToList();
        var salaryAccounts = await _financeContext.Accounts
            .Where(a => employeeNumbers.Any(emp => a.AccountNumber == $"SAL-{emp}"))
            .ToListAsync();

        var totalPayroll = salaryAccounts.Sum(a => a.Balance);

        return new
        {
            Department = new
            {
                department.Name,
                department.Code,
                department.Budget
            },
            EmployeeCount = department.Employees.Count,
            TotalSalaryBudget = department.Employees.Sum(e => e.Salary),
            TotalPayrollPaid = totalPayroll,
            BudgetUtilization = department.Budget > 0 ? (totalPayroll / department.Budget) * 100 : 0
        };
    }
}
```

### Program.cs สำหรับ Multiple Databases

```csharp
// Program.cs
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
using MultiDbProject.Data.HumanResources;
using MultiDbProject.Data.Finance;
using MultiDbProject.Services;
using MultiDbProject.Services.Interfaces;

var host = Host.CreateDefaultBuilder(args)
    .ConfigureServices((context, services) =>
    {
        // Register multiple DbContexts
        services.AddDbContext<HRDbContext>(options =>
            options.UseSqlServer(context.Configuration.GetConnectionString("HRConnection")));

        services.AddDbContext<FinanceDbContext>(options =>
            options.UseSqlServer(context.Configuration.GetConnectionString("FinanceConnection")));

        // Register services
        services.AddScoped<ICrossDatabaseService, CrossDatabaseService>();
    })
    .Build();

using (var scope = host.Services.CreateScope())
{
    var logger = scope.ServiceProvider.GetRequiredService<ILogger<Program>>();
    var hrContext = scope.ServiceProvider.GetRequiredService<HRDbContext>();
    var financeContext = scope.ServiceProvider.GetRequiredService<FinanceDbContext>();
    var crossDbService = scope.ServiceProvider.GetRequiredService<ICrossDatabaseService>();

    try
    {
        logger.LogInformation("🚀 Starting Multi-Database Application");

        // Ensure databases exist
        await hrContext.Database.EnsureCreatedAsync();
        await financeContext.Database.EnsureCreatedAsync();

        // Seed HR data
        if (!await hrContext.Departments.AnyAsync())
        {
            var department = new Data.HumanResources.Models.Department
            {
                Name = "Information Technology",
                Code = "IT",
                Budget = 100000
            };
            hrContext.Departments.Add(department);
            await hrContext.SaveChangesAsync();

            var employee = new Data.HumanResources.Models.Employee
            {
                FirstName = "สมชาย",
                LastName = "นักพัฒนา",
                Email = "somchai.dev@company.com",
                EmployeeNumber = "EMP001",
                HireDate = DateTime.Now.AddYears(-2),
                Salary = 50000,
                DepartmentId = department.Id
            };
            hrContext.Employees.Add(employee);
            await hrContext.SaveChangesAsync();
        }

        // Demo: Cross-database operations
        logger.LogInformation("💰 Processing payroll...");
        await crossDbService.ProcessEmployeePayrollAsync("EMP001", 50000);

        logger.LogInformation("📊 Getting employee financial summary...");
        var summary = await crossDbService.GetEmployeeFinancialSummaryAsync("EMP001");
        logger.LogInformation("Employee Summary: {Summary}", System.Text.Json.JsonSerializer.Serialize(summary, new System.Text.Json.JsonSerializerOptions { WriteIndented = true }));

        logger.LogInformation("🎉 Multi-Database Application completed successfully!");
    }
    catch (Exception ex)
    {
        logger.LogError(ex, "💥 Application failed: {Message}", ex.Message);
    }
}

Console.WriteLine("Press any key to exit...");
Console.ReadKey();
```

---

## 📋 สรุป: สิ่งที่จำเป็นสำหรับ EF Core Project

### ✅ **สำหรับ Single Database Project**
1. **Packages:** EF Core, Provider (SQL Server/SQLite), Tools, Design
2. **Configuration:** appsettings.json, Connection Strings
3. **Models:** Entity classes พร้อม Data Annotations
4. **DbContext:** Configuration และ DbSets
5. **Entity Configurations:** Fluent API สำหรับ complex mapping
6. **Repository Pattern:** สำหรับ Data Access Layer
7. **Unit of Work:** สำหรับ Transaction Management
8. **Seed Data:** ข้อมูลเริ่มต้น

### ✅ **สำหรับ Multiple Database Project**
1. **Multiple DbContexts:** แยก DbContext ต่าง Database
2. **Separate Configurations:** Connection Strings แยกกัน
3. **Cross-Database Services:** สำหรับ operations ข้าม Database
4. **Transaction Coordination:** สำหรับ distributed transactions
5. **Proper Error Handling:** Rollback strategies

### 🎯 **Best Practices**
- ใช้ Repository Pattern สำหรับ testability
- แยก Configuration files ตาม environment
- ใช้ Logging สำหรับ debugging
- มี Error Handling ที่เหมาะสม
- ใช้ Dependency Injection
- มี Seed Data สำหรับ development

---

## 🔗 การนำไปใช้จริง

### ขั้นตอนการ Deploy

1. **Development Environment**
   ```bash
   # ใช้ Development connection string
   dotnet ef database update
   ```

2. **Production Environment**
   ```bash
   # ใช้ Production connection string
   dotnet ef migrations script --idempotent --output migrations.sql
   # นำไฟล์ migrations.sql ไป run ใน Production
   ```

3. **Docker Container**
   ```dockerfile
   FROM mcr.microsoft.com/dotnet/aspnet:6.0
   WORKDIR /app
   COPY . .
   ENTRYPOINT ["dotnet", "EFCoreProject.dll"]
   ```

### 💡 เคล็ดลับ

- ✅ ใช้ Environment Variables สำหรับ Connection Strings ใน Production
- ✅ ใช้ Migration Scripts แทนการ run `database update` ตรงใน Production
- ✅ มี Backup Database ก่อน deploy เสมอ
- ✅ Test Migration ใน Staging Environment ก่อน
- ✅ Monitor performance หลัง deploy

---

## 📚 Navigation

- [← บทที่ 15: การทดสอบ (Testing) กับ EF Core](15-testing.md)
- [→ บทที่ 17: Advanced Patterns และ Architectures](17-advanced-patterns.md)
- [📖 กลับไปหน้าหลัก](README.md)