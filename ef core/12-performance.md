# บทที่ 12: Performance และ Best Practices

## การปรับปรุงประสิทธิภาพ

### 1. N+1 Query Problem

```csharp
// ❌ แย่ - N+1 Query Problem
public async Task<List<StudentWithDepartmentDto>> GetStudentsBadAsync()
{
    var students = await _context.Students.ToListAsync(); // 1 query
    
    var result = new List<StudentWithDepartmentDto>();
    
    foreach (var student in students) // N queries!
    {
        var department = await _context.Departments
            .FirstOrDefaultAsync(d => d.Id == student.DepartmentId); // Query ในลูป!
        
        result.Add(new StudentWithDepartmentDto
        {
            StudentName = $"{student.FirstName} {student.LastName}",
            DepartmentName = department?.Name ?? "ไม่ระบุ"
        });
    }
    
    return result;
}

// ✅ ดี - ใช้ Include
public async Task<List<StudentWithDepartmentDto>> GetStudentsGoodAsync()
{
    return await _context.Students
        .Include(s => s.Department) // Single query with JOIN
        .Select(s => new StudentWithDepartmentDto
        {
            StudentName = $"{s.FirstName} {s.LastName}",
            DepartmentName = s.Department != null ? s.Department.Name : "ไม่ระบุ"
        })
        .ToListAsync();
}

// ✅ ดีมาก - ใช้ Projection โดยตรง
public async Task<List<StudentWithDepartmentDto>> GetStudentsBestAsync()
{
    return await _context.Students
        .Where(s => s.Status == StudentStatus.Active) // Filter ก่อน
        .Select(s => new StudentWithDepartmentDto // Project เฉพาะที่ต้องการ
        {
            StudentName = s.FirstName + " " + s.LastName,
            DepartmentName = s.Department != null ? s.Department.Name : "ไม่ระบุ"
        })
        .ToListAsync();
}
```

### 2. การใช้งาน AsNoTracking

```csharp
public class PerformanceOptimizedService
{
    private readonly ApplicationDbContext _context;

    public PerformanceOptimizedService(ApplicationDbContext context)
    {
        _context = context;
    }

    // ✅ ใช้ AsNoTracking สำหรับ read-only queries
    public async Task<List<Student>> GetStudentsForDisplayAsync()
    {
        return await _context.Students
            .AsNoTracking() // ไม่ track changes
            .Include(s => s.Department)
            .Where(s => s.Status == StudentStatus.Active)
            .OrderBy(s => s.LastName)
            .ToListAsync();
    }

    // ✅ ใช้ AsNoTrackingWithIdentityResolution เมื่อต้องการ identity resolution
    public async Task<List<Student>> GetStudentsWithIdentityResolutionAsync()
    {
        return await _context.Students
            .AsNoTrackingWithIdentityResolution() // Track identity แต่ไม่ track changes
            .Include(s => s.Department)
            .Include(s => s.Enrollments)
                .ThenInclude(e => e.Course)
            .ToListAsync();
    }

    // ✅ ใช้ AsSplitQuery เมื่อมี multiple collections
    public async Task<List<Department>> GetDepartmentsWithDataAsync()
    {
        return await _context.Departments
            .AsSplitQuery() // แยก query เพื่อหลีกเลี่ยง cartesian product
            .Include(d => d.Students)
            .Include(d => d.Courses)
            .Include(d => d.Instructors)
            .AsNoTracking()
            .ToListAsync();
    }
}
```

### 3. Query Performance Optimization

