# บทที่ 9: การเปลี่ยนชื่อ Database และ Schema Operations

## การเปลี่ยนชื่อ Database

### 1. การเปลี่ยนชื่อ Database ด้วย SQL

```csharp
public class DatabaseRenameService
{
    private readonly IConfiguration _configuration;
    private readonly ILogger<DatabaseRenameService> _logger;

    public DatabaseRenameService(IConfiguration configuration, ILogger<DatabaseRenameService> logger)
    {
        _configuration = configuration;
        _logger = logger;
    }

    public async Task RenameDatabaseAsync(string oldName, string newName)
    {
        var connectionString = _configuration.GetConnectionString("DefaultConnection");
        var masterConnectionString = connectionString.Replace($"Database={oldName}", "Database=master");

        using var connection = new SqlConnection(masterConnectionString);
        await connection.OpenAsync();

        try
        {
            // ตรวจสอบว่า Database เก่ามีอยู่หรือไม่
            var checkOldDbSql = "SELECT COUNT(*) FROM sys.databases WHERE name = @oldName";
            using var checkOldCmd = new SqlCommand(checkOldDbSql, connection);
            checkOldCmd.Parameters.AddWithValue("@oldName", oldName);
            var oldDbExists = (int)await checkOldCmd.ExecuteScalarAsync() > 0;

            if (!oldDbExists)
            {
                throw new InvalidOperationException($"Database '{oldName}' does not exist.");
            }

            // ตรวจสอบว่า Database ใหม่มีอยู่แล้วหรือไม่
            var checkNewDbSql = "SELECT COUNT(*) FROM sys.databases WHERE name = @newName";
            using var checkNewCmd = new SqlCommand(checkNewDbSql, connection);
            checkNewCmd.Parameters.AddWithValue("@newName", newName);
            var newDbExists = (int)await checkNewCmd.ExecuteScalarAsync() > 0;

            if (newDbExists)
            {
                throw new InvalidOperationException($"Database '{newName}' already exists.");
            }

            // ปิดการเชื่อมต่อทั้งหมดไปยัง Database เก่า
            var killConnectionsSql = $@"
                USE master;
                DECLARE @kill VARCHAR(MAX) = '';
                SELECT @kill = @kill + 'KILL ' + CONVERT(VARCHAR(5), session_id) + ';'
                FROM sys.dm_exec_sessions
                WHERE database_id = DB_ID('{oldName}')
                    AND session_id <> @@SPID;
                EXEC(@kill);";

            using var killCmd = new SqlCommand(killConnectionsSql, connection);
            await killCmd.ExecuteNonQueryAsync();

            _logger.LogInformation($"Closed all connections to database '{oldName}'");

            // เปลี่ยนชื่อ Database
            var renameSql = $"ALTER DATABASE [{oldName}] MODIFY NAME = [{newName}]";
            using var renameCmd = new SqlCommand(renameSql, connection);
            await renameCmd.ExecuteNonQueryAsync();

            _logger.LogInformation($"Successfully renamed database from '{oldName}' to '{newName}'");

            // อัปเดต Connection String ในไฟล์ config
            await UpdateConnectionStringAsync(oldName, newName);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, $"Failed to rename database from '{oldName}' to '{newName}'");
            throw;
        }
    }

    private async Task UpdateConnectionStringAsync(string oldName, string newName)
    {
        var configPath = "appsettings.json";
        
        if (!File.Exists(configPath))
        {
            _logger.LogWarning("appsettings.json not found. Please manually update the connection string.");
            return;
        }

        var json = await File.ReadAllTextAsync(configPath);
        var updatedJson = json.Replace($"Database={oldName}", $"Database={newName}");

        if (json != updatedJson)
        {
            await File.WriteAllTextAsync(configPath, updatedJson);
            _logger.LogInformation("Updated connection string in appsettings.json");
        }
    }

    public async Task BackupDatabaseAsync(string databaseName, string backupPath)
    {
        var connectionString = _configuration.GetConnectionString("DefaultConnection");
        var masterConnectionString = connectionString.Replace($"Database={databaseName}", "Database=master");

        using var connection = new SqlConnection(masterConnectionString);
        await connection.OpenAsync();

        var backupSql = $@"
            BACKUP DATABASE [{databaseName}] 
            TO DISK = N'{backupPath}' 
            WITH FORMAT, INIT, NAME = N'{databaseName}-Full Database Backup', 
            SKIP, NOREWIND, NOUNLOAD, STATS = 10";

        using var command = new SqlCommand(backupSql, connection);
        command.CommandTimeout = 300; // 5 minutes timeout

        await command.ExecuteNonQueryAsync();
        _logger.LogInformation($"Database '{databaseName}' backed up to '{backupPath}'");
    }
}
```

