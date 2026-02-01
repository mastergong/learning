# บทที่ 14: EF Core CLI Commands และ Tools

## รู้จัก EF Core CLI

EF Core CLI (Command Line Interface) เป็นเครื่องมือสำคัญสำหรับจัดการ EF Core ผ่าน command line ทำให้สามารถสร้าง migrations, อัพเดทฐานข้อมูล, และจัดการ schema ได้อย่างมีประสิทธิภาพ

### คำสั่งหลักของ EF Core CLI

```bash
dotnet ef --help           # ดูคำสั่งทั้งหมด
dotnet ef --version        # ตรวจสอบเวอร์ชั่น
```

---

## การติดตั้งและตั้งค่า CLI Tools

### 1. ติดตั้ง Global Tools

```bash
# ติดตั้ง EF Core Tools (Global)
dotnet tool install --global dotnet-ef

# อัพเดท Tools
dotnet tool update --global dotnet-ef

# ถอนการติดตั้ง
dotnet tool uninstall --global dotnet-ef

# ดูรายการ Tools ที่ติดตั้งแล้ว
dotnet tool list --global
```

### 2. เพิ่ม Packages ที่จำเป็น

```bash
# เพิ่ม Design-time tools
dotnet add package Microsoft.EntityFrameworkCore.Design

# เพิ่ม Provider (เลือกตามฐานข้อมูลที่ใช้)
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Sqlite
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
```

### 3. การตรวจสอบการติดตั้ง

```bash
# ตรวจสอบว่าติดตั้งถูกต้องหรือไม่
dotnet ef

# ควรแสดงข้อมูลดังนี้:
# Entity Framework Core .NET Command-line Tools 6.x.x
```

---

## คำสั่ง Database Management

### 1. การสร้างและจัดการ Database

```bash
# สร้างฐานข้อมูล (ตาม DbContext)
dotnet ef database update

# สร้างฐานข้อมูลจาก specific migration
dotnet ef database update 20240201_InitialCreate

# ลบฐานข้อมูล
dotnet ef database drop

# ลบฐานข้อมูลโดยไม่ถาม confirmation
dotnet ef database drop --force

# ดู SQL script ที่จะรัน (ไม่รันจริง)
dotnet ef migrations script
```

### 2. ตัวอย่างการใช้งาน Database Commands

```bash
# ตัวอย่างการสร้างโปรเจคใหม่
mkdir MyEFApp
cd MyEFApp

# สร้าง .NET project
dotnet new console

# เพิ่ม EF Core packages
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Design

# สร้างและอัพเดทฐานข้อมูล
dotnet ef migrations add InitialCreate
dotnet ef database update
```

---

## คำสั่ง Migrations Management

### 1. การสร้าง Migrations

```bash
# สร้าง migration ใหม่
dotnet ef migrations add [MigrationName]

# ตัวอย่าง
dotnet ef migrations add InitialCreate
dotnet ef migrations add AddUserTable
dotnet ef migrations add UpdateProductPrice

# สร้าง migration ใน folder เฉพาะ
dotnet ef migrations add AddUserTable --output-dir Migrations/UserModule

# สร้าง migration สำหรับ DbContext เฉพาะ (ถ้ามีหลายตัว)
dotnet ef migrations add AddUserTable --context UserDbContext
```

### 2. การจัดการ Migrations

```bash
# ดูรายการ migrations ทั้งหมด
dotnet ef migrations list

# ลบ migration ล่าสุด (ที่ยังไม่ได้ apply)
dotnet ef migrations remove

# ลบ migration ล่าสุดโดยไม่ถาม confirmation
dotnet ef migrations remove --force

# ดู SQL script ของ migration
dotnet ef migrations script

# ดู SQL script ระหว่าง migration 2 ตัว
dotnet ef migrations script 20240201_Initial 20240202_AddUser

# สร้าง SQL script ทั้งหมดจากเริ่มต้น
dotnet ef migrations script --from-migration 0
```

