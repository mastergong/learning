# 🌐 Chapter 22: Hybrid Approaches

หลังจากที่เราได้เรียนรู้ advanced patterns และ architectural strategies แล้ว ในบทนี้เราจะมาเจาะลึกถึง hybrid approaches ที่ผสมผสานระหว่าง native MAUI กับ web technologies เช่น Blazor Hybrid และ WebView integration

## 🎯 Learning Objectives

- เรียนรู้ Blazor Hybrid development ใน .NET MAUI
- ใช้งาน WebView2 control สำหรับ web content integration
- สร้าง hybrid navigation และ communication systems
- เข้าใจ performance considerations สำหรับ hybrid apps
- ประยุกต์ใช้ PWA (Progressive Web App) concepts
- จัดการ security และ data sharing ระหว่าง native และ web

---

## 1. 🔥 Blazor Hybrid Overview

### 1.1 Setting Up Blazor Hybrid

#### Project Configuration

```xml
<!-- Project file setup for Blazor Hybrid -->
<Project Sdk="Microsoft.NET.Sdk.Razor">
  <PropertyGroup>
    <TargetFrameworks>net8.0-android;net8.0-ios;net8.0-maccatalyst;net8.0-windows10.0.19041.0</TargetFrameworks>
    <OutputType>Exe</OutputType>
    <RootNamespace>BlazorMauiApp</RootNamespace>
    <UseMaui>true</UseMaui>
    <SingleProject>true</SingleProject>
    <ImplicitUsings>enable</ImplicitUsings>
    <EnableDefaultCssItems>false</EnableDefaultCssItems>
    
    <!-- Include Razor compiler -->
    <AddRazorSupportForMvc>true</AddRazorSupportForMvc>
  </PropertyGroup>

  <ItemGroup>
    <!-- MAUI Essentials -->
    <PackageReference Include="Microsoft.Maui.Controls" Version="8.0.7" />
    <PackageReference Include="Microsoft.Maui.Controls.Compatibility" Version="8.0.7" />
    <PackageReference Include="Microsoft.AspNetCore.Components.WebView.Maui" Version="8.0.7" />
    <PackageReference Include="Microsoft.Extensions.Logging.Debug" Version="8.0.0" />
    
    <!-- Additional Blazor packages -->
    <PackageReference Include="Microsoft.AspNetCore.Components.Web" Version="8.0.0" />
    <PackageReference Include="Microsoft.AspNetCore.Components.Authorization" Version="8.0.0" />
  </ItemGroup>

  <ItemGroup>
    <MauiIcon Include="Resources\AppIcon\appicon.svg" ForegroundFile="Resources\AppIcon\appiconfg.svg" Color="#512BD4" />
    <MauiSplashScreen Include="Resources\Splash\splash.svg" Color="#512BD4" BaseSize="128,128" />
    
    <!-- Web assets -->
    <MauiAsset Include="Resources\Raw\**" LogicalName="%(RecursiveDir)%(Filename)%(Extension)" />
  </ItemGroup>
</Project>
```

#### MauiProgram Configuration

```csharp
// MauiProgram.cs
public static class MauiProgram
{
    public static MauiApp CreateMauiApp()
    {
        var builder = MauiApp.CreateBuilder();
        builder
            .UseMauiApp<App>()
            .ConfigureFonts(fonts =>
            {
                fonts.AddFont("OpenSans-Regular.ttf", "OpenSansRegular");
            });

        // Add Blazor WebView
        builder.Services.AddMauiBlazorWebView();
        
        // Configure logging
        builder.Services.AddLogging(configure =>
        {
            configure.AddDebug();
#if DEBUG
            configure.SetMinimumLevel(LogLevel.Debug);
#else
            configure.SetMinimumLevel(LogLevel.Information);
#endif
        });
        
        // Register services
        builder.Services.AddSingleton<WeatherForecastService>();
        builder.Services.AddSingleton<IDataService, DataService>();
        builder.Services.AddSingleton<INativeService, NativeService>();
        
        // Add HTTP client
        builder.Services.AddHttpClient();
        
        // Add authentication
        builder.Services.AddAuthorizationCore();
        builder.Services.AddScoped<AuthenticationStateProvider, CustomAuthenticationStateProvider>();

#if DEBUG
        builder.Services.AddBlazorWebViewDeveloperTools();
        builder.Logging.AddDebug();
#endif

        return builder.Build();
    }
}
```

### 1.2 Creating Blazor Hybrid Pages

#### Main Page with BlazorWebView

