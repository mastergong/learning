# บทที่ 18: EF Core กับ ASP.NET Core Web API

## 🎯 ภาพรวมของบทนี้

บทนี้จะสอนการใช้ EF Core กับ ASP.NET Core Web API แบบครบวงจร พร้อมตัวอย่าง RESTful API, Pagination, Filtering, AutoMapper และ Best Practices

### สิ่งที่คุณจะได้เรียนรู้:
- สร้าง RESTful API ด้วย EF Core
- DTOs และ AutoMapper
- Pagination และ Filtering
- Error Handling และ Validation
- API Versioning
- Caching Strategies

---

## 1️⃣ สร้าง Web API Project

### 📦 สร้าง Project

```bash
dotnet new webapi -n StudentAPI
cd StudentAPI

# ติดตั้ง packages ที่จำเป็น
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
dotnet add package FluentValidation.AspNetCore
dotnet add package Swashbuckle.AspNetCore
```

### 📁 โครงสร้าง Project

```
StudentAPI/
├── Controllers/
│   └── StudentsController.cs
├── Data/
│   ├── ApplicationDbContext.cs
│   └── Configurations/
├── Models/
│   ├── Entities/
│   │   └── Student.cs
│   └── DTOs/
│       ├── StudentDto.cs
│       ├── CreateStudentDto.cs
│       └── UpdateStudentDto.cs
├── Services/
│   ├── Interfaces/
│   │   └── IStudentService.cs
│   └── StudentService.cs
├── Repositories/
│   ├── Interfaces/
│   │   └── IStudentRepository.cs
│   └── StudentRepository.cs
├── Mappings/
│   └── MappingProfile.cs
├── Validators/
│   └── CreateStudentValidator.cs
├── Helpers/
│   ├── PagedList.cs
│   └── QueryParameters.cs
├── Middleware/
│   └── ErrorHandlingMiddleware.cs
├── appsettings.json
└── Program.cs
```

---

## 2️⃣ Models และ DTOs

### Entity Models

```csharp
// Models/Entities/Student.cs
namespace StudentAPI.Models.Entities;

public class Student
{
    public int Id { get; set; }
    public string FirstName { get; set; } = string.Empty;
    public string LastName { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public string StudentNumber { get; set; } = string.Empty;
    public DateTime DateOfBirth { get; set; }
    public int? DepartmentId { get; set; }
    public virtual Department? Department { get; set; }
    public virtual ICollection<Enrollment> Enrollments { get; set; } = new List<Enrollment>();
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public DateTime? UpdatedAt { get; set; }
    public bool IsDeleted { get; set; }
}
```

### DTOs (Data Transfer Objects)

```csharp
// Models/DTOs/StudentDto.cs
namespace StudentAPI.Models.DTOs;

public class StudentDto
{
    public int Id { get; set; }
    public string FirstName { get; set; } = string.Empty;
    public string LastName { get; set; } = string.Empty;
    public string FullName { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public string StudentNumber { get; set; } = string.Empty;
    public int Age { get; set; }
    public string? DepartmentName { get; set; }
    public int EnrollmentCount { get; set; }
    public DateTime CreatedAt { get; set; }
}

// Models/DTOs/CreateStudentDto.cs
namespace StudentAPI.Models.DTOs;

public class CreateStudentDto
{
    public string FirstName { get; set; } = string.Empty;
    public string LastName { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public DateTime DateOfBirth { get; set; }
    public int? DepartmentId { get; set; }
}

// Models/DTOs/UpdateStudentDto.cs
namespace StudentAPI.Models.DTOs;

public class UpdateStudentDto
{
    public string FirstName { get; set; } = string.Empty;
    public string LastName { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public DateTime DateOfBirth { get; set; }
    public int? DepartmentId { get; set; }
}

// Models/DTOs/ApiResponse.cs
namespace StudentAPI.Models.DTOs;

public class ApiResponse<T>
{
    public bool Success { get; set; }
    public string Message { get; set; } = string.Empty;
    public T? Data { get; set; }
    public List<string>? Errors { get; set; }

    public static ApiResponse<T> SuccessResponse(T data, string message = "Success")
    {
        return new ApiResponse<T>
        {
            Success = true,
            Message = message,
            Data = data
        };
    }

    public static ApiResponse<T> ErrorResponse(string message, List<string>? errors = null)
    {
        return new ApiResponse<T>
        {
            Success = false,
            Message = message,
            Errors = errors
        };
    }
}
```

