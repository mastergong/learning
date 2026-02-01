# บทที่ 13: Database First และ Scaffolding

## รู้จักกับ Database First Approach

Database First เป็นแนวทางการพัฒนาที่เริ่มต้นจากฐานข้อมูลที่มีอยู่แล้ว แล้วสร้าง Models และ DbContext ให้ตรงกับโครงสร้างฐานข้อมูลนั้น

### สถานการณ์ที่ใช้ Database First

1. **ฐานข้อมูลมีอยู่แล้ว** - มีระบบเก่าที่ต้องการนำมาใช้งาน
2. **ทำงานร่วมกับ DBA** - Database Administrator ออกแบบฐานข้อมูลแล้ว
3. **ระบบขนาดใหญ่** - ฐานข้อมูลซับซ้อนที่สร้างจาก Script
4. **Legacy System Integration** - เชื่อมต่อกับระบบเก่า

---

## การติดตั้ง Tools ที่จำเป็น

### 1. ติดตั้ง EF Core Tools

```bash
# ติดตั้ง Global Tool
dotnet tool install --global dotnet-ef

# อัพเดท Tool (ถ้าติดตั้งแล้ว)
dotnet tool update --global dotnet-ef

# ตรวจสอบเวอร์ชั่น
dotnet ef
```

### 2. เพิ่ม Package ในโปรเจค

```xml
<!-- ใน .csproj file -->
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="6.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="6.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="6.0.0" />
```

---

## คำสั่ง Scaffold พื้นฐาน

### 1. คำสั่งสำหรับ SQL Server

```bash
dotnet ef dbcontext scaffold "Server=(localdb)\mssqllocaldb;Database=SchoolDB;Trusted_Connection=true;" Microsoft.EntityFrameworkCore.SqlServer
```

### 2. คำสั่งแบบละเอียด

```bash
dotnet ef dbcontext scaffold "Data Source=localhost;Initial Catalog=SchoolDB;Integrated Security=True" Microsoft.EntityFrameworkCore.SqlServer --output-dir Models --context-dir Data --context SchoolContext --force
```

### พารามิเตอร์สำคัญ

- `--output-dir Models` - โฟลเดอร์ที่เก็บ Model classes
- `--context-dir Data` - โฟลเดอร์ที่เก็บ DbContext
- `--context SchoolContext` - ชื่อ DbContext class
- `--force` - เขียนทับไฟล์ที่มีอยู่แล้ว
- `--tables Table1,Table2` - เลือกเฉพาะตารางที่ต้องการ
- `--schemas schema1,schema2` - เลือกเฉพาะ schema ที่ต้องการ

---

## ตัวอย่างการ Scaffold

### ฐานข้อมูลตัวอย่าง

```sql
-- สร้างฐานข้อมูลตัวอย่าง
CREATE DATABASE CompanyDB;
GO

USE CompanyDB;
GO

-- ตารางแผนก
CREATE TABLE Departments (
    DepartmentId INT PRIMARY KEY IDENTITY(1,1),
    DepartmentName NVARCHAR(100) NOT NULL,
    Location NVARCHAR(100),
    Budget DECIMAL(15,2),
    CreatedDate DATETIME2 DEFAULT GETDATE()
);

-- ตารางพนักงาน
CREATE TABLE Employees (
    EmployeeId INT PRIMARY KEY IDENTITY(1,1),
    FirstName NVARCHAR(50) NOT NULL,
    LastName NVARCHAR(50) NOT NULL,
    Email NVARCHAR(100) UNIQUE NOT NULL,
    Phone NVARCHAR(20),
    HireDate DATE NOT NULL,
    Salary DECIMAL(10,2),
    DepartmentId INT,
    IsActive BIT DEFAULT 1,
    FOREIGN KEY (DepartmentId) REFERENCES Departments(DepartmentId)
);

-- ตารางโครงการ
CREATE TABLE Projects (
    ProjectId INT PRIMARY KEY IDENTITY(1,1),
    ProjectName NVARCHAR(200) NOT NULL,
    Description NVARCHAR(MAX),
    StartDate DATE,
    EndDate DATE,
    Budget DECIMAL(15,2),
    Status NVARCHAR(50) DEFAULT 'Planning'
);

-- ตารางความสัมพันธ์ พนักงาน-โครงการ
CREATE TABLE EmployeeProjects (
    EmployeeId INT,
    ProjectId INT,
    Role NVARCHAR(100),
    AssignedDate DATE DEFAULT GETDATE(),
    PRIMARY KEY (EmployeeId, ProjectId),
    FOREIGN KEY (EmployeeId) REFERENCES Employees(EmployeeId),
    FOREIGN KEY (ProjectId) REFERENCES Projects(ProjectId)
);
```

### คำสั่ง Scaffold

```bash
dotnet ef dbcontext scaffold "Server=(localdb)\mssqllocaldb;Database=CompanyDB;Trusted_Connection=true;" Microsoft.EntityFrameworkCore.SqlServer --output-dir Models --context-dir Data --context CompanyDbContext --force
```

---

## ผลลัพธ์ที่ได้จาก Scaffolding

### 1. DbContext ที่สร้างขึ้น

