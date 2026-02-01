# บทที่ 8: การสร้างและจัดการ Index

## Index คืออะไร?

Index เป็นโครงสร้างข้อมูลที่ช่วยเพิ่มความเร็วในการค้นหาข้อมูลในฐานข้อมูล โดยสร้างตัวชี้ (pointer) ไปยังข้อมูลจริงในตาราง

### ประเภทของ Index

1. **Clustered Index**: กำหนดการเรียงลำดับข้อมูลจริงในตาราง (มีได้ 1 อันต่อตาราง)
2. **Non-Clustered Index**: สร้างโครงสร้างแยกจากข้อมูลจริง
3. **Unique Index**: ป้องกันค่าซ้ำ
4. **Composite Index**: Index ที่ประกอบด้วยหลาย columns
5. **Partial Index**: Index เฉพาะข้อมูลที่ตรงตามเงื่อนไข

## การสร้าง Index ด้วย Data Annotations

### 1. Basic Index

```csharp
// Single column index
[Index(nameof(Email), IsUnique = true)]
[Index(nameof(StudentNumber), IsUnique = true)]
public class Student
{
    public int Id { get; set; }
    
    [Required]
    [StringLength(20)]
    public string StudentNumber { get; set; }
    
    [Required]
    [StringLength(100)]
    [EmailAddress]
    public string Email { get; set; }
    
    [StringLength(50)]
    public string FirstName { get; set; }
    
    [StringLength(50)]
    public string LastName { get; set; }
}
```

### 2. Composite Index

```csharp
// Multiple column index
[Index(nameof(FirstName), nameof(LastName), Name = "IX_Student_FullName")]
[Index(nameof(DepartmentId), nameof(Status), Name = "IX_Student_Dept_Status")]
[Index(nameof(EnrollmentDate), nameof(Status), Name = "IX_Student_Enrollment_Status")]
public class Student
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public int? DepartmentId { get; set; }
    public StudentStatus Status { get; set; }
    public DateTime EnrollmentDate { get; set; }
}
```

### 3. Partial Index (Filter)

```csharp
// Index เฉพาะข้อมูลที่ Active
[Index(nameof(Email), IsUnique = true, Filter = "IsDeleted = 0")]
[Index(nameof(Status), Filter = "Status = 1")] // Active students only
public class Student
{
    public int Id { get; set; }
    public string Email { get; set; }
    public StudentStatus Status { get; set; }
    public bool IsDeleted { get; set; } = false;
}
```

## การสร้าง Index ด้วย Fluent API

### 1. Basic Index Configuration

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    var studentEntity = modelBuilder.Entity<Student>();

    // Single column index
    studentEntity.HasIndex(s => s.Email)
        .IsUnique()
        .HasDatabaseName("IX_Student_Email");

    studentEntity.HasIndex(s => s.StudentNumber)
        .IsUnique()
        .HasDatabaseName("IX_Student_Number");

    // Non-unique index
    studentEntity.HasIndex(s => s.LastName)
        .HasDatabaseName("IX_Student_LastName");

    // Index with included columns (SQL Server only)
    studentEntity.HasIndex(s => s.DepartmentId)
        .HasDatabaseName("IX_Student_Department")
        .IncludeProperties(s => new { s.FirstName, s.LastName, s.Email });
}
```

### 2. Composite Index

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    var studentEntity = modelBuilder.Entity<Student>();

    // Composite index
    studentEntity.HasIndex(s => new { s.FirstName, s.LastName })
        .HasDatabaseName("IX_Student_FullName");

    // Composite unique index
    studentEntity.HasIndex(s => new { s.DepartmentId, s.StudentNumber })
        .IsUnique()
        .HasDatabaseName("IX_Student_Dept_Number");

    // Complex composite index
    studentEntity.HasIndex(s => new { s.DepartmentId, s.EnrollmentDate, s.Status })
        .HasDatabaseName("IX_Student_Dept_Enrollment_Status")
        .HasFilter("Status = 1"); // SQL Server filter
}
```

