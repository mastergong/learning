# บทที่ 4: การสร้างและจัดการ Database

## การสร้าง Database

### 1. การสร้าง Database ด้วย EnsureCreated()

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);
        
        builder.Services.AddDbContext<ApplicationDbContext>(options =>
            options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

        var app = builder.Build();

        // สร้างฐานข้อมูลและตารางทั้งหมด
        using (var scope = app.Services.CreateScope())
        {
            var context = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
            
            // สร้างฐานข้อมูลหากไม่มี
            await context.Database.EnsureCreatedAsync();
            
            // ตรวจสอบว่าฐานข้อมูลมีอยู่หรือไม่
            var canConnect = await context.Database.CanConnectAsync();
            Console.WriteLine($"Database connection: {(canConnect ? "Success" : "Failed")}");
        }

        app.Run();
    }
}
```

### 2. การใช้ SQL Scripts

```csharp
public static class DatabaseInitializer
{
    public static async Task InitializeDatabaseAsync(ApplicationDbContext context)
    {
        // ลบฐานข้อมูลเดิม (ระวังใช้เฉพาะ Development)
        if (await context.Database.EnsureDeletedAsync())
        {
            Console.WriteLine("Database deleted successfully.");
        }

        // สร้างฐานข้อมูลใหม่
        if (await context.Database.EnsureCreatedAsync())
        {
            Console.WriteLine("Database created successfully.");
        }

        // รัน SQL Script เพิ่มเติม
        await ExecuteCustomScripts(context);
    }

    private static async Task ExecuteCustomScripts(ApplicationDbContext context)
    {
        // สร้าง Views
        var createViewSql = @"
            CREATE VIEW vw_StudentSummary AS
            SELECT 
                s.Id,
                s.StudentNumber,
                s.FirstName + ' ' + s.LastName AS FullName,
                s.Email,
                d.Name AS DepartmentName,
                COUNT(e.CourseId) AS EnrolledCoursesCount
            FROM Students s
            LEFT JOIN Departments d ON s.DepartmentId = d.Id
            LEFT JOIN Enrollments e ON s.Id = e.StudentId
            GROUP BY s.Id, s.StudentNumber, s.FirstName, s.LastName, s.Email, d.Name
        ";

        await context.Database.ExecuteSqlRawAsync(createViewSql);

        // สร้าง Stored Procedures
        var createProcedureSql = @"
            CREATE PROCEDURE sp_GetStudentsByDepartment
                @DepartmentId INT
            AS
            BEGIN
                SELECT s.*, d.Name AS DepartmentName
                FROM Students s
                INNER JOIN Departments d ON s.DepartmentId = d.Id
                WHERE s.DepartmentId = @DepartmentId
                ORDER BY s.LastName, s.FirstName
            END
        ";

        await context.Database.ExecuteSqlRawAsync(createProcedureSql);

        // สร้าง Functions
        var createFunctionSql = @"
            CREATE FUNCTION fn_CalculateAge(@DateOfBirth DATE)
            RETURNS INT
            AS
            BEGIN
                DECLARE @Age INT
                SET @Age = DATEDIFF(YEAR, @DateOfBirth, GETDATE())
                IF (MONTH(@DateOfBirth) > MONTH(GETDATE()) OR 
                   (MONTH(@DateOfBirth) = MONTH(GETDATE()) AND DAY(@DateOfBirth) > DAY(GETDATE())))
                    SET @Age = @Age - 1
                RETURN @Age
            END
        ";

        await context.Database.ExecuteSqlRawAsync(createFunctionSql);
    }
}
```

### 3. การสร้าง Database จาก Code First

```csharp
public class SchoolDbContextInitializer
{
    private readonly ApplicationDbContext _context;
    private readonly ILogger<SchoolDbContextInitializer> _logger;

    public SchoolDbContextInitializer(
        ApplicationDbContext context,
        ILogger<SchoolDbContextInitializer> logger)
    {
        _context = context;
        _logger = logger;
    }

