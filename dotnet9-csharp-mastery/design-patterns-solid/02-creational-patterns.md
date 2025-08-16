# Design 02 — Creational Patterns (C#)

## Factory Method
```csharp
public abstract class Logger
{
    public abstract void Log(string message);
    public static Logger Create(string type) => type switch
    {
        "console" => new ConsoleLogger(),
        "file" => new FileLogger("app.log"),
        _ => new ConsoleLogger()
    };
}

public class ConsoleLogger : Logger
{ public override void Log(string m) => Console.WriteLine(m); }
public class FileLogger : Logger
{
    private readonly string _path;
    public FileLogger(string path) => _path = path;
    public override void Log(string m) => File.AppendAllText(_path, m + Environment.NewLine);
}
```

## Abstract Factory
```csharp
public interface IUiFactory { IButton CreateButton(); ITextBox CreateTextBox(); }
public interface IButton { void Render(); }
public interface ITextBox { void Render(); }

public class WinFactory : IUiFactory
{ public IButton CreateButton() => new WinButton(); public ITextBox CreateTextBox() => new WinTextBox(); }
public class MacFactory : IUiFactory
{ public IButton CreateButton() => new MacButton(); public ITextBox CreateTextBox() => new MacTextBox(); }

public class WinButton : IButton { public void Render() => Console.WriteLine("WinButton"); }
public class MacButton : IButton { public void Render() => Console.WriteLine("MacButton"); }
public class WinTextBox : ITextBox { public void Render() => Console.WriteLine("WinTextBox"); }
public class MacTextBox : ITextBox { public void Render() => Console.WriteLine("MacTextBox"); }
```

## Builder
```csharp
public record User(string Name, int Age, string Email);

public class UserBuilder
{
    private string _name = string.Empty;
    private int _age;
    private string _email = string.Empty;

    public UserBuilder WithName(string n){ _name = n; return this; }
    public UserBuilder WithAge(int a){ _age = a; return this; }
    public UserBuilder WithEmail(string e){ _email = e; return this; }
    public User Build() => new(_name, _age, _email);
}

var user = new UserBuilder().WithName("Ann").WithAge(30).WithEmail("a@b.com").Build();
```

## Singleton (thread-safe)
```csharp
public sealed class Config
{
    private static readonly Lazy<Config> _instance = new(() => new Config());
    public static Config Instance => _instance.Value;
    private Config(){ }
    public string Environment { get; init; } = "Production";
}
```
