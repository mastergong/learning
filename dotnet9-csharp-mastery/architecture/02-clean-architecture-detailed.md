# Clean Architecture ใน .NET 9: แนวทางการออกแบบเพื่อความยั่งยืน

## 🎯 เป้าหมายการเรียนรู้
เมื่อเสร็จสิ้นบทเรียนนี้ คุณจะสามารถ:
- เข้าใจหลักการและประโยชน์ของ Clean Architecture
- อธิบาย layers ต่างๆ ใน Clean Architecture และความสัมพันธ์ระหว่างกัน
- สร้างโปรเจกต์ .NET 9 ที่ใช้ Clean Architecture
- ประยุกต์ใช้ CQRS และ MediatR กับ Clean Architecture
- ทำ Dependency Injection อย่างถูกต้องใน Clean Architecture

## 📘 Clean Architecture คืออะไร?

Clean Architecture คือรูปแบบการออกแบบซอฟต์แวร์ที่เน้นการแยกความรับผิดชอบ (separation of concerns) โดยจัดโครงสร้างแอปพลิเคชันให้เป็น layers ที่มีขอบเขตชัดเจน มีกฎการพึ่งพา (dependency rules) ที่ชัดเจน ซึ่งช่วยให้โค้ดมีความยืดหยุ่น ทดสอบได้ง่าย และปรับเปลี่ยนได้ในระยะยาว

Clean Architecture ถูกพัฒนาโดย Robert C. Martin (Uncle Bob) และเป็นการนำแนวคิดจากสถาปัตยกรรมต่างๆ เช่น Hexagonal Architecture, Onion Architecture และ Screaming Architecture มาผสมผสานกัน

### หลักการพื้นฐานของ Clean Architecture
1. **การแยกส่วน (Separation of concerns)** - แบ่งโค้ดเป็นส่วนๆ ตามหน้าที่
2. **กฎการพึ่งพา (Dependency rule)** - การพึ่งพาต้องพุ่งเข้าสู่ศูนย์กลางเท่านั้น (วงในพึ่งพาวงนอกไม่ได้)
3. **การแยกเรื่อง frameworks** - โดเมนโค้ดไม่ควรรู้จักเฟรมเวิร์ก
4. **ทดสอบได้ (Testability)** - ออกแบบให้ทดสอบได้ง่าย โดยเฉพาะส่วนที่เป็น business logic
5. **เป็นอิสระจากรายละเอียดการนำไปใช้งาน** - business rules ไม่ควรขึ้นอยู่กับ UI, ฐานข้อมูล, หรือ external systems

## 🏗️ โครงสร้าง Clean Architecture

Clean Architecture แบ่งออกเป็น layers ดังนี้:

### 1. Domain Layer (Core)
เป็นศูนย์กลางของแอปพลิเคชัน ประกอบด้วย:
- **Entities** - objects ที่เป็นตัวแทนของแนวคิดทางธุรกิจ
- **Value Objects** - objects ที่ไม่มี identity แต่อธิบายคุณลักษณะของ Entity
- **Domain Events** - เหตุการณ์ที่เกิดขึ้นในโดเมน
- **Domain Exceptions** - ข้อยกเว้นเฉพาะของโดเมน
- **Domain Services** - บริการที่จำเป็นสำหรับกระบวนการทางธุรกิจที่ซับซ้อน

**สิ่งสำคัญ**: Layer นี้ไม่มีการพึ่งพาภายนอก

### 2. Application Layer
จัดการกับ use cases ของแอปพลิเคชัน ประกอบด้วย:
- **Use Cases / Application Services** - orchestrators ที่ดำเนินการตาม business workflows
- **DTOs (Data Transfer Objects)** - objects สำหรับส่งข้อมูลระหว่าง layers
- **Interfaces / Ports** - contracts ที่ Application Layer ต้องการจาก Infrastructure Layer
- **Event Handlers** - ตัวจัดการกับ domain events

**สิ่งสำคัญ**: Layer นี้พึ่งพา Domain Layer เท่านั้น

### 3. Infrastructure Layer
จัดการรายละเอียดทางเทคนิค เช่น:
- **Repositories** - การเข้าถึงฐานข้อมูล
- **External Services** - การเชื่อมต่อกับ APIs ภายนอก
- **Email Senders** - บริการส่งอีเมล
- **File Systems** - การจัดการไฟล์
- **Identity Services** - การจัดการตัวตนและการรับรองความถูกต้อง

