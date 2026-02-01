# บทที่ 5: ความสัมพันธ์ (Relationships)

## ประเภทของความสัมพันธ์

EF Core รองรับความสัมพันธ์หลักได้ 4 ประเภท:
1. **One-to-One** (หนึ่งต่อหนึ่ง)
2. **One-to-Many** (หนึ่งต่อหลาย) 
3. **Many-to-Many** (หลายต่อหลาย)
4. **Self-Referencing** (อ้างอิงตัวเอง)

## 1. One-to-One Relationship

ความสัมพันธ์แบบหนึ่งต่อหนึ่ง เช่น Student กับ StudentProfile

### การกำหนดด้วย Data Annotations

```csharp
// Principal Entity (หลัก)
public class Student
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }

    // Navigation property
    public virtual StudentProfile Profile { get; set; }
}

// Dependent Entity (ตาม)
public class StudentProfile
{
    [Key]
    [ForeignKey("Student")]
    public int StudentId { get; set; }  // Primary Key และ Foreign Key

    public string? ParentName { get; set; }
    public string? ParentPhone { get; set; }
    public string? MedicalCondition { get; set; }
    public string? Allergies { get; set; }
    public DateTime CreatedDate { get; set; } = DateTime.Now;

    // Navigation property
    public virtual Student Student { get; set; }
}
```

### การกำหนดด้วย Fluent API

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // One-to-One relationship
    modelBuilder.Entity<Student>()
        .HasOne(s => s.Profile)
        .WithOne(p => p.Student)
        .HasForeignKey<StudentProfile>(p => p.StudentId)
        .OnDelete(DeleteBehavior.Cascade);

    // หรือแบบนี้ก็ได้
    modelBuilder.Entity<StudentProfile>()
        .HasOne(p => p.Student)
        .WithOne(s => s.Profile)
        .HasForeignKey<StudentProfile>(p => p.StudentId);
}
```

## 2. One-to-Many Relationship

ความสัมพันธ์แบบหนึ่งต่อหลาย เช่น Department กับ Students

### การกำหนดด้วย Data Annotations

```csharp
// Principal Entity (หลัก - One)
public class Department
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Code { get; set; }
    public string? Description { get; set; }

    // Navigation property (Collection)
    public virtual ICollection<Student> Students { get; set; } = new List<Student>();
    public virtual ICollection<Course> Courses { get; set; } = new List<Course>();
    public virtual ICollection<Instructor> Instructors { get; set; } = new List<Instructor>();
}

// Dependent Entity (ตาม - Many)
public class Student
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }

    [ForeignKey("Department")]
    public int? DepartmentId { get; set; }  // Foreign Key (nullable)

    // Navigation property
    public virtual Department? Department { get; set; }

    // อื่นๆ
    public virtual ICollection<Enrollment> Enrollments { get; set; } = new List<Enrollment>();
}
```

### การกำหนดด้วย Fluent API

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // One-to-Many relationship
    modelBuilder.Entity<Department>()
        .HasMany(d => d.Students)
        .WithOne(s => s.Department)
        .HasForeignKey(s => s.DepartmentId)
        .OnDelete(DeleteBehavior.SetNull); // เมื่อลบ Department ให้ตั้งค่า DepartmentId เป็น null

    modelBuilder.Entity<Department>()
        .HasMany(d => d.Courses)
        .WithOne(c => c.Department)
        .HasForeignKey(c => c.DepartmentId)
        .OnDelete(DeleteBehavior.Restrict); // ป้องกันการลบ Department ที่มี Course

    // หลายความสัมพันธ์ในตารางเดียวกัน
    modelBuilder.Entity<Course>()
        .HasOne(c => c.Instructor)
        .WithMany(i => i.Courses)
        .HasForeignKey(c => c.InstructorId)
        .OnDelete(DeleteBehavior.SetNull);
}
```

## 3. Many-to-Many Relationship

ความสัมพันธ์แบบหลายต่อหลาย เช่น Student กับ Course ผ่าน Enrollment

### แบบที่ 1: ใช้ Join Entity (แนะนำ)

