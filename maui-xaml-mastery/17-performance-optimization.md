# 🚀 Chapter 17: Performance Optimization

## การเพิ่มประสิทธิภาพ MAUI Applications

### 📋 Chapter Overview

ในบทนี้เราจะเรียนรู้การ optimize performance ของ .NET MAUI applications ตั้งแต่ memory management, rendering optimization, ไปจนถึงการใช้ profiling tools เพื่อวิเคราะห์และแก้ไขปัญหา performance

### 🎯 Learning Objectives

- เข้าใจ performance bottlenecks ใน MAUI applications
- ใช้ memory profiling tools และ performance analyzers
- Optimize UI rendering และ layout calculations
- จัดการ memory leaks และ garbage collection
- ใช้ async/await patterns อย่างมีประสิทธิภาพ
- Implement caching strategies และ lazy loading

---

## 1. 🔍 Performance Analysis Tools

### 1.1 Visual Studio Diagnostic Tools

```csharp
// PerformanceProfiler.cs - Performance monitoring utilities
using System.Diagnostics;
using Microsoft.Extensions.Logging;

public class PerformanceProfiler : IDisposable
{
    private readonly ILogger<PerformanceProfiler> _logger;
    private readonly Stopwatch _stopwatch;
    private readonly string _operationName;
    private readonly long _initialMemory;

    public PerformanceProfiler(ILogger<PerformanceProfiler> logger, string operationName)
    {
        _logger = logger;
        _operationName = operationName;
        _initialMemory = GC.GetTotalMemory(false);
        _stopwatch = Stopwatch.StartNew();
        
        _logger.LogInformation("Starting operation: {OperationName}", operationName);
    }

    public void Dispose()
    {
        _stopwatch.Stop();
        var finalMemory = GC.GetTotalMemory(false);
        var memoryDelta = finalMemory - _initialMemory;
        
        _logger.LogInformation(
            "Completed operation: {OperationName} in {ElapsedMs}ms, Memory delta: {MemoryDelta} bytes",
            _operationName, 
            _stopwatch.ElapsedMilliseconds,
            memoryDelta);
    }

    public static void MeasureAction(ILogger logger, string name, Action action)
    {
        using var profiler = new PerformanceProfiler(logger, name);
        action.Invoke();
    }

    public static async Task MeasureActionAsync(ILogger logger, string name, Func<Task> action)
    {
        using var profiler = new PerformanceProfiler(logger, name);
        await action.Invoke();
    }

    public static T MeasureFunction<T>(ILogger logger, string name, Func<T> function)
    {
        using var profiler = new PerformanceProfiler(logger, name);
        return function.Invoke();
    }
}

// Custom Performance Counters
public class PerformanceCounters
{
    private readonly Dictionary<string, PerformanceCounter> _counters = new();

    public void IncrementCounter(string counterName)
    {
        if (!_counters.ContainsKey(counterName))
        {
            _counters[counterName] = new PerformanceCounter { Count = 0, LastUpdated = DateTime.UtcNow };
        }
        
        _counters[counterName].Count++;
        _counters[counterName].LastUpdated = DateTime.UtcNow;
    }

    public void RecordTiming(string operationName, TimeSpan duration)
    {
        var key = $"{operationName}_timing";
        if (!_counters.ContainsKey(key))
        {
            _counters[key] = new PerformanceCounter();
        }
        
        var counter = _counters[key];
        counter.Count++;
        counter.TotalDuration += duration;
        counter.AverageDuration = TimeSpan.FromTicks(counter.TotalDuration.Ticks / counter.Count);
        counter.LastUpdated = DateTime.UtcNow;
    }

    public Dictionary<string, PerformanceCounter> GetCounters() => new(_counters);

    public class PerformanceCounter
    {
        public long Count { get; set; }
        public TimeSpan TotalDuration { get; set; }
        public TimeSpan AverageDuration { get; set; }
        public DateTime LastUpdated { get; set; }
    }
}
```

### 1.2 Memory Usage Monitoring