### 2. การใช้งาน Database Rename Service

```csharp
public class DatabaseManagementController : ControllerBase
{
    private readonly DatabaseRenameService _renameService;

    public DatabaseManagementController(DatabaseRenameService renameService)
    {
        _renameService = renameService;
    }

    [HttpPost("rename-database")]
    public async Task<IActionResult> RenameDatabase([FromBody] RenameDatabaseRequest request)
    {
        try
        {
            // Backup ก่อนเปลี่ยนชื่อ
            var backupPath = Path.Combine(
                Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments),
                "DatabaseBackups",
                $"{request.OldName}_backup_{DateTime.Now:yyyyMMdd_HHmmss}.bak"
            );

            Directory.CreateDirectory(Path.GetDirectoryName(backupPath)!);
            await _renameService.BackupDatabaseAsync(request.OldName, backupPath);

            // เปลี่ยนชื่อ Database
            await _renameService.RenameDatabaseAsync(request.OldName, request.NewName);

            return Ok(new { 
                Success = true, 
                Message = $"Database renamed from '{request.OldName}' to '{request.NewName}' successfully.",
                BackupPath = backupPath
            });
        }
        catch (Exception ex)
        {
            return BadRequest(new { Success = false, Error = ex.Message });
        }
    }
}

public class RenameDatabaseRequest
{
    [Required]
    public string OldName { get; set; }
    
    [Required]
    public string NewName { get; set; }
}
```

## การเปลี่ยนชื่อตาราง

### 1. การเปลี่ยนชื่อตารางด้วย Migration

```csharp
// Migration สำหรับเปลี่ยนชื่อตาราง
public partial class RenameStudentsToStudentProfiles : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // เปลี่ยนชื่อตาราง
        migrationBuilder.RenameTable(
            name: "Students",
            newName: "StudentProfiles");

        // เปลี่ยนชื่อ Index ถ้าจำเป็น
        migrationBuilder.RenameIndex(
            table: "StudentProfiles",
            name: "IX_Students_Email",
            newName: "IX_StudentProfiles_Email");

        migrationBuilder.RenameIndex(
            table: "StudentProfiles", 
            name: "IX_Students_StudentNumber",
            newName: "IX_StudentProfiles_StudentNumber");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        // ย้อนกลับการเปลี่ยนชื่อ
        migrationBuilder.RenameIndex(
            table: "StudentProfiles",
            name: "IX_StudentProfiles_Email",
            newName: "IX_Students_Email");

        migrationBuilder.RenameIndex(
            table: "StudentProfiles",
            name: "IX_StudentProfiles_StudentNumber", 
            newName: "IX_Students_StudentNumber");

        migrationBuilder.RenameTable(
            name: "StudentProfiles",
            newName: "Students");
    }
}

// อัปเดต Entity Configuration
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // เปลี่ยนจาก
    // modelBuilder.Entity<Student>().ToTable("Students");
    
    // เป็น
    modelBuilder.Entity<Student>().ToTable("StudentProfiles");
}
```

### 2. การเปลี่ยนชื่อตารางพร้อม Schema