### 3. Conditional และ Filtered Index

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    var studentEntity = modelBuilder.Entity<Student>();

    // Filtered index สำหรับ Active students เท่านั้น
    studentEntity.HasIndex(s => s.Email)
        .IsUnique()
        .HasFilter("Status = 1 AND IsDeleted = 0")
        .HasDatabaseName("IX_Student_Email_Active");

    // Partial index สำหรับค่าที่ไม่เป็น null
    studentEntity.HasIndex(s => s.PhoneNumber)
        .HasFilter("PhoneNumber IS NOT NULL")
        .HasDatabaseName("IX_Student_Phone");

    // Sparse index สำหรับ Foreign Key
    studentEntity.HasIndex(s => s.DepartmentId)
        .HasFilter("DepartmentId IS NOT NULL")
        .HasDatabaseName("IX_Student_Department_NotNull");
}
```

### 4. Include Columns (SQL Server)

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    var studentEntity = modelBuilder.Entity<Student>();
    
    // Index with included columns เพื่อ Covering Query
    studentEntity.HasIndex(s => new { s.DepartmentId, s.Status })
        .IncludeProperties(s => new { 
            s.FirstName, 
            s.LastName, 
            s.Email, 
            s.EnrollmentDate 
        })
        .HasDatabaseName("IX_Student_Dept_Status_Covering");

    // Single column index with includes
    studentEntity.HasIndex(s => s.LastName)
        .IncludeProperties(s => new { s.FirstName, s.Email, s.PhoneNumber })
        .HasDatabaseName("IX_Student_LastName_Covering");
}
```

## Index สำหรับความสัมพันธ์

### 1. Foreign Key Index

```csharp
public class Course
{
    public int Id { get; set; }
    public string CourseCode { get; set; }
    public string Title { get; set; }
    
    // Foreign Keys
    public int DepartmentId { get; set; }
    public int? InstructorId { get; set; }
    
    // Navigation Properties
    public virtual Department Department { get; set; }
    public virtual Instructor? Instructor { get; set; }
    public virtual ICollection<Enrollment> Enrollments { get; set; }
}

// Configuration
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    var courseEntity = modelBuilder.Entity<Course>();

    // Foreign Key indexes สร้างอัตโนมัติ แต่สามารถปรับแต่งได้
    courseEntity.HasIndex(c => c.DepartmentId)
        .HasDatabaseName("IX_Course_Department");

    courseEntity.HasIndex(c => c.InstructorId)
        .HasDatabaseName("IX_Course_Instructor");

    // Composite index สำหรับ query ที่ซับซ้อน
    courseEntity.HasIndex(c => new { c.DepartmentId, c.Semester, c.AcademicYear })
        .HasDatabaseName("IX_Course_Dept_Semester_Year");
}
```

### 2. Many-to-Many Index

```csharp
public class Enrollment
{
    public int StudentId { get; set; }
    public int CourseId { get; set; }
    public DateTime EnrollmentDate { get; set; }
    public string? Grade { get; set; }
    public EnrollmentStatus Status { get; set; }

    public virtual Student Student { get; set; }
    public virtual Course Course { get; set; }
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    var enrollmentEntity = modelBuilder.Entity<Enrollment>();

    // Primary key (composite)
    enrollmentEntity.HasKey(e => new { e.StudentId, e.CourseId });

    // Additional indexes สำหรับ performance
    enrollmentEntity.HasIndex(e => e.StudentId)
        .HasDatabaseName("IX_Enrollment_Student");

    enrollmentEntity.HasIndex(e => e.CourseId)
        .HasDatabaseName("IX_Enrollment_Course");

    enrollmentEntity.HasIndex(e => e.EnrollmentDate)
        .HasDatabaseName("IX_Enrollment_Date");

    // Composite index สำหรับ reporting queries
    enrollmentEntity.HasIndex(e => new { e.StudentId, e.Status, e.EnrollmentDate })
        .HasDatabaseName("IX_Enrollment_Student_Status_Date");

    enrollmentEntity.HasIndex(e => new { e.CourseId, e.Status })
        .IncludeProperties(e => new { e.Grade, e.EnrollmentDate })
        .HasDatabaseName("IX_Enrollment_Course_Status_Covering");
}
```

