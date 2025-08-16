# 🧪 Chapter 18: Advanced Testing Strategies

## การทดสอบ MAUI Applications อย่างครอบคลุม

### 📋 Chapter Overview

ในบทนี้เราจะเรียนรู้การสร้าง comprehensive testing strategy สำหรับ .NET MAUI applications รวมถึง unit testing, integration testing, UI testing, และการใช้ modern testing frameworks และ patterns

### 🎯 Learning Objectives

- สร้าง unit tests สำหรับ ViewModels และ Services
- ใช้ UI testing frameworks เช่น Appium และ Xamarin.UITest
- ทำ integration testing สำหรับ APIs และ databases
- ใช้ mocking และ dependency injection ใน tests
- สร้าง automated testing pipelines
- ทำ performance และ load testing

---

## 1. 🔬 Unit Testing Foundation

### 1.1 Test Project Setup

```xml
<!-- Tests/UnitTests/UnitTests.csproj -->
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <IsPackable>false</IsPackable>
    <IsTestProject>true</IsTestProject>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.8.0" />
    <PackageReference Include="xunit" Version="2.6.1" />
    <PackageReference Include="xunit.runner.visualstudio" Version="2.5.3" />
    <PackageReference Include="FluentAssertions" Version="6.12.0" />
    <PackageReference Include="NSubstitute" Version="5.1.0" />
    <PackageReference Include="Microsoft.Extensions.DependencyInjection" Version="8.0.0" />
    <PackageReference Include="Microsoft.Extensions.Logging.Testing" Version="8.0.0" />
    <PackageReference Include="AutoFixture" Version="4.18.0" />
    <PackageReference Include="AutoFixture.AutoNSubstitute" Version="4.18.0" />
    <PackageReference Include="coverlet.collector" Version="6.0.0" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\..\src\MyApp.Core\MyApp.Core.csproj" />
    <ProjectReference Include="..\..\src\MyApp.Services\MyApp.Services.csproj" />
  </ItemGroup>

</Project>
```

### 1.2 Base Test Classes

```csharp
// Tests/UnitTests/Base/BaseTestClass.cs
public abstract class BaseTestClass
{
    protected IFixture Fixture { get; }
    protected ITestOutputHelper Output { get; }

    protected BaseTestClass(ITestOutputHelper output)
    {
        Output = output;
        Fixture = new Fixture().Customize(new AutoNSubstituteCustomization());
        
        // Configure AutoFixture for common scenarios
        Fixture.Behaviors.OfType<ThrowingRecursionBehavior>().ToList()
            .ForEach(b => Fixture.Behaviors.Remove(b));
        Fixture.Behaviors.Add(new OmitOnRecursionBehavior());
        
        // Custom specimen builders
        Fixture.Register(() => DateTime.UtcNow.AddDays(Fixture.Create<int>() % 365));
        Fixture.Register(() => new CancellationTokenSource().Token);
    }

    protected T CreateMock<T>() where T : class
    {
        return Substitute.For<T>();
    }

    protected List<T> CreateMany<T>(int count = 3)
    {
        return Fixture.CreateMany<T>(count).ToList();
    }

    protected void AssertPropertyChanged<T>(T viewModel, string propertyName, Action action)
        where T : INotifyPropertyChanged
    {
        var propertyChangedRaised = false;
        viewModel.PropertyChanged += (sender, e) =>
        {
            if (e.PropertyName == propertyName)
                propertyChangedRaised = true;
        };

        action();

        propertyChangedRaised.Should().BeTrue($"PropertyChanged should be raised for {propertyName}");
    }
}

// Tests/UnitTests/Base/ViewModelTestBase.cs
public abstract class ViewModelTestBase<TViewModel> : BaseTestClass, IDisposable
    where TViewModel : BaseViewModel
{
    protected TViewModel ViewModel { get; }
    protected INavigationService NavigationService { get; }
    protected IDialogService DialogService { get; }
    protected ILogger<TViewModel> Logger { get; }

    protected ViewModelTestBase(ITestOutputHelper output) : base(output)
    {
        NavigationService = CreateMock<INavigationService>();
        DialogService = CreateMock<IDialogService>();
        Logger = CreateMock<ILogger<TViewModel>>();
        
        ViewModel = CreateViewModel();
    }

    protected abstract TViewModel CreateViewModel();

    protected async Task<bool> WaitForConditionAsync(Func<bool> condition, TimeSpan? timeout = null)
    {
        timeout ??= TimeSpan.FromSeconds(5);
        var cancellationToken = new CancellationTokenSource(timeout.Value).Token;

        while (!condition() && !cancellationToken.IsCancellationRequested)
        {
            await Task.Delay(50, cancellationToken);
        }

        return condition();
    }

    public virtual void Dispose()
    {
        ViewModel?.Dispose();
        GC.SuppressFinalize(this);
    }
}
```

### 1.3 Service Testing