    public async Task InitializeAsync()
    {
        try
        {
            // สร้างฐานข้อมูล
            await _context.Database.EnsureCreatedAsync();
            _logger.LogInformation("Database created successfully");

            // เรียก Seed Data
            await SeedDataAsync();
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "An error occurred while initializing the database");
            throw;
        }
    }

    private async Task SeedDataAsync()
    {
        // ตรวจสอบว่ามีข้อมูลแล้วหรือไม่
        if (await _context.Departments.AnyAsync())
        {
            _logger.LogInformation("Database already contains data");
            return;
        }

        // เพิ่มข้อมูลพื้นฐาน
        await SeedDepartmentsAsync();
        await SeedInstructorsAsync();
        await SeedCoursesAsync();
        await SeedStudentsAsync();
        await SeedEnrollmentsAsync();

        await _context.SaveChangesAsync();
        _logger.LogInformation("Database seeded successfully");
    }

    private async Task SeedDepartmentsAsync()
    {
        var departments = new[]
        {
            new Department
            {
                Name = "วิทยาการคอมพิวเตอร์",
                Code = "CS",
                Description = "สาขาวิชาวิทยาการคอมพิวเตอร์",
                EstablishedDate = new DateTime(1995, 6, 15),
                Building = "อาคารวิทยาศาสตร์",
                Floor = "ชั้น 3",
                PhoneNumber = "02-123-4567",
                Email = "cs@university.ac.th"
            },
            new Department
            {
                Name = "วิศวกรรมซอฟต์แวร์",
                Code = "SE",
                Description = "สาขาวิชาวิศวกรรมซอฟต์แวร์",
                EstablishedDate = new DateTime(2000, 8, 20),
                Building = "อาคารวิศวกรรม",
                Floor = "ชั้น 2",
                PhoneNumber = "02-123-4568",
                Email = "se@university.ac.th"
            }
        };

        await _context.Departments.AddRangeAsync(departments);
    }

    private async Task SeedInstructorsAsync()
    {
        var instructors = new[]
        {
            new Instructor
            {
                FirstName = "ดร.สมชาย",
                LastName = "วิทยาการ",
                Email = "somchai.v@university.ac.th",
                PhoneNumber = "02-123-4569",
                HireDate = new DateTime(2010, 1, 15),
                DepartmentId = 1, // CS Department
                Position = "อาจารย์ประจำ"
            },
            new Instructor
            {
                FirstName = "ดร.สมหญิง",
                LastName = "ซอฟต์แวร์",
                Email = "somying.s@university.ac.th",
                PhoneNumber = "02-123-4570",
                HireDate = new DateTime(2015, 3, 1),
                DepartmentId = 2, // SE Department
                Position = "อาจารย์ประจำ"
            }
        };

        await _context.Instructors.AddRangeAsync(instructors);
    }

    private async Task SeedCoursesAsync()
    {
        var courses = new[]
        {
            new Course
            {
                CourseCode = "CS101",
                Title = "การเขียนโปรแกรมเบื้องต้น",
                Credits = 3.0m,
                Description = "เรียนรู้พื้นฐานการเขียนโปรแกรม",
                DepartmentId = 1,
                InstructorId = 1,
                LectureHours = 3,
                LabHours = 3,
                Semester = Semester.First,
                AcademicYear = 2024,
                MaxStudents = 40
            },
            new Course
            {
                CourseCode = "CS102",
                Title = "โครงสร้างข้อมูลและอัลกอริทึม",
                Credits = 3.0m,
                Description = "เรียนรู้โครงสร้างข้อมูลพื้นฐาน",
                DepartmentId = 1,
                InstructorId = 1,
                LectureHours = 3,
                LabHours = 3,
                Semester = Semester.Second,
                AcademicYear = 2024,
                MaxStudents = 35
            }
        };

        await _context.Courses.AddRangeAsync(courses);
    }

    private async Task SeedStudentsAsync()
    {
        var students = new[]
        {
            new Student
            {
                StudentNumber = "6506001",
                FirstName = "นางสาวสุดารัตน์",
                LastName = "ใจดี",
                Email = "sudarat.j@student.university.ac.th",
                PhoneNumber = "081-234-5678",
                DateOfBirth = new DateTime(2002, 5, 15),
                Gender = Gender.Female,
                DepartmentId = 1,
                EnrollmentDate = new DateTime(2023, 6, 1),
                Status = StudentStatus.Active,
                Address = "123 ถนนสุขุมวิท กรุงเทพฯ 10110"
            },
            new Student
            {
                StudentNumber = "6506002",
                FirstName = "นายสมศักดิ์",
                LastName = "เก่งการ",
                Email = "somsak.k@student.university.ac.th",
                PhoneNumber = "081-234-5679",
                DateOfBirth = new DateTime(2001, 12, 3),
                Gender = Gender.Male,
                DepartmentId = 1,
                EnrollmentDate = new DateTime(2023, 6, 1),
                Status = StudentStatus.Active,
                Address = "456 ถนนรัชดาภิเษก กรุงเทพฯ 10400"
            }
        };

        await _context.Students.AddRangeAsync(students);
    }

    private async Task SeedEnrollmentsAsync()
    {
        // ต้องบันทึกข้อมูลก่อนเพื่อให้มี Id
        await _context.SaveChangesAsync();

        var enrollments = new[]
        {
            new Enrollment
            {
                StudentId = 1,
                CourseId = 1,
                EnrollmentDate = new DateTime(2024, 1, 15),
                Grade = "A"
            },
            new Enrollment
            {
                StudentId = 2,
                CourseId = 1,
                EnrollmentDate = new DateTime(2024, 1, 15),
                Grade = "B+"
            }
        };

        await _context.Enrollments.AddRangeAsync(enrollments);
    }
}
```

## การจัดการ Database Connection

### 1. Connection String Management

```csharp
public class DatabaseConnectionService
{
    private readonly IConfiguration _configuration;
    private readonly ILogger<DatabaseConnectionService> _logger;

