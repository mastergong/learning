# บทที่ 7: Migrations และ Database Updates

## Migrations คืออะไร?

Migrations เป็นระบบที่ใช้จัดการการเปลี่ยนแปลง Schema ของฐานข้อมูลอย่างเป็นระบบ โดยจะสร้างไฟล์ที่บันทึกการเปลี่ยนแปลงทั้งการเพิ่ม ลบ แก้ไข ตาราง คอลัมน์ หรือ Index

### ข้อดีของ Migrations

- **Version Control**: ควบคุมเวอร์ชันของฐานข้อมูล
- **Team Collaboration**: ทีมสามารถ sync การเปลี่ยนแปลงได้
- **Rollback**: ย้อนกลับการเปลี่ยนแปลงได้
- **Deployment**: ติดตั้งใน Environment ต่างๆ ได้อย่างสะดวก

## การสร้างและใช้งาน Migrations

### 1. การสร้าง Migration แรก

```bash
# สร้าง Migration แรก
dotnet ef migrations add InitialCreate

# ระบุ Output Directory
dotnet ef migrations add InitialCreate --output-dir Data/Migrations

# ระบุ DbContext (ถ้ามีหลายตัว)
dotnet ef migrations add InitialCreate --context ApplicationDbContext

# ดู SQL Script ที่จะถูกสร้าง (ไม่รัน)
dotnet ef migrations script
```

### 2. การอัปเดตฐานข้อมูล

```bash
# อัปเดตฐานข้อมูลด้วย Migration ล่าสุด
dotnet ef database update

# อัปเดตไปยัง Migration ที่ระบุ
dotnet ef database update AddStudentProfile

# ย้อนกลับไปยัง Migration ก่อนหน้า
dotnet ef database update InitialCreate

# ลบ Migration ทั้งหมด (ย้อนกลับไปยังฐานข้อมูลเปล่า)
dotnet ef database update 0
```

### 3. การจัดการ Migrations

```bash
# ดูรายการ Migrations
dotnet ef migrations list

# ลบ Migration ล่าสุดที่ยังไม่ได้ apply
dotnet ef migrations remove

# สร้าง SQL Script จาก Migration
dotnet ef migrations script > update.sql

# สร้าง SQL Script จาก Migration ที่ระบุ
dotnet ef migrations script InitialCreate AddStudentProfile

# ดู SQL Script สำหรับ Migration ล่าสุด
dotnet ef migrations script --idempotent
```

## โครงสร้างไฟล์ Migration

### ตัวอย่างไฟล์ Migration