```csharp
// Tests/UnitTests/Services/DataServiceTests.cs
public class DataServiceTests : BaseTestClass
{
    private readonly IHttpClientFactory _httpClientFactory;
    private readonly IMemoryCache _cache;
    private readonly ILogger<DataService> _logger;
    private readonly DataService _dataService;
    private readonly MockHttpMessageHandler _mockHttpHandler;

    public DataServiceTests(ITestOutputHelper output) : base(output)
    {
        _mockHttpHandler = new MockHttpMessageHandler();
        var httpClient = new HttpClient(_mockHttpHandler)
        {
            BaseAddress = new Uri("https://api.example.com/")
        };

        _httpClientFactory = CreateMock<IHttpClientFactory>();
        _httpClientFactory.CreateClient(Arg.Any<string>()).Returns(httpClient);

        _cache = new MemoryCache(new MemoryCacheOptions());
        _logger = CreateMock<ILogger<DataService>>();

        _dataService = new DataService(_httpClientFactory, _cache, _logger);
    }

    [Fact]
    public async Task LoadDataAsync_ShouldReturnData_WhenApiCallSucceeds()
    {
        // Arrange
        var expectedData = CreateMany<DataItem>();
        var jsonResponse = JsonSerializer.Serialize(expectedData);
        
        _mockHttpHandler.When("*")
            .Respond("application/json", jsonResponse);

        // Act
        var result = await _dataService.LoadDataAsync(1, 10);

        // Assert
        result.Should().NotBeNull();
        result.Should().HaveCount(expectedData.Count);
        result.Should().BeEquivalentTo(expectedData);
    }

    [Fact]
    public async Task LoadDataAsync_ShouldReturnCachedData_WhenCalledTwice()
    {
        // Arrange
        var expectedData = CreateMany<DataItem>();
        var jsonResponse = JsonSerializer.Serialize(expectedData);
        
        _mockHttpHandler.When("*")
            .Respond("application/json", jsonResponse);

        // Act
        var firstResult = await _dataService.LoadDataAsync(1, 10);
        var secondResult = await _dataService.LoadDataAsync(1, 10);

        // Assert
        firstResult.Should().BeEquivalentTo(secondResult);
        _mockHttpHandler.GetMatchedRequests().Should().HaveCount(1, "second call should use cache");
    }

    [Fact]
    public async Task LoadDataAsync_ShouldThrowException_WhenApiCallFails()
    {
        // Arrange
        _mockHttpHandler.When("*")
            .Respond(HttpStatusCode.InternalServerError);

        // Act & Assert
        var act = async () => await _dataService.LoadDataAsync(1, 10);
        
        await act.Should().ThrowAsync<HttpRequestException>()
            .WithMessage("*500*");
    }

    [Theory]
    [InlineData(0, 10, "page")]
    [InlineData(1, 0, "pageSize")]
    [InlineData(-1, 10, "page")]
    [InlineData(1, -5, "pageSize")]
    public async Task LoadDataAsync_ShouldThrowArgumentException_WhenParametersAreInvalid(
        int page, int pageSize, string expectedParameter)
    {
        // Act & Assert
        var act = async () => await _dataService.LoadDataAsync(page, pageSize);
        
        await act.Should().ThrowAsync<ArgumentException>()
            .And.ParamName.Should().Be(expectedParameter);
    }

    [Fact]
    public async Task LoadDataAsync_ShouldHandleCancellation_WhenTokenIsCancelled()
    {
        // Arrange
        var cts = new CancellationTokenSource();
        var delay = Task.Delay(100).ContinueWith(_ => cts.Cancel());

        _mockHttpHandler.When("*")
            .Respond(async () =>
            {
                await Task.Delay(1000, cts.Token);
                return new HttpResponseMessage(HttpStatusCode.OK);
            });

        // Act & Assert
        var act = async () => await _dataService.LoadDataAsync(1, 10, cts.Token);
        
        await act.Should().ThrowAsync<OperationCanceledException>();
    }

    public void Dispose()
    {
        _mockHttpHandler?.Dispose();
        _cache?.Dispose();
        _dataService?.Dispose();
    }
}

// Helper class for HTTP mocking
public class MockHttpMessageHandler : HttpMessageHandler
{
    private readonly List<(Func<HttpRequestMessage, bool> predicate, Func<Task<HttpResponseMessage>> response)> _handlers = new();
    private readonly List<HttpRequestMessage> _requests = new();

    public MockHttpMessageHandler When(string pattern)
    {
        var predicate = CreatePredicate(pattern);
        return When(predicate);
    }

    public MockHttpMessageHandler When(Func<HttpRequestMessage, bool> predicate)
    {
        _currentPredicate = predicate;
        return this;
    }

    public MockHttpMessageHandler Respond(HttpStatusCode statusCode)
    {
        return Respond(() => Task.FromResult(new HttpResponseMessage(statusCode)));
    }

    public MockHttpMessageHandler Respond(string mediaType, string content)
    {
        return Respond(() => Task.FromResult(new HttpResponseMessage(HttpStatusCode.OK)
        {
            Content = new StringContent(content, Encoding.UTF8, mediaType)
        }));
    }

    public MockHttpMessageHandler Respond(Func<Task<HttpResponseMessage>> responseFactory)
    {
        if (_currentPredicate != null)
        {
            _handlers.Add((_currentPredicate, responseFactory));
            _currentPredicate = null;
        }
        return this;
    }

    protected override async Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request, 
        CancellationToken cancellationToken)
    {
        _requests.Add(request);

        var handler = _handlers.FirstOrDefault(h => h.predicate(request));
        if (handler.response != null)
        {
            return await handler.response();
        }

        return new HttpResponseMessage(HttpStatusCode.NotFound);
    }

    public IReadOnlyList<HttpRequestMessage> GetMatchedRequests() => _requests.AsReadOnly();

    private Func<HttpRequestMessage, bool> CreatePredicate(string pattern)
    {
        if (pattern == "*")
            return _ => true;

        return request => request.RequestUri?.ToString().Contains(pattern, StringComparison.OrdinalIgnoreCase) == true;
    }

    private Func<HttpRequestMessage, bool>? _currentPredicate;
}
```

### 1.4 ViewModel Testing

