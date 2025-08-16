# ⚡ Chapter 14: Performance Optimization

> **หัวข้อ**: Memory Management, UI Performance & Resource Optimization  
> **ระยะเวลา**: 4 ชั่วโมง  
> **ความยากง่าย**: ระดับสูง  
> **Prerequisites**: บทที่ 1-13

## 🎯 Learning Objectives

หลังจากเรียนบทนี้แล้ว คุณจะสามารถ:

- ✅ วิเคราะห์และปรับปรุง app performance
- ✅ จัดการ memory และ resource อย่างมีประสิทธิภาพ
- ✅ ปรับแต่ง UI rendering performance
- ✅ ใช้ profiling tools สำหรับ performance debugging
- ✅ สร้าง high-performance MAUI applications

---

## 📱 Chapter Overview

### 🏗️ What We'll Optimize
สร้าง **"Performance Dashboard App"** ที่มี:
- 📊 Real-time performance monitoring
- 🧠 Memory usage tracking
- ⚡ UI performance optimization
- 🔧 Resource management tools
- 📈 Performance analytics

---

## 1. 🧠 Memory Management Strategies

### 1.1 Memory Profiling และ Monitoring

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\14-Performance\Services\MemoryProfiler.cs
using System.Diagnostics;

namespace Performance.Services;

public interface IMemoryProfiler
{
    Task<MemorySnapshot> TakeSnapshotAsync();
    Task StartMonitoringAsync();
    Task StopMonitoringAsync();
    event EventHandler<MemoryWarningEventArgs> MemoryWarning;
    event EventHandler<MemorySnapshotEventArgs> SnapshotTaken;
}

public class MemoryProfiler : IMemoryProfiler
{
    private readonly Timer _monitoringTimer;
    private bool _isMonitoring;
    private long _baselineMemory;
    private readonly List<MemorySnapshot> _snapshots = new();

    public event EventHandler<MemoryWarningEventArgs> MemoryWarning;
    public event EventHandler<MemorySnapshotEventArgs> SnapshotTaken;

    private const long WARNING_THRESHOLD_MB = 100; // 100MB
    private const long CRITICAL_THRESHOLD_MB = 200; // 200MB

    public MemoryProfiler()
    {
        _monitoringTimer = new Timer(MonitorMemoryUsage, null, Timeout.Infinite, Timeout.Infinite);
    }

    public async Task<MemorySnapshot> TakeSnapshotAsync()
    {
        var snapshot = new MemorySnapshot
        {
            Timestamp = DateTime.UtcNow,
            WorkingSet = GC.GetTotalMemory(false),
            TotalAllocatedBytes = GC.GetTotalAllocatedBytes(),
            Gen0Collections = GC.CollectionCount(0),
            Gen1Collections = GC.CollectionCount(1),
            Gen2Collections = GC.CollectionCount(2)
        };

        // Get platform-specific memory info
        await EnrichWithPlatformInfoAsync(snapshot);

        _snapshots.Add(snapshot);
        
        // Keep only last 100 snapshots
        if (_snapshots.Count > 100)
        {
            _snapshots.RemoveAt(0);
        }

        SnapshotTaken?.Invoke(this, new MemorySnapshotEventArgs { Snapshot = snapshot });
        
        // Check for warnings
        CheckMemoryWarnings(snapshot);

        return snapshot;
    }

    public async Task StartMonitoringAsync()
    {
        if (_isMonitoring) return;

        _baselineMemory = GC.GetTotalMemory(false);
        _isMonitoring = true;
        
        // Monitor every 5 seconds
        _monitoringTimer.Change(TimeSpan.Zero, TimeSpan.FromSeconds(5));
        
        System.Diagnostics.Debug.WriteLine("Memory monitoring started");
    }

    public async Task StopMonitoringAsync()
    {
        if (!_isMonitoring) return;

        _isMonitoring = false;
        _monitoringTimer.Change(Timeout.Infinite, Timeout.Infinite);
        
        System.Diagnostics.Debug.WriteLine("Memory monitoring stopped");
    }

    private async void MonitorMemoryUsage(object state)
    {
        if (!_isMonitoring) return;

        try
        {
            await TakeSnapshotAsync();
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"Memory monitoring error: {ex.Message}");
        }
    }

    private async Task EnrichWithPlatformInfoAsync(MemorySnapshot snapshot)
    {
#if ANDROID
        var activityManager = Platform.CurrentActivity?.GetSystemService(Context.ActivityService) as Android.App.ActivityManager;
        if (activityManager != null)
        {
            var memoryInfo = new Android.App.ActivityManager.MemoryInfo();
            activityManager.GetMemoryInfo(memoryInfo);
            
            snapshot.PlatformTotalMemory = memoryInfo.TotalMem;
            snapshot.PlatformAvailableMemory = memoryInfo.AvailMem;
            snapshot.PlatformLowMemory = memoryInfo.LowMemory;
        }
#elif IOS
        var info = new Foundation.NSProcessInfo();
        snapshot.PlatformTotalMemory = (long)info.PhysicalMemory;
#elif WINDOWS
        var pc = new PerformanceCounter("Process", "Working Set", Process.GetCurrentProcess().ProcessName);
        snapshot.PlatformTotalMemory = (long)pc.NextValue();
#endif
    }

    private void CheckMemoryWarnings(MemorySnapshot snapshot)
    {
        var currentMB = snapshot.WorkingSet / (1024 * 1024);
        var baselineMB = _baselineMemory / (1024 * 1024);
        var growthMB = currentMB - baselineMB;

        if (growthMB > CRITICAL_THRESHOLD_MB)
        {
            MemoryWarning?.Invoke(this, new MemoryWarningEventArgs
            {
                Level = MemoryWarningLevel.Critical,
                CurrentUsage = currentMB,
                Growth = growthMB,
                Message = $"Critical memory usage: {currentMB}MB (grew {growthMB}MB from baseline)"
            });
        }
        else if (growthMB > WARNING_THRESHOLD_MB)
        {
            MemoryWarning?.Invoke(this, new MemoryWarningEventArgs
            {
                Level = MemoryWarningLevel.Warning,
                CurrentUsage = currentMB,
                Growth = growthMB,
                Message = $"High memory usage: {currentMB}MB (grew {growthMB}MB from baseline)"
            });
        }
    }

    public List<MemorySnapshot> GetRecentSnapshots(int count = 20)
    {
        return _snapshots.TakeLast(count).ToList();
    }

    public void Dispose()
    {
        _monitoringTimer?.Dispose();
    }
}