```csharp
public class QueryOptimizationService
{
    private readonly ApplicationDbContext _context;

    public QueryOptimizationService(ApplicationDbContext context)
    {
        _context = context;
    }

    // ✅ Filter ก่อน Include
    public async Task<List<Student>> GetActiveStudentsWithDepartmentAsync()
    {
        return await _context.Students
            .Where(s => s.Status == StudentStatus.Active) // Filter ก่อน
            .Include(s => s.Department)
            .AsNoTracking()
            .ToListAsync();
    }

    // ✅ ใช้ Projection แทน Include เมื่อต้องการข้อมูลบางส่วน
    public async Task<List<StudentSummaryDto>> GetStudentSummariesAsync()
    {
        return await _context.Students
            .Where(s => s.Status == StudentStatus.Active)
            .Select(s => new StudentSummaryDto
            {
                Id = s.Id,
                Name = s.FirstName + " " + s.LastName,
                Email = s.Email,
                DepartmentName = s.Department != null ? s.Department.Name : null,
                EnrollmentCount = s.Enrollments.Count(e => e.Status == EnrollmentStatus.Active)
            })
            .ToListAsync();
    }

    // ✅ ใช้ Any() แทน Count() > 0
    public async Task<bool> HasActiveEnrollmentsAsync(int studentId)
    {
        return await _context.Enrollments
            .AnyAsync(e => e.StudentId == studentId && e.Status == EnrollmentStatus.Active);
    }

    // ✅ ใช้ FirstOrDefault แทน Where().FirstOrDefault()
    public async Task<Student?> GetStudentByNumberAsync(string studentNumber)
    {
        return await _context.Students
            .AsNoTracking()
            .FirstOrDefaultAsync(s => s.StudentNumber == studentNumber);
    }

    // ✅ Batch operations แทน single operations ในลูป
    public async Task UpdateStudentStatusesBatchAsync(List<int> studentIds, StudentStatus status)
    {
        await _context.Students
            .Where(s => studentIds.Contains(s.Id))
            .ExecuteUpdateAsync(s => s.SetProperty(x => x.Status, status));
    }

    // ✅ Pagination สำหรับ large datasets
    public async Task<PagedResult<Student>> GetStudentsPagedAsync(int page, int pageSize)
    {
        var totalCount = await _context.Students
            .CountAsync(s => s.Status == StudentStatus.Active);

        var students = await _context.Students
            .Where(s => s.Status == StudentStatus.Active)
            .OrderBy(s => s.Id) // Consistent ordering
            .Skip((page - 1) * pageSize)
            .Take(pageSize)
            .AsNoTracking()
            .Select(s => new Student
            {
                Id = s.Id,
                FirstName = s.FirstName,
                LastName = s.LastName,
                Email = s.Email,
                StudentNumber = s.StudentNumber
            })
            .ToListAsync();

        return new PagedResult<Student>
        {
            Items = students,
            TotalCount = totalCount,
            PageNumber = page,
            PageSize = pageSize
        };
    }
}
```

## การจัดการ DbContext Lifetime

### 1. DbContext Scoping

```csharp
// ✅ ถูกต้อง - ใช้ Dependency Injection
public class StudentController : ControllerBase
{
    private readonly ApplicationDbContext _context;

    public StudentController(ApplicationDbContext context)
    {
        _context = context; // DI จะจัดการ lifetime
    }

    [HttpGet]
    public async Task<IActionResult> GetStudents()
    {
        var students = await _context.Students
            .AsNoTracking()
            .ToListAsync();
        
        return Ok(students);
    } // DbContext จะถูก dispose อัตโนมัติ
}

// ❌ ผิด - Manual DbContext management ใน Controller
public class BadStudentController : ControllerBase
{
    [HttpGet]
    public async Task<IActionResult> GetStudents()
    {
        using var context = new ApplicationDbContext(); // ไม่ควรทำแบบนี้
        var students = await context.Students.ToListAsync();
        return Ok(students);
    }
}

// ✅ ถูกต้อง - สำหรับ Background Services
public class StudentReportService : BackgroundService
{
    private readonly IServiceProvider _serviceProvider;
    private readonly ILogger<StudentReportService> _logger;

    public StudentReportService(IServiceProvider serviceProvider, ILogger<StudentReportService> logger)
    {
        _serviceProvider = serviceProvider;
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            using var scope = _serviceProvider.CreateScope(); // Create scope
            var context = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
            
            await GenerateReportsAsync(context);
            
            await Task.Delay(TimeSpan.FromHours(1), stoppingToken);
        } // Scope จะ dispose DbContext อัตโนมัติ
    }

    private async Task GenerateReportsAsync(ApplicationDbContext context)
    {
        // ทำงานกับ context
    }
}
```

### 2. DbContextFactory Pattern

