# บทที่ 15: การทดสอบ (Testing) กับ EF Core

## ความสำคัญของการทดสอบด้วย EF Core

การทดสอบเป็นส่วนสำคัญในการพัฒนาซอฟต์แวร์ EF Core มีเครื่องมือและรูปแบบการทดสอบหลายแบบที่ช่วยให้เราทดสอบ data access layer ได้อย่างมีประสิทธิภาพ

### ประเภทของการทดสอบที่สำคัญ

1. **Unit Tests** - ทดสอบ business logic แต่ละส่วน
2. **Integration Tests** - ทดสอบการทำงานรวมกับฐานข้อมูลจริง
3. **Repository Tests** - ทดสอบ data access methods
4. **Performance Tests** - ทดสอบประสิทธิภาพ queries

---

## การติดตั้ง Testing Packages

### 1. เพิ่ม Testing Packages

```bash
# สร้าง Test project
dotnet new xunit -n MyApp.Tests
cd MyApp.Tests

# เพิ่ม EF Core testing packages
dotnet add package Microsoft.EntityFrameworkCore.InMemory
dotnet add package Microsoft.EntityFrameworkCore.Sqlite
dotnet add package Microsoft.Extensions.DependencyInjection

# เพิ่ม testing utilities
dotnet add package FluentAssertions
dotnet add package Moq
dotnet add package Microsoft.AspNetCore.Mvc.Testing

# เพิ่ม reference ไปยัง main project
dotnet add reference ../MyApp/MyApp.csproj
```

### 2. โครงสร้าง Test Project

```
MyApp.Tests/
├── UnitTests/
│   ├── Services/
│   ├── Repositories/
│   └── Models/
├── IntegrationTests/
│   ├── Data/
│   └── Controllers/
├── Helpers/
│   ├── TestDbContextFactory.cs
│   └── SeedData.cs
└── MyApp.Tests.csproj
```

---

## การทดสอบด้วย In-Memory Database

### 1. การสร้าง In-Memory DbContext สำหรับทดสอบ

```csharp
// Helpers/TestDbContextFactory.cs
using Microsoft.EntityFrameworkCore;
using MyApp.Data;

public class TestDbContextFactory
{
    public static ApplicationDbContext CreateInMemoryContext(string databaseName = null)
    {
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseInMemoryDatabase(databaseName ?? Guid.NewGuid().ToString())
            .EnableSensitiveDataLogging()
            .Options;

        return new ApplicationDbContext(options);
    }
    
    public static ApplicationDbContext CreateSqliteInMemoryContext()
    {
        var connection = new SqliteConnection("DataSource=:memory:");
        connection.Open();
        
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseSqlite(connection)
            .Options;

        var context = new ApplicationDbContext(options);
        context.Database.EnsureCreated();
        
        return context;
    }
}
```

### 2. การสร้าง Test Data

```csharp
// Helpers/SeedData.cs
public static class SeedData
{
    public static List<Student> GetTestStudents()
    {
        return new List<Student>
        {
            new Student
            {
                Id = 1,
                FirstName = "สมชาย",
                LastName = "ใจดี",
                Email = "somchai@example.com",
                DateOfBirth = new DateTime(2000, 1, 1)
            },
            new Student
            {
                Id = 2,
                FirstName = "สมหญิง",
                LastName = "สุขใจ",
                Email = "somying@example.com",
                DateOfBirth = new DateTime(2001, 2, 15)
            },
            new Student
            {
                Id = 3,
                FirstName = "วิชัย",
                LastName = "เก่งเรียน",
                Email = "vichai@example.com",
                DateOfBirth = new DateTime(1999, 12, 10)
            }
        };
    }
    
    public static List<Course> GetTestCourses()
    {
        return new List<Course>
        {
            new Course
            {
                Id = 1,
                Title = "คณิตศาสตร์พื้นฐาน",
                Credits = 3,
                DepartmentId = 1
            },
            new Course
            {
                Id = 2,
                Title = "ฟิสิกส์เบื้องต้น",
                Credits = 4,
                DepartmentId = 1
            }
        };
    }
    
    public static void SeedDatabase(ApplicationDbContext context)
    {
        context.Students.AddRange(GetTestStudents());
        context.Courses.AddRange(GetTestCourses());
        context.SaveChanges();
    }
}
```