```csharp
public partial class MoveStudentsToAcademicSchema : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // สร้าง Schema ใหม่
        migrationBuilder.EnsureSchema("Academic");

        // เปลี่ยนชื่อและย้าย Schema
        migrationBuilder.RenameTable(
            name: "Students",
            schema: "dbo",
            newName: "Students",
            newSchema: "Academic");

        // อัปเดต Foreign Key references ถ้าจำเป็น
        migrationBuilder.DropForeignKey(
            name: "FK_Enrollments_Students_StudentId",
            table: "Enrollments");

        migrationBuilder.AddForeignKey(
            name: "FK_Enrollments_Academic_Students_StudentId",
            table: "Enrollments",
            column: "StudentId",
            principalSchema: "Academic",
            principalTable: "Students",
            principalColumn: "Id",
            onDelete: ReferentialAction.Cascade);
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropForeignKey(
            name: "FK_Enrollments_Academic_Students_StudentId",
            table: "Enrollments");

        migrationBuilder.RenameTable(
            name: "Students",
            schema: "Academic",
            newName: "Students",
            newSchema: "dbo");

        migrationBuilder.AddForeignKey(
            name: "FK_Enrollments_Students_StudentId",
            table: "Enrollments",
            column: "StudentId",
            principalTable: "Students", 
            principalColumn: "Id",
            onDelete: ReferentialAction.Cascade);
    }
}
```

## การเปลี่ยนชื่อ Column

### 1. การเปลี่ยนชื่อ Column ด้วย Migration

```csharp
public partial class RenameStudentColumns : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // เปลี่ยนชื่อ Column
        migrationBuilder.RenameColumn(
            table: "Students",
            name: "FirstName",
            newName: "GivenName");

        migrationBuilder.RenameColumn(
            table: "Students",
            name: "LastName", 
            newName: "FamilyName");

        migrationBuilder.RenameColumn(
            table: "Students",
            name: "DateOfBirth",
            newName: "BirthDate");

        // อัปเดต Index ที่เกี่ยวข้อง
        migrationBuilder.RenameIndex(
            table: "Students",
            name: "IX_Students_FirstName_LastName",
            newName: "IX_Students_GivenName_FamilyName");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.RenameIndex(
            table: "Students",
            name: "IX_Students_GivenName_FamilyName",
            newName: "IX_Students_FirstName_LastName");

        migrationBuilder.RenameColumn(
            table: "Students",
            name: "GivenName",
            newName: "FirstName");

        migrationBuilder.RenameColumn(
            table: "Students",
            name: "FamilyName",
            newName: "LastName");

        migrationBuilder.RenameColumn(
            table: "Students", 
            name: "BirthDate",
            newName: "DateOfBirth");
    }
}

// อัปเดต Entity Model
public class Student
{
    public int Id { get; set; }
    
    [Column("GivenName")]  // Map to renamed column
    public string FirstName { get; set; }  // Keep property name for backward compatibility
    
    [Column("FamilyName")]
    public string LastName { get; set; }
    
    [Column("BirthDate")]
    public DateTime DateOfBirth { get; set; }
}

// หรือใช้ Fluent API
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Student>(entity =>
    {
        entity.Property(e => e.FirstName).HasColumnName("GivenName");
        entity.Property(e => e.LastName).HasColumnName("FamilyName");
        entity.Property(e => e.DateOfBirth).HasColumnName("BirthDate");
    });
}
```

### 2. การเปลี่ยนชื่อ Column พร้อมรักษาข้อมูล