```xml
<!-- MainPage.xaml -->
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:local="clr-namespace:BlazorMauiApp"
             x:Class="BlazorMauiApp.MainPage"
             BackgroundColor="{DynamicResource PageBackgroundColor}">

    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto" />
            <RowDefinition Height="*" />
        </Grid.RowDefinitions>
        
        <!-- Native header with controls -->
        <Grid Grid.Row="0" 
              BackgroundColor="{DynamicResource PrimaryColor}"
              Padding="15">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="*" />
                <ColumnDefinition Width="Auto" />
                <ColumnDefinition Width="Auto" />
            </Grid.ColumnDefinitions>
            
            <Label Grid.Column="0"
                   Text="Blazor Hybrid App"
                   FontSize="18"
                   FontAttributes="Bold"
                   TextColor="White"
                   VerticalOptions="Center" />
                   
            <Button Grid.Column="1"
                    Text="Settings"
                    BackgroundColor="Transparent"
                    TextColor="White"
                    Clicked="OnSettingsClicked" />
                    
            <Button Grid.Column="2"
                    Text="Native"
                    BackgroundColor="Transparent"
                    TextColor="White"
                    Clicked="OnNativePageClicked" />
        </Grid>

        <!-- Blazor WebView -->
        <BlazorWebView Grid.Row="1" 
                       x:Name="blazorWebView"
                       HostPage="wwwroot/index.html">
            <BlazorWebView.RootComponents>
                <RootComponent Selector="#app" ComponentType="{x:Type local:Components.App}" />
            </BlazorWebView.RootComponents>
        </BlazorWebView>
    </Grid>
</ContentPage>
```

#### MainPage Code-Behind

```csharp
// MainPage.xaml.cs
public partial class MainPage : ContentPage
{
    private readonly INativeService _nativeService;
    
    public MainPage(INativeService nativeService)
    {
        InitializeComponent();
        _nativeService = nativeService;
        
        // Setup JavaScript bridge
        SetupJavaScriptBridge();
    }
    
    private void SetupJavaScriptBridge()
    {
        blazorWebView.BlazorWebViewInitialized += (sender, e) =>
        {
            // Add native functions to JavaScript
            e.WebView.CoreWebView2.AddWebResourceRequestedFilter("*", Microsoft.Web.WebView2.Core.CoreWebView2WebResourceContext.All);
            
            // Handle messages from Blazor
            blazorWebView.BlazorWebViewInitializing += OnBlazorWebViewInitializing;
        };
    }
    
    private void OnBlazorWebViewInitializing(object sender, BlazorWebViewInitializingEventArgs e)
    {
        // Configure WebView2 settings
        e.EnvironmentOptions.AdditionalBrowserArguments = "--disable-web-security --allow-running-insecure-content";
        
#if DEBUG
        e.EnvironmentOptions.AdditionalBrowserArguments += " --remote-debugging-port=9222";
#endif
    }
    
    private async void OnSettingsClicked(object sender, EventArgs e)
    {
        await Navigation.PushAsync(new SettingsPage());
    }
    
    private async void OnNativePageClicked(object sender, EventArgs e)
    {
        await Navigation.PushAsync(new NativePage());
    }
}
```

### 1.3 Blazor Components

#### App Component

```razor
@* Components/App.razor *@
<Router AppAssembly="@typeof(App).Assembly">
    <Found Context="routeData">
        <AuthorizeRouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)">
            <NotAuthorized>
                <RedirectToLogin />
            </NotAuthorized>
        </AuthorizeRouteView>
        <FocusOnNavigate RouteData="@routeData" Selector="h1" />
    </Found>
    <NotFound>
        <PageTitle>Not found</PageTitle>
        <LayoutView Layout="@typeof(MainLayout)">
            <p role="alert">Sorry, there's nothing at this address.</p>
        </LayoutView>
    </NotFound>
</Router>
```

#### Main Layout

```razor
@* Components/Layout/MainLayout.razor *@
@inherits LayoutView
@inject IJSRuntime JSRuntime
@inject INativeService NativeService

<div class="page">
    <div class="sidebar">
        <NavMenu />
    </div>

    <main>
        <div class="top-row px-4">
            <div class="d-flex align-items-center">
                <h3>Blazor Hybrid App</h3>
                <div class="ms-auto">
                    <button class="btn btn-primary" @onclick="ShowNativeAlert">
                        Native Alert
                    </button>
                    <button class="btn btn-success" @onclick="TakePhoto">
                        Take Photo
                    </button>
                </div>
            </div>
        </div>

        <article class="content px-4">
            @Body
        </article>
    </main>
</div>

@code {
    private async Task ShowNativeAlert()
    {
        await NativeService.ShowAlertAsync("Hello from Blazor!", "This alert is shown using native MAUI APIs");
    }
    
    private async Task TakePhoto()
    {
        try
        {
            var photo = await NativeService.TakePhotoAsync();
            if (photo != null)
            {
                await JSRuntime.InvokeVoidAsync("console.log", $"Photo taken: {photo.FullPath}");
                // Handle photo result
            }
        }
        catch (Exception ex)
        {
            await JSRuntime.InvokeVoidAsync("console.error", $"Error taking photo: {ex.Message}");
        }
    }
}
```

#### Home Page Component