### 3. ตัวอย่าง Unit Test พื้นฐาน

```csharp
// UnitTests/Repositories/StudentRepositoryTests.cs
using Xunit;
using FluentAssertions;
using MyApp.Repositories;

public class StudentRepositoryTests : IDisposable
{
    private readonly ApplicationDbContext _context;
    private readonly StudentRepository _repository;

    public StudentRepositoryTests()
    {
        _context = TestDbContextFactory.CreateInMemoryContext();
        _repository = new StudentRepository(_context);
        
        // Seed test data
        SeedData.SeedDatabase(_context);
    }

    [Fact]
    public async Task GetAllAsync_ShouldReturnAllStudents()
    {
        // Act
        var result = await _repository.GetAllAsync();

        // Assert
        result.Should().HaveCount(3);
        result.Should().Contain(s => s.FirstName == "สมชาย");
    }

    [Fact]
    public async Task GetByIdAsync_WithValidId_ShouldReturnStudent()
    {
        // Act
        var result = await _repository.GetByIdAsync(1);

        // Assert
        result.Should().NotBeNull();
        result.FirstName.Should().Be("สมชาย");
        result.Email.Should().Be("somchai@example.com");
    }

    [Fact]
    public async Task GetByIdAsync_WithInvalidId_ShouldReturnNull()
    {
        // Act
        var result = await _repository.GetByIdAsync(999);

        // Assert
        result.Should().BeNull();
    }

    [Fact]
    public async Task AddAsync_WithValidStudent_ShouldAddToDatabase()
    {
        // Arrange
        var newStudent = new Student
        {
            FirstName = "ทดสอบ",
            LastName = "ใหม่",
            Email = "test@example.com",
            DateOfBirth = DateTime.Now.AddYears(-20)
        };

        // Act
        await _repository.AddAsync(newStudent);
        var result = await _repository.GetByEmailAsync("test@example.com");

        // Assert
        result.Should().NotBeNull();
        result.FirstName.Should().Be("ทดสอบ");
    }

    [Fact]
    public async Task UpdateAsync_WithValidStudent_ShouldUpdateDatabase()
    {
        // Arrange
        var student = await _repository.GetByIdAsync(1);
        student.FirstName = "อัพเดท";

        // Act
        await _repository.UpdateAsync(student);
        var updatedStudent = await _repository.GetByIdAsync(1);

        // Assert
        updatedStudent.FirstName.Should().Be("อัพเดท");
    }

    [Fact]
    public async Task DeleteAsync_WithValidId_ShouldRemoveFromDatabase()
    {
        // Act
        await _repository.DeleteAsync(1);
        var result = await _repository.GetByIdAsync(1);

        // Assert
        result.Should().BeNull();
    }

    public void Dispose()
    {
        _context.Dispose();
    }
}
```

---

## การทดสอบด้วย SQLite In-Memory

### 1. ข้อดีของ SQLite สำหรับการทดสอบ