```csharp
// 20240129120000_InitialCreate.cs
using System;
using Microsoft.EntityFrameworkCore.Migrations;

#nullable disable

namespace EFCoreDemo.Data.Migrations
{
    /// <inheritdoc />
    public partial class InitialCreate : Migration
    {
        /// <inheritdoc />
        protected override void Up(MigrationBuilder migrationBuilder)
        {
            // การเปลี่ยนแปลงไปข้างหน้า (Forward)
            migrationBuilder.CreateTable(
                name: "Departments",
                columns: table => new
                {
                    Id = table.Column<int>(type: "int", nullable: false)
                        .Annotation("SqlServer:Identity", "1, 1"),
                    Name = table.Column<string>(type: "nvarchar(100)", maxLength: 100, nullable: false),
                    Code = table.Column<string>(type: "nvarchar(20)", maxLength: 20, nullable: false),
                    Description = table.Column<string>(type: "nvarchar(500)", maxLength: 500, nullable: true),
                    EstablishedDate = table.Column<DateTime>(type: "datetime2", nullable: true),
                    Building = table.Column<string>(type: "nvarchar(200)", maxLength: 200, nullable: true),
                    Floor = table.Column<string>(type: "nvarchar(50)", maxLength: 50, nullable: true),
                    PhoneNumber = table.Column<string>(type: "nvarchar(15)", maxLength: 15, nullable: true),
                    Email = table.Column<string>(type: "nvarchar(100)", maxLength: 100, nullable: true),
                    Website = table.Column<string>(type: "nvarchar(200)", maxLength: 200, nullable: true),
                    DeanId = table.Column<int>(type: "int", nullable: true)
                },
                constraints: table =>
                {
                    table.PrimaryKey("PK_Departments", x => x.Id);
                });

            migrationBuilder.CreateTable(
                name: "Students",
                columns: table => new
                {
                    Id = table.Column<int>(type: "int", nullable: false)
                        .Annotation("SqlServer:Identity", "1, 1"),
                    StudentNumber = table.Column<string>(type: "nvarchar(20)", maxLength: 20, nullable: false),
                    FirstName = table.Column<string>(type: "nvarchar(50)", maxLength: 50, nullable: false),
                    LastName = table.Column<string>(type: "nvarchar(50)", maxLength: 50, nullable: false),
                    NickName = table.Column<string>(type: "nvarchar(50)", maxLength: 50, nullable: true),
                    Email = table.Column<string>(type: "nvarchar(100)", maxLength: 100, nullable: false),
                    PhoneNumber = table.Column<string>(type: "nvarchar(15)", maxLength: 15, nullable: true),
                    DateOfBirth = table.Column<DateTime>(type: "date", nullable: false),
                    Gender = table.Column<int>(type: "int", nullable: false),
                    Address = table.Column<string>(type: "nvarchar(500)", maxLength: 500, nullable: true),
                    DepartmentId = table.Column<int>(type: "int", nullable: true),
                    GPA = table.Column<decimal>(type: "decimal(3,2)", nullable: true),
                    EnrollmentDate = table.Column<DateTime>(type: "datetime2", nullable: false),
                    Status = table.Column<int>(type: "int", nullable: false)
                },
                constraints: table =>
                {
                    table.PrimaryKey("PK_Students", x => x.Id);
                    table.ForeignKey(
                        name: "FK_Students_Departments_DepartmentId",
                        column: x => x.DepartmentId,
                        principalTable: "Departments",
                        principalColumn: "Id",
                        onDelete: ReferentialAction.SetNull);
                });

            // สร้าง Index
            migrationBuilder.CreateIndex(
                name: "IX_Students_DepartmentId",
                table: "Students",
                column: "DepartmentId");

            migrationBuilder.CreateIndex(
                name: "IX_Students_Email",
                table: "Students",
                column: "Email",
                unique: true);

            migrationBuilder.CreateIndex(
                name: "IX_Students_StudentNumber",
                table: "Students",
                column: "StudentNumber",
                unique: true);
        }

        /// <inheritdoc />
        protected override void Down(MigrationBuilder migrationBuilder)
        {
            // การย้อนกลับ (Rollback)
            migrationBuilder.DropTable(
                name: "Students");

            migrationBuilder.DropTable(
                name: "Departments");
        }
    }
}
```

### Model Snapshot

```csharp
// ApplicationDbContextModelSnapshot.cs
[DbContext(typeof(ApplicationDbContext))]
partial class ApplicationDbContextModelSnapshot : ModelSnapshot
{
    protected override void BuildModel(ModelBuilder modelBuilder)
    {
        // สร้าง Model ปัจจุบันทั้งหมด
        modelBuilder
            .HasAnnotation("ProductVersion", "8.0.0")
            .HasAnnotation("Relational:MaxIdentifierLength", 128);

        SqlServerModelBuilderExtensions.UseIdentityColumns(modelBuilder);

        modelBuilder.Entity("EFCoreDemo.Models.Department", b =>
        {
            b.Property<int>("Id")
                .ValueGeneratedOnAdd()
                .HasColumnType("int");

            SqlServerPropertyBuilderExtensions.UseIdentityColumn(b.Property<int>("Id"));

            b.Property<string>("Code")
                .IsRequired()
                .HasMaxLength(20)
                .HasColumnType("nvarchar(20)");

            // ... properties อื่นๆ

            b.HasKey("Id");
            b.ToTable("Departments", (string)null);
        });

        // ... entities อื่นๆ
    }
}
```

## การสร้าง Migration ขั้นสูง

### 1. การเพิ่ม Column

```csharp
// เพิ่ม Column ใหม่ในโมเดล
public class Student
{
    // ... properties เดิม
    
    [StringLength(100)]
    public string? MiddleName { get; set; }  // เพิ่ม property ใหม่
    
    public bool IsActive { get; set; } = true;
}
```

```bash
# สร้าง Migration
dotnet ef migrations add AddMiddleNameAndIsActiveToStudent
```

```csharp
// Migration ที่สร้างขึ้น
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.AddColumn<bool>(
        name: "IsActive",
        table: "Students",
        type: "bit",
        nullable: false,
        defaultValue: true);

    migrationBuilder.AddColumn<string>(
        name: "MiddleName",
        table: "Students",
        type: "nvarchar(100)",
        maxLength: 100,
        nullable: true);
}

protected override void Down(MigrationBuilder migrationBuilder)
{
    migrationBuilder.DropColumn(
        name: "IsActive",
        table: "Students");

    migrationBuilder.DropColumn(
        name: "MiddleName",
        table: "Students");
}
```

### 2. การแก้ไข Column