```csharp
public partial class RenameEmailToEmailAddress : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // วิธีที่ปลอดภัย: สร้าง column ใหม่ก่อน
        migrationBuilder.AddColumn<string>(
            name: "EmailAddress",
            table: "Students",
            type: "nvarchar(100)",
            maxLength: 100,
            nullable: true);

        // คัดลอกข้อมูลจาก column เก่าไปยัง column ใหม่
        migrationBuilder.Sql("UPDATE Students SET EmailAddress = Email");

        // ตั้งค่า constraint
        migrationBuilder.AlterColumn<string>(
            name: "EmailAddress",
            table: "Students",
            type: "nvarchar(100)",
            maxLength: 100,
            nullable: false,
            oldClrType: typeof(string),
            oldType: "nvarchar(100)",
            oldMaxLength: 100,
            oldNullable: true);

        // สร้าง unique constraint ใหม่
        migrationBuilder.CreateIndex(
            name: "IX_Students_EmailAddress",
            table: "Students",
            column: "EmailAddress",
            unique: true);

        // ลบ index เก่า
        migrationBuilder.DropIndex(
            name: "IX_Students_Email",
            table: "Students");

        // ลบ column เก่า
        migrationBuilder.DropColumn(
            name: "Email",
            table: "Students");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        // ย้อนกลับการเปลี่ยนแปลง
        migrationBuilder.AddColumn<string>(
            name: "Email",
            table: "Students",
            type: "nvarchar(100)",
            maxLength: 100,
            nullable: true);

        migrationBuilder.Sql("UPDATE Students SET Email = EmailAddress");

        migrationBuilder.AlterColumn<string>(
            name: "Email",
            table: "Students",
            type: "nvarchar(100)",
            maxLength: 100,
            nullable: false,
            oldClrType: typeof(string),
            oldType: "nvarchar(100)",
            oldMaxLength: 100,
            oldNullable: true);

        migrationBuilder.CreateIndex(
            name: "IX_Students_Email",
            table: "Students",
            column: "Email",
            unique: true);

        migrationBuilder.DropIndex(
            name: "IX_Students_EmailAddress",
            table: "Students");

        migrationBuilder.DropColumn(
            name: "EmailAddress",
            table: "Students");
    }
}
```

## การจัดการ Schema

### 1. การสร้างและจัดการ Schema

```csharp
public partial class OrganizeTablesIntoSchemas : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // สร้าง Schemas
        migrationBuilder.EnsureSchema("Academic");
        migrationBuilder.EnsureSchema("Administration");
        migrationBuilder.EnsureSchema("Finance");
        migrationBuilder.EnsureSchema("Security");

        // ย้ายตารางไปยัง Schema ที่เหมาะสม
        
        // Academic Schema
        migrationBuilder.RenameTable("Students", newName: "Students", newSchema: "Academic");
        migrationBuilder.RenameTable("Courses", newName: "Courses", newSchema: "Academic");
        migrationBuilder.RenameTable("Enrollments", newName: "Enrollments", newSchema: "Academic");
        migrationBuilder.RenameTable("Departments", newName: "Departments", newSchema: "Academic");

        // Administration Schema  
        migrationBuilder.RenameTable("Instructors", newName: "Instructors", newSchema: "Administration");
        migrationBuilder.RenameTable("Schedules", newName: "Schedules", newSchema: "Administration");

        // Finance Schema
        migrationBuilder.RenameTable("Payments", newName: "Payments", newSchema: "Finance");
        migrationBuilder.RenameTable("Fees", newName: "Fees", newSchema: "Finance");

        // Security Schema
        migrationBuilder.RenameTable("Users", newName: "Users", newSchema: "Security");
        migrationBuilder.RenameTable("Roles", newName: "Roles", newSchema: "Security");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        // ย้อนกลับไปยัง dbo schema
        migrationBuilder.RenameTable("Students", "Academic", newName: "Students");
        migrationBuilder.RenameTable("Courses", "Academic", newName: "Courses");
        migrationBuilder.RenameTable("Enrollments", "Academic", newName: "Enrollments");
        migrationBuilder.RenameTable("Departments", "Academic", newName: "Departments");
        
        migrationBuilder.RenameTable("Instructors", "Administration", newName: "Instructors");
        migrationBuilder.RenameTable("Schedules", "Administration", newName: "Schedules");
        
        migrationBuilder.RenameTable("Payments", "Finance", newName: "Payments");
        migrationBuilder.RenameTable("Fees", "Finance", newName: "Fees");
        
        migrationBuilder.RenameTable("Users", "Security", newName: "Users");
        migrationBuilder.RenameTable("Roles", "Security", newName: "Roles");
    }
}

// อัปเดต Entity Configuration
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // กำหนด Schema สำหรับแต่ละ Entity
    modelBuilder.Entity<Student>().ToTable("Students", "Academic");
    modelBuilder.Entity<Course>().ToTable("Courses", "Academic");
    modelBuilder.Entity<Enrollment>().ToTable("Enrollments", "Academic");
    modelBuilder.Entity<Department>().ToTable("Departments", "Academic");
    
    modelBuilder.Entity<Instructor>().ToTable("Instructors", "Administration");
    modelBuilder.Entity<Payment>().ToTable("Payments", "Finance");
    modelBuilder.Entity<User>().ToTable("Users", "Security");
}
```

