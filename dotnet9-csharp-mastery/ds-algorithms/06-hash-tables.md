# DS&A 06 — Hash Table / Dictionary (C#)

- Dictionary<TKey,TValue> ให้ O(1) โดยเฉลี่ย
- ควร override Equals/GetHashCode สำหรับ custom key

```csharp
public readonly record struct Point(int X, int Y);

public sealed class PointKey
{
    public int X { get; }
    public int Y { get; }
    public PointKey(int x, int y) { X=x; Y=y; }
    public override bool Equals(object? obj)
        => obj is PointKey o && X==o.X && Y==o.Y;
    public override int GetHashCode() => HashCode.Combine(X, Y);
}

var map = new Dictionary<PointKey, string>();
map[new PointKey(1,2)] = "A";
Console.WriteLine(map[new PointKey(1,2)]); // A
```

## Collision Handling Insight
- .NET ใช้ bucket + chaining; ขยายเมื่อ load factor สูง
- ระวัง mutation ของ key หลัง insert
