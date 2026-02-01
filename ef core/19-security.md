# บทที่ 19: Security Best Practices สำหรับ EF Core

## 🎯 ภาพรวมของบทนี้

บทนี้จะสอนเกี่ยวกับการรักษาความปลอดภัยเมื่อใช้งาน EF Core รวมถึงการป้องกัน SQL Injection, การจัดการ Connection Strings, การเข้ารหัสข้อมูล และ best practices อื่นๆ

### สิ่งที่คุณจะได้เรียนรู้:
- SQL Injection Prevention
- Connection String Security
- Data Encryption (At Rest & In Transit)
- Row-Level Security
- Audit Logging
- Secure Configuration Management

---

## 1️⃣ SQL Injection Prevention

### 📖 แนวคิด

EF Core ป้องกัน SQL Injection โดยอัตโนมัติเมื่อใช้ LINQ queries แต่ต้องระวังเมื่อใช้ Raw SQL

### ✅ ปลอดภัย: ใช้ LINQ

```csharp
// ✅ SAFE: LINQ queries are automatically parameterized
public async Task<Student?> GetStudentByEmailAsync(string email)
{
    return await _context.Students
        .FirstOrDefaultAsync(s => s.Email == email);
}

// ✅ SAFE: Parameterized queries
public async Task<List<Student>> SearchStudentsAsync(string searchTerm)
{
    return await _context.Students
        .Where(s => s.FirstName.Contains(searchTerm) || 
                   s.LastName.Contains(searchTerm))
        .ToListAsync();
}
```

### ⚠️ อันตราย: Raw SQL แบบ String Concatenation

```csharp
// ❌ DANGEROUS: SQL Injection vulnerability!
public async Task<Student?> GetStudentUnsafeAsync(string email)
{
    // อย่าทำแบบนี้!
    var sql = $"SELECT * FROM Students WHERE Email = '{email}'";
    return await _context.Students
        .FromSqlRaw(sql)
        .FirstOrDefaultAsync();
}

// ถ้า email = "test@test.com' OR '1'='1"
// จะกลายเป็น: SELECT * FROM Students WHERE Email = 'test@test.com' OR '1'='1'
// จะ return นักเรียนทั้งหมด!
```

### ✅ ปลอดภัย: ใช้ Parameterized Queries

```csharp
// ✅ SAFE: Using parameters with FromSqlRaw
public async Task<Student?> GetStudentSafeAsync(string email)
{
    return await _context.Students
        .FromSqlRaw("SELECT * FROM Students WHERE Email = {0}", email)
        .FirstOrDefaultAsync();
}

// ✅ SAFE: Using SqlParameter
using Microsoft.Data.SqlClient;

public async Task<List<Student>> GetStudentsByDepartmentAsync(int departmentId)
{
    var parameter = new SqlParameter("@departmentId", departmentId);
    
    return await _context.Students
        .FromSqlRaw("SELECT * FROM Students WHERE DepartmentId = @departmentId", parameter)
        .ToListAsync();
}

// ✅ SAFE: Using interpolated strings (EF Core 7+)
public async Task<List<Student>> GetActiveStudentsAsync(DateTime afterDate)
{
    return await _context.Students
        .FromSql($"SELECT * FROM Students WHERE CreatedAt > {afterDate} AND IsDeleted = 0")
        .ToListAsync();
}

// EF Core จะแปลงเป็น parameterized query อัตโนมัติ
```

### 🛡️ Input Validation