```csharp
// Entity หลัก 1
public class Student
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }

    // Navigation property ผ่าน Join Entity
    public virtual ICollection<Enrollment> Enrollments { get; set; } = new List<Enrollment>();
}

// Entity หลัก 2  
public class Course
{
    public int Id { get; set; }
    public string CourseCode { get; set; }
    public string Title { get; set; }
    public decimal Credits { get; set; }

    // Navigation property ผ่าน Join Entity
    public virtual ICollection<Enrollment> Enrollments { get; set; } = new List<Enrollment>();
}

// Join Entity
public class Enrollment
{
    [Key, Column(Order = 0)]
    public int StudentId { get; set; }

    [Key, Column(Order = 1)]
    public int CourseId { get; set; }

    // Additional properties
    public DateTime EnrollmentDate { get; set; } = DateTime.Now;
    public string? Grade { get; set; }
    public int? Score { get; set; }
    public EnrollmentStatus Status { get; set; } = EnrollmentStatus.Active;

    // Navigation properties
    [ForeignKey("StudentId")]
    public virtual Student Student { get; set; }

    [ForeignKey("CourseId")]
    public virtual Course Course { get; set; }
}

public enum EnrollmentStatus
{
    [Display(Name = "กำลังเรียน")]
    Active = 1,
    
    [Display(Name = "ถอนวิชา")]
    Dropped = 2,
    
    [Display(Name = "จบแล้ว")]
    Completed = 3
}
```

### การกำหนดด้วย Fluent API (Join Entity)

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // กำหนด Composite Key
    modelBuilder.Entity<Enrollment>()
        .HasKey(e => new { e.StudentId, e.CourseId });

    // Student -> Enrollment
    modelBuilder.Entity<Enrollment>()
        .HasOne(e => e.Student)
        .WithMany(s => s.Enrollments)
        .HasForeignKey(e => e.StudentId)
        .OnDelete(DeleteBehavior.Cascade);

    // Course -> Enrollment
    modelBuilder.Entity<Enrollment>()
        .HasOne(e => e.Course)
        .WithMany(c => c.Enrollments)
        .HasForeignKey(e => e.CourseId)
        .OnDelete(DeleteBehavior.Cascade);

    // กำหนด Index
    modelBuilder.Entity<Enrollment>()
        .HasIndex(e => new { e.StudentId, e.CourseId })
        .IsUnique();
        
    // กำหนดค่า Default
    modelBuilder.Entity<Enrollment>()
        .Property(e => e.EnrollmentDate)
        .HasDefaultValueSql("GETDATE()");
}
```

### แบบที่ 2: Many-to-Many โดยตรง (EF Core 5+)

```csharp
public class Student
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }

    // Direct Many-to-Many
    public virtual ICollection<Course> Courses { get; set; } = new List<Course>();
}

public class Course
{
    public int Id { get; set; }
    public string CourseCode { get; set; }
    public string Title { get; set; }

    // Direct Many-to-Many
    public virtual ICollection<Student> Students { get; set; } = new List<Student>();
}

// Fluent API Configuration
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Student>()
        .HasMany(s => s.Courses)
        .WithMany(c => c.Students)
        .UsingEntity<Dictionary<string, object>>(
            "StudentCourse", // ชื่อ Join Table
            j => j.HasOne<Course>().WithMany().HasForeignKey("CourseId"),
            j => j.HasOne<Student>().WithMany().HasForeignKey("StudentId"),
            j =>
            {
                j.HasKey("StudentId", "CourseId");
                j.ToTable("StudentCourses");
            });
}
```

## 4. Self-Referencing Relationship

ความสัมพันธ์ที่อ้างอิงตัวเอง เช่น Employee กับ Manager

```csharp
public class Employee
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }

    [ForeignKey("Manager")]
    public int? ManagerId { get; set; }  // Self-reference

    // Navigation properties
    public virtual Employee? Manager { get; set; }  // ผู้บังคับบัญชา
    public virtual ICollection<Employee> Subordinates { get; set; } = new List<Employee>(); // ผู้ใต้บังคับบัญชา
}