### 2. Schema-based DbContext

```csharp
// แยก DbContext ตาม Schema
public class AcademicDbContext : DbContext
{
    public AcademicDbContext(DbContextOptions<AcademicDbContext> options) : base(options) { }

    public DbSet<Student> Students { get; set; }
    public DbSet<Course> Courses { get; set; }
    public DbSet<Enrollment> Enrollments { get; set; }
    public DbSet<Department> Departments { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.HasDefaultSchema("Academic");
        
        // กำหนดค่าเฉพาะสำหรับ Academic entities
        ConfigureAcademicEntities(modelBuilder);
    }

    private void ConfigureAcademicEntities(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Student>(entity =>
        {
            entity.ToTable("Students");
            entity.HasIndex(s => s.StudentNumber).IsUnique();
            entity.HasIndex(s => s.Email).IsUnique();
        });

        modelBuilder.Entity<Course>(entity =>
        {
            entity.ToTable("Courses");
            entity.HasIndex(c => c.CourseCode).IsUnique();
        });
    }
}

public class AdministrationDbContext : DbContext
{
    public AdministrationDbContext(DbContextOptions<AdministrationDbContext> options) : base(options) { }

    public DbSet<Instructor> Instructors { get; set; }
    public DbSet<Schedule> Schedules { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.HasDefaultSchema("Administration");
    }
}

// Registration
builder.Services.AddDbContext<AcademicDbContext>(options =>
    options.UseSqlServer(connectionString));

builder.Services.AddDbContext<AdministrationDbContext>(options =>
    options.UseSqlServer(connectionString));
```

## การ Rename ด้วย Raw SQL

### 1. Rename Service ขั้นสูง