### 3. ตัวอย่างการใช้งาน Migrations

```bash
# ตัวอย่าง workflow การพัฒนา
# 1. แก้ไข Models
# 2. สร้าง migration
dotnet ef migrations add "AddEmailToUser"

# 3. ดู SQL ที่จะรันก่อน
dotnet ef migrations script

# 4. Apply migration to database
dotnet ef database update

# หากผิดพลาด สามารถ rollback ได้
dotnet ef database update [PreviousMigrationName]
```

---

## คำสั่ง DbContext Management

### 1. การจัดการ DbContext

```bash
# Scaffold DbContext จาก existing database
dotnet ef dbcontext scaffold [ConnectionString] [Provider]

# ตัวอย่าง SQL Server
dotnet ef dbcontext scaffold "Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer

# ตัวอย่าง SQLite
dotnet ef dbcontext scaffold "Data Source=blog.db" Microsoft.EntityFrameworkCore.Sqlite

# Scaffold with options
dotnet ef dbcontext scaffold "ConnectionString" Microsoft.EntityFrameworkCore.SqlServer --output-dir Models --context BloggingContext --context-dir Data
```

### 2. การจัดการ DbContext Options

```bash
# Scaffold เฉพาะตารางที่ต้องการ
dotnet ef dbcontext scaffold "ConnectionString" Provider --tables Users,Posts,Comments

# Scaffold เฉพาะ schema
dotnet ef dbcontext scaffold "ConnectionString" Provider --schema dbo,admin

# Force overwrite existing files
dotnet ef dbcontext scaffold "ConnectionString" Provider --force

# ไม่สร้าง OnConfiguring method
dotnet ef dbcontext scaffold "ConnectionString" Provider --no-onconfiguring

# ใช้ namespace เฉพาะ
dotnet ef dbcontext scaffold "ConnectionString" Provider --namespace MyApp.Models --context-namespace MyApp.Data
```

### 3. ตัวอย่างการ Scaffold จากฐานข้อมูลจริง

```bash
# ตัวอย่าง: มีฐานข้อมูล HR อยู่แล้ว
dotnet ef dbcontext scaffold "Server=localhost;Database=HRSystem;Trusted_Connection=true;" Microsoft.EntityFrameworkCore.SqlServer --output-dir Models --context HRDbContext --context-dir Data --force --tables Employees,Departments,Positions

# ผลลัพธ์:
# Data/HRDbContext.cs
# Models/Employee.cs
# Models/Department.cs
# Models/Position.cs
```

---

## คำสั่งขั้นสูงและ Options

### 1. การระบุ Project และ Startup Project

```bash
# ระบุ project ที่มี DbContext
dotnet ef migrations add AddUser --project MyApp.Data

# ระบุ startup project (project ที่มี appsettings.json)
dotnet ef migrations add AddUser --startup-project MyApp.Web

# ใช้ทั้งคู่
dotnet ef migrations add AddUser --project MyApp.Data --startup-project MyApp.Web
```

### 2. การระบุ Configuration

```bash
# ใช้ environment เฉพาะ
dotnet ef database update --environment Production

# ใช้ configuration เฉพาะ
dotnet ef database update --configuration Release

# ระบุ connection string โดยตรง
dotnet ef database update --connection "Server=prod-server;Database=MyApp;..."
```

### 3. การใช้งานกับหลาย DbContext

```bash
# เมื่อมี DbContext หลายตัวในโปรเจคเดียวกัน
dotnet ef migrations add AddUser --context UserDbContext
dotnet ef migrations add AddProduct --context ProductDbContext

# อัพเดทฐานข้อมูลสำหรับ context เฉพาะ
dotnet ef database update --context UserDbContext
dotnet ef database update --context ProductDbContext
```

---

## การใช้งาน CLI ใน CI/CD

### 1. Scripts สำหรับ Deployment

