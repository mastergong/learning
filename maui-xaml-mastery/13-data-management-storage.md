# 💾 Chapter 13: Data Management & Storage

> **หัวข้อ**: Local Database, Cloud Sync & Offline-First Architecture  
> **ระยะเวลา**: 3.5 ชั่วโมง  
> **ความยากง่าย**: ระดับกลาง-สูง  
> **Prerequisites**: บทที่ 1-12

## 🎯 Learning Objectives

หลังจากเรียนบทนี้แล้ว คุณจะสามารถ:

- ✅ สร้าง local database ด้วย SQLite และ Entity Framework
- ✅ ใช้ cloud synchronization services
- ✅ ออกแบบ offline-first architecture
- ✅ จัดการ data caching และ performance optimization
- ✅ สร้าง data management layer ที่ scalable

---

## 📱 Chapter Overview

### 🏗️ What We'll Build
สร้าง **"Personal Finance Manager App"** ที่มี:
- 💰 Transaction management system
- 📊 Budget tracking และ reports
- ☁️ Cloud synchronization
- 📴 Offline-first capabilities
- 🔒 Data encryption และ security

---

## 1. 🗄️ SQLite Database Setup

### 1.1 Entity Models Design

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\13-DataManagement\Models\BaseEntity.cs
using SQLite;

namespace DataManagement.Models;

public abstract class BaseEntity
{
    [PrimaryKey, AutoIncrement]
    public int Id { get; set; }

    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public DateTime UpdatedAt { get; set; } = DateTime.UtcNow;
    public bool IsDeleted { get; set; } = false;
    public string SyncId { get; set; } = Guid.NewGuid().ToString();
    public bool IsLocal { get; set; } = true;
    public DateTime? LastSyncedAt { get; set; }
}
```

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\13-DataManagement\Models\Account.cs
using SQLite;

namespace DataManagement.Models;

[Table("Accounts")]
public class Account : BaseEntity
{
    [MaxLength(100)]
    public string Name { get; set; }

    [MaxLength(20)]
    public string AccountType { get; set; } // Checking, Savings, Credit, Investment

    public decimal Balance { get; set; }

    [MaxLength(10)]
    public string Currency { get; set; } = "USD";

    [MaxLength(500)]
    public string Description { get; set; }

    public bool IsActive { get; set; } = true;

    [MaxLength(50)]
    public string Color { get; set; } = "#2196F3";

    [MaxLength(50)]
    public string Icon { get; set; } = "account_balance";

    // Navigation properties (not stored in SQLite directly)
    [Ignore]
    public List<Transaction> Transactions { get; set; } = new();
}
```

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\13-DataManagement\Models\Category.cs
namespace DataManagement.Models;

[Table("Categories")]
public class Category : BaseEntity
{
    [MaxLength(100)]
    public string Name { get; set; }

    [MaxLength(20)]
    public string Type { get; set; } // Income, Expense

    [MaxLength(50)]
    public string Color { get; set; } = "#4CAF50";

    [MaxLength(50)]
    public string Icon { get; set; } = "category";

    public int? ParentCategoryId { get; set; }

    [MaxLength(500)]
    public string Description { get; set; }

    public bool IsActive { get; set; } = true;

    // Navigation properties
    [Ignore]
    public Category ParentCategory { get; set; }

    [Ignore]
    public List<Category> SubCategories { get; set; } = new();

    [Ignore]
    public List<Transaction> Transactions { get; set; } = new();
}
```

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\13-DataManagement\Models\Transaction.cs
namespace DataManagement.Models;

[Table("Transactions")]
public class Transaction : BaseEntity
{
    public decimal Amount { get; set; }

    [MaxLength(500)]
    public string Description { get; set; }

    public DateTime TransactionDate { get; set; } = DateTime.Now;

    [MaxLength(20)]
    public string Type { get; set; } // Income, Expense, Transfer

    public int AccountId { get; set; }

    public int? CategoryId { get; set; }

    public int? ToAccountId { get; set; } // For transfers

    [MaxLength(1000)]
    public string Notes { get; set; }

    [MaxLength(500)]
    public string Tags { get; set; } // JSON array of tags

    [MaxLength(200)]
    public string Location { get; set; }

    [MaxLength(500)]
    public string Receipt { get; set; } // File path or URL

    public bool IsRecurring { get; set; } = false;

    [MaxLength(50)]
    public string RecurrencePattern { get; set; } // Monthly, Weekly, etc.

    // Navigation properties
    [Ignore]
    public Account Account { get; set; }

    [Ignore]
    public Account ToAccount { get; set; }

    [Ignore]
    public Category Category { get; set; }

    // Helper properties
    [Ignore]
    public List<string> TagsList 
    { 
        get => string.IsNullOrEmpty(Tags) ? new() : JsonSerializer.Deserialize<List<string>>(Tags);
        set => Tags = JsonSerializer.Serialize(value);
    }
}
```

### 1.2 Database Context และ Repository Pattern

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\13-DataManagement\Data\FinanceDbContext.cs
using SQLite;

namespace DataManagement.Data;

public class FinanceDbContext
{
    private readonly SQLiteAsyncConnection _database;
    private readonly string _dbPath;

    public FinanceDbContext(string dbPath)
    {
        _dbPath = dbPath;
        _database = new SQLiteAsyncConnection(dbPath);
        
        InitializeDatabaseAsync().ConfigureAwait(false);
    }

    private async Task InitializeDatabaseAsync()
    {
        // Create tables
        await _database.CreateTableAsync<Account>();
        await _database.CreateTableAsync<Category>();
        await _database.CreateTableAsync<Transaction>();

        // Create indexes for better performance
        await CreateIndexesAsync();

        // Seed default data
        await SeedDefaultDataAsync();
    }