```csharp
// Data/CompanyDbContext.cs
using Microsoft.EntityFrameworkCore;
using Models;

namespace Data
{
    public partial class CompanyDbContext : DbContext
    {
        public CompanyDbContext()
        {
        }

        public CompanyDbContext(DbContextOptions<CompanyDbContext> options)
            : base(options)
        {
        }

        public virtual DbSet<Department> Departments { get; set; } = null!;
        public virtual DbSet<Employee> Employees { get; set; } = null!;
        public virtual DbSet<Project> Projects { get; set; } = null!;
        public virtual DbSet<EmployeeProject> EmployeeProjects { get; set; } = null!;

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            if (!optionsBuilder.IsConfigured)
            {
                optionsBuilder.UseSqlServer("Server=(localdb)\\mssqllocaldb;Database=CompanyDB;Trusted_Connection=true;");
            }
        }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Department>(entity =>
            {
                entity.HasKey(e => e.DepartmentId);
                entity.Property(e => e.DepartmentName)
                    .HasMaxLength(100)
                    .IsRequired();
                entity.Property(e => e.Location).HasMaxLength(100);
                entity.Property(e => e.Budget).HasColumnType("decimal(15,2)");
                entity.Property(e => e.CreatedDate)
                    .HasDefaultValueSql("(getdate())")
                    .HasColumnType("datetime2");
            });

            modelBuilder.Entity<Employee>(entity =>
            {
                entity.HasKey(e => e.EmployeeId);
                entity.HasIndex(e => e.Email).IsUnique();
                entity.Property(e => e.FirstName)
                    .HasMaxLength(50)
                    .IsRequired();
                entity.Property(e => e.LastName)
                    .HasMaxLength(50)
                    .IsRequired();
                entity.Property(e => e.Email)
                    .HasMaxLength(100)
                    .IsRequired();
                entity.Property(e => e.Phone).HasMaxLength(20);
                entity.Property(e => e.Salary).HasColumnType("decimal(10,2)");
                entity.Property(e => e.IsActive)
                    .HasDefaultValueSql("((1))");

                entity.HasOne(d => d.Department)
                    .WithMany(p => p.Employees)
                    .HasForeignKey(d => d.DepartmentId)
                    .HasConstraintName("FK__Employees__Depar__XXX");
            });

            modelBuilder.Entity<EmployeeProject>(entity =>
            {
                entity.HasKey(e => new { e.EmployeeId, e.ProjectId });
                entity.Property(e => e.Role).HasMaxLength(100);
                entity.Property(e => e.AssignedDate)
                    .HasDefaultValueSql("(getdate())");

                entity.HasOne(d => d.Employee)
                    .WithMany(p => p.EmployeeProjects)
                    .HasForeignKey(d => d.EmployeeId)
                    .OnDelete(DeleteBehavior.ClientSetNull);

                entity.HasOne(d => d.Project)
                    .WithMany(p => p.EmployeeProjects)
                    .HasForeignKey(d => d.ProjectId)
                    .OnDelete(DeleteBehavior.ClientSetNull);
            });

            OnModelCreatingPartial(modelBuilder);
        }

        partial void OnModelCreatingPartial(ModelBuilder modelBuilder);
    }
}
```

### 2. Model Classes ที่สร้างขึ้น

```csharp
// Models/Department.cs
namespace Models
{
    public partial class Department
    {
        public Department()
        {
            Employees = new HashSet<Employee>();
        }

        public int DepartmentId { get; set; }
        public string DepartmentName { get; set; } = null!;
        public string? Location { get; set; }
        public decimal? Budget { get; set; }
        public DateTime CreatedDate { get; set; }

        public virtual ICollection<Employee> Employees { get; set; }
    }
}

// Models/Employee.cs
namespace Models
{
    public partial class Employee
    {
        public Employee()
        {
            EmployeeProjects = new HashSet<EmployeeProject>();
        }

        public int EmployeeId { get; set; }
        public string FirstName { get; set; } = null!;
        public string LastName { get; set; } = null!;
        public string Email { get; set; } = null!;
        public string? Phone { get; set; }
        public DateTime HireDate { get; set; }
        public decimal? Salary { get; set; }
        public int? DepartmentId { get; set; }
        public bool? IsActive { get; set; }

        public virtual Department? Department { get; set; }
        public virtual ICollection<EmployeeProject> EmployeeProjects { get; set; }
    }
}

// Models/EmployeeProject.cs
namespace Models
{
    public partial class EmployeeProject
    {
        public int EmployeeId { get; set; }
        public int ProjectId { get; set; }
        public string? Role { get; set; }
        public DateTime? AssignedDate { get; set; }

        public virtual Employee Employee { get; set; } = null!;
        public virtual Project Project { get; set; } = null!;
    }
}
```

---

## การปรับแต่งหลัง Scaffolding

### 1. ย้าย Connection String

```csharp
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=CompanyDB;Trusted_Connection=true;"
  }
}

// Program.cs หรือ Startup.cs
builder.Services.AddDbContext<CompanyDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

### 2. ลบ Connection String จาก OnConfiguring

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    // ลบหรือ comment ออก เพื่อใช้ DI แทน
    /*
    if (!optionsBuilder.IsConfigured)
    {
        optionsBuilder.UseSqlServer("...");
    }
    */
}
```

### 3. เพิ่ม Custom Properties และ Methods

```csharp
// Models/Employee.cs
public partial class Employee
{
    // เพิ่ม computed properties
    public string FullName => $"{FirstName} {LastName}";
    
    public int YearsOfService => DateTime.Now.Year - HireDate.Year;
    
    public bool IsLongTermEmployee => YearsOfService >= 5;
}
```

---

## คำสั่ง Scaffolding ขั้นสูง

### 1. เลือกเฉพาะตารางที่ต้องการ

```bash
dotnet ef dbcontext scaffold "connection-string" Microsoft.EntityFrameworkCore.SqlServer --tables Employees,Departments --output-dir Models --context CompanyDbContext --force
```

### 2. เลือกเฉพาะ Schema

```bash
dotnet ef dbcontext scaffold "connection-string" Microsoft.EntityFrameworkCore.SqlServer --schemas HR,Finance --output-dir Models --context CompanyDbContext --force
```

### 3. ใช้ Namespace แยก

```bash
dotnet ef dbcontext scaffold "connection-string" Microsoft.EntityFrameworkCore.SqlServer --output-dir Models --namespace Company.Models --context-namespace Company.Data --context CompanyDbContext --force
```

### 4. ไม่สร้าง OnConfiguring Method

```bash
dotnet ef dbcontext scaffold "connection-string" Microsoft.EntityFrameworkCore.SqlServer --output-dir Models --context CompanyDbContext --no-onconfiguring --force
```

---

## การอัพเดท Models เมื่อฐานข้อมูลเปลี่ยนแปลง

### 1. Re-scaffold ทั้งหมด

```bash
# จะเขียนทับไฟล์เก่าทั้งหมด
dotnet ef dbcontext scaffold "connection-string" Microsoft.EntityFrameworkCore.SqlServer --output-dir Models --context CompanyDbContext --force
```

### 2. Backup Custom Code

```csharp
// สร้าง partial class แยกสำหรับ custom code
// Models/Employee.Custom.cs
public partial class Employee
{
    public string FullName => $"{FirstName} {LastName}";
    
    public void UpdateSalary(decimal newSalary)
    {
        if (newSalary > 0)
        {
            Salary = newSalary;
        }
    }
}
```

### 3. ใช้ OnModelCreatingPartial

```csharp
// Data/CompanyDbContext.Custom.cs
public partial class CompanyDbContext
{
    partial void OnModelCreatingPartial(ModelBuilder modelBuilder)
    {
        // Custom configurations ที่ไม่ต้องการให้ re-scaffold เขียนทับ
        modelBuilder.Entity<Employee>()
            .HasQueryFilter(e => e.IsActive == true);
            
        modelBuilder.Entity<Department>()
            .HasIndex(d => d.DepartmentName)
            .IsUnique();
    }
}
```

---

## การใช้งานจริง

### 1. การ Query ข้อมูล

