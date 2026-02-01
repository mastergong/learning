# บทที่ 17: Advanced Patterns และ Architectures

## 🎯 ภาพรวมของบทนี้

บทนี้จะสอนเกี่ยวกับ Design Patterns และ Architectural Patterns ขั้นสูงที่ใช้กับ EF Core เพื่อสร้างระบบที่มีความยืดหยุ่น maintainable และ scalable

### สิ่งที่คุณจะได้เรียนรู้:
- CQRS Pattern (Command Query Responsibility Segregation)
- Mediator Pattern with MediatR
- Specification Pattern
- Domain Events
- Repository Pattern ขั้นสูง
- Clean Architecture with EF Core

---

## 1️⃣ CQRS Pattern (Command Query Responsibility Segregation)

### 📖 แนวคิด

CQRS แยกการอ่านข้อมูล (Query) และการเขียนข้อมูล (Command) ออกจากกัน เพื่อให้แต่ละส่วนสามารถ optimize ได้อย่างอิสระ

### ⚙️ โครงสร้าง

```
CQRS/
├── Commands/
│   ├── CreateStudentCommand.cs
│   └── CreateStudentCommandHandler.cs
├── Queries/
│   ├── GetStudentQuery.cs
│   └── GetStudentQueryHandler.cs
└── Models/
    ├── Student.cs (Write Model)
    └── StudentDto.cs (Read Model)
```

### 💻 การ Implement

#### Commands (Write Operations)

```csharp
// Commands/CreateStudentCommand.cs
namespace EFCoreAdvanced.Commands;

public class CreateStudentCommand
{
    public string FirstName { get; set; } = string.Empty;
    public string LastName { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public DateTime DateOfBirth { get; set; }
    public int DepartmentId { get; set; }
}

public class CreateStudentCommandResult
{
    public int StudentId { get; set; }
    public bool Success { get; set; }
    public string Message { get; set; } = string.Empty;
}

// Commands/CreateStudentCommandHandler.cs
using Microsoft.EntityFrameworkCore;

namespace EFCoreAdvanced.Commands;

public class CreateStudentCommandHandler
{
    private readonly ApplicationDbContext _context;
    private readonly ILogger<CreateStudentCommandHandler> _logger;

    public CreateStudentCommandHandler(
        ApplicationDbContext context,
        ILogger<CreateStudentCommandHandler> logger)
    {
        _context = context;
        _logger = logger;
    }

    public async Task<CreateStudentCommandResult> HandleAsync(
        CreateStudentCommand command, 
        CancellationToken cancellationToken = default)
    {
        try
        {
            // Validation
            if (await _context.Students.AnyAsync(
                s => s.Email == command.Email, cancellationToken))
            {
                return new CreateStudentCommandResult
                {
                    Success = false,
                    Message = "Email already exists"
                };
            }

            // Create entity
            var student = new Student
            {
                FirstName = command.FirstName,
                LastName = command.LastName,
                Email = command.Email,
                DateOfBirth = command.DateOfBirth,
                DepartmentId = command.DepartmentId,
                StudentNumber = await GenerateStudentNumberAsync(cancellationToken)
            };

            _context.Students.Add(student);
            await _context.SaveChangesAsync(cancellationToken);

            _logger.LogInformation("Created student {StudentId}", student.Id);

            return new CreateStudentCommandResult
            {
                StudentId = student.Id,
                Success = true,
                Message = "Student created successfully"
            };
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error creating student");
            return new CreateStudentCommandResult
            {
                Success = false,
                Message = ex.Message
            };
        }
    }

    private async Task<string> GenerateStudentNumberAsync(CancellationToken cancellationToken)
    {
        var year = DateTime.Now.Year.ToString().Substring(2);
        var count = await _context.Students.CountAsync(cancellationToken);
        return $"{year}{(count + 1):D5}";
    }
}
```

#### Queries (Read Operations)

