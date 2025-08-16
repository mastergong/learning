# 🧪 Chapter 15: Testing Strategies

> **หัวข้อ**: Unit Testing, UI Testing & Test-Driven Development  
> **ระยะเวลา**: 3.5 ชั่วโมง  
> **ความยากง่าย**: ระดับกลาง-สูง  
> **Prerequisites**: บทที่ 1-14

## 🎯 Learning Objectives

หลังจากเรียนบทนี้แล้ว คุณจะสามารถ:

- ✅ สร้าง comprehensive test suite สำหรับ MAUI apps
- ✅ ใช้ TDD (Test-Driven Development) approach
- ✅ เขียน unit tests สำหรับ ViewModels และ Services
- ✅ สร้าง UI automation tests
- ✅ ใช้ testing frameworks และ mocking libraries

---

## 📱 Chapter Overview

### 🏗️ What We'll Test
สร้าง **"Comprehensive Testing Suite"** ที่มี:
- 🧪 Unit tests สำหรับ business logic
- 🤖 UI automation tests
- 📊 Code coverage analysis
- 🔄 Continuous integration testing
- 📈 Performance testing

---

## 1. 🧪 Unit Testing Fundamentals

### 1.1 Test Project Setup

```xml
<!-- d:\A-Work\learn from ai\maui-xaml-mastery\tests\UnitTests\UnitTests.csproj -->
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
    <PackageReference Include="coverlet.collector" Version="6.0.0" />
    <PackageReference Include="FluentAssertions" Version="6.12.0" />
    <PackageReference Include="NSubstitute" Version="5.1.0" />
    <PackageReference Include="AutoFixture" Version="4.18.0" />
    <PackageReference Include="AutoFixture.Xunit2" Version="4.18.0" />
    <PackageReference Include="Microsoft.Extensions.DependencyInjection" Version="8.0.0" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\..\src\TestingExample\TestingExample.csproj" />
  </ItemGroup>

</Project>
```

### 1.2 Base Test Class

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\tests\UnitTests\BaseTestClass.cs
using AutoFixture;
using NSubstitute;
using Microsoft.Extensions.DependencyInjection;

namespace UnitTests;

public abstract class BaseTestClass
{
    protected readonly IFixture Fixture;
    protected readonly IServiceProvider ServiceProvider;

    protected BaseTestClass()
    {
        Fixture = new Fixture();
        
        // Configure AutoFixture
        Fixture.Behaviors.OfType<ThrowingRecursionBehavior>().ToList()
            .ForEach(b => Fixture.Behaviors.Remove(b));
        Fixture.Behaviors.Add(new OmitOnRecursionBehavior());

        // Setup service container for dependency injection testing
        var services = new ServiceCollection();
        ConfigureServices(services);
        ServiceProvider = services.BuildServiceProvider();
    }

    protected virtual void ConfigureServices(IServiceCollection services)
    {
        // Override in derived classes to add specific services
    }

    protected T CreateMock<T>() where T : class
    {
        return Substitute.For<T>();
    }

    protected T CreateService<T>() where T : class
    {
        return ServiceProvider.GetRequiredService<T>();
    }

    protected async Task<T> CreateAsync<T>() where T : class
    {
        var instance = Fixture.Create<T>();
        
        // If it's an async-initializable object, initialize it
        if (instance is IAsyncInitializable asyncInit)
        {
            await asyncInit.InitializeAsync();
        }
        
        return instance;
    }
}