---

## 3️⃣ AutoMapper Configuration

```csharp
// Mappings/MappingProfile.cs
using AutoMapper;
using StudentAPI.Models.Entities;
using StudentAPI.Models.DTOs;

namespace StudentAPI.Mappings;

public class MappingProfile : Profile
{
    public MappingProfile()
    {
        // Student mappings
        CreateMap<Student, StudentDto>()
            .ForMember(dest => dest.FullName, 
                opt => opt.MapFrom(src => $"{src.FirstName} {src.LastName}"))
            .ForMember(dest => dest.Age, 
                opt => opt.MapFrom(src => DateTime.Now.Year - src.DateOfBirth.Year))
            .ForMember(dest => dest.DepartmentName, 
                opt => opt.MapFrom(src => src.Department != null ? src.Department.Name : null))
            .ForMember(dest => dest.EnrollmentCount, 
                opt => opt.MapFrom(src => src.Enrollments.Count));

        CreateMap<CreateStudentDto, Student>()
            .ForMember(dest => dest.Id, opt => opt.Ignore())
            .ForMember(dest => dest.StudentNumber, opt => opt.Ignore())
            .ForMember(dest => dest.CreatedAt, opt => opt.MapFrom(_ => DateTime.UtcNow))
            .ForMember(dest => dest.IsDeleted, opt => opt.MapFrom(_ => false));

        CreateMap<UpdateStudentDto, Student>()
            .ForMember(dest => dest.Id, opt => opt.Ignore())
            .ForMember(dest => dest.StudentNumber, opt => opt.Ignore())
            .ForMember(dest => dest.CreatedAt, opt => opt.Ignore())
            .ForMember(dest => dest.UpdatedAt, opt => opt.MapFrom(_ => DateTime.UtcNow));
    }
}
```

---

## 4️⃣ Pagination และ Filtering

```csharp
// Helpers/QueryParameters.cs
namespace StudentAPI.Helpers;

public class QueryParameters
{
    private const int MaxPageSize = 100;
    private int _pageSize = 10;

    public int PageNumber { get; set; } = 1;
    
    public int PageSize
    {
        get => _pageSize;
        set => _pageSize = (value > MaxPageSize) ? MaxPageSize : value;
    }

    public string? SearchTerm { get; set; }
    public string? SortBy { get; set; }
    public bool SortDescending { get; set; }
}

// Helpers/PagedList.cs
using Microsoft.EntityFrameworkCore;

namespace StudentAPI.Helpers;

public class PagedList<T>
{
    public List<T> Items { get; set; }
    public int PageNumber { get; set; }
    public int PageSize { get; set; }
    public int TotalCount { get; set; }
    public int TotalPages => (int)Math.Ceiling(TotalCount / (double)PageSize);
    public bool HasPrevious => PageNumber > 1;
    public bool HasNext => PageNumber < TotalPages;

    public PagedList(List<T> items, int count, int pageNumber, int pageSize)
    {
        Items = items;
        TotalCount = count;
        PageNumber = pageNumber;
        PageSize = pageSize;
    }

    public static async Task<PagedList<T>> CreateAsync(
        IQueryable<T> source, 
        int pageNumber, 
        int pageSize)
    {
        var count = await source.CountAsync();
        var items = await source
            .Skip((pageNumber - 1) * pageSize)
            .Take(pageSize)
            .ToListAsync();

        return new PagedList<T>(items, count, pageNumber, pageSize);
    }
}

// Helpers/PaginationMetadata.cs
namespace StudentAPI.Helpers;

public class PaginationMetadata
{
    public int CurrentPage { get; set; }
    public int PageSize { get; set; }
    public int TotalCount { get; set; }
    public int TotalPages { get; set; }
    public bool HasPrevious { get; set; }
    public bool HasNext { get; set; }
}
```

---

## 5️⃣ Repository และ Service Layer

