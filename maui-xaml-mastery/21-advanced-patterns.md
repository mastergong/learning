# 🏗️ Chapter 21: Advanced Patterns & Architectural Strategies

หลังจากที่เราได้เรียนรู้เรื่อง deployment และ distribution แล้ว ในบทนี้เราจะมาเจาะลึกถึง advanced patterns และ architectural strategies ที่จะช่วยให้การพัฒนา .NET MAUI applications มีความ maintainable, scalable และ testable มากขึ้น

## 🎯 Learning Objectives

- เรียนรู้ Advanced MVVM patterns และ architectural approaches
- ใช้งาน Dependency Injection และ IoC containers ขั้นสูง
- สร้าง messaging systems และ event-driven architectures
- ประยุกต์ใช้ CQRS และ Event Sourcing patterns
- เข้าใจ Clean Architecture และ Domain-Driven Design
- สร้าง plugin architectures และ modular systems

---

## 1. 🏛️ Advanced MVVM Patterns

### 1.1 Enhanced Base Classes

#### Advanced ViewModelBase

```csharp
// ViewModels/Base/ViewModelBase.cs
public abstract class ViewModelBase : ObservableObject, INavigationAware, IInitializeAsync, IDisposable
{
    private readonly INavigationService _navigationService;
    private readonly IDialogService _dialogService;
    private readonly IAnalyticsService _analyticsService;
    
    protected ViewModelBase(
        INavigationService navigationService,
        IDialogService dialogService,
        IAnalyticsService analyticsService)
    {
        _navigationService = navigationService;
        _dialogService = dialogService;
        _analyticsService = analyticsService;
        
        // Initialize common commands
        InitializeCommands();
        
        // Subscribe to property changes for analytics
        PropertyChanged += OnPropertyChangedForAnalytics;
    }
    
    #region Properties
    
    private bool _isBusy;
    public bool IsBusy
    {
        get => _isBusy;
        set
        {
            if (SetProperty(ref _isBusy, value))
            {
                OnPropertyChanged(nameof(IsNotBusy));
                // Refresh command can execute states
                RefreshAllCommands();
            }
        }
    }
    
    public bool IsNotBusy => !IsBusy;
    
    private string _title = string.Empty;
    public string Title
    {
        get => _title;
        set => SetProperty(ref _title, value);
    }
    
    private string _subtitle = string.Empty;
    public string Subtitle
    {
        get => _subtitle;
        set => SetProperty(ref _subtitle, value);
    }
    
    #endregion
    
    #region Commands
    
    private readonly List<ICommand> _commands = new();
    
    protected void RegisterCommand(ICommand command)
    {
        _commands.Add(command);
    }
    
    private void RefreshAllCommands()
    {
        foreach (var command in _commands.OfType<IRelayCommand>())
        {
            command.NotifyCanExecuteChanged();
        }
    }
    
    protected virtual void InitializeCommands()
    {
        GoBackCommand = new AsyncRelayCommand(GoBack, CanGoBack);
        RefreshCommand = new AsyncRelayCommand(Refresh, CanRefresh);
        
        RegisterCommand(GoBackCommand);
        RegisterCommand(RefreshCommand);
    }
    
    public IAsyncRelayCommand GoBackCommand { get; private set; }
    public IAsyncRelayCommand RefreshCommand { get; private set; }
    
    #endregion
    
    #region Navigation Methods
    
    protected virtual bool CanGoBack() => _navigationService.CanGoBack && !IsBusy;
    
    protected virtual async Task GoBack()
    {
        if (CanGoBack())
        {
            await _navigationService.GoBackAsync();
        }
    }
    
    protected virtual bool CanRefresh() => !IsBusy;
    protected virtual async Task Refresh() { }
    
    #endregion
    
    #region INavigationAware Implementation
    
    public virtual Task OnNavigatedToAsync(INavigationParameters parameters)
    {
        _analyticsService.TrackPageView(GetType().Name, parameters?.GetValue<string>("source"));
        return Task.CompletedTask;
    }
    
    public virtual Task OnNavigatedFromAsync(INavigationParameters parameters)
    {
        return Task.CompletedTask;
    }
    
    #endregion
    
    #region IInitializeAsync Implementation
    
    public virtual async Task InitializeAsync(INavigationParameters parameters)
    {
        await LoadDataAsync(parameters);
    }
    
    protected virtual Task LoadDataAsync(INavigationParameters parameters)
    {
        return Task.CompletedTask;
    }
    
    #endregion
    
    #region Error Handling
    
    protected async Task ExecuteSafelyAsync(Func<Task> operation, [CallerMemberName] string operationName = null)
    {
        try
        {
            IsBusy = true;
            await operation();
        }
        catch (Exception ex)
        {
            await HandleErrorAsync(ex, operationName);
        }
        finally
        {
            IsBusy = false;
        }
    }
    
    protected async Task<T> ExecuteSafelyAsync<T>(Func<Task<T>> operation, T defaultValue = default, [CallerMemberName] string operationName = null)
    {
        try
        {
            IsBusy = true;
            return await operation();
        }
        catch (Exception ex)
        {
            await HandleErrorAsync(ex, operationName);
            return defaultValue;
        }
        finally
        {
            IsBusy = false;
        }
    }
    
    protected virtual async Task HandleErrorAsync(Exception exception, string operationName)
    {
        _analyticsService.TrackError(exception, new Dictionary<string, string>
        {
            { "ViewModel", GetType().Name },
            { "Operation", operationName ?? "Unknown" }
        });
        
        await _dialogService.ShowErrorAsync("An error occurred", exception.Message);
    }
    
    #endregion
    
    #region Analytics
    
    private void OnPropertyChangedForAnalytics(object sender, PropertyChangedEventArgs e)
    {
        // Track important property changes
        if (e.PropertyName is nameof(Title) or nameof(IsBusy))
            return; // Skip common properties
            
        _analyticsService.TrackEvent("PropertyChanged", new Dictionary<string, string>
        {
            { "ViewModel", GetType().Name },
            { "PropertyName", e.PropertyName }
        });
    }
    
    protected void TrackUserAction(string action, Dictionary<string, string> additionalData = null)
    {
        var data = new Dictionary<string, string>
        {
            { "ViewModel", GetType().Name },
            { "Action", action }
        };
        
        if (additionalData != null)
        {
            foreach (var kvp in additionalData)
                data[kvp.Key] = kvp.Value;
        }
        
        _analyticsService.TrackUserAction(action, GetType().Name, data);
    }
    
    #endregion
    
    #region IDisposable
    
    private bool _disposed;
    
    public virtual void Dispose()
    {
        if (!_disposed)
        {
            PropertyChanged -= OnPropertyChangedForAnalytics;
            OnDispose(true);
            _disposed = true;
        }
        
        GC.SuppressFinalize(this);
    }
    
    protected virtual void OnDispose(bool disposing)
    {
        if (disposing)
        {
            // Dispose managed resources
            foreach (var command in _commands.OfType<IDisposable>())
            {
                command.Dispose();
            }
            _commands.Clear();
        }
    }
    
    ~ViewModelBase()
    {
        OnDispose(false);
    }
    
    #endregion
}

// Supporting interfaces
public interface INavigationAware
{
    Task OnNavigatedToAsync(INavigationParameters parameters);
    Task OnNavigatedFromAsync(INavigationParameters parameters);
}

public interface IInitializeAsync
{
    Task InitializeAsync(INavigationParameters parameters);
}
```