public interface IAsyncInitializable
{
    Task InitializeAsync();
}
```

### 1.3 ViewModel Testing

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\tests\UnitTests\ViewModels\TaskListViewModelTests.cs
using FluentAssertions;
using NSubstitute;
using TestingExample.Models;
using TestingExample.Services;
using TestingExample.ViewModels;

namespace UnitTests.ViewModels;

public class TaskListViewModelTests : BaseTestClass
{
    private readonly ITaskService _taskService;
    private readonly INavigationService _navigationService;
    private readonly IDialogService _dialogService;
    private readonly TaskListViewModel _viewModel;

    public TaskListViewModelTests()
    {
        _taskService = CreateMock<ITaskService>();
        _navigationService = CreateMock<INavigationService>();
        _dialogService = CreateMock<IDialogService>();
        
        _viewModel = new TaskListViewModel(
            _taskService,
            _navigationService,
            _dialogService);
    }

    [Fact]
    public async Task LoadTasksAsync_ShouldPopulateTasksCollection()
    {
        // Arrange
        var tasks = Fixture.CreateMany<TaskItem>(5).ToList();
        _taskService.GetAllTasksAsync().Returns(tasks);

        // Act
        await _viewModel.LoadTasksAsync();

        // Assert
        _viewModel.Tasks.Should().HaveCount(5);
        _viewModel.Tasks.Should().BeEquivalentTo(tasks);
        _viewModel.IsLoading.Should().BeFalse();
    }

    [Fact]
    public async Task LoadTasksAsync_WhenServiceThrows_ShouldShowError()
    {
        // Arrange
        var exception = new Exception("Service error");
        _taskService.GetAllTasksAsync().ThrowsAsync(exception);

        // Act
        await _viewModel.LoadTasksAsync();

        // Assert
        _viewModel.Tasks.Should().BeEmpty();
        _viewModel.HasError.Should().BeTrue();
        _viewModel.ErrorMessage.Should().Be("Service error");
        await _dialogService.Received(1).ShowErrorAsync("Error", "Service error");
    }

    [Fact]
    public async Task AddTaskCommand_WithValidInput_ShouldAddTask()
    {
        // Arrange
        var newTask = Fixture.Create<TaskItem>();
        _taskService.CreateTaskAsync(Arg.Any<TaskItem>()).Returns(newTask);
        _viewModel.NewTaskTitle = "New Task";

        // Act
        await _viewModel.AddTaskCommand.ExecuteAsync(null);

        // Assert
        _viewModel.Tasks.Should().Contain(t => t.Title == "New Task");
        _viewModel.NewTaskTitle.Should().BeEmpty();
        await _taskService.Received(1).CreateTaskAsync(Arg.Is<TaskItem>(t => t.Title == "New Task"));
    }

    [Fact]
    public void AddTaskCommand_WithEmptyTitle_ShouldNotExecute()
    {
        // Arrange
        _viewModel.NewTaskTitle = "";

        // Act & Assert
        _viewModel.AddTaskCommand.CanExecute(null).Should().BeFalse();
    }

    [Fact]
    public async Task DeleteTaskCommand_ShouldRemoveTaskFromCollection()
    {
        // Arrange
        var task = Fixture.Create<TaskItem>();
        _viewModel.Tasks.Add(task);
        _taskService.DeleteTaskAsync(task.Id).Returns(true);

        // Act
        await _viewModel.DeleteTaskCommand.ExecuteAsync(task);

        // Assert
        _viewModel.Tasks.Should().NotContain(task);
        await _taskService.Received(1).DeleteTaskAsync(task.Id);
    }

    [Fact]
    public async Task ToggleTaskCommand_ShouldUpdateTaskStatus()
    {
        // Arrange
        var task = Fixture.Build<TaskItem>()
            .With(t => t.IsCompleted, false)
            .Create();
        
        _viewModel.Tasks.Add(task);
        
        var updatedTask = task with { IsCompleted = true };
        _taskService.UpdateTaskAsync(Arg.Any<TaskItem>()).Returns(updatedTask);

        // Act
        await _viewModel.ToggleTaskCommand.ExecuteAsync(task);

        // Assert
        var taskInCollection = _viewModel.Tasks.First(t => t.Id == task.Id);
        taskInCollection.IsCompleted.Should().BeTrue();
        await _taskService.Received(1).UpdateTaskAsync(Arg.Is<TaskItem>(t => t.IsCompleted == true));
    }

    [Theory]
    [InlineData("")]
    [InlineData("   ")]
    [InlineData(null)]
    public void NewTaskTitle_WhenEmpty_ShouldDisableAddCommand(string title)
    {
        // Arrange & Act
        _viewModel.NewTaskTitle = title;

        // Assert
        _viewModel.AddTaskCommand.CanExecute(null).Should().BeFalse();
    }

    [Fact]
    public void NewTaskTitle_WhenValid_ShouldEnableAddCommand()
    {
        // Arrange & Act
        _viewModel.NewTaskTitle = "Valid Task Title";

        // Assert
        _viewModel.AddTaskCommand.CanExecute(null).Should().BeTrue();
    }

    [Fact]
    public async Task RefreshCommand_ShouldReloadTasks()
    {
        // Arrange
        var initialTasks = Fixture.CreateMany<TaskItem>(3).ToList();
        var refreshedTasks = Fixture.CreateMany<TaskItem>(5).ToList();
        
        _taskService.GetAllTasksAsync().Returns(initialTasks, refreshedTasks);

        // Load initial tasks
        await _viewModel.LoadTasksAsync();
        _viewModel.Tasks.Should().HaveCount(3);

        // Act
        await _viewModel.RefreshCommand.ExecuteAsync(null);

        // Assert
        _viewModel.Tasks.Should().HaveCount(5);
        _viewModel.Tasks.Should().BeEquivalentTo(refreshedTasks);
        _viewModel.IsRefreshing.Should().BeFalse();
    }

    [Fact]
    public async Task NavigateToTaskDetailCommand_ShouldNavigateWithCorrectParameter()
    {
        // Arrange
        var task = Fixture.Create<TaskItem>();

        // Act
        await _viewModel.NavigateToTaskDetailCommand.ExecuteAsync(task);

        // Assert
        await _navigationService.Received(1).NavigateToAsync("TaskDetailPage", Arg.Is<Dictionary<string, object>>(
            dict => dict.ContainsKey("TaskId") && (int)dict["TaskId"] == task.Id));
    }

    [Fact]
    public void TasksCollectionChanged_ShouldUpdateTaskCount()
    {
        // Arrange
        var initialCount = _viewModel.TaskCount;

        // Act
        _viewModel.Tasks.Add(Fixture.Create<TaskItem>());
        _viewModel.Tasks.Add(Fixture.Create<TaskItem>());

        // Assert
        _viewModel.TaskCount.Should().Be(initialCount + 2);
    }

    [Fact]
    public void CompletedTasksCount_ShouldReturnCorrectCount()
    {
        // Arrange
        var completedTask1 = Fixture.Build<TaskItem>().With(t => t.IsCompleted, true).Create();
        var completedTask2 = Fixture.Build<TaskItem>().With(t => t.IsCompleted, true).Create();
        var incompleteTask = Fixture.Build<TaskItem>().With(t => t.IsCompleted, false).Create();

        _viewModel.Tasks.Add(completedTask1);
        _viewModel.Tasks.Add(completedTask2);
        _viewModel.Tasks.Add(incompleteTask);

        // Act & Assert
        _viewModel.CompletedTasksCount.Should().Be(2);
    }
}
```

---

## 2. 🤖 UI Testing with Appium

### 2.1 UI Test Project Setup

```xml
<!-- d:\A-Work\learn from ai\maui-xaml-mastery\tests\UITests\UITests.csproj -->
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
    <PackageReference Include="Appium.WebDriver" Version="5.0.0-rc.1" />
    <PackageReference Include="FluentAssertions" Version="6.12.0" />
    <PackageReference Include="Microsoft.Extensions.Configuration" Version="8.0.0" />
    <PackageReference Include="Microsoft.Extensions.Configuration.Json" Version="8.0.0" />
  </ItemGroup>

</Project>
```

### 2.2 Base UI Test Class

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\tests\UITests\BaseUITest.cs
using OpenQA.Selenium.Appium;
using OpenQA.Selenium.Appium.Android;
using OpenQA.Selenium.Appium.iOS;
using OpenQA.Selenium.Appium.Windows;
using Microsoft.Extensions.Configuration;

namespace UITests;

public abstract class BaseUITest : IDisposable
{
    protected AppiumDriver Driver { get; private set; }
    protected TestConfiguration Config { get; private set; }

    protected BaseUITest()
    {
        LoadConfiguration();
        InitializeDriver();
    }

    private void LoadConfiguration()
    {
        var builder = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("appsettings.json", optional: false)
            .AddJsonFile($"appsettings.{Environment.GetEnvironmentVariable("TEST_ENVIRONMENT") ?? "Development"}.json", optional: true);

        var configuration = builder.Build();
        Config = configuration.Get<TestConfiguration>() ?? new TestConfiguration();
    }