## การปรับแต่ง Index เพื่อประสิทธิภาพ

### 1. Index สำหรับการค้นหา

```csharp
public class ProductSearchIndex
{
    public static void Configure(ModelBuilder modelBuilder)
    {
        var productEntity = modelBuilder.Entity<Product>();

        // Full-text search index
        productEntity.HasIndex(p => p.Name)
            .HasDatabaseName("IX_Product_Name_Search");

        productEntity.HasIndex(p => p.Description)
            .HasDatabaseName("IX_Product_Description_Search");

        // Category + Status search
        productEntity.HasIndex(p => new { p.CategoryId, p.Status })
            .IncludeProperties(p => new { 
                p.Name, 
                p.Price, 
                p.StockQuantity,
                p.CreatedDate 
            })
            .HasDatabaseName("IX_Product_Category_Status_Search");

        // Price range queries
        productEntity.HasIndex(p => p.Price)
            .HasDatabaseName("IX_Product_Price");

        // Date range queries
        productEntity.HasIndex(p => p.CreatedDate)
            .HasDatabaseName("IX_Product_CreatedDate");

        // Complex search scenarios
        productEntity.HasIndex(p => new { p.CategoryId, p.Price, p.Status })
            .HasDatabaseName("IX_Product_Category_Price_Status");
    }
}
```

### 2. Index สำหรับ Sorting

```csharp
public class StudentSortingIndex
{
    public static void Configure(ModelBuilder modelBuilder)
    {
        var studentEntity = modelBuilder.Entity<Student>();

        // Sorting by name
        studentEntity.HasIndex(s => new { s.LastName, s.FirstName })
            .HasDatabaseName("IX_Student_Name_Sort");

        // Sorting by enrollment date
        studentEntity.HasIndex(s => s.EnrollmentDate)
            .HasDatabaseName("IX_Student_EnrollmentDate_Sort");

        // Department-wise sorting
        studentEntity.HasIndex(s => new { s.DepartmentId, s.LastName, s.FirstName })
            .HasDatabaseName("IX_Student_Dept_Name_Sort");

        // GPA ranking
        studentEntity.HasIndex(s => new { s.DepartmentId, s.GPA })
            .HasFilter("GPA IS NOT NULL")
            .HasDatabaseName("IX_Student_Dept_GPA_Ranking");
    }
}
```

### 3. Index สำหรับ Aggregation

```csharp
public class ReportingIndex
{
    public static void Configure(ModelBuilder modelBuilder)
    {
        var enrollmentEntity = modelBuilder.Entity<Enrollment>();

        // Student count per course
        enrollmentEntity.HasIndex(e => new { e.CourseId, e.Status })
            .HasDatabaseName("IX_Enrollment_Course_Count");

        // Course count per student
        enrollmentEntity.HasIndex(e => new { e.StudentId, e.Status })
            .HasDatabaseName("IX_Enrollment_Student_Count");

        // Enrollment by date ranges
        enrollmentEntity.HasIndex(e => e.EnrollmentDate)
            .IncludeProperties(e => new { e.StudentId, e.CourseId, e.Status })
            .HasDatabaseName("IX_Enrollment_Date_Report");

        // Grade distribution
        enrollmentEntity.HasIndex(e => new { e.CourseId, e.Grade })
            .HasFilter("Grade IS NOT NULL")
            .HasDatabaseName("IX_Enrollment_Course_Grade_Stats");
    }
}
```

## การจัดการ Index ใน Production

### 1. Index Monitoring Service