```razor
@* Components/Pages/Home.razor *@
@page "/"
@inject IDataService DataService
@inject IJSRuntime JSRuntime
@inject ILogger<Home> Logger

<PageTitle>Home</PageTitle>

<div class="container-fluid">
    <div class="row">
        <div class="col-12">
            <h1>Welcome to Blazor Hybrid!</h1>
            <p>This page combines the power of Blazor with native MAUI capabilities.</p>
        </div>
    </div>
    
    <div class="row mt-4">
        <div class="col-md-6">
            <div class="card">
                <div class="card-header">
                    <h5>Native Capabilities</h5>
                </div>
                <div class="card-body">
                    <button class="btn btn-primary me-2" @onclick="GetDeviceInfo">
                        Get Device Info
                    </button>
                    <button class="btn btn-secondary me-2" @onclick="CheckConnectivity">
                        Check Connectivity
                    </button>
                    <button class="btn btn-success" @onclick="ShareContent">
                        Share Content
                    </button>
                    
                    @if (!string.IsNullOrEmpty(deviceInfo))
                    {
                        <div class="mt-3">
                            <h6>Device Information:</h6>
                            <pre class="bg-light p-2">@deviceInfo</pre>
                        </div>
                    }
                </div>
            </div>
        </div>
        
        <div class="col-md-6">
            <div class="card">
                <div class="card-header">
                    <h5>Data Operations</h5>
                </div>
                <div class="card-body">
                    <button class="btn btn-info me-2" @onclick="LoadData">
                        Load Data
                    </button>
                    <button class="btn btn-warning" @onclick="ClearData">
                        Clear Data
                    </button>
                    
                    @if (isLoading)
                    {
                        <div class="mt-3">
                            <div class="spinner-border" role="status">
                                <span class="visually-hidden">Loading...</span>
                            </div>
                            <span class="ms-2">Loading data...</span>
                        </div>
                    }
                    
                    @if (data != null && data.Any())
                    {
                        <div class="mt-3">
                            <h6>Loaded Data:</h6>
                            <ul class="list-group">
                                @foreach (var item in data)
                                {
                                    <li class="list-group-item">@item</li>
                                }
                            </ul>
                        </div>
                    }
                </div>
            </div>
        </div>
    </div>
    
    <div class="row mt-4">
        <div class="col-12">
            <div class="card">
                <div class="card-header">
                    <h5>JavaScript Interop</h5>
                </div>
                <div class="card-body">
                    <div class="row">
                        <div class="col-md-6">
                            <input class="form-control" @bind="messageToJs" placeholder="Enter message for JavaScript" />
                        </div>
                        <div class="col-md-6">
                            <button class="btn btn-primary" @onclick="CallJavaScript">
                                Call JavaScript
                            </button>
                        </div>
                    </div>
                    
                    @if (!string.IsNullOrEmpty(jsResult))
                    {
                        <div class="mt-3">
                            <strong>JavaScript Result:</strong> @jsResult
                        </div>
                    }
                </div>
            </div>
        </div>
    </div>
</div>

@code {
    private string deviceInfo = "";
    private List<string> data = new();
    private bool isLoading = false;
    private string messageToJs = "";
    private string jsResult = "";
    
    private async Task GetDeviceInfo()
    {
        try
        {
            var info = new StringBuilder();
            info.AppendLine($"Platform: {DeviceInfo.Platform}");
            info.AppendLine($"Model: {DeviceInfo.Model}");
            info.AppendLine($"Manufacturer: {DeviceInfo.Manufacturer}");
            info.AppendLine($"Name: {DeviceInfo.Name}");
            info.AppendLine($"Version: {DeviceInfo.VersionString}");
            info.AppendLine($"Idiom: {DeviceInfo.Idiom}");
            info.AppendLine($"Device Type: {DeviceInfo.DeviceType}");
            
            deviceInfo = info.ToString();
            StateHasChanged();
        }
        catch (Exception ex)
        {
            Logger.LogError(ex, "Error getting device info");
            await JSRuntime.InvokeVoidAsync("console.error", $"Error: {ex.Message}");
        }
    }
    
    private async Task CheckConnectivity()
    {
        try
        {
            var connectivity = Connectivity.NetworkAccess;
            var profiles = Connectivity.ConnectionProfiles;
            
            var message = $"Network Access: {connectivity}\nProfiles: {string.Join(", ", profiles)}";
            await JSRuntime.InvokeVoidAsync("alert", message);
        }
        catch (Exception ex)
        {
            Logger.LogError(ex, "Error checking connectivity");
        }
    }
    
    private async Task ShareContent()
    {
        try
        {
            await Share.RequestAsync(new ShareTextRequest
            {
                Text = "Check out this amazing Blazor Hybrid app!",
                Title = "Share Blazor Hybrid App"
            });
        }
        catch (Exception ex)
        {
            Logger.LogError(ex, "Error sharing content");
        }
    }
    
    private async Task LoadData()
    {
        isLoading = true;
        StateHasChanged();
        
        try
        {
            data = await DataService.GetDataAsync();
        }
        catch (Exception ex)
        {
            Logger.LogError(ex, "Error loading data");
            await JSRuntime.InvokeVoidAsync("console.error", $"Error loading data: {ex.Message}");
        }
        finally
        {
            isLoading = false;
            StateHasChanged();
        }
    }
    
    private void ClearData()
    {
        data.Clear();
        StateHasChanged();
    }
    
    private async Task CallJavaScript()
    {
        try
        {
            jsResult = await JSRuntime.InvokeAsync<string>("processMessage", messageToJs);
            StateHasChanged();
        }
        catch (Exception ex)
        {
            Logger.LogError(ex, "Error calling JavaScript");
            jsResult = $"Error: {ex.Message}";
            StateHasChanged();
        }
    }
}
```