    private void InitializeDriver()
    {
        var serverUri = new Uri(Config.AppiumServerUrl);

        Driver = Config.Platform.ToLower() switch
        {
            "android" => CreateAndroidDriver(serverUri),
            "ios" => CreateIOSDriver(serverUri),
            "windows" => CreateWindowsDriver(serverUri),
            _ => throw new ArgumentException($"Unsupported platform: {Config.Platform}")
        };

        Driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(Config.ImplicitWaitSeconds);
    }

    private AndroidDriver CreateAndroidDriver(Uri serverUri)
    {
        var options = new AppiumOptions();
        options.AddAdditionalCapability("platformName", "Android");
        options.AddAdditionalCapability("platformVersion", Config.Android.PlatformVersion);
        options.AddAdditionalCapability("deviceName", Config.Android.DeviceName);
        options.AddAdditionalCapability("app", Config.Android.AppPath);
        options.AddAdditionalCapability("appPackage", Config.Android.AppPackage);
        options.AddAdditionalCapability("appActivity", Config.Android.AppActivity);
        options.AddAdditionalCapability("automationName", "UiAutomator2");
        options.AddAdditionalCapability("newCommandTimeout", 300);

        return new AndroidDriver(serverUri, options);
    }

    private IOSDriver CreateIOSDriver(Uri serverUri)
    {
        var options = new AppiumOptions();
        options.AddAdditionalCapability("platformName", "iOS");
        options.AddAdditionalCapability("platformVersion", Config.iOS.PlatformVersion);
        options.AddAdditionalCapability("deviceName", Config.iOS.DeviceName);
        options.AddAdditionalCapability("app", Config.iOS.AppPath);
        options.AddAdditionalCapability("bundleId", Config.iOS.BundleId);
        options.AddAdditionalCapability("automationName", "XCUITest");
        options.AddAdditionalCapability("newCommandTimeout", 300);

        return new IOSDriver(serverUri, options);
    }

    private WindowsDriver CreateWindowsDriver(Uri serverUri)
    {
        var options = new AppiumOptions();
        options.AddAdditionalCapability("platformName", "Windows");
        options.AddAdditionalCapability("app", Config.Windows.AppPath);
        options.AddAdditionalCapability("automationName", "Windows");
        options.AddAdditionalCapability("newCommandTimeout", 300);

        return new WindowsDriver(serverUri, options);
    }

    protected AppiumElement FindElement(string accessibilityId)
    {
        return Driver.FindElement(MobileBy.AccessibilityId(accessibilityId));
    }

    protected AppiumElement FindElementByXPath(string xpath)
    {
        return Driver.FindElement(MobileBy.XPath(xpath));
    }

    protected IList<AppiumElement> FindElements(string accessibilityId)
    {
        return Driver.FindElements(MobileBy.AccessibilityId(accessibilityId));
    }

    protected void WaitForElement(string accessibilityId, int timeoutSeconds = 10)
    {
        var wait = new WebDriverWait(Driver, TimeSpan.FromSeconds(timeoutSeconds));
        wait.Until(driver => 
        {
            try
            {
                var element = FindElement(accessibilityId);
                return element.Displayed;
            }
            catch
            {
                return false;
            }
        });
    }

    protected void TakeScreenshot(string fileName = null)
    {
        fileName ??= $"screenshot_{DateTime.Now:yyyyMMdd_HHmmss}.png";
        var screenshot = Driver.GetScreenshot();
        
        var screenshotDir = Path.Combine(Directory.GetCurrentDirectory(), "Screenshots");
        Directory.CreateDirectory(screenshotDir);
        
        var filePath = Path.Combine(screenshotDir, fileName);
        screenshot.SaveAsFile(filePath);
        
        Console.WriteLine($"Screenshot saved: {filePath}");
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
    }
}

public class TestConfiguration
{
    public string Platform { get; set; } = "Android";
    public string AppiumServerUrl { get; set; } = "http://localhost:4723";
    public int ImplicitWaitSeconds { get; set; } = 10;
    public AndroidConfig Android { get; set; } = new();
    public IOSConfig iOS { get; set; } = new();
    public WindowsConfig Windows { get; set; } = new();
}

public class AndroidConfig
{
    public string PlatformVersion { get; set; } = "13";
    public string DeviceName { get; set; } = "Android Emulator";
    public string AppPath { get; set; } = "";
    public string AppPackage { get; set; } = "com.companyname.testingexample";
    public string AppActivity { get; set; } = "crc64...MainActivity";
}

public class IOSConfig
{
    public string PlatformVersion { get; set; } = "17.0";
    public string DeviceName { get; set; } = "iPhone 15 Simulator";
    public string AppPath { get; set; } = "";
    public string BundleId { get; set; } = "com.companyname.testingexample";
}

