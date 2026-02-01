# บทที่ 11: Query Operations

## พื้นฐานการ Query ด้วย LINQ

### 1. Basic Queries

```csharp
public class StudentQueryService
{
    private readonly ApplicationDbContext _context;

    public StudentQueryService(ApplicationDbContext context)
    {
        _context = context;
    }

    // ดึงข้อมูลทั้งหมด
    public async Task<List<Student>> GetAllStudentsAsync()
    {
        return await _context.Students.ToListAsync();
    }

    // ดึงข้อมูลตาม ID
    public async Task<Student?> GetStudentByIdAsync(int id)
    {
        return await _context.Students
            .FirstOrDefaultAsync(s => s.Id == id);
    }

    // ดึงข้อมูลหลาย records ตามเงื่อนไข
    public async Task<List<Student>> GetStudentsByDepartmentAsync(int departmentId)
    {
        return await _context.Students
            .Where(s => s.DepartmentId == departmentId)
            .ToListAsync();
    }

    // Count records
    public async Task<int> GetStudentCountAsync()
    {
        return await _context.Students.CountAsync();
    }

    // Check existence
    public async Task<bool> StudentExistsAsync(string studentNumber)
    {
        return await _context.Students
            .AnyAsync(s => s.StudentNumber == studentNumber);
    }
}
```

### 2. Filtering และ Searching

```csharp
public class StudentSearchService
{
    private readonly ApplicationDbContext _context;

    public StudentSearchService(ApplicationDbContext context)
    {
        _context = context;
    }

    // การค้นหาแบบ Contains
    public async Task<List<Student>> SearchByNameAsync(string searchTerm)
    {
        return await _context.Students
            .Where(s => s.FirstName.Contains(searchTerm) || 
                       s.LastName.Contains(searchTerm))
            .ToListAsync();
    }

    // การค้นหาแบบ StartsWith
    public async Task<List<Student>> GetStudentsByLastNamePrefixAsync(string prefix)
    {
        return await _context.Students
            .Where(s => s.LastName.StartsWith(prefix))
            .OrderBy(s => s.LastName)
            .ThenBy(s => s.FirstName)
            .ToListAsync();
    }

    // การค้นหาแบบ Multiple conditions
    public async Task<List<Student>> SearchStudentsAsync(StudentSearchCriteria criteria)
    {
        var query = _context.Students.AsQueryable();

        if (!string.IsNullOrEmpty(criteria.Name))
        {
            query = query.Where(s => s.FirstName.Contains(criteria.Name) || 
                                    s.LastName.Contains(criteria.Name));
        }

        if (criteria.DepartmentId.HasValue)
        {
            query = query.Where(s => s.DepartmentId == criteria.DepartmentId.Value);
        }

        if (criteria.Status.HasValue)
        {
            query = query.Where(s => s.Status == criteria.Status.Value);
        }

        if (criteria.MinGPA.HasValue)
        {
            query = query.Where(s => s.GPA >= criteria.MinGPA.Value);
        }

        if (criteria.MaxGPA.HasValue)
        {
            query = query.Where(s => s.GPA <= criteria.MaxGPA.Value);
        }

        if (criteria.EnrollmentDateFrom.HasValue)
        {
            query = query.Where(s => s.EnrollmentDate >= criteria.EnrollmentDateFrom.Value);
        }

        if (criteria.EnrollmentDateTo.HasValue)
        {
            query = query.Where(s => s.EnrollmentDate <= criteria.EnrollmentDateTo.Value);
        }

        return await query
            .OrderBy(s => s.LastName)
            .ThenBy(s => s.FirstName)
            .ToListAsync();
    }

    // Case-insensitive search
    public async Task<List<Student>> SearchByEmailAsync(string email)
    {
        return await _context.Students
            .Where(s => EF.Functions.Like(s.Email.ToLower(), $"%{email.ToLower()}%"))
            .ToListAsync();
    }
}

public class StudentSearchCriteria
{
    public string? Name { get; set; }
    public int? DepartmentId { get; set; }
    public StudentStatus? Status { get; set; }
    public decimal? MinGPA { get; set; }
    public decimal? MaxGPA { get; set; }
    public DateTime? EnrollmentDateFrom { get; set; }
    public DateTime? EnrollmentDateTo { get; set; }
}
```

### 3. Sorting และ Ordering