public class MemorySnapshot
{
    public DateTime Timestamp { get; set; }
    public long WorkingSet { get; set; }
    public long TotalAllocatedBytes { get; set; }
    public int Gen0Collections { get; set; }
    public int Gen1Collections { get; set; }
    public int Gen2Collections { get; set; }
    public long PlatformTotalMemory { get; set; }
    public long PlatformAvailableMemory { get; set; }
    public bool PlatformLowMemory { get; set; }

    public long WorkingSetMB => WorkingSet / (1024 * 1024);
    public long TotalAllocatedMB => TotalAllocatedBytes / (1024 * 1024);
}

public class MemoryWarningEventArgs : EventArgs
{
    public MemoryWarningLevel Level { get; set; }
    public long CurrentUsage { get; set; }
    public long Growth { get; set; }
    public string Message { get; set; }
}

public class MemorySnapshotEventArgs : EventArgs
{
    public MemorySnapshot Snapshot { get; set; }
}

public enum MemoryWarningLevel
{
    Info,
    Warning,
    Critical
}
```

### 1.2 Object Pool Pattern สำหรับ Memory Efficiency

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\14-Performance\Patterns\ObjectPool.cs
using System.Collections.Concurrent;

namespace Performance.Patterns;

public interface IObjectPool<T>
{
    T Get();
    void Return(T item);
    void Clear();
    int Count { get; }
}

public class ObjectPool<T> : IObjectPool<T> where T : class, new()
{
    private readonly ConcurrentQueue<T> _objects = new();
    private readonly Func<T> _objectGenerator;
    private readonly Action<T> _resetAction;
    private readonly int _maxSize;
    private int _currentCount;

    public int Count => _currentCount;

    public ObjectPool(Func<T> objectGenerator = null, Action<T> resetAction = null, int maxSize = 100)
    {
        _objectGenerator = objectGenerator ?? (() => new T());
        _resetAction = resetAction;
        _maxSize = maxSize;
    }

    public T Get()
    {
        if (_objects.TryDequeue(out T item))
        {
            Interlocked.Decrement(ref _currentCount);
            return item;
        }

        return _objectGenerator();
    }

    public void Return(T item)
    {
        if (item == null) return;

        if (_currentCount < _maxSize)
        {
            _resetAction?.Invoke(item);
            _objects.Enqueue(item);
            Interlocked.Increment(ref _currentCount);
        }
    }

    public void Clear()
    {
        while (_objects.TryDequeue(out _))
        {
            Interlocked.Decrement(ref _currentCount);
        }
    }
}

// Specific pools for common UI objects
public static class ViewModelPools
{
    public static readonly ObjectPool<ListItemViewModel> ListItemPool = new(
        objectGenerator: () => new ListItemViewModel(),
        resetAction: item => item.Reset(),
        maxSize: 50
    );

    public static readonly ObjectPool<StringBuilder> StringBuilderPool = new(
        objectGenerator: () => new StringBuilder(),
        resetAction: sb => sb.Clear(),
        maxSize: 20
    );

    public static readonly ObjectPool<HttpClient> HttpClientPool = new(
        objectGenerator: () => new HttpClient(),
        resetAction: client => client.DefaultRequestHeaders.Clear(),
        maxSize: 5
    );
}

// Example usage in ViewModels
public class ListItemViewModel : INotifyPropertyChanged
{
    private string _title;
    private string _subtitle;
    private bool _isSelected;

    public string Title
    {
        get => _title;
        set => SetProperty(ref _title, value);
    }

    public string Subtitle
    {
        get => _subtitle;
        set => SetProperty(ref _subtitle, value);
    }

    public bool IsSelected
    {
        get => _isSelected;
        set => SetProperty(ref _isSelected, value);
    }

    public void Reset()
    {
        Title = null;
        Subtitle = null;
        IsSelected = false;
    }

    public event PropertyChangedEventHandler PropertyChanged;

    protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }

    protected bool SetProperty<T>(ref T backingStore, T value, [CallerMemberName] string propertyName = "")
    {
        if (EqualityComparer<T>.Default.Equals(backingStore, value))
            return false;

        backingStore = value;
        OnPropertyChanged(propertyName);
        return true;
    }
}
```

### 1.3 Memory-Efficient Collections

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\14-Performance\Collections\VirtualizedCollection.cs
using System.Collections.Specialized;

namespace Performance.Collections;

public class VirtualizedObservableCollection<T> : INotifyCollectionChanged, IList<T>
{
    private readonly IList<T> _backingStore;
    private readonly int _pageSize;
    private readonly Dictionary<int, List<T>> _loadedPages = new();
    private readonly Func<int, int, Task<List<T>>> _dataLoader;

    public event NotifyCollectionChangedEventHandler CollectionChanged;

    public VirtualizedObservableCollection(
        IList<T> backingStore,
        Func<int, int, Task<List<T>>> dataLoader,
        int pageSize = 50)
    {
        _backingStore = backingStore;
        _dataLoader = dataLoader;
        _pageSize = pageSize;
    }

    public T this[int index]
    {
        get => GetItemAsync(index).Result;
        set => SetItem(index, value);
    }

    public int Count => _backingStore.Count;
    public bool IsReadOnly => false;

    private async Task<T> GetItemAsync(int index)
    {
        var pageNumber = index / _pageSize;
        
        if (!_loadedPages.ContainsKey(pageNumber))
        {
            await LoadPageAsync(pageNumber);
        }

        var pageIndex = index % _pageSize;
        var page = _loadedPages[pageNumber];
        
        return pageIndex < page.Count ? page[pageIndex] : default(T);
    }