public class WindowsConfig
{
    public string AppPath { get; set; } = "";
}
```

### 2.3 Page Object Model

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\tests\UITests\PageObjects\TaskListPage.cs
using OpenQA.Selenium.Appium;
using FluentAssertions;

namespace UITests.PageObjects;

public class TaskListPage : BasePage
{
    // Element locators
    private const string AddButtonId = "AddTaskButton";
    private const string TaskTitleEntryId = "TaskTitleEntry";
    private const string TaskListId = "TaskList";
    private const string RefreshButtonId = "RefreshButton";
    private const string TaskItemPrefix = "TaskItem_";
    private const string DeleteButtonPrefix = "DeleteButton_";
    private const string ToggleButtonPrefix = "ToggleButton_";

    public TaskListPage(AppiumDriver driver) : base(driver)
    {
    }

    public void WaitForPageToLoad()
    {
        WaitForElement(TaskListId);
    }

    public void EnterTaskTitle(string title)
    {
        var titleEntry = FindElement(TaskTitleEntryId);
        titleEntry.Clear();
        titleEntry.SendKeys(title);
    }

    public void TapAddButton()
    {
        var addButton = FindElement(AddButtonId);
        addButton.Click();
    }

    public void AddTask(string title)
    {
        EnterTaskTitle(title);
        TapAddButton();
    }

    public IList<AppiumElement> GetTaskItems()
    {
        return FindElements(MobileBy.XPath($"//*[starts-with(@AutomationId, '{TaskItemPrefix}')]"));
    }

    public AppiumElement GetTaskItem(int index)
    {
        return FindElement($"{TaskItemPrefix}{index}");
    }

    public void DeleteTask(int index)
    {
        var deleteButton = FindElement($"{DeleteButtonPrefix}{index}");
        deleteButton.Click();
    }

    public void ToggleTask(int index)
    {
        var toggleButton = FindElement($"{ToggleButtonPrefix}{index}");
        toggleButton.Click();
    }

    public void RefreshTasks()
    {
        var refreshButton = FindElement(RefreshButtonId);
        refreshButton.Click();
    }

    public void SwipeToRefresh()
    {
        // Perform pull-to-refresh gesture
        var taskList = FindElement(TaskListId);
        var startX = taskList.Size.Width / 2;
        var startY = taskList.Size.Height / 4;
        var endY = taskList.Size.Height * 3 / 4;

        TouchAction touchAction = new TouchAction(Driver);
        touchAction.Press(startX, startY)
                  .MoveTo(startX, endY)
                  .Release()
                  .Perform();
    }

    public string GetTaskTitle(int index)
    {
        var taskItem = GetTaskItem(index);
        var titleElement = taskItem.FindElement(MobileBy.AccessibilityId($"TaskTitle_{index}"));
        return titleElement.Text;
    }

    public bool IsTaskCompleted(int index)
    {
        var taskItem = GetTaskItem(index);
        var completedAttribute = taskItem.GetAttribute("IsCompleted");
        return bool.Parse(completedAttribute ?? "false");
    }

    public int GetTaskCount()
    {
        return GetTaskItems().Count;
    }

    public void VerifyTaskExists(string title)
    {
        var tasks = GetTaskItems();
        var taskExists = tasks.Any(task => 
        {
            try
            {
                var titleElement = task.FindElement(MobileBy.XPath(".//*/[@AutomationId[starts-with(., 'TaskTitle_')]]"));
                return titleElement.Text == title;
            }
            catch
            {
                return false;
            }
        });

        taskExists.Should().BeTrue($"Task with title '{title}' should exist");
    }

    public void VerifyTaskDoesNotExist(string title)
    {
        var tasks = GetTaskItems();
        var taskExists = tasks.Any(task =>
        {
            try
            {
                var titleElement = task.FindElement(MobileBy.XPath(".//*/[@AutomationId[starts-with(., 'TaskTitle_')]]"));
                return titleElement.Text == title;
            }
            catch
            {
                return false;
            }
        });

        taskExists.Should().BeFalse($"Task with title '{title}' should not exist");
    }
}

public abstract class BasePage
{
    protected AppiumDriver Driver { get; }

    protected BasePage(AppiumDriver driver)
    {
        Driver = driver;
    }

    protected AppiumElement FindElement(string accessibilityId)
    {
        return Driver.FindElement(MobileBy.AccessibilityId(accessibilityId));
    }

    protected IList<AppiumElement> FindElements(MobileBy locator)
    {
        return Driver.FindElements(locator);
    }

    protected void WaitForElement(string accessibilityId, int timeoutSeconds = 10)
    {
        var wait = new WebDriverWait(Driver, TimeSpan.FromSeconds(timeoutSeconds));
        wait.Until(driver =>
        {
            try
            {
                var element = FindElement(accessibilityId);
                return element.Displayed;
            }
            catch
            {
                return false;
            }
        });
    }

    protected void TakeScreenshot(string fileName = null)
    {
        fileName ??= $"screenshot_{DateTime.Now:yyyyMMdd_HHmmss}.png";
        var screenshot = Driver.GetScreenshot();

        var screenshotDir = Path.Combine(Directory.GetCurrentDirectory(), "Screenshots");
        Directory.CreateDirectory(screenshotDir);

        var filePath = Path.Combine(screenshotDir, fileName);
        screenshot.SaveAsFile(filePath);

        Console.WriteLine($"Screenshot saved: {filePath}");
    }
}
```

### 2.4 UI Test Implementation

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\tests\UITests\TaskListUITests.cs
using FluentAssertions;
using UITests.PageObjects;

namespace UITests;

public class TaskListUITests : BaseUITest
{
    private TaskListPage _taskListPage;

    public TaskListUITests()
    {
        _taskListPage = new TaskListPage(Driver);
    }

    [Fact]
    public void TaskListPage_ShouldLoadSuccessfully()
    {
        // Act
        _taskListPage.WaitForPageToLoad();

        // Assert
        // If we reach this point without timeout, the page loaded successfully
        Assert.True(true);
    }

    [Fact]
    public void AddTask_WithValidTitle_ShouldCreateNewTask()
    {
        // Arrange
        var taskTitle = "Test Task " + DateTime.Now.Ticks;
        _taskListPage.WaitForPageToLoad();
        var initialTaskCount = _taskListPage.GetTaskCount();

        // Act
        _taskListPage.AddTask(taskTitle);

        // Assert
        _taskListPage.GetTaskCount().Should().Be(initialTaskCount + 1);
        _taskListPage.VerifyTaskExists(taskTitle);
    }

    [Fact]
    public void AddTask_WithEmptyTitle_ShouldNotCreateTask()
    {
        // Arrange
        _taskListPage.WaitForPageToLoad();
        var initialTaskCount = _taskListPage.GetTaskCount();

        // Act
        _taskListPage.EnterTaskTitle("");
        _taskListPage.TapAddButton();

        // Assert
        _taskListPage.GetTaskCount().Should().Be(initialTaskCount);
    }

    [Fact]
    public void DeleteTask_ShouldRemoveTaskFromList()
    {
        // Arrange
        var taskTitle = "Task to Delete " + DateTime.Now.Ticks;
        _taskListPage.WaitForPageToLoad();
        _taskListPage.AddTask(taskTitle);
        
        var taskCountAfterAdd = _taskListPage.GetTaskCount();
        _taskListPage.VerifyTaskExists(taskTitle);

        // Act
        _taskListPage.DeleteTask(taskCountAfterAdd - 1); // Delete the last task

        // Assert
        _taskListPage.GetTaskCount().Should().Be(taskCountAfterAdd - 1);
        _taskListPage.VerifyTaskDoesNotExist(taskTitle);
    }