```csharp
// MemoryMonitor.cs - Memory usage tracking
public class MemoryMonitor
{
    private readonly Timer _timer;
    private readonly ILogger<MemoryMonitor> _logger;
    private long _peakMemoryUsage;
    private readonly List<MemorySnapshot> _snapshots = new();

    public MemoryMonitor(ILogger<MemoryMonitor> logger)
    {
        _logger = logger;
        _timer = new Timer(TakeSnapshot, null, TimeSpan.Zero, TimeSpan.FromSeconds(5));
    }

    private void TakeSnapshot(object? state)
    {
        var snapshot = new MemorySnapshot
        {
            Timestamp = DateTime.UtcNow,
            TotalMemory = GC.GetTotalMemory(false),
            Generation0Collections = GC.CollectionCount(0),
            Generation1Collections = GC.CollectionCount(1),
            Generation2Collections = GC.CollectionCount(2)
        };

        _snapshots.Add(snapshot);
        
        if (snapshot.TotalMemory > _peakMemoryUsage)
        {
            _peakMemoryUsage = snapshot.TotalMemory;
            _logger.LogInformation("New peak memory usage: {PeakMemory} bytes", _peakMemoryUsage);
        }

        // Keep only last 100 snapshots
        if (_snapshots.Count > 100)
        {
            _snapshots.RemoveAt(0);
        }

        // Alert if memory usage is growing rapidly
        if (_snapshots.Count >= 10)
        {
            var recentSnapshots = _snapshots.TakeLast(10).ToList();
            var memoryGrowth = recentSnapshots.Last().TotalMemory - recentSnapshots.First().TotalMemory;
            
            if (memoryGrowth > 50 * 1024 * 1024) // 50MB growth in last 10 samples
            {
                _logger.LogWarning("Rapid memory growth detected: {Growth} bytes", memoryGrowth);
            }
        }
    }

    public void ForceGarbageCollection()
    {
        _logger.LogInformation("Forcing garbage collection...");
        var beforeMemory = GC.GetTotalMemory(false);
        
        GC.Collect();
        GC.WaitForPendingFinalizers();
        GC.Collect();
        
        var afterMemory = GC.GetTotalMemory(true);
        var freed = beforeMemory - afterMemory;
        
        _logger.LogInformation("Garbage collection completed. Freed: {FreedMemory} bytes", freed);
    }

    public MemoryReport GenerateReport()
    {
        return new MemoryReport
        {
            PeakMemoryUsage = _peakMemoryUsage,
            CurrentMemoryUsage = GC.GetTotalMemory(false),
            TotalSnapshots = _snapshots.Count,
            AverageMemoryUsage = _snapshots.Count > 0 ? (long)_snapshots.Average(s => s.TotalMemory) : 0,
            Generation0Collections = GC.CollectionCount(0),
            Generation1Collections = GC.CollectionCount(1),
            Generation2Collections = GC.CollectionCount(2),
            RecentSnapshots = _snapshots.TakeLast(10).ToList()
        };
    }

    public class MemorySnapshot
    {
        public DateTime Timestamp { get; set; }
        public long TotalMemory { get; set; }
        public int Generation0Collections { get; set; }
        public int Generation1Collections { get; set; }
        public int Generation2Collections { get; set; }
    }

    public class MemoryReport
    {
        public long PeakMemoryUsage { get; set; }
        public long CurrentMemoryUsage { get; set; }
        public long AverageMemoryUsage { get; set; }
        public int TotalSnapshots { get; set; }
        public int Generation0Collections { get; set; }
        public int Generation1Collections { get; set; }
        public int Generation2Collections { get; set; }
        public List<MemorySnapshot> RecentSnapshots { get; set; } = new();
    }

    public void Dispose()
    {
        _timer?.Dispose();
    }
}
```

---

## 2. 🎨 UI Rendering Optimization

### 2.1 Efficient XAML Layouts