    private async Task CreateIndexesAsync()
    {
        try
        {
            // Transaction indexes
            await _database.ExecuteAsync("CREATE INDEX IF NOT EXISTS idx_transactions_account ON Transactions(AccountId)");
            await _database.ExecuteAsync("CREATE INDEX IF NOT EXISTS idx_transactions_category ON Transactions(CategoryId)");
            await _database.ExecuteAsync("CREATE INDEX IF NOT EXISTS idx_transactions_date ON Transactions(TransactionDate)");
            await _database.ExecuteAsync("CREATE INDEX IF NOT EXISTS idx_transactions_type ON Transactions(Type)");
            
            // Sync indexes
            await _database.ExecuteAsync("CREATE INDEX IF NOT EXISTS idx_accounts_sync ON Accounts(SyncId)");
            await _database.ExecuteAsync("CREATE INDEX IF NOT EXISTS idx_categories_sync ON Categories(SyncId)");
            await _database.ExecuteAsync("CREATE INDEX IF NOT EXISTS idx_transactions_sync ON Transactions(SyncId)");
            
            // Performance indexes
            await _database.ExecuteAsync("CREATE INDEX IF NOT EXISTS idx_accounts_active ON Accounts(IsActive)");
            await _database.ExecuteAsync("CREATE INDEX IF NOT EXISTS idx_categories_active ON Categories(IsActive)");
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"Error creating indexes: {ex.Message}");
        }
    }

    private async Task SeedDefaultDataAsync()
    {
        // Check if data already exists
        var accountCount = await _database.Table<Account>().CountAsync();
        if (accountCount > 0) return;

        // Seed default accounts
        var accounts = new[]
        {
            new Account { Name = "Checking Account", AccountType = "Checking", Balance = 0, Color = "#2196F3", Icon = "account_balance" },
            new Account { Name = "Savings Account", AccountType = "Savings", Balance = 0, Color = "#4CAF50", Icon = "savings" },
            new Account { Name = "Credit Card", AccountType = "Credit", Balance = 0, Color = "#FF9800", Icon = "credit_card" }
        };

        await _database.InsertAllAsync(accounts);

        // Seed default categories
        var categories = new[]
        {
            // Income categories
            new Category { Name = "Salary", Type = "Income", Color = "#4CAF50", Icon = "work" },
            new Category { Name = "Freelance", Type = "Income", Color = "#8BC34A", Icon = "laptop" },
            new Category { Name = "Investments", Type = "Income", Color = "#CDDC39", Icon = "trending_up" },
            
            // Expense categories
            new Category { Name = "Food & Dining", Type = "Expense", Color = "#FF5722", Icon = "restaurant" },
            new Category { Name = "Transportation", Type = "Expense", Color = "#607D8B", Icon = "directions_car" },
            new Category { Name = "Shopping", Type = "Expense", Color = "#E91E63", Icon = "shopping_cart" },
            new Category { Name = "Entertainment", Type = "Expense", Color = "#9C27B0", Icon = "movie" },
            new Category { Name = "Bills & Utilities", Type = "Expense", Color = "#3F51B5", Icon = "receipt" },
            new Category { Name = "Healthcare", Type = "Expense", Color = "#F44336", Icon = "local_hospital" },
            new Category { Name = "Education", Type = "Expense", Color = "#FF9800", Icon = "school" }
        };

        await _database.InsertAllAsync(categories);
    }

    // Generic repository methods
    public async Task<List<T>> GetAllAsync<T>() where T : BaseEntity, new()
    {
        return await _database.Table<T>().Where(x => !x.IsDeleted).ToListAsync();
    }

    public async Task<T> GetByIdAsync<T>(int id) where T : BaseEntity, new()
    {
        return await _database.Table<T>().Where(x => x.Id == id && !x.IsDeleted).FirstOrDefaultAsync();
    }

    public async Task<T> GetBySyncIdAsync<T>(string syncId) where T : BaseEntity, new()
    {
        return await _database.Table<T>().Where(x => x.SyncId == syncId && !x.IsDeleted).FirstOrDefaultAsync();
    }

    public async Task<int> SaveAsync<T>(T entity) where T : BaseEntity, new()
    {
        entity.UpdatedAt = DateTime.UtcNow;
        
        if (entity.Id != 0)
        {
            return await _database.UpdateAsync(entity);
        }
        else
        {
            entity.CreatedAt = DateTime.UtcNow;
            return await _database.InsertAsync(entity);
        }
    }

    public async Task<int> DeleteAsync<T>(T entity) where T : BaseEntity, new()
    {
        entity.IsDeleted = true;
        entity.UpdatedAt = DateTime.UtcNow;
        return await _database.UpdateAsync(entity);
    }

    public async Task<int> DeleteAsync<T>(int id) where T : BaseEntity, new()
    {
        var entity = await GetByIdAsync<T>(id);
        if (entity != null)
        {
            return await DeleteAsync(entity);
        }
        return 0;
    }

    // Batch operations for better performance
    public async Task<int> SaveAllAsync<T>(IEnumerable<T> entities) where T : BaseEntity, new()
    {
        var now = DateTime.UtcNow;
        var entitiesList = entities.ToList();
        
        foreach (var entity in entitiesList)
        {
            entity.UpdatedAt = now;
            if (entity.Id == 0)
            {
                entity.CreatedAt = now;
            }
        }

        return await _database.InsertAllAsync(entitiesList);
    }

    // Specific query methods
    public async Task<List<Transaction>> GetTransactionsByDateRangeAsync(DateTime startDate, DateTime endDate)
    {
        return await _database.Table<Transaction>()
            .Where(t => !t.IsDeleted && t.TransactionDate >= startDate && t.TransactionDate <= endDate)
            .OrderByDescending(t => t.TransactionDate)
            .ToListAsync();
    }

    public async Task<List<Transaction>> GetTransactionsByAccountAsync(int accountId)
    {
        return await _database.Table<Transaction>()
            .Where(t => !t.IsDeleted && t.AccountId == accountId)
            .OrderByDescending(t => t.TransactionDate)
            .ToListAsync();
    }

    public async Task<decimal> GetAccountBalanceAsync(int accountId)
    {
        var account = await GetByIdAsync<Account>(accountId);
        if (account == null) return 0;

        var transactions = await GetTransactionsByAccountAsync(accountId);
        
        decimal calculatedBalance = 0;
        foreach (var transaction in transactions)
        {
            switch (transaction.Type)
            {
                case "Income":
                    calculatedBalance += transaction.Amount;
                    break;
                case "Expense":
                    calculatedBalance -= transaction.Amount;
                    break;
                case "Transfer":
                    if (transaction.AccountId == accountId)
                        calculatedBalance -= transaction.Amount;
                    if (transaction.ToAccountId == accountId)
                        calculatedBalance += transaction.Amount;
                    break;
            }
        }

        return calculatedBalance;
    }

    // Database maintenance
    public async Task<int> GetDatabaseSizeAsync()
    {
        var fileInfo = new FileInfo(_dbPath);
        return (int)fileInfo.Length;
    }

    public async Task VacuumDatabaseAsync()
    {
        await _database.ExecuteAsync("VACUUM");
    }

    public async Task CloseAsync()
    {
        await _database.CloseAsync();
    }
}
```

---

## 2. 📦 Repository Pattern Implementation

### 2.1 Generic Repository Interface

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\13-DataManagement\Repositories\IRepository.cs
namespace DataManagement.Repositories;

public interface IRepository<T> where T : BaseEntity
{
    Task<List<T>> GetAllAsync();
    Task<T> GetByIdAsync(int id);
    Task<T> GetBySyncIdAsync(string syncId);
    Task<int> SaveAsync(T entity);
    Task<int> DeleteAsync(T entity);
    Task<int> DeleteAsync(int id);
    Task<List<T>> SaveAllAsync(IEnumerable<T> entities);
    Task<List<T>> GetUnsyncedAsync();
    Task<List<T>> GetModifiedSinceAsync(DateTime lastSync);
}
```