// Fluent API Configuration
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Employee>()
        .HasOne(e => e.Manager)
        .WithMany(m => m.Subordinates)
        .HasForeignKey(e => e.ManagerId)
        .OnDelete(DeleteBehavior.Restrict); // ป้องกันการลบแบบ Cascade
}
```

### ตัวอย่าง Category Tree

```csharp
public class Category
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string? Description { get; set; }

    [ForeignKey("ParentCategory")]
    public int? ParentCategoryId { get; set; }

    // Self-referencing navigation properties
    public virtual Category? ParentCategory { get; set; }
    public virtual ICollection<Category> SubCategories { get; set; } = new List<Category>();

    // Other relationships
    public virtual ICollection<Product> Products { get; set; } = new List<Product>();

    // Computed properties
    [NotMapped]
    public int Level
    {
        get
        {
            var level = 0;
            var current = ParentCategory;
            while (current != null)
            {
                level++;
                current = current.ParentCategory;
            }
            return level;
        }
    }

    [NotMapped]
    public string FullPath
    {
        get
        {
            var path = new List<string>();
            var current = this;
            while (current != null)
            {
                path.Insert(0, current.Name);
                current = current.ParentCategory;
            }
            return string.Join(" > ", path);
        }
    }
}
```

## การใช้งาน Include และ ThenInclude

### การโหลดข้อมูลที่เกี่ยวข้อง

```csharp
public class StudentService
{
    private readonly ApplicationDbContext _context;

    public StudentService(ApplicationDbContext context)
    {
        _context = context;
    }

    // โหลดข้อมูล Student พร้อม Department
    public async Task<List<Student>> GetStudentsWithDepartmentAsync()
    {
        return await _context.Students
            .Include(s => s.Department)
            .ToListAsync();
    }

    // โหลดข้อมูลหลายชั้น
    public async Task<Student?> GetStudentDetailAsync(int studentId)
    {
        return await _context.Students
            .Include(s => s.Department)
            .Include(s => s.Profile)
            .Include(s => s.Enrollments)
                .ThenInclude(e => e.Course)
                    .ThenInclude(c => c.Instructor)
            .FirstOrDefaultAsync(s => s.Id == studentId);
    }

    // โหลดข้อมูล Department พร้อม Students และ Courses
    public async Task<Department?> GetDepartmentDetailAsync(int departmentId)
    {
        return await _context.Departments
            .Include(d => d.Students)
                .ThenInclude(s => s.Profile)
            .Include(d => d.Courses)
                .ThenInclude(c => c.Enrollments)
                    .ThenInclude(e => e.Student)
            .Include(d => d.Instructors)
            .FirstOrDefaultAsync(d => d.Id == departmentId);
    }

    // การใช้ Projection เพื่อเลือกข้อมูลเฉพาะที่ต้องการ
    public async Task<List<StudentSummaryDto>> GetStudentSummaryAsync()
    {
        return await _context.Students
            .Select(s => new StudentSummaryDto
            {
                Id = s.Id,
                FullName = s.FirstName + " " + s.LastName,
                Email = s.Email,
                DepartmentName = s.Department != null ? s.Department.Name : "ไม่ระบุ",
                EnrolledCoursesCount = s.Enrollments.Count,
                GPA = s.GPA
            })
            .ToListAsync();
    }

    // การใช้ Split Query สำหรับ Multiple Includes
    public async Task<List<Student>> GetStudentsWithCoursesAsync()
    {
        return await _context.Students
            .AsSplitQuery() // แยก Query เพื่อป้องกัน Cartesian Product
            .Include(s => s.Department)
            .Include(s => s.Enrollments)
                .ThenInclude(e => e.Course)
            .Include(s => s.Profile)
            .ToListAsync();
    }
}

public class StudentSummaryDto
{
    public int Id { get; set; }
    public string FullName { get; set; }
    public string Email { get; set; }
    public string DepartmentName { get; set; }
    public int EnrolledCoursesCount { get; set; }
    public decimal? GPA { get; set; }
}
```

## การจัดการ Delete Behavior

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Cascade Delete: ลบ Parent จะลบ Child ด้วย
    modelBuilder.Entity<Student>()
        .HasOne(s => s.Profile)
        .WithOne(p => p.Student)
        .HasForeignKey<StudentProfile>(p => p.StudentId)
        .OnDelete(DeleteBehavior.Cascade);

    // Restrict: ป้องกันการลบ Parent ถ้ามี Child
    modelBuilder.Entity<Department>()
        .HasMany(d => d.Courses)
        .WithOne(c => c.Department)
        .HasForeignKey(c => c.DepartmentId)
        .OnDelete(DeleteBehavior.Restrict);

    // SetNull: ลับ Parent จะตั้งค่า FK ใน Child เป็น null
    modelBuilder.Entity<Department>()
        .HasMany(d => d.Students)
        .WithOne(s => s.Department)
        .HasForeignKey(s => s.DepartmentId)
        .OnDelete(DeleteBehavior.SetNull);

    // NoAction: ไม่ทำอะไร (ต้องจัดการเอง)
    modelBuilder.Entity<Course>()
        .HasOne(c => c.Instructor)
        .WithMany(i => i.Courses)
        .HasForeignKey(c => c.InstructorId)
        .OnDelete(DeleteBehavior.NoAction);
}
```

