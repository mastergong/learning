# บทที่ 2: ชนิดข้อมูลและ Data Annotations

## ชนิดข้อมูลที่ EF Core รองรับ

### ชนิดข้อมูลพื้นฐาน (Primitive Types)

| C# Type | SQL Server Type | ตัวอย่าง |
|---------|----------------|----------|
| `int` | `int` | อายุ, จำนวน |
| `long` | `bigint` | ID ขนาดใหญ่ |
| `short` | `smallint` | ตัวเลขเล็ก |
| `byte` | `tinyint` | สถานะ 0-255 |
| `bool` | `bit` | true/false |
| `float` | `real` | ตัวเลขทศนิยม |
| `double` | `float` | ตัวเลขทศนิยมความแม่นยำสูง |
| `decimal` | `decimal(18,2)` | เงิน, ราคา |
| `string` | `nvarchar(max)` | ข้อความ |
| `char` | `nchar(1)` | ตัวอักษรเดียว |
| `DateTime` | `datetime2(7)` | วันที่และเวลา |
| `DateOnly` | `date` | วันที่เท่านั้น (.NET 6+) |
| `TimeOnly` | `time` | เวลาเท่านั้น (.NET 6+) |
| `Guid` | `uniqueidentifier` | UUID |
| `byte[]` | `varbinary(max)` | ไฟล์, รูปภาพ |

### ชนิดข้อมูล Nullable

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Nullable types
    public int? CategoryId { get; set; }  // อาจมีหรือไม่มี Category
    public decimal? Discount { get; set; }  // ส่วนลดอาจไม่มี
    public DateTime? ExpiredDate { get; set; }  // วันหมดอายุอาจไม่มี
}
```

## Data Annotations หลัก

### 1. Key Attributes

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

public class Student
{
    [Key]  // กำหนดเป็น Primary Key
    public int StudentId { get; set; }
    
    [Key]
    [Column(Order = 1)]  // Composite Key - ลำดับที่ 1
    public int SchoolId { get; set; }
    
    [Key]
    [Column(Order = 2)]  // Composite Key - ลำดับที่ 2
    public int ClassroomId { get; set; }
}
```

### 2. String Attributes

```csharp
public class Person
{
    public int Id { get; set; }
    
    [Required(ErrorMessage = "กรุณาใส่ชื่อ")]
    [MaxLength(50, ErrorMessage = "ชื่อต้องไม่เกิน 50 ตัวอักษร")]
    [MinLength(2, ErrorMessage = "ชื่อต้องมีอย่างน้อย 2 ตัวอักษร")]
    public string FirstName { get; set; }
    
    [StringLength(100, MinimumLength = 2)]
    public string LastName { get; set; }
    
    [EmailAddress(ErrorMessage = "รูปแบบอีเมลไม่ถูกต้อง")]
    public string Email { get; set; }
    
    [Phone]
    public string PhoneNumber { get; set; }
    
    [Url]
    public string Website { get; set; }
    
    [RegularExpression(@"^[a-zA-Z0-9]+$", 
        ErrorMessage = "ใช้ได้เฉพาะตัวอักษรและตัวเลขเท่านั้น")]
    public string Username { get; set; }
}
```

### 3. Numeric Attributes

```csharp
public class Product
{
    public int Id { get; set; }
    
    [Range(0, double.MaxValue, ErrorMessage = "ราคาต้องมากกว่า 0")]
    public decimal Price { get; set; }
    
    [Range(0, 100)]
    public int DiscountPercent { get; set; }
    
    [Range(typeof(decimal), "0.00", "999999.99")]
    public decimal Amount { get; set; }
}
```

### 4. DateTime Attributes

```csharp
public class Event
{
    public int Id { get; set; }
    
    [DataType(DataType.Date)]
    [Display(Name = "วันที่เริ่มต้น")]
    public DateTime StartDate { get; set; }
    
    [DataType(DataType.Time)]
    public TimeOnly StartTime { get; set; }  // .NET 6+
    
    [DataType(DataType.DateTime)]
    public DateTime CreatedAt { get; set; }
}
```

### 5. Database Configuration Attributes

```csharp
[Table("Students", Schema = "School")]  // กำหนดชื่อตารางและ Schema
public class Student
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]  // Auto increment
    public int Id { get; set; }
    
    [Column("student_name", TypeName = "varchar(100)")]  // กำหนดชื่อคอลัมน์และชนิด
    public string Name { get; set; }
    
    [Column(TypeName = "decimal(10,2)")]
    public decimal GPA { get; set; }
    
    [NotMapped]  // ไม่สร้างคอลัมน์ในฐานข้อมูล
    public string FullName => $"{FirstName} {LastName}";
    
    [DatabaseGenerated(DatabaseGeneratedOption.Computed)]
    public DateTime CalculatedField { get; set; }  // คำนวณจากฐานข้อมูล
}
```

### 6. Index Attributes

```csharp
[Index(nameof(Email), IsUnique = true)]  // Index ที่ไม่ซ้ำ
[Index(nameof(FirstName), nameof(LastName), Name = "IX_FullName")]  // Composite Index
public class User
{
    public int Id { get; set; }
    
    [Required]
    public string FirstName { get; set; }
    
    [Required]
    public string LastName { get; set; }
    
    [Required]
    public string Email { get; set; }
}
```