```csharp
// Queries/GetStudentQuery.cs
namespace EFCoreAdvanced.Queries;

public class GetStudentQuery
{
    public int? StudentId { get; set; }
    public string? Email { get; set; }
    public int? DepartmentId { get; set; }
}

public class StudentDto
{
    public int Id { get; set; }
    public string FullName { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public int Age { get; set; }
    public string DepartmentName { get; set; } = string.Empty;
    public int EnrollmentCount { get; set; }
}

// Queries/GetStudentQueryHandler.cs
using Microsoft.EntityFrameworkCore;

namespace EFCoreAdvanced.Queries;

public class GetStudentQueryHandler
{
    private readonly ApplicationDbContext _context;

    public GetStudentQueryHandler(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<StudentDto?> GetByIdAsync(int id, CancellationToken cancellationToken = default)
    {
        // Optimized read-only query
        return await _context.Students
            .AsNoTracking()
            .Where(s => s.Id == id)
            .Select(s => new StudentDto
            {
                Id = s.Id,
                FullName = s.FirstName + " " + s.LastName,
                Email = s.Email,
                Age = DateTime.Now.Year - s.DateOfBirth.Year,
                DepartmentName = s.Department!.Name,
                EnrollmentCount = s.Enrollments.Count
            })
            .FirstOrDefaultAsync(cancellationToken);
    }

    public async Task<List<StudentDto>> GetAllAsync(
        GetStudentQuery query, 
        CancellationToken cancellationToken = default)
    {
        var queryable = _context.Students.AsNoTracking().AsQueryable();

        if (query.StudentId.HasValue)
            queryable = queryable.Where(s => s.Id == query.StudentId.Value);

        if (!string.IsNullOrEmpty(query.Email))
            queryable = queryable.Where(s => s.Email.Contains(query.Email));

        if (query.DepartmentId.HasValue)
            queryable = queryable.Where(s => s.DepartmentId == query.DepartmentId.Value);

        return await queryable
            .Select(s => new StudentDto
            {
                Id = s.Id,
                FullName = s.FirstName + " " + s.LastName,
                Email = s.Email,
                Age = DateTime.Now.Year - s.DateOfBirth.Year,
                DepartmentName = s.Department!.Name,
                EnrollmentCount = s.Enrollments.Count
            })
            .ToListAsync(cancellationToken);
    }
}
```

---

## 2️⃣ Mediator Pattern with MediatR

### 📦 ติดตั้ง MediatR

```bash
dotnet add package MediatR
dotnet add package MediatR.Extensions.Microsoft.DependencyInjection
```

### 💻 การ Implement

```csharp
// Commands/CreateStudentCommand.cs (with MediatR)
using MediatR;

namespace EFCoreAdvanced.Commands;

public class CreateStudentCommand : IRequest<CreateStudentCommandResult>
{
    public string FirstName { get; set; } = string.Empty;
    public string LastName { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public DateTime DateOfBirth { get; set; }
    public int DepartmentId { get; set; }
}

// Commands/CreateStudentCommandHandler.cs (with MediatR)
using MediatR;
using Microsoft.EntityFrameworkCore;

namespace EFCoreAdvanced.Commands;

public class CreateStudentCommandHandler 
    : IRequestHandler<CreateStudentCommand, CreateStudentCommandResult>
{
    private readonly ApplicationDbContext _context;
    private readonly ILogger<CreateStudentCommandHandler> _logger;
    private readonly IMediator _mediator;

    public CreateStudentCommandHandler(
        ApplicationDbContext context,
        ILogger<CreateStudentCommandHandler> logger,
        IMediator mediator)
    {
        _context = context;
        _logger = logger;
        _mediator = mediator;
    }

    public async Task<CreateStudentCommandResult> Handle(
        CreateStudentCommand request, 
        CancellationToken cancellationToken)
    {
        var student = new Student
        {
            FirstName = request.FirstName,
            LastName = request.LastName,
            Email = request.Email,
            DateOfBirth = request.DateOfBirth,
            DepartmentId = request.DepartmentId
        };

        _context.Students.Add(student);
        await _context.SaveChangesAsync(cancellationToken);

        // Publish domain event
        await _mediator.Publish(
            new StudentCreatedEvent { StudentId = student.Id }, 
            cancellationToken);

        return new CreateStudentCommandResult
        {
            StudentId = student.Id,
            Success = true,
            Message = "Student created successfully"
        };
    }
}

// Queries/GetStudentQuery.cs (with MediatR)
using MediatR;

namespace EFCoreAdvanced.Queries;

public class GetStudentByIdQuery : IRequest<StudentDto?>
{
    public int StudentId { get; set; }
}

public class GetStudentByIdQueryHandler 
    : IRequestHandler<GetStudentByIdQuery, StudentDto?>
{
    private readonly ApplicationDbContext _context;

    public GetStudentByIdQueryHandler(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<StudentDto?> Handle(
        GetStudentByIdQuery request, 
        CancellationToken cancellationToken)
    {
        return await _context.Students
            .AsNoTracking()
            .Where(s => s.Id == request.StudentId)
            .Select(s => new StudentDto
            {
                Id = s.Id,
                FullName = s.FirstName + " " + s.LastName,
                Email = s.Email,
                Age = DateTime.Now.Year - s.DateOfBirth.Year,
                DepartmentName = s.Department!.Name
            })
            .FirstOrDefaultAsync(cancellationToken);
    }
}

// Program.cs - Registration
builder.Services.AddMediatR(cfg => 
    cfg.RegisterServicesFromAssembly(typeof(Program).Assembly));

// Usage Example
public class StudentService
{
    private readonly IMediator _mediator;

    public StudentService(IMediator mediator)
    {
        _mediator = mediator;
    }

    public async Task<CreateStudentCommandResult> CreateStudentAsync(CreateStudentCommand command)
    {
        return await _mediator.Send(command);
    }

    public async Task<StudentDto?> GetStudentAsync(int id)
    {
        return await _mediator.Send(new GetStudentByIdQuery { StudentId = id });
    }
}
```