**สิ่งสำคัญ**: Layer นี้นำ interfaces จาก Application Layer มาทำการ implement

### 4. Presentation Layer
จัดการการแสดงผลและการปฏิสัมพันธ์กับผู้ใช้:
- **Controllers** (API) - รับ requests และส่งคืน responses
- **View Models** - models สำหรับการแสดงผล
- **Views** (ถ้าเป็น MVC) - UI templates
- **Formatters** - การแปลงข้อมูลให้เหมาะสมกับการแสดงผล

## 💼 ตัวอย่าง Clean Architecture ใน .NET 9

### โครงสร้างโปรเจกต์ Clean Architecture

```
CleanArchitectureDemo/
│
├── src/
│   ├── CleanArchitecture.Domain/
│   │   ├── Entities/
│   │   │   ├── Customer.cs
│   │   │   ├── Order.cs
│   │   │   └── Product.cs
│   │   ├── ValueObjects/
│   │   │   ├── Address.cs
│   │   │   ├── Money.cs
│   │   │   └── Email.cs
│   │   ├── Events/
│   │   │   ├── OrderPlacedEvent.cs
│   │   │   └── ProductOutOfStockEvent.cs
│   │   ├── Exceptions/
│   │   │   ├── DomainException.cs
│   │   │   └── InsufficientStockException.cs
│   │   └── Services/
│   │       └── OrderDomainService.cs
│   │
│   ├── CleanArchitecture.Application/
│   │   ├── DTOs/
│   │   │   ├── CustomerDto.cs
│   │   │   ├── OrderDto.cs
│   │   │   └── ProductDto.cs
│   │   ├── Interfaces/
│   │   │   ├── ICustomerRepository.cs
│   │   │   ├── IOrderRepository.cs
│   │   │   ├── IProductRepository.cs
│   │   │   ├── IEmailService.cs
│   │   │   └── IUnitOfWork.cs
│   │   ├── Services/
│   │   │   ├── CustomerService.cs
│   │   │   ├── OrderService.cs
│   │   │   └── ProductService.cs
│   │   └── Behaviors/
│   │       ├── ValidationBehavior.cs
│   │       └── LoggingBehavior.cs
│   │
│   ├── CleanArchitecture.Infrastructure/
│   │   ├── Data/
│   │   │   ├── AppDbContext.cs
│   │   │   ├── Configurations/
│   │   │   │   ├── CustomerConfiguration.cs
│   │   │   │   ├── OrderConfiguration.cs
│   │   │   │   └── ProductConfiguration.cs
│   │   │   └── Repositories/
│   │   │       ├── CustomerRepository.cs
│   │   │       ├── OrderRepository.cs
│   │   │       ├── ProductRepository.cs
│   │   │       └── UnitOfWork.cs
│   │   ├── Services/
│   │   │   ├── EmailService.cs
│   │   │   └── PaymentService.cs
│   │   └── DependencyInjection.cs
│   │
│   └── CleanArchitecture.API/
│       ├── Controllers/
│       │   ├── CustomersController.cs
│       │   ├── OrdersController.cs
│       │   └── ProductsController.cs
│       ├── ViewModels/
│       │   ├── CustomerViewModel.cs
│       │   ├── OrderViewModel.cs
│       │   └── ProductViewModel.cs
│       ├── Filters/
│       │   └── ApiExceptionFilter.cs
│       ├── Program.cs
│       └── appsettings.json
│
└── tests/
    ├── CleanArchitecture.UnitTests/
    │   ├── Domain/
    │   │   ├── CustomerTests.cs
    │   │   ├── OrderTests.cs
    │   │   └── ProductTests.cs
    │   └── Application/
    │       ├── CustomerServiceTests.cs
    │       ├── OrderServiceTests.cs
    │       └── ProductServiceTests.cs
    │
    └── CleanArchitecture.IntegrationTests/
        ├── API/
        │   ├── CustomersControllerTests.cs
        │   ├── OrdersControllerTests.cs
        │   └── ProductsControllerTests.cs
        └── Infrastructure/
            └── Repositories/
                ├── CustomerRepositoryTests.cs
                ├── OrderRepositoryTests.cs
                └── ProductRepositoryTests.cs
```