```csharp
// UnitTests/Services/StudentServiceTests.cs
public class StudentServiceTests : IDisposable
{
    private readonly ApplicationDbContext _context;
    private readonly StudentService _service;
    private readonly SqliteConnection _connection;

    public StudentServiceTests()
    {
        // ใช้ SQLite in-memory เพื่อทดสอบ SQL ได้ใกล้เคียงจริง
        _connection = new SqliteConnection("DataSource=:memory:");
        _connection.Open();
        
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseSqlite(_connection)
            .Options;

        _context = new ApplicationDbContext(options);
        _context.Database.EnsureCreated();
        
        _service = new StudentService(_context);
        
        // Seed data
        SeedData.SeedDatabase(_context);
    }

    [Fact]
    public async Task GetStudentsByAgeRangeAsync_ShouldReturnCorrectStudents()
    {
        // Arrange
        var fromAge = 20;
        var toAge = 25;
        var currentDate = DateTime.Now;

        // Act
        var result = await _service.GetStudentsByAgeRangeAsync(fromAge, toAge);

        // Assert
        result.Should().NotBeNull();
        result.Should().OnlyContain(s => 
            CalculateAge(s.DateOfBirth, currentDate) >= fromAge && 
            CalculateAge(s.DateOfBirth, currentDate) <= toAge);
    }

    [Theory]
    [InlineData("test@email.com", true)]
    [InlineData("somchai@example.com", false)] // มีอยู่แล้ว
    public async Task IsEmailUniqueAsync_ShouldReturnCorrectResult(string email, bool expected)
    {
        // Act
        var result = await _service.IsEmailUniqueAsync(email);

        // Assert
        result.Should().Be(expected);
    }

    private static int CalculateAge(DateTime birthDate, DateTime currentDate)
    {
        var age = currentDate.Year - birthDate.Year;
        if (currentDate < birthDate.AddYears(age))
            age--;
        return age;
    }

    public void Dispose()
    {
        _context.Dispose();
        _connection.Dispose();
    }
}
```

---

## Integration Tests

### 1. การทดสอบกับฐานข้อมูลจริง

```csharp
// IntegrationTests/Data/ApplicationDbContextIntegrationTests.cs
[Collection("Database")]
public class ApplicationDbContextIntegrationTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;

    public ApplicationDbContextIntegrationTests(DatabaseFixture fixture)
    {
        _fixture = fixture;
    }

    [Fact]
    public async Task SaveChanges_WithValidData_ShouldPersistToDatabase()
    {
        // Arrange
        using var context = _fixture.CreateContext();
        var student = new Student
        {
            FirstName = "Integration",
            LastName = "Test",
            Email = $"integration{DateTime.Now.Ticks}@example.com",
            DateOfBirth = DateTime.Now.AddYears(-20)
        };

        // Act
        context.Students.Add(student);
        await context.SaveChangesAsync();

        // Assert
        using var verifyContext = _fixture.CreateContext();
        var savedStudent = await verifyContext.Students
            .FirstOrDefaultAsync(s => s.Email == student.Email);
            
        savedStudent.Should().NotBeNull();
        savedStudent.FirstName.Should().Be("Integration");
    }

    [Fact]
    public async Task BulkInsert_WithManyRecords_ShouldPerformWell()
    {
        // Arrange
        using var context = _fixture.CreateContext();
        var students = Enumerable.Range(1, 1000)
            .Select(i => new Student
            {
                FirstName = $"Student{i}",
                LastName = $"Test{i}",
                Email = $"student{i}@bulk-test.com",
                DateOfBirth = DateTime.Now.AddYears(-20).AddDays(i)
            })
            .ToList();

        // Act
        var stopwatch = Stopwatch.StartNew();
        context.Students.AddRange(students);
        await context.SaveChangesAsync();
        stopwatch.Stop();

        // Assert
        stopwatch.ElapsedMilliseconds.Should().BeLessThan(5000); // ควรใช้เวลาไม่เกิน 5 วินาที
        
        var count = await context.Students.CountAsync(s => s.Email.Contains("bulk-test"));
        count.Should().Be(1000);
    }
}

// DatabaseFixture สำหรับ Integration Tests
public class DatabaseFixture : IDisposable
{
    private const string ConnectionString = "Server=(localdb)\\mssqllocaldb;Database=TestDB;Trusted_Connection=true;";
    
    public DatabaseFixture()
    {
        using var context = CreateContext();
        context.Database.EnsureDeleted();
        context.Database.EnsureCreated();
    }
    
    public ApplicationDbContext CreateContext()
    {
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseSqlServer(ConnectionString)
            .Options;
            
        return new ApplicationDbContext(options);
    }
    
    public void Dispose()
    {
        using var context = CreateContext();
        context.Database.EnsureDeleted();
    }
}
```