```xml
<!-- OptimizedLayouts.xaml - Performance-optimized layout patterns -->
<ContentPage x:Class="PerformanceApp.Views.OptimizedLayouts"
             xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">

    <!-- Efficient Grid Layout with caching -->
    <Grid x:Name="MainGrid" 
          RowDefinitions="Auto,*,Auto" 
          ColumnDefinitions="*,*"
          CachingStrategy="RecycleElement">
        
        <!-- Header with minimal nesting -->
        <Frame Grid.Row="0" Grid.ColumnSpan="2" 
               BackgroundColor="{StaticResource PrimaryColor}"
               Padding="16,12">
            <Label Text="{Binding Title}" 
                   Style="{StaticResource HeaderStyle}"
                   LineBreakMode="TailTruncation"
                   MaxLines="1" />
        </Frame>

        <!-- Optimized CollectionView with virtualization -->
        <CollectionView Grid.Row="1" Grid.ColumnSpan="2"
                        x:Name="ItemsCollectionView"
                        ItemsSource="{Binding Items}"
                        CachingStrategy="RecycleElement"
                        RemainingItemsThreshold="5"
                        RemainingItemsThresholdReachedCommand="{Binding LoadMoreCommand}">
            
            <!-- Efficient ItemTemplate with minimal controls -->
            <CollectionView.ItemTemplate>
                <DataTemplate>
                    <Grid Padding="16,8" 
                          RowDefinitions="Auto,Auto" 
                          ColumnDefinitions="48,*,Auto">
                        
                        <!-- Cached image with placeholder -->
                        <Frame Grid.RowSpan="2" 
                               WidthRequest="40" 
                               HeightRequest="40"
                               CornerRadius="20"
                               IsClippedToBounds="True"
                               Padding="0">
                            <Image Source="{Binding ImageUrl, TargetNullValue={StaticResource DefaultAvatar}}"
                                   Aspect="AspectFill"
                                   CachingEnabled="True"
                                   LoadingPlaceholder="{StaticResource PlaceholderImage}"
                                   ErrorPlaceholder="{StaticResource ErrorImage}" />
                        </Frame>

                        <!-- Text content with performance optimizations -->
                        <Label Grid.Column="1" 
                               Text="{Binding Title}"
                               Style="{StaticResource ListItemTitleStyle}"
                               LineBreakMode="TailTruncation"
                               MaxLines="1" />
                        
                        <Label Grid.Row="1" Grid.Column="1"
                               Text="{Binding Description}"
                               Style="{StaticResource ListItemSubtitleStyle}"
                               LineBreakMode="TailTruncation"
                               MaxLines="2" />

                        <!-- Action button with command binding -->
                        <Button Grid.RowSpan="2" Grid.Column="2"
                                Text="Action"
                                Style="{StaticResource SecondaryButtonStyle}"
                                Command="{Binding Source={x:Reference ItemsCollectionView}, Path=BindingContext.ItemActionCommand}"
                                CommandParameter="{Binding}" />
                    </Grid>
                </DataTemplate>
            </CollectionView.ItemTemplate>

            <!-- Custom header/footer for better performance -->
            <CollectionView.Header>
                <ContentView HeightRequest="8" />
            </CollectionView.Header>
            
            <CollectionView.Footer>
                <ActivityIndicator IsVisible="{Binding IsLoading}"
                                   IsRunning="{Binding IsLoading}"
                                   HeightRequest="44" />
            </CollectionView.Footer>
        </CollectionView>

        <!-- Bottom action bar -->
        <StackLayout Grid.Row="2" Grid.ColumnSpan="2"
                     Orientation="Horizontal"
                     BackgroundColor="{StaticResource SurfaceColor}"
                     Padding="16,8">
            
            <Button Text="Add Item"
                    Style="{StaticResource PrimaryButtonStyle}"
                    Command="{Binding AddItemCommand}"
                    HorizontalOptions="FillAndExpand" />
            
            <Button Text="Refresh"
                    Style="{StaticResource SecondaryButtonStyle}"
                    Command="{Binding RefreshCommand}"
                    HorizontalOptions="FillAndExpand" />
        </StackLayout>
    </Grid>
</ContentPage>
```

### 2.2 Custom Renderer Optimization

```csharp
// Platforms/Android/Renderers/OptimizedEntryRenderer.cs
#if ANDROID
using AndroidX.Core.Content;
using Microsoft.Maui.Handlers;
using Microsoft.Maui.Platform;

public class OptimizedEntryRenderer : EntryHandler
{
    private bool _isDisposed;

    protected override MauiTextView CreatePlatformView()
    {
        var editText = new MauiTextView(Context)
        {
            ImeOptions = Android.Views.InputMethods.ImeAction.Done,
            InputType = Android.Text.InputTypes.ClassText
        };

        // Optimize for performance
        editText.SetIncludeFontPadding(false);
        editText.SetSelectAllOnFocus(true);
        
        // Reduce memory allocations
        editText.SetTextIsSelectable(true);
        editText.SetCursorVisible(true);
        
        return editText;
    }

    protected override void ConnectHandler(MauiTextView platformView)
    {
        base.ConnectHandler(platformView);
        
        if (platformView != null)
        {
            // Optimize event handling
            platformView.TextChanged += OnTextChanged;
            platformView.FocusChange += OnFocusChanged;
        }
    }

    protected override void DisconnectHandler(MauiTextView platformView)
    {
        if (platformView != null && !_isDisposed)
        {
            // Clean up event handlers to prevent memory leaks
            platformView.TextChanged -= OnTextChanged;
            platformView.FocusChange -= OnFocusChanged;
        }
        
        base.DisconnectHandler(platformView);
    }

    private void OnTextChanged(object sender, Android.Text.TextChangedEventArgs e)
    {
        if (VirtualView != null && !_isDisposed)
        {
            VirtualView.Text = e.Text?.ToString();
        }
    }

    private void OnFocusChanged(object sender, Android.Views.View.FocusChangeEventArgs e)
    {
        if (VirtualView != null && !_isDisposed)
        {
            if (e.HasFocus)
                VirtualView.Focus();
            else
                VirtualView.Unfocus();
        }
    }

    protected override void Dispose(bool disposing)
    {
        if (disposing && !_isDisposed)
        {
            _isDisposed = true;
        }
        
        base.Dispose(disposing);
    }
}
#endif
```