```csharp
// Repositories/Interfaces/IStudentRepository.cs
using StudentAPI.Models.Entities;
using StudentAPI.Helpers;

namespace StudentAPI.Repositories.Interfaces;

public interface IStudentRepository
{
    Task<PagedList<Student>> GetAllAsync(QueryParameters parameters);
    Task<Student?> GetByIdAsync(int id);
    Task<Student?> GetByEmailAsync(string email);
    Task<bool> ExistsAsync(int id);
    Task<bool> EmailExistsAsync(string email, int? excludeId = null);
    Task<Student> CreateAsync(Student student);
    Task UpdateAsync(Student student);
    Task DeleteAsync(Student student);
    Task<int> SaveChangesAsync();
}

// Repositories/StudentRepository.cs
using Microsoft.EntityFrameworkCore;
using StudentAPI.Data;
using StudentAPI.Models.Entities;
using StudentAPI.Helpers;
using StudentAPI.Repositories.Interfaces;

namespace StudentAPI.Repositories;

public class StudentRepository : IStudentRepository
{
    private readonly ApplicationDbContext _context;

    public StudentRepository(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<PagedList<Student>> GetAllAsync(QueryParameters parameters)
    {
        var query = _context.Students
            .Include(s => s.Department)
            .Include(s => s.Enrollments)
            .Where(s => !s.IsDeleted)
            .AsQueryable();

        // Search
        if (!string.IsNullOrWhiteSpace(parameters.SearchTerm))
        {
            var searchTerm = parameters.SearchTerm.ToLower();
            query = query.Where(s =>
                s.FirstName.ToLower().Contains(searchTerm) ||
                s.LastName.ToLower().Contains(searchTerm) ||
                s.Email.ToLower().Contains(searchTerm) ||
                s.StudentNumber.ToLower().Contains(searchTerm));
        }

        // Sorting
        query = parameters.SortBy?.ToLower() switch
        {
            "firstname" => parameters.SortDescending
                ? query.OrderByDescending(s => s.FirstName)
                : query.OrderBy(s => s.FirstName),
            "lastname" => parameters.SortDescending
                ? query.OrderByDescending(s => s.LastName)
                : query.OrderBy(s => s.LastName),
            "email" => parameters.SortDescending
                ? query.OrderByDescending(s => s.Email)
                : query.OrderBy(s => s.Email),
            "createdat" => parameters.SortDescending
                ? query.OrderByDescending(s => s.CreatedAt)
                : query.OrderBy(s => s.CreatedAt),
            _ => query.OrderBy(s => s.Id)
        };

        return await PagedList<Student>.CreateAsync(
            query, 
            parameters.PageNumber, 
            parameters.PageSize);
    }

    public async Task<Student?> GetByIdAsync(int id)
    {
        return await _context.Students
            .Include(s => s.Department)
            .Include(s => s.Enrollments)
                .ThenInclude(e => e.Course)
            .FirstOrDefaultAsync(s => s.Id == id && !s.IsDeleted);
    }

    public async Task<Student?> GetByEmailAsync(string email)
    {
        return await _context.Students
            .FirstOrDefaultAsync(s => s.Email == email && !s.IsDeleted);
    }

    public async Task<bool> ExistsAsync(int id)
    {
        return await _context.Students.AnyAsync(s => s.Id == id && !s.IsDeleted);
    }

    public async Task<bool> EmailExistsAsync(string email, int? excludeId = null)
    {
        var query = _context.Students.Where(s => s.Email == email && !s.IsDeleted);
        
        if (excludeId.HasValue)
        {
            query = query.Where(s => s.Id != excludeId.Value);
        }

        return await query.AnyAsync();
    }

    public async Task<Student> CreateAsync(Student student)
    {
        student.StudentNumber = await GenerateStudentNumberAsync();
        await _context.Students.AddAsync(student);
        return student;
    }

    public Task UpdateAsync(Student student)
    {
        student.UpdatedAt = DateTime.UtcNow;
        _context.Students.Update(student);
        return Task.CompletedTask;
    }

    public Task DeleteAsync(Student student)
    {
        student.IsDeleted = true;
        student.UpdatedAt = DateTime.UtcNow;
        _context.Students.Update(student);
        return Task.CompletedTask;
    }

    public async Task<int> SaveChangesAsync()
    {
        return await _context.SaveChangesAsync();
    }

    private async Task<string> GenerateStudentNumberAsync()
    {
        var year = DateTime.Now.Year.ToString()[2..];
        var count = await _context.Students.CountAsync();
        return $"{year}{(count + 1):D5}";
    }
}

// Services/Interfaces/IStudentService.cs
using StudentAPI.Models.DTOs;
using StudentAPI.Helpers;

namespace StudentAPI.Services.Interfaces;

public interface IStudentService
{
    Task<PagedList<StudentDto>> GetAllStudentsAsync(QueryParameters parameters);
    Task<StudentDto?> GetStudentByIdAsync(int id);
    Task<StudentDto> CreateStudentAsync(CreateStudentDto createDto);
    Task<StudentDto?> UpdateStudentAsync(int id, UpdateStudentDto updateDto);
    Task<bool> DeleteStudentAsync(int id);
}

// Services/StudentService.cs
using AutoMapper;
using StudentAPI.Models.DTOs;
using StudentAPI.Models.Entities;
using StudentAPI.Helpers;
using StudentAPI.Repositories.Interfaces;
using StudentAPI.Services.Interfaces;

namespace StudentAPI.Services;

public class StudentService : IStudentService
{
    private readonly IStudentRepository _repository;
    private readonly IMapper _mapper;
    private readonly ILogger<StudentService> _logger;

    public StudentService(
        IStudentRepository repository,
        IMapper mapper,
        ILogger<StudentService> logger)
    {
        _repository = repository;
        _mapper = mapper;
        _logger = logger;
    }

    public async Task<PagedList<StudentDto>> GetAllStudentsAsync(QueryParameters parameters)
    {
        var students = await _repository.GetAllAsync(parameters);
        var studentDtos = _mapper.Map<List<StudentDto>>(students.Items);

        return new PagedList<StudentDto>(
            studentDtos,
            students.TotalCount,
            students.PageNumber,
            students.PageSize);
    }

    public async Task<StudentDto?> GetStudentByIdAsync(int id)
    {
        var student = await _repository.GetByIdAsync(id);
        return student != null ? _mapper.Map<StudentDto>(student) : null;
    }

    public async Task<StudentDto> CreateStudentAsync(CreateStudentDto createDto)
    {
        // Check if email already exists
        if (await _repository.EmailExistsAsync(createDto.Email))
        {
            throw new InvalidOperationException("Email already exists");
        }

        var student = _mapper.Map<Student>(createDto);
        await _repository.CreateAsync(student);
        await _repository.SaveChangesAsync();

        _logger.LogInformation("Created student with ID {StudentId}", student.Id);

        return _mapper.Map<StudentDto>(student);
    }

    public async Task<StudentDto?> UpdateStudentAsync(int id, UpdateStudentDto updateDto)
    {
        var student = await _repository.GetByIdAsync(id);
        if (student == null)
        {
            return null;
        }

        // Check if email already exists for another student
        if (await _repository.EmailExistsAsync(updateDto.Email, id))
        {
            throw new InvalidOperationException("Email already exists");
        }

        _mapper.Map(updateDto, student);
        await _repository.UpdateAsync(student);
        await _repository.SaveChangesAsync();

        _logger.LogInformation("Updated student with ID {StudentId}", id);

        return _mapper.Map<StudentDto>(student);
    }

    public async Task<bool> DeleteStudentAsync(int id)
    {
        var student = await _repository.GetByIdAsync(id);
        if (student == null)
        {
            return false;
        }

        await _repository.DeleteAsync(student);
        await _repository.SaveChangesAsync();

        _logger.LogInformation("Deleted student with ID {StudentId}", id);

        return true;
    }
}
```