### 1.2 Repository Pattern with Unit of Work

#### Generic Repository Implementation

```csharp
// Repository/IRepository.cs
public interface IRepository<T> where T : class, IEntity
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate);
    Task<T> SingleOrDefaultAsync(Expression<Func<T, bool>> predicate);
    Task<bool> ExistsAsync(Expression<Func<T, bool>> predicate);
    Task<T> AddAsync(T entity);
    Task<IEnumerable<T>> AddRangeAsync(IEnumerable<T> entities);
    Task UpdateAsync(T entity);
    Task DeleteAsync(T entity);
    Task DeleteAsync(int id);
    Task<int> CountAsync();
    Task<int> CountAsync(Expression<Func<T, bool>> predicate);
}

// Repository/Repository.cs
public class Repository<T> : IRepository<T> where T : class, IEntity
{
    protected readonly DbContext _context;
    protected readonly DbSet<T> _dbSet;
    
    public Repository(DbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }
    
    public virtual async Task<T> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }
    
    public virtual async Task<IEnumerable<T>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }
    
    public virtual async Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate)
    {
        return await _dbSet.Where(predicate).ToListAsync();
    }
    
    public virtual async Task<T> SingleOrDefaultAsync(Expression<Func<T, bool>> predicate)
    {
        return await _dbSet.SingleOrDefaultAsync(predicate);
    }
    
    public virtual async Task<bool> ExistsAsync(Expression<Func<T, bool>> predicate)
    {
        return await _dbSet.AnyAsync(predicate);
    }
    
    public virtual async Task<T> AddAsync(T entity)
    {
        _dbSet.Add(entity);
        return entity;
    }
    
    public virtual async Task<IEnumerable<T>> AddRangeAsync(IEnumerable<T> entities)
    {
        _dbSet.AddRange(entities);
        return entities;
    }
    
    public virtual async Task UpdateAsync(T entity)
    {
        _dbSet.Update(entity);
        await Task.CompletedTask;
    }
    
    public virtual async Task DeleteAsync(T entity)
    {
        _dbSet.Remove(entity);
        await Task.CompletedTask;
    }
    
    public virtual async Task DeleteAsync(int id)
    {
        var entity = await GetByIdAsync(id);
        if (entity != null)
        {
            await DeleteAsync(entity);
        }
    }
    
    public virtual async Task<int> CountAsync()
    {
        return await _dbSet.CountAsync();
    }
    
    public virtual async Task<int> CountAsync(Expression<Func<T, bool>> predicate)
    {
        return await _dbSet.CountAsync(predicate);
    }
}
```

#### Unit of Work Pattern

```csharp
// Repository/IUnitOfWork.cs
public interface IUnitOfWork : IDisposable
{
    IRepository<T> Repository<T>() where T : class, IEntity;
    Task<int> SaveChangesAsync();
    Task BeginTransactionAsync();
    Task CommitTransactionAsync();
    Task RollbackTransactionAsync();
    Task ExecuteInTransactionAsync(Func<Task> operation);
    Task<T> ExecuteInTransactionAsync<T>(Func<Task<T>> operation);
}

// Repository/UnitOfWork.cs
public class UnitOfWork : IUnitOfWork
{
    private readonly DbContext _context;
    private readonly Dictionary<Type, object> _repositories = new();
    private IDbContextTransaction _transaction;
    
    public UnitOfWork(DbContext context)
    {
        _context = context;
    }
    
    public IRepository<T> Repository<T>() where T : class, IEntity
    {
        var type = typeof(T);
        
        if (!_repositories.ContainsKey(type))
        {
            var repositoryType = typeof(Repository<>).MakeGenericType(type);
            var repository = Activator.CreateInstance(repositoryType, _context);
            _repositories[type] = repository;
        }
        
        return (IRepository<T>)_repositories[type];
    }
    
    public async Task<int> SaveChangesAsync()
    {
        try
        {
            return await _context.SaveChangesAsync();
        }
        catch (Exception ex)
        {
            // Log error
            throw new RepositoryException("Failed to save changes", ex);
        }
    }
    
    public async Task BeginTransactionAsync()
    {
        _transaction = await _context.Database.BeginTransactionAsync();
    }
    
    public async Task CommitTransactionAsync()
    {
        if (_transaction != null)
        {
            await _transaction.CommitAsync();
            await _transaction.DisposeAsync();
            _transaction = null;
        }
    }
    
    public async Task RollbackTransactionAsync()
    {
        if (_transaction != null)
        {
            await _transaction.RollbackAsync();
            await _transaction.DisposeAsync();
            _transaction = null;
        }
    }
    
    public async Task ExecuteInTransactionAsync(Func<Task> operation)
    {
        await BeginTransactionAsync();
        try
        {
            await operation();
            await SaveChangesAsync();
            await CommitTransactionAsync();
        }
        catch
        {
            await RollbackTransactionAsync();
            throw;
        }
    }
    
    public async Task<T> ExecuteInTransactionAsync<T>(Func<Task<T>> operation)
    {
        await BeginTransactionAsync();
        try
        {
            var result = await operation();
            await SaveChangesAsync();
            await CommitTransactionAsync();
            return result;
        }
        catch
        {
            await RollbackTransactionAsync();
            throw;
        }
    }
    
    public void Dispose()
    {
        _transaction?.Dispose();
        _context?.Dispose();
    }
}

public class RepositoryException : Exception
{
    public RepositoryException(string message) : base(message) { }
    public RepositoryException(string message, Exception innerException) : base(message, innerException) { }
}
```