    private async Task LoadPageAsync(int pageNumber)
    {
        try
        {
            var skip = pageNumber * _pageSize;
            var data = await _dataLoader(skip, _pageSize);
            
            _loadedPages[pageNumber] = data;
            
            // Unload old pages to manage memory
            if (_loadedPages.Count > 10) // Keep only 10 pages in memory
            {
                var oldestPage = _loadedPages.Keys.Min();
                _loadedPages.Remove(oldestPage);
            }
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"Failed to load page {pageNumber}: {ex.Message}");
            _loadedPages[pageNumber] = new List<T>();
        }
    }

    private void SetItem(int index, T value)
    {
        if (index < _backingStore.Count)
        {
            _backingStore[index] = value;
            
            var pageNumber = index / _pageSize;
            if (_loadedPages.ContainsKey(pageNumber))
            {
                var pageIndex = index % _pageSize;
                var page = _loadedPages[pageNumber];
                if (pageIndex < page.Count)
                {
                    page[pageIndex] = value;
                }
            }
            
            CollectionChanged?.Invoke(this, new NotifyCollectionChangedEventArgs(
                NotifyCollectionChangedAction.Replace, value, index));
        }
    }

    public void Add(T item)
    {
        _backingStore.Add(item);
        CollectionChanged?.Invoke(this, new NotifyCollectionChangedEventArgs(
            NotifyCollectionChangedAction.Add, item, _backingStore.Count - 1));
    }

    public void Clear()
    {
        _backingStore.Clear();
        _loadedPages.Clear();
        CollectionChanged?.Invoke(this, new NotifyCollectionChangedEventArgs(
            NotifyCollectionChangedAction.Reset));
    }

    public bool Contains(T item) => _backingStore.Contains(item);
    public void CopyTo(T[] array, int arrayIndex) => _backingStore.CopyTo(array, arrayIndex);
    public IEnumerator<T> GetEnumerator() => _backingStore.GetEnumerator();
    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
    public int IndexOf(T item) => _backingStore.IndexOf(item);
    
    public void Insert(int index, T item)
    {
        _backingStore.Insert(index, item);
        _loadedPages.Clear(); // Clear cache as indices changed
        CollectionChanged?.Invoke(this, new NotifyCollectionChangedEventArgs(
            NotifyCollectionChangedAction.Add, item, index));
    }

    public bool Remove(T item)
    {
        var index = _backingStore.IndexOf(item);
        if (index >= 0)
        {
            RemoveAt(index);
            return true;
        }
        return false;
    }

    public void RemoveAt(int index)
    {
        var item = _backingStore[index];
        _backingStore.RemoveAt(index);
        _loadedPages.Clear(); // Clear cache as indices changed
        CollectionChanged?.Invoke(this, new NotifyCollectionChangedEventArgs(
            NotifyCollectionChangedAction.Remove, item, index));
    }

    public void UnloadAllPages()
    {
        _loadedPages.Clear();
        GC.Collect(); // Force garbage collection
    }
}
```

---

## 2. ⚡ UI Performance Optimization

### 2.1 CollectionView Virtualization

```xml
<!-- d:\A-Work\learn from ai\maui-xaml-mastery\src\14-Performance\Views\OptimizedListView.xaml -->
<ContentPage x:Class="Performance.Views.OptimizedListView"
             xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:performance="clr-namespace:Performance.ViewModels"
             Title="Optimized List">

    <ContentPage.Resources>
        <ResourceDictionary>
            <!-- Optimized DataTemplate with minimal binding -->
            <DataTemplate x:Key="OptimizedItemTemplate">
                <Grid Padding="16,8" RowDefinitions="Auto,Auto">
                    <!-- Use simple binding paths -->
                    <Label Grid.Row="0" 
                           Text="{Binding Title}"
                           FontSize="16"
                           FontAttributes="Bold"
                           LineBreakMode="TailTruncation" />
                    
                    <Label Grid.Row="1"
                           Text="{Binding Subtitle}"
                           FontSize="14"
                           TextColor="{AppThemeBinding Light={StaticResource Gray600}, Dark={StaticResource Gray300}}"
                           LineBreakMode="TailTruncation" />
                </Grid>
            </DataTemplate>

            <!-- Lightweight header template -->
            <DataTemplate x:Key="GroupHeaderTemplate">
                <Grid BackgroundColor="{AppThemeBinding Light={StaticResource Gray100}, Dark={StaticResource Gray800}}"
                      Padding="16,8">
                    <Label Text="{Binding Name}"
                           FontSize="18"
                           FontAttributes="Bold" />
                </Grid>
            </DataTemplate>
        </ResourceDictionary>
    </ContentPage.Resources>

    <Grid RowDefinitions="Auto,*,Auto">
        <!-- Performance indicators -->
        <StackLayout Grid.Row="0" 
                     Orientation="Horizontal" 
                     Padding="16,8"
                     BackgroundColor="{AppThemeBinding Light={StaticResource Gray50}, Dark={StaticResource Gray900}}">
            <Label Text="{Binding MemoryUsage, StringFormat='Memory: {0:F1}MB'}"
                   FontSize="12" />
            <Label Text="{Binding FrameRate, StringFormat='FPS: {0}'}"
                   FontSize="12"
                   Margin="16,0,0,0" />
            <Label Text="{Binding LoadedItems, StringFormat='Loaded: {0}'}"
                   FontSize="12"
                   Margin="16,0,0,0" />
        </StackLayout>

        <!-- Optimized CollectionView -->
        <CollectionView Grid.Row="1"
                        ItemsSource="{Binding Items}"
                        ItemTemplate="{StaticResource OptimizedItemTemplate}"
                        RemainingItemsThreshold="10"
                        RemainingItemsThresholdReachedCommand="{Binding LoadMoreCommand}"
                        SelectionMode="None"
                        x:Name="MainCollectionView">
            
            <!-- Performance optimizations -->
            <CollectionView.ItemsLayout>
                <LinearItemsLayout Orientation="Vertical" 
                                   ItemSpacing="1" />
            </CollectionView.ItemsLayout>
            
            <!-- Grouping for better organization -->
            <CollectionView.GroupHeaderTemplate>
                <DataTemplate>
                    <Grid BackgroundColor="{AppThemeBinding Light={StaticResource Primary100}, Dark={StaticResource Primary900}}"
                          Padding="16,12">
                        <Label Text="{Binding Name}"
                               FontSize="16"
                               FontAttributes="Bold"
                               TextColor="{AppThemeBinding Light={StaticResource Primary}, Dark={StaticResource PrimaryDark}}" />
                    </Grid>
                </DataTemplate>
            </CollectionView.GroupHeaderTemplate>

            <!-- Empty view for better UX -->
            <CollectionView.EmptyView>
                <StackLayout HorizontalOptions="Center" VerticalOptions="Center">
                    <Label Text="No items to display"
                           FontSize="18"
                           HorizontalOptions="Center" />
                    <Button Text="Reload"
                            Command="{Binding ReloadCommand}"
                            HorizontalOptions="Center"
                            Margin="0,16,0,0" />
                </StackLayout>
            </CollectionView.EmptyView>
        </CollectionView>

        <!-- Performance controls -->
        <Grid Grid.Row="2" 
              ColumnDefinitions="*,*,*" 
              Padding="16,8"
              BackgroundColor="{AppThemeBinding Light={StaticResource Gray50}, Dark={StaticResource Gray900}}">
            
            <Button Grid.Column="0"
                    Text="GC Collect"
                    Command="{Binding ForceGCCommand}"
                    FontSize="12" />
            
            <Button Grid.Column="1"
                    Text="Clear Cache"
                    Command="{Binding ClearCacheCommand}"
                    FontSize="12" />
            
            <Button Grid.Column="2"
                    Text="Profile"
                    Command="{Binding ProfileCommand}"
                    FontSize="12" />
        </Grid>
    </Grid>