```csharp
public class EmployeeService
{
    private readonly CompanyDbContext _context;

    public EmployeeService(CompanyDbContext context)
    {
        _context = context;
    }

    public async Task<List<Employee>> GetEmployeesWithDepartmentAsync()
    {
        return await _context.Employees
            .Include(e => e.Department)
            .Where(e => e.IsActive == true)
            .OrderBy(e => e.LastName)
            .ToListAsync();
    }

    public async Task<Employee?> GetEmployeeWithProjectsAsync(int employeeId)
    {
        return await _context.Employees
            .Include(e => e.Department)
            .Include(e => e.EmployeeProjects)
                .ThenInclude(ep => ep.Project)
            .FirstOrDefaultAsync(e => e.EmployeeId == employeeId);
    }
}
```

### 2. การจัดการข้อมูล

```csharp
public class DepartmentService
{
    private readonly CompanyDbContext _context;

    public DepartmentService(CompanyDbContext context)
    {
        _context = context;
    }

    public async Task<Department> CreateDepartmentAsync(Department department)
    {
        _context.Departments.Add(department);
        await _context.SaveChangesAsync();
        return department;
    }

    public async Task<bool> TransferEmployeeAsync(int employeeId, int newDepartmentId)
    {
        var employee = await _context.Employees.FindAsync(employeeId);
        var department = await _context.Departments.FindAsync(newDepartmentId);
        
        if (employee == null || department == null)
            return false;

        employee.DepartmentId = newDepartmentId;
        await _context.SaveChangesAsync();
        return true;
    }
}
```

---

## Best Practices

### 1. การจัดการไฟล์
- แยก custom code เป็น partial classes
- ใช้ namespace ที่เหมาะสม
- backup ไฟล์ก่อน re-scaffold

### 2. การ Configuration
- ย้าย connection string ไป configuration
- ใช้ Dependency Injection
- เพิ่ม error handling

### 3. การ Maintenance
- ใช้ version control ติดตาม changes
- document การเปลี่ยนแปลง custom code
- test หลังจาก re-scaffold

### 4. Performance
- เพิ่ม necessary indexes
- ใช้ query filters สำหรับ common conditions
- optimize navigation properties

---

## สรุป

Database First Approach เหมาะสำหรับ:
- โปรเจคที่มีฐานข้อมูลอยู่แล้ว
- การทำงานร่วมกับ Database Team
- Legacy System Integration
- ระบบขนาดใหญ่ที่มี Database Schema ซับซ้อน

ข้อดี:
- รวดเร็วในการเริ่มต้น
- รับรองความถูกต้องของ Schema
- ทำงานร่วมกับ DBA ได้ดี

ข้อควรระวัง:
- Custom code อาจหายเมื่อ re-scaffold
- ต้องจัดการ Connection String
- ต้องเข้าใจโครงสร้างฐานข้อมูลดี

---

## การทำงานกับหลายฐานข้อมูล (Multiple Databases)

### สถานการณ์การใช้งานหลายฐานข้อมูล

1. **แยกตาม Business Domain** - HR DB, Finance DB, Inventory DB
2. **แยกตาม Performance** - Read DB, Write DB
3. **Multi-tenant Architecture** - แต่ละลูกค้าใช้ DB แยกกัน
4. **Legacy System Integration** - เชื่อมต่อกับระบบเก่าหลายตัว
5. **Microservices** - แต่ละ service มี database ของตัวเอง

---

### วิธีที่ 1: Multiple DbContext (แนะนำ)

#### 1.1 สร้าง DbContext แยกกันต่าง Database

```csharp
// HRDbContext.cs - สำหรับฐานข้อมูล HR
public class HRDbContext : DbContext
{
    public HRDbContext(DbContextOptions<HRDbContext> options) : base(options) { }
    
    public DbSet<Employee> Employees { get; set; }
    public DbSet<Department> Departments { get; set; }
    public DbSet<Payroll> Payrolls { get; set; }
}

// FinanceDbContext.cs - สำหรับฐานข้อมูล Finance
public class FinanceDbContext : DbContext
{
    public FinanceDbContext(DbContextOptions<FinanceDbContext> options) : base(options) { }
    
    public DbSet<Account> Accounts { get; set; }
    public DbSet<Transaction> Transactions { get; set; }
    public DbSet<Budget> Budgets { get; set; }
}

// InventoryDbContext.cs - สำหรับฐานข้อมูล Inventory
public class InventoryDbContext : DbContext
{
    public InventoryDbContext(DbContextOptions<InventoryDbContext> options) : base(options) { }
    
    public DbSet<Product> Products { get; set; }
    public DbSet<Stock> Stocks { get; set; }
    public DbSet<Supplier> Suppliers { get; set; }
}
```

#### 1.2 Configuration ใน Program.cs

```csharp
// appsettings.json
{
  "ConnectionStrings": {
    "HRConnection": "Server=(localdb)\\mssqllocaldb;Database=HRDB;Trusted_Connection=true;",
    "FinanceConnection": "Server=(localdb)\\mssqllocaldb;Database=FinanceDB;Trusted_Connection=true;",
    "InventoryConnection": "Server=(localdb)\\mssqllocaldb;Database=InventoryDB;Trusted_Connection=true;"
  }
}

// Program.cs
var builder = WebApplication.CreateBuilder(args);

// ลงทะเบียน DbContext หลายตัว
builder.Services.AddDbContext<HRDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("HRConnection")));
    
builder.Services.AddDbContext<FinanceDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("FinanceConnection")));
    
builder.Services.AddDbContext<InventoryDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("InventoryConnection")));

var app = builder.Build();
```

#### 1.3 การใช้งานใน Service

```csharp
public class ReportService
{
    private readonly HRDbContext _hrContext;
    private readonly FinanceDbContext _financeContext;
    private readonly InventoryDbContext _inventoryContext;

    public ReportService(
        HRDbContext hrContext,
        FinanceDbContext financeContext,
        InventoryDbContext inventoryContext)
    {
        _hrContext = hrContext;
        _financeContext = financeContext;
        _inventoryContext = inventoryContext;
    }

    public async Task<ComprehensiveReportDto> GenerateComprehensiveReportAsync()
    {
        // ข้อมูลพนักงาน
        var employees = await _hrContext.Employees
            .Where(e => e.IsActive)
            .ToListAsync();

        // ข้อมูลการเงิน
        var monthlyBudget = await _financeContext.Budgets
            .Where(b => b.Month == DateTime.Now.Month)
            .SumAsync(b => b.Amount);

        // ข้อมูลสินค้าคงคลัง
        var lowStockProducts = await _inventoryContext.Products
            .Join(_inventoryContext.Stocks, p => p.ProductId, s => s.ProductId, (p, s) => new { p, s })
            .Where(x => x.s.Quantity < x.p.MinimumStock)
            .Select(x => x.p)
            .ToListAsync();

        return new ComprehensiveReportDto
        {
            TotalEmployees = employees.Count,
            MonthlyBudget = monthlyBudget,
            LowStockProducts = lowStockProducts
        };
    }
}
```

