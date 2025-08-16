# Design 01 — SOLID Principles หลักการออกแบบที่มั่นคงใน C#

## 🎯 เป้าหมายการเรียนรู้
เมื่อเสร็จสิ้นบทเรียนนี้ คุณจะสามารถ:
- เข้าใจหลักการ SOLID ทั้ง 5 ข้อ และความสำคัญต่อการออกแบบซอฟต์แวร์
- ระบุปัญหาในโค้ดที่ละเมิดหลักการ SOLID
- ปรับปรุงการออกแบบโค้ดให้เป็นไปตามหลักการ SOLID
- ประยุกต์ใช้หลักการ SOLID กับโปรเจกต์จริง

## 📘 SOLID คืออะไร?

SOLID เป็นอักษรย่อของหลักการออกแบบซอฟต์แวร์เชิงวัตถุ 5 ข้อ ที่คิดค้นโดย Robert C. Martin (Uncle Bob) หลักการเหล่านี้ช่วยให้นักพัฒนาสร้างซอฟต์แวร์ที่บำรุงรักษาได้ง่าย ขยายได้ และปรับเปลี่ยนได้ในระยะยาว

## 🅢 Single Responsibility Principle (SRP)
### หลักการความรับผิดชอบเดียว

> "A class should have one, and only one, reason to change."

คลาสควรมีเหตุผลเพียงหนึ่งเดียวในการเปลี่ยนแปลง หรืออีกนัยหนึ่ง คลาสควรมีหน้าที่รับผิดชอบเพียงเรื่องเดียวเท่านั้น

### ตัวอย่างที่ละเมิดหลักการ SRP:

```csharp
// ❌ ผิดหลักการ SRP: คลาสนี้ทำหลายหน้าที่
public class UserService
{
    public void Register(string username, string password)
    {
        // ตรวจสอบความถูกต้อง
        if (string.IsNullOrWhiteSpace(username) || string.IsNullOrWhiteSpace(password))
            throw new ArgumentException("Username and password are required");
        
        if (password.Length < 8)
            throw new ArgumentException("Password must be at least 8 characters");
            
        // เข้ารหัสรหัสผ่าน
        string passwordHash = BCrypt.HashPassword(password);
        
        // บันทึกลงฐานข้อมูล
        using (var connection = new SqlConnection("connection_string"))
        {
            connection.Open();
            using (var command = new SqlCommand())
            {
                command.Connection = connection;
                command.CommandText = "INSERT INTO Users (Username, PasswordHash) VALUES (@Username, @PasswordHash)";
                command.Parameters.AddWithValue("@Username", username);
                command.Parameters.AddWithValue("@PasswordHash", passwordHash);
                command.ExecuteNonQuery();
            }
        }
        
        // ส่งอีเมล
        SmtpClient client = new SmtpClient("smtp.example.com");
        client.Send(new MailMessage(
            "noreply@example.com", 
            $"{username}@example.com",
            "Welcome to our service",
            "Thank you for registering!"
        ));
    }
}
```

### ตัวอย่างที่ปฏิบัติตามหลักการ SRP:

```csharp
// ✅ ถูกต้องตามหลักการ SRP: แต่ละคลาสมีหน้าที่เฉพาะ
public class UserValidator
{
    public void ValidateForRegistration(string username, string password)
    {
        if (string.IsNullOrWhiteSpace(username) || string.IsNullOrWhiteSpace(password))
            throw new ArgumentException("Username and password are required");
        
        if (password.Length < 8)
            throw new ArgumentException("Password must be at least 8 characters");
    }
}

public class PasswordHasher
{
    public string HashPassword(string password)
    {
        return BCrypt.HashPassword(password);
    }
    
    public bool VerifyPassword(string password, string hash)
    {
        return BCrypt.Verify(password, hash);
    }
}

public class UserRepository
{
    private readonly string _connectionString;
    
    public UserRepository(string connectionString)
    {
        _connectionString = connectionString;
    }
    
    public void SaveUser(string username, string passwordHash)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            connection.Open();
            using (var command = new SqlCommand())
            {
                command.Connection = connection;
                command.CommandText = "INSERT INTO Users (Username, PasswordHash) VALUES (@Username, @PasswordHash)";
                command.Parameters.AddWithValue("@Username", username);
                command.Parameters.AddWithValue("@PasswordHash", passwordHash);
                command.ExecuteNonQuery();
            }
        }
    }
}

public class EmailService
{
    private readonly string _smtpServer;
    
    public EmailService(string smtpServer)
    {
        _smtpServer = smtpServer;
    }
    
    public void SendWelcomeEmail(string email)
    {
        SmtpClient client = new SmtpClient(_smtpServer);
        client.Send(new MailMessage(
            "noreply@example.com", 
            email,
            "Welcome to our service",
            "Thank you for registering!"
        ));
    }
}

// คลาสที่ใช้งานร่วมกัน
public class UserService
{
    private readonly UserValidator _validator;
    private readonly PasswordHasher _passwordHasher;
    private readonly UserRepository _userRepository;
    private readonly EmailService _emailService;
    
    public UserService(
        UserValidator validator,
        PasswordHasher passwordHasher,
        UserRepository userRepository,
        EmailService emailService)
    {
        _validator = validator;
        _passwordHasher = passwordHasher;
        _userRepository = userRepository;
        _emailService = emailService;
    }
    
    public void Register(string username, string password)
    {
        // ตรวจสอบความถูกต้อง
        _validator.ValidateForRegistration(username, password);
        
        // เข้ารหัสรหัสผ่าน
        string passwordHash = _passwordHasher.HashPassword(password);
        
        // บันทึกลงฐานข้อมูล
        _userRepository.SaveUser(username, passwordHash);
        
        // ส่งอีเมล
        _emailService.SendWelcomeEmail($"{username}@example.com");
    }
}
```

### ประโยชน์ของ SRP:
1. โค้ดอ่านเข้าใจง่ายขึ้น
2. การทดสอบทำได้ง่ายขึ้น
3. ลดการพึ่งพาระหว่างคลาส
4. การเปลี่ยนแปลงทำได้ง่ายและส่งผลกระทบน้อยลง

## 🅞 Open/Closed Principle (OCP)
### หลักการเปิดปิด

> "Software entities should be open for extension, but closed for modification."

เอนทิตี้ซอฟต์แวร์ (คลาส, โมดูล, ฟังก์ชัน, ฯลฯ) ควรเปิดรับการขยาย แต่ปิดรับการแก้ไข ซึ่งหมายความว่าคุณควรสามารถขยายพฤติกรรมได้โดยไม่ต้องแก้ไขโค้ดเดิม

### ตัวอย่างที่ละเมิดหลักการ OCP:

```csharp
// ❌ ผิดหลักการ OCP: ต้องแก้ไขคลาสเมื่อเพิ่มประเภทการชำระเงินใหม่
public class PaymentProcessor
{
    public void ProcessPayment(string paymentType, decimal amount)
    {
        if (paymentType == "CreditCard")
        {
            // ตรรกะการชำระเงินด้วยบัตรเครดิต
            Console.WriteLine($"Processing credit card payment of ${amount}");
            // ติดต่อ payment gateway, เก็บข้อมูลธุรกรรม, ฯลฯ
        }
        else if (paymentType == "PayPal")
        {
            // ตรรกะการชำระเงินด้วย PayPal
            Console.WriteLine($"Processing PayPal payment of ${amount}");
            // ติดต่อ PayPal API, จัดการการชำระเงิน, ฯลฯ
        }
        else if (paymentType == "BankTransfer")
        {
            // ตรรกะการชำระเงินด้วยการโอนเงินผ่านธนาคาร
            Console.WriteLine($"Processing bank transfer of ${amount}");
            // จัดการการโอนเงิน, บันทึกข้อมูลธนาคาร, ฯลฯ
        }
        // หากต้องการเพิ่มวิธีชำระเงินใหม่ เช่น "Cryptocurrency"
        // จำเป็นต้องแก้ไขคลาสนี้โดยการเพิ่ม else if ใหม่
    }
}
```