    [Fact]
    public void ToggleTask_ShouldChangeTaskCompletionStatus()
    {
        // Arrange
        var taskTitle = "Task to Toggle " + DateTime.Now.Ticks;
        _taskListPage.WaitForPageToLoad();
        _taskListPage.AddTask(taskTitle);
        
        var taskIndex = _taskListPage.GetTaskCount() - 1;
        var initialStatus = _taskListPage.IsTaskCompleted(taskIndex);

        // Act
        _taskListPage.ToggleTask(taskIndex);

        // Assert
        _taskListPage.IsTaskCompleted(taskIndex).Should().Be(!initialStatus);
    }

    [Fact]
    public void RefreshTasks_ShouldReloadTaskList()
    {
        // Arrange
        _taskListPage.WaitForPageToLoad();

        // Act
        _taskListPage.RefreshTasks();

        // Assert
        // Verify that the refresh completed (page is still accessible)
        _taskListPage.WaitForPageToLoad();
        Assert.True(true); // If we reach here, refresh worked
    }

    [Fact]
    public void SwipeToRefresh_ShouldReloadTaskList()
    {
        // Arrange
        _taskListPage.WaitForPageToLoad();

        // Act
        _taskListPage.SwipeToRefresh();

        // Wait for refresh to complete
        Thread.Sleep(2000);

        // Assert
        _taskListPage.WaitForPageToLoad();
        Assert.True(true); // If we reach here, swipe refresh worked
    }

    [Fact]
    public void AddMultipleTasks_ShouldCreateAllTasks()
    {
        // Arrange
        var taskTitles = new[]
        {
            "First Task " + DateTime.Now.Ticks,
            "Second Task " + DateTime.Now.Ticks,
            "Third Task " + DateTime.Now.Ticks
        };
        
        _taskListPage.WaitForPageToLoad();
        var initialTaskCount = _taskListPage.GetTaskCount();

        // Act
        foreach (var title in taskTitles)
        {
            _taskListPage.AddTask(title);
        }

        // Assert
        _taskListPage.GetTaskCount().Should().Be(initialTaskCount + taskTitles.Length);
        
        foreach (var title in taskTitles)
        {
            _taskListPage.VerifyTaskExists(title);
        }
    }

    [Fact]
    public void TaskInteraction_CompleteWorkflow_ShouldWorkCorrectly()
    {
        // Arrange
        var taskTitle = "Workflow Test Task " + DateTime.Now.Ticks;
        _taskListPage.WaitForPageToLoad();

        try
        {
            // Act & Assert - Add task
            _taskListPage.AddTask(taskTitle);
            _taskListPage.VerifyTaskExists(taskTitle);

            var taskIndex = _taskListPage.GetTaskCount() - 1;

            // Act & Assert - Toggle task to completed
            _taskListPage.ToggleTask(taskIndex);
            _taskListPage.IsTaskCompleted(taskIndex).Should().BeTrue();

            // Act & Assert - Toggle task back to incomplete
            _taskListPage.ToggleTask(taskIndex);
            _taskListPage.IsTaskCompleted(taskIndex).Should().BeFalse();

            // Act & Assert - Delete task
            _taskListPage.DeleteTask(taskIndex);
            _taskListPage.VerifyTaskDoesNotExist(taskTitle);
        }
        catch (Exception ex)
        {
            // Take screenshot on failure
            _taskListPage.TakeScreenshot($"workflow_test_failure_{DateTime.Now:yyyyMMdd_HHmmss}.png");
            throw;
        }
    }