```csharp
// สำหรับ scenarios ที่ต้องการ multiple DbContext instances
public class ReportGenerationService
{
    private readonly IDbContextFactory<ApplicationDbContext> _contextFactory;

    public ReportGenerationService(IDbContextFactory<ApplicationDbContext> contextFactory)
    {
        _contextFactory = contextFactory;
    }

    public async Task GenerateMonthlyReportAsync()
    {
        using var context = _contextFactory.CreateDbContext();
        
        // ใช้ context สำหรับ report generation
        var data = await context.Students
            .AsNoTracking()
            .Include(s => s.Department)
            .ToListAsync();
            
        // Process report...
    } // DbContext จะถูก dispose

    // สำหรับ parallel processing
    public async Task ProcessStudentsParallelAsync()
    {
        var studentIds = await GetStudentIdsAsync();
        
        await Parallel.ForEachAsync(studentIds, async (studentId, ct) =>
        {
            using var context = _contextFactory.CreateDbContext(); // แต่ละ thread มี context ของตัวเอง
            await ProcessStudentAsync(context, studentId);
        });
    }

    private async Task<List<int>> GetStudentIdsAsync()
    {
        using var context = _contextFactory.CreateDbContext();
        return await context.Students
            .Select(s => s.Id)
            .ToListAsync();
    }

    private async Task ProcessStudentAsync(ApplicationDbContext context, int studentId)
    {
        var student = await context.Students.FindAsync(studentId);
        // Process student...
    }
}

// Registration
builder.Services.AddDbContextFactory<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString));
```

## Caching Strategies

### 1. Memory Caching

```csharp
public class CachedDepartmentService
{
    private readonly ApplicationDbContext _context;
    private readonly IMemoryCache _cache;
    private readonly ILogger<CachedDepartmentService> _logger;
    private static readonly TimeSpan CacheDuration = TimeSpan.FromMinutes(30);

    public CachedDepartmentService(
        ApplicationDbContext context, 
        IMemoryCache cache,
        ILogger<CachedDepartmentService> logger)
    {
        _context = context;
        _cache = cache;
        _logger = logger;
    }

    public async Task<List<Department>> GetDepartmentsAsync()
    {
        const string cacheKey = "all_departments";

        if (_cache.TryGetValue(cacheKey, out List<Department>? cachedDepartments))
        {
            _logger.LogInformation("Returning departments from cache");
            return cachedDepartments!;
        }

        _logger.LogInformation("Loading departments from database");
        var departments = await _context.Departments
            .AsNoTracking()
            .OrderBy(d => d.Name)
            .ToListAsync();

        _cache.Set(cacheKey, departments, new MemoryCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = CacheDuration,
            SlidingExpiration = TimeSpan.FromMinutes(5)
        });

        return departments;
    }

    public async Task<Department?> GetDepartmentByIdAsync(int id)
    {
        var cacheKey = $"department_{id}";

        if (_cache.TryGetValue(cacheKey, out Department? cachedDepartment))
        {
            return cachedDepartment;
        }

        var department = await _context.Departments
            .AsNoTracking()
            .FirstOrDefaultAsync(d => d.Id == id);

        if (department != null)
        {
            _cache.Set(cacheKey, department, CacheDuration);
        }

        return department;
    }

    public async Task InvalidateDepartmentCacheAsync()
    {
        _cache.Remove("all_departments");
        // Remove individual department caches if needed
    }
}
```

### 2. Distributed Caching

```csharp
public class DistributedCachedService
{
    private readonly ApplicationDbContext _context;
    private readonly IDistributedCache _distributedCache;
    private readonly ILogger<DistributedCachedService> _logger;

    public DistributedCachedService(
        ApplicationDbContext context,
        IDistributedCache distributedCache,
        ILogger<DistributedCachedService> logger)
    {
        _context = context;
        _distributedCache = distributedCache;
        _logger = logger;
    }

    public async Task<List<StudentSummaryDto>> GetStudentSummariesAsync(int departmentId)
    {
        var cacheKey = $"student_summaries_{departmentId}";
        var cachedData = await _distributedCache.GetStringAsync(cacheKey);

        if (cachedData != null)
        {
            _logger.LogInformation("Returning data from distributed cache");
            return JsonSerializer.Deserialize<List<StudentSummaryDto>>(cachedData)!;
        }

        _logger.LogInformation("Loading data from database");
        var students = await _context.Students
            .Where(s => s.DepartmentId == departmentId)
            .AsNoTracking()
            .Select(s => new StudentSummaryDto
            {
                Id = s.Id,
                Name = s.FirstName + " " + s.LastName,
                Email = s.Email,
                DepartmentName = s.Department!.Name
            })
            .ToListAsync();

        var serializedData = JsonSerializer.Serialize(students);
        var cacheOptions = new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(15),
            SlidingExpiration = TimeSpan.FromMinutes(5)
        };

        await _distributedCache.SetStringAsync(cacheKey, serializedData, cacheOptions);
        return students;
    }
}
```

## Connection Management

### 1. Connection Pooling