---

## 2. 💬 Messaging & Event-Driven Architecture

### 2.1 Advanced Messaging System

#### Message Bus Implementation

```csharp
// Messaging/IMessageBus.cs
public interface IMessageBus
{
    Task PublishAsync<T>(T message) where T : class, IMessage;
    void Subscribe<T>(Func<T, Task> handler) where T : class, IMessage;
    void Subscribe<T>(IMessageHandler<T> handler) where T : class, IMessage;
    void Unsubscribe<T>(Func<T, Task> handler) where T : class, IMessage;
    void Unsubscribe<T>(IMessageHandler<T> handler) where T : class, IMessage;
}

// Messaging/MessageBus.cs
public class MessageBus : IMessageBus
{
    private readonly ConcurrentDictionary<Type, List<object>> _subscriptions = new();
    private readonly ILogger<MessageBus> _logger;
    private readonly IServiceProvider _serviceProvider;
    
    public MessageBus(ILogger<MessageBus> logger, IServiceProvider serviceProvider)
    {
        _logger = logger;
        _serviceProvider = serviceProvider;
    }
    
    public async Task PublishAsync<T>(T message) where T : class, IMessage
    {
        var messageType = typeof(T);
        _logger.LogDebug("Publishing message of type {MessageType}", messageType.Name);
        
        if (!_subscriptions.TryGetValue(messageType, out var handlers))
        {
            _logger.LogWarning("No handlers found for message type {MessageType}", messageType.Name);
            return;
        }
        
        var tasks = new List<Task>();
        
        foreach (var handler in handlers.ToList()) // Create copy to avoid enumeration issues
        {
            try
            {
                var task = handler switch
                {
                    Func<T, Task> funcHandler => funcHandler(message),
                    IMessageHandler<T> messageHandler => messageHandler.HandleAsync(message),
                    _ => Task.CompletedTask
                };
                
                tasks.Add(task);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error executing handler for message type {MessageType}", messageType.Name);
            }
        }
        
        await Task.WhenAll(tasks);
        _logger.LogDebug("Completed publishing message of type {MessageType}", messageType.Name);
    }
    
    public void Subscribe<T>(Func<T, Task> handler) where T : class, IMessage
    {
        var messageType = typeof(T);
        _subscriptions.AddOrUpdate(
            messageType,
            new List<object> { handler },
            (key, existing) =>
            {
                existing.Add(handler);
                return existing;
            });
            
        _logger.LogDebug("Subscribed func handler to message type {MessageType}", messageType.Name);
    }
    
    public void Subscribe<T>(IMessageHandler<T> handler) where T : class, IMessage
    {
        var messageType = typeof(T);
        _subscriptions.AddOrUpdate(
            messageType,
            new List<object> { handler },
            (key, existing) =>
            {
                existing.Add(handler);
                return existing;
            });
            
        _logger.LogDebug("Subscribed message handler to message type {MessageType}", messageType.Name);
    }
    
    public void Unsubscribe<T>(Func<T, Task> handler) where T : class, IMessage
    {
        var messageType = typeof(T);
        if (_subscriptions.TryGetValue(messageType, out var handlers))
        {
            handlers.Remove(handler);
            _logger.LogDebug("Unsubscribed func handler from message type {MessageType}", messageType.Name);
        }
    }
    
    public void Unsubscribe<T>(IMessageHandler<T> handler) where T : class, IMessage
    {
        var messageType = typeof(T);
        if (_subscriptions.TryGetValue(messageType, out var handlers))
        {
            handlers.Remove(handler);
            _logger.LogDebug("Unsubscribed message handler from message type {MessageType}", messageType.Name);
        }
    }
}

// Messaging/IMessage.cs
public interface IMessage
{
    Guid Id { get; }
    DateTime Timestamp { get; }
    string Source { get; }
}

// Messaging/IMessageHandler.cs
public interface IMessageHandler<T> where T : class, IMessage
{
    Task HandleAsync(T message);
}

// Messaging/MessageBase.cs
public abstract class MessageBase : IMessage
{
    protected MessageBase(string source = null)
    {
        Id = Guid.NewGuid();
        Timestamp = DateTime.UtcNow;
        Source = source ?? GetType().Name;
    }
    
    public Guid Id { get; }
    public DateTime Timestamp { get; }
    public string Source { get; }
}
```

### 2.2 Domain Events

#### Domain Event System

```csharp
// Domain/Events/IDomainEvent.cs
public interface IDomainEvent : IMessage
{
    string AggregateId { get; }
    int Version { get; }
}

// Domain/Events/DomainEventBase.cs
public abstract class DomainEventBase : MessageBase, IDomainEvent
{
    protected DomainEventBase(string aggregateId, int version, string source = null) : base(source)
    {
        AggregateId = aggregateId;
        Version = version;
    }
    
    public string AggregateId { get; }
    public int Version { get; }
}

// Domain/Events/UserRegisteredEvent.cs
public class UserRegisteredEvent : DomainEventBase
{
    public UserRegisteredEvent(string userId, string email, string name, int version) 
        : base(userId, version, nameof(UserRegisteredEvent))
    {
        UserId = userId;
        Email = email;
        Name = name;
    }
    
    public string UserId { get; }
    public string Email { get; }
    public string Name { get; }
}

// Domain/Events/OrderCompletedEvent.cs
public class OrderCompletedEvent : DomainEventBase
{
    public OrderCompletedEvent(string orderId, string customerId, decimal totalAmount, int version)
        : base(orderId, version, nameof(OrderCompletedEvent))
    {
        OrderId = orderId;
        CustomerId = customerId;
        TotalAmount = totalAmount;
        CompletedAt = DateTime.UtcNow;
    }
    
    public string OrderId { get; }
    public string CustomerId { get; }
    public decimal TotalAmount { get; }
    public DateTime CompletedAt { get; }
}
```