---

### วิธีที่ 2: Database ร่วมกันผ่าน Schema

#### 2.1 ใช้ Schema แยกใน Database เดียว

```sql
-- สร้าง Schema ต่าง ๆ
CREATE SCHEMA HR;
CREATE SCHEMA Finance;
CREATE SCHEMA Inventory;

-- ย้ายตารางไป Schema ที่เหมาะสม
CREATE TABLE HR.Employees (...);
CREATE TABLE Finance.Accounts (...);
CREATE TABLE Inventory.Products (...);
```

#### 2.2 กำหนด Schema ใน DbContext

```csharp
public class CompanyDbContext : DbContext
{
    public CompanyDbContext(DbContextOptions<CompanyDbContext> options) : base(options) { }
    
    // HR Tables
    public DbSet<Employee> Employees { get; set; }
    public DbSet<Department> Departments { get; set; }
    
    // Finance Tables
    public DbSet<Account> Accounts { get; set; }
    public DbSet<Transaction> Transactions { get; set; }
    
    // Inventory Tables
    public DbSet<Product> Products { get; set; }
    public DbSet<Stock> Stocks { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // กำหนด Schema สำหรับแต่ละตาราง
        modelBuilder.Entity<Employee>().ToTable("Employees", "HR");
        modelBuilder.Entity<Department>().ToTable("Departments", "HR");
        
        modelBuilder.Entity<Account>().ToTable("Accounts", "Finance");
        modelBuilder.Entity<Transaction>().ToTable("Transactions", "Finance");
        
        modelBuilder.Entity<Product>().ToTable("Products", "Inventory");
        modelBuilder.Entity<Stock>().ToTable("Stocks", "Inventory");
    }
}
```

---

### วิธีที่ 3: Multi-Tenant Database

#### 3.1 แบบ Database per Tenant

```csharp
public class TenantService
{
    private readonly IConfiguration _configuration;
    
    public TenantService(IConfiguration configuration)
    {
        _configuration = configuration;
    }
    
    public DbContextOptions<CompanyDbContext> GetDbContextOptions(string tenantId)
    {
        var connectionString = _configuration.GetConnectionString($"Tenant_{tenantId}");
        
        var optionsBuilder = new DbContextOptionsBuilder<CompanyDbContext>();
        optionsBuilder.UseSqlServer(connectionString);
        
        return optionsBuilder.Options;
    }
}

// การใช้งาน
public class TenantAwareService
{
    private readonly TenantService _tenantService;
    
    public TenantAwareService(TenantService tenantService)
    {
        _tenantService = tenantService;
    }
    
    public async Task<List<Employee>> GetEmployeesForTenantAsync(string tenantId)
    {
        var options = _tenantService.GetDbContextOptions(tenantId);
        
        using var context = new CompanyDbContext(options);
        return await context.Employees.ToListAsync();
    }
}
```

#### 3.2 แบบ Shared Database with Tenant Filter

```csharp
public class MultiTenantDbContext : DbContext
{
    private readonly string _tenantId;
    
    public MultiTenantDbContext(DbContextOptions options, string tenantId) 
        : base(options)
    {
        _tenantId = tenantId;
    }
    
    public DbSet<Employee> Employees { get; set; }
    public DbSet<Department> Departments { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // เพิ่ม Global Query Filter สำหรับ Tenant
        modelBuilder.Entity<Employee>()
            .HasQueryFilter(e => e.TenantId == _tenantId);
            
        modelBuilder.Entity<Department>()
            .HasQueryFilter(d => d.TenantId == _tenantId);
    }

    public override async Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
    {
        // เพิ่ม TenantId ให้กับ entity ใหม่โดยอัตโนมัติ
        var entries = ChangeTracker.Entries<ITenantEntity>()
            .Where(e => e.State == EntityState.Added);
            
        foreach (var entry in entries)
        {
            entry.Entity.TenantId = _tenantId;
        }
        
        return await base.SaveChangesAsync(cancellationToken);
    }
}

// Interface สำหรับ Tenant Entity
public interface ITenantEntity
{
    string TenantId { get; set; }
}

// ตัวอย่าง Entity ที่ implement ITenantEntity
public class Employee : ITenantEntity
{
    public int EmployeeId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string TenantId { get; set; } // สำหรับ Multi-tenant
}
```

---

### วิธีที่ 4: Cross-Database Queries

#### 4.1 Manual Data Aggregation

```csharp
public class CrossDatabaseService
{
    private readonly HRDbContext _hrContext;
    private readonly FinanceDbContext _financeContext;
    
    public CrossDatabaseService(HRDbContext hrContext, FinanceDbContext financeContext)
    {
        _hrContext = hrContext;
        _financeContext = financeContext;
    }
    
    public async Task<EmployeeFinanceReportDto> GetEmployeeFinanceReportAsync(int employeeId)
    {
        // ดึงข้อมูลพนักงานจาก HR Database
        var employee = await _hrContext.Employees
            .FirstOrDefaultAsync(e => e.EmployeeId == employeeId);
            
        if (employee == null) return null;
        
        // ดึงข้อมูลการเงินจาก Finance Database (ใช้ Employee Number เป็น key)
        var payrollRecords = await _financeContext.PayrollRecords
            .Where(p => p.EmployeeNumber == employee.EmployeeNumber)
            .OrderByDescending(p => p.PayDate)
            .Take(12) // 12 เดือนล่าสุด
            .ToListAsync();
        
        return new EmployeeFinanceReportDto
        {
            Employee = employee,
            PayrollRecords = payrollRecords,
            TotalEarnings = payrollRecords.Sum(p => p.GrossPay),
            AverageMonthlyEarnings = payrollRecords.Average(p => p.GrossPay)
        };
    }
}
```

#### 4.2 ใช้ Raw SQL สำหรับ Linked Server

```csharp
public async Task<List<CrossDatabaseReportDto>> GetCrossDatabaseReportAsync()
{
    var sql = @"
        SELECT 
            e.FirstName,
            e.LastName,
            e.DepartmentId,
            f.TotalSalary
        FROM [HRDB].[dbo].[Employees] e
        INNER JOIN [FinanceDB].[dbo].[EmployeeSalaries] f 
            ON e.EmployeeNumber = f.EmployeeNumber
        WHERE e.IsActive = 1";
    
    using var connection = new SqlConnection(_connectionString);
    var result = await connection.QueryAsync<CrossDatabaseReportDto>(sql);
    
    return result.ToList();
}
```

---

### วิธีที่ 5: Transaction ข้าม Database