---

## 2. 🌐 WebView Integration

### 2.1 Advanced WebView Control

#### Custom WebView Page

```xml
<!-- Views/WebViewPage.xaml -->
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="HybridApp.Views.WebViewPage"
             Title="WebView Integration">
    
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto" />
            <RowDefinition Height="*" />
            <RowDefinition Height="Auto" />
        </Grid.RowDefinitions>
        
        <!-- Navigation Bar -->
        <Grid Grid.Row="0" BackgroundColor="{StaticResource PrimaryColor}" Padding="10">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="Auto" />
                <ColumnDefinition Width="Auto" />
                <ColumnDefinition Width="*" />
                <ColumnDefinition Width="Auto" />
                <ColumnDefinition Width="Auto" />
            </Grid.ColumnDefinitions>
            
            <Button Grid.Column="0" 
                    Text="←" 
                    BackgroundColor="Transparent"
                    TextColor="White"
                    Command="{Binding GoBackCommand}" />
                    
            <Button Grid.Column="1" 
                    Text="→" 
                    BackgroundColor="Transparent"
                    TextColor="White"
                    Command="{Binding GoForwardCommand}" />
                    
            <Entry Grid.Column="2" 
                   Text="{Binding CurrentUrl, Mode=TwoWay}"
                   Placeholder="Enter URL..."
                   BackgroundColor="White"
                   Margin="10,0" />
                   
            <Button Grid.Column="3" 
                    Text="Go" 
                    BackgroundColor="Transparent"
                    TextColor="White"
                    Command="{Binding NavigateCommand}" />
                    
            <Button Grid.Column="4" 
                    Text="⟳" 
                    BackgroundColor="Transparent"
                    TextColor="White"
                    Command="{Binding RefreshCommand}" />
        </Grid>
        
        <!-- WebView -->
        <WebView Grid.Row="1" 
                 x:Name="webView"
                 Source="{Binding CurrentUrl}"
                 Navigating="OnNavigating"
                 Navigated="OnNavigated">
        </WebView>
        
        <!-- Status Bar -->
        <Grid Grid.Row="2" 
              BackgroundColor="{StaticResource SecondaryColor}" 
              Padding="10"
              IsVisible="{Binding IsLoading}">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="Auto" />
                <ColumnDefinition Width="*" />
                <ColumnDefinition Width="Auto" />
            </Grid.ColumnDefinitions>
            
            <ActivityIndicator Grid.Column="0" 
                             IsRunning="{Binding IsLoading}"
                             Color="{StaticResource PrimaryColor}" />
                             
            <Label Grid.Column="1" 
                   Text="{Binding StatusMessage}"
                   VerticalOptions="Center"
                   Margin="10,0" />
                   
            <Label Grid.Column="2" 
                   Text="{Binding LoadingProgress, StringFormat='{0:P0}'}"
                   VerticalOptions="Center" />
        </Grid>
    </Grid>
</ContentPage>
```

#### WebView Page Code-Behind