---

## การทดสอบ Repository Pattern

### 1. การทดสอบ Generic Repository

```csharp
// UnitTests/Repositories/GenericRepositoryTests.cs
public class GenericRepositoryTests
{
    private readonly ApplicationDbContext _context;
    private readonly IGenericRepository<Student> _repository;

    public GenericRepositoryTests()
    {
        _context = TestDbContextFactory.CreateSqliteInMemoryContext();
        _repository = new GenericRepository<Student>(_context);
        SeedData.SeedDatabase(_context);
    }

    [Fact]
    public async Task FindAsync_WithPredicate_ShouldReturnMatchingEntities()
    {
        // Act
        var result = await _repository.FindAsync(s => s.FirstName.Contains("สม"));

        // Assert
        result.Should().HaveCount(2);
        result.Should().Contain(s => s.FirstName == "สมชาย");
        result.Should().Contain(s => s.FirstName == "สมหญิง");
    }

    [Fact]
    public async Task GetPagedAsync_ShouldReturnCorrectPage()
    {
        // Arrange
        var pageNumber = 1;
        var pageSize = 2;

        // Act
        var result = await _repository.GetPagedAsync(
            pageNumber, 
            pageSize, 
            s => s.OrderBy(x => x.FirstName));

        // Assert
        result.Data.Should().HaveCount(2);
        result.TotalCount.Should().Be(3);
        result.PageNumber.Should().Be(1);
        result.PageSize.Should().Be(2);
        result.TotalPages.Should().Be(2);
    }

    [Fact]
    public async Task ExistsAsync_WithValidPredicate_ShouldReturnTrue()
    {
        // Act
        var exists = await _repository.ExistsAsync(s => s.Email == "somchai@example.com");

        // Assert
        exists.Should().BeTrue();
    }

    [Fact]
    public async Task CountAsync_WithPredicate_ShouldReturnCorrectCount()
    {
        // Act
        var count = await _repository.CountAsync(s => s.FirstName.StartsWith("สม"));

        // Assert
        count.Should().Be(2);
    }
}
```

### 2. การทดสอบ Unit of Work Pattern

```csharp
// UnitTests/Services/StudentManagementServiceTests.cs
public class StudentManagementServiceTests
{
    private readonly Mock<IUnitOfWork> _mockUnitOfWork;
    private readonly Mock<IStudentRepository> _mockStudentRepo;
    private readonly Mock<ICourseRepository> _mockCourseRepo;
    private readonly StudentManagementService _service;

    public StudentManagementServiceTests()
    {
        _mockUnitOfWork = new Mock<IUnitOfWork>();
        _mockStudentRepo = new Mock<IStudentRepository>();
        _mockCourseRepo = new Mock<ICourseRepository>();

        _mockUnitOfWork.Setup(u => u.Students).Returns(_mockStudentRepo.Object);
        _mockUnitOfWork.Setup(u => u.Courses).Returns(_mockCourseRepo.Object);

        _service = new StudentManagementService(_mockUnitOfWork.Object);
    }

    [Fact]
    public async Task EnrollStudentToCourseAsync_WithValidData_ShouldSucceed()
    {
        // Arrange
        var student = SeedData.GetTestStudents().First();
        var course = SeedData.GetTestCourses().First();

        _mockStudentRepo.Setup(r => r.GetByIdAsync(student.Id))
            .ReturnsAsync(student);
        _mockCourseRepo.Setup(r => r.GetByIdAsync(course.Id))
            .ReturnsAsync(course);
        _mockUnitOfWork.Setup(u => u.SaveChangesAsync())
            .ReturnsAsync(true);

        // Act
        var result = await _service.EnrollStudentToCourseAsync(student.Id, course.Id);

        // Assert
        result.Should().BeTrue();
        _mockUnitOfWork.Verify(u => u.SaveChangesAsync(), Times.Once);
    }

    [Fact]
    public async Task EnrollStudentToCourseAsync_WithNonExistentStudent_ShouldThrowException()
    {
        // Arrange
        _mockStudentRepo.Setup(r => r.GetByIdAsync(It.IsAny<int>()))
            .ReturnsAsync((Student)null);

        // Act & Assert
        var action = async () => await _service.EnrollStudentToCourseAsync(999, 1);
        await action.Should().ThrowAsync<StudentNotFoundException>();
    }
}
```