```csharp
using System.Text.RegularExpressions;

public class SecurityHelper
{
    // Validate email format
    public static bool IsValidEmail(string email)
    {
        if (string.IsNullOrWhiteSpace(email))
            return false;

        var emailRegex = new Regex(@"^[^@\s]+@[^@\s]+\.[^@\s]+$");
        return emailRegex.IsMatch(email);
    }

    // Sanitize input (remove dangerous characters)
    public static string SanitizeInput(string input)
    {
        if (string.IsNullOrWhiteSpace(input))
            return string.Empty;

        // Remove SQL keywords and special characters
        var dangerous = new[] { "'", "\"", ";", "--", "/*", "*/", "xp_", "sp_", "DROP", "DELETE", "INSERT", "UPDATE" };
        var sanitized = input;

        foreach (var keyword in dangerous)
        {
            sanitized = sanitized.Replace(keyword, "", StringComparison.OrdinalIgnoreCase);
        }

        return sanitized.Trim();
    }

    // Validate numeric input
    public static bool IsValidId(string id)
    {
        return int.TryParse(id, out var result) && result > 0;
    }
}

// Usage in Service
public class StudentService
{
    public async Task<Student?> GetStudentByEmailAsync(string email)
    {
        // Validate before querying
        if (!SecurityHelper.IsValidEmail(email))
        {
            throw new ArgumentException("Invalid email format");
        }

        return await _context.Students
            .FirstOrDefaultAsync(s => s.Email == email);
    }
}
```

---

## 2️⃣ Connection String Security

### 🔐 การจัดการ Connection Strings อย่างปลอดภัย

#### ❌ อย่าเก็บใน Source Code

```csharp
// ❌ NEVER DO THIS!
public class ApplicationDbContext : DbContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        // อย่าฮาร์ดโค้ด connection string!
        optionsBuilder.UseSqlServer(
            "Server=myserver;Database=mydb;User Id=sa;Password=MyPassword123!");
    }
}
```

#### ✅ ใช้ Configuration Files

```json
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=SchoolDB;Trusted_Connection=true;"
  }
}

// appsettings.Development.json (ไม่ commit ไปใน git)
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=dev-server;Database=SchoolDB_Dev;User Id=dev_user;Password=DevPass123!;"
  }
}

// appsettings.Production.json (เก็บบน server เท่านั้น)
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=prod-server;Database=SchoolDB;User Id=prod_user;Password=SecurePass456!;"
  }
}
```

#### ✅ ใช้ Environment Variables

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// อ่านจาก Environment Variable
var connectionString = Environment.GetEnvironmentVariable("DB_CONNECTION_STRING")
    ?? builder.Configuration.GetConnectionString("DefaultConnection");

builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString));
```

```bash
# Linux/Mac
export DB_CONNECTION_STRING="Server=prod;Database=SchoolDB;User Id=user;Password=pass;"

# Windows
setx DB_CONNECTION_STRING "Server=prod;Database=SchoolDB;User Id=user;Password=pass;"

# Docker
docker run -e DB_CONNECTION_STRING="..." myapp
```

#### ✅ ใช้ Azure Key Vault

```csharp
// Install packages
// dotnet add package Azure.Identity
// dotnet add package Azure.Extensions.AspNetCore.Configuration.Secrets

using Azure.Identity;
using Azure.Extensions.AspNetCore.Configuration.Secrets;

var builder = WebApplication.CreateBuilder(args);

// Add Azure Key Vault
if (!builder.Environment.IsDevelopment())
{
    var keyVaultEndpoint = new Uri(builder.Configuration["KeyVaultEndpoint"]!);
    builder.Configuration.AddAzureKeyVault(
        keyVaultEndpoint,
        new DefaultAzureCredential());
}

var connectionString = builder.Configuration["ConnectionStrings:DefaultConnection"];
```

#### ✅ ใช้ User Secrets (Development)

```bash
# Initialize user secrets
dotnet user-secrets init

# Set connection string
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=...;Database=...;..."

# List all secrets
dotnet user-secrets list
```

```csharp
// Program.cs (เพิ่มอัตโนมัติใน Development mode)
var builder = WebApplication.CreateBuilder(args);

if (builder.Environment.IsDevelopment())
{
    builder.Configuration.AddUserSecrets<Program>();
}
```

---

## 3️⃣ Data Encryption

### 🔒 Encryption at Rest (เข้ารหัสข้อมูลในฐานข้อมูล)

#### Column-Level Encryption

```csharp
using System.Security.Cryptography;
using System.Text;