```csharp
// Program.cs
builder.Services.AddDbContextPool<ApplicationDbContext>(options =>
{
    options.UseSqlServer(connectionString, sqlOptions =>
    {
        // Connection timeout
        sqlOptions.CommandTimeout(30);
        
        // Connection resilience
        sqlOptions.EnableRetryOnFailure(
            maxRetryCount: 5,
            maxRetryDelay: TimeSpan.FromSeconds(10),
            errorNumbersToAdd: null);
    });
    
    // Pooling configuration
}, poolSize: 128); // Pool size

// สำหรับ high-traffic applications
public class ConnectionManagementService
{
    private readonly IDbContextFactory<ApplicationDbContext> _contextFactory;

    public ConnectionManagementService(IDbContextFactory<ApplicationDbContext> contextFactory)
    {
        _contextFactory = contextFactory;
    }

    // ใช้ multiple contexts เพื่อ parallel processing
    public async Task<List<ReportData>> GenerateReportsParallelAsync()
    {
        var tasks = new List<Task<ReportData>>();

        for (int i = 0; i < 10; i++)
        {
            tasks.Add(GenerateReportAsync(i));
        }

        return (await Task.WhenAll(tasks)).ToList();
    }

    private async Task<ReportData> GenerateReportAsync(int reportId)
    {
        using var context = _contextFactory.CreateDbContext();
        
        // Each task gets its own DbContext
        var data = await context.Students
            .AsNoTracking()
            .Where(s => s.Id % 10 == reportId)
            .ToListAsync();

        return new ReportData { Id = reportId, Data = data };
    }
}
```

### 2. Read/Write Separation

```csharp
// Separate contexts for read and write operations
public interface IReadOnlyDbContext
{
    IQueryable<Student> Students { get; }
    IQueryable<Course> Courses { get; }
    IQueryable<Department> Departments { get; }
}

public interface IWriteDbContext
{
    DbSet<Student> Students { get; }
    DbSet<Course> Courses { get; }
    DbSet<Department> Departments { get; }
    Task<int> SaveChangesAsync(CancellationToken cancellationToken = default);
}

public class ReadOnlyApplicationDbContext : DbContext, IReadOnlyDbContext
{
    public ReadOnlyApplicationDbContext(DbContextOptions<ReadOnlyApplicationDbContext> options)
        : base(options) { }

    public IQueryable<Student> Students => Set<Student>().AsNoTracking();
    public IQueryable<Course> Courses => Set<Course>().AsNoTracking();
    public IQueryable<Department> Departments => Set<Department>().AsNoTracking();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configure entities
        base.OnModelCreating(modelBuilder);
    }
}

public class WriteApplicationDbContext : DbContext, IWriteDbContext
{
    public WriteApplicationDbContext(DbContextOptions<WriteApplicationDbContext> options)
        : base(options) { }

    public DbSet<Student> Students { get; set; }
    public DbSet<Course> Courses { get; set; }
    public DbSet<Department> Departments { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configure entities
        base.OnModelCreating(modelBuilder);
    }
}

// Service using separated contexts
public class StudentService
{
    private readonly IReadOnlyDbContext _readContext;
    private readonly IWriteDbContext _writeContext;

    public StudentService(IReadOnlyDbContext readContext, IWriteDbContext writeContext)
    {
        _readContext = readContext;
        _writeContext = writeContext;
    }

    // Read operations use read-only context
    public async Task<List<Student>> GetStudentsAsync()
    {
        return await _readContext.Students.ToListAsync();
    }

    // Write operations use write context
    public async Task<Student> CreateStudentAsync(Student student)
    {
        _writeContext.Students.Add(student);
        await _writeContext.SaveChangesAsync();
        return student;
    }
}

// Registration
builder.Services.AddDbContext<ReadOnlyApplicationDbContext>(options =>
    options.UseSqlServer(readOnlyConnectionString));
    
builder.Services.AddDbContext<WriteApplicationDbContext>(options =>
    options.UseSqlServer(readWriteConnectionString));

builder.Services.AddScoped<IReadOnlyDbContext>(provider =>
    provider.GetRequiredService<ReadOnlyApplicationDbContext>());
    
builder.Services.AddScoped<IWriteDbContext>(provider =>
    provider.GetRequiredService<WriteApplicationDbContext>());
```

## Best Practices Summary

### 1. Query Best Practices