### 2.3 Event Handlers

#### User Event Handlers

```csharp
// Handlers/UserEventHandlers.cs
public class UserRegisteredEventHandler : IMessageHandler<UserRegisteredEvent>
{
    private readonly IAnalyticsService _analytics;
    private readonly IEmailService _emailService;
    private readonly ILogger<UserRegisteredEventHandler> _logger;
    
    public UserRegisteredEventHandler(
        IAnalyticsService analytics,
        IEmailService emailService,
        ILogger<UserRegisteredEventHandler> logger)
    {
        _analytics = analytics;
        _emailService = emailService;
        _logger = logger;
    }
    
    public async Task HandleAsync(UserRegisteredEvent message)
    {
        _logger.LogInformation("Handling user registration for {UserId}", message.UserId);
        
        try
        {
            // Track analytics
            _analytics.TrackEvent("UserRegistered", new Dictionary<string, string>
            {
                { "UserId", message.UserId },
                { "Email", message.Email },
                { "Name", message.Name }
            });
            
            // Send welcome email
            await _emailService.SendWelcomeEmailAsync(message.Email, message.Name);
            
            _logger.LogInformation("Successfully processed user registration for {UserId}", message.UserId);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to handle user registration for {UserId}", message.UserId);
            throw;
        }
    }
}

// Handlers/OrderEventHandlers.cs
public class OrderCompletedEventHandler : IMessageHandler<OrderCompletedEvent>
{
    private readonly INotificationService _notificationService;
    private readonly IInventoryService _inventoryService;
    private readonly IAnalyticsService _analytics;
    private readonly ILogger<OrderCompletedEventHandler> _logger;
    
    public OrderCompletedEventHandler(
        INotificationService notificationService,
        IInventoryService inventoryService,
        IAnalyticsService analytics,
        ILogger<OrderCompletedEventHandler> logger)
    {
        _notificationService = notificationService;
        _inventoryService = inventoryService;
        _analytics = analytics;
        _logger = logger;
    }
    
    public async Task HandleAsync(OrderCompletedEvent message)
    {
        _logger.LogInformation("Handling order completion for {OrderId}", message.OrderId);
        
        try
        {
            // Update inventory
            await _inventoryService.UpdateAfterOrderCompletionAsync(message.OrderId);
            
            // Send notification to customer
            await _notificationService.SendOrderCompletionNotificationAsync(
                message.CustomerId, 
                message.OrderId);
            
            // Track analytics
            _analytics.TrackEvent("OrderCompleted", new Dictionary<string, string>
            {
                { "OrderId", message.OrderId },
                { "CustomerId", message.CustomerId },
                { "TotalAmount", message.TotalAmount.ToString("C") }
            });
            
            _logger.LogInformation("Successfully processed order completion for {OrderId}", message.OrderId);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to handle order completion for {OrderId}", message.OrderId);
            throw;
        }
    }
}
```

---

## 3. 🎯 CQRS Pattern Implementation

### 3.1 Command Query Separation

#### Command Pattern

```csharp
// Commands/ICommand.cs
public interface ICommand<TResult>
{
    Guid Id { get; }
    DateTime Timestamp { get; }
}

// Commands/ICommandHandler.cs
public interface ICommandHandler<TCommand, TResult> where TCommand : ICommand<TResult>
{
    Task<TResult> HandleAsync(TCommand command);
}

// Commands/CreateUserCommand.cs
public class CreateUserCommand : ICommand<CreateUserResult>
{
    public CreateUserCommand(string email, string password, string firstName, string lastName)
    {
        Id = Guid.NewGuid();
        Timestamp = DateTime.UtcNow;
        Email = email;
        Password = password;
        FirstName = firstName;
        LastName = lastName;
    }
    
    public Guid Id { get; }
    public DateTime Timestamp { get; }
    public string Email { get; }
    public string Password { get; }
    public string FirstName { get; }
    public string LastName { get; }
}

public class CreateUserResult
{
    public string UserId { get; set; }
    public bool Success { get; set; }
    public string ErrorMessage { get; set; }
    public List<string> ValidationErrors { get; set; } = new();
}

// Commands/CreateUserCommandHandler.cs
public class CreateUserCommandHandler : ICommandHandler<CreateUserCommand, CreateUserResult>
{
    private readonly IUnitOfWork _unitOfWork;
    private readonly IPasswordHasher _passwordHasher;
    private readonly IMessageBus _messageBus;
    private readonly ILogger<CreateUserCommandHandler> _logger;
    
    public CreateUserCommandHandler(
        IUnitOfWork unitOfWork,
        IPasswordHasher passwordHasher,
        IMessageBus messageBus,
        ILogger<CreateUserCommandHandler> logger)
    {
        _unitOfWork = unitOfWork;
        _passwordHasher = passwordHasher;
        _messageBus = messageBus;
        _logger = logger;
    }
    
    public async Task<CreateUserResult> HandleAsync(CreateUserCommand command)
    {
        _logger.LogInformation("Creating user with email {Email}", command.Email);
        
        try
        {
            // Validate command
            var validationResult = await ValidateCommandAsync(command);
            if (!validationResult.IsValid)
            {
                return new CreateUserResult
                {
                    Success = false,
                    ValidationErrors = validationResult.Errors
                };
            }
            
            // Create user entity
            var user = new User
            {
                Id = Guid.NewGuid().ToString(),
                Email = command.Email,
                PasswordHash = _passwordHasher.HashPassword(command.Password),
                FirstName = command.FirstName,
                LastName = command.LastName,
                CreatedAt = DateTime.UtcNow,
                IsActive = true
            };
            
            // Save to database
            await _unitOfWork.Repository<User>().AddAsync(user);
            await _unitOfWork.SaveChangesAsync();
            
            // Publish domain event
            var userRegisteredEvent = new UserRegisteredEvent(
                user.Id, 
                user.Email, 
                $"{user.FirstName} {user.LastName}",
                1);
                
            await _messageBus.PublishAsync(userRegisteredEvent);
            
            _logger.LogInformation("Successfully created user {UserId}", user.Id);
            
            return new CreateUserResult
            {
                UserId = user.Id,
                Success = true
            };
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to create user with email {Email}", command.Email);
            
            return new CreateUserResult
            {
                Success = false,
                ErrorMessage = "An error occurred while creating the user"
            };
        }
    }
    
    private async Task<ValidationResult> ValidateCommandAsync(CreateUserCommand command)
    {
        var errors = new List<string>();
        
        // Email validation
        if (string.IsNullOrWhiteSpace(command.Email))
            errors.Add("Email is required");
        else if (!IsValidEmail(command.Email))
            errors.Add("Email format is invalid");
        else
        {
            // Check if email already exists
            var existingUser = await _unitOfWork.Repository<User>()
                .SingleOrDefaultAsync(u => u.Email == command.Email);
            if (existingUser != null)
                errors.Add("Email already exists");
        }
        
        // Password validation
        if (string.IsNullOrWhiteSpace(command.Password))
            errors.Add("Password is required");
        else if (command.Password.Length < 8)
            errors.Add("Password must be at least 8 characters long");
        
        // Name validation
        if (string.IsNullOrWhiteSpace(command.FirstName))
            errors.Add("First name is required");
            
        if (string.IsNullOrWhiteSpace(command.LastName))
            errors.Add("Last name is required");
        
        return new ValidationResult
        {
            IsValid = errors.Count == 0,
            Errors = errors
        };
    }
    
    private bool IsValidEmail(string email)
    {
        try
        {
            var addr = new System.Net.Mail.MailAddress(email);
            return addr.Address == email;
        }
        catch
        {
            return false;
        }
    }
}

public class ValidationResult
{
    public bool IsValid { get; set; }
    public List<string> Errors { get; set; } = new();
}
```