</ContentPage>
```

### 2.2 Optimized ViewModel

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\14-Performance\ViewModels\OptimizedListViewModel.cs
using System.Collections.ObjectModel;

namespace Performance.ViewModels;

public partial class OptimizedListViewModel : BaseViewModel
{
    private readonly IMemoryProfiler _memoryProfiler;
    private readonly IDataService _dataService;
    private readonly Timer _performanceTimer;

    [ObservableProperty]
    private ObservableCollection<ItemGroup> items;

    [ObservableProperty]
    private bool isLoading;

    [ObservableProperty]
    private double memoryUsage;

    [ObservableProperty]
    private int frameRate;

    [ObservableProperty]
    private int loadedItems;

    private int _currentPage = 0;
    private const int PAGE_SIZE = 50;
    private DateTime _lastFrameTime = DateTime.UtcNow;
    private int _frameCount = 0;

    public OptimizedListViewModel(IMemoryProfiler memoryProfiler, IDataService dataService)
    {
        _memoryProfiler = memoryProfiler;
        _dataService = dataService;
        
        Items = new ObservableCollection<ItemGroup>();
        
        // Start performance monitoring
        _performanceTimer = new Timer(UpdatePerformanceMetrics, null, 
            TimeSpan.Zero, TimeSpan.FromSeconds(1));
        
        // Subscribe to memory warnings
        _memoryProfiler.MemoryWarning += OnMemoryWarning;
        
        // Initial data load
        _ = LoadInitialDataAsync();
    }

    private async Task LoadInitialDataAsync()
    {
        try
        {
            IsLoading = true;
            
            var data = await _dataService.GetItemsAsync(0, PAGE_SIZE);
            var groups = GroupItems(data);
            
            Items.Clear();
            foreach (var group in groups)
            {
                Items.Add(group);
            }
            
            LoadedItems = data.Count;
            _currentPage = 1;
        }
        catch (Exception ex)
        {
            await ShowErrorAsync($"Failed to load data: {ex.Message}");
        }
        finally
        {
            IsLoading = false;
        }
    }

    [RelayCommand]
    private async Task LoadMoreAsync()
    {
        if (IsLoading) return;

        try
        {
            IsLoading = true;
            
            var data = await _dataService.GetItemsAsync(_currentPage * PAGE_SIZE, PAGE_SIZE);
            if (!data.Any()) return;
            
            var groups = GroupItems(data);
            
            // Efficiently merge new groups
            foreach (var newGroup in groups)
            {
                var existingGroup = Items.FirstOrDefault(g => g.Name == newGroup.Name);
                if (existingGroup != null)
                {
                    foreach (var item in newGroup)
                    {
                        existingGroup.Add(item);
                    }
                }
                else
                {
                    Items.Add(newGroup);
                }
            }
            
            LoadedItems += data.Count;
            _currentPage++;
        }
        catch (Exception ex)
        {
            await ShowErrorAsync($"Failed to load more data: {ex.Message}");
        }
        finally
        {
            IsLoading = false;
        }
    }

    [RelayCommand]
    private async Task ReloadAsync()
    {
        _currentPage = 0;
        LoadedItems = 0;
        await LoadInitialDataAsync();
    }

    [RelayCommand]
    private void ForceGC()
    {
        // Force garbage collection (use sparingly)
        GC.Collect();
        GC.WaitForPendingFinalizers();
        GC.Collect();
        
        MainThread.BeginInvokeOnMainThread(async () =>
        {
            await Application.Current.MainPage.DisplayAlert("GC", "Garbage collection completed", "OK");
        });
    }

    [RelayCommand]
    private async Task ClearCacheAsync()
    {
        // Clear any cached data
        Items.Clear();
        LoadedItems = 0;
        _currentPage = 0;
        
        // Force UI to release references
        await Task.Delay(100);
        
        // Reload fresh data
        await LoadInitialDataAsync();
    }

    [RelayCommand]
    private async Task ProfileAsync()
    {
        var snapshot = await _memoryProfiler.TakeSnapshotAsync();
        
        var details = $"""
            Memory Usage: {snapshot.WorkingSetMB} MB
            Allocated: {snapshot.TotalAllocatedMB} MB
            GC Gen0: {snapshot.Gen0Collections}
            GC Gen1: {snapshot.Gen1Collections}
            GC Gen2: {snapshot.Gen2Collections}
            """;
        
        await Application.Current.MainPage.DisplayAlert("Memory Profile", details, "OK");
    }

    private void UpdatePerformanceMetrics(object state)
    {
        MainThread.BeginInvokeOnMainThread(async () =>
        {
            // Update memory usage
            var snapshot = await _memoryProfiler.TakeSnapshotAsync();
            MemoryUsage = snapshot.WorkingSetMB;

            // Calculate frame rate (simplified)
            _frameCount++;
            var now = DateTime.UtcNow;
            var elapsed = now - _lastFrameTime;
            
            if (elapsed.TotalSeconds >= 1.0)
            {
                FrameRate = (int)(_frameCount / elapsed.TotalSeconds);
                _frameCount = 0;
                _lastFrameTime = now;
            }
        });
    }

    private void OnMemoryWarning(object sender, MemoryWarningEventArgs e)
    {
        MainThread.BeginInvokeOnMainThread(async () =>
        {
            switch (e.Level)
            {
                case MemoryWarningLevel.Warning:
                    System.Diagnostics.Debug.WriteLine($"Memory warning: {e.Message}");
                    // Consider reducing cache size
                    await OptimizeMemoryUsageAsync();
                    break;
                    
                case MemoryWarningLevel.Critical:
                    await Application.Current.MainPage.DisplayAlert(
                        "Memory Warning", 
                        "Application is using high memory. Consider closing and reopening.",
                        "OK");
                    
                    // Aggressive memory cleanup
                    await AggressiveMemoryCleanupAsync();
                    break;
            }
        });
    }

    private async Task OptimizeMemoryUsageAsync()
    {
        // Remove items beyond visible area
        const int maxItemsToKeep = 100;
        
        var totalItems = Items.SelectMany(g => g).Count();
        if (totalItems > maxItemsToKeep)
        {
            // Keep only recent items
            var itemsToRemove = totalItems - maxItemsToKeep;
            var removed = 0;
            
            for (int i = Items.Count - 1; i >= 0 && removed < itemsToRemove; i--)
            {
                var group = Items[i];
                var toRemoveFromGroup = Math.Min(group.Count, itemsToRemove - removed);
                
                for (int j = 0; j < toRemoveFromGroup; j++)
                {
                    group.RemoveAt(0);
                    removed++;
                }
                
                if (group.Count == 0)
                {
                    Items.RemoveAt(i);
                }
            }
            
            LoadedItems = Items.SelectMany(g => g).Count();
        }
    }

    private async Task AggressiveMemoryCleanupAsync()
    {
        // Clear most data, keep only essentials
        var essentialGroups = Items.Take(2).ToList();
        Items.Clear();
        
        foreach (var group in essentialGroups)
        {
            // Keep only first 10 items per group
            var itemsToKeep = group.Take(10).ToList();
            group.Clear();
            foreach (var item in itemsToKeep)
            {
                group.Add(item);
            }
            Items.Add(group);
        }
        
        LoadedItems = Items.SelectMany(g => g).Count();
        _currentPage = 1;
        
        // Force garbage collection
        GC.Collect();
        GC.WaitForPendingFinalizers();
        GC.Collect();
    }

    private List<ItemGroup> GroupItems(List<DataItem> items)
    {
        return items
            .GroupBy(item => item.Category)
            .Select(group => new ItemGroup(group.Key, group.ToList()))
            .ToList();
    }

    protected override void OnDispose()
    {
        _performanceTimer?.Dispose();
        _memoryProfiler.MemoryWarning -= OnMemoryWarning;
        base.OnDispose();
    }
}

public class ItemGroup : ObservableCollection<DataItem>
{
    public string Name { get; }

    public ItemGroup(string name, IEnumerable<DataItem> items) : base(items)
    {
        Name = name;
    }
}

public class DataItem
{
    public string Title { get; set; }
    public string Subtitle { get; set; }
    public string Category { get; set; }
    public DateTime CreatedAt { get; set; }
}
```