```csharp
public class IndexMonitoringService
{
    private readonly ApplicationDbContext _context;
    private readonly ILogger<IndexMonitoringService> _logger;

    public IndexMonitoringService(ApplicationDbContext context, ILogger<IndexMonitoringService> logger)
    {
        _context = context;
        _logger = logger;
    }

    public async Task<List<IndexUsageStats>> GetIndexUsageStatsAsync()
    {
        var sql = @"
            SELECT 
                i.name AS IndexName,
                t.name AS TableName,
                s.user_seeks AS UserSeeks,
                s.user_scans AS UserScans,
                s.user_lookups AS UserLookups,
                s.user_updates AS UserUpdates,
                s.last_user_seek AS LastUserSeek,
                s.last_user_scan AS LastUserScan,
                s.last_user_lookup AS LastUserLookup
            FROM sys.indexes i
            INNER JOIN sys.tables t ON i.object_id = t.object_id
            LEFT JOIN sys.dm_db_index_usage_stats s ON i.object_id = s.object_id AND i.index_id = s.index_id
            WHERE i.index_id > 0  -- Exclude heap tables
                AND t.is_ms_shipped = 0  -- Exclude system tables
            ORDER BY s.user_seeks + s.user_scans + s.user_lookups DESC";

        var result = new List<IndexUsageStats>();
        
        using var command = _context.Database.GetDbConnection().CreateCommand();
        command.CommandText = sql;
        
        await _context.Database.OpenConnectionAsync();
        using var reader = await command.ExecuteReaderAsync();
        
        while (await reader.ReadAsync())
        {
            result.Add(new IndexUsageStats
            {
                IndexName = reader.GetString("IndexName"),
                TableName = reader.GetString("TableName"),
                UserSeeks = reader.IsDBNull("UserSeeks") ? 0 : reader.GetInt64("UserSeeks"),
                UserScans = reader.IsDBNull("UserScans") ? 0 : reader.GetInt64("UserScans"),
                UserLookups = reader.IsDBNull("UserLookups") ? 0 : reader.GetInt64("UserLookups"),
                UserUpdates = reader.IsDBNull("UserUpdates") ? 0 : reader.GetInt64("UserUpdates"),
                LastUserSeek = reader.IsDBNull("LastUserSeek") ? null : reader.GetDateTime("LastUserSeek"),
                LastUserScan = reader.IsDBNull("LastUserScan") ? null : reader.GetDateTime("LastUserScan"),
                LastUserLookup = reader.IsDBNull("LastUserLookup") ? null : reader.GetDateTime("LastUserLookup")
            });
        }

        return result;
    }

    public async Task<List<MissingIndexInfo>> GetMissingIndexesAsync()
    {
        var sql = @"
            SELECT 
                d.statement AS TableName,
                d.equality_columns AS EqualityColumns,
                d.inequality_columns AS InequalityColumns,
                d.included_columns AS IncludedColumns,
                s.user_seeks AS UserSeeks,
                s.user_scans AS UserScans,
                s.avg_total_user_cost AS AvgTotalUserCost,
                s.avg_user_impact AS AvgUserImpact,
                s.avg_total_user_cost * s.avg_user_impact * (s.user_seeks + s.user_scans) AS ImpactScore
            FROM sys.dm_db_missing_index_details d
            INNER JOIN sys.dm_db_missing_index_groups g ON d.index_handle = g.index_handle
            INNER JOIN sys.dm_db_missing_index_group_stats s ON g.index_group_handle = s.group_handle
            ORDER BY ImpactScore DESC";

        var result = new List<MissingIndexInfo>();
        
        using var command = _context.Database.GetDbConnection().CreateCommand();
        command.CommandText = sql;
        
        await _context.Database.OpenConnectionAsync();
        using var reader = await command.ExecuteReaderAsync();
        
        while (await reader.ReadAsync())
        {
            result.Add(new MissingIndexInfo
            {
                TableName = reader.GetString("TableName"),
                EqualityColumns = reader.IsDBNull("EqualityColumns") ? null : reader.GetString("EqualityColumns"),
                InequalityColumns = reader.IsDBNull("InequalityColumns") ? null : reader.GetString("InequalityColumns"),
                IncludedColumns = reader.IsDBNull("IncludedColumns") ? null : reader.GetString("IncludedColumns"),
                UserSeeks = reader.GetInt64("UserSeeks"),
                UserScans = reader.GetInt64("UserScans"),
                AvgTotalUserCost = reader.GetDouble("AvgTotalUserCost"),
                AvgUserImpact = reader.GetDouble("AvgUserImpact"),
                ImpactScore = reader.GetDouble("ImpactScore")
            });
        }

        return result;
    }
}

public class IndexUsageStats
{
    public string IndexName { get; set; }
    public string TableName { get; set; }
    public long UserSeeks { get; set; }
    public long UserScans { get; set; }
    public long UserLookups { get; set; }
    public long UserUpdates { get; set; }
    public DateTime? LastUserSeek { get; set; }
    public DateTime? LastUserScan { get; set; }
    public DateTime? LastUserLookup { get; set; }
    
    public long TotalReads => UserSeeks + UserScans + UserLookups;
    public bool IsUnused => TotalReads == 0;
    public double ReadWriteRatio => UserUpdates == 0 ? TotalReads : (double)TotalReads / UserUpdates;
}

public class MissingIndexInfo
{
    public string TableName { get; set; }
    public string? EqualityColumns { get; set; }
    public string? InequalityColumns { get; set; }
    public string? IncludedColumns { get; set; }
    public long UserSeeks { get; set; }
    public long UserScans { get; set; }
    public double AvgTotalUserCost { get; set; }
    public double AvgUserImpact { get; set; }
    public double ImpactScore { get; set; }
}
```