```bash
#!/bin/bash
# deploy-database.sh

echo "Starting database deployment..."

# ตรวจสอบ pending migrations
PENDING_MIGRATIONS=$(dotnet ef migrations list --pending --no-color | wc -l)

if [ $PENDING_MIGRATIONS -gt 0 ]; then
    echo "Found $PENDING_MIGRATIONS pending migrations"
    
    # สร้าง SQL script สำหรับการ review
    dotnet ef migrations script --output migrations.sql
    
    # อัพเดทฐานข้อมูล
    dotnet ef database update
    
    echo "Database updated successfully"
else
    echo "No pending migrations"
fi
```

### 2. PowerShell Scripts สำหรับ Windows

```powershell
# Deploy-Database.ps1
param(
    [string]$Environment = "Development",
    [string]$ConnectionString
)

Write-Host "Deploying database for $Environment environment..."

if ($ConnectionString) {
    $env:ConnectionStrings__DefaultConnection = $ConnectionString
}

try {
    # ตรวจสอบ pending migrations
    $pendingMigrations = dotnet ef migrations list --pending --json | ConvertFrom-Json
    
    if ($pendingMigrations.Count -gt 0) {
        Write-Host "Applying $($pendingMigrations.Count) migrations..."
        
        # Generate script first for review
        dotnet ef migrations script --output "deployment-$Environment.sql"
        
        # Apply migrations
        dotnet ef database update
        
        Write-Host "Database deployment completed successfully"
    } else {
        Write-Host "No pending migrations"
    }
}
catch {
    Write-Error "Database deployment failed: $_"
    exit 1
}
```

### 3. Docker Integration

```dockerfile
# Dockerfile สำหรับ Migration
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /app

# Copy project files
COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

# Runtime image
FROM mcr.microsoft.com/dotnet/aspnet:6.0
WORKDIR /app
COPY --from=build /app/out .

# Install EF Core tools
RUN dotnet tool install --global dotnet-ef
ENV PATH="$PATH:/root/.dotnet/tools"

# Migration script
COPY migrate.sh ./
RUN chmod +x migrate.sh

ENTRYPOINT ["./migrate.sh"]
```

```bash
# migrate.sh
#!/bin/bash
echo "Starting database migration..."

# Wait for database to be ready
until dotnet ef database update; do
    echo "Database not ready, waiting..."
    sleep 5
done

echo "Migration completed, starting application..."
exec dotnet YourApp.dll
```

---

## การ Troubleshooting CLI

### 1. ปัญหาที่เจอบ่อยและการแก้ไข

```bash
# Error: "No project was found. Change the current working directory..."
# แก้ไข: ไปที่โฟลเดอร์ที่มี .csproj file
cd MyProject
dotnet ef --help

# Error: "Unable to create an object of type 'XXXContext'"
# แก้ไข: เพิ่ม parameterless constructor หรือ IDesignTimeDbContextFactory
```

```csharp
// วิธีแก้ปัญหา DbContext Design-time error
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext() { }  // สำหรับ design-time
    
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) 
        : base(options) { }
        
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        if (!optionsBuilder.IsConfigured)
        {
            // ใช้สำหรับ design-time เท่านั้น
            optionsBuilder.UseSqlServer("Server=(localdb)\\mssqllocaldb;Database=TempDB;Trusted_Connection=true;");
        }
    }
}
```

### 2. การ Debug CLI Commands

```bash
# เปิด verbose logging
dotnet ef migrations add TestMigration --verbose

# ตรวจสอบ configuration
dotnet ef dbcontext info

# ตรวจสอบ connection
dotnet ef database update --dry-run
```

### 3. การใช้ IDesignTimeDbContextFactory