---

## 3. 💾 Memory Management

### 3.1 Efficient Data Loading

```csharp
// Services/OptimizedDataService.cs
public class OptimizedDataService : IDataService
{
    private readonly HttpClient _httpClient;
    private readonly IMemoryCache _cache;
    private readonly ILogger<OptimizedDataService> _logger;
    private readonly SemaphoreSlim _semaphore = new(3); // Limit concurrent requests

    public OptimizedDataService(
        HttpClient httpClient,
        IMemoryCache cache,
        ILogger<OptimizedDataService> logger)
    {
        _httpClient = httpClient;
        _cache = cache;
        _logger = logger;
    }

    public async Task<List<DataItem>> LoadDataAsync(int page, int pageSize, CancellationToken cancellationToken = default)
    {
        var cacheKey = $"data_page_{page}_{pageSize}";
        
        // Try cache first
        if (_cache.TryGetValue(cacheKey, out List<DataItem> cachedData))
        {
            _logger.LogDebug("Retrieved data from cache for page {Page}", page);
            return cachedData;
        }

        await _semaphore.WaitAsync(cancellationToken);
        
        try
        {
            using var profiler = new PerformanceProfiler(_logger, $"LoadData_Page_{page}");
            
            var response = await _httpClient.GetAsync(
                $"api/data?page={page}&pageSize={pageSize}", 
                cancellationToken);
            
            if (!response.IsSuccessStatusCode)
            {
                throw new HttpRequestException($"Failed to load data: {response.StatusCode}");
            }

            // Use streaming JSON for large datasets
            await using var stream = await response.Content.ReadAsStreamAsync(cancellationToken);
            var data = await JsonSerializer.DeserializeAsync<List<DataItem>>(
                stream, 
                JsonOptions.Default, 
                cancellationToken);

            // Cache with expiration
            var cacheOptions = new MemoryCacheEntryOptions
            {
                SlidingExpiration = TimeSpan.FromMinutes(10),
                Size = data?.Count ?? 0,
                Priority = CacheItemPriority.Normal
            };
            
            _cache.Set(cacheKey, data, cacheOptions);
            
            _logger.LogInformation("Loaded {Count} items for page {Page}", data?.Count ?? 0, page);
            return data ?? new List<DataItem>();
        }
        finally
        {
            _semaphore.Release();
        }
    }

    public async Task<DataItem?> LoadItemAsync(int id, CancellationToken cancellationToken = default)
    {
        var cacheKey = $"item_{id}";
        
        if (_cache.TryGetValue(cacheKey, out DataItem cachedItem))
        {
            return cachedItem;
        }

        await _semaphore.WaitAsync(cancellationToken);
        
        try
        {
            var response = await _httpClient.GetAsync($"api/data/{id}", cancellationToken);
            
            if (response.StatusCode == HttpStatusCode.NotFound)
            {
                return null;
            }
            
            response.EnsureSuccessStatusCode();
            
            await using var stream = await response.Content.ReadAsStreamAsync(cancellationToken);
            var item = await JsonSerializer.DeserializeAsync<DataItem>(
                stream, 
                JsonOptions.Default, 
                cancellationToken);

            if (item != null)
            {
                var cacheOptions = new MemoryCacheEntryOptions
                {
                    SlidingExpiration = TimeSpan.FromMinutes(30),
                    Size = 1,
                    Priority = CacheItemPriority.High
                };
                
                _cache.Set(cacheKey, item, cacheOptions);
            }

            return item;
        }
        finally
        {
            _semaphore.Release();
        }
    }

    // Batch loading for better performance
    public async Task<Dictionary<int, DataItem>> LoadItemsBatchAsync(
        IEnumerable<int> ids, 
        CancellationToken cancellationToken = default)
    {
        var result = new Dictionary<int, DataItem>();
        var uncachedIds = new List<int>();

        // Check cache first
        foreach (var id in ids)
        {
            var cacheKey = $"item_{id}";
            if (_cache.TryGetValue(cacheKey, out DataItem cachedItem))
            {
                result[id] = cachedItem;
            }
            else
            {
                uncachedIds.Add(id);
            }
        }

        if (!uncachedIds.Any())
        {
            return result;
        }

        await _semaphore.WaitAsync(cancellationToken);
        
        try
        {
            var idsQuery = string.Join(",", uncachedIds);
            var response = await _httpClient.GetAsync(
                $"api/data/batch?ids={idsQuery}", 
                cancellationToken);
            
            response.EnsureSuccessStatusCode();
            
            await using var stream = await response.Content.ReadAsStreamAsync(cancellationToken);
            var items = await JsonSerializer.DeserializeAsync<List<DataItem>>(
                stream, 
                JsonOptions.Default, 
                cancellationToken);

            if (items != null)
            {
                foreach (var item in items)
                {
                    result[item.Id] = item;
                    
                    // Cache individual items
                    var cacheKey = $"item_{item.Id}";
                    var cacheOptions = new MemoryCacheEntryOptions
                    {
                        SlidingExpiration = TimeSpan.FromMinutes(30),
                        Size = 1,
                        Priority = CacheItemPriority.High
                    };
                    
                    _cache.Set(cacheKey, item, cacheOptions);
                }
            }

            return result;
        }
        finally
        {
            _semaphore.Release();
        }
    }

    public void ClearCache()
    {
        if (_cache is MemoryCache memoryCache)
        {
            memoryCache.Clear();
            _logger.LogInformation("Data cache cleared");
        }
    }

    public void Dispose()
    {
        _semaphore?.Dispose();
    }
}

// JSON serialization options for performance
public static class JsonOptions
{
    public static readonly JsonSerializerOptions Default = new()
    {
        PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
        PropertyNameCaseInsensitive = true,
        DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull,
        NumberHandling = JsonNumberHandling.AllowReadingFromString,
        ReadCommentHandling = JsonCommentHandling.Skip,
        AllowTrailingCommas = true
    };
}
```