### 1. Domain Layer

#### Customer Entity
```csharp
namespace CleanArchitecture.Domain.Entities
{
    public class Customer
    {
        public Guid Id { get; private set; }
        public string FirstName { get; private set; }
        public string LastName { get; private set; }
        public Email Email { get; private set; }
        public Address Address { get; private set; }
        private readonly List<Order> _orders = new();
        public IReadOnlyCollection<Order> Orders => _orders.AsReadOnly();
        
        private Customer() { } // For ORM
        
        public Customer(string firstName, string lastName, Email email, Address address)
        {
            Id = Guid.NewGuid();
            FirstName = firstName;
            LastName = lastName;
            Email = email;
            Address = address;
        }
        
        public void UpdateAddress(Address newAddress)
        {
            Address = newAddress;
        }
        
        public void AddOrder(Order order)
        {
            _orders.Add(order);
        }
        
        public string FullName => $"{FirstName} {LastName}";
    }
}
```

#### Email Value Object
```csharp
namespace CleanArchitecture.Domain.ValueObjects
{
    public class Email : ValueObject
    {
        public string Value { get; private set; }
        
        private Email() { } // For ORM
        
        private Email(string value)
        {
            Value = value;
        }
        
        public static Result<Email> Create(string email)
        {
            if (string.IsNullOrWhiteSpace(email))
                return Result.Failure<Email>("Email cannot be empty");
                
            if (!email.Contains("@") || !email.Contains("."))
                return Result.Failure<Email>("Email is not in valid format");
                
            return Result.Success(new Email(email));
        }
        
        protected override IEnumerable<object> GetEqualityComponents()
        {
            yield return Value;
        }
        
        public override string ToString() => Value;
    }
}
```

#### OrderPlacedEvent
```csharp
namespace CleanArchitecture.Domain.Events
{
    public class OrderPlacedEvent : DomainEvent
    {
        public Order Order { get; }
        public Customer Customer { get; }
        
        public OrderPlacedEvent(Order order, Customer customer)
        {
            Order = order;
            Customer = customer;
        }
    }
}
```

### 2. Application Layer

#### IOrderRepository Interface
```csharp
namespace CleanArchitecture.Application.Interfaces
{
    public interface IOrderRepository
    {
        Task<Order> GetByIdAsync(Guid id);
        Task<IEnumerable<Order>> GetByCustomerIdAsync(Guid customerId);
        Task AddAsync(Order order);
        Task UpdateAsync(Order order);
        Task DeleteAsync(Order order);
    }
}
```

#### OrderService
```csharp
namespace CleanArchitecture.Application.Services
{
    public class OrderService : IOrderService
    {
        private readonly IOrderRepository _orderRepository;
        private readonly IProductRepository _productRepository;
        private readonly ICustomerRepository _customerRepository;
        private readonly IUnitOfWork _unitOfWork;
        private readonly IEmailService _emailService;
        
        public OrderService(
            IOrderRepository orderRepository,
            IProductRepository productRepository,
            ICustomerRepository customerRepository,
            IUnitOfWork unitOfWork,
            IEmailService emailService)
        {
            _orderRepository = orderRepository;
            _productRepository = productRepository;
            _customerRepository = customerRepository;
            _unitOfWork = unitOfWork;
            _emailService = emailService;
        }
        
        public async Task<OrderDto> GetOrderByIdAsync(Guid id)
        {
            var order = await _orderRepository.GetByIdAsync(id);
            if (order == null)
                return null;
                
            return new OrderDto
            {
                Id = order.Id,
                CustomerId = order.CustomerId,
                OrderDate = order.OrderDate,
                Status = order.Status.ToString(),
                TotalAmount = order.TotalAmount,
                Items = order.Items.Select(i => new OrderItemDto
                {
                    ProductId = i.ProductId,
                    Quantity = i.Quantity,
                    Price = i.Price
                }).ToList()
            };
        }
        
        public async Task<Result<Guid>> PlaceOrderAsync(PlaceOrderDto orderDto)
        {
            // Get customer
            var customer = await _customerRepository.GetByIdAsync(orderDto.CustomerId);
            if (customer == null)
                return Result.Failure<Guid>("Customer not found");
                
            // Create order
            var order = new Order(customer.Id, DateTime.UtcNow);
            
            // Add items to order
            foreach (var item in orderDto.Items)
            {
                var product = await _productRepository.GetByIdAsync(item.ProductId);
                if (product == null)
                    return Result.Failure<Guid>($"Product {item.ProductId} not found");
                    
                if (product.Stock < item.Quantity)
                    return Result.Failure<Guid>($"Insufficient stock for product {product.Name}");
                    
                // Reduce product stock
                product.ReduceStock(item.Quantity);
                await _productRepository.UpdateAsync(product);
                
                // Add item to order
                order.AddItem(product.Id, item.Quantity, product.Price);
            }
            
            // Add domain event
            order.AddDomainEvent(new OrderPlacedEvent(order, customer));
            
            // Save order
            await _orderRepository.AddAsync(order);
            await _unitOfWork.SaveChangesAsync();
            
            // Send confirmation email
            await _emailService.SendOrderConfirmationAsync(customer.Email.Value, order.Id);
            
            return Result.Success(order.Id);
        }
    }
}
```