---

## 3️⃣ Specification Pattern

### 📖 แนวคิด

Specification Pattern ช่วยให้เราสามารถสร้าง query logic ที่ reusable และ testable โดยแยก business rules ออกจาก repository

### 💻 การ Implement

```csharp
// Specifications/ISpecification.cs
using System.Linq.Expressions;

namespace EFCoreAdvanced.Specifications;

public interface ISpecification<T>
{
    Expression<Func<T, bool>>? Criteria { get; }
    List<Expression<Func<T, object>>> Includes { get; }
    List<string> IncludeStrings { get; }
    Expression<Func<T, object>>? OrderBy { get; }
    Expression<Func<T, object>>? OrderByDescending { get; }
    int Take { get; }
    int Skip { get; }
    bool IsPagingEnabled { get; }
}

// Specifications/BaseSpecification.cs
using System.Linq.Expressions;

namespace EFCoreAdvanced.Specifications;

public abstract class BaseSpecification<T> : ISpecification<T>
{
    protected BaseSpecification()
    {
    }

    protected BaseSpecification(Expression<Func<T, bool>> criteria)
    {
        Criteria = criteria;
    }

    public Expression<Func<T, bool>>? Criteria { get; private set; }
    public List<Expression<Func<T, object>>> Includes { get; } = new();
    public List<string> IncludeStrings { get; } = new();
    public Expression<Func<T, object>>? OrderBy { get; private set; }
    public Expression<Func<T, object>>? OrderByDescending { get; private set; }
    public int Take { get; private set; }
    public int Skip { get; private set; }
    public bool IsPagingEnabled { get; private set; }

    protected virtual void AddInclude(Expression<Func<T, object>> includeExpression)
    {
        Includes.Add(includeExpression);
    }

    protected virtual void AddInclude(string includeString)
    {
        IncludeStrings.Add(includeString);
    }

    protected virtual void ApplyPaging(int skip, int take)
    {
        Skip = skip;
        Take = take;
        IsPagingEnabled = true;
    }

    protected virtual void ApplyOrderBy(Expression<Func<T, object>> orderByExpression)
    {
        OrderBy = orderByExpression;
    }

    protected virtual void ApplyOrderByDescending(Expression<Func<T, object>> orderByDescExpression)
    {
        OrderByDescending = orderByDescExpression;
    }
}

// Specifications/StudentSpecifications.cs
using System.Linq.Expressions;

namespace EFCoreAdvanced.Specifications;

// Specification: Students by Department
public class StudentsByDepartmentSpecification : BaseSpecification<Student>
{
    public StudentsByDepartmentSpecification(int departmentId)
        : base(s => s.DepartmentId == departmentId)
    {
        AddInclude(s => s.Department);
        AddInclude(s => s.Enrollments);
        ApplyOrderBy(s => s.LastName);
    }
}

// Specification: Active Students with Enrollments
public class ActiveStudentsWithEnrollmentsSpecification : BaseSpecification<Student>
{
    public ActiveStudentsWithEnrollmentsSpecification()
        : base(s => s.IsDeleted == false && s.Enrollments.Any())
    {
        AddInclude(s => s.Department);
        AddInclude("Enrollments.Course");
    }
}

// Specification: Students by Email Domain
public class StudentsByEmailDomainSpecification : BaseSpecification<Student>
{
    public StudentsByEmailDomainSpecification(string domain)
        : base(s => s.Email.EndsWith(domain))
    {
        AddInclude(s => s.Department);
    }
}

// Specification: Paginated Students
public class PaginatedStudentsSpecification : BaseSpecification<Student>
{
    public PaginatedStudentsSpecification(int pageNumber, int pageSize)
        : base(s => !s.IsDeleted)
    {
        ApplyPaging((pageNumber - 1) * pageSize, pageSize);
        ApplyOrderBy(s => s.Id);
        AddInclude(s => s.Department);
    }
}

// Repositories/SpecificationEvaluator.cs
using Microsoft.EntityFrameworkCore;

namespace EFCoreAdvanced.Repositories;

public static class SpecificationEvaluator<T> where T : class
{
    public static IQueryable<T> GetQuery(
        IQueryable<T> inputQuery, 
        ISpecification<T> specification)
    {
        var query = inputQuery;

        // Apply criteria (where clause)
        if (specification.Criteria != null)
        {
            query = query.Where(specification.Criteria);
        }

        // Apply includes
        query = specification.Includes.Aggregate(
            query, 
            (current, include) => current.Include(include));

        // Apply string-based includes
        query = specification.IncludeStrings.Aggregate(
            query, 
            (current, include) => current.Include(include));

        // Apply ordering
        if (specification.OrderBy != null)
        {
            query = query.OrderBy(specification.OrderBy);
        }
        else if (specification.OrderByDescending != null)
        {
            query = query.OrderByDescending(specification.OrderByDescending);
        }

        // Apply paging
        if (specification.IsPagingEnabled)
        {
            query = query.Skip(specification.Skip).Take(specification.Take);
        }

        return query;
    }
}

// Repositories/IGenericRepository.cs (with Specification support)
namespace EFCoreAdvanced.Repositories;

public interface IGenericRepository<T> where T : class
{
    Task<T?> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<IEnumerable<T>> GetAsync(ISpecification<T> spec);
    Task<T?> GetSingleAsync(ISpecification<T> spec);
    Task<int> CountAsync(ISpecification<T> spec);
    Task AddAsync(T entity);
    void Update(T entity);
    void Remove(T entity);
}

// Repositories/GenericRepository.cs (with Specification support)
using Microsoft.EntityFrameworkCore;

namespace EFCoreAdvanced.Repositories;

public class GenericRepository<T> : IGenericRepository<T> where T : class
{
    private readonly ApplicationDbContext _context;
    private readonly DbSet<T> _dbSet;

    public GenericRepository(ApplicationDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }

    public async Task<T?> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }

    public async Task<IEnumerable<T>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }

    public async Task<IEnumerable<T>> GetAsync(ISpecification<T> spec)
    {
        return await ApplySpecification(spec).ToListAsync();
    }

    public async Task<T?> GetSingleAsync(ISpecification<T> spec)
    {
        return await ApplySpecification(spec).FirstOrDefaultAsync();
    }

    public async Task<int> CountAsync(ISpecification<T> spec)
    {
        return await ApplySpecification(spec).CountAsync();
    }

    public async Task AddAsync(T entity)
    {
        await _dbSet.AddAsync(entity);
    }

    public void Update(T entity)
    {
        _dbSet.Update(entity);
    }

    public void Remove(T entity)
    {
        _dbSet.Remove(entity);
    }

    private IQueryable<T> ApplySpecification(ISpecification<T> spec)
    {
        return SpecificationEvaluator<T>.GetQuery(_dbSet.AsQueryable(), spec);
    }
}

// Usage Example
public class StudentService
{
    private readonly IGenericRepository<Student> _studentRepository;
    private readonly IUnitOfWork _unitOfWork;

    public StudentService(
        IGenericRepository<Student> studentRepository,
        IUnitOfWork unitOfWork)
    {
        _studentRepository = studentRepository;
        _unitOfWork = unitOfWork;
    }

    public async Task<IEnumerable<Student>> GetStudentsByDepartmentAsync(int departmentId)
    {
        var spec = new StudentsByDepartmentSpecification(departmentId);
        return await _studentRepository.GetAsync(spec);
    }

    public async Task<IEnumerable<Student>> GetActiveStudentsWithEnrollmentsAsync()
    {
        var spec = new ActiveStudentsWithEnrollmentsSpecification();
        return await _studentRepository.GetAsync(spec);
    }

    public async Task<IEnumerable<Student>> GetPaginatedStudentsAsync(int page, int pageSize)
    {
        var spec = new PaginatedStudentsSpecification(page, pageSize);
        return await _studentRepository.GetAsync(spec);
    }
}
```