### 3.2 Image Caching and Optimization

```csharp
// Services/ImageCacheService.cs
public class ImageCacheService : IImageCacheService
{
    private readonly HttpClient _httpClient;
    private readonly string _cacheDirectory;
    private readonly ILogger<ImageCacheService> _logger;
    private readonly SemaphoreSlim _downloadSemaphore = new(5);
    private readonly ConcurrentDictionary<string, Task<string?>> _downloadTasks = new();

    public ImageCacheService(HttpClient httpClient, ILogger<ImageCacheService> logger)
    {
        _httpClient = httpClient;
        _logger = logger;
        _cacheDirectory = Path.Combine(FileSystem.AppDataDirectory, "ImageCache");
        
        if (!Directory.Exists(_cacheDirectory))
        {
            Directory.CreateDirectory(_cacheDirectory);
        }
    }

    public async Task<string?> GetCachedImagePathAsync(string imageUrl, CancellationToken cancellationToken = default)
    {
        if (string.IsNullOrEmpty(imageUrl))
            return null;

        var fileName = GetCacheFileName(imageUrl);
        var localPath = Path.Combine(_cacheDirectory, fileName);

        // Return cached file if exists and not expired
        if (File.Exists(localPath) && !IsCacheExpired(localPath))
        {
            return localPath;
        }

        // Use task-based downloading to prevent duplicate downloads
        return await _downloadTasks.GetOrAdd(imageUrl, async url =>
        {
            await _downloadSemaphore.WaitAsync(cancellationToken);
            try
            {
                return await DownloadAndCacheImageAsync(url, localPath, cancellationToken);
            }
            finally
            {
                _downloadSemaphore.Release();
                _downloadTasks.TryRemove(url, out _);
            }
        });
    }

    private async Task<string?> DownloadAndCacheImageAsync(
        string imageUrl, 
        string localPath, 
        CancellationToken cancellationToken)
    {
        try
        {
            using var response = await _httpClient.GetAsync(imageUrl, cancellationToken);
            
            if (!response.IsSuccessStatusCode)
            {
                _logger.LogWarning("Failed to download image: {Url} - {StatusCode}", imageUrl, response.StatusCode);
                return null;
            }

            // Create optimized image
            await using var sourceStream = await response.Content.ReadAsStreamAsync(cancellationToken);
            var optimizedPath = await OptimizeAndSaveImageAsync(sourceStream, localPath, cancellationToken);
            
            _logger.LogDebug("Cached image: {Url} -> {Path}", imageUrl, optimizedPath);
            return optimizedPath;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error downloading image: {Url}", imageUrl);
            return null;
        }
    }

    private async Task<string> OptimizeAndSaveImageAsync(
        Stream sourceStream, 
        string outputPath, 
        CancellationToken cancellationToken)
    {
        using var image = await Image.LoadAsync(sourceStream, cancellationToken);
        
        // Resize if too large
        if (image.Width > 800 || image.Height > 800)
        {
            var ratio = Math.Min(800.0 / image.Width, 800.0 / image.Height);
            var newWidth = (int)(image.Width * ratio);
            var newHeight = (int)(image.Height * ratio);
            
            image.Mutate(x => x.Resize(newWidth, newHeight));
        }

        // Save with optimized quality
        var encoder = new JpegEncoder
        {
            Quality = 85,
            Subsample = JpegSubsample.Ratio420
        };

        await image.SaveAsync(outputPath, encoder, cancellationToken);
        return outputPath;
    }

    private string GetCacheFileName(string url)
    {
        using var sha256 = SHA256.Create();
        var hash = sha256.ComputeHash(Encoding.UTF8.GetBytes(url));
        var hashString = Convert.ToHexString(hash).ToLowerInvariant();
        return $"{hashString}.jpg";
    }

    private bool IsCacheExpired(string filePath)
    {
        var fileInfo = new FileInfo(filePath);
        return DateTime.UtcNow - fileInfo.CreationTimeUtc > TimeSpan.FromDays(7);
    }

    public async Task ClearCacheAsync()
    {
        try
        {
            if (Directory.Exists(_cacheDirectory))
            {
                var files = Directory.GetFiles(_cacheDirectory);
                foreach (var file in files)
                {
                    File.Delete(file);
                }
                
                _logger.LogInformation("Cleared image cache: {Count} files deleted", files.Length);
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error clearing image cache");
        }
    }

    public long GetCacheSize()
    {
        try
        {
            if (!Directory.Exists(_cacheDirectory))
                return 0;

            return Directory.GetFiles(_cacheDirectory)
                .Sum(file => new FileInfo(file).Length);
        }
        catch
        {
            return 0;
        }
    }

    public void Dispose()
    {
        _downloadSemaphore?.Dispose();
    }
}
```