```csharp
// Views/WebViewPage.xaml.cs
public partial class WebViewPage : ContentPage
{
    private readonly WebViewPageViewModel _viewModel;
    
    public WebViewPage(WebViewPageViewModel viewModel)
    {
        InitializeComponent();
        _viewModel = viewModel;
        BindingContext = viewModel;
        
        SetupWebViewBridge();
    }
    
    private void SetupWebViewBridge()
    {
        // Add JavaScript bridge for communication
        var html = @"
            <script>
                window.nativeHost = {
                    showAlert: function(message) {
                        window.location = 'native://alert?message=' + encodeURIComponent(message);
                    },
                    
                    getDeviceInfo: function() {
                        window.location = 'native://deviceinfo';
                    },
                    
                    shareContent: function(text) {
                        window.location = 'native://share?text=' + encodeURIComponent(text);
                    },
                    
                    openCamera: function() {
                        window.location = 'native://camera';
                    }
                };
                
                // Log function for debugging
                window.logToNative = function(message) {
                    window.location = 'native://log?message=' + encodeURIComponent(message);
                };
            </script>
        ";
        
        // Inject JavaScript into WebView
        webView.EvaluateJavaScriptAsync(html);
    }
    
    private async void OnNavigating(object sender, WebNavigatingEventArgs e)
    {
        if (e.Url.StartsWith("native://"))
        {
            e.Cancel = true;
            await HandleNativeCall(e.Url);
            return;
        }
        
        _viewModel.IsLoading = true;
        _viewModel.StatusMessage = $"Loading {e.Url}...";
    }
    
    private void OnNavigated(object sender, WebNavigatedEventArgs e)
    {
        _viewModel.IsLoading = false;
        _viewModel.StatusMessage = "Ready";
        _viewModel.CurrentUrl = e.Url;
        
        // Update navigation button states
        _viewModel.CanGoBack = webView.CanGoBack;
        _viewModel.CanGoForward = webView.CanGoForward;
    }
    
    private async Task HandleNativeCall(string url)
    {
        var uri = new Uri(url);
        var command = uri.Host;
        var query = HttpUtility.ParseQueryString(uri.Query);
        
        switch (command)
        {
            case "alert":
                var message = query["message"];
                await DisplayAlert("Web Alert", message, "OK");
                break;
                
            case "deviceinfo":
                var deviceInfo = $"Platform: {DeviceInfo.Platform}\nModel: {DeviceInfo.Model}\nVersion: {DeviceInfo.VersionString}";
                await webView.EvaluateJavaScriptAsync($"alert('Device Info:\\n{deviceInfo}');");
                break;
                
            case "share":
                var text = query["text"];
                await Share.RequestAsync(new ShareTextRequest
                {
                    Text = text,
                    Title = "Shared from WebView"
                });
                break;
                
            case "camera":
                await OpenCamera();
                break;
                
            case "log":
                var logMessage = query["message"];
                System.Diagnostics.Debug.WriteLine($"WebView Log: {logMessage}");
                break;
        }
    }
    
    private async Task OpenCamera()
    {
        try
        {
            var photo = await MediaPicker.CapturePhotoAsync();
            if (photo != null)
            {
                // Convert photo to base64 and send to WebView
                using var stream = await photo.OpenReadAsync();
                using var memoryStream = new MemoryStream();
                await stream.CopyToAsync(memoryStream);
                var base64 = Convert.ToBase64String(memoryStream.ToArray());
                
                await webView.EvaluateJavaScriptAsync($@"
                    if (window.onPhotoTaken) {{
                        window.onPhotoTaken('data:image/jpeg;base64,{base64}');
                    }}
                ");
            }
        }
        catch (Exception ex)
        {
            await DisplayAlert("Error", $"Failed to take photo: {ex.Message}", "OK");
        }
    }
}
```

### 2.2 WebView ViewModel

```csharp
// ViewModels/WebViewPageViewModel.cs
public class WebViewPageViewModel : ViewModelBase
{
    private string _currentUrl = "https://www.example.com";
    private bool _isLoading;
    private string _statusMessage = "Ready";
    private double _loadingProgress;
    private bool _canGoBack;
    private bool _canGoForward;
    
    public WebViewPageViewModel(INavigationService navigationService) : base(navigationService)
    {
        Title = "WebView Browser";
        
        GoBackCommand = new RelayCommand(GoBack, () => CanGoBack);
        GoForwardCommand = new RelayCommand(GoForward, () => CanGoForward);
        NavigateCommand = new RelayCommand(Navigate);
        RefreshCommand = new RelayCommand(Refresh);
    }
    
    public string CurrentUrl
    {
        get => _currentUrl;
        set => SetProperty(ref _currentUrl, value);
    }
    
    public bool IsLoading
    {
        get => _isLoading;
        set => SetProperty(ref _isLoading, value);
    }
    
    public string StatusMessage
    {
        get => _statusMessage;
        set => SetProperty(ref _statusMessage, value);
    }
    
    public double LoadingProgress
    {
        get => _loadingProgress;
        set => SetProperty(ref _loadingProgress, value);
    }
    
    public bool CanGoBack
    {
        get => _canGoBack;
        set
        {
            if (SetProperty(ref _canGoBack, value))
            {
                ((RelayCommand)GoBackCommand).NotifyCanExecuteChanged();
            }
        }
    }
    
    public bool CanGoForward
    {
        get => _canGoForward;
        set
        {
            if (SetProperty(ref _canGoForward, value))
            {
                ((RelayCommand)GoForwardCommand).NotifyCanExecuteChanged();
            }
        }
    }
    
    public ICommand GoBackCommand { get; }
    public ICommand GoForwardCommand { get; }
    public ICommand NavigateCommand { get; }
    public ICommand RefreshCommand { get; }
    
    private void GoBack()
    {
        // This will be handled by the WebView control
        MessagingCenter.Send(this, "WebViewGoBack");
    }
    
    private void GoForward()
    {
        // This will be handled by the WebView control
        MessagingCenter.Send(this, "WebViewGoForward");
    }
    
    private void Navigate()
    {
        if (!string.IsNullOrWhiteSpace(CurrentUrl))
        {
            // Ensure URL has protocol
            if (!CurrentUrl.StartsWith("http://") && !CurrentUrl.StartsWith("https://"))
            {
                CurrentUrl = "https://" + CurrentUrl;
            }
            
            MessagingCenter.Send(this, "WebViewNavigate", CurrentUrl);
        }
    }
    
    private void Refresh()
    {
        MessagingCenter.Send(this, "WebViewRefresh");
    }
}
```

---

## 3. 🔗 Native-Web Communication

### 3.1 JavaScript Bridge Service