public class EncryptionService
{
    private readonly byte[] _key;
    private readonly byte[] _iv;

    public EncryptionService(IConfiguration configuration)
    {
        // อ่าน encryption key จาก secure storage
        var keyString = configuration["Encryption:Key"];
        var ivString = configuration["Encryption:IV"];
        
        _key = Convert.FromBase64String(keyString!);
        _iv = Convert.FromBase64String(ivString!);
    }

    public string Encrypt(string plainText)
    {
        if (string.IsNullOrEmpty(plainText))
            return plainText;

        using var aes = Aes.Create();
        aes.Key = _key;
        aes.IV = _iv;

        var encryptor = aes.CreateEncryptor(aes.Key, aes.IV);

        using var msEncrypt = new MemoryStream();
        using var csEncrypt = new CryptoStream(msEncrypt, encryptor, CryptoStreamMode.Write);
        using (var swEncrypt = new StreamWriter(csEncrypt))
        {
            swEncrypt.Write(plainText);
        }

        return Convert.ToBase64String(msEncrypt.ToArray());
    }

    public string Decrypt(string cipherText)
    {
        if (string.IsNullOrEmpty(cipherText))
            return cipherText;

        using var aes = Aes.Create();
        aes.Key = _key;
        aes.IV = _iv;

        var decryptor = aes.CreateDecryptor(aes.Key, aes.IV);

        using var msDecrypt = new MemoryStream(Convert.FromBase64String(cipherText));
        using var csDecrypt = new CryptoStream(msDecrypt, decryptor, CryptoStreamMode.Read);
        using var srDecrypt = new StreamReader(csDecrypt);
        
        return srDecrypt.ReadToEnd();
    }
}

// Entity with encrypted fields
public class Student
{
    public int Id { get; set; }
    public string FirstName { get; set; } = string.Empty;
    public string LastName { get; set; } = string.Empty;
    
    // Encrypted field (stored as encrypted string in DB)
    public string EncryptedSocialSecurityNumber { get; set; } = string.Empty;
    
    // Not mapped to database
    [NotMapped]
    public string SocialSecurityNumber
    {
        get => _encryptionService?.Decrypt(EncryptedSocialSecurityNumber) ?? string.Empty;
        set => EncryptedSocialSecurityNumber = _encryptionService?.Encrypt(value) ?? string.Empty;
    }

    private IEncryptionService? _encryptionService;

    public void SetEncryptionService(IEncryptionService service)
    {
        _encryptionService = service;
    }
}

// Usage
public class StudentRepository
{
    private readonly ApplicationDbContext _context;
    private readonly EncryptionService _encryptionService;

    public async Task<Student> CreateStudentAsync(Student student)
    {
        student.SetEncryptionService(_encryptionService);
        
        _context.Students.Add(student);
        await _context.SaveChangesAsync();
        
        return student;
    }
}
```

#### SQL Server Always Encrypted

```csharp
// Enable Always Encrypted in connection string
var connectionString = "Server=myserver;Database=mydb;Column Encryption Setting=Enabled;";

// Entity
public class Student
{
    public int Id { get; set; }
    
    // This column will be automatically encrypted/decrypted by SQL Server
    public string SocialSecurityNumber { get; set; } = string.Empty;
}

// Configure in SQL Server
/*
CREATE COLUMN MASTER KEY CMK
WITH (
    KEY_STORE_PROVIDER_NAME = 'MSSQL_CERTIFICATE_STORE',
    KEY_PATH = 'CurrentUser/My/...'
);

CREATE COLUMN ENCRYPTION KEY CEK
WITH VALUES (
    COLUMN_MASTER_KEY = CMK,
    ALGORITHM = 'RSA_OAEP',
    ENCRYPTED_VALUE = 0x...
);

ALTER TABLE Students
ALTER COLUMN SocialSecurityNumber 
ADD ENCRYPTED WITH (
    COLUMN_ENCRYPTION_KEY = CEK,
    ENCRYPTION_TYPE = DETERMINISTIC,
    ALGORITHM = 'AEAD_AES_256_CBC_HMAC_SHA_256'
);
*/
```

### 🔐 Encryption in Transit (SSL/TLS)

```json
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=myserver;Database=mydb;Encrypt=True;TrustServerCertificate=False;"
  }
}
```

---

## 4️⃣ Row-Level Security

### 🛡️ Filtering Data Based on User

```csharp
// Claims-based filtering
public interface ICurrentUserService
{
    int? UserId { get; }
    string? UserRole { get; }
    int? DepartmentId { get; }
}