---

## 4. ⚡ Async/Await Optimization

### 4.1 Efficient Async Patterns

```csharp
// ViewModels/OptimizedViewModel.cs
public class OptimizedViewModel : BaseViewModel
{
    private readonly IDataService _dataService;
    private readonly IImageCacheService _imageService;
    private readonly CancellationTokenSource _cancellationTokenSource = new();
    
    public OptimizedViewModel(IDataService dataService, IImageCacheService imageService)
    {
        _dataService = dataService;
        _imageService = imageService;
        
        LoadDataCommand = new AsyncRelayCommand(LoadDataAsync, CanLoadData);
        RefreshCommand = new AsyncRelayCommand(RefreshDataAsync);
        LoadMoreCommand = new AsyncRelayCommand(LoadMoreDataAsync, CanLoadMore);
    }

    [ObservableProperty]
    private ObservableCollection<DataItemViewModel> _items = new();

    [ObservableProperty]
    private bool _isLoading;

    [ObservableProperty]
    private bool _isRefreshing;

    [ObservableProperty]
    private bool _isLoadingMore;

    private int _currentPage = 1;
    private bool _hasMoreItems = true;
    private const int PageSize = 20;

    public IAsyncRelayCommand LoadDataCommand { get; }
    public IAsyncRelayCommand RefreshCommand { get; }
    public IAsyncRelayCommand LoadMoreCommand { get; }

    private async Task LoadDataAsync()
    {
        if (IsLoading) return;

        try
        {
            IsLoading = true;
            
            // Use ConfigureAwait(false) to avoid deadlocks
            var data = await _dataService.LoadDataAsync(1, PageSize, _cancellationTokenSource.Token)
                .ConfigureAwait(false);

            // Update UI on main thread
            await MainThread.InvokeOnMainThreadAsync(() =>
            {
                Items.Clear();
                foreach (var item in data)
                {
                    Items.Add(new DataItemViewModel(item, _imageService));
                }
            });

            _currentPage = 1;
            _hasMoreItems = data.Count == PageSize;
        }
        catch (OperationCanceledException)
        {
            // Handle cancellation gracefully
        }
        catch (Exception ex)
        {
            await HandleErrorAsync(ex);
        }
        finally
        {
            IsLoading = false;
        }
    }

    private async Task RefreshDataAsync()
    {
        if (IsRefreshing) return;

        try
        {
            IsRefreshing = true;
            
            // Clear cache before refresh
            _dataService.ClearCache();
            
            var data = await _dataService.LoadDataAsync(1, PageSize, _cancellationTokenSource.Token)
                .ConfigureAwait(false);

            await MainThread.InvokeOnMainThreadAsync(() =>
            {
                Items.Clear();
                foreach (var item in data)
                {
                    Items.Add(new DataItemViewModel(item, _imageService));
                }
            });

            _currentPage = 1;
            _hasMoreItems = data.Count == PageSize;
        }
        catch (OperationCanceledException)
        {
            // Handle cancellation gracefully
        }
        catch (Exception ex)
        {
            await HandleErrorAsync(ex);
        }
        finally
        {
            IsRefreshing = false;
        }
    }

    private async Task LoadMoreDataAsync()
    {
        if (IsLoadingMore || !_hasMoreItems) return;

        try
        {
            IsLoadingMore = true;
            
            var nextPage = _currentPage + 1;
            var data = await _dataService.LoadDataAsync(nextPage, PageSize, _cancellationTokenSource.Token)
                .ConfigureAwait(false);

            if (data.Any())
            {
                await MainThread.InvokeOnMainThreadAsync(() =>
                {
                    foreach (var item in data)
                    {
                        Items.Add(new DataItemViewModel(item, _imageService));
                    }
                });

                _currentPage = nextPage;
                _hasMoreItems = data.Count == PageSize;
            }
            else
            {
                _hasMoreItems = false;
            }
        }
        catch (OperationCanceledException)
        {
            // Handle cancellation gracefully
        }
        catch (Exception ex)
        {
            await HandleErrorAsync(ex);
        }
        finally
        {
            IsLoadingMore = false;
        }
    }

    private bool CanLoadData() => !IsLoading && !IsRefreshing;
    private bool CanLoadMore() => !IsLoadingMore && _hasMoreItems && !IsLoading;

    private async Task HandleErrorAsync(Exception ex)
    {
        await MainThread.InvokeOnMainThreadAsync(async () =>
        {
            await Application.Current?.MainPage?.DisplayAlert(
                "Error", 
                $"An error occurred: {ex.Message}", 
                "OK");
        });
    }

    protected override void Dispose(bool disposing)
    {
        if (disposing)
        {
            _cancellationTokenSource?.Cancel();
            _cancellationTokenSource?.Dispose();
        }
        
        base.Dispose(disposing);
    }
}

// Optimized item view model with lazy loading
public class DataItemViewModel : BaseViewModel
{
    private readonly DataItem _dataItem;
    private readonly IImageCacheService _imageService;
    private string? _cachedImagePath;

    public DataItemViewModel(DataItem dataItem, IImageCacheService imageService)
    {
        _dataItem = dataItem;
        _imageService = imageService;
    }

    public int Id => _dataItem.Id;
    public string Title => _dataItem.Title;
    public string Description => _dataItem.Description;
    public DateTime CreatedAt => _dataItem.CreatedAt;

    public string ImagePath
    {
        get
        {
            if (_cachedImagePath == null && !string.IsNullOrEmpty(_dataItem.ImageUrl))
            {
                LoadImageAsync();
                return "placeholder.png"; // Return placeholder while loading
            }
            return _cachedImagePath ?? "placeholder.png";
        }
        private set => SetProperty(ref _cachedImagePath, value);
    }

    private async void LoadImageAsync()
    {
        try
        {
            var imagePath = await _imageService.GetCachedImagePathAsync(_dataItem.ImageUrl);
            if (!string.IsNullOrEmpty(imagePath))
            {
                await MainThread.InvokeOnMainThreadAsync(() =>
                {
                    ImagePath = imagePath;
                });
            }
        }
        catch
        {
            // Ignore image loading errors
        }
    }
}
```