    protected override void Dispose()
    {
        try
        {
            // Take a final screenshot before closing
            _taskListPage?.TakeScreenshot($"test_end_{DateTime.Now:yyyyMMdd_HHmmss}.png");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error taking final screenshot: {ex.Message}");
        }
        
        base.Dispose();
    }
}
```

---

## 3. 🧩 Service Testing

### 3.1 Repository Testing

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\tests\UnitTests\Services\TaskRepositoryTests.cs
using FluentAssertions;
using NSubstitute;
using TestingExample.Data;
using TestingExample.Models;
using TestingExample.Repositories;
using Microsoft.EntityFrameworkCore;

namespace UnitTests.Services;

public class TaskRepositoryTests : BaseTestClass, IDisposable
{
    private readonly DbContextOptions<TaskDbContext> _dbOptions;
    private readonly TaskDbContext _context;
    private readonly TaskRepository _repository;

    public TaskRepositoryTests()
    {
        // Use in-memory database for testing
        _dbOptions = new DbContextOptionsBuilder<TaskDbContext>()
            .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString())
            .Options;

        _context = new TaskDbContext(_dbOptions);
        _repository = new TaskRepository(_context);
    }

    [Fact]
    public async Task GetAllAsync_ShouldReturnAllTasks()
    {
        // Arrange
        var tasks = Fixture.CreateMany<TaskItem>(5).ToList();
        await _context.Tasks.AddRangeAsync(tasks);
        await _context.SaveChangesAsync();

        // Act
        var result = await _repository.GetAllAsync();

        // Assert
        result.Should().HaveCount(5);
        result.Should().BeEquivalentTo(tasks);
    }

    [Fact]
    public async Task GetByIdAsync_WithValidId_ShouldReturnTask()
    {
        // Arrange
        var task = Fixture.Create<TaskItem>();
        await _context.Tasks.AddAsync(task);
        await _context.SaveChangesAsync();

        // Act
        var result = await _repository.GetByIdAsync(task.Id);

        // Assert
        result.Should().NotBeNull();
        result.Should().BeEquivalentTo(task);
    }

    [Fact]
    public async Task GetByIdAsync_WithInvalidId_ShouldReturnNull()
    {
        // Act
        var result = await _repository.GetByIdAsync(999);

        // Assert
        result.Should().BeNull();
    }

    [Fact]
    public async Task CreateAsync_WithValidTask_ShouldAddTask()
    {
        // Arrange
        var task = Fixture.Build<TaskItem>()
            .Without(t => t.Id) // Don't set ID for new entity
            .Create();

        // Act
        var result = await _repository.CreateAsync(task);

        // Assert
        result.Should().NotBeNull();
        result.Id.Should().BeGreaterThan(0);
        result.Title.Should().Be(task.Title);

        var taskInDb = await _context.Tasks.FindAsync(result.Id);
        taskInDb.Should().NotBeNull();
    }

    [Fact]
    public async Task UpdateAsync_WithValidTask_ShouldUpdateTask()
    {
        // Arrange
        var task = Fixture.Create<TaskItem>();
        await _context.Tasks.AddAsync(task);
        await _context.SaveChangesAsync();

        var updatedTitle = "Updated Title";
        task.Title = updatedTitle;
        task.IsCompleted = !task.IsCompleted;

        // Act
        var result = await _repository.UpdateAsync(task);

        // Assert
        result.Should().NotBeNull();
        result.Title.Should().Be(updatedTitle);
        result.IsCompleted.Should().Be(task.IsCompleted);

        var taskInDb = await _context.Tasks.FindAsync(task.Id);
        taskInDb.Title.Should().Be(updatedTitle);
    }

    [Fact]
    public async Task DeleteAsync_WithValidId_ShouldRemoveTask()
    {
        // Arrange
        var task = Fixture.Create<TaskItem>();
        await _context.Tasks.AddAsync(task);
        await _context.SaveChangesAsync();

        // Act
        var result = await _repository.DeleteAsync(task.Id);

        // Assert
        result.Should().BeTrue();

        var taskInDb = await _context.Tasks.FindAsync(task.Id);
        taskInDb.Should().BeNull();
    }

    [Fact]
    public async Task DeleteAsync_WithInvalidId_ShouldReturnFalse()
    {
        // Act
        var result = await _repository.DeleteAsync(999);

        // Assert
        result.Should().BeFalse();
    }

    [Fact]
    public async Task GetCompletedTasksAsync_ShouldReturnOnlyCompletedTasks()
    {
        // Arrange
        var completedTasks = Fixture.Build<TaskItem>()
            .With(t => t.IsCompleted, true)
            .CreateMany(3)
            .ToList();

        var incompleteTasks = Fixture.Build<TaskItem>()
            .With(t => t.IsCompleted, false)
            .CreateMany(2)
            .ToList();

        await _context.Tasks.AddRangeAsync(completedTasks);
        await _context.Tasks.AddRangeAsync(incompleteTasks);
        await _context.SaveChangesAsync();

        // Act
        var result = await _repository.GetCompletedTasksAsync();

        // Assert
        result.Should().HaveCount(3);
        result.Should().OnlyContain(t => t.IsCompleted);
        result.Should().BeEquivalentTo(completedTasks);
    }

    [Fact]
    public async Task GetTasksByDateRangeAsync_ShouldReturnTasksInRange()
    {
        // Arrange
        var startDate = new DateTime(2024, 1, 1);
        var endDate = new DateTime(2024, 1, 31);

        var tasksInRange = Fixture.Build<TaskItem>()
            .With(t => t.CreatedAt, new DateTime(2024, 1, 15))
            .CreateMany(3)
            .ToList();

        var tasksOutOfRange = Fixture.Build<TaskItem>()
            .With(t => t.CreatedAt, new DateTime(2024, 2, 15))
            .CreateMany(2)
            .ToList();

        await _context.Tasks.AddRangeAsync(tasksInRange);
        await _context.Tasks.AddRangeAsync(tasksOutOfRange);
        await _context.SaveChangesAsync();

        // Act
        var result = await _repository.GetTasksByDateRangeAsync(startDate, endDate);

        // Assert
        result.Should().HaveCount(3);
        result.Should().OnlyContain(t => t.CreatedAt >= startDate && t.CreatedAt <= endDate);
        result.Should().BeEquivalentTo(tasksInRange);
    }

    [Fact]
    public async Task SearchTasksAsync_ShouldReturnMatchingTasks()
    {
        // Arrange
        var searchTerm = "important";
        var matchingTasks = new[]
        {
            Fixture.Build<TaskItem>().With(t => t.Title, "Important task 1").Create(),
            Fixture.Build<TaskItem>().With(t => t.Description, "This is important").Create(),
            Fixture.Build<TaskItem>().With(t => t.Title, "IMPORTANT TASK 2").Create()
        };

        var nonMatchingTasks = new[]
        {
            Fixture.Build<TaskItem>().With(t => t.Title, "Regular task").Create(),
            Fixture.Build<TaskItem>().With(t => t.Description, "Nothing special").Create()
        };

        await _context.Tasks.AddRangeAsync(matchingTasks);
        await _context.Tasks.AddRangeAsync(nonMatchingTasks);
        await _context.SaveChangesAsync();

        // Act
        var result = await _repository.SearchTasksAsync(searchTerm);

        // Assert
        result.Should().HaveCount(3);
        result.Should().BeEquivalentTo(matchingTasks);
    }

    public void Dispose()
    {
        _context?.Dispose();
    }
}
```

---

## 4. 🧪 Integration Testing

### 4.1 API Integration Tests

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\tests\IntegrationTests\TaskApiIntegrationTests.cs
using Microsoft.AspNetCore.Mvc.Testing;
using Microsoft.Extensions.DependencyInjection;
using System.Net.Http.Json;
using FluentAssertions;
using TestingExample.Models;

namespace IntegrationTests;

public class TaskApiIntegrationTests : BaseIntegrationTest
{
    public TaskApiIntegrationTests(WebApplicationFactory<Program> factory) : base(factory)
    {
    }

    [Fact]
    public async Task GetTasks_ShouldReturnAllTasks()
    {
        // Arrange
        await SeedTestDataAsync();

        // Act
        var response = await HttpClient.GetAsync("/api/tasks");

        // Assert
        response.Should().BeSuccessful();
        var tasks = await response.Content.ReadFromJsonAsync<List<TaskItem>>();
        tasks.Should().NotBeEmpty();
    }