#### Query Pattern

```csharp
// Queries/IQuery.cs
public interface IQuery<TResult>
{
}

// Queries/IQueryHandler.cs
public interface IQueryHandler<TQuery, TResult> where TQuery : IQuery<TResult>
{
    Task<TResult> HandleAsync(TQuery query);
}

// Queries/GetUserByIdQuery.cs
public class GetUserByIdQuery : IQuery<UserDto>
{
    public GetUserByIdQuery(string userId)
    {
        UserId = userId;
    }
    
    public string UserId { get; }
}

public class UserDto
{
    public string Id { get; set; }
    public string Email { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string FullName => $"{FirstName} {LastName}";
    public DateTime CreatedAt { get; set; }
    public bool IsActive { get; set; }
}

// Queries/GetUserByIdQueryHandler.cs
public class GetUserByIdQueryHandler : IQueryHandler<GetUserByIdQuery, UserDto>
{
    private readonly IUnitOfWork _unitOfWork;
    private readonly IMapper _mapper;
    private readonly ILogger<GetUserByIdQueryHandler> _logger;
    
    public GetUserByIdQueryHandler(
        IUnitOfWork unitOfWork,
        IMapper mapper,
        ILogger<GetUserByIdQueryHandler> logger)
    {
        _unitOfWork = unitOfWork;
        _mapper = mapper;
        _logger = logger;
    }
    
    public async Task<UserDto> HandleAsync(GetUserByIdQuery query)
    {
        _logger.LogDebug("Getting user by ID {UserId}", query.UserId);
        
        try
        {
            var user = await _unitOfWork.Repository<User>().GetByIdAsync(int.Parse(query.UserId));
            
            if (user == null)
            {
                _logger.LogWarning("User not found with ID {UserId}", query.UserId);
                return null;
            }
            
            var userDto = _mapper.Map<UserDto>(user);
            
            _logger.LogDebug("Successfully retrieved user {UserId}", query.UserId);
            return userDto;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to get user by ID {UserId}", query.UserId);
            throw;
        }
    }
}
```

### 3.2 CQRS Service Implementation

#### Command/Query Dispatcher

```csharp
// Services/ICommandDispatcher.cs
public interface ICommandDispatcher
{
    Task<TResult> DispatchAsync<TResult>(ICommand<TResult> command);
}

// Services/IQueryDispatcher.cs
public interface IQueryDispatcher
{
    Task<TResult> DispatchAsync<TResult>(IQuery<TResult> query);
}

// Services/CqrsDispatcher.cs
public class CqrsDispatcher : ICommandDispatcher, IQueryDispatcher
{
    private readonly IServiceProvider _serviceProvider;
    private readonly ILogger<CqrsDispatcher> _logger;
    
    public CqrsDispatcher(IServiceProvider serviceProvider, ILogger<CqrsDispatcher> logger)
    {
        _serviceProvider = serviceProvider;
        _logger = logger;
    }
    
    public async Task<TResult> DispatchAsync<TResult>(ICommand<TResult> command)
    {
        var commandType = command.GetType();
        var handlerType = typeof(ICommandHandler<,>).MakeGenericType(commandType, typeof(TResult));
        
        var handler = _serviceProvider.GetRequiredService(handlerType);
        
        _logger.LogDebug("Dispatching command {CommandType} to handler {HandlerType}", 
            commandType.Name, handlerType.Name);
        
        var method = handlerType.GetMethod(nameof(ICommandHandler<ICommand<TResult>, TResult>.HandleAsync));
        var task = (Task<TResult>)method.Invoke(handler, new object[] { command });
        
        return await task;
    }
    
    public async Task<TResult> DispatchAsync<TResult>(IQuery<TResult> query)
    {
        var queryType = query.GetType();
        var handlerType = typeof(IQueryHandler<,>).MakeGenericType(queryType, typeof(TResult));
        
        var handler = _serviceProvider.GetRequiredService(handlerType);
        
        _logger.LogDebug("Dispatching query {QueryType} to handler {HandlerType}", 
            queryType.Name, handlerType.Name);
        
        var method = handlerType.GetMethod(nameof(IQueryHandler<IQuery<TResult>, TResult>.HandleAsync));
        var task = (Task<TResult>)method.Invoke(handler, new object[] { query });
        
        return await task;
    }
}
```

---

## 4. 🧱 Clean Architecture Implementation

### 4.1 Domain Layer

#### Domain Entities

