# Architecture 02 — Clean Architecture (C#/.NET)

โครงสร้างโดยแยกเลเยอร์และการพึ่งพา โดยให้ Domain ไม่พึ่งพา Infrastructure

## โครงสร้างโฟลเดอร์ตัวอย่าง
```
CleanShop/
  src/
    CleanShop.Domain/
      Entities/ Product.cs
      ValueObjects/ Money.cs
      Interfaces/ IProductRepository.cs
    CleanShop.Application/
      UseCases/ CreateProduct.cs
      Common/ Result.cs
    CleanShop.Infrastructure/
      Persistence/ EfProductRepository.cs
      EfDbContext.cs
    CleanShop.Web/
      Program.cs
      Controllers/
```

## Domain Layer (ไม่มี EF/HTTP)
```csharp
// Entities/Product.cs
namespace CleanShop.Domain.Entities;

public class Product
{
    public Guid Id { get; private set; } = Guid.NewGuid();
    public string Name { get; private set; }
    public decimal Price { get; private set; }

    public Product(string name, decimal price)
    {
        if (string.IsNullOrWhiteSpace(name)) throw new ArgumentException("name");
        if (price < 0) throw new ArgumentOutOfRangeException(nameof(price));
        Name = name; Price = price;
    }
    public void Rename(string name){ if (string.IsNullOrWhiteSpace(name)) throw new ArgumentException("name"); Name = name; }
}

// Interfaces/IProductRepository.cs
namespace CleanShop.Domain.Interfaces;
public interface IProductRepository
{
    Task AddAsync(Product p, CancellationToken ct = default);
    Task<Product?> GetAsync(Guid id, CancellationToken ct = default);
}
```

## Application Layer
```csharp
// UseCases/CreateProduct.cs
using CleanShop.Domain.Entities;
using CleanShop.Domain.Interfaces;

namespace CleanShop.Application.UseCases;

public sealed class CreateProduct
{
    private readonly IProductRepository _repo;
    public CreateProduct(IProductRepository repo) => _repo = repo;

    public async Task<Guid> Handle(string name, decimal price, CancellationToken ct = default)
    {
        var p = new Product(name, price);
        await _repo.AddAsync(p, ct);
        return p.Id;
    }
}
```

## Infrastructure Layer (EF Core)
```csharp
// EfDbContext.cs
using CleanShop.Domain.Entities;
using Microsoft.EntityFrameworkCore;

public class EfDbContext : DbContext
{
    public DbSet<Product> Products => Set<Product>();
    public EfDbContext(DbContextOptions<EfDbContext> options) : base(options) { }
}

// Persistence/EfProductRepository.cs
using CleanShop.Domain.Entities;
using CleanShop.Domain.Interfaces;
using Microsoft.EntityFrameworkCore;

public class EfProductRepository : IProductRepository
{
    private readonly EfDbContext _db;
    public EfProductRepository(EfDbContext db) => _db = db;
    public async Task AddAsync(Product p, CancellationToken ct = default)
    {
        _db.Products.Add(p);
        await _db.SaveChangesAsync(ct);
    }
    public Task<Product?> GetAsync(Guid id, CancellationToken ct = default)
        => _db.Products.FirstOrDefaultAsync(x => x.Id == id, ct);
}
```

## Web Layer (ประกอบ DI)
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<EfDbContext>(o => o.UseInMemoryDatabase("clean-shop"));
builder.Services.AddScoped<IProductRepository, EfProductRepository>();
builder.Services.AddScoped<CreateProduct>();

var app = builder.Build();

app.MapPost("/products", async (CreateProduct useCase, ProductDto dto, CancellationToken ct) =>
{
    var id = await useCase.Handle(dto.Name, dto.Price, ct);
    return Results.Created($"/products/{id}", new { id });
});

app.Run();

record ProductDto(string Name, decimal Price);
```

## แนวปฏิบัติ
- Domain บริสุทธิ์ (ไม่อิง EF/Framework)
- Application ใช้ UseCase/Service จัดการธุรกิจ เรียกผ่าน Interface
- Infrastructure ติดตั้งเทคโนโลยีจริง (EF, Http, Email)
- Web ต่อ DI, Mapping, Validation, Auth