```csharp
public class StudentSortingService
{
    private readonly ApplicationDbContext _context;

    public StudentSortingService(ApplicationDbContext context)
    {
        _context = context;
    }

    // Single column sorting
    public async Task<List<Student>> GetStudentsSortedByNameAsync()
    {
        return await _context.Students
            .OrderBy(s => s.LastName)
            .ThenBy(s => s.FirstName)
            .ToListAsync();
    }

    // Descending sort
    public async Task<List<Student>> GetStudentsByGPADescendingAsync()
    {
        return await _context.Students
            .Where(s => s.GPA.HasValue)
            .OrderByDescending(s => s.GPA)
            .ThenBy(s => s.LastName)
            .ToListAsync();
    }

    // Dynamic sorting
    public async Task<List<Student>> GetSortedStudentsAsync(string sortBy, bool descending = false)
    {
        var query = _context.Students.AsQueryable();

        query = sortBy.ToLower() switch
        {
            "firstname" => descending ? query.OrderByDescending(s => s.FirstName) : query.OrderBy(s => s.FirstName),
            "lastname" => descending ? query.OrderByDescending(s => s.LastName) : query.OrderBy(s => s.LastName),
            "email" => descending ? query.OrderByDescending(s => s.Email) : query.OrderBy(s => s.Email),
            "gpa" => descending ? query.OrderByDescending(s => s.GPA) : query.OrderBy(s => s.GPA),
            "enrollmentdate" => descending ? query.OrderByDescending(s => s.EnrollmentDate) : query.OrderBy(s => s.EnrollmentDate),
            _ => query.OrderBy(s => s.LastName).ThenBy(s => s.FirstName)
        };

        return await query.ToListAsync();
    }

    // Complex sorting with multiple criteria
    public async Task<List<Student>> GetStudentsWithComplexSortingAsync()
    {
        return await _context.Students
            .OrderBy(s => s.Department!.Name) // Department name first
            .ThenByDescending(s => s.GPA ?? 0) // Then by GPA (nulls treated as 0)
            .ThenBy(s => s.LastName)
            .ThenBy(s => s.FirstName)
            .ToListAsync();
    }
}
```

## การ Join และ Include

### 1. Eager Loading ด้วย Include

```csharp
public class StudentIncludeService
{
    private readonly ApplicationDbContext _context;

    public StudentIncludeService(ApplicationDbContext context)
    {
        _context = context;
    }

    // Single include
    public async Task<List<Student>> GetStudentsWithDepartmentAsync()
    {
        return await _context.Students
            .Include(s => s.Department)
            .ToListAsync();
    }

    // Multiple includes
    public async Task<List<Student>> GetStudentsWithAllRelatedDataAsync()
    {
        return await _context.Students
            .Include(s => s.Department)
            .Include(s => s.Profile)
            .Include(s => s.Enrollments)
            .ToListAsync();
    }

    // Nested includes
    public async Task<Student?> GetStudentWithCoursesAsync(int studentId)
    {
        return await _context.Students
            .Include(s => s.Department)
            .Include(s => s.Profile)
            .Include(s => s.Enrollments)
                .ThenInclude(e => e.Course)
                    .ThenInclude(c => c.Department)
            .Include(s => s.Enrollments)
                .ThenInclude(e => e.Course)
                    .ThenInclude(c => c.Instructor)
            .FirstOrDefaultAsync(s => s.Id == studentId);
    }

    // Filtered include
    public async Task<List<Student>> GetStudentsWithActiveEnrollmentsAsync()
    {
        return await _context.Students
            .Include(s => s.Enrollments.Where(e => e.Status == EnrollmentStatus.Active))
                .ThenInclude(e => e.Course)
            .ToListAsync();
    }

    // Split query เพื่อหลีกเลี่ยง Cartesian explosion
    public async Task<List<Student>> GetStudentsWithSplitQueryAsync()
    {
        return await _context.Students
            .AsSplitQuery()
            .Include(s => s.Department)
            .Include(s => s.Enrollments)
                .ThenInclude(e => e.Course)
            .Include(s => s.Profile)
            .ToListAsync();
    }
}
```

### 2. Explicit Loading