---

## 3. 🔧 Resource Management

### 3.1 Image Loading Optimization

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\14-Performance\Services\ImageCacheService.cs
using Microsoft.Extensions.Caching.Memory;

namespace Performance.Services;

public interface IImageCacheService
{
    Task<ImageSource> GetImageAsync(string url);
    Task<byte[]> GetImageDataAsync(string url);
    void ClearCache();
    Task PreloadImagesAsync(IEnumerable<string> urls);
    long GetCacheSize();
}

public class ImageCacheService : IImageCacheService
{
    private readonly HttpClient _httpClient;
    private readonly IMemoryCache _memoryCache;
    private readonly string _cacheDirectory;
    private readonly SemaphoreSlim _downloadSemaphore;
    
    private const int MAX_CONCURRENT_DOWNLOADS = 3;
    private const int CACHE_EXPIRY_HOURS = 24;
    private const long MAX_CACHE_SIZE_MB = 50;

    public ImageCacheService(HttpClient httpClient, IMemoryCache memoryCache)
    {
        _httpClient = httpClient;
        _memoryCache = memoryCache;
        _downloadSemaphore = new SemaphoreSlim(MAX_CONCURRENT_DOWNLOADS);
        
        _cacheDirectory = Path.Combine(FileSystem.CacheDirectory, "images");
        Directory.CreateDirectory(_cacheDirectory);
    }

    public async Task<ImageSource> GetImageAsync(string url)
    {
        if (string.IsNullOrEmpty(url))
            return null;

        var cacheKey = GetCacheKey(url);
        
        // Check memory cache first
        if (_memoryCache.TryGetValue(cacheKey, out ImageSource cachedImage))
        {
            return cachedImage;
        }

        // Check disk cache
        var cachedFilePath = GetCachedFilePath(url);
        if (File.Exists(cachedFilePath))
        {
            try
            {
                var imageSource = ImageSource.FromFile(cachedFilePath);
                
                // Add to memory cache with expiration
                var cacheOptions = new MemoryCacheEntryOptions
                {
                    AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(CACHE_EXPIRY_HOURS),
                    SlidingExpiration = TimeSpan.FromHours(1),
                    Priority = CacheItemPriority.Normal
                };
                
                _memoryCache.Set(cacheKey, imageSource, cacheOptions);
                return imageSource;
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine($"Failed to load cached image: {ex.Message}");
                // Delete corrupted cache file
                File.Delete(cachedFilePath);
            }
        }

        // Download and cache image
        return await DownloadAndCacheImageAsync(url);
    }

    public async Task<byte[]> GetImageDataAsync(string url)
    {
        if (string.IsNullOrEmpty(url))
            return null;

        var cachedFilePath = GetCachedFilePath(url);
        
        // Check disk cache
        if (File.Exists(cachedFilePath))
        {
            try
            {
                return await File.ReadAllBytesAsync(cachedFilePath);
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine($"Failed to read cached image data: {ex.Message}");
                File.Delete(cachedFilePath);
            }
        }

        // Download image data
        return await DownloadImageDataAsync(url);
    }