```csharp
public static class QueryBestPractices
{
    // ✅ DO: Use AsNoTracking for read-only queries
    public static IQueryable<T> ReadOnly<T>(this DbSet<T> dbSet) where T : class
    {
        return dbSet.AsNoTracking();
    }

    // ✅ DO: Use projections to select only needed data
    public static async Task<List<TResult>> SelectAsync<T, TResult>(
        this IQueryable<T> query,
        Expression<Func<T, TResult>> selector)
    {
        return await query.Select(selector).ToListAsync();
    }

    // ✅ DO: Filter before including related data
    public static IQueryable<Student> WithActiveStatus(this IQueryable<Student> query)
    {
        return query.Where(s => s.Status == StudentStatus.Active);
    }

    // ✅ DO: Use Any() instead of Count() > 0
    public static async Task<bool> HasAnyAsync<T>(this IQueryable<T> query)
    {
        return await query.AnyAsync();
    }
}

// Usage
public class OptimizedStudentService
{
    private readonly ApplicationDbContext _context;

    public OptimizedStudentService(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<List<StudentDto>> GetActiveStudentsAsync()
    {
        return await _context.Students
            .ReadOnly()
            .WithActiveStatus()
            .SelectAsync(s => new StudentDto
            {
                Id = s.Id,
                Name = s.FirstName + " " + s.LastName,
                Email = s.Email
            });
    }
}
```

### 2. Entity Configuration Best Practices

```csharp
// ✅ DO: Separate configuration into different classes
public static class EntityConfigurationExtensions
{
    public static void ConfigureStudentEntity(this ModelBuilder modelBuilder)
    {
        var entity = modelBuilder.Entity<Student>();
        
        entity.ToTable("Students", "Academic");
        entity.HasKey(s => s.Id);
        
        entity.Property(s => s.StudentNumber)
            .IsRequired()
            .HasMaxLength(20);
            
        entity.HasIndex(s => s.StudentNumber)
            .IsUnique();
            
        // Configure relationships
        entity.HasOne(s => s.Department)
            .WithMany(d => d.Students)
            .HasForeignKey(s => s.DepartmentId)
            .OnDelete(DeleteBehavior.SetNull);
    }
}

// ✅ DO: Use meaningful names for constraints
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Student>()
        .HasIndex(s => s.Email)
        .IsUnique()
        .HasDatabaseName("IX_Student_Email_Unique");
        
    modelBuilder.Entity<Enrollment>()
        .HasKey(e => new { e.StudentId, e.CourseId })
        .HasName("PK_Enrollment_Student_Course");
}
```

### 3. Error Handling Best Practices

```csharp
public class RobustStudentService
{
    private readonly ApplicationDbContext _context;
    private readonly ILogger<RobustStudentService> _logger;

    public RobustStudentService(ApplicationDbContext context, ILogger<RobustStudentService> logger)
    {
        _context = context;
        _logger = logger;
    }

    public async Task<Result<Student>> CreateStudentAsync(Student student)
    {
        try
        {
            // Validate business rules
            if (await _context.Students.AnyAsync(s => s.Email == student.Email))
            {
                return Result<Student>.Failure("Email already exists");
            }

            _context.Students.Add(student);
            await _context.SaveChangesAsync();

            _logger.LogInformation("Student created successfully with ID: {StudentId}", student.Id);
            return Result<Student>.Success(student);
        }
        catch (DbUpdateException ex) when (ex.InnerException?.Message.Contains("UNIQUE constraint") == true)
        {
            _logger.LogWarning(ex, "Unique constraint violation when creating student");
            return Result<Student>.Failure("Student with this information already exists");
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error creating student: {@Student}", student);
            return Result<Student>.Failure("An error occurred while creating the student");
        }
    }
}

public class Result<T>
{
    public bool IsSuccess { get; init; }
    public T? Value { get; init; }
    public string? Error { get; init; }

    public static Result<T> Success(T value) => new() { IsSuccess = true, Value = value };
    public static Result<T> Failure(string error) => new() { IsSuccess = false, Error = error };
}
```

---

**สรุป Best Practices:**

1. **ใช้ AsNoTracking** สำหรับ read-only queries
2. **หลีกเลี่ยง N+1 Problem** ด้วย Include หรือ Projection
3. **ใช้ Pagination** สำหรับ large datasets  
4. **Cache ข้อมูลที่ไม่เปลี่ยนแปลงบ่อย**
5. **จัดการ DbContext Lifetime** อย่างถูกต้อง
6. **ใช้ Connection Pooling** เพื่อประสิทธิภาพ
7. **Monitor และ Profile** queries เป็นประจำ
8. **Error Handling** ที่เหมาะสม
9. **Separate Concerns** ด้วย Repository Pattern หรือ Service Layer
10. **Test Performance** ด้วยข้อมูลจริงในขนาดใกล้เคียง Production