```csharp
// Services/IJavaScriptBridgeService.cs
public interface IJavaScriptBridgeService
{
    Task<string> CallJavaScriptAsync(string function, params object[] args);
    Task RegisterJavaScriptFunctionAsync(string functionName, Func<string[], Task<object>> handler);
    Task InjectJavaScriptAsync(string script);
    event EventHandler<JavaScriptCallEventArgs> JavaScriptCallReceived;
}

// Services/JavaScriptBridgeService.cs
public class JavaScriptBridgeService : IJavaScriptBridgeService
{
    private readonly Dictionary<string, Func<string[], Task<object>>> _registeredFunctions = new();
    private WebView _currentWebView;
    
    public event EventHandler<JavaScriptCallEventArgs> JavaScriptCallReceived;
    
    public void SetWebView(WebView webView)
    {
        _currentWebView = webView;
    }
    
    public async Task<string> CallJavaScriptAsync(string function, params object[] args)
    {
        if (_currentWebView == null)
            throw new InvalidOperationException("WebView not set");
            
        var argsJson = JsonSerializer.Serialize(args);
        var script = $"{function}(...{argsJson})";
        
        return await _currentWebView.EvaluateJavaScriptAsync(script);
    }
    
    public async Task RegisterJavaScriptFunctionAsync(string functionName, Func<string[], Task<object>> handler)
    {
        _registeredFunctions[functionName] = handler;
        
        // Inject the function into the WebView
        var script = $@"
            window.{functionName} = function(...args) {{
                var callId = Date.now() + Math.random();
                window.location = 'native://call?function={functionName}&args=' + encodeURIComponent(JSON.stringify(args)) + '&callId=' + callId;
                return new Promise((resolve, reject) => {{
                    window['resolve_' + callId] = resolve;
                    window['reject_' + callId] = reject;
                }});
            }};
        ";
        
        await InjectJavaScriptAsync(script);
    }
    
    public async Task InjectJavaScriptAsync(string script)
    {
        if (_currentWebView == null)
            throw new InvalidOperationException("WebView not set");
            
        await _currentWebView.EvaluateJavaScriptAsync(script);
    }
    
    public async Task HandleNativeCall(string url)
    {
        var uri = new Uri(url);
        var query = HttpUtility.ParseQueryString(uri.Query);
        
        if (uri.Host == "call")
        {
            var functionName = query["function"];
            var argsJson = query["args"];
            var callId = query["callId"];
            
            if (_registeredFunctions.TryGetValue(functionName, out var handler))
            {
                try
                {
                    var args = JsonSerializer.Deserialize<string[]>(argsJson);
                    var result = await handler(args);
                    var resultJson = JsonSerializer.Serialize(result);
                    
                    // Resolve the promise in JavaScript
                    await _currentWebView.EvaluateJavaScriptAsync($@"
                        if (window['resolve_{callId}']) {{
                            window['resolve_{callId}']({resultJson});
                            delete window['resolve_{callId}'];
                            delete window['reject_{callId}'];
                        }}
                    ");
                }
                catch (Exception ex)
                {
                    // Reject the promise in JavaScript
                    await _currentWebView.EvaluateJavaScriptAsync($@"
                        if (window['reject_{callId}']) {{
                            window['reject_{callId}']({JsonSerializer.Serialize(ex.Message)});
                            delete window['resolve_{callId}'];
                            delete window['reject_{callId}'];
                        }}
                    ");
                }
            }
        }
        
        JavaScriptCallReceived?.Invoke(this, new JavaScriptCallEventArgs(url));
    }
}

public class JavaScriptCallEventArgs : EventArgs
{
    public string Url { get; }
    
    public JavaScriptCallEventArgs(string url)
    {
        Url = url;
    }
}
```

### 3.2 Native Services for Web

```csharp
// Services/INativeService.cs
public interface INativeService
{
    Task ShowAlertAsync(string title, string message);
    Task<FileResult> TakePhotoAsync();
    Task<string> GetDeviceInfoAsync();
    Task ShareTextAsync(string text, string title = null);
    Task<bool> CheckPermissionAsync<T>() where T : Permissions.BasePermission, new();
    Task<PermissionStatus> RequestPermissionAsync<T>() where T : Permissions.BasePermission, new();
}

// Services/NativeService.cs
public class NativeService : INativeService
{
    private readonly ILogger<NativeService> _logger;
    
    public NativeService(ILogger<NativeService> logger)
    {
        _logger = logger;
    }
    
    public async Task ShowAlertAsync(string title, string message)
    {
        await Application.Current.MainPage.DisplayAlert(title, message, "OK");
    }
    
    public async Task<FileResult> TakePhotoAsync()
    {
        try
        {
            var permission = await RequestPermissionAsync<Permissions.Camera>();
            if (permission != PermissionStatus.Granted)
            {
                await ShowAlertAsync("Permission Denied", "Camera permission is required to take photos.");
                return null;
            }
            
            var photo = await MediaPicker.CapturePhotoAsync(new MediaPickerOptions
            {
                Title = "Take a photo"
            });
            
            return photo;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error taking photo");
            await ShowAlertAsync("Error", $"Failed to take photo: {ex.Message}");
            return null;
        }
    }
    
    public async Task<string> GetDeviceInfoAsync()
    {
        await Task.CompletedTask;
        
        var info = new
        {
            Platform = DeviceInfo.Platform.ToString(),
            Model = DeviceInfo.Model,
            Manufacturer = DeviceInfo.Manufacturer,
            Name = DeviceInfo.Name,
            VersionString = DeviceInfo.VersionString,
            Idiom = DeviceInfo.Idiom.ToString(),
            DeviceType = DeviceInfo.DeviceType.ToString()
        };
        
        return JsonSerializer.Serialize(info, new JsonSerializerOptions { WriteIndented = true });
    }
    
    public async Task ShareTextAsync(string text, string title = null)
    {
        try
        {
            await Share.RequestAsync(new ShareTextRequest
            {
                Text = text,
                Title = title ?? "Share"
            });
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error sharing text");
            await ShowAlertAsync("Error", $"Failed to share: {ex.Message}");
        }
    }
    
    public async Task<bool> CheckPermissionAsync<T>() where T : Permissions.BasePermission, new()
    {
        var permission = await Permissions.CheckStatusAsync<T>();
        return permission == PermissionStatus.Granted;
    }
    
    public async Task<PermissionStatus> RequestPermissionAsync<T>() where T : Permissions.BasePermission, new()
    {
        return await Permissions.RequestAsync<T>();
    }
}
```