    private async Task<ImageSource> DownloadAndCacheImageAsync(string url)
    {
        await _downloadSemaphore.WaitAsync();
        
        try
        {
            var imageData = await DownloadImageDataAsync(url);
            if (imageData == null) return null;

            // Save to disk cache
            var cachedFilePath = GetCachedFilePath(url);
            await File.WriteAllBytesAsync(cachedFilePath, imageData);

            // Create ImageSource
            var imageSource = ImageSource.FromStream(() => new MemoryStream(imageData));
            
            // Add to memory cache
            var cacheKey = GetCacheKey(url);
            var cacheOptions = new MemoryCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(CACHE_EXPIRY_HOURS),
                SlidingExpiration = TimeSpan.FromHours(1),
                Priority = CacheItemPriority.Normal
            };
            
            _memoryCache.Set(cacheKey, imageSource, cacheOptions);
            
            return imageSource;
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"Failed to download image {url}: {ex.Message}");
            return null;
        }
        finally
        {
            _downloadSemaphore.Release();
        }
    }

    private async Task<byte[]> DownloadImageDataAsync(string url)
    {
        try
        {
            using var response = await _httpClient.GetAsync(url);
            response.EnsureSuccessStatusCode();
            
            var contentLength = response.Content.Headers.ContentLength;
            if (contentLength > 10 * 1024 * 1024) // 10MB limit
            {
                System.Diagnostics.Debug.WriteLine($"Image too large: {contentLength} bytes");
                return null;
            }

            return await response.Content.ReadAsByteArrayAsync();
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"Download failed for {url}: {ex.Message}");
            return null;
        }
    }

    public void ClearCache()
    {
        // Clear memory cache
        if (_memoryCache is MemoryCache mc)
        {
            mc.Compact(1.0); // Remove all entries
        }

        // Clear disk cache
        try
        {
            if (Directory.Exists(_cacheDirectory))
            {
                Directory.Delete(_cacheDirectory, true);
                Directory.CreateDirectory(_cacheDirectory);
            }
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"Failed to clear disk cache: {ex.Message}");
        }
    }

    public async Task PreloadImagesAsync(IEnumerable<string> urls)
    {
        var preloadTasks = urls.Select(async url =>
        {
            try
            {
                await GetImageAsync(url);
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine($"Preload failed for {url}: {ex.Message}");
            }
        });

        await Task.WhenAll(preloadTasks);
    }

    public long GetCacheSize()
    {
        try
        {
            if (!Directory.Exists(_cacheDirectory))
                return 0;

            var files = Directory.GetFiles(_cacheDirectory, "*", SearchOption.AllDirectories);
            return files.Sum(file => new FileInfo(file).Length);
        }
        catch
        {
            return 0;
        }
    }

    private string GetCacheKey(string url)
    {
        return $"image_{url.GetHashCode():X8}";
    }

    private string GetCachedFilePath(string url)
    {
        var fileName = $"{url.GetHashCode():X8}";
        return Path.Combine(_cacheDirectory, fileName);
    }

    // Cleanup old cache files periodically
    public async Task CleanupOldCacheAsync()
    {
        try
        {
            if (!Directory.Exists(_cacheDirectory))
                return;

            var cutoffDate = DateTime.Now.AddHours(-CACHE_EXPIRY_HOURS);
            var files = Directory.GetFiles(_cacheDirectory);
            
            foreach (var file in files)
            {
                var fileInfo = new FileInfo(file);
                if (fileInfo.LastAccessTime < cutoffDate)
                {
                    try
                    {
                        File.Delete(file);
                    }
                    catch (Exception ex)
                    {
                        System.Diagnostics.Debug.WriteLine($"Failed to delete old cache file {file}: {ex.Message}");
                    }
                }
            }

            // Check total cache size and remove oldest files if needed
            await EnforceCacheSizeLimitAsync();
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"Cache cleanup failed: {ex.Message}");
        }
    }

    private async Task EnforceCacheSizeLimitAsync()
    {
        var maxSizeBytes = MAX_CACHE_SIZE_MB * 1024 * 1024;
        var currentSize = GetCacheSize();
        
        if (currentSize <= maxSizeBytes)
            return;

        var files = Directory.GetFiles(_cacheDirectory)
            .Select(f => new FileInfo(f))
            .OrderBy(f => f.LastAccessTime)
            .ToList();

        var sizeToRemove = currentSize - maxSizeBytes;
        var removedSize = 0L;

        foreach (var file in files)
        {
            try
            {
                removedSize += file.Length;
                File.Delete(file.FullName);
                
                if (removedSize >= sizeToRemove)
                    break;
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine($"Failed to delete cache file {file.FullName}: {ex.Message}");
            }
        }
    }
}
```

---

## 4. 📊 Performance Analytics & Monitoring

### 4.1 Performance Analytics Service

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\14-Performance\Services\PerformanceAnalytics.cs
namespace Performance.Services;

public interface IPerformanceAnalytics
{
    void TrackPageLoad(string pageName, TimeSpan loadTime);
    void TrackUserAction(string action, Dictionary<string, object> properties = null);
    void TrackException(Exception exception, Dictionary<string, object> properties = null);
    void TrackPerformanceMetric(string metricName, double value, string unit = null);
    Task<PerformanceReport> GenerateReportAsync(TimeSpan period);
    void StartSession();
    void EndSession();
}

public class PerformanceAnalytics : IPerformanceAnalytics
{
    private readonly List<PerformanceEvent> _events = new();
    private readonly object _lock = new object();
    private DateTime _sessionStart;
    private readonly Timer _metricsCollector;

    public PerformanceAnalytics()
    {
        // Collect performance metrics every 30 seconds
        _metricsCollector = new Timer(CollectSystemMetrics, null, 
            TimeSpan.Zero, TimeSpan.FromSeconds(30));
    }

    public void TrackPageLoad(string pageName, TimeSpan loadTime)
    {
        var performanceEvent = new PerformanceEvent
        {
            Type = PerformanceEventType.PageLoad,
            Name = pageName,
            Timestamp = DateTime.UtcNow,
            Duration = loadTime,
            Properties = new Dictionary<string, object>
            {
                ["LoadTimeMs"] = loadTime.TotalMilliseconds,
                ["LoadTimeCategory"] = GetLoadTimeCategory(loadTime)
            }
        };

        AddEvent(performanceEvent);
        
        // Log slow page loads
        if (loadTime.TotalSeconds > 2)
        {
            System.Diagnostics.Debug.WriteLine($"Slow page load detected: {pageName} took {loadTime.TotalSeconds:F2}s");
        }
    }

    public void TrackUserAction(string action, Dictionary<string, object> properties = null)
    {
        var performanceEvent = new PerformanceEvent
        {
            Type = PerformanceEventType.UserAction,
            Name = action,
            Timestamp = DateTime.UtcNow,
            Properties = properties ?? new Dictionary<string, object>()
        };

        AddEvent(performanceEvent);
    }

    public void TrackException(Exception exception, Dictionary<string, object> properties = null)
    {
        var performanceEvent = new PerformanceEvent
        {
            Type = PerformanceEventType.Exception,
            Name = exception.GetType().Name,
            Timestamp = DateTime.UtcNow,
            Properties = properties ?? new Dictionary<string, object>()
        };

        performanceEvent.Properties["ExceptionMessage"] = exception.Message;
        performanceEvent.Properties["StackTrace"] = exception.StackTrace;
        performanceEvent.Properties["Source"] = exception.Source;

        AddEvent(performanceEvent);
    }

    public void TrackPerformanceMetric(string metricName, double value, string unit = null)
    {
        var performanceEvent = new PerformanceEvent
        {
            Type = PerformanceEventType.Metric,
            Name = metricName,
            Timestamp = DateTime.UtcNow,
            Value = value,
            Properties = new Dictionary<string, object>
            {
                ["Unit"] = unit ?? "count",
                ["Value"] = value
            }
        };

        AddEvent(performanceEvent);
    }

    public async Task<PerformanceReport> GenerateReportAsync(TimeSpan period)
    {
        var cutoffTime = DateTime.UtcNow - period;
        
        List<PerformanceEvent> relevantEvents;
        lock (_lock)
        {
            relevantEvents = _events.Where(e => e.Timestamp >= cutoffTime).ToList();
        }

        var report = new PerformanceReport
        {
            Period = period,
            GeneratedAt = DateTime.UtcNow,
            TotalEvents = relevantEvents.Count
        };

        // Page load analysis
        var pageLoads = relevantEvents.Where(e => e.Type == PerformanceEventType.PageLoad).ToList();
        if (pageLoads.Any())
        {
            report.AveragePageLoadTime = TimeSpan.FromMilliseconds(
                pageLoads.Average(e => e.Duration?.TotalMilliseconds ?? 0));
            
            report.SlowestPageLoad = pageLoads
                .OrderByDescending(e => e.Duration)
                .First();
        }

        // Memory analysis
        var memoryMetrics = relevantEvents.Where(e => e.Name == "MemoryUsage").ToList();
        if (memoryMetrics.Any())
        {
            report.AverageMemoryUsage = memoryMetrics.Average(e => e.Value);
            report.PeakMemoryUsage = memoryMetrics.Max(e => e.Value);
        }

        // Exception analysis
        var exceptions = relevantEvents.Where(e => e.Type == PerformanceEventType.Exception).ToList();
        report.ExceptionCount = exceptions.Count;
        report.TopExceptions = exceptions
            .GroupBy(e => e.Name)
            .OrderByDescending(g => g.Count())
            .Take(5)
            .ToDictionary(g => g.Key, g => g.Count());

        // User action analysis
        var userActions = relevantEvents.Where(e => e.Type == PerformanceEventType.UserAction).ToList();
        report.UserActionCount = userActions.Count;
        report.TopUserActions = userActions
            .GroupBy(e => e.Name)
            .OrderByDescending(g => g.Count())
            .Take(10)
            .ToDictionary(g => g.Key, g => g.Count());

        return report;
    }

    public void StartSession()
    {
        _sessionStart = DateTime.UtcNow;
        TrackUserAction("SessionStart");
    }

    public void EndSession()
    {
        var sessionDuration = DateTime.UtcNow - _sessionStart;
        TrackUserAction("SessionEnd", new Dictionary<string, object>
        {
            ["SessionDurationMinutes"] = sessionDuration.TotalMinutes
        });
    }

    private void AddEvent(PerformanceEvent performanceEvent)
    {
        lock (_lock)
        {
            _events.Add(performanceEvent);
            
            // Keep only last 1000 events to prevent memory issues
            if (_events.Count > 1000)
            {
                _events.RemoveAt(0);
            }
        }
    }

    private void CollectSystemMetrics(object state)
    {
        try
        {
            // Collect memory metrics
            var memoryUsage = GC.GetTotalMemory(false) / (1024.0 * 1024.0); // MB
            TrackPerformanceMetric("MemoryUsage", memoryUsage, "MB");

            // Collect GC metrics
            var gen0Collections = GC.CollectionCount(0);
            var gen1Collections = GC.CollectionCount(1);
            var gen2Collections = GC.CollectionCount(2);
            
            TrackPerformanceMetric("GCGen0Collections", gen0Collections);
            TrackPerformanceMetric("GCGen1Collections", gen1Collections);
            TrackPerformanceMetric("GCGen2Collections", gen2Collections);

            // Platform-specific metrics
#if ANDROID
            CollectAndroidMetrics();
#elif IOS
            CollectIOSMetrics();
#elif WINDOWS
            CollectWindowsMetrics();
#endif
        }
        catch (Exception ex)
        {
            TrackException(ex);
        }
    }

#if ANDROID
    private void CollectAndroidMetrics()
    {
        try
        {
            var context = Platform.CurrentActivity ?? Android.App.Application.Context;
            var activityManager = context.GetSystemService(Context.ActivityService) as Android.App.ActivityManager;
            
            if (activityManager != null)
            {
                var memoryInfo = new Android.App.ActivityManager.MemoryInfo();
                activityManager.GetMemoryInfo(memoryInfo);
                
                TrackPerformanceMetric("AndroidAvailableMemory", memoryInfo.AvailMem / (1024.0 * 1024.0), "MB");
                TrackPerformanceMetric("AndroidTotalMemory", memoryInfo.TotalMem / (1024.0 * 1024.0), "MB");
                TrackPerformanceMetric("AndroidLowMemory", memoryInfo.LowMemory ? 1 : 0);
            }
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"Failed to collect Android metrics: {ex.Message}");
        }
    }
#endif

    private string GetLoadTimeCategory(TimeSpan loadTime)
    {
        var seconds = loadTime.TotalSeconds;
        return seconds switch
        {
            < 0.5 => "Excellent",
            < 1.0 => "Good",
            < 2.0 => "Fair",
            _ => "Poor"
        };
    }
}

public class PerformanceEvent
{
    public PerformanceEventType Type { get; set; }
    public string Name { get; set; }
    public DateTime Timestamp { get; set; }
    public TimeSpan? Duration { get; set; }
    public double Value { get; set; }
    public Dictionary<string, object> Properties { get; set; } = new();
}

public enum PerformanceEventType
{
    PageLoad,
    UserAction,
    Exception,
    Metric
}

public class PerformanceReport
{
    public TimeSpan Period { get; set; }
    public DateTime GeneratedAt { get; set; }
    public int TotalEvents { get; set; }
    public TimeSpan AveragePageLoadTime { get; set; }
    public PerformanceEvent SlowestPageLoad { get; set; }
    public double AverageMemoryUsage { get; set; }
    public double PeakMemoryUsage { get; set; }
    public int ExceptionCount { get; set; }
    public Dictionary<string, int> TopExceptions { get; set; } = new();
    public int UserActionCount { get; set; }
    public Dictionary<string, int> TopUserActions { get; set; } = new();
}
```

