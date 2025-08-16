# Design 04 — Behavioral Patterns (C#)

## Strategy
```csharp
public interface ISortStrategy { void Sort(List<int> data); }
public class QuickSort : ISortStrategy { public void Sort(List<int> d) => d.Sort(); }
public class BubbleSort : ISortStrategy { public void Sort(List<int> d) { /* demo */ } }

public class Sorter
{ private ISortStrategy _s; public Sorter(ISortStrategy s)=>_s=s; public void Execute(List<int> d)=>_s.Sort(d); }
```

## Observer (Event)
```csharp
public class NewsPublisher
{
    public event EventHandler<string>? OnNews;
    public void Publish(string msg) => OnNews?.Invoke(this, msg);
}

var pub = new NewsPublisher();
pub.OnNews += (s, m) => Console.WriteLine($"Subscriber A: {m}");
pub.Publish("Hello");
```

## Command
```csharp
public interface ICommand { void Execute(); }
public class PrintCommand : ICommand { private readonly string _m; public PrintCommand(string m)=>_m=m; public void Execute()=>Console.WriteLine(_m); }
public class Invoker { public void Invoke(ICommand c)=>c.Execute(); }
```
