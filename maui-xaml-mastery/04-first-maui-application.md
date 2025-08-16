# 📱 บทที่ 4: First MAUI Application
> **สร้างแอปพลิเคชัน MAUI แรกของคุณ**

---

## 📋 สารบัญ
1. [Creating New Project](#creating-new-project)
2. [Project Structure Deep Dive](#project-structure-deep-dive)
3. [MainPage Implementation](#mainpage-implementation)
4. [Navigation Fundamentals](#navigation-fundamentals)
5. [Data Management](#data-management)
6. [UI Styling](#ui-styling)
7. [Platform-Specific Features](#platform-specific-features)
8. [Testing & Debugging](#testing-debugging)

---

## 🎯 Learning Objectives

เมื่อจบบทเรียนนี้ คุณจะสามารถ:
- ✅ สร้างและจัดการ MAUI project ใหม่
- ✅ เข้าใจโครงสร้างของ MAUI application 
- ✅ สร้าง UI หน้าแรกพร้อมการนำทาง
- ✅ จัดการข้อมูลและ state ในแอป
- ✅ ใช้ CSS-like approaches ในการ styling
- ✅ Test และ debug แอปบนอุปกรณ์ต่างๆ

---

## 🚀 Creating New Project

### การสร้าง Project ใหม่

```xml
<!-- ใช้ Visual Studio 2022 -->
File → New → Project → .NET Multi-platform App UI

<!-- หรือใช้ CLI -->
dotnet new maui -n MyFirstMauiApp
cd MyFirstMauiApp
```

### Project Configuration

```xml
<!-- MyFirstMauiApp.csproj -->
<Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <TargetFrameworks>net8.0-android;net8.0-ios;net8.0-maccatalyst</TargetFrameworks>
        <TargetFrameworks Condition="$([MSBuild]::IsOSPlatform('windows'))">$(TargetFrameworks);net8.0-windows10.0.19041.0</TargetFrameworks>
        
        <!-- Display Version -->
        <DisplayVersion>1.0</DisplayVersion>
        <ApplicationDisplayVersion>1.0</ApplicationDisplayVersion>
        <ApplicationVersion>1</ApplicationVersion>
        
        <OutputType>Exe</OutputType>
        <RootNamespace>MyFirstMauiApp</RootNamespace>
        <UseMaui>true</UseMaui>
        <SingleProject>true</SingleProject>
        <ImplicitUsings>enable</ImplicitUsings>
        
        <!-- Application -->
        <ApplicationTitle>My First MAUI App</ApplicationTitle>
        <ApplicationId>com.yourcompany.myfirstmauiapp</ApplicationId>
        <ApplicationIdGuid>C2482DA0-9DBF-4B2F-9E0A-123456789ABC</ApplicationIdGuid>
        
        <!-- Display -->
        <ApplicationDisplayVersion>1.0</ApplicationDisplayVersion>
        <ApplicationVersion>1</ApplicationVersion>
        
        <!-- Platform Versions -->
        <SupportedOSPlatformVersion Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'android'">21.0</SupportedOSPlatformVersion>
        <SupportedOSPlatformVersion Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'ios'">11.0</SupportedOSPlatformVersion>
        <SupportedOSPlatformVersion Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'maccatalyst'">13.1</SupportedOSPlatformVersion>
        <SupportedOSPlatformVersion Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'windows'">10.0.17763.0</SupportedOSPlatformVersion>
        <TargetPlatformMinVersion Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'windows'">10.0.17763.0</TargetPlatformMinVersion>
        <SupportedOSPlatformVersion Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'tizen'">6.5</SupportedOSPlatformVersion>
    </PropertyGroup>

    <ItemGroup>
        <!-- App Icon -->
        <MauiIcon Include="Resources\AppIcon\appicon.svg" ForegroundFile="Resources\AppIcon\appiconfg.svg" Color="#512BD4" />
        
        <!-- Splash Screen -->
        <MauiSplashScreen Include="Resources\Splash\splash.svg" Color="#512BD4" BaseSize="128,128" />
        
        <!-- Images -->
        <MauiImage Include="Resources\Images\*" />
        <MauiImage Update="Resources\Images\dotnet_bot.svg" BaseSize="168,208" />
        
        <!-- Custom Fonts -->
        <MauiFont Include="Resources\Fonts\*" />
        
        <!-- Raw Assets (also remove the "Resources\Raw" prefix) -->
        <MauiAsset Include="Resources\Raw\**" LogicalName="%(RecursiveDir)%(Filename)%(Extension)" />
    </ItemGroup>

    <ItemGroup>
        <PackageReference Include="Microsoft.Maui.Controls" Version="$(MauiVersion)" />
        <PackageReference Include="Microsoft.Maui.Controls.Compatibility" Version="$(MauiVersion)" />
        <PackageReference Include="Microsoft.Extensions.Logging.Debug" Version="8.0.0" />
        <PackageReference Include="CommunityToolkit.Mvvm" Version="8.2.2" />
    </ItemGroup>

</Project>
```

---

## 📁 Project Structure Deep Dive

### ไฟล์หลักและหน้าที่

```
MyFirstMauiApp/
├── 📁 Platforms/              # Platform-specific code
│   ├── 📁 Android/
│   ├── 📁 iOS/
│   ├── 📁 MacCatalyst/
│   ├── 📁 Windows/
│   └── 📁 Tizen/
├── 📁 Resources/              # App resources
│   ├── 📁 AppIcon/           # App icon
│   ├── 📁 Fonts/             # Custom fonts
│   ├── 📁 Images/            # Images
│   ├── 📁 Raw/               # Raw assets
│   └── 📁 Splash/            # Splash screen
├── 📁 Views/                  # XAML pages
├── 📁 ViewModels/            # View models (MVVM)
├── 📁 Models/                # Data models
├── 📁 Services/              # Business logic
├── 📄 App.xaml               # Application-level resources
├── 📄 App.xaml.cs            # Application class
├── 📄 AppShell.xaml          # Shell navigation
├── 📄 AppShell.xaml.cs       # Shell code-behind
├── 📄 MainPage.xaml          # Main page UI
├── 📄 MainPage.xaml.cs       # Main page logic
└── 📄 MauiProgram.cs         # App configuration
```

### MauiProgram.cs Configuration

```csharp
// MauiProgram.cs
using Microsoft.Extensions.Logging;
using CommunityToolkit.Maui;

namespace MyFirstMauiApp;

public static class MauiProgram
{
    public static MauiApp CreateMauiApp()
    {
        var builder = MauiApp.CreateBuilder();
        builder
            .UseMauiApp<App>()
            .UseMauiCommunityToolkit()
            .ConfigureFonts(fonts =>
            {
                fonts.AddFont("OpenSans-Regular.ttf", "OpenSansRegular");
                fonts.AddFont("OpenSans-Semibold.ttf", "OpenSansSemibold");
            });

        // Add logging
        builder.Services.AddLogging(configure =>
        {
            configure.AddDebug();
            configure.SetMinimumLevel(LogLevel.Debug);
        });

        // Register services
        builder.Services.AddSingleton<MainPage>();
        builder.Services.AddSingleton<MainPageViewModel>();

#if DEBUG
        builder.Logging.AddDebug();
#endif

        return builder.Build();
    }
}
```

### App.xaml Application Resources

```xml
<!-- App.xaml -->
<?xml version="1.0" encoding="UTF-8" ?>
<Application 
    x:Class="MyFirstMauiApp.App"
    xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
    xmlns:local="clr-namespace:MyFirstMauiApp">
    
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="Resources/Styles/Colors.xaml" />
                <ResourceDictionary Source="Resources/Styles/Styles.xaml" />
            </ResourceDictionary.MergedDictionaries>
            
            <!-- CSS-like Color System -->
            <Color x:Key="Primary">#512BD4</Color>
            <Color x:Key="Secondary">#DFD8F7</Color>
            <Color x:Key="Tertiary">#2B0B98</Color>
            
            <!-- Semantic Colors -->
            <Color x:Key="White">White</Color>
            <Color x:Key="Black">Black</Color>
            <Color x:Key="Gray100">#F8F9FA</Color>
            <Color x:Key="Gray200">#E9ECEF</Color>
            <Color x:Key="Gray300">#DEE2E6</Color>
            <Color x:Key="Gray400">#CED4DA</Color>
            <Color x:Key="Gray500">#6C757D</Color>
            <Color x:Key="Gray600">#495057</Color>
            <Color x:Key="Gray900">#212529</Color>
            
            <!-- Status Colors -->
            <Color x:Key="Success">#28A745</Color>
            <Color x:Key="Danger">#DC3545</Color>
            <Color x:Key="Warning">#FFC107</Color>
            <Color x:Key="Info">#17A2B8</Color>
            
            <!-- CSS-like Utility Classes -->
            <Style x:Key="H1" TargetType="Label">
                <Setter Property="FontSize" Value="32" />
                <Setter Property="FontAttributes" Value="Bold" />
                <Setter Property="TextColor" Value="{AppThemeBinding Light={StaticResource Gray900}, Dark={StaticResource White}}" />
            </Style>
            
            <Style x:Key="H2" TargetType="Label">
                <Setter Property="FontSize" Value="24" />
                <Setter Property="FontAttributes" Value="Bold" />
                <Setter Property="TextColor" Value="{AppThemeBinding Light={StaticResource Gray900}, Dark={StaticResource White}}" />
            </Style>
            
            <Style x:Key="Body" TargetType="Label">
                <Setter Property="FontSize" Value="16" />
                <Setter Property="TextColor" Value="{AppThemeBinding Light={StaticResource Gray600}, Dark={StaticResource Gray300}}" />
            </Style>
            
            <!-- Button Styles -->
            <Style x:Key="PrimaryButton" TargetType="Button">
                <Setter Property="BackgroundColor" Value="{StaticResource Primary}" />
                <Setter Property="TextColor" Value="{StaticResource White}" />
                <Setter Property="CornerRadius" Value="8" />
                <Setter Property="Padding" Value="16,12" />
                <Setter Property="FontAttributes" Value="Bold" />
            </Style>
            
            <Style x:Key="SecondaryButton" TargetType="Button">
                <Setter Property="BackgroundColor" Value="Transparent" />
                <Setter Property="TextColor" Value="{StaticResource Primary}" />
                <Setter Property="BorderColor" Value="{StaticResource Primary}" />
                <Setter Property="BorderWidth" Value="2" />
                <Setter Property="CornerRadius" Value="8" />
                <Setter Property="Padding" Value="16,12" />
            </Style>
            
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

---

## 🏠 MainPage Implementation

### MainPage.xaml - Modern UI Design

```xml
<!-- MainPage.xaml -->
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage 
    x:Class="MyFirstMauiApp.MainPage"
    xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
    Title="My First MAUI App">

    <ScrollView>
        <VerticalStackLayout 
            Spacing="25" 
            Padding="30,0" 
            VerticalOptions="Center">

            <!-- Hero Section -->
            <Grid>
                <RoundRectangle Fill="{StaticResource Primary}" CornerRadius="16" />
                <VerticalStackLayout Padding="24" Spacing="16">
                    <Image 
                        Source="dotnet_bot.png"
                        SemanticProperties.Description="Cute dot net bot waving hi to you!"
                        HeightRequest="150"
                        HorizontalOptions="Center" />
                    
                    <Label 
                        x:Name="WelcomeLabel"
                        Text="Welcome to .NET MAUI!"
                        Style="{StaticResource H1}"
                        TextColor="{StaticResource White}"
                        HorizontalOptions="Center" />
                    
                    <Label 
                        Text="Build native cross-platform apps with C# and XAML"
                        Style="{StaticResource Body}"
                        TextColor="{StaticResource Secondary}"
                        HorizontalOptions="Center"
                        HorizontalTextAlignment="Center" />
                </VerticalStackLayout>
            </Grid>

            <!-- Counter Section -->
            <Frame BackgroundColor="{AppThemeBinding Light={StaticResource Gray100}, Dark={StaticResource Gray600}}" 
                   CornerRadius="12" 
                   Padding="20" 
                   HasShadow="True">
                <VerticalStackLayout Spacing="16">
                    <Label 
                        Text="Click Counter Demo" 
                        Style="{StaticResource H2}"
                        HorizontalOptions="Center" />
                    
                    <Label 
                        x:Name="CounterLabel"
                        Text="{Binding ClickCount, StringFormat='Current count: {0}'}"
                        Style="{StaticResource Body}"
                        FontSize="18"
                        HorizontalOptions="Center" />
                    
                    <Button 
                        x:Name="CounterBtn"
                        Text="Click me!"
                        Style="{StaticResource PrimaryButton}"
                        Command="{Binding IncrementCommand}"
                        SemanticProperties.Hint="Counts the number of times you click"
                        HorizontalOptions="Center" />
                </VerticalStackLayout>
            </Frame>

            <!-- Navigation Section -->
            <Frame BackgroundColor="{AppThemeBinding Light={StaticResource Gray100}, Dark={StaticResource Gray600}}" 
                   CornerRadius="12" 
                   Padding="20" 
                   HasShadow="True">
                <VerticalStackLayout Spacing="16">
                    <Label 
                        Text="Navigation Demo" 
                        Style="{StaticResource H2}"
                        HorizontalOptions="Center" />
                    
                    <Grid ColumnDefinitions="*,*" ColumnSpacing="12">
                        <Button 
                            Grid.Column="0"
                            Text="About Page"
                            Style="{StaticResource SecondaryButton}"
                            Command="{Binding NavigateToAboutCommand}" />
                        
                        <Button 
                            Grid.Column="1"
                            Text="Settings"
                            Style="{StaticResource SecondaryButton}"
                            Command="{Binding NavigateToSettingsCommand}" />
                    </Grid>
                </VerticalStackLayout>
            </Frame>

            <!-- Features Section -->
            <Frame BackgroundColor="{AppThemeBinding Light={StaticResource Gray100}, Dark={StaticResource Gray600}}" 
                   CornerRadius="12" 
                   Padding="20" 
                   HasShadow="True">
                <VerticalStackLayout Spacing="16">
                    <Label 
                        Text="MAUI Features" 
                        Style="{StaticResource H2}"
                        HorizontalOptions="Center" />
                    
                    <CollectionView ItemsSource="{Binding Features}">
                        <CollectionView.ItemTemplate>
                            <DataTemplate>
                                <Grid Padding="8" ColumnDefinitions="Auto,*">
                                    <Ellipse Grid.Column="0" 
                                             Fill="{StaticResource Success}" 
                                             WidthRequest="8" 
                                             HeightRequest="8" 
                                             VerticalOptions="Center" />
                                    <Label Grid.Column="1" 
                                           Text="{Binding .}" 
                                           Style="{StaticResource Body}"
                                           Margin="12,0,0,0" />
                                </Grid>
                            </DataTemplate>
                        </CollectionView.ItemTemplate>
                    </CollectionView>
                </VerticalStackLayout>
            </Frame>

        </VerticalStackLayout>
    </ScrollView>

</ContentPage>
```

### MainPage.xaml.cs - Code Behind

```csharp
// MainPage.xaml.cs
namespace MyFirstMauiApp;

public partial class MainPage : ContentPage
{
    public MainPage(MainPageViewModel viewModel)
    {
        InitializeComponent();
        BindingContext = viewModel;
    }
}
```

### MainPageViewModel - MVVM Pattern

```csharp
// ViewModels/MainPageViewModel.cs
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;
using System.Collections.ObjectModel;

namespace MyFirstMauiApp.ViewModels;

public partial class MainPageViewModel : ObservableObject
{
    [ObservableProperty]
    private int clickCount = 0;

    [ObservableProperty]
    private ObservableCollection<string> features = new()
    {
        "Cross-platform (iOS, Android, Windows, macOS)",
        "Single codebase with native performance",
        "Hot Reload for fast development",
        "MVVM and Data Binding support",
        "Rich set of UI controls",
        "Platform-specific integrations"
    };

    [RelayCommand]
    private async Task Increment()
    {
        ClickCount++;
        
        // Add some visual feedback
        if (ClickCount % 5 == 0)
        {
            await Application.Current.MainPage.DisplayAlert(
                "Milestone!", 
                $"You've clicked {ClickCount} times! 🎉", 
                "OK");
        }
    }

    [RelayCommand]
    private async Task NavigateToAbout()
    {
        await Shell.Current.GoToAsync("///about");
    }

    [RelayCommand]
    private async Task NavigateToSettings()
    {
        await Shell.Current.GoToAsync("///settings");
    }
}
```

---

## 🧭 Navigation Fundamentals

### AppShell.xaml - Shell Navigation

```xml
<!-- AppShell.xaml -->
<?xml version="1.0" encoding="UTF-8" ?>
<Shell
    x:Class="MyFirstMauiApp.AppShell"
    xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
    xmlns:local="clr-namespace:MyFirstMauiApp"
    Title="MyFirstMauiApp">

    <!-- Shell Resources -->
    <Shell.Resources>
        <ResourceDictionary>
            <Style x:Key="BaseStyle" TargetType="Element">
                <Setter Property="Shell.BackgroundColor" Value="{AppThemeBinding Light={StaticResource White}, Dark={StaticResource Black}}" />
                <Setter Property="Shell.ForegroundColor" Value="{AppThemeBinding Light={StaticResource Primary}, Dark={StaticResource White}}" />
                <Setter Property="Shell.TitleColor" Value="{AppThemeBinding Light={StaticResource Primary}, Dark={StaticResource White}}" />
                <Setter Property="Shell.DisabledColor" Value="{AppThemeBinding Light={StaticResource Gray200}, Dark={StaticResource Gray600}}" />
                <Setter Property="Shell.UnselectedColor" Value="{AppThemeBinding Light={StaticResource Gray200}, Dark={StaticResource Gray600}}" />
                <Setter Property="Shell.TabBarBackgroundColor" Value="{AppThemeBinding Light={StaticResource White}, Dark={StaticResource Black}}" />
                <Setter Property="Shell.TabBarForegroundColor" Value="{AppThemeBinding Light={StaticResource Primary}, Dark={StaticResource White}}" />
                <Setter Property="Shell.TabBarUnselectedColor" Value="{AppThemeBinding Light={StaticResource Gray200}, Dark={StaticResource Gray600}}" />
                <Setter Property="Shell.TabBarTitleColor" Value="{AppThemeBinding Light={StaticResource Primary}, Dark={StaticResource White}}" />
            </Style>
            <Style TargetType="TabBar" BasedOn="{StaticResource BaseStyle}" />
            <Style TargetType="FlyoutItem" BasedOn="{StaticResource BaseStyle}" />
        </ResourceDictionary>
    </Shell.Resources>

    <!-- Main Tabs -->
    <TabBar>
        <!-- Home Tab -->
        <ShellContent
            Title="Home"
            ContentTemplate="{DataTemplate local:MainPage}"
            Route="main"
            Icon="home.png" />
        
        <!-- About Tab -->
        <ShellContent
            Title="About"
            ContentTemplate="{DataTemplate local:AboutPage}"
            Route="about"
            Icon="info.png" />
        
        <!-- Settings Tab -->
        <ShellContent
            Title="Settings"
            ContentTemplate="{DataTemplate local:SettingsPage}"
            Route="settings"
            Icon="settings.png" />
    </TabBar>

</Shell>
```

### Additional Pages

```xml
<!-- Views/AboutPage.xaml -->
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage 
    x:Class="MyFirstMauiApp.AboutPage"
    xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
    Title="About">
    
    <VerticalStackLayout Spacing="25" Padding="30,0" VerticalOptions="Center">
        
        <Image Source="dotnet_bot.png" HeightRequest="185" />
        
        <Label 
            Text="About This App"
            Style="{StaticResource H1}"
            HorizontalOptions="Center" />
            
        <Label 
            Text="This is my first .NET MAUI application! Built with love using C# and XAML."
            Style="{StaticResource Body}"
            HorizontalOptions="Center"
            HorizontalTextAlignment="Center" />
            
        <Button 
            Text="Learn More"
            Style="{StaticResource PrimaryButton}"
            Clicked="OnLearnMoreClicked" />
            
    </VerticalStackLayout>
    
</ContentPage>
```

```csharp
// Views/AboutPage.xaml.cs
namespace MyFirstMauiApp;

public partial class AboutPage : ContentPage
{
    public AboutPage()
    {
        InitializeComponent();
    }

    private async void OnLearnMoreClicked(object sender, EventArgs e)
    {
        await Launcher.OpenAsync("https://aka.ms/maui");
    }
}
```

---

## 📊 Data Management

### Models and Services

```csharp
// Models/AppInfo.cs
namespace MyFirstMauiApp.Models;

public class AppInfo
{
    public string Name { get; set; } = "My First MAUI App";
    public string Version { get; set; } = "1.0.0";
    public string Developer { get; set; } = "Your Name";
    public DateTime CreatedDate { get; set; } = DateTime.Now;
    public List<string> Platforms { get; set; } = new()
    {
        "iOS", "Android", "Windows", "macOS"
    };
}

// Models/UserPreferences.cs
public class UserPreferences
{
    public string Theme { get; set; } = "System";
    public string Language { get; set; } = "en";
    public bool NotificationsEnabled { get; set; } = true;
    public double FontSize { get; set; } = 16.0;
}
```

```csharp
// Services/PreferencesService.cs
namespace MyFirstMauiApp.Services;

public interface IPreferencesService
{
    T Get<T>(string key, T defaultValue);
    void Set<T>(string key, T value);
    void Remove(string key);
    void Clear();
}

public class PreferencesService : IPreferencesService
{
    public T Get<T>(string key, T defaultValue)
    {
        return Preferences.Get(key, defaultValue);
    }

    public void Set<T>(string key, T value)
    {
        Preferences.Set(key, value);
    }

    public void Remove(string key)
    {
        Preferences.Remove(key);
    }

    public void Clear()
    {
        Preferences.Clear();
    }
}
```

---

## 🎨 UI Styling

### CSS-like Styling System

```xml
<!-- Resources/Styles/Styles.xaml -->
<?xml version="1.0" encoding="UTF-8" ?>
<ResourceDictionary 
    xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">

    <!-- CSS-like Spacing Utilities -->
    <Style x:Key="m-0" TargetType="View">
        <Setter Property="Margin" Value="0" />
    </Style>
    
    <Style x:Key="m-1" TargetType="View">
        <Setter Property="Margin" Value="4" />
    </Style>
    
    <Style x:Key="m-2" TargetType="View">
        <Setter Property="Margin" Value="8" />
    </Style>
    
    <Style x:Key="m-3" TargetType="View">
        <Setter Property="Margin" Value="12" />
    </Style>
    
    <Style x:Key="m-4" TargetType="View">
        <Setter Property="Margin" Value="16" />
    </Style>
    
    <Style x:Key="p-0" TargetType="Layout">
        <Setter Property="Padding" Value="0" />
    </Style>
    
    <Style x:Key="p-1" TargetType="Layout">
        <Setter Property="Padding" Value="4" />
    </Style>
    
    <Style x:Key="p-2" TargetType="Layout">
        <Setter Property="Padding" Value="8" />
    </Style>
    
    <Style x:Key="p-3" TargetType="Layout">
        <Setter Property="Padding" Value="12" />
    </Style>
    
    <Style x:Key="p-4" TargetType="Layout">
        <Setter Property="Padding" Value="16" />
    </Style>

    <!-- CSS-like Text Utilities -->
    <Style x:Key="text-xs" TargetType="Label">
        <Setter Property="FontSize" Value="12" />
    </Style>
    
    <Style x:Key="text-sm" TargetType="Label">
        <Setter Property="FontSize" Value="14" />
    </Style>
    
    <Style x:Key="text-base" TargetType="Label">
        <Setter Property="FontSize" Value="16" />
    </Style>
    
    <Style x:Key="text-lg" TargetType="Label">
        <Setter Property="FontSize" Value="18" />
    </Style>
    
    <Style x:Key="text-xl" TargetType="Label">
        <Setter Property="FontSize" Value="20" />
    </Style>
    
    <Style x:Key="text-2xl" TargetType="Label">
        <Setter Property="FontSize" Value="24" />
    </Style>
    
    <Style x:Key="text-center" TargetType="Label">
        <Setter Property="HorizontalTextAlignment" Value="Center" />
        <Setter Property="HorizontalOptions" Value="Center" />
    </Style>
    
    <Style x:Key="text-left" TargetType="Label">
        <Setter Property="HorizontalTextAlignment" Value="Start" />
    </Style>
    
    <Style x:Key="text-right" TargetType="Label">
        <Setter Property="HorizontalTextAlignment" Value="End" />
    </Style>

    <!-- CSS-like Background Utilities -->
    <Style x:Key="bg-primary" TargetType="VisualElement">
        <Setter Property="BackgroundColor" Value="{StaticResource Primary}" />
    </Style>
    
    <Style x:Key="bg-secondary" TargetType="VisualElement">
        <Setter Property="BackgroundColor" Value="{StaticResource Secondary}" />
    </Style>
    
    <Style x:Key="bg-white" TargetType="VisualElement">
        <Setter Property="BackgroundColor" Value="{StaticResource White}" />
    </Style>
    
    <Style x:Key="bg-gray-100" TargetType="VisualElement">
        <Setter Property="BackgroundColor" Value="{StaticResource Gray100}" />
    </Style>

    <!-- CSS-like Border Utilities -->
    <Style x:Key="rounded-sm" TargetType="Frame">
        <Setter Property="CornerRadius" Value="4" />
    </Style>
    
    <Style x:Key="rounded" TargetType="Frame">
        <Setter Property="CornerRadius" Value="8" />
    </Style>
    
    <Style x:Key="rounded-lg" TargetType="Frame">
        <Setter Property="CornerRadius" Value="12" />
    </Style>
    
    <Style x:Key="rounded-xl" TargetType="Frame">
        <Setter Property="CornerRadius" Value="16" />
    </Style>

    <!-- CSS-like Shadow Utilities -->
    <Style x:Key="shadow-sm" TargetType="Frame">
        <Setter Property="HasShadow" Value="True" />
    </Style>
    
    <Style x:Key="shadow" TargetType="Frame">
        <Setter Property="HasShadow" Value="True" />
    </Style>

</ResourceDictionary>
```

### Usage Examples

```xml
<!-- CSS-like approach in XAML -->
<VerticalStackLayout>
    <!-- Using utility classes -->
    <Label 
        Text="Large centered title"
        Style="{StaticResource text-2xl}"
        HorizontalOptions="Center" />
    
    <Frame 
        Style="{StaticResource bg-white}"
        CornerRadius="12"
        HasShadow="True"
        Padding="16">
        
        <Label 
            Text="Card content with background and shadow"
            Style="{StaticResource text-base}" />
    </Frame>
    
    <!-- Multiple style composition -->
    <Button>
        <Button.Style>
            <Style TargetType="Button" BasedOn="{StaticResource PrimaryButton}">
                <Setter Property="Margin" Value="16" />
                <Setter Property="CornerRadius" Value="12" />
            </Style>
        </Button.Style>
    </Button>
</VerticalStackLayout>
```

---

## 📱 Platform-Specific Features

### Platform-Specific Code

```csharp
// Platforms/Android/MainActivity.cs
using Android.App;
using Android.Content.PM;
using Android.OS;

namespace MyFirstMauiApp;

[Activity(Theme = "@style/Maui.SplashTheme", MainLauncher = true, ConfigurationChanges = ConfigChanges.ScreenSize | ConfigChanges.Orientation | ConfigChanges.UiMode | ConfigChanges.ScreenLayout | ConfigChanges.SmallestScreenSize | ConfigChanges.Density)]
public class MainActivity : MauiAppCompatActivity
{
    protected override void OnCreate(Bundle savedInstanceState)
    {
        base.OnCreate(savedInstanceState);
        
        // Android-specific initialization
        Platform.Init(this, savedInstanceState);
    }
}
```

```csharp
// Platform-specific implementations
#if ANDROID
using AndroidX.Core.Content;
using Android;
#elif IOS
using UIKit;
#endif

namespace MyFirstMauiApp.Services;

public class DeviceService
{
    public string GetDeviceInfo()
    {
#if ANDROID
        return $"Android {Android.OS.Build.VERSION.Release}";
#elif IOS
        return $"iOS {UIDevice.CurrentDevice.SystemVersion}";
#elif WINDOWS
        return "Windows";
#elif MACCATALYST
        return "macOS";
#else
        return "Unknown Platform";
#endif
    }

    public async Task<bool> CheckPermissionAsync()
    {
#if ANDROID
        var status = await Permissions.CheckStatusAsync<Permissions.Camera>();
        return status == PermissionStatus.Granted;
#elif IOS
        return true; // iOS handles permissions differently
#else
        return true;
#endif
    }
}
```

---

## 🔧 Testing & Debugging

### Unit Testing Setup

```csharp
// Tests/ViewModelTests.cs
using Xunit;
using MyFirstMauiApp.ViewModels;

namespace MyFirstMauiApp.Tests;

public class MainPageViewModelTests
{
    [Fact]
    public void IncrementCommand_ShouldIncreaseClickCount()
    {
        // Arrange
        var viewModel = new MainPageViewModel();
        var initialCount = viewModel.ClickCount;

        // Act
        viewModel.IncrementCommand.Execute(null);

        // Assert
        Assert.Equal(initialCount + 1, viewModel.ClickCount);
    }

    [Theory]
    [InlineData(0)]
    [InlineData(5)]
    [InlineData(10)]
    public void ClickCount_ShouldUpdateProperly(int expectedCount)
    {
        // Arrange
        var viewModel = new MainPageViewModel();

        // Act
        for (int i = 0; i < expectedCount; i++)
        {
            viewModel.IncrementCommand.Execute(null);
        }

        // Assert
        Assert.Equal(expectedCount, viewModel.ClickCount);
    }
}
```

### Debugging Tips

```csharp
// Debug helpers in MainPage.xaml.cs
namespace MyFirstMauiApp;

public partial class MainPage : ContentPage
{
    public MainPage(MainPageViewModel viewModel)
    {
        InitializeComponent();
        BindingContext = viewModel;

#if DEBUG
        // Debug information
        System.Diagnostics.Debug.WriteLine($"MainPage initialized at {DateTime.Now}");
        System.Diagnostics.Debug.WriteLine($"Device: {DeviceInfo.Platform} - {DeviceInfo.Model}");
        System.Diagnostics.Debug.WriteLine($"Screen: {DeviceDisplay.MainDisplayInfo.Width}x{DeviceDisplay.MainDisplayInfo.Height}");
#endif
    }

    protected override void OnAppearing()
    {
        base.OnAppearing();
        
#if DEBUG
        System.Diagnostics.Debug.WriteLine("MainPage appeared");
#endif
    }

    protected override void OnDisappearing()
    {
        base.OnDisappearing();
        
#if DEBUG
        System.Diagnostics.Debug.WriteLine("MainPage disappeared");
#endif
    }
}
```

---

## 🎯 Exercises

### Exercise 1: Customize the Welcome Message
```xml
<!-- Add a personalized greeting with user input -->
<Entry x:Name="NameEntry" Placeholder="Enter your name" />
<Button Text="Greet Me" Clicked="OnGreetClicked" />
<Label x:Name="GreetingLabel" />
```

```csharp
private void OnGreetClicked(object sender, EventArgs e)
{
    var name = NameEntry.Text ?? "Friend";
    GreetingLabel.Text = $"Hello, {name}! Welcome to MAUI! 👋";
}
```

### Exercise 2: Add Theme Switching
```csharp
// Add theme toggle functionality
private void OnThemeToggleClicked(object sender, EventArgs e)
{
    Application.Current.UserAppTheme = Application.Current.UserAppTheme == AppTheme.Dark 
        ? AppTheme.Light 
        : AppTheme.Dark;
}
```

### Exercise 3: Create Custom Counter
```csharp
// Create a counter with custom increment values
[ObservableProperty]
private int incrementValue = 1;

[RelayCommand]
private void SetIncrementValue(string value)
{
    if (int.TryParse(value, out int result))
    {
        IncrementValue = result;
    }
}
```

---

## 📝 Quiz

1. **MAUI Project Structure**: แฟ้มใดที่ใช้สำหรับกำหนดค่าแอปพลิเคชัน MAUI?
   - a) App.xaml
   - b) MauiProgram.cs
   - c) AppShell.xaml
   - d) ทั้งหมดข้างต้น

2. **CSS-like Styling**: Style ใดที่ใช้สำหรับจัดกึ่งกลางข้อความ?
   - a) `text-center`
   - b) `center-text`
   - c) `align-center`
   - d) `text-middle`

3. **MVVM Pattern**: คำสั่งใดที่ใช้สำหรับสร้าง RelayCommand?
   - a) `[Command]`
   - b) `[RelayCommand]`
   - c) `[AsyncCommand]`
   - d) `[BindableCommand]`

<details>
<summary>เฉลย</summary>

1. **d) ทั้งหมดข้างต้น** - แฟ้มทั้งหมดมีบทบาทในการกำหนดค่าแอป
2. **a) text-center** - ใช้สำหรับจัดข้อความกึ่งกลาง
3. **b) [RelayCommand]** - Attribute สำหรับสร้าง RelayCommand

</details>

---

## 🎉 สรุป

ในบทเรียนนี้ คุณได้เรียนรู้:

✅ **Project Setup** - การสร้างและจัดการ MAUI project  
✅ **Project Structure** - โครงสร้างไฟล์และหน้าที่ของแต่ละส่วน  
✅ **UI Implementation** - การสร้าง UI ด้วย XAML และ CSS-like approaches  
✅ **MVVM Pattern** - การใช้ ViewModel และ Data Binding  
✅ **Navigation** - การนำทางด้วย Shell  
✅ **Platform Features** - การใช้ฟีเจอร์เฉพาะแพลตฟอร์ม  
✅ **Testing & Debugging** - เทคนิคการทดสอบและแก้ไขข้อผิดพลาด

### Next Steps
🔜 **บทที่ 5**: Layouts & Controls - เรียนรู้การจัดวางและควบคุม UI  
🔜 **บทที่ 6**: Data Binding & MVVM - ลงลึกใน MVVM pattern  
🔜 **บทที่ 7**: Styling & Theming - การจัดรูปแบบและธีมขั้นสูง

---

**Happy Coding! 🚀**

> บทเรียนต่อไป: [05-layouts-and-controls.md](05-layouts-and-controls.md)