```csharp
public class ExplicitLoadingService
{
    private readonly ApplicationDbContext _context;

    public ExplicitLoadingService(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<Student> LoadStudentDataExplicitlyAsync(int studentId)
    {
        var student = await _context.Students
            .FirstAsync(s => s.Id == studentId);

        // Load related collection
        await _context.Entry(student)
            .Collection(s => s.Enrollments)
            .LoadAsync();

        // Load related reference
        await _context.Entry(student)
            .Reference(s => s.Department)
            .LoadAsync();

        // Load with query
        await _context.Entry(student)
            .Collection(s => s.Enrollments)
            .Query()
            .Where(e => e.Status == EnrollmentStatus.Active)
            .LoadAsync();

        return student;
    }

    // Load count without loading all data
    public async Task<int> GetStudentEnrollmentCountAsync(int studentId)
    {
        var student = await _context.Students
            .FirstAsync(s => s.Id == studentId);

        return await _context.Entry(student)
            .Collection(s => s.Enrollments)
            .Query()
            .CountAsync();
    }
}
```

### 3. Manual Joins

```csharp
public class ManualJoinService
{
    private readonly ApplicationDbContext _context;

    public ManualJoinService(ApplicationDbContext context)
    {
        _context = context;
    }

    // Inner join
    public async Task<List<StudentDepartmentDto>> GetStudentsWithDepartmentNamesAsync()
    {
        return await _context.Students
            .Join(_context.Departments,
                student => student.DepartmentId,
                department => department.Id,
                (student, department) => new StudentDepartmentDto
                {
                    StudentId = student.Id,
                    StudentName = student.FirstName + " " + student.LastName,
                    StudentNumber = student.StudentNumber,
                    DepartmentName = department.Name,
                    DepartmentCode = department.Code
                })
            .ToListAsync();
    }

    // Left join
    public async Task<List<StudentDepartmentDto>> GetAllStudentsWithOptionalDepartmentAsync()
    {
        return await _context.Students
            .GroupJoin(_context.Departments,
                student => student.DepartmentId,
                department => department.Id,
                (student, departments) => new { student, departments })
            .SelectMany(
                x => x.departments.DefaultIfEmpty(),
                (x, department) => new StudentDepartmentDto
                {
                    StudentId = x.student.Id,
                    StudentName = x.student.FirstName + " " + x.student.LastName,
                    StudentNumber = x.student.StudentNumber,
                    DepartmentName = department != null ? department.Name : "ไม่ระบุ",
                    DepartmentCode = department != null ? department.Code : ""
                })
            .ToListAsync();
    }

    // Complex join with multiple tables
    public async Task<List<EnrollmentReportDto>> GetEnrollmentReportAsync()
    {
        return await _context.Enrollments
            .Join(_context.Students,
                enrollment => enrollment.StudentId,
                student => student.Id,
                (enrollment, student) => new { enrollment, student })
            .Join(_context.Courses,
                x => x.enrollment.CourseId,
                course => course.Id,
                (x, course) => new { x.enrollment, x.student, course })
            .Join(_context.Departments,
                x => x.course.DepartmentId,
                department => department.Id,
                (x, department) => new EnrollmentReportDto
                {
                    StudentName = x.student.FirstName + " " + x.student.LastName,
                    StudentNumber = x.student.StudentNumber,
                    CourseName = x.course.Title,
                    CourseCode = x.course.CourseCode,
                    DepartmentName = department.Name,
                    EnrollmentDate = x.enrollment.EnrollmentDate,
                    Grade = x.enrollment.Grade,
                    Status = x.enrollment.Status
                })
            .ToListAsync();
    }
}

public class StudentDepartmentDto
{
    public int StudentId { get; set; }
    public string StudentName { get; set; }
    public string StudentNumber { get; set; }
    public string DepartmentName { get; set; }
    public string DepartmentCode { get; set; }
}

public class EnrollmentReportDto
{
    public string StudentName { get; set; }
    public string StudentNumber { get; set; }
    public string CourseName { get; set; }
    public string CourseCode { get; set; }
    public string DepartmentName { get; set; }
    public DateTime EnrollmentDate { get; set; }
    public string? Grade { get; set; }
    public EnrollmentStatus Status { get; set; }
}
```

## การใช้งาน Aggregation

### 1. Basic Aggregations