---

## 5. 🧪 Practical Exercise

### Exercise: Performance Benchmark Suite

สร้าง comprehensive performance testing suite:

```csharp
// Tests/PerformanceTests/PerformanceBenchmark.cs
[MemoryDiagnoser]
[SimpleJob(RuntimeMoniker.Net80)]
public class PerformanceBenchmark
{
    private List<DataItem> _testData;
    private ObservableCollection<DataItemViewModel> _viewModels;
    private IDataService _dataService;
    private IMemoryCache _cache;

    [GlobalSetup]
    public void GlobalSetup()
    {
        _testData = GenerateTestData(1000);
        _cache = new MemoryCache(new MemoryCacheOptions { SizeLimit = 1000 });
        _dataService = new MockDataService(_cache);
        _viewModels = new ObservableCollection<DataItemViewModel>();
    }

    [Benchmark]
    public void CreateViewModels()
    {
        _viewModels.Clear();
        foreach (var item in _testData.Take(100))
        {
            _viewModels.Add(new DataItemViewModel(item, null));
        }
    }

    [Benchmark]
    public async Task LoadDataWithCaching()
    {
        await _dataService.LoadDataAsync(1, 50);
    }

    [Benchmark]
    public async Task LoadDataWithoutCaching()
    {
        _cache.Clear();
        await _dataService.LoadDataAsync(1, 50);
    }

    [Benchmark]
    public void FilterLargeDataset()
    {
        var filtered = _testData
            .Where(x => x.Title.Contains("test", StringComparison.OrdinalIgnoreCase))
            .OrderBy(x => x.CreatedAt)
            .Take(50)
            .ToList();
    }

    [Benchmark]
    public void SortLargeDataset()
    {
        var sorted = _testData
            .OrderByDescending(x => x.CreatedAt)
            .ThenBy(x => x.Title)
            .ToList();
    }

    private List<DataItem> GenerateTestData(int count)
    {
        var random = new Random(42); // Fixed seed for consistent results
        return Enumerable.Range(1, count)
            .Select(i => new DataItem
            {
                Id = i,
                Title = $"Test Item {i}",
                Description = $"Description for test item {i}",
                CreatedAt = DateTime.UtcNow.AddDays(-random.Next(365)),
                ImageUrl = $"https://picsum.photos/200/200?random={i}"
            })
            .ToList();
    }
}

// Run benchmarks
public class Program
{
    public static void Main(string[] args)
    {
        var summary = BenchmarkRunner.Run<PerformanceBenchmark>();
        Console.WriteLine(summary);
    }
}
```