```csharp
// Domain/Entities/Entity.cs
public abstract class Entity : IEntity
{
    public int Id { get; set; }
    public DateTime CreatedAt { get; set; }
    public DateTime? UpdatedAt { get; set; }
    public bool IsDeleted { get; set; }
    
    private readonly List<IDomainEvent> _domainEvents = new();
    
    public IReadOnlyCollection<IDomainEvent> DomainEvents => _domainEvents.AsReadOnly();
    
    protected void AddDomainEvent(IDomainEvent domainEvent)
    {
        _domainEvents.Add(domainEvent);
    }
    
    public void ClearDomainEvents()
    {
        _domainEvents.Clear();
    }
    
    public override bool Equals(object obj)
    {
        return obj is Entity entity && Id == entity.Id;
    }
    
    public override int GetHashCode()
    {
        return HashCode.Combine(Id);
    }
}

// Domain/Entities/User.cs
public class User : Entity
{
    public string Email { get; set; }
    public string PasswordHash { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public bool IsActive { get; set; }
    public DateTime? LastLoginAt { get; set; }
    
    // Navigation properties
    public virtual ICollection<Order> Orders { get; set; } = new List<Order>();
    
    // Domain methods
    public void ChangeEmail(string newEmail)
    {
        if (string.IsNullOrWhiteSpace(newEmail))
            throw new DomainException("Email cannot be empty");
            
        if (Email != newEmail)
        {
            var oldEmail = Email;
            Email = newEmail;
            UpdatedAt = DateTime.UtcNow;
            
            AddDomainEvent(new UserEmailChangedEvent(Id.ToString(), newEmail, oldEmail, 1));
        }
    }
    
    public void Activate()
    {
        if (!IsActive)
        {
            IsActive = true;
            UpdatedAt = DateTime.UtcNow;
            
            AddDomainEvent(new UserActivatedEvent(Id.ToString(), 1));
        }
    }
    
    public void Deactivate()
    {
        if (IsActive)
        {
            IsActive = false;
            UpdatedAt = DateTime.UtcNow;
            
            AddDomainEvent(new UserDeactivatedEvent(Id.ToString(), 1));
        }
    }
    
    public void RecordLogin()
    {
        LastLoginAt = DateTime.UtcNow;
        UpdatedAt = DateTime.UtcNow;
        
        AddDomainEvent(new UserLoggedInEvent(Id.ToString(), LastLoginAt.Value, 1));
    }
}

// Domain/Exceptions/DomainException.cs
public class DomainException : Exception
{
    public DomainException(string message) : base(message) { }
    public DomainException(string message, Exception innerException) : base(message, innerException) { }
}
```

### 4.2 Application Layer

#### Application Services

```csharp
// Application/Services/IUserService.cs
public interface IUserService
{
    Task<CreateUserResult> CreateUserAsync(CreateUserCommand command);
    Task<UserDto> GetUserByIdAsync(string userId);
    Task<UserDto> GetUserByEmailAsync(string email);
    Task<bool> UpdateUserEmailAsync(string userId, string newEmail);
    Task<bool> ActivateUserAsync(string userId);
    Task<bool> DeactivateUserAsync(string userId);
    Task<bool> RecordUserLoginAsync(string userId);
}

// Application/Services/UserService.cs
public class UserService : IUserService
{
    private readonly ICommandDispatcher _commandDispatcher;
    private readonly IQueryDispatcher _queryDispatcher;
    private readonly ILogger<UserService> _logger;
    
    public UserService(
        ICommandDispatcher commandDispatcher,
        IQueryDispatcher queryDispatcher,
        ILogger<UserService> logger)
    {
        _commandDispatcher = commandDispatcher;
        _queryDispatcher = queryDispatcher;
        _logger = logger;
    }
    
    public async Task<CreateUserResult> CreateUserAsync(CreateUserCommand command)
    {
        _logger.LogInformation("Creating user with email {Email}", command.Email);
        return await _commandDispatcher.DispatchAsync(command);
    }
    
    public async Task<UserDto> GetUserByIdAsync(string userId)
    {
        var query = new GetUserByIdQuery(userId);
        return await _queryDispatcher.DispatchAsync(query);
    }
    
    public async Task<UserDto> GetUserByEmailAsync(string email)
    {
        var query = new GetUserByEmailQuery(email);
        return await _queryDispatcher.DispatchAsync(query);
    }
    
    public async Task<bool> UpdateUserEmailAsync(string userId, string newEmail)
    {
        var command = new UpdateUserEmailCommand(userId, newEmail);
        var result = await _commandDispatcher.DispatchAsync(command);
        return result.Success;
    }
    
    public async Task<bool> ActivateUserAsync(string userId)
    {
        var command = new ActivateUserCommand(userId);
        var result = await _commandDispatcher.DispatchAsync(command);
        return result.Success;
    }
    
    public async Task<bool> DeactivateUserAsync(string userId)
    {
        var command = new DeactivateUserCommand(userId);
        var result = await _commandDispatcher.DispatchAsync(command);
        return result.Success;
    }
    
    public async Task<bool> RecordUserLoginAsync(string userId)
    {
        var command = new RecordUserLoginCommand(userId);
        var result = await _commandDispatcher.DispatchAsync(command);
        return result.Success;
    }
}
```

---

## 5. 🔌 Plugin Architecture

### 5.1 Plugin System Implementation

#### Plugin Interface

