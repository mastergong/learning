# Design 01 — SOLID Principles (พร้อมตัวอย่าง C#)

## S — Single Responsibility Principle (SRP)
คลาสควรมีเหตุผลเดียวในการเปลี่ยนแปลง

```csharp
// ❌ ผิด: ทำหลายหน้าที่
public class InvoiceService
{
    public void CreateInvoice() { /* ... */ }
    public void SaveToDb() { /* ... */ }
    public void SendEmail() { /* ... */ }
}

// ✅ ถูก: แยกความรับผิดชอบ
public class InvoiceCreator { public void CreateInvoice() { /* ... */ } }
public class InvoiceRepository { public void Save(Invoice inv) { /* ... */ } }
public class EmailNotifier { public void Send(Invoice inv) { /* ... */ } }
```

## O — Open/Closed Principle (OCP)
เปิดรับการขยาย ปิดรับการแก้ไข

```csharp
public interface ITaxStrategy { decimal Calc(decimal amount); }
public class VatStrategy : ITaxStrategy { public decimal Calc(decimal a) => a * 0.07m; }
public class NoTaxStrategy : ITaxStrategy { public decimal Calc(decimal a) => 0m; }

public class Checkout
{
    private readonly ITaxStrategy _tax;
    public Checkout(ITaxStrategy tax) => _tax = tax;
    public decimal Total(decimal amount) => amount + _tax.Calc(amount);
}
```

## L — Liskov Substitution Principle (LSP)
ชนิดย่อยต้องแทนที่ชนิดแม่ได้โดยไม่ผิดพฤติกรรม

```csharp
public abstract class Bird { public abstract void Move(); }
public class Sparrow : Bird { public override void Move() => Console.WriteLine("fly"); }
public class Penguin : Bird { public override void Move() => Console.WriteLine("walk"); }
```

## I — Interface Segregation Principle (ISP)
ใช้หลาย interface เฉพาะทาง แทน interface ใหญ่ก้อนเดียว

```csharp
public interface IPrinter { void Print(); }
public interface IScanner { void Scan(); }
public class AllInOne : IPrinter, IScanner { public void Print(){} public void Scan(){} }
```

## D — Dependency Inversion Principle (DIP)
พึ่งพา abstraction ไม่ใช่ concrete

```csharp
public interface IMessageSender { void Send(string msg); }
public class EmailSender : IMessageSender { public void Send(string m) => Console.WriteLine($"Email: {m}"); }

public class AlertService
{
    private readonly IMessageSender _sender;
    public AlertService(IMessageSender sender) => _sender = sender;
    public void Alert(string text) => _sender.Send(text);
}
```

## สรุป
- ออกแบบให้ทดสอบง่าย ขยายได้ และยืดหยุ่น ด้วย SOLID