```csharp
public class AggregationService
{
    private readonly ApplicationDbContext _context;

    public AggregationService(ApplicationDbContext context)
    {
        _context = context;
    }

    // Count
    public async Task<int> GetTotalStudentsAsync()
    {
        return await _context.Students.CountAsync();
    }

    public async Task<int> GetActiveStudentsCountAsync()
    {
        return await _context.Students
            .CountAsync(s => s.Status == StudentStatus.Active);
    }

    // Sum
    public async Task<decimal> GetTotalCreditsForStudentAsync(int studentId)
    {
        return await _context.Enrollments
            .Where(e => e.StudentId == studentId && e.Status == EnrollmentStatus.Active)
            .SumAsync(e => e.Course.Credits);
    }

    // Average
    public async Task<decimal> GetAverageGPAAsync()
    {
        return await _context.Students
            .Where(s => s.GPA.HasValue)
            .AverageAsync(s => s.GPA!.Value);
    }

    public async Task<decimal> GetAverageGPAByDepartmentAsync(int departmentId)
    {
        return await _context.Students
            .Where(s => s.DepartmentId == departmentId && s.GPA.HasValue)
            .AverageAsync(s => s.GPA!.Value);
    }

    // Min/Max
    public async Task<decimal> GetHighestGPAAsync()
    {
        return await _context.Students
            .Where(s => s.GPA.HasValue)
            .MaxAsync(s => s.GPA!.Value);
    }

    public async Task<decimal> GetLowestGPAAsync()
    {
        return await _context.Students
            .Where(s => s.GPA.HasValue)
            .MinAsync(s => s.GPA!.Value);
    }
}
```

### 2. Group By Operations

```csharp
public class GroupByService
{
    private readonly ApplicationDbContext _context;

    public GroupByService(ApplicationDbContext context)
    {
        _context = context;
    }

    // Simple group by
    public async Task<List<DepartmentStatistics>> GetStudentCountByDepartmentAsync()
    {
        return await _context.Students
            .Where(s => s.DepartmentId.HasValue)
            .GroupBy(s => s.Department!)
            .Select(g => new DepartmentStatistics
            {
                DepartmentName = g.Key.Name,
                DepartmentCode = g.Key.Code,
                StudentCount = g.Count(),
                AverageGPA = g.Where(s => s.GPA.HasValue).Average(s => s.GPA!.Value),
                HighestGPA = g.Where(s => s.GPA.HasValue).Max(s => s.GPA!.Value),
                LowestGPA = g.Where(s => s.GPA.HasValue).Min(s => s.GPA!.Value)
            })
            .OrderBy(d => d.DepartmentName)
            .ToListAsync();
    }

    // Multiple group by
    public async Task<List<EnrollmentStatistics>> GetEnrollmentStatisticsAsync()
    {
        return await _context.Enrollments
            .GroupBy(e => new { e.Course.DepartmentId, e.Course.Department!.Name, e.Status })
            .Select(g => new EnrollmentStatistics
            {
                DepartmentId = g.Key.DepartmentId,
                DepartmentName = g.Key.Name,
                Status = g.Key.Status,
                Count = g.Count(),
                AverageScore = g.Where(e => e.Score.HasValue).Average(e => e.Score!.Value) ?? 0
            })
            .OrderBy(e => e.DepartmentName)
            .ThenBy(e => e.Status)
            .ToListAsync();
    }

    // Group by with having clause
    public async Task<List<PopularCoursesDto>> GetPopularCoursesAsync(int minEnrollments = 10)
    {
        return await _context.Enrollments
            .GroupBy(e => e.Course)
            .Where(g => g.Count() >= minEnrollments)
            .Select(g => new PopularCoursesDto
            {
                CourseId = g.Key.Id,
                CourseCode = g.Key.CourseCode,
                CourseName = g.Key.Title,
                EnrollmentCount = g.Count(),
                AverageGrade = g.Where(e => e.Score.HasValue).Average(e => e.Score!.Value) ?? 0,
                PassRate = g.Count(e => e.Grade != "F") * 100.0 / g.Count()
            })
            .OrderByDescending(c => c.EnrollmentCount)
            .ToListAsync();
    }

    // Time-based grouping
    public async Task<List<MonthlyEnrollmentReport>> GetMonthlyEnrollmentReportAsync(int year)
    {
        return await _context.Enrollments
            .Where(e => e.EnrollmentDate.Year == year)
            .GroupBy(e => e.EnrollmentDate.Month)
            .Select(g => new MonthlyEnrollmentReport
            {
                Month = g.Key,
                Year = year,
                EnrollmentCount = g.Count(),
                UniqueStudents = g.Select(e => e.StudentId).Distinct().Count(),
                UniqueCourses = g.Select(e => e.CourseId).Distinct().Count()
            })
            .OrderBy(r => r.Month)
            .ToListAsync();
    }
}

public class DepartmentStatistics
{
    public string DepartmentName { get; set; }
    public string DepartmentCode { get; set; }
    public int StudentCount { get; set; }
    public decimal AverageGPA { get; set; }
    public decimal HighestGPA { get; set; }
    public decimal LowestGPA { get; set; }
}

public class EnrollmentStatistics
{
    public int DepartmentId { get; set; }
    public string DepartmentName { get; set; }
    public EnrollmentStatus Status { get; set; }
    public int Count { get; set; }
    public decimal AverageScore { get; set; }
}

public class PopularCoursesDto
{
    public int CourseId { get; set; }
    public string CourseCode { get; set; }
    public string CourseName { get; set; }
    public int EnrollmentCount { get; set; }
    public decimal AverageGrade { get; set; }
    public double PassRate { get; set; }
}

public class MonthlyEnrollmentReport
{
    public int Month { get; set; }
    public int Year { get; set; }
    public int EnrollmentCount { get; set; }
    public int UniqueStudents { get; set; }
    public int UniqueCourses { get; set; }
}
```