```csharp
// Tests/UnitTests/ViewModels/TaskListViewModelTests.cs
public class TaskListViewModelTests : ViewModelTestBase<TaskListViewModel>
{
    private readonly ITaskService _taskService;
    private readonly IConnectivityService _connectivityService;

    public TaskListViewModelTests(ITestOutputHelper output) : base(output)
    {
        _taskService = CreateMock<ITaskService>();
        _connectivityService = CreateMock<IConnectivityService>();
        
        // Setup default connectivity
        _connectivityService.IsConnected.Returns(true);
    }

    protected override TaskListViewModel CreateViewModel()
    {
        return new TaskListViewModel(
            _taskService,
            _connectivityService,
            NavigationService,
            DialogService,
            Logger);
    }

    [Fact]
    public async Task LoadTasksCommand_ShouldLoadTasks_WhenExecuted()
    {
        // Arrange
        var expectedTasks = CreateMany<TaskItem>();
        _taskService.GetTasksAsync(Arg.Any<CancellationToken>())
            .Returns(expectedTasks);

        // Act
        await ViewModel.LoadTasksCommand.ExecuteAsync(null);

        // Assert
        ViewModel.Tasks.Should().HaveCount(expectedTasks.Count);
        ViewModel.IsLoading.Should().BeFalse();
        ViewModel.HasErrors.Should().BeFalse();
    }

    [Fact]
    public async Task LoadTasksCommand_ShouldSetError_WhenServiceThrowsException()
    {
        // Arrange
        var exception = new Exception("Service error");
        _taskService.GetTasksAsync(Arg.Any<CancellationToken>())
            .ThrowsAsync(exception);

        // Act
        await ViewModel.LoadTasksCommand.ExecuteAsync(null);

        // Assert
        ViewModel.Tasks.Should().BeEmpty();
        ViewModel.HasErrors.Should().BeTrue();
        ViewModel.ErrorMessage.Should().Contain("Service error");
    }

    [Fact]
    public async Task AddTaskCommand_ShouldAddNewTask_WhenExecuted()
    {
        // Arrange
        var newTask = Fixture.Create<TaskItem>();
        _taskService.CreateTaskAsync(Arg.Any<TaskItem>(), Arg.Any<CancellationToken>())
            .Returns(newTask);

        ViewModel.NewTaskTitle = newTask.Title;

        // Act
        await ViewModel.AddTaskCommand.ExecuteAsync(null);

        // Assert
        ViewModel.Tasks.Should().Contain(t => t.Title == newTask.Title);
        ViewModel.NewTaskTitle.Should().BeEmpty();
        await _taskService.Received(1).CreateTaskAsync(
            Arg.Is<TaskItem>(t => t.Title == newTask.Title),
            Arg.Any<CancellationToken>());
    }

    [Theory]
    [InlineData("")]
    [InlineData(" ")]
    [InlineData(null)]
    public void AddTaskCommand_ShouldBeDisabled_WhenTitleIsInvalid(string title)
    {
        // Arrange
        ViewModel.NewTaskTitle = title;

        // Act & Assert
        ViewModel.AddTaskCommand.CanExecute(null).Should().BeFalse();
    }

    [Fact]
    public async Task DeleteTaskCommand_ShouldRemoveTask_WhenExecuted()
    {
        // Arrange
        var tasks = CreateMany<TaskItem>();
        var taskToDelete = tasks.First();
        
        _taskService.GetTasksAsync(Arg.Any<CancellationToken>()).Returns(tasks);
        await ViewModel.LoadTasksCommand.ExecuteAsync(null);

        _taskService.DeleteTaskAsync(taskToDelete.Id, Arg.Any<CancellationToken>())
            .Returns(Task.CompletedTask);

        // Act
        await ViewModel.DeleteTaskCommand.ExecuteAsync(taskToDelete);

        // Assert
        ViewModel.Tasks.Should().NotContain(t => t.Id == taskToDelete.Id);
        await _taskService.Received(1).DeleteTaskAsync(taskToDelete.Id, Arg.Any<CancellationToken>());
    }

    [Fact]
    public void SelectedTask_ShouldNavigateToDetails_WhenSet()
    {
        // Arrange
        var task = Fixture.Create<TaskItem>();
        ViewModel.Tasks.Add(task);

        // Act
        ViewModel.SelectedTask = task;

        // Assert
        NavigationService.Received(1).NavigateToAsync(
            "TaskDetail",
            Arg.Is<Dictionary<string, object>>(d => d.ContainsKey("TaskId") && d["TaskId"].Equals(task.Id)));
    }

    [Fact]
    public async Task RefreshCommand_ShouldReloadTasks_WhenExecuted()
    {
        // Arrange
        var initialTasks = CreateMany<TaskItem>(3);
        var refreshedTasks = CreateMany<TaskItem>(5);

        _taskService.GetTasksAsync(Arg.Any<CancellationToken>())
            .Returns(initialTasks, refreshedTasks);

        await ViewModel.LoadTasksCommand.ExecuteAsync(null);
        ViewModel.Tasks.Should().HaveCount(3);

        // Act
        await ViewModel.RefreshCommand.ExecuteAsync(null);

        // Assert
        ViewModel.Tasks.Should().HaveCount(5);
        ViewModel.IsRefreshing.Should().BeFalse();
    }

    [Fact]
    public async Task PropertyChanged_ShouldBeRaised_ForIsLoadingProperty()
    {
        // Arrange
        var tasks = CreateMany<TaskItem>();
        _taskService.GetTasksAsync(Arg.Any<CancellationToken>()).Returns(tasks);

        // Act & Assert
        AssertPropertyChanged(ViewModel, nameof(ViewModel.IsLoading), async () =>
        {
            await ViewModel.LoadTasksCommand.ExecuteAsync(null);
        });
    }

    [Fact]
    public void ShouldShowOfflineMessage_WhenNotConnected()
    {
        // Arrange
        _connectivityService.IsConnected.Returns(false);

        // Act
        var viewModel = CreateViewModel();

        // Assert
        viewModel.ShowOfflineMessage.Should().BeTrue();
    }
}
```

---

## 2. 🖱️ UI Testing with Appium

### 2.1 Appium Test Setup

```csharp
// Tests/UITests/Base/AppiumTestBase.cs
[Collection("Appium")]
public abstract class AppiumTestBase : IDisposable
{
    protected AppiumDriver Driver { get; }
    protected TestSettings Settings { get; }

    protected AppiumTestBase()
    {
        Settings = TestSettings.Load();
        Driver = CreateDriver();
    }

    private AppiumDriver CreateDriver()
    {
        var options = Settings.Platform.ToLowerInvariant() switch
        {
            "android" => CreateAndroidOptions(),
            "ios" => CreateiOSOptions(),
            _ => throw new ArgumentException($"Unsupported platform: {Settings.Platform}")
        };

        var driver = new AppiumDriver(new Uri(Settings.AppiumServerUrl), options);
        driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(10);
        
        return driver;
    }

    private AppiumOptions CreateAndroidOptions()
    {
        var options = new AppiumOptions();
        options.AddAdditionalAppiumOption("platformName", "Android");
        options.AddAdditionalAppiumOption("deviceName", Settings.DeviceName);
        options.AddAdditionalAppiumOption("app", Settings.AppPath);
        options.AddAdditionalAppiumOption("automationName", "UiAutomator2");
        options.AddAdditionalAppiumOption("newCommandTimeout", 300);
        options.AddAdditionalAppiumOption("autoGrantPermissions", true);
        
        // Performance optimizations
        options.AddAdditionalAppiumOption("skipDeviceInitialization", true);
        options.AddAdditionalAppiumOption("skipServerInstallation", true);
        options.AddAdditionalAppiumOption("noReset", false);
        
        return options;
    }

    private AppiumOptions CreateiOSOptions()
    {
        var options = new AppiumOptions();
        options.AddAdditionalAppiumOption("platformName", "iOS");
        options.AddAdditionalAppiumOption("deviceName", Settings.DeviceName);
        options.AddAdditionalAppiumOption("app", Settings.AppPath);
        options.AddAdditionalAppiumOption("automationName", "XCUITest");
        options.AddAdditionalAppiumOption("newCommandTimeout", 300);
        options.AddAdditionalAppiumOption("wdaLaunchTimeout", 60000);
        
        return options;
    }

    protected void WaitForElement(By locator, TimeSpan? timeout = null)
    {
        timeout ??= TimeSpan.FromSeconds(30);
        var wait = new WebDriverWait(Driver, timeout.Value);
        wait.Until(driver => driver.FindElement(locator).Displayed);
    }

    protected void WaitForElementToDisappear(By locator, TimeSpan? timeout = null)
    {
        timeout ??= TimeSpan.FromSeconds(10);
        var wait = new WebDriverWait(Driver, timeout.Value);
        wait.Until(driver => 
        {
            try
            {
                var element = driver.FindElement(locator);
                return !element.Displayed;
            }
            catch (NoSuchElementException)
            {
                return true;
            }
        });
    }

    protected void TakeScreenshot(string fileName)
    {
        try
        {
            var screenshot = Driver.GetScreenshot();
            var filePath = Path.Combine(Settings.ScreenshotDirectory, $"{fileName}.png");
            screenshot.SaveAsFile(filePath);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Failed to take screenshot: {ex.Message}");
        }
    }

    protected void ScrollToElement(IWebElement element)
    {
        var actions = new Actions(Driver);
        actions.MoveToElement(element).Perform();
    }

    public virtual void Dispose()
    {
        try
        {
            Driver?.Quit();
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error disposing driver: {ex.Message}");
        }
        
        GC.SuppressFinalize(this);
    }
}

// Tests/UITests/Configuration/TestSettings.cs
public class TestSettings
{
    public string Platform { get; set; } = "Android";
    public string DeviceName { get; set; } = "emulator-5554";
    public string AppPath { get; set; } = "";
    public string AppiumServerUrl { get; set; } = "http://127.0.0.1:4723/wd/hub";
    public string ScreenshotDirectory { get; set; } = "Screenshots";
    public int DefaultTimeoutSeconds { get; set; } = 30;

    public static TestSettings Load()
    {
        var settingsPath = Path.Combine(
            AppDomain.CurrentDomain.BaseDirectory, 
            "testsettings.json");

        if (File.Exists(settingsPath))
        {
            var json = File.ReadAllText(settingsPath);
            return JsonSerializer.Deserialize<TestSettings>(json) ?? new TestSettings();
        }

        return new TestSettings();
    }
}
```