    public DatabaseConnectionService(IConfiguration configuration, ILogger<DatabaseConnectionService> logger)
    {
        _configuration = configuration;
        _logger = logger;
    }

    public string GetConnectionString(string name = "DefaultConnection")
    {
        var connectionString = _configuration.GetConnectionString(name);
        
        if (string.IsNullOrEmpty(connectionString))
        {
            throw new InvalidOperationException($"Connection string '{name}' not found");
        }

        // แทนที่ Environment Variables
        connectionString = ReplaceEnvironmentVariables(connectionString);
        
        return connectionString;
    }

    private string ReplaceEnvironmentVariables(string connectionString)
    {
        var replacements = new Dictionary<string, string>
        {
            { "{DB_SERVER}", Environment.GetEnvironmentVariable("DB_SERVER") ?? "localhost" },
            { "{DB_NAME}", Environment.GetEnvironmentVariable("DB_NAME") ?? "SchoolDB" },
            { "{DB_USER}", Environment.GetEnvironmentVariable("DB_USER") ?? "" },
            { "{DB_PASSWORD}", Environment.GetEnvironmentVariable("DB_PASSWORD") ?? "" }
        };

        foreach (var replacement in replacements)
        {
            connectionString = connectionString.Replace(replacement.Key, replacement.Value);
        }

        return connectionString;
    }

    public async Task<bool> TestConnectionAsync()
    {
        try
        {
            var options = new DbContextOptionsBuilder<ApplicationDbContext>()
                .UseSqlServer(GetConnectionString())
                .Options;

            using var context = new ApplicationDbContext(options);
            await context.Database.CanConnectAsync();
            
            _logger.LogInformation("Database connection test successful");
            return true;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Database connection test failed");
            return false;
        }
    }
}
```

### 2. Connection Pooling Configuration

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // การกำหนดค่า Connection Pool
        services.AddDbContextPool<ApplicationDbContext>(options =>
        {
            options.UseSqlServer(connectionString, sqlOptions =>
            {
                // Command Timeout
                sqlOptions.CommandTimeout(30);
                
                // Connection Resilience
                sqlOptions.EnableRetryOnFailure(
                    maxRetryCount: 5,
                    maxRetryDelay: TimeSpan.FromSeconds(10),
                    errorNumbersToAdd: null);
                
                // Migration Assembly
                sqlOptions.MigrationsAssembly("EFCoreDemo.Data");
            });
            
            // Query ที่ซับซ้อน
            options.EnableServiceProviderCaching();
            options.EnableSensitiveDataLogging(isDevelopment);
            
        }, poolSize: 128); // ขนาด Connection Pool
    }
}
```

### 3. Multiple Database Support

```csharp
public class MultiDatabaseConfiguration
{
    public static void ConfigureDatabases(IServiceCollection services, IConfiguration configuration)
    {
        // Primary Database (SQL Server)
        services.AddDbContext<ApplicationDbContext>(options =>
            options.UseSqlServer(configuration.GetConnectionString("SqlServer")));

        // Secondary Database (PostgreSQL) สำหรับ Analytics
        services.AddDbContext<AnalyticsDbContext>(options =>
            options.UseNpgsql(configuration.GetConnectionString("PostgreSQL")));

        // Cache Database (Redis) สำหรับ Session
        services.AddDbContext<CacheDbContext>(options =>
            options.UseInMemoryDatabase("CacheDatabase"));

        // Audit Database (SQLite) สำหรับ Logging
        services.AddDbContext<AuditDbContext>(options =>
            options.UseSqlite(configuration.GetConnectionString("Audit")));
    }
}

// appsettings.json
{
  "ConnectionStrings": {
    "SqlServer": "Server=localhost;Database=SchoolDB;Trusted_Connection=true;",
    "PostgreSQL": "Host=localhost;Database=school_analytics;Username=postgres;Password=password",
    "Audit": "Data Source=audit.db"
  }
}
```

## Database Operations

### 1. การตรวจสอบและจัดการ Database