```csharp
public class AdvancedRenameService
{
    private readonly ApplicationDbContext _context;
    private readonly ILogger<AdvancedRenameService> _logger;

    public AdvancedRenameService(ApplicationDbContext context, ILogger<AdvancedRenameService> logger)
    {
        _context = context;
        _logger = logger;
    }

    public async Task RenameTableWithDependenciesAsync(string oldTableName, string newTableName)
    {
        await using var transaction = await _context.Database.BeginTransactionAsync();

        try
        {
            // 1. ค้นหา dependencies
            var dependencies = await GetTableDependenciesAsync(oldTableName);
            
            // 2. ลบ dependencies ชั่วคราว
            await DropDependenciesAsync(dependencies);

            // 3. เปลี่ยนชื่อตาราง
            await _context.Database.ExecuteSqlRawAsync($"EXEC sp_rename '{oldTableName}', '{newTableName}'");

            // 4. สร้าง dependencies ใหม่
            await RecreateDependenciesAsync(dependencies, oldTableName, newTableName);

            await transaction.CommitAsync();
            _logger.LogInformation($"Successfully renamed table from '{oldTableName}' to '{newTableName}'");
        }
        catch (Exception ex)
        {
            await transaction.RollbackAsync();
            _logger.LogError(ex, $"Failed to rename table from '{oldTableName}' to '{newTableName}'");
            throw;
        }
    }

    private async Task<List<TableDependency>> GetTableDependenciesAsync(string tableName)
    {
        var sql = @"
            SELECT 
                fk.name AS ForeignKeyName,
                OBJECT_SCHEMA_NAME(fk.parent_object_id) AS SchemaName,
                OBJECT_NAME(fk.parent_object_id) AS TableName,
                fk.delete_referential_action_desc AS DeleteAction,
                fk.update_referential_action_desc AS UpdateAction
            FROM sys.foreign_keys fk
            WHERE OBJECT_NAME(fk.referenced_object_id) = @tableName";

        var dependencies = new List<TableDependency>();
        
        using var command = _context.Database.GetDbConnection().CreateCommand();
        command.CommandText = sql;
        command.Parameters.Add(new SqlParameter("@tableName", tableName));

        await _context.Database.OpenConnectionAsync();
        using var reader = await command.ExecuteReaderAsync();

        while (await reader.ReadAsync())
        {
            dependencies.Add(new TableDependency
            {
                ForeignKeyName = reader.GetString("ForeignKeyName"),
                SchemaName = reader.GetString("SchemaName"),
                TableName = reader.GetString("TableName"),
                DeleteAction = reader.GetString("DeleteAction"),
                UpdateAction = reader.GetString("UpdateAction")
            });
        }

        return dependencies;
    }

    private async Task DropDependenciesAsync(List<TableDependency> dependencies)
    {
        foreach (var dependency in dependencies)
        {
            var sql = $"ALTER TABLE [{dependency.SchemaName}].[{dependency.TableName}] DROP CONSTRAINT [{dependency.ForeignKeyName}]";
            await _context.Database.ExecuteSqlRawAsync(sql);
            _logger.LogInformation($"Dropped foreign key: {dependency.ForeignKeyName}");
        }
    }

    private async Task RecreateDependenciesAsync(List<TableDependency> dependencies, string oldTableName, string newTableName)
    {
        foreach (var dependency in dependencies)
        {
            // ต้องมีการ query เพิ่มเติมเพื่อได้ข้อมูล column names
            // นี่เป็นตัวอย่างแบบง่าย
            var sql = $@"
                ALTER TABLE [{dependency.SchemaName}].[{dependency.TableName}] 
                ADD CONSTRAINT [{dependency.ForeignKeyName}] 
                FOREIGN KEY (Id) REFERENCES [{newTableName}] (Id)";
            
            await _context.Database.ExecuteSqlRawAsync(sql);
            _logger.LogInformation($"Recreated foreign key: {dependency.ForeignKeyName}");
        }
    }
}

public class TableDependency
{
    public string ForeignKeyName { get; set; }
    public string SchemaName { get; set; }
    public string TableName { get; set; }
    public string DeleteAction { get; set; }
    public string UpdateAction { get; set; }
}
```

### 2. การสำรอง Metadata

```csharp
public class SchemaBackupService
{
    public async Task<SchemaSnapshot> CreateSchemaSnapshotAsync(ApplicationDbContext context)
    {
        var snapshot = new SchemaSnapshot
        {
            CreatedDate = DateTime.Now,
            Tables = await GetTablesAsync(context),
            Indexes = await GetIndexesAsync(context),
            ForeignKeys = await GetForeignKeysAsync(context)
        };

        return snapshot;
    }

    public async Task SaveSchemaSnapshotAsync(SchemaSnapshot snapshot, string filePath)
    {
        var json = JsonSerializer.Serialize(snapshot, new JsonSerializerOptions 
        { 
            WriteIndented = true 
        });
        
        await File.WriteAllTextAsync(filePath, json);
    }

    private async Task<List<TableInfo>> GetTablesAsync(ApplicationDbContext context)
    {
        var sql = @"
            SELECT 
                TABLE_SCHEMA as SchemaName,
                TABLE_NAME as TableName,
                TABLE_TYPE as TableType
            FROM INFORMATION_SCHEMA.TABLES
            WHERE TABLE_TYPE = 'BASE TABLE'
            ORDER BY TABLE_SCHEMA, TABLE_NAME";

        // Implementation จริงจะใช้ ADO.NET หรือ Dapper
        // เพื่อ query ข้อมูล metadata
        return new List<TableInfo>();
    }
}

public class SchemaSnapshot
{
    public DateTime CreatedDate { get; set; }
    public List<TableInfo> Tables { get; set; } = new();
    public List<IndexInfo> Indexes { get; set; } = new();
    public List<ForeignKeyInfo> ForeignKeys { get; set; } = new();
}
```

---

**หมายเหตุ**: การเปลี่ยนชื่อ Database, Table หรือ Column ควรทำอย่างระมัดระวัง โดยเฉพาะใน Production Environment ควรมีการ Backup และทดสอบก่อนเสมอ