### 2.2 Transaction Repository

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\13-DataManagement\Repositories\TransactionRepository.cs
namespace DataManagement.Repositories;

public interface ITransactionRepository : IRepository<Transaction>
{
    Task<List<Transaction>> GetByDateRangeAsync(DateTime startDate, DateTime endDate);
    Task<List<Transaction>> GetByAccountAsync(int accountId);
    Task<List<Transaction>> GetByCategoryAsync(int categoryId);
    Task<List<TransactionSummary>> GetMonthlyExpenseSummaryAsync(int year);
    Task<List<TransactionSummary>> GetCategorySummaryAsync(DateTime startDate, DateTime endDate);
    Task<decimal> GetTotalIncomeAsync(DateTime startDate, DateTime endDate);
    Task<decimal> GetTotalExpenseAsync(DateTime startDate, DateTime endDate);
    Task<List<Transaction>> SearchAsync(string searchTerm);
}

public class TransactionRepository : ITransactionRepository
{
    private readonly FinanceDbContext _context;

    public TransactionRepository(FinanceDbContext context)
    {
        _context = context;
    }

    public async Task<List<Transaction>> GetAllAsync()
    {
        return await _context.GetAllAsync<Transaction>();
    }

    public async Task<Transaction> GetByIdAsync(int id)
    {
        var transaction = await _context.GetByIdAsync<Transaction>(id);
        if (transaction != null)
        {
            await LoadNavigationPropertiesAsync(transaction);
        }
        return transaction;
    }

    public async Task<Transaction> GetBySyncIdAsync(string syncId)
    {
        return await _context.GetBySyncIdAsync<Transaction>(syncId);
    }

    public async Task<int> SaveAsync(Transaction entity)
    {
        return await _context.SaveAsync(entity);
    }

    public async Task<int> DeleteAsync(Transaction entity)
    {
        return await _context.DeleteAsync(entity);
    }

    public async Task<int> DeleteAsync(int id)
    {
        return await _context.DeleteAsync<Transaction>(id);
    }

    public async Task<List<Transaction>> SaveAllAsync(IEnumerable<Transaction> entities)
    {
        await _context.SaveAllAsync(entities);
        return entities.ToList();
    }

    public async Task<List<Transaction>> GetUnsyncedAsync()
    {
        var db = _context._database;
        return await db.Table<Transaction>()
            .Where(t => !t.IsDeleted && t.IsLocal && t.LastSyncedAt == null)
            .ToListAsync();
    }

    public async Task<List<Transaction>> GetModifiedSinceAsync(DateTime lastSync)
    {
        var db = _context._database;
        return await db.Table<Transaction>()
            .Where(t => !t.IsDeleted && t.UpdatedAt > lastSync)
            .ToListAsync();
    }

    public async Task<List<Transaction>> GetByDateRangeAsync(DateTime startDate, DateTime endDate)
    {
        var transactions = await _context.GetTransactionsByDateRangeAsync(startDate, endDate);
        
        // Load navigation properties
        foreach (var transaction in transactions)
        {
            await LoadNavigationPropertiesAsync(transaction);
        }
        
        return transactions;
    }