```csharp
public class DatabaseManager
{
    private readonly ApplicationDbContext _context;
    private readonly ILogger<DatabaseManager> _logger;

    public DatabaseManager(ApplicationDbContext context, ILogger<DatabaseManager> logger)
    {
        _context = context;
        _logger = logger;
    }

    public async Task<bool> DatabaseExistsAsync()
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

    public async Task CreateDatabaseAsync()
    {
        var created = await _context.Database.EnsureCreatedAsync();
        if (created)
        {
            _logger.LogInformation("Database created successfully");
        }
        else
        {
            _logger.LogInformation("Database already exists");
        }
    }

    public async Task DeleteDatabaseAsync()
    {
        var deleted = await _context.Database.EnsureDeletedAsync();
        if (deleted)
        {
            _logger.LogWarning("Database deleted successfully");
        }
        else
        {
            _logger.LogInformation("Database did not exist");
        }
    }

    public async Task<List<string>> GetTableNamesAsync()
    {
        var sql = @"
            SELECT TABLE_NAME 
            FROM INFORMATION_SCHEMA.TABLES 
            WHERE TABLE_TYPE = 'BASE TABLE'
            ORDER BY TABLE_NAME";

        var connection = _context.Database.GetDbConnection();
        await connection.OpenAsync();

        using var command = connection.CreateCommand();
        command.CommandText = sql;

        var tableNames = new List<string>();
        using var reader = await command.ExecuteReaderAsync();
        
        while (await reader.ReadAsync())
        {
            tableNames.Add(reader.GetString("TABLE_NAME"));
        }

        return tableNames;
    }

    public async Task<DatabaseInfo> GetDatabaseInfoAsync()
    {
        var info = new DatabaseInfo();

        // ข้อมูลทั่วไป
        info.DatabaseName = _context.Database.GetDbConnection().Database;
        info.ProviderName = _context.Database.ProviderName;

        // นับจำนวนตาราง
        var sql = @"
            SELECT 
                COUNT(*) as TableCount,
                (SELECT COUNT(*) FROM INFORMATION_SCHEMA.VIEWS) as ViewCount,
                (SELECT COUNT(*) FROM INFORMATION_SCHEMA.ROUTINES WHERE ROUTINE_TYPE = 'PROCEDURE') as ProcedureCount
            FROM INFORMATION_SCHEMA.TABLES 
            WHERE TABLE_TYPE = 'BASE TABLE'";

        var result = await _context.Database
            .SqlQueryRaw<dynamic>(sql)
            .FirstOrDefaultAsync();

        // ขนาดฐานข้อมูล
        var sizeSql = @"
            SELECT 
                SUM(size * 8.0 / 1024) as SizeMB
            FROM sys.database_files";

        var sizeResult = await _context.Database
            .SqlQueryRaw<decimal>(sizeSql)
            .FirstOrDefaultAsync();

        info.SizeMB = sizeResult;

        return info;
    }

    public async Task BackupDatabaseAsync(string backupPath)
    {
        var databaseName = _context.Database.GetDbConnection().Database;
        var sql = $@"
            BACKUP DATABASE [{databaseName}] 
            TO DISK = '{backupPath}' 
            WITH FORMAT, INIT";

        await _context.Database.ExecuteSqlRawAsync(sql);
        _logger.LogInformation($"Database backed up to {backupPath}");
    }

    public async Task RestoreDatabaseAsync(string backupPath)
    {
        var databaseName = _context.Database.GetDbConnection().Database;
        var sql = $@"
            RESTORE DATABASE [{databaseName}] 
            FROM DISK = '{backupPath}' 
            WITH REPLACE";

        await _context.Database.ExecuteSqlRawAsync(sql);
        _logger.LogInformation($"Database restored from {backupPath}");
    }
}

public class DatabaseInfo
{
    public string DatabaseName { get; set; }
    public string ProviderName { get; set; }
    public int TableCount { get; set; }
    public int ViewCount { get; set; }
    public int ProcedureCount { get; set; }
    public decimal SizeMB { get; set; }
    public DateTime LastBackup { get; set; }
}
```

### 2. การทำ Health Check

```csharp
public class DatabaseHealthCheck : IHealthCheck
{
    private readonly ApplicationDbContext _context;

    public DatabaseHealthCheck(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context, 
        CancellationToken cancellationToken = default)
    {
        try
        {
            // ทดสอบการเชื่อมต่อ
            await _context.Database.CanConnectAsync(cancellationToken);

            // ทดสอบ Query
            var studentCount = await _context.Students.CountAsync(cancellationToken);

            var data = new Dictionary<string, object>
            {
                { "database", _context.Database.GetDbConnection().Database },
                { "provider", _context.Database.ProviderName },
                { "student_count", studentCount }
            };

            return HealthCheckResult.Healthy("Database is healthy", data);
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Unhealthy("Database is unhealthy", ex);
        }
    }
}

// Registration
builder.Services.AddHealthChecks()
    .AddCheck<DatabaseHealthCheck>("database");
```

---

**หมายเหตุ**: การจัดการฐานข้อมูลอย่างถูกต้องเป็นสิ่งสำคัญ โดยเฉพาะในสภาพแวดล้อม Production ควรมีการ Backup และ Monitor อย่างสม่ำเสมอ