public class CurrentUserService : ICurrentUserService
{
    private readonly IHttpContextAccessor _httpContextAccessor;

    public CurrentUserService(IHttpContextAccessor httpContextAccessor)
    {
        _httpContextAccessor = httpContextAccessor;
    }

    public int? UserId => int.TryParse(
        _httpContextAccessor.HttpContext?.User?.FindFirst("sub")?.Value, 
        out var id) ? id : null;

    public string? UserRole => 
        _httpContextAccessor.HttpContext?.User?.FindFirst("role")?.Value;

    public int? DepartmentId => int.TryParse(
        _httpContextAccessor.HttpContext?.User?.FindFirst("departmentId")?.Value,
        out var deptId) ? deptId : null;
}

// Global Query Filter in DbContext
public class ApplicationDbContext : DbContext
{
    private readonly ICurrentUserService _currentUserService;

    public ApplicationDbContext(
        DbContextOptions<ApplicationDbContext> options,
        ICurrentUserService currentUserService) : base(options)
    {
        _currentUserService = currentUserService;
    }

    public DbSet<Student> Students { get; set; } = null!;

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Row-level security: Users can only see students in their department
        modelBuilder.Entity<Student>().HasQueryFilter(s =>
            _currentUserService.UserRole == "Admin" || 
            s.DepartmentId == _currentUserService.DepartmentId);

        base.OnModelCreating(modelBuilder);
    }
}

// Bypass filter when needed (Admin operations)
public class StudentRepository
{
    public async Task<List<Student>> GetAllStudentsAsync(bool includeOtherDepartments = false)
    {
        var query = _context.Students.AsQueryable();

        if (includeOtherDepartments)
        {
            // Bypass query filter
            query = query.IgnoreQueryFilters();
        }

        return await query.ToListAsync();
    }
}
```

### 🔐 Multi-Tenancy Security

```csharp
// Tenant-based filtering
public interface ITenantService
{
    string TenantId { get; }
}

public class TenantService : ITenantService
{
    private readonly IHttpContextAccessor _httpContextAccessor;

    public TenantService(IHttpContextAccessor httpContextAccessor)
    {
        _httpContextAccessor = httpContextAccessor;
    }

    public string TenantId => 
        _httpContextAccessor.HttpContext?.User?.FindFirst("tenantId")?.Value 
        ?? throw new UnauthorizedAccessException("Tenant ID not found");
}

// Base entity for multi-tenant
public abstract class TenantEntity
{
    public int Id { get; set; }
    public string TenantId { get; set; } = string.Empty;
}

// Configure global filter
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    foreach (var entityType in modelBuilder.Model.GetEntityTypes())
    {
        if (typeof(TenantEntity).IsAssignableFrom(entityType.ClrType))
        {
            var parameter = Expression.Parameter(entityType.ClrType, "e");
            var property = Expression.Property(parameter, nameof(TenantEntity.TenantId));
            var tenantId = Expression.Constant(_tenantService.TenantId);
            var equals = Expression.Equal(property, tenantId);
            var lambda = Expression.Lambda(equals, parameter);

            modelBuilder.Entity(entityType.ClrType).HasQueryFilter(lambda);
        }
    }
}