```csharp
// แก้ไข property
public class Student
{
    [StringLength(200)] // เพิ่มจาก 100 เป็น 200
    public string FirstName { get; set; }
    
    [Required] // เปลี่ยนจาก nullable เป็น required
    public string? MiddleName { get; set; }
}
```

```bash
dotnet ef migrations add UpdateStudentNameFields
```

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.AlterColumn<string>(
        name: "MiddleName",
        table: "Students",
        type: "nvarchar(100)",
        maxLength: 100,
        nullable: false,
        defaultValue: "",
        oldClrType: typeof(string),
        oldType: "nvarchar(100)",
        oldMaxLength: 100,
        oldNullable: true);

    migrationBuilder.AlterColumn<string>(
        name: "FirstName",
        table: "Students",
        type: "nvarchar(200)",
        maxLength: 200,
        nullable: false,
        oldClrType: typeof(string),
        oldType: "nvarchar(100)",
        oldMaxLength: 100);
}
```

### 3. การเพิ่มตารางใหม่

```csharp
// เพิ่ม Entity ใหม่
public class StudentProfile
{
    [Key]
    [ForeignKey("Student")]
    public int StudentId { get; set; }
    
    public string? ParentName { get; set; }
    public string? ParentPhone { get; set; }
    public string? MedicalCondition { get; set; }
    
    public virtual Student Student { get; set; }
}

// อัปเดต DbContext
public DbSet<StudentProfile> StudentProfiles { get; set; }
```

```bash
dotnet ef migrations add AddStudentProfile
```

### 4. การลบ Column

```csharp
// ลบ property จากโมเดล
public class Student
{
    // ลบ NickName property
    // public string? NickName { get; set; } // ลบบรรทัดนี้
}
```

```bash
dotnet ef migrations add RemoveNickNameFromStudent
```

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.DropColumn(
        name: "NickName",
        table: "Students");
}

protected override void Down(MigrationBuilder migrationBuilder)
{
    migrationBuilder.AddColumn<string>(
        name: "NickName",
        table: "Students",
        type: "nvarchar(50)",
        maxLength: 50,
        nullable: true);
}
```

## การปรับแต่ง Migration ด้วยตนเอง

### 1. การเพิ่ม Data Seeding

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    // สร้างตารางก่อน
    migrationBuilder.CreateTable(
        name: "Departments",
        // ... column definitions
    );

    // เพิ่มข้อมูลเริ่มต้น
    migrationBuilder.InsertData(
        table: "Departments",
        columns: new[] { "Name", "Code", "Description", "EstablishedDate" },
        values: new object[,]
        {
            { "วิทยาการคอมพิวเตอร์", "CS", "สาขาวิทยาการคอมพิวเตอร์", new DateTime(1995, 6, 15) },
            { "วิศวกรรมซอฟต์แวร์", "SE", "สาขาวิศวกรรมซอฟต์แวร์", new DateTime(2000, 8, 20) },
            { "เทคโนโลยีสารสนเทศ", "IT", "สาขาเทคโนโลยีสารสนเทศ", new DateTime(2005, 3, 10) }
        });
}
```

### 2. การสร้าง Stored Procedures

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    // สร้าง Stored Procedure
    migrationBuilder.Sql(@"
        CREATE PROCEDURE sp_GetStudentsByDepartment
            @DepartmentId INT
        AS
        BEGIN
            SELECT 
                s.Id,
                s.StudentNumber,
                s.FirstName,
                s.LastName,
                s.Email,
                d.Name AS DepartmentName
            FROM Students s
            INNER JOIN Departments d ON s.DepartmentId = d.Id
            WHERE s.DepartmentId = @DepartmentId
                AND s.Status = 1  -- Active students only
            ORDER BY s.LastName, s.FirstName
        END
    ");

    // สร้าง Function
    migrationBuilder.Sql(@"
        CREATE FUNCTION fn_CalculateGPA(@StudentId INT)
        RETURNS DECIMAL(3,2)
        AS
        BEGIN
            DECLARE @GPA DECIMAL(3,2)
            
            SELECT @GPA = AVG(
                CASE e.Grade
                    WHEN 'A' THEN 4.0
                    WHEN 'B+' THEN 3.5
                    WHEN 'B' THEN 3.0
                    WHEN 'C+' THEN 2.5
                    WHEN 'C' THEN 2.0
                    WHEN 'D+' THEN 1.5
                    WHEN 'D' THEN 1.0
                    ELSE 0.0
                END * c.Credits) / NULLIF(SUM(c.Credits), 0)
            FROM Enrollments e
            INNER JOIN Courses c ON e.CourseId = c.Id
            WHERE e.StudentId = @StudentId
                AND e.Grade IS NOT NULL
            
            RETURN ISNULL(@GPA, 0.0)
        END
    ");
}

protected override void Down(MigrationBuilder migrationBuilder)
{
    migrationBuilder.Sql("DROP PROCEDURE IF EXISTS sp_GetStudentsByDepartment");
    migrationBuilder.Sql("DROP FUNCTION IF EXISTS fn_CalculateGPA");
}
```