    [Fact]
    public async Task CreateTask_WithValidData_ShouldCreateTask()
    {
        // Arrange
        var newTask = new TaskItem
        {
            Title = "Integration Test Task",
            Description = "Created by integration test",
            IsCompleted = false,
            CreatedAt = DateTime.UtcNow
        };

        // Act
        var response = await HttpClient.PostAsJsonAsync("/api/tasks", newTask);

        // Assert
        response.Should().BeSuccessful();
        var createdTask = await response.Content.ReadFromJsonAsync<TaskItem>();
        createdTask.Should().NotBeNull();
        createdTask.Id.Should().BeGreaterThan(0);
        createdTask.Title.Should().Be(newTask.Title);
    }

    [Fact]
    public async Task UpdateTask_WithValidData_ShouldUpdateTask()
    {
        // Arrange
        var task = await CreateTestTaskAsync();
        task.Title = "Updated Task Title";
        task.IsCompleted = true;

        // Act
        var response = await HttpClient.PutAsJsonAsync($"/api/tasks/{task.Id}", task);

        // Assert
        response.Should().BeSuccessful();
        var updatedTask = await response.Content.ReadFromJsonAsync<TaskItem>();
        updatedTask.Should().NotBeNull();
        updatedTask.Title.Should().Be("Updated Task Title");
        updatedTask.IsCompleted.Should().BeTrue();
    }

    [Fact]
    public async Task DeleteTask_WithValidId_ShouldDeleteTask()
    {
        // Arrange
        var task = await CreateTestTaskAsync();

        // Act
        var response = await HttpClient.DeleteAsync($"/api/tasks/{task.Id}");

        // Assert
        response.Should().BeSuccessful();

        // Verify task is deleted
        var getResponse = await HttpClient.GetAsync($"/api/tasks/{task.Id}");
        getResponse.StatusCode.Should().Be(System.Net.HttpStatusCode.NotFound);
    }

    [Fact]
    public async Task GetTask_WithInvalidId_ShouldReturnNotFound()
    {
        // Act
        var response = await HttpClient.GetAsync("/api/tasks/999999");

        // Assert
        response.StatusCode.Should().Be(System.Net.HttpStatusCode.NotFound);
    }

    [Fact]
    public async Task CreateTask_WithInvalidData_ShouldReturnBadRequest()
    {
        // Arrange
        var invalidTask = new TaskItem
        {
            Title = "", // Invalid - empty title
            Description = null,
            IsCompleted = false
        };

        // Act
        var response = await HttpClient.PostAsJsonAsync("/api/tasks", invalidTask);

        // Assert
        response.StatusCode.Should().Be(System.Net.HttpStatusCode.BadRequest);
    }

    private async Task<TaskItem> CreateTestTaskAsync()
    {
        var task = new TaskItem
        {
            Title = "Test Task " + DateTime.Now.Ticks,
            Description = "Test task description",
            IsCompleted = false,
            CreatedAt = DateTime.UtcNow
        };

        var response = await HttpClient.PostAsJsonAsync("/api/tasks", task);
        response.EnsureSuccessStatusCode();

        return await response.Content.ReadFromJsonAsync<TaskItem>();
    }

    private async Task SeedTestDataAsync()
    {
        var tasks = new[]
        {
            new TaskItem { Title = "Task 1", Description = "Description 1", IsCompleted = false },
            new TaskItem { Title = "Task 2", Description = "Description 2", IsCompleted = true },
            new TaskItem { Title = "Task 3", Description = "Description 3", IsCompleted = false }
        };

        foreach (var task in tasks)
        {
            await HttpClient.PostAsJsonAsync("/api/tasks", task);
        }
    }
}

public abstract class BaseIntegrationTest : IClassFixture<WebApplicationFactory<Program>>
{
    protected readonly WebApplicationFactory<Program> Factory;
    protected readonly HttpClient HttpClient;

    protected BaseIntegrationTest(WebApplicationFactory<Program> factory)
    {
        Factory = factory.WithWebHostBuilder(builder =>
        {
            builder.ConfigureServices(services =>
            {
                // Configure test services here
                // For example, replace database with in-memory version
            });
        });

        HttpClient = Factory.CreateClient();
    }
}
```

---

## 5. 📊 Code Coverage และ Test Reporting

### 5.1 Test Configuration

```json
<!-- d:\A-Work\learn from ai\maui-xaml-mastery\tests\UITests\appsettings.json -->
{
  "Platform": "Android",
  "AppiumServerUrl": "http://localhost:4723",
  "ImplicitWaitSeconds": 10,
  "Android": {
    "PlatformVersion": "13",
    "DeviceName": "Android Emulator",
    "AppPath": "C:\\path\\to\\your\\app.apk",
    "AppPackage": "com.companyname.testingexample",
    "AppActivity": "crc64...MainActivity"
  },
  "iOS": {
    "PlatformVersion": "17.0",
    "DeviceName": "iPhone 15 Simulator",
    "AppPath": "C:\\path\\to\\your\\app.app",
    "BundleId": "com.companyname.testingexample"
  },
  "Windows": {
    "AppPath": "C:\\path\\to\\your\\app.exe"
  }
}
```

### 5.2 Test Scripts

```bash
# d:\A-Work\learn from ai\maui-xaml-mastery\scripts\run-tests.sh
#!/bin/bash

echo "Starting test execution..."

# Set test environment
export TEST_ENVIRONMENT=Development

# Run unit tests with coverage
echo "Running unit tests..."
dotnet test tests/UnitTests/UnitTests.csproj \
  --configuration Release \
  --logger "trx;LogFileName=unit-tests.trx" \
  --collect:"XPlat Code Coverage" \
  --results-directory TestResults

# Generate coverage report
echo "Generating coverage report..."
dotnet tool install -g dotnet-reportgenerator-globaltool
reportgenerator \
  -reports:"TestResults/*/coverage.cobertura.xml" \
  -targetdir:"TestResults/CoverageReport" \
  -reporttypes:Html

# Run integration tests
echo "Running integration tests..."
dotnet test tests/IntegrationTests/IntegrationTests.csproj \
  --configuration Release \
  --logger "trx;LogFileName=integration-tests.trx" \
  --results-directory TestResults

# Start Appium server for UI tests
echo "Starting Appium server..."
appium --port 4723 --log-level error &
APPIUM_PID=$!

# Wait for Appium to start
sleep 10