### 2.2 Page Object Model

```csharp
// Tests/UITests/PageObjects/TaskListPage.cs
public class TaskListPage : BasePage
{
    private readonly By _addTaskButton = By.Id("AddTaskButton");
    private readonly By _newTaskEntry = By.Id("NewTaskEntry");
    private readonly By _tasksList = By.Id("TasksList");
    private readonly By _refreshButton = By.Id("RefreshButton");
    private readonly By _loadingIndicator = By.Id("LoadingIndicator");
    private readonly By _emptyStateMessage = By.Id("EmptyStateMessage");

    public TaskListPage(AppiumDriver driver) : base(driver) { }

    public void AddTask(string taskTitle)
    {
        WaitForPageToLoad();
        
        var newTaskEntry = Driver.FindElement(_newTaskEntry);
        newTaskEntry.Clear();
        newTaskEntry.SendKeys(taskTitle);
        
        var addButton = Driver.FindElement(_addTaskButton);
        addButton.Click();
        
        WaitForTaskToAppear(taskTitle);
    }

    public void DeleteTask(int taskIndex)
    {
        WaitForPageToLoad();
        
        var tasks = GetTaskElements();
        if (taskIndex >= 0 && taskIndex < tasks.Count)
        {
            var task = tasks[taskIndex];
            var deleteButton = task.FindElement(By.Id("DeleteButton"));
            deleteButton.Click();
            
            // Wait for confirmation dialog and confirm
            var confirmButton = Driver.FindElement(By.Id("ConfirmDeleteButton"));
            confirmButton.Click();
        }
    }

    public void ToggleTask(int taskIndex)
    {
        WaitForPageToLoad();
        
        var tasks = GetTaskElements();
        if (taskIndex >= 0 && taskIndex < tasks.Count)
        {
            var task = tasks[taskIndex];
            var checkbox = task.FindElement(By.Id("TaskCheckbox"));
            checkbox.Click();
        }
    }

    public bool IsTaskCompleted(int taskIndex)
    {
        var tasks = GetTaskElements();
        if (taskIndex >= 0 && taskIndex < tasks.Count)
        {
            var task = tasks[taskIndex];
            var checkbox = task.FindElement(By.Id("TaskCheckbox"));
            return checkbox.GetAttribute("checked") == "true";
        }
        return false;
    }

    public int GetTaskCount()
    {
        WaitForPageToLoad();
        return GetTaskElements().Count;
    }

    public bool VerifyTaskExists(string taskTitle)
    {
        try
        {
            var taskLocator = By.XPath($"//android.widget.TextView[@text='{taskTitle}']");
            WaitForElement(taskLocator, TimeSpan.FromSeconds(5));
            return true;
        }
        catch (WebDriverTimeoutException)
        {
            return false;
        }
    }

    public bool VerifyTaskDoesNotExist(string taskTitle)
    {
        try
        {
            var taskLocator = By.XPath($"//android.widget.TextView[@text='{taskTitle}']");
            var element = Driver.FindElement(taskLocator);
            return !element.Displayed;
        }
        catch (NoSuchElementException)
        {
            return true;
        }
    }

    public void RefreshTasks()
    {
        WaitForPageToLoad();
        
        var refreshButton = Driver.FindElement(_refreshButton);
        refreshButton.Click();
        
        WaitForLoadingToComplete();
    }

    public void WaitForPageToLoad()
    {
        try
        {
            WaitForElementToDisappear(_loadingIndicator, TimeSpan.FromSeconds(10));
            WaitForElement(_tasksList, TimeSpan.FromSeconds(5));
        }
        catch (WebDriverTimeoutException)
        {
            // Page might be loaded but empty
            WaitForElement(_emptyStateMessage, TimeSpan.FromSeconds(5));
        }
    }

    public bool IsEmptyStateShown()
    {
        try
        {
            var emptyMessage = Driver.FindElement(_emptyStateMessage);
            return emptyMessage.Displayed;
        }
        catch (NoSuchElementException)
        {
            return false;
        }
    }

    private void WaitForTaskToAppear(string taskTitle)
    {
        var taskLocator = By.XPath($"//android.widget.TextView[@text='{taskTitle}']");
        WaitForElement(taskLocator, TimeSpan.FromSeconds(10));
    }

    private void WaitForLoadingToComplete()
    {
        try
        {
            WaitForElementToDisappear(_loadingIndicator, TimeSpan.FromSeconds(30));
        }
        catch (WebDriverTimeoutException)
        {
            // Loading might have completed before we started waiting
        }
    }

    private List<IWebElement> GetTaskElements()
    {
        try
        {
            var tasksList = Driver.FindElement(_tasksList);
            return tasksList.FindElements(By.ClassName("TaskItem")).ToList();
        }
        catch (NoSuchElementException)
        {
            return new List<IWebElement>();
        }
    }
}

// Tests/UITests/PageObjects/BasePage.cs
public abstract class BasePage
{
    protected AppiumDriver Driver { get; }

    protected BasePage(AppiumDriver driver)
    {
        Driver = driver;
    }

    protected void WaitForElement(By locator, TimeSpan? timeout = null)
    {
        timeout ??= TimeSpan.FromSeconds(10);
        var wait = new WebDriverWait(Driver, timeout.Value);
        wait.Until(driver => driver.FindElement(locator).Displayed);
    }

    protected void WaitForElementToDisappear(By locator, TimeSpan? timeout = null)
    {
        timeout ??= TimeSpan.FromSeconds(10);
        var wait = new WebDriverWait(Driver, timeout.Value);
        wait.Until(driver =>
        {
            try
            {
                var element = driver.FindElement(locator);
                return !element.Displayed;
            }
            catch (NoSuchElementException)
            {
                return true;
            }
        });
    }

    protected void TakeScreenshot(string fileName)
    {
        try
        {
            var screenshot = Driver.GetScreenshot();
            var directory = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "Screenshots");
            Directory.CreateDirectory(directory);
            
            var filePath = Path.Combine(directory, $"{fileName}_{DateTime.Now:yyyyMMdd_HHmmss}.png");
            screenshot.SaveAsFile(filePath);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Failed to take screenshot: {ex.Message}");
        }
    }

    protected void ScrollToElement(IWebElement element)
    {
        var actions = new Actions(Driver);
        actions.MoveToElement(element).Perform();
    }

    protected void SwipeUp()
    {
        var size = Driver.Manage().Window.Size;
        var startX = size.Width / 2;
        var startY = (int)(size.Height * 0.8);
        var endY = (int)(size.Height * 0.2);

        var touchAction = new TouchAction(Driver);
        touchAction.Press(startX, startY)
                  .MoveTo(startX, endY)
                  .Release()
                  .Perform();
    }

    protected void SwipeDown()
    {
        var size = Driver.Manage().Window.Size;
        var startX = size.Width / 2;
        var startY = (int)(size.Height * 0.2);
        var endY = (int)(size.Height * 0.8);

        var touchAction = new TouchAction(Driver);
        touchAction.Press(startX, startY)
                  .MoveTo(startX, endY)
                  .Release()
                  .Perform();
    }
}
```