#### 5.1 ใช้ TransactionScope

```csharp
public class CrossDatabaseTransactionService
{
    private readonly HRDbContext _hrContext;
    private readonly FinanceDbContext _financeContext;
    
    public CrossDatabaseTransactionService(
        HRDbContext hrContext, 
        FinanceDbContext financeContext)
    {
        _hrContext = hrContext;
        _financeContext = financeContext;
    }
    
    public async Task<bool> TransferEmployeeWithFinanceAsync(
        int employeeId, 
        int newDepartmentId, 
        decimal salaryAdjustment)
    {
        using var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled);
        
        try
        {
            // อัพเดทข้อมูลใน HR Database
            var employee = await _hrContext.Employees.FindAsync(employeeId);
            if (employee == null) return false;
            
            employee.DepartmentId = newDepartmentId;
            await _hrContext.SaveChangesAsync();
            
            // อัพเดทข้อมูลใน Finance Database
            var salaryRecord = await _financeContext.EmployeeSalaries
                .FirstOrDefaultAsync(s => s.EmployeeNumber == employee.EmployeeNumber);
                
            if (salaryRecord != null)
            {
                salaryRecord.BaseSalary += salaryAdjustment;
                salaryRecord.LastModified = DateTime.Now;
                await _financeContext.SaveChangesAsync();
            }
            
            // Commit ทุก transaction
            scope.Complete();
            return true;
        }
        catch (Exception)
        {
            // Transaction จะ rollback อัตโนมัติ
            return false;
        }
    }
}
```

#### 5.2 Manual Transaction Management

```csharp
public async Task<bool> ManualTransactionAsync()
{
    using var hrTransaction = await _hrContext.Database.BeginTransactionAsync();
    using var financeTransaction = await _financeContext.Database.BeginTransactionAsync();
    
    try
    {
        // ทำงานใน HR Database
        var employee = new Employee { FirstName = "John", LastName = "Doe" };
        _hrContext.Employees.Add(employee);
        await _hrContext.SaveChangesAsync();
        
        // ทำงานใน Finance Database
        var payroll = new PayrollRecord 
        { 
            EmployeeNumber = employee.EmployeeNumber,
            BaseSalary = 50000 
        };
        _financeContext.PayrollRecords.Add(payroll);
        await _financeContext.SaveChangesAsync();
        
        // Commit ทั้งคู่
        await hrTransaction.CommitAsync();
        await financeTransaction.CommitAsync();
        
        return true;
    }
    catch (Exception)
    {
        await hrTransaction.RollbackAsync();
        await financeTransaction.RollbackAsync();
        return false;
    }
}
```

---

### การจัดการ Connection Pool

#### การตั้งค่า Connection Pool แยกต่าง Database

```csharp
// Program.cs
builder.Services.AddDbContext<HRDbContext>(options =>
{
    options.UseSqlServer(builder.Configuration.GetConnectionString("HRConnection"), 
        sqlOptions =>
        {
            sqlOptions.CommandTimeout(30);
        });
}, ServiceLifetime.Scoped);

builder.Services.AddDbContext<FinanceDbContext>(options =>
{
    options.UseSqlServer(builder.Configuration.GetConnectionString("FinanceConnection"),
        sqlOptions =>
        {
            sqlOptions.CommandTimeout(60); // Finance queries อาจใช้เวลานานกว่า
        });
}, ServiceLifetime.Scoped);

// การปรับแต่ง Connection Pool
builder.Services.AddDbContextPool<InventoryDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("InventoryConnection")),
    poolSize: 128); // เพิ่ม pool size สำหรับ high-traffic database
```

---

### Best Practices สำหรับหลาย Database

#### 1. การออกแบบ Architecture

```csharp
// ใช้ Interface เพื่อ Abstract การทำงาน
public interface IEmployeeRepository
{
    Task<Employee> GetByIdAsync(int id);
    Task<List<Employee>> GetByDepartmentAsync(int departmentId);
}

public interface IFinanceRepository
{
    Task<decimal> GetTotalSalaryByDepartmentAsync(int departmentId);
    Task<List<PayrollRecord>> GetPayrollHistoryAsync(string employeeNumber);
}

// Implementation แยกต่าง Database
public class HREmployeeRepository : IEmployeeRepository
{
    private readonly HRDbContext _context;
    
    public HREmployeeRepository(HRDbContext context)
    {
        _context = context;
    }
    
    public async Task<Employee> GetByIdAsync(int id)
    {
        return await _context.Employees.FindAsync(id);
    }
    
    public async Task<List<Employee>> GetByDepartmentAsync(int departmentId)
    {
        return await _context.Employees
            .Where(e => e.DepartmentId == departmentId)
            .ToListAsync();
    }
}
```

#### 2. Error Handling และ Monitoring

```csharp
public class DatabaseHealthService
{
    private readonly HRDbContext _hrContext;
    private readonly FinanceDbContext _financeContext;
    private readonly ILogger<DatabaseHealthService> _logger;
    
    public async Task<DatabaseHealthReport> CheckHealthAsync()
    {
        var report = new DatabaseHealthReport();
        
        try
        {
            // ตรวจสอบ HR Database
            var hrCanConnect = await _hrContext.Database.CanConnectAsync();
            report.HRDatabaseStatus = hrCanConnect ? "Healthy" : "Unhealthy";
            
            // ตรวจสอบ Finance Database
            var financeCanConnect = await _financeContext.Database.CanConnectAsync();
            report.FinanceDatabaseStatus = financeCanConnect ? "Healthy" : "Unhealthy";
            
            _logger.LogInformation("Database health check completed");
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error during database health check");
            report.HasErrors = true;
            report.ErrorMessage = ex.Message;
        }
        
        return report;
    }
}
```

#### 3. Configuration Management

```csharp
// DatabaseConfiguration.cs
public class DatabaseConfiguration
{
    public class DatabaseConnectionConfig
    {
        public string ConnectionString { get; set; }
        public int CommandTimeout { get; set; } = 30;
        public int MaxRetryCount { get; set; } = 3;
        public bool EnableSensitiveDataLogging { get; set; } = false;
    }
    
    public DatabaseConnectionConfig HR { get; set; }
    public DatabaseConnectionConfig Finance { get; set; }
    public DatabaseConnectionConfig Inventory { get; set; }
}

// Program.cs
var dbConfig = builder.Configuration.GetSection("DatabaseConfiguration")
    .Get<DatabaseConfiguration>();

builder.Services.AddDbContext<HRDbContext>(options =>
{
    options.UseSqlServer(dbConfig.HR.ConnectionString, sqlOptions =>
    {
        sqlOptions.CommandTimeout(dbConfig.HR.CommandTimeout);
        sqlOptions.EnableRetryOnFailure(dbConfig.HR.MaxRetryCount);
    });
    
    if (dbConfig.HR.EnableSensitiveDataLogging)
    {
        options.EnableSensitiveDataLogging();
    }
});
```