---

## 4. 🎨 Progressive Web App (PWA) Features

### 4.1 PWA Configuration

#### Web Manifest

```json
// wwwroot/manifest.json
{
  "name": "Blazor Hybrid App",
  "short_name": "BlazorHybrid",
  "description": "A hybrid app combining Blazor and native MAUI capabilities",
  "start_url": "/",
  "scope": "/",
  "display": "standalone",
  "orientation": "portrait",
  "theme_color": "#512bd4",
  "background_color": "#ffffff",
  "icons": [
    {
      "src": "icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "maskable"
    },
    {
      "src": "icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any"
    }
  ],
  "categories": ["productivity", "utilities"],
  "lang": "en-US",
  "dir": "ltr"
}
```

#### Service Worker

```javascript
// wwwroot/sw.js
const CACHE_NAME = 'blazor-hybrid-v1';
const urlsToCache = [
  '/',
  '/css/app.css',
  '/js/app.js',
  '/manifest.json',
  '/_framework/blazor.webview.js'
];

// Install event
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => {
        console.log('Opened cache');
        return cache.addAll(urlsToCache);
      })
  );
});

// Fetch event
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request)
      .then(response => {
        // Return cached version or fetch from network
        return response || fetch(event.request);
      })
  );
});

// Activate event
self.addEventListener('activate', event => {
  event.waitUntil(
    caches.keys().then(cacheNames => {
      return Promise.all(
        cacheNames.map(cacheName => {
          if (cacheName !== CACHE_NAME) {
            console.log('Deleting old cache:', cacheName);
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});

// Background sync for offline functionality
self.addEventListener('sync', event => {
  if (event.tag === 'background-sync') {
    event.waitUntil(syncData());
  }
});

async function syncData() {
  try {
    // Implement your background sync logic here
    console.log('Background sync completed');
  } catch (error) {
    console.error('Background sync failed:', error);
  }
}
```

### 4.2 Offline Support Service