    public async Task<List<Transaction>> GetByAccountAsync(int accountId)
    {
        return await _context.GetTransactionsByAccountAsync(accountId);
    }

    public async Task<List<Transaction>> GetByCategoryAsync(int categoryId)
    {
        var db = _context._database;
        return await db.Table<Transaction>()
            .Where(t => !t.IsDeleted && t.CategoryId == categoryId)
            .OrderByDescending(t => t.TransactionDate)
            .ToListAsync();
    }

    public async Task<List<TransactionSummary>> GetMonthlyExpenseSummaryAsync(int year)
    {
        var startDate = new DateTime(year, 1, 1);
        var endDate = new DateTime(year, 12, 31, 23, 59, 59);
        
        var transactions = await GetByDateRangeAsync(startDate, endDate);
        
        var summary = transactions
            .Where(t => t.Type == "Expense")
            .GroupBy(t => new { t.TransactionDate.Year, t.TransactionDate.Month })
            .Select(g => new TransactionSummary
            {
                Period = $"{g.Key.Year}-{g.Key.Month:D2}",
                TotalAmount = g.Sum(t => t.Amount),
                TransactionCount = g.Count()
            })
            .OrderBy(s => s.Period)
            .ToList();

        return summary;
    }

    public async Task<List<TransactionSummary>> GetCategorySummaryAsync(DateTime startDate, DateTime endDate)
    {
        var transactions = await GetByDateRangeAsync(startDate, endDate);
        
        var summary = transactions
            .Where(t => t.Category != null)
            .GroupBy(t => t.Category.Name)
            .Select(g => new TransactionSummary
            {
                Category = g.Key,
                TotalAmount = g.Sum(t => t.Amount),
                TransactionCount = g.Count(),
                Period = $"{startDate:yyyy-MM-dd} to {endDate:yyyy-MM-dd}"
            })
            .OrderByDescending(s => s.TotalAmount)
            .ToList();

        return summary;
    }

    public async Task<decimal> GetTotalIncomeAsync(DateTime startDate, DateTime endDate)
    {
        var transactions = await GetByDateRangeAsync(startDate, endDate);
        return transactions.Where(t => t.Type == "Income").Sum(t => t.Amount);
    }

    public async Task<decimal> GetTotalExpenseAsync(DateTime startDate, DateTime endDate)
    {
        var transactions = await GetByDateRangeAsync(startDate, endDate);
        return transactions.Where(t => t.Type == "Expense").Sum(t => t.Amount);
    }

    public async Task<List<Transaction>> SearchAsync(string searchTerm)
    {
        if (string.IsNullOrWhiteSpace(searchTerm))
            return new List<Transaction>();

        var db = _context._database;
        var term = $"%{searchTerm.ToLower()}%";
        
        return await db.QueryAsync<Transaction>(@"
            SELECT * FROM Transactions 
            WHERE IsDeleted = 0 
            AND (LOWER(Description) LIKE ? 
                 OR LOWER(Notes) LIKE ? 
                 OR LOWER(Tags) LIKE ?)
            ORDER BY TransactionDate DESC",
            term, term, term);
    }

    private async Task LoadNavigationPropertiesAsync(Transaction transaction)
    {
        if (transaction.AccountId > 0)
        {
            transaction.Account = await _context.GetByIdAsync<Account>(transaction.AccountId);
        }

        if (transaction.ToAccountId.HasValue && transaction.ToAccountId > 0)
        {
            transaction.ToAccount = await _context.GetByIdAsync<Account>(transaction.ToAccountId.Value);
        }

        if (transaction.CategoryId.HasValue && transaction.CategoryId > 0)
        {
            transaction.Category = await _context.GetByIdAsync<Category>(transaction.CategoryId.Value);
        }
    }
}

public class TransactionSummary
{
    public string Period { get; set; }
    public string Category { get; set; }
    public decimal TotalAmount { get; set; }
    public int TransactionCount { get; set; }
}
```

---

## 3. ☁️ Cloud Synchronization Service

### 3.1 Sync Service Interface

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\13-DataManagement\Services\ISyncService.cs
namespace DataManagement.Services;

public interface ISyncService
{
    Task<SyncResult> SyncAllAsync();
    Task<SyncResult> SyncEntityAsync<T>() where T : BaseEntity;
    Task<SyncResult> UploadChangesAsync();
    Task<SyncResult> DownloadChangesAsync();
    Task<bool> IsOnlineAsync();
    Task<DateTime?> GetLastSyncTimeAsync();
    Task SetLastSyncTimeAsync(DateTime syncTime);
    
    event EventHandler<SyncProgressEventArgs> SyncProgress;
    event EventHandler<SyncCompletedEventArgs> SyncCompleted;
}

public class SyncResult
{
    public bool IsSuccess { get; set; }
    public string ErrorMessage { get; set; }
    public int RecordsUploaded { get; set; }
    public int RecordsDownloaded { get; set; }
    public int ConflictsResolved { get; set; }
    public TimeSpan Duration { get; set; }
    public DateTime SyncTime { get; set; }
}

public class SyncProgressEventArgs : EventArgs
{
    public string Operation { get; set; }
    public int CurrentItem { get; set; }
    public int TotalItems { get; set; }
    public double ProgressPercentage => TotalItems > 0 ? (double)CurrentItem / TotalItems * 100 : 0;
}

public class SyncCompletedEventArgs : EventArgs
{
    public SyncResult Result { get; set; }
}
```