# Run UI tests
echo "Running UI tests..."
dotnet test tests/UITests/UITests.csproj \
  --configuration Release \
  --logger "trx;LogFileName=ui-tests.trx" \
  --results-directory TestResults

# Stop Appium server
kill $APPIUM_PID

echo "All tests completed. Results available in TestResults directory."
```

---

## 6. 🧪 Practical Exercises

### Exercise 1: TDD Implementation
สร้าง feature ใหม่โดยใช้ TDD approach:
- เขียน failing tests ก่อน
- Implement minimum code to pass
- Refactor และ improve
- Repeat cycle

### Exercise 2: Advanced Mocking
สร้าง complex testing scenarios:
- Mock external dependencies
- Test async operations
- Verify method calls และ parameters
- Test error handling

### Exercise 3: Performance Testing
สร้าง performance test suite:
- Load testing สำหรับ data operations
- UI responsiveness testing
- Memory usage verification
- Network performance testing

---

## 7. 📝 Testing Best Practices

### 🎯 Unit Testing Best Practices

```csharp
// ✅ DO: Use descriptive test names
[Fact]
public async Task LoadTasksAsync_WhenServiceReturnsData_ShouldPopulateTasksCollection()
{
    // Test implementation
}

// ✅ DO: Follow AAA pattern (Arrange, Act, Assert)
[Fact]
public async Task AddTaskCommand_WithValidInput_ShouldAddTask()
{
    // Arrange
    var expectedTitle = "New Task";
    _viewModel.NewTaskTitle = expectedTitle;
    
    // Act
    await _viewModel.AddTaskCommand.ExecuteAsync(null);
    
    // Assert
    _viewModel.Tasks.Should().Contain(t => t.Title == expectedTitle);
}

// ✅ DO: Test one thing at a time
[Fact]
public void TaskTitle_WhenEmpty_ShouldReturnValidationError()
{
    // Focus on single behavior
}

// ✅ DO: Use object builders for complex objects
var task = Fixture.Build<TaskItem>()
    .With(t => t.IsCompleted, false)
    .With(t => t.Priority, TaskPriority.High)
    .Without(t => t.Id) // Don't set ID for new entities
    .Create();

// ❌ DON'T: Test implementation details
[Fact]
public void ShouldCallRepositoryGetAllMethod() // Bad - testing implementation
{
    // Don't test that specific methods are called
}

// ✅ DO: Test behavior and outcomes
[Fact]
public void ShouldReturnAllTasksWhenRequested() // Good - testing behavior
{
    // Test what the user cares about
}

// ✅ DO: Use meaningful assertions
result.Should().HaveCount(5);
result.Should().OnlyContain(t => t.IsCompleted);
result.Should().BeEquivalentTo(expectedTasks);

// ❌ DON'T: Use magic numbers
await Task.Delay(1000); // Bad - unclear why 1000ms

// ✅ DO: Use named constants
private const int NETWORK_TIMEOUT_MS = 1000;
await Task.Delay(NETWORK_TIMEOUT_MS);
```

### 🤖 UI Testing Best Practices

```csharp
// ✅ DO: Use Page Object Model
public class TaskListPage : BasePage
{
    private const string AddButtonId = "AddTaskButton";
    
    public void AddTask(string title)
    {
        EnterTaskTitle(title);
        TapAddButton();
    }
}

// ✅ DO: Wait for elements properly
protected void WaitForElement(string accessibilityId, int timeoutSeconds = 10)
{
    var wait = new WebDriverWait(Driver, TimeSpan.FromSeconds(timeoutSeconds));
    wait.Until(driver => FindElement(accessibilityId).Displayed);
}

// ✅ DO: Take screenshots on failures
try
{
    // Test steps
}
catch (Exception ex)
{
    TakeScreenshot($"test_failure_{DateTime.Now:yyyyMMdd_HHmmss}.png");
    throw;
}

// ✅ DO: Use stable locators
// Good - using AutomationId
FindElement(MobileBy.AccessibilityId("TaskItem_123"));

// Avoid - using text that might change
FindElement(MobileBy.XPath("//label[text()='Add Task']"));

// ✅ DO: Design for testability
<!-- In XAML, add AutomationId for testing -->
<Button x:Name="AddButton" 
        AutomationId="AddTaskButton"
        Text="Add Task" />
```

---

## 8. 🎯 Quiz & Assessment

### Quiz Questions

1. **TDD Process**: อธิบาย Red-Green-Refactor cycle ใน TDD

2. **Mock vs Stub**: ความแตกต่างระหว่าง Mock และ Stub คืออะไร?

3. **UI Testing**: เหตุผลที่ควรใช้ Page Object Model ใน UI testing

4. **Test Coverage**: ทำไม 100% code coverage ไม่ได้หมายความว่า testing ที่ดี?

### Practical Assessment
สร้าง **"Complete Testing Suite"** ที่มี:
- ✅ Comprehensive unit tests (>80% coverage)
- ✅ Integration tests สำหรับ API endpoints
- ✅ UI automation tests สำหรับ core workflows
- ✅ Performance tests
- ✅ CI/CD pipeline integration

---

## 9. 🚀 Next Steps

### Advanced Testing Topics
- **Contract Testing**: API contract testing with Pact
- **Load Testing**: Performance testing under load
- **Security Testing**: Automated security testing
- **Accessibility Testing**: Automated accessibility validation

### Chapter 16 Preview
หัวข้อต่อไป: **"Security & Authentication"**
- Authentication mechanisms
- Authorization strategies
- Data encryption
- Security best practices

---

## 📚 Additional Resources

### 🔗 Testing Tools
- [xUnit Documentation](https://xunit.net/)
- [FluentAssertions](https://fluentassertions.com/)
- [NSubstitute](https://nsubstitute.github.io/)
- [Appium Documentation](http://appium.io/)

### 📖 Recommended Reading
- "The Art of Unit Testing"
- "Test-Driven Development: By Example"
- "Growing Object-Oriented Software, Guided by Tests"

---

*หมายเหตุ: บทนี้เน้นการสร้าง comprehensive testing strategy ที่ช่วยให้ MAUI applications มีคุณภาพสูงและเชื่อถือได้*