### 2. การสร้าง Index ด้วย Migration

```csharp
// Migration สำหรับ Index
public partial class AddPerformanceIndexes : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // Student search indexes
        migrationBuilder.CreateIndex(
            name: "IX_Student_Search_Name",
            table: "Students",
            columns: new[] { "LastName", "FirstName" })
            .Annotation("SqlServer:Include", new[] { "Email", "StudentNumber", "DepartmentId" });

        migrationBuilder.CreateIndex(
            name: "IX_Student_Department_Status",
            table: "Students",
            columns: new[] { "DepartmentId", "Status" },
            filter: "Status = 1 AND IsDeleted = 0");

        // Course enrollment indexes
        migrationBuilder.CreateIndex(
            name: "IX_Enrollment_Student_Active",
            table: "Enrollments",
            columns: new[] { "StudentId", "Status" },
            filter: "Status = 1")
            .Annotation("SqlServer:Include", new[] { "CourseId", "Grade", "EnrollmentDate" });

        migrationBuilder.CreateIndex(
            name: "IX_Enrollment_Course_Date",
            table: "Enrollments",
            columns: new[] { "CourseId", "EnrollmentDate" });

        // Reporting indexes
        migrationBuilder.CreateIndex(
            name: "IX_Student_GPA_Ranking",
            table: "Students",
            columns: new[] { "DepartmentId", "GPA" },
            filter: "GPA IS NOT NULL AND Status = 1",
            descending: new[] { false, true }); // GPA descending for ranking
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropIndex("IX_Student_Search_Name", "Students");
        migrationBuilder.DropIndex("IX_Student_Department_Status", "Students");
        migrationBuilder.DropIndex("IX_Enrollment_Student_Active", "Enrollments");
        migrationBuilder.DropIndex("IX_Enrollment_Course_Date", "Enrollments");
        migrationBuilder.DropIndex("IX_Student_GPA_Ranking", "Students");
    }
}
```

## Best Practices สำหรับ Index

### 1. Index Strategy Guidelines