### 3.2 Cloud Sync Implementation

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\13-DataManagement\Services\CloudSyncService.cs
using System.Net.Http.Json;

namespace DataManagement.Services;

public class CloudSyncService : ISyncService
{
    private readonly HttpClient _httpClient;
    private readonly FinanceDbContext _context;
    private readonly IConnectivity _connectivity;
    private readonly IPreferences _preferences;
    
    public event EventHandler<SyncProgressEventArgs> SyncProgress;
    public event EventHandler<SyncCompletedEventArgs> SyncCompleted;

    private const string LAST_SYNC_KEY = "LastSyncTime";
    private const string API_BASE_URL = "https://api.financeapp.com/v1";

    public CloudSyncService(
        HttpClient httpClient,
        FinanceDbContext context,
        IConnectivity connectivity,
        IPreferences preferences)
    {
        _httpClient = httpClient;
        _context = context;
        _connectivity = connectivity;
        _preferences = preferences;
        
        ConfigureHttpClient();
    }

    private void ConfigureHttpClient()
    {
        _httpClient.BaseAddress = new Uri(API_BASE_URL);
        _httpClient.DefaultRequestHeaders.Add("User-Agent", "FinanceApp/1.0");
        
        // Add authentication headers here
        // _httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
    }

    public async Task<bool> IsOnlineAsync()
    {
        return _connectivity.NetworkAccess == NetworkAccess.Internet;
    }

    public async Task<DateTime?> GetLastSyncTimeAsync()
    {
        var lastSyncString = _preferences.Get(LAST_SYNC_KEY, string.Empty);
        if (DateTime.TryParse(lastSyncString, out var lastSync))
        {
            return lastSync;
        }
        return null;
    }

    public async Task SetLastSyncTimeAsync(DateTime syncTime)
    {
        _preferences.Set(LAST_SYNC_KEY, syncTime.ToString("O"));
    }

    public async Task<SyncResult> SyncAllAsync()
    {
        var stopwatch = System.Diagnostics.Stopwatch.StartNew();
        var result = new SyncResult
        {
            SyncTime = DateTime.UtcNow
        };

        try
        {
            if (!await IsOnlineAsync())
            {
                result.ErrorMessage = "No internet connection available";
                return result;
            }

            OnSyncProgress("Starting synchronization...", 0, 100);

            // 1. Upload local changes
            OnSyncProgress("Uploading local changes...", 10, 100);
            var uploadResult = await UploadChangesAsync();
            result.RecordsUploaded = uploadResult.RecordsUploaded;

            // 2. Download remote changes
            OnSyncProgress("Downloading remote changes...", 50, 100);
            var downloadResult = await DownloadChangesAsync();
            result.RecordsDownloaded = downloadResult.RecordsDownloaded;

            // 3. Resolve conflicts
            OnSyncProgress("Resolving conflicts...", 80, 100);
            result.ConflictsResolved = await ResolveConflictsAsync();

            // 4. Update last sync time
            await SetLastSyncTimeAsync(result.SyncTime);

            OnSyncProgress("Synchronization completed", 100, 100);
            result.IsSuccess = true;
        }
        catch (Exception ex)
        {
            result.IsSuccess = false;
            result.ErrorMessage = ex.Message;
            System.Diagnostics.Debug.WriteLine($"Sync error: {ex}");
        }
        finally
        {
            stopwatch.Stop();
            result.Duration = stopwatch.Elapsed;
            
            SyncCompleted?.Invoke(this, new SyncCompletedEventArgs { Result = result });
        }

        return result;
    }

    public async Task<SyncResult> SyncEntityAsync<T>() where T : BaseEntity
    {
        var result = new SyncResult { SyncTime = DateTime.UtcNow };
        
        try
        {
            if (!await IsOnlineAsync())
            {
                result.ErrorMessage = "No internet connection";
                return result;
            }

            var entityName = typeof(T).Name;
            OnSyncProgress($"Syncing {entityName}...", 0, 100);

            // Upload unsaved local entities
            var localEntities = await GetUnsyncedEntitiesAsync<T>();
            if (localEntities.Any())
            {
                await UploadEntitiesAsync(localEntities);
                result.RecordsUploaded = localEntities.Count;
            }

            // Download remote changes
            var lastSync = await GetLastSyncTimeAsync();
            var remoteEntities = await DownloadEntitiesAsync<T>(lastSync);
            if (remoteEntities.Any())
            {
                await SaveRemoteEntitiesAsync(remoteEntities);
                result.RecordsDownloaded = remoteEntities.Count;
            }

            OnSyncProgress($"{entityName} sync completed", 100, 100);
            result.IsSuccess = true;
        }
        catch (Exception ex)
        {
            result.ErrorMessage = ex.Message;
            System.Diagnostics.Debug.WriteLine($"Entity sync error: {ex}");
        }

        return result;
    }

    public async Task<SyncResult> UploadChangesAsync()
    {
        var result = new SyncResult();
        var totalUploaded = 0;

        try
        {
            // Upload Accounts
            var accounts = await GetUnsyncedEntitiesAsync<Account>();
            if (accounts.Any())
            {
                await UploadEntitiesAsync(accounts);
                totalUploaded += accounts.Count;
            }

            // Upload Categories
            var categories = await GetUnsyncedEntitiesAsync<Category>();
            if (categories.Any())
            {
                await UploadEntitiesAsync(categories);
                totalUploaded += categories.Count;
            }

            // Upload Transactions
            var transactions = await GetUnsyncedEntitiesAsync<Transaction>();
            if (transactions.Any())
            {
                await UploadEntitiesAsync(transactions);
                totalUploaded += transactions.Count;
            }

            result.RecordsUploaded = totalUploaded;
            result.IsSuccess = true;
        }
        catch (Exception ex)
        {
            result.ErrorMessage = ex.Message;
            System.Diagnostics.Debug.WriteLine($"Upload error: {ex}");
        }

        return result;
    }