---

## 6️⃣ Controllers

```csharp
// Controllers/StudentsController.cs
using Microsoft.AspNetCore.Mvc;
using StudentAPI.Models.DTOs;
using StudentAPI.Helpers;
using StudentAPI.Services.Interfaces;
using System.Text.Json;

namespace StudentAPI.Controllers;

[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]
public class StudentsController : ControllerBase
{
    private readonly IStudentService _studentService;
    private readonly ILogger<StudentsController> _logger;

    public StudentsController(
        IStudentService studentService,
        ILogger<StudentsController> logger)
    {
        _studentService = studentService;
        _logger = logger;
    }

    /// <summary>
    /// Get all students with pagination and filtering
    /// </summary>
    /// <param name="parameters">Query parameters for pagination and filtering</param>
    /// <returns>Paged list of students</returns>
    [HttpGet]
    [ProducesResponseType(typeof(ApiResponse<List<StudentDto>>), StatusCodes.Status200OK)]
    public async Task<ActionResult<ApiResponse<List<StudentDto>>>> GetStudents(
        [FromQuery] QueryParameters parameters)
    {
        try
        {
            var students = await _studentService.GetAllStudentsAsync(parameters);

            // Add pagination metadata to response headers
            var metadata = new PaginationMetadata
            {
                CurrentPage = students.PageNumber,
                PageSize = students.PageSize,
                TotalCount = students.TotalCount,
                TotalPages = students.TotalPages,
                HasPrevious = students.HasPrevious,
                HasNext = students.HasNext
            };

            Response.Headers.Add("X-Pagination", JsonSerializer.Serialize(metadata));

            return Ok(ApiResponse<List<StudentDto>>.SuccessResponse(
                students.Items,
                "Students retrieved successfully"));
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error retrieving students");
            return StatusCode(500, ApiResponse<List<StudentDto>>.ErrorResponse(
                "An error occurred while retrieving students"));
        }
    }

    /// <summary>
    /// Get student by ID
    /// </summary>
    /// <param name="id">Student ID</param>
    /// <returns>Student details</returns>
    [HttpGet("{id}")]
    [ProducesResponseType(typeof(ApiResponse<StudentDto>), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<ApiResponse<StudentDto>>> GetStudent(int id)
    {
        try
        {
            var student = await _studentService.GetStudentByIdAsync(id);

            if (student == null)
            {
                return NotFound(ApiResponse<StudentDto>.ErrorResponse(
                    $"Student with ID {id} not found"));
            }

            return Ok(ApiResponse<StudentDto>.SuccessResponse(
                student,
                "Student retrieved successfully"));
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error retrieving student {StudentId}", id);
            return StatusCode(500, ApiResponse<StudentDto>.ErrorResponse(
                "An error occurred while retrieving the student"));
        }
    }

    /// <summary>
    /// Create a new student
    /// </summary>
    /// <param name="createDto">Student creation data</param>
    /// <returns>Created student</returns>
    [HttpPost]
    [ProducesResponseType(typeof(ApiResponse<StudentDto>), StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<ApiResponse<StudentDto>>> CreateStudent(
        [FromBody] CreateStudentDto createDto)
    {
        try
        {
            var student = await _studentService.CreateStudentAsync(createDto);

            return CreatedAtAction(
                nameof(GetStudent),
                new { id = student.Id },
                ApiResponse<StudentDto>.SuccessResponse(
                    student,
                    "Student created successfully"));
        }
        catch (InvalidOperationException ex)
        {
            return BadRequest(ApiResponse<StudentDto>.ErrorResponse(ex.Message));
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error creating student");
            return StatusCode(500, ApiResponse<StudentDto>.ErrorResponse(
                "An error occurred while creating the student"));
        }
    }

    /// <summary>
    /// Update an existing student
    /// </summary>
    /// <param name="id">Student ID</param>
    /// <param name="updateDto">Student update data</param>
    /// <returns>Updated student</returns>
    [HttpPut("{id}")]
    [ProducesResponseType(typeof(ApiResponse<StudentDto>), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<ApiResponse<StudentDto>>> UpdateStudent(
        int id,
        [FromBody] UpdateStudentDto updateDto)
    {
        try
        {
            var student = await _studentService.UpdateStudentAsync(id, updateDto);

            if (student == null)
            {
                return NotFound(ApiResponse<StudentDto>.ErrorResponse(
                    $"Student with ID {id} not found"));
            }

            return Ok(ApiResponse<StudentDto>.SuccessResponse(
                student,
                "Student updated successfully"));
        }
        catch (InvalidOperationException ex)
        {
            return BadRequest(ApiResponse<StudentDto>.ErrorResponse(ex.Message));
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error updating student {StudentId}", id);
            return StatusCode(500, ApiResponse<StudentDto>.ErrorResponse(
                "An error occurred while updating the student"));
        }
    }

    /// <summary>
    /// Delete a student (soft delete)
    /// </summary>
    /// <param name="id">Student ID</param>
    /// <returns>Success status</returns>
    [HttpDelete("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> DeleteStudent(int id)
    {
        try
        {
            var result = await _studentService.DeleteStudentAsync(id);

            if (!result)
            {
                return NotFound(ApiResponse<object>.ErrorResponse(
                    $"Student with ID {id} not found"));
            }

            return NoContent();
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error deleting student {StudentId}", id);
            return StatusCode(500, ApiResponse<object>.ErrorResponse(
                "An error occurred while deleting the student"));
        }
    }
}
```