---

## Performance Testing

### 1. การทดสอบประสิทธิภาพ Query

```csharp
// IntegrationTests/Performance/QueryPerformanceTests.cs
public class QueryPerformanceTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;

    public QueryPerformanceTests(DatabaseFixture fixture)
    {
        _fixture = fixture;
        SeedLargeDataSet();
    }

    [Fact]
    public async Task GetStudentsWithIncludes_ShouldPerformWell()
    {
        // Arrange
        using var context = _fixture.CreateContext();

        // Act
        var stopwatch = Stopwatch.StartNew();
        var students = await context.Students
            .Include(s => s.Enrollments)
            .ThenInclude(e => e.Course)
            .Take(100)
            .ToListAsync();
        stopwatch.Stop();

        // Assert
        stopwatch.ElapsedMilliseconds.Should().BeLessThan(1000);
        students.Should().HaveCount(100);
    }

    [Fact]
    public async Task GetStudentsWithSplitQuery_ShouldBeFasterThanSingleQuery()
    {
        using var context = _fixture.CreateContext();

        // Single query
        var stopwatch1 = Stopwatch.StartNew();
        var students1 = await context.Students
            .Include(s => s.Enrollments)
            .ThenInclude(e => e.Course)
            .Take(50)
            .ToListAsync();
        stopwatch1.Stop();

        // Split query
        var stopwatch2 = Stopwatch.StartNew();
        var students2 = await context.Students
            .AsSplitQuery()
            .Include(s => s.Enrollments)
            .ThenInclude(e => e.Course)
            .Take(50)
            .ToListAsync();
        stopwatch2.Stop();

        // Assert
        students1.Should().HaveCount(50);
        students2.Should().HaveCount(50);
        
        // Split query ควรเร็วกว่าหรือเท่ากับ single query
        stopwatch2.ElapsedMilliseconds.Should().BeLessOrEqualTo(stopwatch1.ElapsedMilliseconds * 1.2);
    }

    private void SeedLargeDataSet()
    {
        using var context = _fixture.CreateContext();
        
        if (context.Students.Any()) return; // มีข้อมูลแล้ว

        var students = new List<Student>();
        var courses = new List<Course>();
        var enrollments = new List<Enrollment>();

        // สร้างข้อมูลจำนวนมาก
        for (int i = 1; i <= 10000; i++)
        {
            students.Add(new Student
            {
                FirstName = $"Student{i}",
                LastName = $"LastName{i}",
                Email = $"student{i}@test.com",
                DateOfBirth = DateTime.Now.AddYears(-20).AddDays(i)
            });
        }

        for (int i = 1; i <= 100; i++)
        {
            courses.Add(new Course
            {
                Title = $"Course {i}",
                Credits = i % 5 + 1
            });
        }

        context.Students.AddRange(students);
        context.Courses.AddRange(courses);
        context.SaveChanges();

        // สร้าง enrollments
        var random = new Random();
        for (int i = 1; i <= 50000; i++)
        {
            enrollments.Add(new Enrollment
            {
                StudentId = random.Next(1, 10001),
                CourseId = random.Next(1, 101),
                EnrolledDate = DateTime.Now.AddDays(-random.Next(365))
            });
        }

        context.Enrollments.AddRange(enrollments);
        context.SaveChanges();
    }
}
```

---

## การทดสอบ Concurrency

### 1. ทดสอบการทำงานพร้อมกัน