// Auto-set TenantId on SaveChanges
public override async Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
{
    foreach (var entry in ChangeTracker.Entries<TenantEntity>())
    {
        if (entry.State == EntityState.Added)
        {
            entry.Entity.TenantId = _tenantService.TenantId;
        }
        else if (entry.State == EntityState.Modified)
        {
            // Prevent changing TenantId
            entry.Property(nameof(TenantEntity.TenantId)).IsModified = false;
        }
    }

    return await base.SaveChangesAsync(cancellationToken);
}
```

---

## 5️⃣ Audit Logging

### 📝 Comprehensive Audit Trail

```csharp
// Audit Log Entity
public class AuditLog
{
    public long Id { get; set; }
    public string UserId { get; set; } = string.Empty;
    public string UserName { get; set; } = string.Empty;
    public string Action { get; set; } = string.Empty; // Create, Update, Delete
    public string EntityType { get; set; } = string.Empty;
    public string EntityId { get; set; } = string.Empty;
    public string? OldValues { get; set; }
    public string? NewValues { get; set; }
    public string? Changes { get; set; }
    public string IpAddress { get; set; } = string.Empty;
    public string UserAgent { get; set; } = string.Empty;
    public DateTime Timestamp { get; set; } = DateTime.UtcNow;
}

// Audit Service
public class AuditService
{
    private readonly ICurrentUserService _currentUserService;
    private readonly IHttpContextAccessor _httpContextAccessor;

    public AuditLog CreateAuditLog(
        EntityEntry entry,
        string action)
    {
        var httpContext = _httpContextAccessor.HttpContext;
        
        var auditLog = new AuditLog
        {
            UserId = _currentUserService.UserId?.ToString() ?? "System",
            UserName = httpContext?.User?.Identity?.Name ?? "System",
            Action = action,
            EntityType = entry.Entity.GetType().Name,
            EntityId = entry.Property("Id").CurrentValue?.ToString() ?? "0",
            IpAddress = httpContext?.Connection?.RemoteIpAddress?.ToString() ?? "Unknown",
            UserAgent = httpContext?.Request?.Headers["User-Agent"].ToString() ?? "Unknown",
            Timestamp = DateTime.UtcNow
        };

        if (action == "Update")
        {
            var changes = new Dictionary<string, object?>();
            var oldValues = new Dictionary<string, object?>();
            var newValues = new Dictionary<string, object?>();

            foreach (var property in entry.Properties)
            {
                if (property.IsModified)
                {
                    changes[property.Metadata.Name] = new
                    {
                        Old = property.OriginalValue,
                        New = property.CurrentValue
                    };
                    oldValues[property.Metadata.Name] = property.OriginalValue;
                    newValues[property.Metadata.Name] = property.CurrentValue;
                }
            }

            auditLog.Changes = System.Text.Json.JsonSerializer.Serialize(changes);
            auditLog.OldValues = System.Text.Json.JsonSerializer.Serialize(oldValues);
            auditLog.NewValues = System.Text.Json.JsonSerializer.Serialize(newValues);
        }
        else if (action == "Create")
        {
            var values = new Dictionary<string, object?>();
            foreach (var property in entry.Properties)
            {
                values[property.Metadata.Name] = property.CurrentValue;
            }
            auditLog.NewValues = System.Text.Json.JsonSerializer.Serialize(values);
        }
        else if (action == "Delete")
        {
            var values = new Dictionary<string, object?>();
            foreach (var property in entry.Properties)
            {
                values[property.Metadata.Name] = property.CurrentValue;
            }
            auditLog.OldValues = System.Text.Json.JsonSerializer.Serialize(values);
        }

        return auditLog;
    }
}

// DbContext with Audit Logging
public class ApplicationDbContext : DbContext
{
    private readonly IAuditService _auditService;

    public DbSet<Student> Students { get; set; } = null!;
    public DbSet<AuditLog> AuditLogs { get; set; } = null!;