    public async Task<SyncResult> DownloadChangesAsync()
    {
        var result = new SyncResult();
        var totalDownloaded = 0;
        var lastSync = await GetLastSyncTimeAsync();

        try
        {
            // Download Accounts
            var accounts = await DownloadEntitiesAsync<Account>(lastSync);
            if (accounts.Any())
            {
                await SaveRemoteEntitiesAsync(accounts);
                totalDownloaded += accounts.Count;
            }

            // Download Categories
            var categories = await DownloadEntitiesAsync<Category>(lastSync);
            if (categories.Any())
            {
                await SaveRemoteEntitiesAsync(categories);
                totalDownloaded += categories.Count;
            }

            // Download Transactions
            var transactions = await DownloadEntitiesAsync<Transaction>(lastSync);
            if (transactions.Any())
            {
                await SaveRemoteEntitiesAsync(transactions);
                totalDownloaded += transactions.Count;
            }

            result.RecordsDownloaded = totalDownloaded;
            result.IsSuccess = true;
        }
        catch (Exception ex)
        {
            result.ErrorMessage = ex.Message;
            System.Diagnostics.Debug.WriteLine($"Download error: {ex}");
        }

        return result;
    }

    private async Task<List<T>> GetUnsyncedEntitiesAsync<T>() where T : BaseEntity
    {
        // This would use your repository to get unsynced entities
        // For now, using direct database access
        var tableName = typeof(T).Name + "s";
        
        // Simplified - in real implementation, use proper repository pattern
        return new List<T>();
    }

    private async Task UploadEntitiesAsync<T>(List<T> entities) where T : BaseEntity
    {
        foreach (var entity in entities)
        {
            try
            {
                var response = await _httpClient.PostAsJsonAsync($"/{typeof(T).Name.ToLower()}s", entity);
                
                if (response.IsSuccessStatusCode)
                {
                    // Mark as synced
                    entity.IsLocal = false;
                    entity.LastSyncedAt = DateTime.UtcNow;
                    await _context.SaveAsync(entity);
                }
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine($"Upload entity error: {ex.Message}");
                // Continue with next entity
            }
        }
    }