```csharp
// สำหรับโปรเจคที่ซับซ้อน
public class ApplicationDbContextFactory : IDesignTimeDbContextFactory<ApplicationDbContext>
{
    public ApplicationDbContext CreateDbContext(string[] args)
    {
        var optionsBuilder = new DbContextOptionsBuilder<ApplicationDbContext>();
        
        // ใช้ configuration จาก appsettings.json
        var configuration = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("appsettings.json")
            .Build();
            
        var connectionString = configuration.GetConnectionString("DefaultConnection");
        optionsBuilder.UseSqlServer(connectionString);
        
        return new ApplicationDbContext(optionsBuilder.Options);
    }
}
```

---

## Best Practices สำหรับ CLI Usage

### 1. การตั้งชื่อ Migrations

```bash
# ชื่อที่ดี - บอกถึงการเปลี่ยนแปลงชัดเจน
dotnet ef migrations add "CreateUserTable"
dotnet ef migrations add "AddEmailIndexToUsers"
dotnet ef migrations add "UpdateProductPriceColumn"

# ชื่อที่ไม่ดี
dotnet ef migrations add "Update1"
dotnet ef migrations add "Fix"
dotnet ef migrations add "Changes"
```

### 2. การจัดการ Migration Files

```bash
# แยก migrations ตาม feature
dotnet ef migrations add "UserModule_InitialCreate" --output-dir Migrations/Users
dotnet ef migrations add "ProductModule_InitialCreate" --output-dir Migrations/Products

# สำหรับ multi-tenant
dotnet ef migrations add "TenantSchema_Create" --output-dir Migrations/Tenant
```

### 3. การทำงานกับ Team

```bash
# ก่อน commit ให้ตรวจสอบ
dotnet ef migrations script > review-migration.sql

# หลังจาก pull code ใหม่
dotnet ef database update

# ตรวจสอบสถานะ database vs code
dotnet ef migrations list
```

---

## คำสั่งอ้างอิง (Quick Reference)

### Database Commands
```bash
dotnet ef database update              # อัพเดทฐานข้อมูล
dotnet ef database drop                # ลบฐานข้อมูล
dotnet ef database update {migration}  # อัพเดทไป migration เฉพาะ
```

### Migration Commands
```bash
dotnet ef migrations add {name}        # สร้าง migration ใหม่
dotnet ef migrations list              # ดูรายการ migrations
dotnet ef migrations remove            # ลบ migration ล่าสุด
dotnet ef migrations script            # สร้าง SQL script
```

### DbContext Commands
```bash
dotnet ef dbcontext scaffold {conn} {provider}  # scaffold จาก database
dotnet ef dbcontext info                        # ดูข้อมูล DbContext
dotnet ef dbcontext list                        # ดู DbContext ทั้งหมด
```

### Common Options
```bash
--project {project}           # ระบุ project
--startup-project {project}   # ระบุ startup project
--context {context}           # ระบุ DbContext
--output-dir {directory}      # ระบุ output directory
--force                       # บังคับเขียนทับ
--verbose                     # แสดง detailed output
```

---

## สรุป

EF Core CLI เป็นเครื่องมือที่จำเป็นสำหรับการพัฒนาด้วย EF Core ช่วยให้:

- **จัดการ Database Schema** ผ่าน Migrations อย่างเป็นระบบ
- **Scaffold Models** จากฐานข้อมูลที่มีอยู่แล้วได้รวดเร็ว
- **Automate Deployment** ได้ง่ายผ่าน CI/CD pipelines
- **Debug และ Troubleshoot** ปัญหาได้อย่างมีประสิทธิภาพ

**การเรียนรู้ที่แนะนำ:**
1. เริ่มจากคำสั่งพื้นฐาน (migrations add, database update)
2. เรียนรู้การ scaffold จากฐานข้อมูลที่มีอยู่
3. ฝึกใช้งาน options และ parameters ต่างๆ
4. นำไปประยุกต์ใน workflow การพัฒนาจริง

---

**หัวข้อถัดไป:** [15 - Testing กับ EF Core](15-testing.md)