### 3. Infrastructure Layer

#### AppDbContext
```csharp
namespace CleanArchitecture.Infrastructure.Data
{
    public class AppDbContext : DbContext
    {
        public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
        
        public DbSet<Customer> Customers { get; set; }
        public DbSet<Order> Orders { get; set; }
        public DbSet<Product> Products { get; set; }
        
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);
        }
        
        public override async Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
        {
            // Before saving, dispatch domain events
            var entities = ChangeTracker.Entries<Entity>()
                .Where(e => e.Entity.DomainEvents.Any())
                .Select(e => e.Entity);
                
            var domainEvents = entities
                .SelectMany(e => e.DomainEvents)
                .ToList();
                
            entities.ToList().ForEach(e => e.ClearDomainEvents());
            
            foreach (var domainEvent in domainEvents)
            {
                // Use MediatR to publish domain events
                await _mediator.Publish(domainEvent);
            }
            
            return await base.SaveChangesAsync(cancellationToken);
        }
    }
}
```

#### OrderRepository
```csharp
namespace CleanArchitecture.Infrastructure.Data.Repositories
{
    public class OrderRepository : IOrderRepository
    {
        private readonly AppDbContext _dbContext;
        
        public OrderRepository(AppDbContext dbContext)
        {
            _dbContext = dbContext;
        }
        
        public async Task<Order> GetByIdAsync(Guid id)
        {
            return await _dbContext.Orders
                .Include(o => o.Items)
                .FirstOrDefaultAsync(o => o.Id == id);
        }
        
        public async Task<IEnumerable<Order>> GetByCustomerIdAsync(Guid customerId)
        {
            return await _dbContext.Orders
                .Include(o => o.Items)
                .Where(o => o.CustomerId == customerId)
                .ToListAsync();
        }
        
        public async Task AddAsync(Order order)
        {
            await _dbContext.Orders.AddAsync(order);
        }
        
        public Task UpdateAsync(Order order)
        {
            _dbContext.Entry(order).State = EntityState.Modified;
            return Task.CompletedTask;
        }
        
        public Task DeleteAsync(Order order)
        {
            _dbContext.Orders.Remove(order);
            return Task.CompletedTask;
        }
    }
}
```

#### DependencyInjection
```csharp
namespace CleanArchitecture.Infrastructure
{
    public static class DependencyInjection
    {
        public static IServiceCollection AddInfrastructure(
            this IServiceCollection services, 
            IConfiguration configuration)
        {
            // Add DbContext
            services.AddDbContext<AppDbContext>(options =>
                options.UseSqlServer(configuration.GetConnectionString("DefaultConnection"),
                b => b.MigrationsAssembly(typeof(AppDbContext).Assembly.FullName)));
            
            // Add repositories
            services.AddScoped<ICustomerRepository, CustomerRepository>();
            services.AddScoped<IOrderRepository, OrderRepository>();
            services.AddScoped<IProductRepository, ProductRepository>();
            services.AddScoped<IUnitOfWork, UnitOfWork>();
            
            // Add services
            services.AddTransient<IEmailService, EmailService>();
            services.AddTransient<IPaymentService, PaymentService>();
            
            return services;
        }
    }
}
```