```csharp
// Plugins/IPlugin.cs
public interface IPlugin
{
    string Name { get; }
    string Version { get; }
    string Description { get; }
    string Author { get; }
    bool IsEnabled { get; set; }
    
    Task<bool> InitializeAsync();
    Task<bool> StartAsync();
    Task<bool> StopAsync();
    Task DisposeAsync();
}

// Plugins/PluginBase.cs
public abstract class PluginBase : IPlugin
{
    protected IServiceProvider ServiceProvider { get; private set; }
    protected ILogger Logger { get; private set; }
    
    public abstract string Name { get; }
    public abstract string Version { get; }
    public abstract string Description { get; }
    public abstract string Author { get; }
    public bool IsEnabled { get; set; } = true;
    
    public virtual async Task<bool> InitializeAsync()
    {
        Logger?.LogInformation("Initializing plugin {PluginName}", Name);
        
        try
        {
            await OnInitializeAsync();
            Logger?.LogInformation("Successfully initialized plugin {PluginName}", Name);
            return true;
        }
        catch (Exception ex)
        {
            Logger?.LogError(ex, "Failed to initialize plugin {PluginName}", Name);
            return false;
        }
    }
    
    public virtual async Task<bool> StartAsync()
    {
        if (!IsEnabled) return true;
        
        Logger?.LogInformation("Starting plugin {PluginName}", Name);
        
        try
        {
            await OnStartAsync();
            Logger?.LogInformation("Successfully started plugin {PluginName}", Name);
            return true;
        }
        catch (Exception ex)
        {
            Logger?.LogError(ex, "Failed to start plugin {PluginName}", Name);
            return false;
        }
    }
    
    public virtual async Task<bool> StopAsync()
    {
        Logger?.LogInformation("Stopping plugin {PluginName}", Name);
        
        try
        {
            await OnStopAsync();
            Logger?.LogInformation("Successfully stopped plugin {PluginName}", Name);
            return true;
        }
        catch (Exception ex)
        {
            Logger?.LogError(ex, "Failed to stop plugin {PluginName}", Name);
            return false;
        }
    }
    
    public virtual async Task DisposeAsync()
    {
        await OnDisposeAsync();
    }
    
    protected virtual Task OnInitializeAsync() => Task.CompletedTask;
    protected virtual Task OnStartAsync() => Task.CompletedTask;
    protected virtual Task OnStopAsync() => Task.CompletedTask;
    protected virtual Task OnDisposeAsync() => Task.CompletedTask;
    
    public void SetServiceProvider(IServiceProvider serviceProvider)
    {
        ServiceProvider = serviceProvider;
        Logger = serviceProvider.GetService<ILogger<PluginBase>>();
    }
}
```

#### Plugin Manager

```csharp
// Plugins/IPluginManager.cs
public interface IPluginManager
{
    Task LoadPluginsAsync();
    Task<bool> EnablePluginAsync(string pluginName);
    Task<bool> DisablePluginAsync(string pluginName);
    Task<T> GetPluginAsync<T>(string pluginName) where T : class, IPlugin;
    IEnumerable<IPlugin> GetAllPlugins();
    IEnumerable<T> GetPluginsOfType<T>() where T : class, IPlugin;
}

// Plugins/PluginManager.cs
public class PluginManager : IPluginManager
{
    private readonly Dictionary<string, IPlugin> _plugins = new();
    private readonly IServiceProvider _serviceProvider;
    private readonly ILogger<PluginManager> _logger;
    
    public PluginManager(IServiceProvider serviceProvider, ILogger<PluginManager> logger)
    {
        _serviceProvider = serviceProvider;
        _logger = logger;
    }
    
    public async Task LoadPluginsAsync()
    {
        _logger.LogInformation("Loading plugins...");
        
        try
        {
            // Get all plugin types from assemblies
            var pluginTypes = GetPluginTypes();
            
            foreach (var pluginType in pluginTypes)
            {
                try
                {
                    var plugin = (IPlugin)Activator.CreateInstance(pluginType);
                    
                    if (plugin is PluginBase pluginBase)
                    {
                        pluginBase.SetServiceProvider(_serviceProvider);
                    }
                    
                    await plugin.InitializeAsync();
                    
                    _plugins[plugin.Name] = plugin;
                    _logger.LogInformation("Loaded plugin {PluginName} v{Version}", plugin.Name, plugin.Version);
                }
                catch (Exception ex)
                {
                    _logger.LogError(ex, "Failed to load plugin {PluginType}", pluginType.Name);
                }
            }
            
            // Start enabled plugins
            foreach (var plugin in _plugins.Values.Where(p => p.IsEnabled))
            {
                await plugin.StartAsync();
            }
            
            _logger.LogInformation("Completed loading {PluginCount} plugins", _plugins.Count);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to load plugins");
        }
    }
    
    public async Task<bool> EnablePluginAsync(string pluginName)
    {
        if (_plugins.TryGetValue(pluginName, out var plugin))
        {
            if (!plugin.IsEnabled)
            {
                plugin.IsEnabled = true;
                return await plugin.StartAsync();
            }
            return true;
        }
        return false;
    }
    
    public async Task<bool> DisablePluginAsync(string pluginName)
    {
        if (_plugins.TryGetValue(pluginName, out var plugin))
        {
            if (plugin.IsEnabled)
            {
                plugin.IsEnabled = false;
                return await plugin.StopAsync();
            }
            return true;
        }
        return false;
    }
    
    public Task<T> GetPluginAsync<T>(string pluginName) where T : class, IPlugin
    {
        if (_plugins.TryGetValue(pluginName, out var plugin) && plugin is T typedPlugin)
        {
            return Task.FromResult(typedPlugin);
        }
        return Task.FromResult<T>(null);
    }
    
    public IEnumerable<IPlugin> GetAllPlugins()
    {
        return _plugins.Values;
    }
    
    public IEnumerable<T> GetPluginsOfType<T>() where T : class, IPlugin
    {
        return _plugins.Values.OfType<T>();
    }
    
    private IEnumerable<Type> GetPluginTypes()
    {
        var assemblies = AppDomain.CurrentDomain.GetAssemblies();
        
        foreach (var assembly in assemblies)
        {
            Type[] types;
            try
            {
                types = assembly.GetTypes();
            }
            catch (ReflectionTypeLoadException ex)
            {
                types = ex.Types.Where(t => t != null).ToArray();
            }
            
            foreach (var type in types)
            {
                if (typeof(IPlugin).IsAssignableFrom(type) && !type.IsInterface && !type.IsAbstract)
                {
                    yield return type;
                }
            }
        }
    }
}
```

### 5.2 Example Plugins

#### Analytics Plugin