## การใช้งาน Pagination

### 1. Simple Pagination

```csharp
public class PaginationService
{
    private readonly ApplicationDbContext _context;

    public PaginationService(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<PagedResult<Student>> GetStudentsPagedAsync(
        int pageNumber = 1, 
        int pageSize = 20)
    {
        var totalCount = await _context.Students.CountAsync();
        
        var students = await _context.Students
            .Include(s => s.Department)
            .OrderBy(s => s.LastName)
            .ThenBy(s => s.FirstName)
            .Skip((pageNumber - 1) * pageSize)
            .Take(pageSize)
            .ToListAsync();

        return new PagedResult<Student>
        {
            Items = students,
            TotalCount = totalCount,
            PageNumber = pageNumber,
            PageSize = pageSize,
            TotalPages = (int)Math.Ceiling((double)totalCount / pageSize)
        };
    }

    // Pagination with search
    public async Task<PagedResult<Student>> SearchStudentsPagedAsync(
        StudentSearchCriteria criteria,
        int pageNumber = 1,
        int pageSize = 20)
    {
        var query = BuildSearchQuery(criteria);

        var totalCount = await query.CountAsync();

        var students = await query
            .OrderBy(s => s.LastName)
            .ThenBy(s => s.FirstName)
            .Skip((pageNumber - 1) * pageSize)
            .Take(pageSize)
            .Include(s => s.Department)
            .ToListAsync();

        return new PagedResult<Student>
        {
            Items = students,
            TotalCount = totalCount,
            PageNumber = pageNumber,
            PageSize = pageSize,
            TotalPages = (int)Math.Ceiling((double)totalCount / pageSize),
            HasNextPage = pageNumber * pageSize < totalCount,
            HasPreviousPage = pageNumber > 1
        };
    }

    private IQueryable<Student> BuildSearchQuery(StudentSearchCriteria criteria)
    {
        var query = _context.Students.AsQueryable();

        if (!string.IsNullOrEmpty(criteria.Name))
        {
            query = query.Where(s => s.FirstName.Contains(criteria.Name) || 
                                    s.LastName.Contains(criteria.Name));
        }

        if (criteria.DepartmentId.HasValue)
        {
            query = query.Where(s => s.DepartmentId == criteria.DepartmentId.Value);
        }

        // ... other criteria

        return query;
    }
}

public class PagedResult<T>
{
    public List<T> Items { get; set; } = new();
    public int TotalCount { get; set; }
    public int PageNumber { get; set; }
    public int PageSize { get; set; }
    public int TotalPages { get; set; }
    public bool HasNextPage { get; set; }
    public bool HasPreviousPage { get; set; }
}
```

### 2. Cursor-based Pagination