### 4. Presentation Layer (API)

#### OrdersController
```csharp
namespace CleanArchitecture.API.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class OrdersController : ControllerBase
    {
        private readonly IOrderService _orderService;
        
        public OrdersController(IOrderService orderService)
        {
            _orderService = orderService;
        }
        
        [HttpGet("{id}")]
        public async Task<ActionResult<OrderViewModel>> Get(Guid id)
        {
            var order = await _orderService.GetOrderByIdAsync(id);
            
            if (order == null)
                return NotFound();
                
            var viewModel = new OrderViewModel
            {
                Id = order.Id,
                CustomerId = order.CustomerId,
                OrderDate = order.OrderDate,
                Status = order.Status,
                TotalAmount = order.TotalAmount,
                Items = order.Items.Select(i => new OrderItemViewModel
                {
                    ProductId = i.ProductId,
                    Quantity = i.Quantity,
                    Price = i.Price
                }).ToList()
            };
            
            return Ok(viewModel);
        }
        
        [HttpPost]
        public async Task<ActionResult<Guid>> Create(CreateOrderViewModel model)
        {
            var orderDto = new PlaceOrderDto
            {
                CustomerId = model.CustomerId,
                Items = model.Items.Select(i => new OrderItemDto
                {
                    ProductId = i.ProductId,
                    Quantity = i.Quantity
                }).ToList()
            };
            
            var result = await _orderService.PlaceOrderAsync(orderDto);
            
            if (result.IsFailure)
                return BadRequest(result.Error);
                
            return CreatedAtAction(nameof(Get), new { id = result.Value }, result.Value);
        }
    }
}
```

#### Program.cs
```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Add application services
builder.Services.AddApplication();

// Add infrastructure services
builder.Services.AddInfrastructure(builder.Configuration);

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

## ⚡ CQRS และ MediatR ใน Clean Architecture

### Command Query Responsibility Segregation (CQRS)

CQRS เป็นรูปแบบการออกแบบที่แยกการอ่าน (queries) ออกจากการเขียน (commands) ซึ่งช่วยให้แต่ละส่วนมีความเรียบง่ายและสามารถเพิ่มประสิทธิภาพได้

#### ประโยชน์ของ CQRS
1. **แยกความซับซ้อน** - การแยกโมเดลการอ่าน/เขียนทำให้แต่ละส่วนไม่ซับซ้อน
2. **ขยายได้** - สามารถปรับขนาดส่วนการอ่านและการเขียนแยกกันได้
3. **ปรับแต่ง** - สามารถปรับแต่งแต่ละส่วนให้เหมาะกับความต้องการได้
4. **ทดสอบได้** - ทำให้การทดสอบง่ายขึ้น

### การใช้ MediatR ใน Clean Architecture

MediatR เป็น library ที่ใช้ในการ implement pattern Mediator ซึ่งช่วยในการส่งข้อความระหว่างคอมโพเนนต์ในแอปพลิเคชัน โดยลดการพึ่งพาโดยตรงระหว่างคอมโพเนนต์

```csharp
// 1. ติดตั้ง MediatR
// dotnet add package MediatR
// dotnet add package MediatR.Extensions.Microsoft.DependencyInjection

// 2. สร้าง Command
namespace CleanArchitecture.Application.Orders.Commands.PlaceOrder
{
    public class PlaceOrderCommand : IRequest<Result<Guid>>
    {
        public Guid CustomerId { get; set; }
        public List<OrderItemDto> Items { get; set; }
    }
    
    public class PlaceOrderCommandHandler : IRequestHandler<PlaceOrderCommand, Result<Guid>>
    {
        private readonly IOrderRepository _orderRepository;
        private readonly IProductRepository _productRepository;
        private readonly ICustomerRepository _customerRepository;
        private readonly IUnitOfWork _unitOfWork;
        
        public PlaceOrderCommandHandler(
            IOrderRepository orderRepository,
            IProductRepository productRepository,
            ICustomerRepository customerRepository,
            IUnitOfWork unitOfWork)
        {
            _orderRepository = orderRepository;
            _productRepository = productRepository;
            _customerRepository = customerRepository;
            _unitOfWork = unitOfWork;
        }
        