### 2.3 UI Test Cases

```csharp
// Tests/UITests/Tests/TaskListUITests.cs
[Collection("UI Tests")]
public class TaskListUITests : AppiumTestBase
{
    private TaskListPage _taskListPage;

    public TaskListUITests()
    {
        _taskListPage = new TaskListPage(Driver);
    }

    [Fact]
    [Trait("Category", "Smoke")]
    public void TaskListPage_ShouldLoad_Successfully()
    {
        // Act
        _taskListPage.WaitForPageToLoad();

        // Assert
        var isLoaded = _taskListPage.IsEmptyStateShown() || _taskListPage.GetTaskCount() >= 0;
        isLoaded.Should().BeTrue("Task list page should load successfully");
        
        TakeScreenshot("task_list_loaded");
    }

    [Fact]
    [Trait("Category", "CRUD")]
    public void AddTask_ShouldCreateNewTask_WhenValidTitleProvided()
    {
        // Arrange
        var taskTitle = $"UI Test Task {DateTime.Now.Ticks}";
        _taskListPage.WaitForPageToLoad();
        var initialCount = _taskListPage.GetTaskCount();

        try
        {
            // Act
            _taskListPage.AddTask(taskTitle);

            // Assert
            _taskListPage.GetTaskCount().Should().Be(initialCount + 1);
            _taskListPage.VerifyTaskExists(taskTitle).Should().BeTrue();
            
            TakeScreenshot("task_added_successfully");
        }
        catch (Exception ex)
        {
            TakeScreenshot("add_task_failure");
            throw;
        }
    }

    [Fact]
    [Trait("Category", "CRUD")]
    public void DeleteTask_ShouldRemoveTask_WhenConfirmed()
    {
        // Arrange
        var taskTitle = $"Task to Delete {DateTime.Now.Ticks}";
        _taskListPage.WaitForPageToLoad();
        _taskListPage.AddTask(taskTitle);
        
        var taskCount = _taskListPage.GetTaskCount();
        var taskIndex = taskCount - 1; // Last added task

        try
        {
            // Act
            _taskListPage.DeleteTask(taskIndex);

            // Assert
            _taskListPage.GetTaskCount().Should().Be(taskCount - 1);
            _taskListPage.VerifyTaskDoesNotExist(taskTitle).Should().BeTrue();
            
            TakeScreenshot("task_deleted_successfully");
        }
        catch (Exception ex)
        {
            TakeScreenshot("delete_task_failure");
            throw;
        }
    }

    [Fact]
    [Trait("Category", "Functionality")]
    public void ToggleTask_ShouldChangeCompletionStatus_WhenClicked()
    {
        // Arrange
        var taskTitle = $"Toggle Task {DateTime.Now.Ticks}";
        _taskListPage.WaitForPageToLoad();
        _taskListPage.AddTask(taskTitle);
        
        var taskIndex = _taskListPage.GetTaskCount() - 1;
        var initialState = _taskListPage.IsTaskCompleted(taskIndex);

        try
        {
            // Act
            _taskListPage.ToggleTask(taskIndex);

            // Assert
            var newState = _taskListPage.IsTaskCompleted(taskIndex);
            newState.Should().Be(!initialState, "Task completion state should toggle");
            
            TakeScreenshot("task_toggled_successfully");
        }
        catch (Exception ex)
        {
            TakeScreenshot("toggle_task_failure");
            throw;
        }
    }

    [Fact]
    [Trait("Category", "Performance")]
    public void LoadManyTasks_ShouldPerformWell_WhenScrolling()
    {
        // Arrange
        _taskListPage.WaitForPageToLoad();
        
        var stopwatch = Stopwatch.StartNew();
        var tasksToAdd = 10;

        try
        {
            // Act - Add multiple tasks
            for (int i = 0; i < tasksToAdd; i++)
            {
                _taskListPage.AddTask($"Performance Test Task {i + 1}");
            }

            // Scroll through the list
            for (int i = 0; i < 3; i++)
            {
                _taskListPage.SwipeUp();
                Thread.Sleep(500); // Allow for smooth scrolling
            }

            stopwatch.Stop();

            // Assert
            _taskListPage.GetTaskCount().Should().BeGreaterOrEqualTo(tasksToAdd);
            stopwatch.ElapsedMilliseconds.Should().BeLessThan(30000, "Loading and scrolling should complete within 30 seconds");
            
            TakeScreenshot("performance_test_completed");
        }
        catch (Exception ex)
        {
            TakeScreenshot("performance_test_failure");
            throw;
        }
    }

    [Fact]
    [Trait("Category", "Data")]
    public void RefreshTasks_ShouldUpdateList_WhenExecuted()
    {
        // Arrange
        _taskListPage.WaitForPageToLoad();
        var initialCount = _taskListPage.GetTaskCount();

        try
        {
            // Act
            _taskListPage.RefreshTasks();

            // Assert
            _taskListPage.WaitForPageToLoad();
            var finalCount = _taskListPage.GetTaskCount();
            
            // Tasks should be reloaded (count might be same or different based on server state)
            finalCount.Should().BeGreaterOrEqualTo(0);
            
            TakeScreenshot("refresh_completed");
        }
        catch (Exception ex)
        {
            TakeScreenshot("refresh_failure");
            throw;
        }
    }

    protected override void Dispose()
    {
        try
        {
            TakeScreenshot("test_cleanup");
        }
        catch
        {
            // Ignore screenshot errors during cleanup
        }
        
        base.Dispose();
    }
}
```

---

## 3. 🔗 Integration Testing

### 3.1 API Integration Tests