### ตัวอย่างที่ปฏิบัติตามหลักการ OCP:

```csharp
// ✅ ถูกต้องตามหลักการ OCP: ใช้ interface และ inheritance ขยายได้โดยไม่ต้องแก้ไขโค้ดเดิม
public interface IPaymentMethod
{
    void ProcessPayment(decimal amount);
}

public class CreditCardPayment : IPaymentMethod
{
    public void ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Processing credit card payment of ${amount}");
        // ตรรกะการชำระเงินด้วยบัตรเครดิต
    }
}

public class PayPalPayment : IPaymentMethod
{
    public void ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Processing PayPal payment of ${amount}");
        // ตรรกะการชำระเงินด้วย PayPal
    }
}

public class BankTransferPayment : IPaymentMethod
{
    public void ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Processing bank transfer of ${amount}");
        // ตรรกะการชำระเงินด้วยการโอนเงินผ่านธนาคาร
    }
}

// สามารถเพิ่มวิธีชำระเงินใหม่ได้โดยเพิ่มคลาสใหม่:
public class CryptocurrencyPayment : IPaymentMethod
{
    public void ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Processing cryptocurrency payment of ${amount}");
        // ตรรกะการชำระเงินด้วย cryptocurrency
    }
}

// คลาสที่ใช้งาน IPaymentMethod ไม่ต้องแก้ไขเมื่อเพิ่มวิธีชำระเงินใหม่
public class PaymentProcessor
{
    public void ProcessPayment(IPaymentMethod paymentMethod, decimal amount)
    {
        paymentMethod.ProcessPayment(amount);
    }
}

// การใช้งาน:
var processor = new PaymentProcessor();
processor.ProcessPayment(new CreditCardPayment(), 100.00m);
processor.ProcessPayment(new PayPalPayment(), 50.00m);
processor.ProcessPayment(new CryptocurrencyPayment(), 0.005m);
```

### ประโยชน์ของ OCP:
1. ลดความเสี่ยงในการแนะนำบั๊กใหม่เมื่อเพิ่มฟังก์ชันใหม่
2. โค้ดมีความยืดหยุ่นและปรับปรุงได้ง่าย
3. ลดการทดสอบซ้ำเมื่อมีการเปลี่ยนแปลง

## 🅛 Liskov Substitution Principle (LSP)
### หลักการแทนที่ของ Liskov

> "Objects of a superclass should be replaceable with objects of a subclass without affecting the correctness of the program."

วัตถุของคลาสแม่ควรสามารถถูกแทนที่ด้วยวัตถุของคลาสลูกโดยไม่กระทบต่อความถูกต้องของโปรแกรม

### ตัวอย่างที่ละเมิดหลักการ LSP:

```csharp
// ❌ ผิดหลักการ LSP: ไม่สามารถแทนที่คลาสแม่ด้วยคลาสลูกได้โดยไม่มีผลกระทบ
public class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }
    
    public int CalculateArea()
    {
        return Width * Height;
    }
}

public class Square : Rectangle
{
    private int _side;
    
    public override int Width
    {
        get => _side;
        set => _side = value;
    }
    
    public override int Height
    {
        get => _side;
        set => _side = value;
    }
}

// ฟังก์ชันที่ทำงานกับ Rectangle:
public void ResizeRectangle(Rectangle rectangle)
{
    rectangle.Width = 10;
    rectangle.Height = 5;
    
    // ในกรณีที่เป็น Rectangle ปกติ ตอนนี้ควรมี Width = 10, Height = 5
    // แต่ถ้าเป็น Square, จะมี Width = 5, Height = 5 (ไม่เป็นไปตามที่คาดหวัง)
    
    Console.WriteLine($"Expected area: 50, Actual area: {rectangle.CalculateArea()}");
}

// การใช้งาน:
var rect = new Rectangle { Width = 5, Height = 5 };
ResizeRectangle(rect); // พิมพ์ "Expected area: 50, Actual area: 50" (ถูกต้อง)

var square = new Square { Width = 5, Height = 5 };
ResizeRectangle(square); // พิมพ์ "Expected area: 50, Actual area: 25" (ผิด!)
```