```csharp
// UnitTests/Concurrency/ConcurrencyTests.cs
public class ConcurrencyTests
{
    [Fact]
    public async Task ConcurrentAccess_ShouldHandleOptimisticConcurrency()
    {
        // Arrange
        using var context1 = TestDbContextFactory.CreateSqliteInMemoryContext();
        using var context2 = TestDbContextFactory.CreateSqliteInMemoryContext();
        
        var student = new Student
        {
            FirstName = "Concurrent",
            LastName = "Test",
            Email = "concurrent@test.com",
            DateOfBirth = DateTime.Now.AddYears(-20)
        };
        
        context1.Students.Add(student);
        await context1.SaveChangesAsync();

        // Act & Assert
        var student1 = await context1.Students.FirstAsync();
        var student2 = await context2.Students.FirstAsync();

        student1.FirstName = "Updated1";
        student2.FirstName = "Updated2";

        await context1.SaveChangesAsync(); // ควรสำเร็จ

        var action = async () => await context2.SaveChangesAsync(); // ควร fail เพราะ concurrency conflict
        await action.Should().ThrowAsync<DbUpdateConcurrencyException>();
    }

    [Fact]
    public async Task BulkOperations_WithMultipleThreads_ShouldNotInterfere()
    {
        // Arrange
        var tasks = new List<Task>();
        var results = new ConcurrentBag<int>();

        // Act
        for (int i = 0; i < 10; i++)
        {
            var taskNumber = i;
            tasks.Add(Task.Run(async () =>
            {
                using var context = TestDbContextFactory.CreateSqliteInMemoryContext();
                var students = Enumerable.Range(1, 100)
                    .Select(j => new Student
                    {
                        FirstName = $"Student{taskNumber}-{j}",
                        LastName = "Concurrent",
                        Email = $"student{taskNumber}-{j}@concurrent.com",
                        DateOfBirth = DateTime.Now.AddYears(-20)
                    })
                    .ToList();

                context.Students.AddRange(students);
                var saved = await context.SaveChangesAsync();
                results.Add(saved);
            }));
        }

        await Task.WhenAll(tasks);

        // Assert
        results.Should().HaveCount(10);
        results.Sum().Should().Be(1000); // 10 tasks × 100 students each
    }
}
```

---

## Test Utilities และ Helpers

### 1. Builder Pattern สำหรับ Test Data

```csharp
// Helpers/StudentBuilder.cs
public class StudentBuilder
{
    private Student _student = new Student();

    public StudentBuilder WithName(string firstName, string lastName)
    {
        _student.FirstName = firstName;
        _student.LastName = lastName;
        return this;
    }

    public StudentBuilder WithEmail(string email)
    {
        _student.Email = email;
        return this;
    }

    public StudentBuilder WithBirthDate(DateTime birthDate)
    {
        _student.DateOfBirth = birthDate;
        return this;
    }

    public StudentBuilder WithAge(int age)
    {
        _student.DateOfBirth = DateTime.Now.AddYears(-age);
        return this;
    }

    public Student Build() => _student;

    public static implicit operator Student(StudentBuilder builder) => builder.Build();
}

// การใช้งาน
[Fact]
public async Task ExampleUsage()
{
    var student = new StudentBuilder()
        .WithName("ทดสอบ", "สร้างง่าย")
        .WithEmail("easy@test.com")
        .WithAge(20)
        .Build();
    
    // หรือ
    Student student2 = new StudentBuilder()
        .WithName("อัตโนมัติ", "แปลง")
        .WithEmail("auto@test.com")
        .WithAge(22);
}
```

### 2. Custom Assertions