## การสร้าง Complex Queries

### การ Join หลายตาราง

```csharp
public class ReportService
{
    private readonly ApplicationDbContext _context;

    public ReportService(ApplicationDbContext context)
    {
        _context = context;
    }

    // รายงานจำนวนนักเรียนแต่ละภาค
    public async Task<List<DepartmentStudentReport>> GetDepartmentStudentReportAsync()
    {
        return await _context.Departments
            .Select(d => new DepartmentStudentReport
            {
                DepartmentName = d.Name,
                TotalStudents = d.Students.Count,
                ActiveStudents = d.Students.Count(s => s.Status == StudentStatus.Active),
                MaleStudents = d.Students.Count(s => s.Gender == Gender.Male),
                FemaleStudents = d.Students.Count(s => s.Gender == Gender.Female),
                AverageGPA = d.Students.Where(s => s.GPA.HasValue).Average(s => s.GPA!.Value)
            })
            .ToListAsync();
    }

    // รายงานวิชาที่มีนักเรียนลงทะเบียนมากที่สุด
    public async Task<List<PopularCourseReport>> GetPopularCoursesAsync(int topCount = 10)
    {
        return await _context.Courses
            .Select(c => new PopularCourseReport
            {
                CourseCode = c.CourseCode,
                CourseTitle = c.Title,
                InstructorName = c.Instructor != null 
                    ? c.Instructor.FirstName + " " + c.Instructor.LastName 
                    : "ไม่ระบุ",
                DepartmentName = c.Department.Name,
                EnrolledStudents = c.Enrollments.Count,
                MaxStudents = c.MaxStudents,
                UtilizationRate = c.MaxStudents > 0 
                    ? (double)c.Enrollments.Count / c.MaxStudents * 100 
                    : 0
            })
            .OrderByDescending(c => c.EnrolledStudents)
            .Take(topCount)
            .ToListAsync();
    }

    // รายงานเกรดเฉลี่ยของแต่ละวิชา
    public async Task<List<CourseGradeReport>> GetCourseGradeReportAsync()
    {
        return await _context.Enrollments
            .Where(e => !string.IsNullOrEmpty(e.Grade))
            .GroupBy(e => new { e.CourseId, e.Course.CourseCode, e.Course.Title })
            .Select(g => new CourseGradeReport
            {
                CourseCode = g.Key.CourseCode,
                CourseTitle = g.Key.Title,
                TotalEnrollments = g.Count(),
                AverageScore = g.Where(e => e.Score.HasValue).Average(e => e.Score!.Value),
                PassRate = g.Count(e => e.Grade != "F") * 100.0 / g.Count(),
                GradeDistribution = g.GroupBy(e => e.Grade)
                    .ToDictionary(gg => gg.Key!, gg => gg.Count())
            })
            .ToListAsync();
    }
}

public class DepartmentStudentReport
{
    public string DepartmentName { get; set; }
    public int TotalStudents { get; set; }
    public int ActiveStudents { get; set; }
    public int MaleStudents { get; set; }
    public int FemaleStudents { get; set; }
    public decimal AverageGPA { get; set; }
}

public class PopularCourseReport
{
    public string CourseCode { get; set; }
    public string CourseTitle { get; set; }
    public string InstructorName { get; set; }
    public string DepartmentName { get; set; }
    public int EnrolledStudents { get; set; }
    public int MaxStudents { get; set; }
    public double UtilizationRate { get; set; }
}

public class CourseGradeReport
{
    public string CourseCode { get; set; }
    public string CourseTitle { get; set; }
    public int TotalEnrollments { get; set; }
    public decimal AverageScore { get; set; }
    public double PassRate { get; set; }
    public Dictionary<string, int> GradeDistribution { get; set; }
}
```

---

**หมายเหตุ**: การออกแบบความสัมพันธ์ที่ดีจะช่วยให้การจัดการข้อมูลและการเขียน Query มีประสิทธิภาพมากขึ้น ควรพิจารณาการใช้ Navigation Properties และ Include อย่างระมัดระวังเพื่อหลีกเลี่ยง N+1 Problem