### ตัวอย่างที่ปฏิบัติตามหลักการ LSP:

```csharp
// ✅ ถูกต้องตามหลักการ LSP: ออกแบบโครงสร้างที่ทำให้คลาสลูกแทนที่คลาสแม่ได้อย่างสมบูรณ์
public interface IShape
{
    int CalculateArea();
}

public class Rectangle : IShape
{
    public int Width { get; set; }
    public int Height { get; set; }
    
    public Rectangle(int width, int height)
    {
        Width = width;
        Height = height;
    }
    
    public int CalculateArea()
    {
        return Width * Height;
    }
}

public class Square : IShape
{
    public int Side { get; set; }
    
    public Square(int side)
    {
        Side = side;
    }
    
    public int CalculateArea()
    {
        return Side * Side;
    }
}

// ฟังก์ชันที่ทำงานกับ IShape:
public void PrintArea(IShape shape)
{
    Console.WriteLine($"Area: {shape.CalculateArea()}");
}

// การใช้งาน:
var rect = new Rectangle(10, 5);
PrintArea(rect); // พิมพ์ "Area: 50"

var square = new Square(5);
PrintArea(square); // พิมพ์ "Area: 25"
```

### ประโยชน์ของ LSP:
1. ทำให้สามารถใช้ polymorphism ได้อย่างถูกต้อง
2. ช่วยให้มั่นใจว่าโปรแกรมทำงานได้อย่างถูกต้องเมื่อใช้การสืบทอด
3. เพิ่มความมั่นใจในการใช้งานระบบ inheritance

## 🅘 Interface Segregation Principle (ISP)
### หลักการแยก Interface

> "No client should be forced to depend on methods it does not use."

ลูกค้า (คลาสที่ใช้ interface) ไม่ควรถูกบังคับให้พึ่งพาเมธอดที่ไม่ได้ใช้ ควรแยก interface ใหญ่ออกเป็น interface เล็กๆ ที่เฉพาะเจาะจงมากขึ้น

### ตัวอย่างที่ละเมิดหลักการ ISP:

```csharp
// ❌ ผิดหลักการ ISP: interface มีเมธอดมากเกินไป ทำให้คลาสที่นำไปใช้ต้องพึ่งพาเมธอดที่ไม่ได้ใช้
public interface IMultiPurposeMachine
{
    void Print(string document);
    void Scan(string document);
    void Fax(string document);
    void Staple(string document);
    void Bind(string document);
    void Laminate(string document);
}

// คลาสที่ใช้ interface นี้ต้องนำทุกเมธอดไปใช้ แม้จะไม่ต้องการฟังก์ชันบางอย่าง:
public class BasicPrinter : IMultiPurposeMachine
{
    public void Print(string document)
    {
        Console.WriteLine($"Printing: {document}");
    }
    
    // ต้องนำเมธอดที่ไม่ต้องการไปใช้:
    public void Scan(string document)
    {
        throw new NotImplementedException("This printer cannot scan");
    }
    
    public void Fax(string document)
    {
        throw new NotImplementedException("This printer cannot fax");
    }
    
    public void Staple(string document)
    {
        throw new NotImplementedException("This printer cannot staple");
    }
    
    public void Bind(string document)
    {
        throw new NotImplementedException("This printer cannot bind");
    }
    
    public void Laminate(string document)
    {
        throw new NotImplementedException("This printer cannot laminate");
    }
}
```

### ตัวอย่างที่ปฏิบัติตามหลักการ ISP:

```csharp
// ✅ ถูกต้องตามหลักการ ISP: แยก interface ออกเป็นส่วนย่อยที่เฉพาะเจาะจง
public interface IPrinter
{
    void Print(string document);
}

public interface IScanner
{
    void Scan(string document);
}

public interface IFax
{
    void Fax(string document);
}

public interface IStapler
{
    void Staple(string document);
}

public interface IBinder
{
    void Bind(string document);
}

public interface ILaminator
{
    void Laminate(string document);
}

// คลาสสามารถเลือกนำ interface ที่ต้องการไปใช้:
public class BasicPrinter : IPrinter
{
    public void Print(string document)
    {
        Console.WriteLine($"Printing: {document}");
    }
}

public class AllInOnePrinter : IPrinter, IScanner, IFax
{
    public void Print(string document)
    {
        Console.WriteLine($"Printing: {document}");
    }
    
    public void Scan(string document)
    {
        Console.WriteLine($"Scanning: {document}");
    }
    
    public void Fax(string document)
    {
        Console.WriteLine($"Faxing: {document}");
    }
}

public class ProfessionalPrintShop : IPrinter, IScanner, IBinder, IStapler, ILaminator
{
    public void Print(string document)
    {
        Console.WriteLine($"Printing: {document}");
    }
    
    public void Scan(string document)
    {
        Console.WriteLine($"Scanning: {document}");
    }
    
    public void Bind(string document)
    {
        Console.WriteLine($"Binding: {document}");
    }
    
    public void Staple(string document)
    {
        Console.WriteLine($"Stapling: {document}");
    }
    
    public void Laminate(string document)
    {
        Console.WriteLine($"Laminating: {document}");
    }
}
```

### ประโยชน์ของ ISP:
1. ลดการพึ่งพาระหว่างคลาส
2. ทำให้ระบบมีความยืดหยุ่นมากขึ้น
3. ทำให้การทดสอบง่ายขึ้น
4. ลดผลกระทบจากการเปลี่ยนแปลง

## 🅓 Dependency Inversion Principle (DIP)
### หลักการผันกลับการพึ่งพา

> "High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions."

โมดูลระดับสูงไม่ควรพึ่งพาโมดูลระดับต่ำ ทั้งสองควรพึ่งพาการนามธรรม (abstractions) การนามธรรมไม่ควรพึ่งพารายละเอียด รายละเอียดควรพึ่งพาการนามธรรม

### ตัวอย่างที่ละเมิดหลักการ DIP:

```csharp
// ❌ ผิดหลักการ DIP: คลาสระดับสูงพึ่งพาคลาสระดับต่ำโดยตรง
public class EmailNotifier
{
    public void SendEmail(string to, string subject, string body)
    {
        // ตรรกะการส่งอีเมล
        Console.WriteLine($"Sending email to {to}: {subject}");
    }
}

public class OrderService
{
    private readonly EmailNotifier _emailNotifier;
    
    public OrderService()
    {
        _emailNotifier = new EmailNotifier(); // การพึ่งพาโดยตรงต่อคลาสระดับต่ำ
    }
    
    public void PlaceOrder(Order order)
    {
        // ตรรกะการสั่งซื้อ
        Console.WriteLine($"Order placed: {order.Id}");
        
        // ส่งการแจ้งเตือน
        _emailNotifier.SendEmail(order.CustomerEmail, "Order Confirmation", $"Your order {order.Id} has been placed.");
    }
}

public class Order
{
    public string Id { get; set; }
    public string CustomerEmail { get; set; }
    // คุณสมบัติอื่น ๆ ของ Order
}
```

### ตัวอย่างที่ปฏิบัติตามหลักการ DIP:

```csharp
// ✅ ถูกต้องตามหลักการ DIP: ทั้งคลาสระดับสูงและระดับต่ำพึ่งพาการนามธรรม
public interface INotificationService
{
    void SendNotification(string to, string subject, string body);
}

public class EmailNotifier : INotificationService
{
    public void SendNotification(string to, string subject, string body)
    {
        // ตรรกะการส่งอีเมล
        Console.WriteLine($"Sending email to {to}: {subject}");
    }
}

public class SmsNotifier : INotificationService
{
    public void SendNotification(string to, string subject, string body)
    {
        // ตรรกะการส่ง SMS
        Console.WriteLine($"Sending SMS to {to}: {subject}");
    }
}

public class PushNotifier : INotificationService
{
    public void SendNotification(string to, string subject, string body)
    {
        // ตรรกะการส่งการแจ้งเตือนแบบ push
        Console.WriteLine($"Sending push notification to {to}: {subject}");
    }
}

public class OrderService
{
    private readonly INotificationService _notificationService;
    
    // การฉีด dependency (dependency injection)
    public OrderService(INotificationService notificationService)
    {
        _notificationService = notificationService;
    }
    
    public void PlaceOrder(Order order)
    {
        // ตรรกะการสั่งซื้อ
        Console.WriteLine($"Order placed: {order.Id}");
        
        // ส่งการแจ้งเตือน
        _notificationService.SendNotification(
            order.CustomerEmail, 
            "Order Confirmation", 
            $"Your order {order.Id} has been placed."
        );
    }
}

// การใช้งาน:
var emailNotifier = new EmailNotifier();
var orderService = new OrderService(emailNotifier);
orderService.PlaceOrder(new Order { Id = "12345", CustomerEmail = "customer@example.com" });

// หากต้องการเปลี่ยนวิธีการแจ้งเตือน:
var smsNotifier = new SmsNotifier();
var orderServiceWithSms = new OrderService(smsNotifier);
orderServiceWithSms.PlaceOrder(new Order { Id = "67890", CustomerEmail = "555-123-4567" });
```

### ประโยชน์ของ DIP:
1. ลดการพึ่งพาระหว่างโมดูล
2. เพิ่มความสามารถในการทดสอบ
3. ทำให้โค้ดยืดหยุ่นและปรับเปลี่ยนได้ง่าย
4. สนับสนุนการทำ dependency injection

## 🛠️ การนำหลักการ SOLID ไปใช้ในโปรเจกต์จริง

### ตัวอย่างการใช้ SOLID ในระบบ E-commerce

```csharp
// SOLID principles applied to an e-commerce system

// Single Responsibility Principle
public class Product
{
    public string Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

public class ProductValidator
{
    public bool IsValid(Product product)
    {
        return !string.IsNullOrEmpty(product.Id) &&
               !string.IsNullOrEmpty(product.Name) &&
               product.Price > 0;
    }
}

public class ProductRepository
{
    public void Save(Product product)
    {
        // Save product to database
    }
    
    public Product GetById(string id)
    {
        // Get product from database
        return new Product();
    }
}

// Open/Closed Principle
public interface IDiscountStrategy
{
    decimal ApplyDiscount(decimal originalPrice);
}

public class NoDiscount : IDiscountStrategy
{
    public decimal ApplyDiscount(decimal originalPrice) => originalPrice;
}

public class PercentageDiscount : IDiscountStrategy
{
    private readonly decimal _percentage;
    
    public PercentageDiscount(decimal percentage)
    {
        _percentage = percentage;
    }
    
    public decimal ApplyDiscount(decimal originalPrice)
    {
        return originalPrice * (1 - _percentage / 100);
    }
}

public class FixedAmountDiscount : IDiscountStrategy
{
    private readonly decimal _amount;
    
    public FixedAmountDiscount(decimal amount)
    {
        _amount = amount;
    }
    
    public decimal ApplyDiscount(decimal originalPrice)
    {
        return Math.Max(0, originalPrice - _amount);
    }
}

// Liskov Substitution Principle
public abstract class PaymentMethod
{
    public abstract bool ProcessPayment(decimal amount);
}

public class CreditCardPayment : PaymentMethod
{
    public override bool ProcessPayment(decimal amount)
    {
        // Process credit card payment
        return true;
    }
}

public class PayPalPayment : PaymentMethod
{
    public override bool ProcessPayment(decimal amount)
    {
        // Process PayPal payment
        return true;
    }
}

// Interface Segregation Principle
public interface IOrderProcessor
{
    void Process(Order order);
}

public interface IOrderValidator
{
    bool Validate(Order order);
}

public interface IOrderRepository
{
    void Save(Order order);
    Order GetById(string id);
}

// Dependency Inversion Principle
public class OrderService
{
    private readonly IOrderValidator _validator;
    private readonly IOrderProcessor _processor;
    private readonly IOrderRepository _repository;
    private readonly INotificationService _notificationService;
    
    public OrderService(
        IOrderValidator validator,
        IOrderProcessor processor,
        IOrderRepository repository,
        INotificationService notificationService)
    {
        _validator = validator;
        _processor = processor;
        _repository = repository;
        _notificationService = notificationService;
    }
    
    public void PlaceOrder(Order order)
    {
        if (_validator.Validate(order))
        {
            _processor.Process(order);
            _repository.Save(order);
            _notificationService.SendNotification(
                order.CustomerEmail,
                "Order Confirmation",
                $"Your order {order.Id} has been placed."
            );
        }
    }
}
```