## ตัวอย่าง Model ที่สมบูรณ์

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;
using Microsoft.EntityFrameworkCore;

[Table("Products")]
[Index(nameof(SKU), IsUnique = true)]
[Index(nameof(CategoryId), nameof(Name), Name = "IX_Category_Product")]
public class Product
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int Id { get; set; }

    [Required(ErrorMessage = "กรุณาใส่ชื่อสินค้า")]
    [StringLength(200, MinimumLength = 3)]
    [Column("product_name")]
    public string Name { get; set; }

    [Required]
    [StringLength(50)]
    public string SKU { get; set; }  // รหัสสินค้า

    [Column(TypeName = "ntext")]
    public string? Description { get; set; }

    [Required]
    [Range(0, double.MaxValue)]
    [Column(TypeName = "decimal(10,2)")]
    public decimal Price { get; set; }

    [Range(0, 100)]
    public int? DiscountPercent { get; set; }

    [Required]
    [Range(0, int.MaxValue)]
    public int StockQuantity { get; set; }

    // Foreign Key
    [Required]
    [ForeignKey("Category")]
    public int CategoryId { get; set; }

    // Navigation property
    public virtual Category Category { get; set; }

    [DataType(DataType.DateTime)]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public DateTime CreatedDate { get; set; }

    [DataType(DataType.DateTime)]
    public DateTime? UpdatedDate { get; set; }

    [Required]
    [StringLength(100)]
    public string CreatedBy { get; set; }

    [StringLength(100)]
    public string? UpdatedBy { get; set; }

    [NotMapped]
    public decimal FinalPrice => Price - (Price * (DiscountPercent ?? 0) / 100);

    [NotMapped]
    public bool IsInStock => StockQuantity > 0;

    [NotMapped]
    public string DisplayName => $"{Name} ({SKU})";
}

[Table("Categories")]
public class Category
{
    [Key]
    public int Id { get; set; }

    [Required]
    [StringLength(100)]
    public string Name { get; set; }

    [StringLength(500)]
    public string? Description { get; set; }

    // Navigation property
    public virtual ICollection<Product> Products { get; set; } = new List<Product>();
}
```

## Custom Validation Attributes

```csharp
// สร้าง Custom Validation
public class ThaiCitizenIdAttribute : ValidationAttribute
{
    public override bool IsValid(object value)
    {
        if (value == null) return true;
        
        var citizenId = value.ToString();
        if (citizenId.Length != 13) return false;
        
        // ตรวจสอบเลขบัตรประชาชนไทย
        // ... logic การตรวจสอบ
        return true;
    }

    public override string FormatErrorMessage(string name)
    {
        return $"เลขบัตรประชาชนใน {name} ไม่ถูกต้อง";
    }
}

// การใช้งาน
public class Person
{
    public int Id { get; set; }
    
    [ThaiCitizenId]
    [StringLength(13, MinimumLength = 13)]
    public string CitizenId { get; set; }
}
```

## เทคนิคขั้นสูง

### 1. Conditional Validation

```csharp
public class Order
{
    public int Id { get; set; }
    
    public OrderType Type { get; set; }
    
    // ถ้าเป็น Online Order ต้องมี Email
    [RequiredIf("Type", OrderType.Online)]
    [EmailAddress]
    public string? Email { get; set; }
    
    // ถ้าเป็น Offline Order ต้องมี Address
    [RequiredIf("Type", OrderType.Offline)]
    public string? Address { get; set; }
}

public enum OrderType
{
    Online,
    Offline
}
```

### 2. Value Converters สำหรับ Enum

```csharp
public class User
{
    public int Id { get; set; }
    
    [Column(TypeName = "varchar(20)")]
    [EnumDataType(typeof(UserStatus))]
    public UserStatus Status { get; set; }
}

public enum UserStatus
{
    [Display(Name = "ใช้งานอยู่")]
    Active,
    
    [Display(Name = "ถูกระงับ")]
    Suspended,
    
    [Display(Name = "ถูกลบ")]
    Deleted
}
```

## การใช้งาน Data Annotations ร่วมกับ Model Validation

```csharp
public class ProductService
{
    public async Task<bool> CreateProductAsync(Product product)
    {
        // ตรวจสอบ Model Validation
        var context = new ValidationContext(product);
        var results = new List<ValidationResult>();
        
        if (!Validator.TryValidateObject(product, context, results, true))
        {
            foreach (var error in results)
            {
                Console.WriteLine($"Validation Error: {error.ErrorMessage}");
            }
            return false;
        }
        
        // บันทึกข้อมูล
        using var dbContext = new SchoolContext();
        dbContext.Products.Add(product);
        await dbContext.SaveChangesAsync();
        
        return true;
    }
}
```

---

**หมายเหตุ**: Data Annotations เป็นการกำหนดค่าผ่าน Attributes ซึ่งเหมาะสำหรับการกำหนดค่าพื้นฐาน สำหรับการกำหนดค่าที่ซับซ้อนควรใช้ Fluent API แทน