### Performance Testing Checklist

**Memory Performance:**
- [ ] No memory leaks in long-running scenarios
- [ ] Efficient garbage collection patterns
- [ ] Proper disposal of resources
- [ ] Event handler cleanup
- [ ] Image cache size management

**UI Performance:**
- [ ] Smooth scrolling (60 FPS target)
- [ ] Fast page navigation (<300ms)
- [ ] Responsive touch interactions
- [ ] Efficient layout calculations
- [ ] Optimized data binding

**Network Performance:**
- [ ] Request deduplication
- [ ] Efficient caching strategies
- [ ] Connection pooling
- [ ] Timeout handling
- [ ] Offline-first architecture

**Startup Performance:**
- [ ] Fast app launch (<2 seconds)
- [ ] Minimal splash screen duration
- [ ] Lazy loading of non-critical components
- [ ] Efficient dependency injection setup
- [ ] Optimized image loading

---

## 6. 📚 Summary

ในบทนี้เราได้เรียนรู้:

### 🔑 Key Concepts
- **Performance Profiling** และการใช้ diagnostic tools
- **Memory Management** patterns และ garbage collection optimization
- **UI Rendering** optimization strategies
- **Async/Await** best practices สำหรับ MAUI
- **Caching Strategies** สำหรับข้อมูลและรูปภาพ

### 🛠️ Technical Skills
- การใช้ Visual Studio diagnostic tools
- การสร้าง custom performance profilers
- การ optimize XAML layouts และ rendering
- การจัดการ memory leaks และ resource cleanup
- การใช้ efficient async patterns

### 📊 Performance Optimization Techniques
- **Memory Optimization:** Efficient object lifecycle management, proper disposal patterns
- **UI Optimization:** Layout caching, virtualization, efficient data binding
- **Network Optimization:** Request batching, intelligent caching, connection management
- **Startup Optimization:** Lazy loading, dependency injection optimization

### 🧪 Testing & Monitoring
- การสร้าง performance benchmarks
- การ monitor memory usage in production
- การใช้ automated performance testing
- การ analyze performance bottlenecks

Performance optimization เป็นกระบวนการต่อเนื่องที่ต้องการการ measure, analyze, และ improve อย่างสม่ำเสมอ การมี proper monitoring และ testing strategies จะช่วยให้แอปพลิเคชันของคุณทำงานได้อย่างมีประสิทธิภาพและให้ user experience ที่ดี

**Next Chapter Preview:** ต่อไปเราจะเรียนรู้เกี่ยวกับ Testing Strategies และการสร้าง comprehensive test suite สำหรับ MAUI applications! 🧪