    public override async Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
    {
        var auditEntries = new List<AuditLog>();

        foreach (var entry in ChangeTracker.Entries())
        {
            if (entry.Entity is AuditLog || entry.State == EntityState.Detached || entry.State == EntityState.Unchanged)
                continue;

            var action = entry.State switch
            {
                EntityState.Added => "Create",
                EntityState.Modified => "Update",
                EntityState.Deleted => "Delete",
                _ => null
            };

            if (action != null)
            {
                var auditLog = _auditService.CreateAuditLog(entry, action);
                auditEntries.Add(auditLog);
            }
        }

        var result = await base.SaveChangesAsync(cancellationToken);

        if (auditEntries.Any())
        {
            AuditLogs.AddRange(auditEntries);
            await base.SaveChangesAsync(cancellationToken);
        }

        return result;
    }
}
```

---

## 6️⃣ Additional Security Best Practices

### 🔒 Password Hashing (ไม่เก็บ plain text)

```csharp
using System.Security.Cryptography;
using Microsoft.AspNetCore.Cryptography.KeyDerivation;

public class PasswordHasher
{
    public static string HashPassword(string password)
    {
        // Generate salt
        byte[] salt = RandomNumberGenerator.GetBytes(128 / 8);

        // Hash password
        string hashed = Convert.ToBase64String(KeyDerivation.Pbkdf2(
            password: password,
            salt: salt,
            prf: KeyDerivationPrf.HMACSHA256,
            iterationCount: 100000,
            numBytesRequested: 256 / 8));

        // Combine salt and hash
        return $"{Convert.ToBase64String(salt)}.{hashed}";
    }

    public static bool VerifyPassword(string password, string hashedPassword)
    {
        var parts = hashedPassword.Split('.');
        var salt = Convert.FromBase64String(parts[0]);
        var hash = parts[1];

        var testHash = Convert.ToBase64String(KeyDerivation.Pbkdf2(
            password: password,
            salt: salt,
            prf: KeyDerivationPrf.HMACSHA256,
            iterationCount: 100000,
            numBytesRequested: 256 / 8));

        return hash == testHash;
    }
}
```

### 🛡️ Rate Limiting

```csharp
// Install package
// dotnet add package AspNetCoreRateLimit

// Program.cs
builder.Services.AddMemoryCache();
builder.Services.Configure<IpRateLimitOptions>(builder.Configuration.GetSection("IpRateLimiting"));
builder.Services.AddInMemoryRateLimiting();
builder.Services.AddSingleton<IRateLimitConfiguration, RateLimitConfiguration>();

// appsettings.json
{
  "IpRateLimiting": {
    "EnableEndpointRateLimiting": true,
    "StackBlockedRequests": false,
    "GeneralRules": [
      {
        "Endpoint": "*",
        "Period": "1m",
        "Limit": 60
      },
      {
        "Endpoint": "POST:/api/students",
        "Period": "1h",
        "Limit": 100
      }
    ]
  }
}
```

---

## 📋 สรุป Security Checklist

### ✅ **Database Security**
- ✔️ ใช้ parameterized queries เสมอ
- ✔️ ไม่ hardcode connection strings
- ✔️ ใช้ least privilege principle สำหรับ database user
- ✔️ Enable SSL/TLS สำหรับ database connections
- ✔️ เข้ารหัสข้อมูลสำคัญ (passwords, SSN, credit cards)

### ✅ **Application Security**
- ✔️ Validate และ sanitize input ทุกครั้ง
- ✔️ Implement row-level security
- ✔️ ใช้ authentication และ authorization
- ✔️ Enable audit logging
- ✔️ Implement rate limiting

### ✅ **Configuration Security**
- ✔️ ใช้ Azure Key Vault หรือ AWS Secrets Manager
- ✔️ ใช้ User Secrets ใน development
- ✔️ ใช้ Environment Variables ใน production
- ✔️ ไม่ commit secrets ใน git

### ✅ **Monitoring**
- ✔️ Log failed login attempts
- ✔️ Monitor suspicious activities
- ✔️ Set up alerts สำหรับ security events
- ✔️ Regular security audits

---

## 📚 Navigation

- [← บทที่ 18: EF Core กับ Web API](18-web-api-integration.md)
- [📖 กลับไปหน้าหลัก](README.md)