---

## 5. 🧪 Practical Exercises

### Exercise 1: Memory Leak Detection
สร้าง memory leak detection system:
- WeakReference monitoring
- Object lifecycle tracking
- Automated leak reports
- Memory usage alerts

### Exercise 2: Performance Benchmarking
สร้าง comprehensive benchmarking suite:
- UI rendering performance tests
- Data operation benchmarks
- Network performance testing
- Cross-platform comparison

### Exercise 3: Real-time Performance Dashboard
สร้าง live performance monitoring:
- Real-time metrics display
- Performance alerts
- Historical trending
- Export capabilities

---

## 6. 📝 Performance Best Practices

### 🎯 UI Performance Tips

```csharp
// ✅ DO: Use lightweight ViewModels
public class LightweightViewModel : INotifyPropertyChanged
{
    private readonly Dictionary<string, object> _properties = new();
    
    protected T GetProperty<T>([CallerMemberName] string propertyName = null)
    {
        return _properties.TryGetValue(propertyName, out var value) ? (T)value : default(T);
    }
    
    protected bool SetProperty<T>(T value, [CallerMemberName] string propertyName = null)
    {
        if (!_properties.TryGetValue(propertyName, out var currentValue) || 
            !EqualityComparer<T>.Default.Equals((T)currentValue, value))
        {
            _properties[propertyName] = value;
            OnPropertyChanged(propertyName);
            return true;
        }
        return false;
    }
}

// ✅ DO: Implement efficient data binding
public class EfficientListItem
{
    // Use value types when possible
    public int Id { get; set; }
    public string Title { get; set; }
    
    // Avoid complex computed properties in UI binding
    public string DisplayText => $"{Id}: {Title}";
    
    // Cache expensive calculations
    private string _expensiveProperty;
    public string ExpensiveProperty
    {
        get
        {
            if (_expensiveProperty == null)
            {
                _expensiveProperty = CalculateExpensiveValue();
            }
            return _expensiveProperty;
        }
    }
}

// ✅ DO: Use resource dictionaries efficiently
<!-- Define styles once, use everywhere -->
<ResourceDictionary>
    <Style x:Key="BaseLabel" TargetType="Label">
        <Setter Property="FontSize" Value="14" />
        <Setter Property="TextColor" Value="{AppThemeBinding Light=Black, Dark=White}" />
    </Style>
    
    <Style x:Key="TitleLabel" TargetType="Label" BasedOn="{StaticResource BaseLabel}">
        <Setter Property="FontSize" Value="18" />
        <Setter Property="FontAttributes" Value="Bold" />
    </Style>
</ResourceDictionary>

// ❌ DON'T: Create objects in property getters
public string BadProperty => new StringBuilder().Append("Value").ToString(); // Creates objects on every access

// ✅ DO: Cache calculated values
private string _cachedProperty;
public string GoodProperty => _cachedProperty ??= CalculateValue();

// ✅ DO: Use efficient collection operations
var result = largeList
    .Where(x => x.IsActive)
    .Take(10) // Limit results early
    .ToList();

// ❌ DON'T: Use heavy operations on UI thread
await Task.Run(() =>
{
    // Heavy computation here
    var result = ProcessLargeDataSet();
    
    MainThread.BeginInvokeOnMainThread(() =>
    {
        // Update UI on main thread
        UpdateUI(result);
    });
});
```

