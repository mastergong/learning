# Design 03 — Structural Patterns (C#)

## Adapter
```csharp
public interface IPayment { void Pay(decimal amount); }
public class LegacyPay { public void DoPay(decimal amt) => Console.WriteLine($"Legacy {amt}"); }
public class PaymentAdapter : IPayment
{ private readonly LegacyPay _legacy; public PaymentAdapter(LegacyPay l)=>_legacy=l; public void Pay(decimal a)=>_legacy.DoPay(a); }
```

## Decorator
```csharp
public interface IReport { string Render(); }
public class BasicReport : IReport { public string Render() => "Report"; }
public class HtmlReport : IReport
{ private readonly IReport _inner; public HtmlReport(IReport i)=>_inner=i; public string Render()=>"<html>"+_inner.Render()+"</html>"; }
```

## Facade
```csharp
public class VideoConverter
{
    public void Convert(string input, string output)
    {
        // ซ่อนความซับซ้อนของหลาย subsystem
        Console.WriteLine($"Convert {input} -> {output}");
    }
}
```