```csharp
// Helpers/StudentAssertions.cs
public static class StudentAssertions
{
    public static void ShouldBeValidStudent(this Student student)
    {
        student.Should().NotBeNull();
        student.FirstName.Should().NotBeNullOrWhiteSpace();
        student.LastName.Should().NotBeNullOrWhiteSpace();
        student.Email.Should().MatchRegex(@"^[^@\s]+@[^@\s]+\.[^@\s]+$");
        student.DateOfBirth.Should().BeBefore(DateTime.Now);
    }

    public static void ShouldHaveValidEnrollments(this Student student)
    {
        if (student.Enrollments?.Any() == true)
        {
            foreach (var enrollment in student.Enrollments)
            {
                enrollment.StudentId.Should().Be(student.Id);
                enrollment.Course.Should().NotBeNull();
                enrollment.EnrolledDate.Should().BeBefore(DateTime.Now.AddDays(1));
            }
        }
    }
}

// การใช้งาน
[Fact]
public void Student_ShouldPassValidation()
{
    var student = new StudentBuilder()
        .WithName("Valid", "Student")
        .WithEmail("valid@email.com")
        .WithAge(20)
        .Build();

    student.ShouldBeValidStudent();
}
```

---

## Best Practices สำหรับการทดสอบ

### 1. การตั้งชื่อ Test Methods

```csharp
// รูปแบบ: MethodName_Scenario_ExpectedResult
[Fact]
public async Task GetStudentById_WithValidId_ShouldReturnStudent()
[Fact]
public async Task GetStudentById_WithInvalidId_ShouldReturnNull()
[Fact]
public async Task CreateStudent_WithDuplicateEmail_ShouldThrowException()
[Fact]
public async Task UpdateStudent_WithValidData_ShouldUpdateSuccessfully()
```

### 2. การใช้ Test Categories

```csharp
// Categorize tests
[Trait("Category", "Unit")]
[Trait("Category", "Database")]
public class StudentRepositoryTests { }

[Trait("Category", "Integration")]
[Trait("Category", "Performance")]
public class QueryPerformanceTests { }
```

### 3. การจัดการ Test Data

```csharp
// ใช้ IDisposable สำหรับ cleanup
public class TestClassWithCleanup : IDisposable
{
    private readonly ApplicationDbContext _context;

    public TestClassWithCleanup()
    {
        _context = TestDbContextFactory.CreateSqliteInMemoryContext();
    }

    [Fact]
    public async Task SomeTest()
    {
        // Test implementation
    }

    public void Dispose()
    {
        _context?.Dispose();
    }
}
```

---

## การรัน Tests

### 1. คำสั่ง dotnet test

```bash
# รัน tests ทั้งหมด
dotnet test

# รัน tests ใน project เฉพาะ
dotnet test MyApp.Tests

# รัน tests ที่มี category เฉพาะ
dotnet test --filter "Category=Unit"

# รัน tests ที่มีชื่อเฉพาะ
dotnet test --filter "DisplayName~Student"

# รัน tests พร้อมแสดง coverage
dotnet test --collect:"XPlat Code Coverage"

# รัน tests แบบ verbose
dotnet test --verbosity detailed
```

### 2. การใช้ Test Explorer ใน Visual Studio

- ใช้ Test Explorer window เพื่อรัน tests แต่ละตัว
- ดู test results และ failures
- Debug tests โดยตรง

---

## สรุป

การทดสอบกับ EF Core มีหลายรูปแบบ:

### **Unit Tests**
- ใช้ In-Memory Database หรือ SQLite
- ทดสอบ Repository และ Service layers
- เร็วและไม่ต้องพึ่งพา external resources

### **Integration Tests**  
- ใช้ฐานข้อมูลจริงหรือ Test Database
- ทดสอบ end-to-end scenarios
- ช้ากว่า unit tests แต่ใกล้เคียงสภาพจริง

### **Performance Tests**
- ทดสอบประสิทธิภาพ queries
- วัด response time และ resource usage
- ช่วยหาปัญหา performance bottlenecks

### **Best Practices**
- แยก test categories ชัดเจน  
- ใช้ Builder Pattern สำหรับ test data
- Cleanup resources หลัง test
- ตั้งชื่อ test methods ให้ชัดเจน

การทดสอบที่ดีช่วยให้:
- มั่นใจในคุณภาพโค้ด
- หา bugs ก่อนนำไป production
- Refactor โค้ดได้อย่างปลอดภัย
- เป็น documentation สำหรับการใช้งาน

---

**หัวข้อถัดไป:** [16 - Deployment และ Production](16-deployment.md)