        public async Task<Result<Guid>> Handle(PlaceOrderCommand request, CancellationToken cancellationToken)
        {
            // Implement the same logic as in OrderService.PlaceOrderAsync
            // ...
            
            return Result.Success(orderId);
        }
    }
}

// 3. สร้าง Query
namespace CleanArchitecture.Application.Orders.Queries.GetOrderById
{
    public class GetOrderByIdQuery : IRequest<OrderDto>
    {
        public Guid Id { get; set; }
    }
    
    public class GetOrderByIdQueryHandler : IRequestHandler<GetOrderByIdQuery, OrderDto>
    {
        private readonly IOrderRepository _orderRepository;
        
        public GetOrderByIdQueryHandler(IOrderRepository orderRepository)
        {
            _orderRepository = orderRepository;
        }
        
        public async Task<OrderDto> Handle(GetOrderByIdQuery request, CancellationToken cancellationToken)
        {
            var order = await _orderRepository.GetByIdAsync(request.Id);
            if (order == null)
                return null;
                
            return new OrderDto
            {
                Id = order.Id,
                CustomerId = order.CustomerId,
                OrderDate = order.OrderDate,
                Status = order.Status.ToString(),
                TotalAmount = order.TotalAmount,
                Items = order.Items.Select(i => new OrderItemDto
                {
                    ProductId = i.ProductId,
                    Quantity = i.Quantity,
                    Price = i.Price
                }).ToList()
            };
        }
    }
}

// 4. ใช้ใน Controller
namespace CleanArchitecture.API.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class OrdersController : ControllerBase
    {
        private readonly IMediator _mediator;
        
        public OrdersController(IMediator mediator)
        {
            _mediator = mediator;
        }
        
        [HttpGet("{id}")]
        public async Task<ActionResult<OrderViewModel>> Get(Guid id)
        {
            var order = await _mediator.Send(new GetOrderByIdQuery { Id = id });
            
            if (order == null)
                return NotFound();
                
            var viewModel = new OrderViewModel
            {
                Id = order.Id,
                CustomerId = order.CustomerId,
                OrderDate = order.OrderDate,
                Status = order.Status,
                TotalAmount = order.TotalAmount,
                Items = order.Items.Select(i => new OrderItemViewModel
                {
                    ProductId = i.ProductId,
                    Quantity = i.Quantity,
                    Price = i.Price
                }).ToList()
            };
            
            return Ok(viewModel);
        }
        
        [HttpPost]
        public async Task<ActionResult<Guid>> Create(CreateOrderViewModel model)
        {
            var command = new PlaceOrderCommand
            {
                CustomerId = model.CustomerId,
                Items = model.Items.Select(i => new OrderItemDto
                {
                    ProductId = i.ProductId,
                    Quantity = i.Quantity
                }).ToList()
            };
            
            var result = await _mediator.Send(command);
            
            if (result.IsFailure)
                return BadRequest(result.Error);
                
            return CreatedAtAction(nameof(Get), new { id = result.Value }, result.Value);
        }
    }
}

// 5. ลงทะเบียน MediatR ใน DI container
public static class ApplicationServiceRegistration
{
    public static IServiceCollection AddApplication(this IServiceCollection services)
    {
        services.AddMediatR(cfg => cfg.RegisterServicesFromAssembly(typeof(ApplicationServiceRegistration).Assembly));
        
        // Add behaviors
        services.AddTransient(typeof(IPipelineBehavior<,>), typeof(ValidationBehavior<,>));
        services.AddTransient(typeof(IPipelineBehavior<,>), typeof(LoggingBehavior<,>));
        
        return services;
    }
}
```

## ♻️ Dependency Injection ใน Clean Architecture

Dependency Injection (DI) เป็นเทคนิคสำคัญใน Clean Architecture ที่ช่วยให้สามารถทดแทน implementations ได้โดยไม่กระทบต่อ business logic

### วิธีการทำ Dependency Injection ใน .NET 9

```csharp
// 1. Domain Layer - ไม่มีการขึ้นต่อ external frameworks
// 2. Application Layer - ลงทะเบียน services และ behaviors

