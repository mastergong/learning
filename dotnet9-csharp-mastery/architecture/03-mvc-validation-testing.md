# Architecture 03 — MVC Validation & Testing

## Validation ด้วย DataAnnotations
```csharp
public class ProductInput
{
    [Required, StringLength(100)]
    public string Name { get; set; } = string.Empty;
    [Range(0, 100000)]
    public decimal Price { get; set; }
}

public class ProductsController : Controller
{
    [HttpPost]
    public IActionResult Create(ProductInput input)
    {
        if (!ModelState.IsValid) return View(input);
        // Save ...
        return RedirectToAction("Index");
    }
}
```

## Minimal API + Validation
```csharp
app.MapPost("/products", ([FromBody] ProductInput input) =>
{
    if (string.IsNullOrWhiteSpace(input.Name) || input.Price < 0)
        return Results.BadRequest();
    return Results.Created("/products/1", input);
});
```

## Testing ด้วย xUnit
```csharp
public class PriceCalculator
{
    public decimal Total(decimal price, int qty) => price * qty;
}

public class PriceCalculatorTests
{
    [Fact]
    public void Total_ShouldMultiply()
    {
        var sut = new PriceCalculator();
        Assert.Equal(200m, sut.Total(100m, 2));
    }
}
```