---

### สรุปการใช้งานหลาย Database

**เลือกใช้ Multiple DbContext เมื่อ:**
- Database แยกกันตาม business domain
- ต้องการ isolation ที่ชัดเจน
- มี database อยู่คนละ server

**เลือกใช้ Schema Separation เมื่อ:**
- อยู่ใน database เดียวกัน
- ต้องการ cross-schema queries
- จัดการง่ายกว่า multiple databases

**เลือกใช้ Multi-tenant เมื่อ:**
- SaaS application
- ต้องการ data isolation ต่าง customer
- Scale ตาม tenant

**ข้อควรระวัง:**
- Transaction ข้าม database ซับซ้อนกว่า
- Connection pool ต้องจัดการแยกกัน
- Monitoring และ logging ต้องครอบคลุมทุก database
- Error handling ต้องพิจารณา partial failures

---

## การจัดการการเปลี่ยนแปลง Database Schema

### สถานการณ์ที่เจอบ่อย

1. **เปลี่ยนชื่อ Column** - FirstName -> First_Name
2. **เปลี่ยน Data Type** - varchar(50) -> nvarchar(100)
3. **เปลี่ยนขนาดข้อมูล** - decimal(10,2) -> decimal(15,4)
4. **เพิ่ม/ลบ Column** ใหม่
5. **เปลี่ยน Nullable/Not Null**
6. **เพิ่ม/เปลี่ยน Constraints**

---

### กรณีที่ 1: Database ที่ไม่ได้สร้างเอง (External Database)

#### 1.1 การเปลี่ยนชื่อ Column ใน External Database

**ปัญหา:** Database มี column ชื่อ `emp_first_name` แต่ต้องการใช้ชื่อ `FirstName` ใน Model

```csharp
// วิธีที่ 1: ใช้ Column Attribute
public class Employee
{
    public int EmployeeId { get; set; }
    
    [Column("emp_first_name")]  // แมปกับชื่อ column จริงใน DB
    public string FirstName { get; set; }
    
    [Column("emp_last_name")]
    public string LastName { get; set; }
    
    [Column("emp_email")]
    public string Email { get; set; }
}

// วิธีที่ 2: ใช้ Fluent API
public class CompanyDbContext : DbContext
{
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Employee>(entity =>
        {
            entity.Property(e => e.FirstName)
                .HasColumnName("emp_first_name");
                
            entity.Property(e => e.LastName)
                .HasColumnName("emp_last_name");
                
            entity.Property(e => e.Email)
                .HasColumnName("emp_email");
        });
    }
}
```

#### 1.2 การจัดการ Data Type ที่ไม่ตรงกัน

```csharp
public class Employee
{
    public int EmployeeId { get; set; }
    
    // กรณี: DB เป็น varchar(10) แต่ต้องการใช้เป็น DateTime
    [Column("hire_date_str")]  // DB: varchar(10) format "2024-01-30"
    public string HireDateString { get; set; }
    
    // สร้าง Property แยกสำหรับใช้งานจริง
    [NotMapped]
    public DateTime HireDate
    {
        get => DateTime.ParseExact(HireDateString, "yyyy-MM-dd", null);
        set => HireDateString = value.ToString("yyyy-MM-dd");
    }
    
    // กรณี: DB เป็น bit แต่ต้องการใช้เป็น enum
    [Column("employee_status")]  // DB: bit (0,1)
    public bool IsActiveFlag { get; set; }
    
    [NotMapped]
    public EmployeeStatus Status
    {
        get => IsActiveFlag ? EmployeeStatus.Active : EmployeeStatus.Inactive;
        set => IsActiveFlag = value == EmployeeStatus.Active;
    }
}

public enum EmployeeStatus
{
    Inactive = 0,
    Active = 1
}
```

#### 1.3 ใช้ Value Converters สำหรับ Complex Mapping

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Converter สำหรับ JSON ใน varchar column
    modelBuilder.Entity<Employee>()
        .Property(e => e.Skills)  // List<string> ใน C#
        .HasColumnName("skills_json")  // varchar ใน DB
        .HasConversion(
            skills => JsonSerializer.Serialize(skills),  // C# -> DB
            json => JsonSerializer.Deserialize<List<string>>(json)  // DB -> C#
        );
    
    // Converter สำหรับ Enum เป็น string
    modelBuilder.Entity<Employee>()
        .Property(e => e.Level)
        .HasColumnName("emp_level")  // varchar(20) ใน DB
        .HasConversion<string>();  // enum -> string
        
    // Converter สำหรับ DateTime กับ varchar
    var dateConverter = new ValueConverter<DateTime, string>(
        dateTime => dateTime.ToString("yyyyMMdd"),
        dateString => DateTime.ParseExact(dateString, "yyyyMMdd", null)
    );
    
    modelBuilder.Entity<Employee>()
        .Property(e => e.HireDate)
        .HasColumnName("hire_date_str")
        .HasConversion(dateConverter);
}
```

---

### กรณีที่ 2: Database ที่สร้างเอง (EF Migrations)

#### 2.1 การเปลี่ยนชื่อ Column ด้วย Migration

```csharp
// เปลี่ยนชื่อ column ใน Model
public class Employee
{
    public int EmployeeId { get; set; }
    public string FullName { get; set; }  // เดิมชื่อ FirstName
    public string LastName { get; set; }
}

// สร้าง Migration
// dotnet ef migrations add RenameFirstNameToFullName

// Migration ที่สร้างขึ้นจะเป็น:
public partial class RenameFirstNameToFullName : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.RenameColumn(
            name: "FirstName",
            table: "Employees",
            newName: "FullName");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.RenameColumn(
            name: "FullName",
            table: "Employees",
            newName: "FirstName");
    }
}
```

#### 2.2 การเปลี่ยน Data Type

```csharp
// เปลี่ยน Salary จาก decimal(10,2) เป็น decimal(15,4)
public class Employee
{
    public int EmployeeId { get; set; }
    public string FirstName { get; set; }
    
    [Column(TypeName = "decimal(15,4)")]  // เปลี่ยนจาก decimal(10,2)
    public decimal Salary { get; set; }
}

// สร้าง Migration
// dotnet ef migrations add ChangeSalaryPrecision