### 🔧 Memory Management Tips

```csharp
// ✅ DO: Dispose resources properly
public class ResourceManager : IDisposable
{
    private readonly HttpClient _httpClient = new();
    private readonly Timer _timer;
    private bool _disposed = false;

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                _httpClient?.Dispose();
                _timer?.Dispose();
            }
            _disposed = true;
        }
    }
}

// ✅ DO: Use weak event subscriptions
public class WeakEventManager
{
    private readonly List<WeakReference> _subscribers = new();

    public void Subscribe(object subscriber)
    {
        _subscribers.Add(new WeakReference(subscriber));
    }

    public void NotifySubscribers<T>(T eventArgs)
    {
        for (int i = _subscribers.Count - 1; i >= 0; i--)
        {
            if (_subscribers[i].Target is IEventHandler<T> handler)
            {
                handler.Handle(eventArgs);
            }
            else
            {
                // Remove dead references
                _subscribers.RemoveAt(i);
            }
        }
    }
}

// ✅ DO: Use object pooling for frequently created objects
public static class Pools
{
    public static readonly ObjectPool<StringBuilder> StringBuilders = new(
        () => new StringBuilder(),
        sb => sb.Clear()
    );
}

// Usage:
var sb = Pools.StringBuilders.Get();
try
{
    sb.Append("Hello").Append(" World");
    return sb.ToString();
}
finally
{
    Pools.StringBuilders.Return(sb);
}
```

---

## 7. 🎯 Quiz & Assessment

### Quiz Questions

1. **Memory Management**: อธิบาย difference ระหว่าง memory leak และ memory pressure

2. **UI Performance**: เทคนิคไหนช่วยปรับปรุง CollectionView performance?

3. **Resource Management**: ทำไม IDisposable pattern สำคัญใน mobile apps?

4. **Performance Monitoring**: วิธีการ detect และ fix performance bottlenecks?

### Practical Assessment
สร้าง **"Performance Optimized App"** ที่มี:
- ✅ Memory monitoring และ optimization
- ✅ Efficient UI rendering
- ✅ Resource management
- ✅ Performance analytics
- ✅ Automated performance testing

---

## 8. 🚀 Next Steps

### Advanced Performance Topics
- **Native Platform Optimization**: Platform-specific performance tuning
- **Profiling Tools**: Advanced debugging และ profiling techniques
- **Load Testing**: Automated performance testing strategies
- **Performance CI/CD**: Continuous performance monitoring

### Chapter 15 Preview
หัวข้อต่อไป: **"Testing Strategies"**
- Unit testing best practices
- UI testing automation
- Performance testing
- Cross-platform testing strategies

---

## 📚 Additional Resources

### 🔗 Performance Tools
- [MAUI Performance Tips](https://docs.microsoft.com/dotnet/maui/fundamentals/performance)
- [.NET Memory Profilers](https://docs.microsoft.com/dotnet/core/diagnostics/)
- [Visual Studio Diagnostic Tools](https://docs.microsoft.com/visualstudio/profiling/)

### 📖 Recommended Reading
- ".NET Performance Optimization"
- "High Performance .NET Applications"
- "Mobile Performance Engineering"

---

*หมายเหตุ: บทนี้เน้นการสร้าง high-performance MAUI applications ด้วยเทคนิค advanced memory management และ optimization strategies*