---

## 7️⃣ Validation with FluentValidation

```csharp
// Validators/CreateStudentValidator.cs
using FluentValidation;
using StudentAPI.Models.DTOs;

namespace StudentAPI.Validators;

public class CreateStudentValidator : AbstractValidator<CreateStudentDto>
{
    public CreateStudentValidator()
    {
        RuleFor(x => x.FirstName)
            .NotEmpty().WithMessage("First name is required")
            .MaximumLength(50).WithMessage("First name must not exceed 50 characters");

        RuleFor(x => x.LastName)
            .NotEmpty().WithMessage("Last name is required")
            .MaximumLength(50).WithMessage("Last name must not exceed 50 characters");

        RuleFor(x => x.Email)
            .NotEmpty().WithMessage("Email is required")
            .EmailAddress().WithMessage("Invalid email format")
            .MaximumLength(100).WithMessage("Email must not exceed 100 characters");

        RuleFor(x => x.DateOfBirth)
            .NotEmpty().WithMessage("Date of birth is required")
            .LessThan(DateTime.Now).WithMessage("Date of birth must be in the past")
            .GreaterThan(DateTime.Now.AddYears(-100)).WithMessage("Invalid date of birth");
    }
}

// Validators/UpdateStudentValidator.cs
using FluentValidation;
using StudentAPI.Models.DTOs;

namespace StudentAPI.Validators;

public class UpdateStudentValidator : AbstractValidator<UpdateStudentDto>
{
    public UpdateStudentValidator()
    {
        RuleFor(x => x.FirstName)
            .NotEmpty().WithMessage("First name is required")
            .MaximumLength(50).WithMessage("First name must not exceed 50 characters");

        RuleFor(x => x.LastName)
            .NotEmpty().WithMessage("Last name is required")
            .MaximumLength(50).WithMessage("Last name must not exceed 50 characters");

        RuleFor(x => x.Email)
            .NotEmpty().WithMessage("Email is required")
            .EmailAddress().WithMessage("Invalid email format")
            .MaximumLength(100).WithMessage("Email must not exceed 100 characters");

        RuleFor(x => x.DateOfBirth)
            .NotEmpty().WithMessage("Date of birth is required")
            .LessThan(DateTime.Now).WithMessage("Date of birth must be in the past")
            .GreaterThan(DateTime.Now.AddYears(-100)).WithMessage("Invalid date of birth");
    }
}
```