// Migration ที่สร้าง:
public partial class ChangeSalaryPrecision : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.AlterColumn<decimal>(
            name: "Salary",
            table: "Employees",
            type: "decimal(15,4)",
            nullable: false,
            oldClrType: typeof(decimal),
            oldType: "decimal(10,2)");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.AlterColumn<decimal>(
            name: "Salary",
            table: "Employees",
            type: "decimal(10,2)",
            nullable: false,
            oldClrType: typeof(decimal),
            oldType: "decimal(15,4)");
    }
}
```

#### 2.3 การเปลี่ยน String Length

```csharp
// เปลี่ยนขนาดของ Email จาก 100 เป็น 200
public class Employee
{
    public int EmployeeId { get; set; }
    
    [MaxLength(200)]  // เปลี่ยนจาก [MaxLength(100)]
    public string Email { get; set; }
}

// หรือใช้ Fluent API
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Employee>()
        .Property(e => e.Email)
        .HasMaxLength(200);  // เปลี่ยนจาก HasMaxLength(100)
}
```

---

### การจัดการข้อมูลระหว่าง Migration

#### 3.1 การเปลี่ยน Column พร้อมรักษาข้อมูล

```csharp
// กรณี: แยก FullName เป็น FirstName และ LastName
public partial class SplitFullNameMigration : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // เพิ่ม column ใหม่
        migrationBuilder.AddColumn<string>(
            name: "FirstName",
            table: "Employees",
            type: "nvarchar(50)",
            maxLength: 50,
            nullable: true);
            
        migrationBuilder.AddColumn<string>(
            name: "LastName",
            table: "Employees",
            type: "nvarchar(50)",
            maxLength: 50,
            nullable: true);
        
        // แยกข้อมูลจาก FullName
        migrationBuilder.Sql(@"
            UPDATE Employees 
            SET FirstName = CASE 
                    WHEN CHARINDEX(' ', FullName) > 0 
                    THEN LEFT(FullName, CHARINDEX(' ', FullName) - 1)
                    ELSE FullName 
                END,
                LastName = CASE 
                    WHEN CHARINDEX(' ', FullName) > 0 
                    THEN SUBSTRING(FullName, CHARINDEX(' ', FullName) + 1, LEN(FullName))
                    ELSE ''
                END
            WHERE FullName IS NOT NULL");
        
        // ลบ column เก่า
        migrationBuilder.DropColumn(
            name: "FullName",
            table: "Employees");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        // สร้าง FullName กลับ
        migrationBuilder.AddColumn<string>(
            name: "FullName",
            table: "Employees",
            type: "nvarchar(100)",
            maxLength: 100,
            nullable: true);
        
        // รวมข้อมูลกลับ
        migrationBuilder.Sql(@"
            UPDATE Employees 
            SET FullName = CONCAT(FirstName, ' ', LastName)
            WHERE FirstName IS NOT NULL OR LastName IS NOT NULL");
        
        // ลบ column ใหม่
        migrationBuilder.DropColumn(name: "FirstName", table: "Employees");
        migrationBuilder.DropColumn(name: "LastName", table: "Employees");
    }
}
```

#### 3.2 การเปลี่ยน Data Type ที่ซับซ้อน

```csharp
// กรณี: เปลี่ยน BirthDate จาก string เป็น DateTime
public partial class ConvertBirthDateMigration : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // เพิ่ม column ใหม่ชั่วคราว
        migrationBuilder.AddColumn<DateTime>(
            name: "BirthDate_New",
            table: "Employees",
            type: "datetime2",
            nullable: true);
        
        // แปลงข้อมูลจาก string เป็น DateTime
        migrationBuilder.Sql(@"
            UPDATE Employees 
            SET BirthDate_New = CAST(BirthDate_String AS datetime2)
            WHERE BirthDate_String IS NOT NULL 
              AND ISDATE(BirthDate_String) = 1");
        
        // ลบ column เก่า
        migrationBuilder.DropColumn(
            name: "BirthDate_String",
            table: "Employees");
        
        // เปลี่ยนชื่อ column ใหม่
        migrationBuilder.RenameColumn(
            name: "BirthDate_New",
            table: "Employees",
            newName: "BirthDate");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.AddColumn<string>(
            name: "BirthDate_String",
            table: "Employees",
            type: "nvarchar(20)",
            maxLength: 20,
            nullable: true);
        
        migrationBuilder.Sql(@"
            UPDATE Employees 
            SET BirthDate_String = CONVERT(varchar, BirthDate, 23)
            WHERE BirthDate IS NOT NULL");
        
        migrationBuilder.DropColumn(name: "BirthDate", table: "Employees");
    }
}
```

---

### การจัดการ External Database Changes

#### 4.1 เมื่อ External Database มีการเปลี่ยน Schema

```bash
# Step 1: Backup current models
cp -r Models/ Models_Backup/

# Step 2: Re-scaffold เพื่อดู changes
dotnet ef dbcontext scaffold "connection-string" Microsoft.EntityFrameworkCore.SqlServer --output-dir Models_New --context CompanyDbContext_New --force

# Step 3: เปรียบเทียบความแตกต่าง
# ใช้ diff tool เปรียบเทียบ Models/ กับ Models_New/
```

#### 4.2 การจัดการ Schema Changes แบบ Manual

```csharp
// วิธีที่ 1: สร้าง Mapping Layer
public class EmployeeMapper
{
    public static Employee MapFromDatabase(EmployeeDto dbEmployee)
    {
        return new Employee
        {
            EmployeeId = dbEmployee.EmpId,  // column เปลี่ยนชื่อ
            FirstName = dbEmployee.FName,   // column เปลี่ยนชื่อ
            LastName = dbEmployee.LName,    // column เปลี่ยนชื่อ
            Email = dbEmployee.EmailAddress,
            // จัดการ type conversion
            HireDate = DateTime.ParseExact(dbEmployee.HireDateString, "yyyyMMdd", null),
            Salary = decimal.Parse(dbEmployee.SalaryString)
        };
    }
}

// วิธีที่ 2: ใช้ Database Views
// สร้าง View ใน Database เพื่อ abstract การเปลี่ยนแปลง
/* SQL:
CREATE VIEW vw_Employees AS
SELECT 
    emp_id as EmployeeId,
    f_name as FirstName,
    l_name as LastName,
    email_addr as Email,
    CAST(hire_date_str AS date) as HireDate,
    CAST(salary_str AS decimal(10,2)) as Salary
FROM employees_new_structure;
*/

// ใน EF Core map กับ View แทน Table
modelBuilder.Entity<Employee>()
    .ToView("vw_Employees")
    .HasNoKey(); // ถ้า View ไม่มี primary key
```

#### 4.3 การใช้ Raw SQL เมื่อ Schema ซับซ้อน

```csharp
public class EmployeeService
{
    private readonly CompanyDbContext _context;
    