```csharp
// Tests/IntegrationTests/ApiIntegrationTests.cs
public class ApiIntegrationTests : IClassFixture<WebApplicationFactory<Program>>, IDisposable
{
    private readonly WebApplicationFactory<Program> _factory;
    private readonly HttpClient _client;
    private readonly string _baseUrl;

    public ApiIntegrationTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory;
        _client = _factory.CreateClient();
        _baseUrl = "/api/tasks";
    }

    [Fact]
    public async Task GetTasks_ShouldReturnTasks_WhenTasksExist()
    {
        // Arrange
        await SeedTestDataAsync();

        // Act
        var response = await _client.GetAsync(_baseUrl);

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.OK);
        
        var content = await response.Content.ReadAsStringAsync();
        var tasks = JsonSerializer.Deserialize<List<TaskDto>>(content, JsonOptions.Default);
        
        tasks.Should().NotBeNull();
        tasks.Should().NotBeEmpty();
    }

    [Fact]
    public async Task CreateTask_ShouldReturnCreatedTask_WhenValidDataProvided()
    {
        // Arrange
        var newTask = new CreateTaskRequest
        {
            Title = "Integration Test Task",
            Description = "This is a test task created by integration test"
        };

        var json = JsonSerializer.Serialize(newTask, JsonOptions.Default);
        var content = new StringContent(json, Encoding.UTF8, "application/json");

        // Act
        var response = await _client.PostAsync(_baseUrl, content);

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.Created);
        
        var responseContent = await response.Content.ReadAsStringAsync();
        var createdTask = JsonSerializer.Deserialize<TaskDto>(responseContent, JsonOptions.Default);
        
        createdTask.Should().NotBeNull();
        createdTask!.Title.Should().Be(newTask.Title);
        createdTask.Description.Should().Be(newTask.Description);
        createdTask.Id.Should().BeGreaterThan(0);
    }

    [Theory]
    [InlineData("")]
    [InlineData(" ")]
    [InlineData(null)]
    public async Task CreateTask_ShouldReturnBadRequest_WhenTitleIsInvalid(string title)
    {
        // Arrange
        var newTask = new CreateTaskRequest
        {
            Title = title,
            Description = "Description"
        };

        var json = JsonSerializer.Serialize(newTask, JsonOptions.Default);
        var content = new StringContent(json, Encoding.UTF8, "application/json");

        // Act
        var response = await _client.PostAsync(_baseUrl, content);

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.BadRequest);
    }

    [Fact]
    public async Task UpdateTask_ShouldModifyTask_WhenValidDataProvided()
    {
        // Arrange
        var taskId = await CreateTestTaskAsync();
        var updateRequest = new UpdateTaskRequest
        {
            Title = "Updated Task Title",
            Description = "Updated description",
            IsCompleted = true
        };

        var json = JsonSerializer.Serialize(updateRequest, JsonOptions.Default);
        var content = new StringContent(json, Encoding.UTF8, "application/json");

        // Act
        var response = await _client.PutAsync($"{_baseUrl}/{taskId}", content);

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.OK);
        
        var responseContent = await response.Content.ReadAsStringAsync();
        var updatedTask = JsonSerializer.Deserialize<TaskDto>(responseContent, JsonOptions.Default);
        
        updatedTask.Should().NotBeNull();
        updatedTask!.Title.Should().Be(updateRequest.Title);
        updatedTask.Description.Should().Be(updateRequest.Description);
        updatedTask.IsCompleted.Should().Be(updateRequest.IsCompleted);
    }

    [Fact]
    public async Task DeleteTask_ShouldRemoveTask_WhenTaskExists()
    {
        // Arrange
        var taskId = await CreateTestTaskAsync();

        // Act
        var deleteResponse = await _client.DeleteAsync($"{_baseUrl}/{taskId}");

        // Assert
        deleteResponse.StatusCode.Should().Be(HttpStatusCode.NoContent);
        
        // Verify task is deleted
        var getResponse = await _client.GetAsync($"{_baseUrl}/{taskId}");
        getResponse.StatusCode.Should().Be(HttpStatusCode.NotFound);
    }

    [Fact]
    public async Task DeleteTask_ShouldReturnNotFound_WhenTaskDoesNotExist()
    {
        // Arrange
        var nonExistentId = 99999;

        // Act
        var response = await _client.DeleteAsync($"{_baseUrl}/{nonExistentId}");

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.NotFound);
    }

    [Fact]
    public async Task GetTask_ShouldReturnTask_WhenTaskExists()
    {
        // Arrange
        var taskId = await CreateTestTaskAsync();

        // Act
        var response = await _client.GetAsync($"{_baseUrl}/{taskId}");

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.OK);
        
        var content = await response.Content.ReadAsStringAsync();
        var task = JsonSerializer.Deserialize<TaskDto>(content, JsonOptions.Default);
        
        task.Should().NotBeNull();
        task!.Id.Should().Be(taskId);
    }

    private async Task<int> CreateTestTaskAsync()
    {
        var newTask = new CreateTaskRequest
        {
            Title = $"Test Task {Guid.NewGuid()}",
            Description = "Test Description"
        };

        var json = JsonSerializer.Serialize(newTask, JsonOptions.Default);
        var content = new StringContent(json, Encoding.UTF8, "application/json");
        
        var response = await _client.PostAsync(_baseUrl, content);
        response.EnsureSuccessStatusCode();
        
        var responseContent = await response.Content.ReadAsStringAsync();
        var createdTask = JsonSerializer.Deserialize<TaskDto>(responseContent, JsonOptions.Default);
        
        return createdTask!.Id;
    }

    private async Task SeedTestDataAsync()
    {
        for (int i = 0; i < 5; i++)
        {
            await CreateTestTaskAsync();
        }
    }

    public void Dispose()
    {
        _client?.Dispose();
    }
}
```

### 3.2 Database Integration Tests