### 3. การสร้าง Views

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.Sql(@"
        CREATE VIEW vw_StudentSummary AS
        SELECT 
            s.Id,
            s.StudentNumber,
            s.FirstName + ' ' + s.LastName AS FullName,
            s.Email,
            d.Name AS DepartmentName,
            s.GPA,
            COUNT(e.CourseId) AS EnrolledCourses,
            s.EnrollmentDate
        FROM Students s
        LEFT JOIN Departments d ON s.DepartmentId = d.Id
        LEFT JOIN Enrollments e ON s.Id = e.StudentId
        WHERE s.Status = 1  -- Active only
        GROUP BY s.Id, s.StudentNumber, s.FirstName, s.LastName, s.Email, d.Name, s.GPA, s.EnrollmentDate
    ");
}

protected override void Down(MigrationBuilder migrationBuilder)
{
    migrationBuilder.Sql("DROP VIEW IF EXISTS vw_StudentSummary");
}
```

### 4. การ Data Migration

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    // เพิ่ม Column ใหม่
    migrationBuilder.AddColumn<string>(
        name: "FullName",
        table: "Students",
        type: "nvarchar(150)",
        maxLength: 150,
        nullable: true);

    // Migration ข้อมูลเดิม
    migrationBuilder.Sql(@"
        UPDATE Students 
        SET FullName = FirstName + ' ' + LastName
        WHERE FullName IS NULL
    ");

    // ตั้งค่าให้เป็น Required หลังจาก migrate ข้อมูลแล้ว
    migrationBuilder.AlterColumn<string>(
        name: "FullName",
        table: "Students",
        type: "nvarchar(150)",
        maxLength: 150,
        nullable: false,
        oldClrType: typeof(string),
        oldType: "nvarchar(150)",
        oldMaxLength: 150,
        oldNullable: true);
}
```

## การจัดการ Migration ใน Production

### 1. Migration Service

```csharp
public class DatabaseMigrationService
{
    private readonly ApplicationDbContext _context;
    private readonly ILogger<DatabaseMigrationService> _logger;

    public DatabaseMigrationService(
        ApplicationDbContext context,
        ILogger<DatabaseMigrationService> logger)
    {
        _context = context;
        _logger = logger;
    }

    public async Task MigrateAsync()
    {
        try
        {
            _logger.LogInformation("Starting database migration...");

            var pendingMigrations = await _context.Database.GetPendingMigrationsAsync();
            
            if (pendingMigrations.Any())
            {
                _logger.LogInformation($"Found {pendingMigrations.Count()} pending migrations");
                
                foreach (var migration in pendingMigrations)
                {
                    _logger.LogInformation($"Applying migration: {migration}");
                }

                await _context.Database.MigrateAsync();
                _logger.LogInformation("Database migration completed successfully");
            }
            else
            {
                _logger.LogInformation("No pending migrations found");
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "An error occurred while migrating the database");
            throw;
        }
    }

    public async Task<List<string>> GetAppliedMigrationsAsync()
    {
        var appliedMigrations = await _context.Database.GetAppliedMigrationsAsync();
        return appliedMigrations.ToList();
    }

    public async Task<List<string>> GetPendingMigrationsAsync()
    {
        var pendingMigrations = await _context.Database.GetPendingMigrationsAsync();
        return pendingMigrations.ToList();
    }

    public async Task<bool> CanConnectAsync()
    {
        try
        {
            return await _context.Database.CanConnectAsync();
        }
        catch
        {
            return false;
        }
    }
}
```

### 2. การใช้งานใน Program.cs

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddScoped<DatabaseMigrationService>();

var app = builder.Build();

// Auto Migration ใน Development
if (app.Environment.IsDevelopment())
{
    using var scope = app.Services.CreateScope();
    var migrationService = scope.ServiceProvider.GetRequiredService<DatabaseMigrationService>();
    await migrationService.MigrateAsync();
}

app.Run();
```

### 3. Migration Script Generation

```csharp
public class MigrationScriptGenerator
{
    private readonly IServiceProvider _serviceProvider;
    private readonly IConfiguration _configuration;