    private async Task<List<T>> DownloadEntitiesAsync<T>(DateTime? lastSync) where T : BaseEntity
    {
        try
        {
            var url = $"/{typeof(T).Name.ToLower()}s";
            if (lastSync.HasValue)
            {
                url += $"?since={lastSync.Value:O}";
            }

            var response = await _httpClient.GetAsync(url);
            response.EnsureSuccessStatusCode();

            var entities = await response.Content.ReadFromJsonAsync<List<T>>();
            return entities ?? new List<T>();
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"Download entities error: {ex.Message}");
            return new List<T>();
        }
    }

    private async Task SaveRemoteEntitiesAsync<T>(List<T> entities) where T : BaseEntity
    {
        foreach (var entity in entities)
        {
            try
            {
                // Check if entity exists locally
                var existingEntity = await _context.GetBySyncIdAsync<T>(entity.SyncId);
                
                if (existingEntity != null)
                {
                    // Update existing entity
                    entity.Id = existingEntity.Id;
                    await _context.SaveAsync(entity);
                }
                else
                {
                    // Insert new entity
                    entity.Id = 0; // Reset ID for local insert
                    entity.IsLocal = false;
                    await _context.SaveAsync(entity);
                }
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine($"Save remote entity error: {ex.Message}");
            }
        }
    }

    private async Task<int> ResolveConflictsAsync()
    {
        // Simplified conflict resolution
        // In a real app, you'd implement proper conflict resolution strategies
        var conflictsResolved = 0;
        
        // Strategy: Last-write-wins
        // More sophisticated strategies could include:
        // - Manual conflict resolution UI
        // - Field-level merging
        // - Business rule-based resolution
        
        return conflictsResolved;
    }

    private void OnSyncProgress(string operation, int current, int total)
    {
        SyncProgress?.Invoke(this, new SyncProgressEventArgs
        {
            Operation = operation,
            CurrentItem = current,
            TotalItems = total
        });
    }
}
```

---

## 4. 📴 Offline-First Architecture

### 4.1 Offline Service Implementation

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\13-DataManagement\Services\OfflineService.cs
namespace DataManagement.Services;

public interface IOfflineService
{
    bool IsOfflineMode { get; }
    Task<bool> EnableOfflineModeAsync();
    Task<bool> DisableOfflineModeAsync();
    Task<List<T>> GetCachedDataAsync<T>() where T : BaseEntity;
    Task CacheDataAsync<T>(List<T> data) where T : BaseEntity;
    Task<OfflineQueueItem> QueueOperationAsync(string operation, object data);
    Task<List<OfflineQueueItem>> GetPendingOperationsAsync();
    Task ProcessQueueAsync();
    Task ClearCacheAsync();
}

public class OfflineService : IOfflineService
{
    private readonly FinanceDbContext _context;
    private readonly IConnectivity _connectivity;
    private readonly ISyncService _syncService;
    private readonly IPreferences _preferences;

    private const string OFFLINE_MODE_KEY = "OfflineMode";
    private bool _isOfflineMode;

    public bool IsOfflineMode => _isOfflineMode;

    public OfflineService(
        FinanceDbContext context,
        IConnectivity connectivity,
        ISyncService syncService,
        IPreferences preferences)
    {
        _context = context;
        _connectivity = connectivity;
        _syncService = syncService;
        _preferences = preferences;

        _isOfflineMode = _preferences.Get(OFFLINE_MODE_KEY, false);
        
        // Monitor connectivity changes
        _connectivity.ConnectivityChanged += OnConnectivityChanged;
    }

    private async void OnConnectivityChanged(object sender, ConnectivityChangedEventArgs e)
    {
        if (e.NetworkAccess == NetworkAccess.Internet && _isOfflineMode)
        {
            // Connection restored - try to process pending operations
            await ProcessQueueAsync();
        }
    }

    public async Task<bool> EnableOfflineModeAsync()
    {
        try
        {
            // Cache essential data before going offline
            await CacheEssentialDataAsync();
            
            _isOfflineMode = true;
            _preferences.Set(OFFLINE_MODE_KEY, true);
            
            return true;
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"Enable offline mode error: {ex.Message}");
            return false;
        }
    }

    public async Task<bool> DisableOfflineModeAsync()
    {
        try
        {
            if (_connectivity.NetworkAccess == NetworkAccess.Internet)
            {
                // Process pending operations before going online
                await ProcessQueueAsync();
                
                // Sync with server
                await _syncService.SyncAllAsync();
            }
            
            _isOfflineMode = false;
            _preferences.Set(OFFLINE_MODE_KEY, false);
            
            return true;
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"Disable offline mode error: {ex.Message}");
            return false;
        }
    }

    public async Task<List<T>> GetCachedDataAsync<T>() where T : BaseEntity
    {
        return await _context.GetAllAsync<T>();
    }

    public async Task CacheDataAsync<T>(List<T> data) where T : BaseEntity
    {
        await _context.SaveAllAsync(data);
    }

    public async Task<OfflineQueueItem> QueueOperationAsync(string operation, object data)
    {
        var queueItem = new OfflineQueueItem
        {
            Operation = operation,
            Data = JsonSerializer.Serialize(data),
            CreatedAt = DateTime.UtcNow,
            Status = OfflineQueueStatus.Pending
        };

        await _context.SaveAsync(queueItem);
        return queueItem;
    }

    public async Task<List<OfflineQueueItem>> GetPendingOperationsAsync()
    {
        return await _context.GetAllAsync<OfflineQueueItem>()
            .ContinueWith(task => task.Result.Where(q => q.Status == OfflineQueueStatus.Pending).ToList());
    }

    public async Task ProcessQueueAsync()
    {
        if (_connectivity.NetworkAccess != NetworkAccess.Internet)
            return;

        var pendingOperations = await GetPendingOperationsAsync();
        
        foreach (var operation in pendingOperations)
        {
            try
            {
                await ProcessQueueItemAsync(operation);
                operation.Status = OfflineQueueStatus.Processed;
                operation.ProcessedAt = DateTime.UtcNow;
                await _context.SaveAsync(operation);
            }
            catch (Exception ex)
            {
                operation.Status = OfflineQueueStatus.Failed;
                operation.ErrorMessage = ex.Message;
                operation.RetryCount++;
                await _context.SaveAsync(operation);
                
                System.Diagnostics.Debug.WriteLine($"Queue processing error: {ex.Message}");
            }
        }
    }

    public async Task ClearCacheAsync()
    {
        // Clear cached data but keep schema
        var tables = new[] { "Accounts", "Categories", "Transactions", "OfflineQueue" };
        
        foreach (var table in tables)
        {
            await _context._database.ExecuteAsync($"DELETE FROM {table}");
        }
    }

    private async Task CacheEssentialDataAsync()
    {
        try
        {
            // Cache last 30 days of transactions
            var startDate = DateTime.Now.AddDays(-30);
            var endDate = DateTime.Now;
            
            var transactions = await _context.GetTransactionsByDateRangeAsync(startDate, endDate);
            var accounts = await _context.GetAllAsync<Account>();
            var categories = await _context.GetAllAsync<Category>();

            // Data is already in local database
            System.Diagnostics.Debug.WriteLine($"Cached {accounts.Count} accounts, {categories.Count} categories, {transactions.Count} transactions");
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"Cache essential data error: {ex.Message}");
        }
    }

    private async Task ProcessQueueItemAsync(OfflineQueueItem queueItem)
    {
        // Process different types of operations
        switch (queueItem.Operation)
        {
            case "CreateTransaction":
                var transaction = JsonSerializer.Deserialize<Transaction>(queueItem.Data);
                await _syncService.SyncEntityAsync<Transaction>();
                break;
                
            case "UpdateTransaction":
                var updatedTransaction = JsonSerializer.Deserialize<Transaction>(queueItem.Data);
                await _syncService.SyncEntityAsync<Transaction>();
                break;
                
            case "DeleteTransaction":
                var deletedTransaction = JsonSerializer.Deserialize<Transaction>(queueItem.Data);
                await _syncService.SyncEntityAsync<Transaction>();
                break;
                
            // Add other operations as needed
            default:
                throw new NotSupportedException($"Operation {queueItem.Operation} is not supported");
        }
    }
}

// Offline Queue Models
[Table("OfflineQueue")]
public class OfflineQueueItem : BaseEntity
{
    [MaxLength(50)]
    public string Operation { get; set; }

    public string Data { get; set; }

    public OfflineQueueStatus Status { get; set; } = OfflineQueueStatus.Pending;

    public DateTime? ProcessedAt { get; set; }

    public int RetryCount { get; set; } = 0;

    [MaxLength(500)]
    public string ErrorMessage { get; set; }
}

public enum OfflineQueueStatus
{
    Pending,
    Processing,
    Processed,
    Failed
}
```