---

## 8️⃣ Global Error Handling

```csharp
// Middleware/ErrorHandlingMiddleware.cs
using System.Net;
using System.Text.Json;
using StudentAPI.Models.DTOs;

namespace StudentAPI.Middleware;

public class ErrorHandlingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ErrorHandlingMiddleware> _logger;

    public ErrorHandlingMiddleware(
        RequestDelegate next,
        ILogger<ErrorHandlingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "An unhandled exception occurred");
            await HandleExceptionAsync(context, ex);
        }
    }

    private static Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        var statusCode = HttpStatusCode.InternalServerError;
        var message = "An internal server error occurred";
        List<string>? errors = null;

        switch (exception)
        {
            case InvalidOperationException:
                statusCode = HttpStatusCode.BadRequest;
                message = exception.Message;
                break;
            case KeyNotFoundException:
                statusCode = HttpStatusCode.NotFound;
                message = exception.Message;
                break;
            case UnauthorizedAccessException:
                statusCode = HttpStatusCode.Unauthorized;
                message = "Unauthorized access";
                break;
            case ArgumentException argumentException:
                statusCode = HttpStatusCode.BadRequest;
                message = argumentException.Message;
                errors = new List<string> { argumentException.ParamName ?? "Unknown parameter" };
                break;
        }

        var response = ApiResponse<object>.ErrorResponse(message, errors);
        
        context.Response.ContentType = "application/json";
        context.Response.StatusCode = (int)statusCode;

        var options = new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        };

        return context.Response.WriteAsync(JsonSerializer.Serialize(response, options));
    }
}
```