// ApplicationServiceRegistration.cs
namespace CleanArchitecture.Application
{
    public static class ApplicationServiceRegistration
    {
        public static IServiceCollection AddApplicationServices(this IServiceCollection services)
        {
            // Register MediatR
            services.AddMediatR(cfg => cfg.RegisterServicesFromAssembly(typeof(ApplicationServiceRegistration).Assembly));
            
            // Register behaviors
            services.AddTransient(typeof(IPipelineBehavior<,>), typeof(ValidationBehavior<,>));
            services.AddTransient(typeof(IPipelineBehavior<,>), typeof(LoggingBehavior<,>));
            
            // Register FluentValidation
            services.AddValidatorsFromAssembly(typeof(ApplicationServiceRegistration).Assembly);
            
            // Register AutoMapper
            services.AddAutoMapper(typeof(ApplicationServiceRegistration).Assembly);
            
            return services;
        }
    }
}

// 3. Infrastructure Layer - ลงทะเบียน implementations ของ interfaces

// InfrastructureServiceRegistration.cs
namespace CleanArchitecture.Infrastructure
{
    public static class InfrastructureServiceRegistration
    {
        public static IServiceCollection AddInfrastructureServices(
            this IServiceCollection services, 
            IConfiguration configuration)
        {
            // Register DbContext
            services.AddDbContext<AppDbContext>(options =>
                options.UseSqlServer(configuration.GetConnectionString("DefaultConnection"),
                    b => b.MigrationsAssembly(typeof(AppDbContext).Assembly.FullName)));
            
            // Register repositories
            services.AddScoped<ICustomerRepository, CustomerRepository>();
            services.AddScoped<IOrderRepository, OrderRepository>();
            services.AddScoped<IProductRepository, ProductRepository>();
            services.AddScoped<IUnitOfWork, UnitOfWork>();
            
            // Register services
            services.AddTransient<IEmailService, EmailService>();
            services.AddTransient<IDateTime, DateTimeService>();
            services.AddTransient<IIdentityService, IdentityService>();
            
            return services;
        }
    }
}

// 4. Presentation Layer (API) - ลงทะเบียน controllers และ UI services

// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Add services to the container
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "Clean Architecture API", Version = "v1" });
});

// Add application services
builder.Services.AddApplicationServices();

// Add infrastructure services
builder.Services.AddInfrastructureServices(builder.Configuration);

// Add API services
builder.Services.AddScoped<ICurrentUserService, CurrentUserService>();
builder.Services.AddHttpContextAccessor();

// Add CORS
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowSpecificOrigin",
        builder => builder
            .WithOrigins("https://example.com")
            .AllowAnyMethod()
            .AllowAnyHeader());
});

var app = builder.Build();

// Configure middleware
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
    app.UseSwagger();
    app.UseSwaggerUI(c => c.SwaggerEndpoint("/swagger/v1/swagger.json", "Clean Architecture API v1"));
}

app.UseHttpsRedirection();
app.UseRouting();
app.UseCors("AllowSpecificOrigin");
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

## 🔍 การจัดการข้อมูลและ Entity Framework Core ใน Clean Architecture

Entity Framework Core (EF Core) เป็นเครื่องมือ ORM (Object-Relational Mapping) ที่นิยมใช้ใน .NET โดยในบริบทของ Clean Architecture จะถูกใช้ในชั้น Infrastructure เท่านั้น

### การกำหนดค่า Entity Framework Core