    public MigrationScriptGenerator(IServiceProvider serviceProvider, IConfiguration configuration)
    {
        _serviceProvider = serviceProvider;
        _configuration = configuration;
    }

    public async Task GenerateScriptAsync(string fromMigration = null, string toMigration = null)
    {
        using var scope = _serviceProvider.CreateScope();
        var context = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();

        var script = context.Database.GenerateCreateScript();
        
        if (!string.IsNullOrEmpty(fromMigration))
        {
            // สร้าง script สำหรับ migration ที่ระบุ
            // Note: ต้องใช้ EF Core Tools API สำหรับฟีเจอร์นี้
        }

        var fileName = $"migration_script_{DateTime.Now:yyyyMMdd_HHmmss}.sql";
        var filePath = Path.Combine(Directory.GetCurrentDirectory(), "Scripts", fileName);

        Directory.CreateDirectory(Path.GetDirectoryName(filePath)!);
        await File.WriteAllTextAsync(filePath, script);

        Console.WriteLine($"Migration script generated: {filePath}");
    }
}
```

## Best Practices สำหรับ Migrations

### 1. การตั้งชื่อ Migration

```bash
# ชื่อที่ดี - อธิบายการเปลี่ยนแปลงชัดเจน
dotnet ef migrations add AddUserProfileTable
dotnet ef migrations add UpdateStudentEmailLength
dotnet ef migrations add RemoveObsoleteColumns
dotnet ef migrations add AddIndexToStudentNumber

# ชื่อที่ไม่ดี - ไม่ชัดเจน
dotnet ef migrations add Update1
dotnet ef migrations add Changes
dotnet ef migrations add Fix
```

### 2. การ Backup ก่อน Migration

```csharp
public class SafeMigrationService
{
    private readonly ApplicationDbContext _context;
    private readonly IConfiguration _configuration;
    private readonly ILogger<SafeMigrationService> _logger;

    public async Task MigrateWithBackupAsync()
    {
        try
        {
            // Backup ฐานข้อมูลก่อน
            await BackupDatabaseAsync();
            
            // ทำ Migration
            await _context.Database.MigrateAsync();
            
            _logger.LogInformation("Migration completed successfully");
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Migration failed. Database backup is available for recovery.");
            throw;
        }
    }

    private async Task BackupDatabaseAsync()
    {
        var databaseName = _context.Database.GetDbConnection().Database;
        var backupPath = Path.Combine(
            Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments),
            "DatabaseBackups",
            $"{databaseName}_backup_{DateTime.Now:yyyyMMdd_HHmmss}.bak"
        );

        Directory.CreateDirectory(Path.GetDirectoryName(backupPath)!);

        var backupSql = $@"
            BACKUP DATABASE [{databaseName}] 
            TO DISK = N'{backupPath}' 
            WITH FORMAT, INIT, NAME = N'{databaseName}-Full Database Backup'";

        await _context.Database.ExecuteSqlRawAsync(backupSql);
        
        _logger.LogInformation($"Database backed up to: {backupPath}");
    }
}
```

### 3. การ Validate Migration

```csharp
public class MigrationValidator
{
    public async Task<bool> ValidateModelAsync(ApplicationDbContext context)
    {
        try
        {
            // ตรวจสอบว่า Model ตรงกับ Database Schema
            var creator = context.GetService<IRelationalDatabaseCreator>();
            return await creator.ExistsAsync();
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Model validation failed: {ex.Message}");
            return false;
        }
    }

    public async Task ValidateDataIntegrityAsync(ApplicationDbContext context)
    {
        // ตรวจสอบ Foreign Key constraints
        var orphanedStudents = await context.Students
            .Where(s => s.DepartmentId.HasValue)
            .Where(s => !context.Departments.Any(d => d.Id == s.DepartmentId))
            .CountAsync();

        if (orphanedStudents > 0)
        {
            throw new InvalidOperationException($"Found {orphanedStudents} students with invalid department references");
        }

        // ตรวจสอบ Required fields
        var studentsWithoutEmail = await context.Students
            .Where(s => string.IsNullOrEmpty(s.Email))
            .CountAsync();

        if (studentsWithoutEmail > 0)
        {
            throw new InvalidOperationException($"Found {studentsWithoutEmail} students without email");
        }
    }
}
```

---

**หมายเหตุ**: Migrations เป็นเครื่องมือที่ทรงพลังแต่ต้องใช้ความระมัดระวัง โดยเฉพาะใน Production Environment ควร Backup และ Test อย่างละเอียดก่อนทำการ Migration จริง