---

## 9️⃣ Program.cs Configuration

```csharp
// Program.cs
using Microsoft.EntityFrameworkCore;
using FluentValidation;
using FluentValidation.AspNetCore;
using StudentAPI.Data;
using StudentAPI.Repositories;
using StudentAPI.Repositories.Interfaces;
using StudentAPI.Services;
using StudentAPI.Services.Interfaces;
using StudentAPI.Middleware;
using StudentAPI.Mappings;
using StudentAPI.Validators;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container
builder.Services.AddControllers();

// Database
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// AutoMapper
builder.Services.AddAutoMapper(typeof(MappingProfile));

// FluentValidation
builder.Services.AddFluentValidationAutoValidation();
builder.Services.AddValidatorsFromAssemblyContaining<CreateStudentValidator>();

// Repositories
builder.Services.AddScoped<IStudentRepository, StudentRepository>();

// Services
builder.Services.AddScoped<IStudentService, StudentService>();

// Swagger/OpenAPI
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new() { Title = "Student API", Version = "v1" });
    
    // Enable XML comments
    var xmlFile = $"{System.Reflection.Assembly.GetExecutingAssembly().GetName().Name}.xml";
    var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
    c.IncludeXmlComments(xmlPath);
});

// CORS (if needed)
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAll", builder =>
    {
        builder.AllowAnyOrigin()
               .AllowAnyMethod()
               .AllowAnyHeader();
    });
});

var app = builder.Build();

// Configure the HTTP request pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

// Global error handling middleware
app.UseMiddleware<ErrorHandlingMiddleware>();

app.UseHttpsRedirection();

app.UseCors("AllowAll");

app.UseAuthorization();

app.MapControllers();

app.Run();
```

---

## 🔟 การทดสอบ API

### ตัวอย่าง HTTP Requests

```http
### Get all students (with pagination)
GET https://localhost:7001/api/students?pageNumber=1&pageSize=10&searchTerm=john&sortBy=firstname

### Get student by ID
GET https://localhost:7001/api/students/1

### Create new student
POST https://localhost:7001/api/students
Content-Type: application/json

{
  "firstName": "สมชาย",
  "lastName": "ใจดี",
  "email": "somchai@example.com",
  "dateOfBirth": "2000-05-15",
  "departmentId": 1
}

### Update student
PUT https://localhost:7001/api/students/1
Content-Type: application/json

{
  "firstName": "สมชาย",
  "lastName": "ใจดีมาก",
  "email": "somchai.new@example.com",
  "dateOfBirth": "2000-05-15",
  "departmentId": 2
}

### Delete student
DELETE https://localhost:7001/api/students/1
```

---

## 📋 สรุป

### ✅ **Key Features**
- ✔️ RESTful API design
- ✔️ DTOs สำหรับแยก presentation จาก domain
- ✔️ AutoMapper สำหรับ object mapping
- ✔️ Pagination และ filtering
- ✔️ FluentValidation สำหรับ validation
- ✔️ Global error handling
- ✔️ Swagger documentation
- ✔️ Repository และ Service patterns

### 💡 **Best Practices**
- ใช้ DTOs แทนการ expose entities โดยตรง
- Implement proper error handling
- ใช้ async/await ทุกที่ที่เป็นไปได้
- Add pagination สำหรับ large datasets
- Validate input data
- Log errors และ important events
- Use dependency injection
- Document API ด้วย Swagger

---

## 📚 Navigation

- [← บทที่ 17: Advanced Patterns และ Architectures](17-advanced-patterns.md)
- [→ บทที่ 19: Security Best Practices](19-security.md)
- [📖 กลับไปหน้าหลัก](README.md)