## 📝 แบบฝึกหัด

### Exercise 1: ระบุการละเมิดหลักการ SOLID
ระบุว่าโค้ดต่อไปนี้ละเมิดหลักการ SOLID ข้อใด และอธิบายว่าทำไม:

```csharp
public class ReportGenerator
{
    public void GenerateReport(string reportType, string data)
    {
        if (reportType == "PDF")
        {
            Console.WriteLine("Generating PDF report");
            // Logic for PDF generation
        }
        else if (reportType == "Excel")
        {
            Console.WriteLine("Generating Excel report");
            // Logic for Excel generation
        }
        
        // Save the report to database
        SaveToDatabase(data);
        
        // Send email notification
        SendEmailNotification(data);
    }
    
    private void SaveToDatabase(string data)
    {
        // Database saving logic
        Console.WriteLine("Saving report to database");
    }
    
    private void SendEmailNotification(string data)
    {
        // Email sending logic
        Console.WriteLine("Sending email notification");
    }
}
```

### Exercise 2: ปรับปรุงโค้ดให้เป็นไปตามหลักการ SOLID
ปรับปรุงโค้ดจาก Exercise 1 ให้เป็นไปตามหลักการ SOLID ทั้ง 5 ข้อ

## 🔑 สรุปหลักการ SOLID

1. **Single Responsibility Principle (SRP)**
   - คลาสควรมีเหตุผลเพียงหนึ่งเดียวในการเปลี่ยนแปลง
   - แยกความรับผิดชอบให้ชัดเจน

2. **Open/Closed Principle (OCP)**
   - ซอฟต์แวร์เอนทิตี้ควรเปิดรับการขยาย แต่ปิดรับการแก้ไข
   - ใช้ interface และ inheritance เพื่อขยายความสามารถโดยไม่ต้องแก้ไขโค้ดเดิม

3. **Liskov Substitution Principle (LSP)**
   - วัตถุของคลาสแม่ควรถูกแทนที่ด้วยวัตถุของคลาสลูกได้โดยไม่กระทบต่อโปรแกรม
   - ออกแบบ inheritance hierarchy อย่างระมัดระวัง

4. **Interface Segregation Principle (ISP)**
   - ลูกค้าไม่ควรถูกบังคับให้พึ่งพาเมธอดที่ไม่ได้ใช้
   - สร้าง interface ที่เฉพาะเจาะจงแทนที่จะเป็น interface ขนาดใหญ่

5. **Dependency Inversion Principle (DIP)**
   - โมดูลระดับสูงไม่ควรพึ่งพาโมดูลระดับต่ำ ทั้งสองควรพึ่งพาการนามธรรม
   - ใช้ dependency injection เพื่อลดการพึ่งพาระหว่างคลาส

การปฏิบัติตามหลักการ SOLID จะช่วยให้โค้ดของคุณมีคุณภาพสูงขึ้น ยืดหยุ่น บำรุงรักษาง่าย และพร้อมรับการเปลี่ยนแปลงในอนาคต

## 📚 แหล่งข้อมูลเพิ่มเติม
- [SOLID Principles in C#](https://www.c-sharpcorner.com/UploadFile/damubetha/solid-principles-in-C-Sharp/)
- [Uncle Bob's Clean Code Blog](https://blog.cleancoder.com/)
- [Practical SOLID in C#](https://www.pluralsight.com/courses/csharp-solid-principles)
- [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612)

## 🚀 ต่อไป
- [Creational Patterns](./02-creational-patterns.md) - รูปแบบการออกแบบสำหรับการสร้างวัตถุ