```csharp
// Services/IOfflineService.cs
public interface IOfflineService
{
    Task<bool> IsOnlineAsync();
    Task CacheDataAsync<T>(string key, T data);
    Task<T> GetCachedDataAsync<T>(string key);
    Task SyncDataAsync();
    event EventHandler<ConnectivityChangedEventArgs> ConnectivityChanged;
}

// Services/OfflineService.cs
public class OfflineService : IOfflineService
{
    private readonly ILogger<OfflineService> _logger;
    private readonly ISecureStorage _secureStorage;
    private readonly Queue<PendingAction> _pendingActions = new();
    
    public event EventHandler<ConnectivityChangedEventArgs> ConnectivityChanged;
    
    public OfflineService(ILogger<OfflineService> logger, ISecureStorage secureStorage)
    {
        _logger = logger;
        _secureStorage = secureStorage;
        
        Connectivity.ConnectivityChanged += OnConnectivityChanged;
    }
    
    public async Task<bool> IsOnlineAsync()
    {
        await Task.CompletedTask;
        return Connectivity.NetworkAccess == NetworkAccess.Internet;
    }
    
    public async Task CacheDataAsync<T>(string key, T data)
    {
        try
        {
            var json = JsonSerializer.Serialize(data);
            await _secureStorage.SetAsync($"cache_{key}", json);
            _logger.LogDebug("Cached data for key: {Key}", key);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to cache data for key: {Key}", key);
        }
    }
    
    public async Task<T> GetCachedDataAsync<T>(string key)
    {
        try
        {
            var json = await _secureStorage.GetAsync($"cache_{key}");
            if (!string.IsNullOrEmpty(json))
            {
                return JsonSerializer.Deserialize<T>(json);
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to get cached data for key: {Key}", key);
        }
        
        return default(T);
    }
    
    public async Task SyncDataAsync()
    {
        if (!await IsOnlineAsync())
        {
            _logger.LogWarning("Cannot sync data - device is offline");
            return;
        }
        
        _logger.LogInformation("Starting data synchronization");
        
        while (_pendingActions.Count > 0)
        {
            var action = _pendingActions.Dequeue();
            try
            {
                await ExecutePendingActionAsync(action);
                _logger.LogDebug("Synchronized pending action: {ActionType}", action.Type);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Failed to sync pending action: {ActionType}", action.Type);
                // Re-queue the action for retry
                _pendingActions.Enqueue(action);
                break;
            }
        }
        
        _logger.LogInformation("Data synchronization completed");
    }
    
    private void OnConnectivityChanged(object sender, ConnectivityChangedEventArgs e)
    {
        ConnectivityChanged?.Invoke(this, e);
        
        if (e.NetworkAccess == NetworkAccess.Internet)
        {
            Task.Run(async () => await SyncDataAsync());
        }
    }
    
    private async Task ExecutePendingActionAsync(PendingAction action)
    {
        // Implement your synchronization logic based on action type
        switch (action.Type)
        {
            case "CREATE":
                // Send create request to server
                break;
            case "UPDATE":
                // Send update request to server
                break;
            case "DELETE":
                // Send delete request to server
                break;
        }
        
        await Task.CompletedTask;
    }
    
    public void QueueAction(string type, object data)
    {
        _pendingActions.Enqueue(new PendingAction
        {
            Type = type,
            Data = JsonSerializer.Serialize(data),
            Timestamp = DateTime.UtcNow
        });
    }
}

public class PendingAction
{
    public string Type { get; set; }
    public string Data { get; set; }
    public DateTime Timestamp { get; set; }
}
```

---

## 5. 🧩 Practical Exercise

### 🎯 Building a Complete Hybrid E-Commerce App

สร้าง hybrid e-commerce application ที่ผสมผสาน Blazor และ native MAUI:

1. **Setup Hybrid Architecture:**
   - สร้าง Blazor Hybrid project
   - เพิ่ม WebView integration
   - ตั้งค่า JavaScript bridge

2. **Create Product Catalog:**
   - ใช้ Blazor components สำหรับ product listing
   - ใช้ native MAUI สำหรับ camera และ barcode scanning
   - สร้าง shopping cart functionality

3. **Implement User Features:**
   - Authentication ด้วย Blazor
   - Profile management ใน native pages
   - Order history และ tracking

4. **Add Offline Support:**
   - Cache product data
   - Queue actions when offline
   - Sync data when online

5. **Native Integration:**
   - Push notifications
   - Camera for product photos
   - GPS for store locator
   - Share functionality

### Model Implementation:

```csharp
// Product model
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }
    public string ImageUrl { get; set; }
    public string Category { get; set; }
    public bool InStock { get; set; }
}

// Shopping cart service
public interface IShoppingCartService
{
    Task AddToCartAsync(Product product, int quantity = 1);
    Task RemoveFromCartAsync(int productId);
    Task<List<CartItem>> GetCartItemsAsync();
    Task ClearCartAsync();
    Task<decimal> GetTotalAsync();
}
```

---

## 📚 Summary

ในบทนี้เราได้เรียนรู้:

### 🔑 Key Concepts
- **Blazor Hybrid**: การผสมผสาน Blazor กับ native MAUI capabilities
- **WebView Integration**: การใช้ WebView สำหรับ web content ใน native apps
- **JavaScript Bridge**: Communication ระหว่าง native code และ web content
- **PWA Features**: Progressive Web App capabilities ใน hybrid apps

### 🛠️ Technical Implementation
- **Project Setup**: Configuration สำหรับ Blazor Hybrid projects
- **Component Architecture**: การสร้าง Blazor components ที่ integrate กับ native
- **Service Integration**: การใช้ native services จาก Blazor components
- **Offline Support**: Caching และ synchronization strategies

### 🌐 Web Technologies
- **Service Workers**: Background processing และ caching
- **Web Manifest**: PWA configuration และ installability
- **JavaScript Interop**: Two-way communication between .NET และ JavaScript
- **Responsive Design**: Adaptive UI สำหรับ different screen sizes

### 📱 Native Integration
- **Platform APIs**: การเข้าถึง device capabilities จาก web code
- **Performance**: Optimization techniques สำหรับ hybrid apps
- **Security**: Safe communication between web และ native contexts
- **User Experience**: Seamless integration ระหว่าง web และ native UI

การใช้ hybrid approaches ช่วยให้เราสามารถใช้ประโยชน์จากทั้ง web technologies และ native capabilities ได้อย่างเต็มที่ ทำให้การพัฒนาแอปพลิเคชันมีความยืดหยุ่นและประสิทธิภาพมากขึ้น

**Next Chapter Preview:** ในบทต่อไป เราจะเรียนรู้เรื่อง **Enterprise Features** รวมถึง security, analytics และ monitoring systems! 🏢