---

## 4️⃣ Domain Events

### 📖 แนวคิด

Domain Events ช่วยให้ different parts ของ application สามารถ respond ต่อ events ที่เกิดขึ้นในระบบได้ โดยไม่ต้อง tightly coupled

### 💻 การ Implement

```csharp
// Events/IDomainEvent.cs
using MediatR;

namespace EFCoreAdvanced.Events;

public interface IDomainEvent : INotification
{
    DateTime OccurredOn { get; }
}

// Events/StudentCreatedEvent.cs
namespace EFCoreAdvanced.Events;

public class StudentCreatedEvent : IDomainEvent
{
    public int StudentId { get; set; }
    public DateTime OccurredOn { get; set; } = DateTime.UtcNow;
}

public class StudentUpdatedEvent : IDomainEvent
{
    public int StudentId { get; set; }
    public string UpdatedBy { get; set; } = string.Empty;
    public DateTime OccurredOn { get; set; } = DateTime.UtcNow;
}

public class StudentEnrolledEvent : IDomainEvent
{
    public int StudentId { get; set; }
    public int CourseId { get; set; }
    public DateTime OccurredOn { get; set; } = DateTime.UtcNow;
}

// Event Handlers
using MediatR;

namespace EFCoreAdvanced.Events.Handlers;

// Handler 1: Send Welcome Email
public class SendWelcomeEmailHandler : INotificationHandler<StudentCreatedEvent>
{
    private readonly ILogger<SendWelcomeEmailHandler> _logger;
    private readonly IEmailService _emailService;

    public SendWelcomeEmailHandler(
        ILogger<SendWelcomeEmailHandler> logger,
        IEmailService emailService)
    {
        _logger = logger;
        _emailService = emailService;
    }

    public async Task Handle(StudentCreatedEvent notification, CancellationToken cancellationToken)
    {
        _logger.LogInformation("Sending welcome email to student {StudentId}", notification.StudentId);
        
        // Send email logic
        await _emailService.SendWelcomeEmailAsync(notification.StudentId);
    }
}

// Handler 2: Create Student Portal Account
public class CreateStudentPortalAccountHandler : INotificationHandler<StudentCreatedEvent>
{
    private readonly ILogger<CreateStudentPortalAccountHandler> _logger;
    private readonly IPortalService _portalService;

    public CreateStudentPortalAccountHandler(
        ILogger<CreateStudentPortalAccountHandler> logger,
        IPortalService portalService)
    {
        _logger = logger;
        _portalService = portalService;
    }

    public async Task Handle(StudentCreatedEvent notification, CancellationToken cancellationToken)
    {
        _logger.LogInformation("Creating portal account for student {StudentId}", notification.StudentId);
        
        await _portalService.CreateAccountAsync(notification.StudentId);
    }
}

// Handler 3: Log Audit Trail
public class AuditLogHandler : 
    INotificationHandler<StudentCreatedEvent>,
    INotificationHandler<StudentUpdatedEvent>
{
    private readonly ApplicationDbContext _context;
    private readonly ILogger<AuditLogHandler> _logger;

    public AuditLogHandler(
        ApplicationDbContext context,
        ILogger<AuditLogHandler> logger)
    {
        _context = context;
        _logger = logger;
    }

    public async Task Handle(StudentCreatedEvent notification, CancellationToken cancellationToken)
    {
        await LogAuditAsync("StudentCreated", notification.StudentId, cancellationToken);
    }

    public async Task Handle(StudentUpdatedEvent notification, CancellationToken cancellationToken)
    {
        await LogAuditAsync("StudentUpdated", notification.StudentId, cancellationToken);
    }

    private async Task LogAuditAsync(string action, int studentId, CancellationToken cancellationToken)
    {
        var auditLog = new AuditLog
        {
            Action = action,
            EntityType = "Student",
            EntityId = studentId,
            Timestamp = DateTime.UtcNow
        };

        _context.AuditLogs.Add(auditLog);
        await _context.SaveChangesAsync(cancellationToken);
        
        _logger.LogInformation("Audit log created: {Action} for Student {StudentId}", action, studentId);
    }
}

// Models/BaseEntity.cs (with Domain Events)
namespace EFCoreAdvanced.Models;

public abstract class BaseEntity
{
    private readonly List<IDomainEvent> _domainEvents = new();

    public int Id { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public DateTime? UpdatedAt { get; set; }
    
    public IReadOnlyCollection<IDomainEvent> DomainEvents => _domainEvents.AsReadOnly();

    public void AddDomainEvent(IDomainEvent eventItem)
    {
        _domainEvents.Add(eventItem);
    }

    public void RemoveDomainEvent(IDomainEvent eventItem)
    {
        _domainEvents.Remove(eventItem);
    }

    public void ClearDomainEvents()
    {
        _domainEvents.Clear();
    }
}

// Data/ApplicationDbContext.cs (with Domain Events)
using MediatR;
using Microsoft.EntityFrameworkCore;

namespace EFCoreAdvanced.Data;

public class ApplicationDbContext : DbContext
{
    private readonly IMediator _mediator;

    public ApplicationDbContext(
        DbContextOptions<ApplicationDbContext> options,
        IMediator mediator) : base(options)
    {
        _mediator = mediator;
    }

    public DbSet<Student> Students { get; set; } = null!;
    public DbSet<AuditLog> AuditLogs { get; set; } = null!;

    public override async Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
    {
        // Get all entities with domain events
        var entitiesWithEvents = ChangeTracker.Entries<BaseEntity>()
            .Select(e => e.Entity)
            .Where(e => e.DomainEvents.Any())
            .ToArray();

        // Get all domain events
        var domainEvents = entitiesWithEvents
            .SelectMany(e => e.DomainEvents)
            .ToList();

        // Clear domain events from entities
        foreach (var entity in entitiesWithEvents)
        {
            entity.ClearDomainEvents();
        }

        // Save changes to database
        var result = await base.SaveChangesAsync(cancellationToken);

        // Publish domain events AFTER saving
        foreach (var domainEvent in domainEvents)
        {
            await _mediator.Publish(domainEvent, cancellationToken);
        }

        return result;
    }
}
```