```csharp
public class CursorPaginationService
{
    private readonly ApplicationDbContext _context;

    public CursorPaginationService(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<CursorPagedResult<Student>> GetStudentsCursorPagedAsync(
        int? afterId = null,
        int pageSize = 20)
    {
        var query = _context.Students.AsQueryable();

        if (afterId.HasValue)
        {
            query = query.Where(s => s.Id > afterId.Value);
        }

        var students = await query
            .OrderBy(s => s.Id)
            .Take(pageSize + 1) // Take one extra to check if there are more
            .Include(s => s.Department)
            .ToListAsync();

        var hasNextPage = students.Count > pageSize;
        
        if (hasNextPage)
        {
            students.RemoveAt(students.Count - 1); // Remove the extra item
        }

        string? nextCursor = null;
        if (hasNextPage && students.Any())
        {
            nextCursor = students.Last().Id.ToString();
        }

        return new CursorPagedResult<Student>
        {
            Items = students,
            NextCursor = nextCursor,
            HasNextPage = hasNextPage,
            PageSize = pageSize
        };
    }
}

public class CursorPagedResult<T>
{
    public List<T> Items { get; set; } = new();
    public string? NextCursor { get; set; }
    public bool HasNextPage { get; set; }
    public int PageSize { get; set; }
}
```

## การใช้งาน Raw SQL

### 1. Raw SQL Queries

```csharp
public class RawSqlService
{
    private readonly ApplicationDbContext _context;

    public RawSqlService(ApplicationDbContext context)
    {
        _context = context;
    }

    // Execute raw SQL query
    public async Task<List<Student>> GetStudentsByRawSqlAsync(int departmentId)
    {
        return await _context.Students
            .FromSqlRaw("SELECT * FROM Students WHERE DepartmentId = {0}", departmentId)
            .ToListAsync();
    }

    // Interpolated SQL (safer)
    public async Task<List<Student>> GetStudentsByInterpolatedSqlAsync(int departmentId)
    {
        return await _context.Students
            .FromSql($"SELECT * FROM Students WHERE DepartmentId = {departmentId}")
            .ToListAsync();
    }

    // Execute stored procedure
    public async Task<List<StudentSummaryDto>> GetStudentSummaryAsync(int departmentId)
    {
        return await _context.Database
            .SqlQueryRaw<StudentSummaryDto>("EXEC sp_GetStudentsByDepartment @p0", departmentId)
            .ToListAsync();
    }

    // Execute non-query SQL
    public async Task<int> UpdateStudentStatusAsync(int studentId, StudentStatus status)
    {
        return await _context.Database
            .ExecuteSqlRawAsync(
                "UPDATE Students SET Status = {0} WHERE Id = {1}", 
                (int)status, studentId);
    }

    // Complex raw query with multiple parameters
    public async Task<List<EnrollmentReportDto>> GetEnrollmentReportRawAsync(
        DateTime fromDate, 
        DateTime toDate, 
        int? departmentId = null)
    {
        var sql = @"
            SELECT 
                s.FirstName + ' ' + s.LastName AS StudentName,
                s.StudentNumber,
                c.Title AS CourseName,
                c.CourseCode,
                d.Name AS DepartmentName,
                e.EnrollmentDate,
                e.Grade,
                e.Status
            FROM Enrollments e
            INNER JOIN Students s ON e.StudentId = s.Id
            INNER JOIN Courses c ON e.CourseId = c.Id
            INNER JOIN Departments d ON c.DepartmentId = d.Id
            WHERE e.EnrollmentDate BETWEEN @fromDate AND @toDate";

        var parameters = new List<object> { fromDate, toDate };

        if (departmentId.HasValue)
        {
            sql += " AND d.Id = @departmentId";
            parameters.Add(departmentId.Value);
        }

        sql += " ORDER BY e.EnrollmentDate DESC";

        return await _context.Database
            .SqlQueryRaw<EnrollmentReportDto>(sql, parameters.ToArray())
            .ToListAsync();
    }
}

public class StudentSummaryDto
{
    public int Id { get; set; }
    public string StudentNumber { get; set; }
    public string FullName { get; set; }
    public string Email { get; set; }
    public string DepartmentName { get; set; }
    public int EnrolledCoursesCount { get; set; }
}
```

---

**หมายเหตุ**: การเลือกใช้วิธีการ Query ที่เหมาะสมขึ้นอยู่กับความซับซ้อนของ Query และความต้องการด้านประสิทธิภาพ LINQ ให้ความสะดวกและ type safety แต่ Raw SQL อาจเร็วกว่าในกรณีที่ซับซ้อน