```csharp
// 1. Entity Configuration
namespace CleanArchitecture.Infrastructure.Data.Configurations
{
    public class CustomerConfiguration : IEntityTypeConfiguration<Customer>
    {
        public void Configure(EntityTypeBuilder<Customer> builder)
        {
            builder.HasKey(c => c.Id);
            
            builder.Property(c => c.FirstName)
                .HasMaxLength(50)
                .IsRequired();
                
            builder.Property(c => c.LastName)
                .HasMaxLength(50)
                .IsRequired();
                
            builder.OwnsOne(c => c.Email, email =>
            {
                email.Property(e => e.Value)
                    .HasColumnName("Email")
                    .HasMaxLength(255)
                    .IsRequired();
            });
            
            builder.OwnsOne(c => c.Address, address =>
            {
                address.Property(a => a.Street)
                    .HasColumnName("Street")
                    .HasMaxLength(100);
                    
                address.Property(a => a.City)
                    .HasColumnName("City")
                    .HasMaxLength(50);
                    
                address.Property(a => a.State)
                    .HasColumnName("State")
                    .HasMaxLength(50);
                    
                address.Property(a => a.ZipCode)
                    .HasColumnName("ZipCode")
                    .HasMaxLength(20);
                    
                address.Property(a => a.Country)
                    .HasColumnName("Country")
                    .HasMaxLength(50);
            });
            
            builder.HasMany(c => c.Orders)
                .WithOne()
                .HasForeignKey(o => o.CustomerId)
                .OnDelete(DeleteBehavior.Cascade);
        }
    }
}

// 2. AppDbContext
namespace CleanArchitecture.Infrastructure.Data
{
    public class AppDbContext : DbContext
    {
        private readonly IMediator _mediator;
        
        public AppDbContext(
            DbContextOptions<AppDbContext> options,
            IMediator mediator) : base(options)
        {
            _mediator = mediator;
        }
        
        public DbSet<Customer> Customers { get; set; }
        public DbSet<Order> Orders { get; set; }
        public DbSet<Product> Products { get; set; }
        
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);
        }
        
        public override async Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
        {
            // Before saving, dispatch domain events
            await DispatchEvents();
            
            return await base.SaveChangesAsync(cancellationToken);
        }
        
        private async Task DispatchEvents()
        {
            var entities = ChangeTracker.Entries<Entity>()
                .Where(e => e.Entity.DomainEvents.Any())
                .Select(e => e.Entity);
                
            var domainEvents = entities
                .SelectMany(e => e.DomainEvents)
                .ToList();
                
            entities.ToList().ForEach(e => e.ClearDomainEvents());
            
            foreach (var domainEvent in domainEvents)
            {
                await _mediator.Publish(domainEvent);
            }
        }
    }
}

// 3. UnitOfWork
namespace CleanArchitecture.Infrastructure.Data
{
    public class UnitOfWork : IUnitOfWork
    {
        private readonly AppDbContext _context;
        
        public UnitOfWork(AppDbContext context)
        {
            _context = context;
        }
        
        public async Task<int> SaveChangesAsync()
        {
            return await _context.SaveChangesAsync();
        }
    }
}
```

## 📝 แบบฝึกหัด Clean Architecture

### แบบฝึกหัด 1: สร้าง Clean Architecture Solution
1. สร้าง solution ที่มีโครงสร้างตามหลัก Clean Architecture
2. กำหนดขอบเขตและความรับผิดชอบของแต่ละ layer
3. ใช้ Domain-Driven Design (DDD) ในการออกแบบ Domain Layer

### แบบฝึกหัด 2: เพิ่ม CQRS และ MediatR
1. ปรับปรุง solution จากแบบฝึกหัด 1 ให้ใช้ CQRS pattern
2. เพิ่ม MediatR เพื่อจัดการ commands และ queries
3. เพิ่ม validation behaviors และ logging behaviors

### แบบฝึกหัด 3: ทำ Repository Pattern และ Entity Framework Core
1. สร้าง DbContext และ repositories ใน Infrastructure Layer
2. เพิ่ม migrations และ seed data
3. ทำ unit tests และ integration tests สำหรับ repositories

## 📚 แหล่งข้อมูลเพิ่มเติม
- [Clean Architecture by Robert C. Martin (Uncle Bob)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Implementing Clean Architecture in .NET](https://docs.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/common-web-application-architectures#clean-architecture)
- [ASP.NET Core + Clean Architecture](https://github.com/jasontaylordev/CleanArchitecture)
- [MediatR GitHub Repository](https://github.com/jbogard/MediatR)
- [Entity Framework Core Documentation](https://docs.microsoft.com/en-us/ef/core/)

## 🚀 สรุป

Clean Architecture เป็นแนวทางการออกแบบซอฟต์แวร์ที่ช่วยให้โค้ดมีความยืดหยุ่น ทดสอบได้ง่าย และมีความยั่งยืนในระยะยาว โดยมีหลักการสำคัญคือการแยกความรับผิดชอบออกเป็น layers และการกำหนดกฎการพึ่งพาที่ชัดเจน

เมื่อนำมาใช้กับ .NET 9 ร่วมกับเทคนิคเช่น CQRS, MediatR, และ Entity Framework Core จะช่วยให้สามารถสร้างแอปพลิเคชันที่มีคุณภาพสูง ขยายได้ และบำรุงรักษาได้ง่าย