    public async Task<List<Employee>> GetEmployeesAsync()
    {
        // ใช้ Raw SQL เพื่อจัดการ schema changes
        var sql = @"
            SELECT 
                emp_id as EmployeeId,
                CONCAT(f_name, ' ', l_name) as FullName,
                email_new_col as Email,
                CAST(hire_date_varchar AS datetime) as HireDate,
                CAST(salary_decimal_15_4 AS decimal(10,2)) as DisplaySalary
            FROM employees_table_v2 
            WHERE is_active_flag = 1";
    
        var employees = await _context.Database
            .SqlQueryRaw<EmployeeQueryResult>(sql)
            .ToListAsync();
        
        return employees.Select(MapToEmployee).ToList();
    }
    
    private Employee MapToEmployee(EmployeeQueryResult result)
    {
        return new Employee
        {
            EmployeeId = result.EmployeeId,
            FullName = result.FullName,
            Email = result.Email,
            HireDate = result.HireDate,
            Salary = result.DisplaySalary
        };
    }
}
```

---

### Strategies สำหรับ Production Environment

#### 5.1 Blue-Green Deployment สำหรับ Schema Changes

```csharp
// สร้าง Compatibility Layer ระหว่าง Old และ New Schema
public class EmployeeCompatibilityService
{
    private readonly IConfiguration _config;
    private readonly bool _useNewSchema;
    
    public EmployeeCompatibilityService(IConfiguration config)
    {
        _config = config;
        _useNewSchema = _config.GetValue<bool>("UseNewSchema");
    }
    
    public async Task<Employee> GetEmployeeAsync(int id)
    {
        if (_useNewSchema)
        {
            return await GetEmployeeFromNewSchemaAsync(id);
        }
        else
        {
            return await GetEmployeeFromOldSchemaAsync(id);
        }
    }
    
    private async Task<Employee> GetEmployeeFromOldSchemaAsync(int id)
    {
        // Logic สำหรับ old schema
        var sql = "SELECT emp_id, first_name, last_name FROM employees WHERE emp_id = @id";
        // ... implementation
    }
    
    private async Task<Employee> GetEmployeeFromNewSchemaAsync(int id)
    {
        // Logic สำหรับ new schema
        var sql = "SELECT employee_id, fname, lname FROM emp_table WHERE employee_id = @id";
        // ... implementation
    }
}
```

#### 5.2 การ Validate Schema Changes

```csharp
public class SchemaValidator
{
    private readonly CompanyDbContext _context;
    
    public async Task<SchemaValidationResult> ValidateSchemaAsync()
    {
        var result = new SchemaValidationResult();
        
        try
        {
            // ตรวจสอบ table ที่จำเป็นมีอยู่หรือไม่
            var tableExists = await _context.Database
                .SqlQueryRaw<int>(@"
                    SELECT COUNT(*) 
                    FROM INFORMATION_SCHEMA.TABLES 
                    WHERE TABLE_NAME = 'Employees'")
                .FirstOrDefaultAsync();
                
            result.TablesExist = tableExists > 0;
            
            // ตรวจสอบ columns ที่จำเป็น
            var columnValidation = await _context.Database
                .SqlQueryRaw<ColumnInfo>(@"
                    SELECT COLUMN_NAME, DATA_TYPE, CHARACTER_MAXIMUM_LENGTH
                    FROM INFORMATION_SCHEMA.COLUMNS 
                    WHERE TABLE_NAME = 'Employees'")
                .ToListAsync();
            
            result.RequiredColumns = ValidateRequiredColumns(columnValidation);
            result.IsValid = result.TablesExist && result.RequiredColumns.All(c => c.Exists);
        }
        catch (Exception ex)
        {
            result.IsValid = false;
            result.ErrorMessage = ex.Message;
        }
        
        return result;
    }
}
```

---

### Best Practices

#### 1. การวางแผน Schema Changes

```csharp
// ใช้ Interface เพื่อ abstract data access
public interface IEmployeeRepository
{
    Task<Employee> GetByIdAsync(int id);
    Task<List<Employee>> GetAllAsync();
    Task SaveAsync(Employee employee);
}

// Implementation สำหรับ Current Schema
public class EmployeeRepository : IEmployeeRepository
{
    // Current implementation
}

// Implementation สำหรับ New Schema (เตรียมไว้ล่วงหน้า)
public class EmployeeRepositoryV2 : IEmployeeRepository
{
    // New implementation for new schema
}

// ใช้ Factory Pattern เพื่อเลือก implementation
public class EmployeeRepositoryFactory
{
    public static IEmployeeRepository Create(string version)
    {
        return version switch
        {
            "v1" => new EmployeeRepository(),
            "v2" => new EmployeeRepositoryV2(),
            _ => throw new ArgumentException("Invalid version")
        };
    }
}
```

#### 2. การ Test Schema Changes

```csharp
[Test]
public async Task Should_Handle_Schema_Changes_Gracefully()
{
    // Arrange
    var oldSchemaContext = CreateOldSchemaContext();
    var newSchemaContext = CreateNewSchemaContext();
    
    // Act & Assert - ข้อมูลต้องเหมือนกันจากทั้ง 2 schema
    var oldData = await oldSchemaContext.Employees.ToListAsync();
    var newData = await newSchemaContext.Employees.ToListAsync();
    
    Assert.Equal(oldData.Count, newData.Count);
    // เปรียบเทียบข้อมูลสำคัญ
}
```

#### 3. การ Monitor Schema Health

```csharp
public class SchemaHealthMonitor
{
    public async Task<HealthReport> CheckSchemaHealthAsync()
    {
        var issues = new List<string>();
        
        // ตรวจสอบ missing tables
        // ตรวจสอบ column data types
        // ตรวจสอบ performance impact
        
        return new HealthReport
        {
            IsHealthy = !issues.Any(),
            Issues = issues,
            LastChecked = DateTime.Now
        };
    }
}
```

---

### สรุป Schema Change Management

**สำหรับ Database ที่สร้างเอง:**
- ใช้ EF Migrations อย่างระมัดระวัง
- เขียน Custom Migration สำหรับ complex changes
- Test ใน staging environment ก่อนเสมอ
- Backup ข้อมูลก่อน apply migration

**สำหรับ External Database:**
- ใช้ Column attributes หรือ Fluent API สำหรับ mapping
- พิจารณาใช้ Database Views สำหรับ abstraction
- สร้าง Compatibility Layer สำหรับ transition period
- Monitor schema changes และมี rollback plan

**Best Practices:**
- วางแผน schema changes ล่วงหน้า
- ใช้ Versioning strategy ที่ชัดเจน
- Automated testing สำหรับ schema compatibility
- Documentation สำหรับทุก schema change

---

**หัวข้อถัดไป:** [14 - EF Core CLI Commands](14-cli-commands.md)