```csharp
// Plugins/AnalyticsPlugin.cs
public class AnalyticsPlugin : PluginBase
{
    public override string Name => "Analytics Plugin";
    public override string Version => "1.0.0";
    public override string Description => "Provides advanced analytics and tracking capabilities";
    public override string Author => "Your Company";
    
    private IAnalyticsService _analyticsService;
    private Timer _metricsTimer;
    
    protected override async Task OnInitializeAsync()
    {
        _analyticsService = ServiceProvider.GetRequiredService<IAnalyticsService>();
        await Task.CompletedTask;
    }
    
    protected override async Task OnStartAsync()
    {
        // Start periodic metrics collection
        _metricsTimer = new Timer(CollectMetrics, null, TimeSpan.Zero, TimeSpan.FromMinutes(5));
        
        _analyticsService.TrackEvent("AnalyticsPluginStarted");
        await Task.CompletedTask;
    }
    
    protected override async Task OnStopAsync()
    {
        _metricsTimer?.Dispose();
        _analyticsService.TrackEvent("AnalyticsPluginStopped");
        await Task.CompletedTask;
    }
    
    private void CollectMetrics(object state)
    {
        try
        {
            var memoryUsage = GC.GetTotalMemory(false);
            var processorTime = Environment.TickCount;
            
            _analyticsService.TrackEvent("PerformanceMetrics", new Dictionary<string, string>
            {
                { "MemoryUsage", memoryUsage.ToString() },
                { "ProcessorTime", processorTime.ToString() },
                { "Timestamp", DateTime.UtcNow.ToString("O") }
            });
        }
        catch (Exception ex)
        {
            Logger?.LogError(ex, "Failed to collect metrics");
        }
    }
}

// Plugins/NotificationPlugin.cs
public class NotificationPlugin : PluginBase
{
    public override string Name => "Notification Plugin";
    public override string Version => "1.0.0";
    public override string Description => "Handles push notifications and local notifications";
    public override string Author => "Your Company";
    
    private INotificationService _notificationService;
    private IMessageBus _messageBus;
    
    protected override async Task OnInitializeAsync()
    {
        _notificationService = ServiceProvider.GetRequiredService<INotificationService>();
        _messageBus = ServiceProvider.GetRequiredService<IMessageBus>();
        
        // Subscribe to events that should trigger notifications
        _messageBus.Subscribe<UserRegisteredEvent>(HandleUserRegisteredAsync);
        _messageBus.Subscribe<OrderCompletedEvent>(HandleOrderCompletedAsync);
        
        await Task.CompletedTask;
    }
    
    protected override async Task OnStartAsync()
    {
        await _notificationService.InitializeAsync();
        Logger?.LogInformation("Notification plugin started and ready to handle notifications");
    }
    
    protected override async Task OnStopAsync()
    {
        _messageBus.Unsubscribe<UserRegisteredEvent>(HandleUserRegisteredAsync);
        _messageBus.Unsubscribe<OrderCompletedEvent>(HandleOrderCompletedAsync);
        
        await Task.CompletedTask;
    }
    
    private async Task HandleUserRegisteredAsync(UserRegisteredEvent message)
    {
        await _notificationService.SendLocalNotificationAsync(
            "Welcome!",
            $"Welcome to our app, {message.Name}!");
    }
    
    private async Task HandleOrderCompletedAsync(OrderCompletedEvent message)
    {
        await _notificationService.SendLocalNotificationAsync(
            "Order Complete",
            $"Your order #{message.OrderId} has been completed successfully!");
    }
}
```

---

## 6. 🧩 Practical Exercise

### 🎯 Complete Architecture Implementation

ในแบบฝึกหัดนี้ คุณจะสร้าง complete architectural solution สำหรับ e-commerce app:

1. **Setup Clean Architecture:**
   - สร้าง Domain layer ด้วย entities และ domain events
   - สร้าง Application layer ด้วย CQRS patterns
   - ใช้งาน Repository และ Unit of Work patterns

2. **Implement Messaging System:**
   - สร้าง message bus สำหรับ event-driven communication
   - สร้าง domain events และ event handlers
   - ใช้งาน async messaging patterns

3. **Create Advanced ViewModels:**
   - สร้าง base ViewModel ด้วย advanced features
   - ใช้งาน command pattern ใน ViewModels
   - เพิ่ม error handling และ analytics

4. **Build Plugin System:**
   - สร้าง plugin architecture
   - สร้าง example plugins (analytics, notifications)
   - ใช้งาน plugin manager

5. **Integration Testing:**
   - ทดสอบ end-to-end scenarios
   - ใช้งาน dependency injection
   - ตรวจสอบ performance และ memory usage

### Model Solution Structure:

```
YourApp/
├── Domain/
│   ├── Entities/
│   ├── Events/
│   └── Exceptions/
├── Application/
│   ├── Commands/
│   ├── Queries/
│   ├── Handlers/
│   └── Services/
├── Infrastructure/
│   ├── Data/
│   ├── Messaging/
│   └── Plugins/
├── Presentation/
│   ├── ViewModels/
│   ├── Views/
│   └── Services/
└── Tests/
    ├── Unit/
    ├── Integration/
    └── E2E/
```

---

## 📚 Summary

ในบทนี้เราได้เรียนรู้:

### 🔑 Key Concepts
- **Advanced MVVM**: Enhanced ViewModels ด้วย error handling, analytics และ command patterns
- **Repository & UoW**: Data access patterns ที่ maintainable และ testable
- **CQRS**: Command Query Responsibility Segregation สำหรับ scalable architecture
- **Event-Driven Architecture**: Message bus และ domain events สำหรับ loose coupling

### 🏗️ Architectural Patterns
- **Clean Architecture**: Separation of concerns และ dependency inversion
- **Domain-Driven Design**: Domain entities และ business logic encapsulation
- **Plugin Architecture**: Modular system ด้วย extensible plugins
- **Messaging Patterns**: Async communication และ event handling

### 🛠️ Implementation Techniques
- **Dependency Injection**: Advanced IoC container usage
- **Error Handling**: Comprehensive exception management
- **Performance**: Memory optimization และ resource management
- **Testing**: Architecture ที่ testable และ maintainable

### 📊 Benefits
- **Maintainability**: Code ที่ organized และ easy to modify
- **Scalability**: Architecture ที่รองรับการเติบโต
- **Testability**: Comprehensive testing capabilities
- **Extensibility**: Plugin system สำหรับ future enhancements

การใช้ advanced patterns และ architectural strategies เหล่านี้จะช่วยให้โปรเจ็กต์ของคุณมีคุณภาพสูง maintainable และพร้อมสำหรับการพัฒนาในระยะยาว

**Next Chapter Preview:** ในบทต่อไป เราจะเรียนรู้เรื่อง **Hybrid Approaches** รวมถึง Blazor Hybrid และ WebView integration! 🌐