```csharp
public static class IndexBestPractices
{
    // DO: Index for foreign keys
    public static void ConfigureForeignKeyIndexes(ModelBuilder modelBuilder)
    {
        // EF Core จะสร้างอัตโนมัติแต่สามารถปรับแต่งได้
        modelBuilder.Entity<Student>()
            .HasIndex(s => s.DepartmentId)
            .HasDatabaseName("IX_Student_Department");
    }

    // DO: Index for unique constraints
    public static void ConfigureUniqueIndexes(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Student>()
            .HasIndex(s => s.Email)
            .IsUnique()
            .HasFilter("IsDeleted = 0"); // Partial unique index
    }

    // DO: Composite indexes for multi-column queries
    public static void ConfigureCompositeIndexes(ModelBuilder modelBuilder)
    {
        // Order matters: most selective column first
        modelBuilder.Entity<Student>()
            .HasIndex(s => new { s.DepartmentId, s.Status, s.EnrollmentDate })
            .HasDatabaseName("IX_Student_Dept_Status_Date");
    }

    // DON'T: Too many indexes on frequently updated tables
    // DON'T: Duplicate indexes
    // DON'T: Indexes on small tables (< 1000 rows usually)

    // DO: Include columns for covering indexes
    public static void ConfigureCoveringIndexes(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Student>()
            .HasIndex(s => new { s.DepartmentId, s.Status })
            .IncludeProperties(s => new { s.FirstName, s.LastName, s.Email })
            .HasDatabaseName("IX_Student_Dept_Status_Covering");
    }

    // DO: Filtered indexes for sparse data
    public static void ConfigureFilteredIndexes(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Student>()
            .HasIndex(s => s.PhoneNumber)
            .HasFilter("PhoneNumber IS NOT NULL AND PhoneNumber <> ''")
            .HasDatabaseName("IX_Student_Phone_NotEmpty");
    }
}
```

### 2. Performance Testing

```csharp
public class IndexPerformanceTester
{
    private readonly ApplicationDbContext _context;

    public async Task<QueryPerformanceResult> TestQueryPerformanceAsync(string testName, Func<IQueryable<Student>, IQueryable<Student>> query)
    {
        var stopwatch = Stopwatch.StartNew();
        
        // Clear plan cache ได้ถ้าต้องการ
        // await _context.Database.ExecuteSqlRawAsync("DBCC FREEPROCCACHE");
        
        var queryable = query(_context.Students);
        var results = await queryable.ToListAsync();
        
        stopwatch.Stop();

        return new QueryPerformanceResult
        {
            TestName = testName,
            ExecutionTime = stopwatch.Elapsed,
            ResultCount = results.Count,
            Query = queryable.ToQueryString()
        };
    }

    public async Task RunPerformanceTests()
    {
        var tests = new[]
        {
            ("Search by LastName", (IQueryable<Student> q) => 
                q.Where(s => s.LastName.StartsWith("Smith"))),
                
            ("Search by Department", (IQueryable<Student> q) => 
                q.Where(s => s.DepartmentId == 1)),
                
            ("Search by Department and Status", (IQueryable<Student> q) => 
                q.Where(s => s.DepartmentId == 1 && s.Status == StudentStatus.Active)),
                
            ("Sort by Name", (IQueryable<Student> q) => 
                q.OrderBy(s => s.LastName).ThenBy(s => s.FirstName)),
                
            ("Complex Query", (IQueryable<Student> q) => 
                q.Where(s => s.DepartmentId == 1 && s.Status == StudentStatus.Active)
                 .OrderBy(s => s.GPA)
                 .Take(10))
        };

        foreach (var (testName, queryFunc) in tests)
        {
            var result = await TestQueryPerformanceAsync(testName, queryFunc);
            Console.WriteLine($"{result.TestName}: {result.ExecutionTime.TotalMilliseconds:F2}ms ({result.ResultCount} rows)");
        }
    }
}

public class QueryPerformanceResult
{
    public string TestName { get; set; }
    public TimeSpan ExecutionTime { get; set; }
    public int ResultCount { get; set; }
    public string Query { get; set; }
}
```

---

**หมายเหตุ**: การออกแบบ Index ที่ดีจะช่วยเพิ่มประสิทธิภาพการค้นหาข้อมูลอย่างมาก แต่ต้องสมดุลกับการใช้พื้นที่และความเร็วในการ Insert/Update/Delete