---

## 5️⃣ Clean Architecture with EF Core

### 📁 โครงสร้าง Clean Architecture

```
CleanArchitecture/
├── 📁 Domain/                      # Core Business Logic
│   ├── Entities/
│   │   ├── Student.cs
│   │   └── Course.cs
│   ├── Interfaces/
│   │   ├── IStudentRepository.cs
│   │   └── IUnitOfWork.cs
│   └── Events/
│       └── StudentCreatedEvent.cs
│
├── 📁 Application/                 # Use Cases
│   ├── Commands/
│   │   └── CreateStudentCommand.cs
│   ├── Queries/
│   │   └── GetStudentQuery.cs
│   ├── DTOs/
│   │   └── StudentDto.cs
│   └── Interfaces/
│       └── IEmailService.cs
│
├── 📁 Infrastructure/              # External Concerns
│   ├── Data/
│   │   ├── ApplicationDbContext.cs
│   │   ├── Repositories/
│   │   │   └── StudentRepository.cs
│   │   └── Configurations/
│   │       └── StudentConfiguration.cs
│   └── Services/
│       └── EmailService.cs
│
└── 📁 WebAPI/                     # Presentation Layer
    ├── Controllers/
    │   └── StudentsController.cs
    └── Program.cs
```

### 💡 Dependency Rule

```
WebAPI → Application → Domain
      ↓               ↑
Infrastructure -------┘

- Domain ไม่ depend ใคร (Core Business Logic)
- Application depend Domain (Use Cases)
- Infrastructure depend Domain (Implementation)
- WebAPI depend ทุกอย่าง (Composition Root)
```

---

## 📋 สรุป

### ✅ **CQRS Pattern**
- แยก Read และ Write operations
- Optimize แต่ละส่วนได้อย่างอิสระ
- ใช้ AsNoTracking() สำหรับ queries

### ✅ **Mediator Pattern**
- Loose coupling ระหว่าง components
- ใช้ MediatR library
- รองรับ Commands, Queries, และ Events

### ✅ **Specification Pattern**
- Reusable query logic
- Testable business rules
- Type-safe queries

### ✅ **Domain Events**
- Decouple business logic
- Multiple handlers per event
- Async event processing

### ✅ **Clean Architecture**
- Separation of Concerns
- Testable และ Maintainable
- Independent of frameworks

---

## 📚 Navigation

- [← บทที่ 16: โครงสร้าง Project และ Configuration](16-project-structure.md)
- [→ บทที่ 18: Web API Integration](18-web-api-integration.md)
- [📖 กลับไปหน้าหลัก](README.md)