---

## 5. 🧪 Practical Exercises

### Exercise 1: Advanced Querying
สร้าง advanced query methods:
- Complex filtering และ sorting
- Aggregation และ grouping
- Full-text search capabilities
- Performance optimization

### Exercise 2: Conflict Resolution
สร้างระบบ conflict resolution:
- Detect data conflicts
- Manual resolution UI
- Automatic resolution strategies
- Merge algorithms

### Exercise 3: Data Export/Import
สร้างระบบ data portability:
- Export to CSV/JSON
- Import from external sources
- Data validation และ sanitization
- Backup และ restore

---

## 6. 📝 Best Practices & Performance

### 🎯 Database Performance Tips

```csharp
// ✅ DO: Use indexes for frequently queried columns
await _database.ExecuteAsync("CREATE INDEX IF NOT EXISTS idx_transactions_date ON Transactions(TransactionDate)");

// ✅ DO: Use batch operations for bulk inserts
await _context.SaveAllAsync(largeListOfEntities);

// ✅ DO: Use async methods consistently
var transactions = await _repository.GetByDateRangeAsync(startDate, endDate);

// ✅ DO: Implement proper error handling
try
{
    await _context.SaveAsync(entity);
}
catch (SQLiteException ex)
{
    // Handle database-specific errors
    _logger.LogError(ex, "Database operation failed");
    throw new DataException("Failed to save data", ex);
}

// ❌ DON'T: Load large datasets without pagination
// Wrong:
var allTransactions = await _repository.GetAllAsync(); // Could be millions of records

// Right:
var recentTransactions = await _repository.GetByDateRangeAsync(DateTime.Now.AddDays(-30), DateTime.Now);

// ✅ DO: Use connection pooling for better performance
public class DatabaseService
{
    private static readonly SemaphoreSlim _semaphore = new(1, 1);
    
    public async Task<T> ExecuteWithLockAsync<T>(Func<Task<T>> operation)
    {
        await _semaphore.WaitAsync();
        try
        {
            return await operation();
        }
        finally
        {
            _semaphore.Release();
        }
    }
}
```

### 🔧 Sync Performance Optimization

```csharp
// ✅ DO: Implement delta sync
public async Task<List<T>> GetChangesSinceAsync<T>(DateTime lastSync) where T : BaseEntity
{
    return await _context._database.Table<T>()
        .Where(e => e.UpdatedAt > lastSync)
        .ToListAsync();
}

// ✅ DO: Use compression for large sync payloads
public async Task<byte[]> CompressSyncDataAsync<T>(List<T> data)
{
    var json = JsonSerializer.Serialize(data);
    var bytes = Encoding.UTF8.GetBytes(json);
    
    using var outputStream = new MemoryStream();
    using var gzipStream = new GZipStream(outputStream, CompressionMode.Compress);
    await gzipStream.WriteAsync(bytes);
    
    return outputStream.ToArray();
}

// ✅ DO: Implement progressive sync
public async Task<SyncResult> ProgressiveSyncAsync()
{
    var batchSize = 100;
    var result = new SyncResult();
    
    // Sync in batches to avoid memory issues
    var offset = 0;
    List<Transaction> batch;
    
    do
    {
        batch = await GetUnsyncedBatchAsync(offset, batchSize);
        if (batch.Any())
        {
            await SyncBatchAsync(batch);
            result.RecordsUploaded += batch.Count;
        }
        offset += batchSize;
    } while (batch.Count == batchSize);
    
    return result;
}
```

---

## 7. 🎯 Quiz & Assessment

### Quiz Questions

1. **Repository Pattern**: ข้อดีของการใช้ Repository Pattern ใน data layer คืออะไร?

2. **Offline-First**: การออกแบบ offline-first architecture มีหลักการอะไรบ้าง?

3. **Sync Conflicts**: วิธีการจัดการ data conflicts ในระบบ sync มีแบบไหนบ้าง?

4. **Performance**: เทคนิคไหนช่วยเพิ่ม database performance ใน mobile apps?

### Practical Assessment
สร้าง **"Personal Finance Manager"** ที่มี:
- ✅ Complete data management layer
- ✅ Offline-first capabilities
- ✅ Cloud synchronization
- ✅ Performance optimization
- ✅ Data export/import features

---

## 8. 🚀 Next Steps

### Advanced Topics to Explore
- **Data Encryption**: End-to-end encryption สำหรับ sensitive data
- **Real-time Sync**: WebSocket-based real-time synchronization
- **Data Analytics**: Built-in analytics และ reporting engine
- **Multi-tenant Architecture**: Support multiple users/organizations

### Chapter 14 Preview
หัวข้อต่อไป: **"Performance Optimization"**
- Memory management techniques
- UI performance optimization
- Network optimization strategies
- Battery usage optimization

---

## 📚 Additional Resources

### 🔗 Useful Links
- [SQLite Documentation](https://www.sqlite.org/docs.html)
- [Entity Framework Core](https://docs.microsoft.com/ef/core/)
- [MAUI Data Access](https://docs.microsoft.com/dotnet/maui/data-cloud/)

### 📖 Recommended Reading
- "Database Design for Mere Mortals"
- "NoSQL Distilled"
- "Building Offline-First Apps"

---

*หมายเหตุ: บทนี้เน้นการสร้าง robust data management system ที่รองรับ offline-first architecture และ cloud synchronization อย่างมีประสิทธิภาพ*