```csharp
// Tests/IntegrationTests/DatabaseIntegrationTests.cs
public class DatabaseIntegrationTests : IDisposable
{
    private readonly TaskDbContext _context;
    private readonly TaskRepository _repository;

    public DatabaseIntegrationTests()
    {
        var options = new DbContextOptionsBuilder<TaskDbContext>()
            .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString())
            .Options;

        _context = new TaskDbContext(options);
        _repository = new TaskRepository(_context);
    }

    [Fact]
    public async Task CreateTask_ShouldPersistToDatabase_WhenValidDataProvided()
    {
        // Arrange
        var task = new TaskEntity
        {
            Title = "Test Task",
            Description = "Test Description",
            CreatedAt = DateTime.UtcNow,
            IsCompleted = false
        };

        // Act
        var createdTask = await _repository.CreateAsync(task);
        await _context.SaveChangesAsync();

        // Assert
        createdTask.Should().NotBeNull();
        createdTask.Id.Should().BeGreaterThan(0);
        
        var savedTask = await _context.Tasks.FindAsync(createdTask.Id);
        savedTask.Should().NotBeNull();
        savedTask!.Title.Should().Be(task.Title);
    }

    [Fact]
    public async Task GetTasks_ShouldReturnFilteredResults_WhenFiltersApplied()
    {
        // Arrange
        await SeedTestDataAsync();

        // Act
        var completedTasks = await _repository.GetTasksAsync(isCompleted: true);
        var incompleteTasks = await _repository.GetTasksAsync(isCompleted: false);

        // Assert
        completedTasks.Should().NotBeEmpty();
        completedTasks.Should().OnlyContain(t => t.IsCompleted);
        
        incompleteTasks.Should().NotBeEmpty();
        incompleteTasks.Should().OnlyContain(t => !t.IsCompleted);
    }

    [Fact]
    public async Task UpdateTask_ShouldModifyExistingTask_WhenTaskExists()
    {
        // Arrange
        var task = await CreateTestTaskAsync();
        var originalTitle = task.Title;
        
        task.Title = "Updated Title";
        task.IsCompleted = true;

        // Act
        var updatedTask = await _repository.UpdateAsync(task);
        await _context.SaveChangesAsync();

        // Assert
        updatedTask.Should().NotBeNull();
        updatedTask.Title.Should().Be("Updated Title");
        updatedTask.Title.Should().NotBe(originalTitle);
        updatedTask.IsCompleted.Should().BeTrue();
    }

    [Fact]
    public async Task DeleteTask_ShouldRemoveFromDatabase_WhenTaskExists()
    {
        // Arrange
        var task = await CreateTestTaskAsync();
        var taskId = task.Id;

        // Act
        await _repository.DeleteAsync(taskId);
        await _context.SaveChangesAsync();

        // Assert
        var deletedTask = await _context.Tasks.FindAsync(taskId);
        deletedTask.Should().BeNull();
    }

    [Fact]
    public async Task SearchTasks_ShouldReturnMatchingResults_WhenSearchTermProvided()
    {
        // Arrange
        await SeedTestDataAsync();
        var searchTerm = "important";

        // Act
        var results = await _repository.SearchTasksAsync(searchTerm);

        // Assert
        results.Should().NotBeEmpty();
        results.Should().OnlyContain(t => 
            t.Title.Contains(searchTerm, StringComparison.OrdinalIgnoreCase) ||
            t.Description.Contains(searchTerm, StringComparison.OrdinalIgnoreCase));
    }

    [Fact]
    public async Task GetTasksPaged_ShouldReturnCorrectPage_WhenPaginationApplied()
    {
        // Arrange
        await SeedTestDataAsync();
        var pageSize = 3;
        var pageNumber = 2;

        // Act
        var pagedResult = await _repository.GetTasksPagedAsync(pageNumber, pageSize);

        // Assert
        pagedResult.Should().NotBeNull();
        pagedResult.Data.Should().HaveCount(pageSize);
        pagedResult.PageNumber.Should().Be(pageNumber);
        pagedResult.TotalItems.Should().BeGreaterThan(pageSize);
    }

    private async Task<TaskEntity> CreateTestTaskAsync()
    {
        var task = new TaskEntity
        {
            Title = $"Test Task {Guid.NewGuid()}",
            Description = "Test Description",
            CreatedAt = DateTime.UtcNow,
            IsCompleted = false
        };

        _context.Tasks.Add(task);
        await _context.SaveChangesAsync();
        return task;
    }

    private async Task SeedTestDataAsync()
    {
        var tasks = new[]
        {
            new TaskEntity { Title = "Important task", Description = "This is important", IsCompleted = false, CreatedAt = DateTime.UtcNow },
            new TaskEntity { Title = "Completed task", Description = "This is done", IsCompleted = true, CreatedAt = DateTime.UtcNow.AddDays(-1) },
            new TaskEntity { Title = "Another important item", Description = "Also important", IsCompleted = false, CreatedAt = DateTime.UtcNow.AddDays(-2) },
            new TaskEntity { Title = "Regular task", Description = "Normal task", IsCompleted = false, CreatedAt = DateTime.UtcNow.AddDays(-3) },
            new TaskEntity { Title = "Old completed task", Description = "Old task", IsCompleted = true, CreatedAt = DateTime.UtcNow.AddDays(-7) },
            new TaskEntity { Title = "Priority task", Description = "High priority", IsCompleted = false, CreatedAt = DateTime.UtcNow.AddMinutes(-30) }
        };

        _context.Tasks.AddRange(tasks);
        await _context.SaveChangesAsync();
    }

    public void Dispose()
    {
        _context?.Dispose();
    }
}
```

---

## 4. 🧪 Practical Exercise

### Exercise: Complete Testing Suite

สร้าง comprehensive testing suite สำหรับ MAUI app ที่รวม unit tests, integration tests, และ UI tests:

```csharp
// Tests/TestingExercise/TestSuiteRunner.cs
public class TestSuiteRunner
{
    public static async Task<TestResults> RunAllTestsAsync()
    {
        var results = new TestResults();
        
        Console.WriteLine("🧪 Starting Comprehensive Test Suite...");
        
        // Run Unit Tests
        Console.WriteLine("📝 Running Unit Tests...");
        results.UnitTestResults = await RunUnitTestsAsync();
        
        // Run Integration Tests
        Console.WriteLine("🔗 Running Integration Tests...");
        results.IntegrationTestResults = await RunIntegrationTestsAsync();
        
        // Run UI Tests
        Console.WriteLine("🖱️ Running UI Tests...");
        results.UITestResults = await RunUITestsAsync();
        
        // Generate Report
        GenerateTestReport(results);
        
        return results;
    }

    private static async Task<TestResult> RunUnitTestsAsync()
    {
        var startTime = DateTime.UtcNow;
        var passed = 0;
        var failed = 0;
        var errors = new List<string>();

        try
        {
            // ServiceTests
            var serviceTestRunner = new ServiceTestRunner();
            var serviceResults = await serviceTestRunner.RunAsync();
            passed += serviceResults.Passed;
            failed += serviceResults.Failed;
            errors.AddRange(serviceResults.Errors);

            // ViewModelTests
            var viewModelTestRunner = new ViewModelTestRunner();
            var viewModelResults = await viewModelTestRunner.RunAsync();
            passed += viewModelResults.Passed;
            failed += viewModelResults.Failed;
            errors.AddRange(viewModelResults.Errors);

            Console.WriteLine($"✅ Unit Tests Completed: {passed} passed, {failed} failed");
        }
        catch (Exception ex)
        {
            errors.Add($"Unit Test Runner Error: {ex.Message}");
            failed++;
        }

        return new TestResult
        {
            Category = "Unit Tests",
            Passed = passed,
            Failed = failed,
            Duration = DateTime.UtcNow - startTime,
            Errors = errors
        };
    }

    private static async Task<TestResult> RunIntegrationTestsAsync()
    {
        var startTime = DateTime.UtcNow;
        var passed = 0;
        var failed = 0;
        var errors = new List<string>();

        try
        {
            // API Integration Tests
            var apiTestRunner = new ApiIntegrationTestRunner();
            var apiResults = await apiTestRunner.RunAsync();
            passed += apiResults.Passed;
            failed += apiResults.Failed;
            errors.AddRange(apiResults.Errors);

            // Database Integration Tests
            var dbTestRunner = new DatabaseIntegrationTestRunner();
            var dbResults = await dbTestRunner.RunAsync();
            passed += dbResults.Passed;
            failed += dbResults.Failed;
            errors.AddRange(dbResults.Errors);

            Console.WriteLine($"✅ Integration Tests Completed: {passed} passed, {failed} failed");
        }
        catch (Exception ex)
        {
            errors.Add($"Integration Test Runner Error: {ex.Message}");
            failed++;
        }

        return new TestResult
        {
            Category = "Integration Tests",
            Passed = passed,
            Failed = failed,
            Duration = DateTime.UtcNow - startTime,
            Errors = errors
        };
    }

    private static async Task<TestResult> RunUITestsAsync()
    {
        var startTime = DateTime.UtcNow;
        var passed = 0;
        var failed = 0;
        var errors = new List<string>();

        try
        {
            var uiTestRunner = new UITestRunner();
            var uiResults = await uiTestRunner.RunAsync();
            passed += uiResults.Passed;
            failed += uiResults.Failed;
            errors.AddRange(uiResults.Errors);

            Console.WriteLine($"✅ UI Tests Completed: {passed} passed, {failed} failed");
        }
        catch (Exception ex)
        {
            errors.Add($"UI Test Runner Error: {ex.Message}");
            failed++;
        }

        return new TestResult
        {
            Category = "UI Tests",
            Passed = passed,
            Failed = failed,
            Duration = DateTime.UtcNow - startTime,
            Errors = errors
        };
    }

    private static void GenerateTestReport(TestResults results)
    {
        var report = new StringBuilder();
        report.AppendLine("# Test Execution Report");
        report.AppendLine($"Generated: {DateTime.UtcNow:yyyy-MM-dd HH:mm:ss} UTC");
        report.AppendLine();

        var totalPassed = results.UnitTestResults.Passed + results.IntegrationTestResults.Passed + results.UITestResults.Passed;
        var totalFailed = results.UnitTestResults.Failed + results.IntegrationTestResults.Failed + results.UITestResults.Failed;
        var totalDuration = results.UnitTestResults.Duration + results.IntegrationTestResults.Duration + results.UITestResults.Duration;

        report.AppendLine("## Summary");
        report.AppendLine($"- Total Tests: {totalPassed + totalFailed}");
        report.AppendLine($"- Passed: {totalPassed}");
        report.AppendLine($"- Failed: {totalFailed}");
        report.AppendLine($"- Success Rate: {(double)totalPassed / (totalPassed + totalFailed) * 100:F1}%");
        report.AppendLine($"- Total Duration: {totalDuration.TotalSeconds:F1} seconds");
        report.AppendLine();

        // Detailed results
        AppendCategoryResults(report, results.UnitTestResults);
        AppendCategoryResults(report, results.IntegrationTestResults);
        AppendCategoryResults(report, results.UITestResults);

        var reportPath = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "TestReport.md");
        File.WriteAllText(reportPath, report.ToString());
        
        Console.WriteLine($"📊 Test report generated: {reportPath}");
    }

    private static void AppendCategoryResults(StringBuilder report, TestResult result)
    {
        report.AppendLine($"## {result.Category}");
        report.AppendLine($"- Passed: {result.Passed}");
        report.AppendLine($"- Failed: {result.Failed}");
        report.AppendLine($"- Duration: {result.Duration.TotalSeconds:F1} seconds");
        
        if (result.Errors.Any())
        {
            report.AppendLine("### Errors:");
            foreach (var error in result.Errors)
            {
                report.AppendLine($"- {error}");
            }
        }
        
        report.AppendLine();
    }
}

public class TestResults
{
    public TestResult UnitTestResults { get; set; } = new();
    public TestResult IntegrationTestResults { get; set; } = new();
    public TestResult UITestResults { get; set; } = new();
}

public class TestResult
{
    public string Category { get; set; } = "";
    public int Passed { get; set; }
    public int Failed { get; set; }
    public TimeSpan Duration { get; set; }
    public List<string> Errors { get; set; } = new();
}
```

### Testing Checklist

**Unit Testing:**
- [ ] All ViewModels have comprehensive tests
- [ ] All Services have unit tests with mocking
- [ ] Business logic is thoroughly tested
- [ ] Edge cases and error scenarios covered
- [ ] Property change notifications tested

**Integration Testing:**
- [ ] API endpoints tested end-to-end
- [ ] Database operations tested
- [ ] Authentication flows tested
- [ ] External service integrations tested
- [ ] Configuration and dependency injection tested

**UI Testing:**
- [ ] Critical user flows automated
- [ ] Cross-platform UI consistency tested
- [ ] Performance and responsiveness tested
- [ ] Accessibility features tested
- [ ] Error handling in UI tested

**Test Infrastructure:**
- [ ] Automated test execution in CI/CD
- [ ] Test data management strategy
- [ ] Test environment isolation
- [ ] Performance benchmarking
- [ ] Test reporting and metrics

---

## 5. 📚 Summary

ในบทนี้เราได้เรียนรู้:

### 🔑 Key Concepts
- **Comprehensive Testing Strategy** สำหรับ MAUI applications
- **Unit Testing** ด้วย xUnit, FluentAssertions, และ NSubstitute
- **UI Testing** ด้วย Appium และ Page Object Model
- **Integration Testing** สำหรับ APIs และ databases
- **Test Automation** และ CI/CD integration

### 🛠️ Technical Skills
- การสร้าง maintainable test suites
- การใช้ mocking frameworks อย่างมีประสิทธิภาพ
- การทำ cross-platform UI testing
- การ setup test infrastructure และ automation
- การสร้าง comprehensive test reports

### 🧪 Testing Best Practices
- **Test Pyramid:** Unit tests (majority), Integration tests (some), UI tests (few but critical)
- **Test Isolation:** Each test should be independent and repeatable
- **Test Data:** Use builders and fixtures for consistent test data
- **Test Performance:** Fast-running tests encourage frequent execution
- **Test Maintenance:** Keep tests simple and focused on single responsibilities

### 📊 Quality Assurance
- การวัด test coverage และ code quality
- การใช้ continuous testing ใน development workflow
- การจัดการ test environments และ data
- การ monitor test results และ trends

Testing เป็นส่วนสำคัญของการพัฒนา software ที่มีคุณภาพ การมี comprehensive testing strategy จะช่วยให้เรามั่นใจในคุณภาพของแอปพลิเคชันและลดความเสี่ยงจาก bugs ใน production

**Next Chapter Preview:** ต่อไปเราจะเรียนรู้เกี่ยวกับ Accessibility & Localization สำหรับการสร้างแอปพลิเคชันที่เข้าถึงได้และรองรับหลายภาษา! 🌍
