# 🎨 บทที่ 7: Styling & Theming
> **เชี่ยวชาญการจัดรูปแบบและธีมแบบมืออาชีพ**

---

## 📋 สารบัญ
1. [Styling Fundamentals](#styling-fundamentals)
2. [Resource Dictionaries](#resource-dictionaries)
3. [CSS-like Utility Classes](#css-like-utility-classes)
4. [Dynamic Theming](#dynamic-theming)
5. [Design Systems](#design-systems)
6. [Custom Controls Styling](#custom-controls-styling)
7. [Advanced Styling Techniques](#advanced-styling-techniques)
8. [Performance & Best Practices](#performance-best-practices)

---

## 🎯 Learning Objectives

เมื่อจบบทเรียนนี้ คุณจะสามารถ:
- ✅ สร้างและจัดการ Style และ Resource Dictionaries
- ✅ ใช้ CSS-like approaches ใน XAML styling
- ✅ สร้าง Dynamic Theming system
- ✅ ออกแบบ Design System ที่ scalable
- ✅ Style Custom Controls อย่างมืออาชีพ
- ✅ ใช้ Advanced styling techniques
- ✅ Optimize performance ของ styling system

---

## 🎨 Styling Fundamentals

### Basic Styles

```xml
<!-- App.xaml - Global Styles -->
<Application.Resources>
    <ResourceDictionary>
        
        <!-- Implicit Styles (auto-applied) -->
        <Style TargetType="Label">
            <Setter Property="FontFamily" Value="OpenSansRegular" />
            <Setter Property="FontSize" Value="16" />
            <Setter Property="TextColor" Value="{AppThemeBinding Light={StaticResource Gray900}, Dark={StaticResource White}}" />
        </Style>
        
        <!-- Explicit Styles (must be referenced) -->
        <Style x:Key="HeadingLabel" TargetType="Label">
            <Setter Property="FontFamily" Value="OpenSansSemibold" />
            <Setter Property="FontSize" Value="24" />
            <Setter Property="FontAttributes" Value="Bold" />
            <Setter Property="TextColor" Value="{StaticResource Primary}" />
        </Style>
        
        <!-- Style Inheritance -->
        <Style x:Key="SubHeadingLabel" TargetType="Label" BasedOn="{StaticResource HeadingLabel}">
            <Setter Property="FontSize" Value="20" />
            <Setter Property="FontAttributes" Value="None" />
        </Style>
        
        <!-- Dynamic Resource Reference -->
        <Style x:Key="PrimaryButton" TargetType="Button">
            <Setter Property="BackgroundColor" Value="{DynamicResource PrimaryColor}" />
            <Setter Property="TextColor" Value="{DynamicResource OnPrimaryColor}" />
            <Setter Property="CornerRadius" Value="8" />
            <Setter Property="Padding" Value="16,12" />
            <Setter Property="FontAttributes" Value="Bold" />
        </Style>
        
    </ResourceDictionary>
</Application.Resources>
```

### Style Triggers

```xml
<!-- Visual State Manager for interactive styles -->
<Style x:Key="InteractiveButton" TargetType="Button">
    <Setter Property="BackgroundColor" Value="{StaticResource Primary}" />
    <Setter Property="TextColor" Value="White" />
    <Setter Property="CornerRadius" Value="8" />
    <Setter Property="Scale" Value="1" />
    
    <Setter Property="VisualStateManager.VisualStateGroups">
        <VisualStateGroupList>
            <VisualStateGroup x:Name="CommonStates">
                
                <VisualState x:Name="Normal">
                    <VisualState.Setters>
                        <Setter Property="Scale" Value="1" />
                        <Setter Property="Opacity" Value="1" />
                    </VisualState.Setters>
                </VisualState>
                
                <VisualState x:Name="Pressed">
                    <VisualState.Setters>
                        <Setter Property="Scale" Value="0.96" />
                        <Setter Property="Opacity" Value="0.8" />
                    </VisualState.Setters>
                </VisualState>
                
                <VisualState x:Name="Disabled">
                    <VisualState.Setters>
                        <Setter Property="BackgroundColor" Value="{StaticResource Gray300}" />
                        <Setter Property="TextColor" Value="{StaticResource Gray500}" />
                        <Setter Property="Opacity" Value="0.6" />
                    </VisualState.Setters>
                </VisualState>
                
            </VisualStateGroup>
        </VisualStateGroupList>
    </Setter>
</Style>

<!-- Data Triggers -->
<Style x:Key="ConditionalLabel" TargetType="Label">
    <Setter Property="TextColor" Value="{StaticResource Gray900}" />
    <Setter Property="FontAttributes" Value="None" />
    
    <Style.Triggers>
        <DataTrigger TargetType="Label" Binding="{Binding IsImportant}" Value="True">
            <Setter Property="TextColor" Value="{StaticResource Danger}" />
            <Setter Property="FontAttributes" Value="Bold" />
        </DataTrigger>
        
        <MultiTrigger TargetType="Label">
            <MultiTrigger.Conditions>
                <BindingCondition Binding="{Binding IsUrgent}" Value="True" />
                <BindingCondition Binding="{Binding IsVisible}" Value="True" />
            </MultiTrigger.Conditions>
            <MultiTrigger.Setters>
                <Setter Property="TextColor" Value="{StaticResource Warning}" />
                <Setter Property="BackgroundColor" Value="{StaticResource WarningLight}" />
            </MultiTrigger.Setters>
        </MultiTrigger>
    </Style.Triggers>
</Style>
```

---

## 📚 Resource Dictionaries

### Organized Resource Structure

```
Resources/
├── Styles/
│   ├── Colors.xaml           # Color definitions
│   ├── Typography.xaml       # Font and text styles
│   ├── Controls.xaml         # Control styles
│   ├── Layouts.xaml         # Layout utilities
│   └── Animations.xaml      # Animation resources
├── Themes/
│   ├── LightTheme.xaml      # Light theme
│   ├── DarkTheme.xaml       # Dark theme
│   └── HighContrastTheme.xaml
└── Components/
    ├── Cards.xaml           # Card components
    ├── Buttons.xaml         # Button variants
    └── Forms.xaml           # Form controls
```

### Colors.xaml

```xml
<!-- Resources/Styles/Colors.xaml -->
<?xml version="1.0" encoding="UTF-8" ?>
<ResourceDictionary 
    xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">

    <!-- Design System Colors -->
    
    <!-- Primary Colors -->
    <Color x:Key="Primary">#6366F1</Color>
    <Color x:Key="PrimaryDark">#4F46E5</Color>
    <Color x:Key="PrimaryLight">#A5B4FC</Color>
    <Color x:Key="OnPrimary">#FFFFFF</Color>
    
    <!-- Secondary Colors -->
    <Color x:Key="Secondary">#EC4899</Color>
    <Color x:Key="SecondaryDark">#DB2777</Color>
    <Color x:Key="SecondaryLight">#F9A8D4</Color>
    <Color x:Key="OnSecondary">#FFFFFF</Color>
    
    <!-- Neutral Colors -->
    <Color x:Key="Gray50">#F9FAFB</Color>
    <Color x:Key="Gray100">#F3F4F6</Color>
    <Color x:Key="Gray200">#E5E7EB</Color>
    <Color x:Key="Gray300">#D1D5DB</Color>
    <Color x:Key="Gray400">#9CA3AF</Color>
    <Color x:Key="Gray500">#6B7280</Color>
    <Color x:Key="Gray600">#4B5563</Color>
    <Color x:Key="Gray700">#374151</Color>
    <Color x:Key="Gray800">#1F2937</Color>
    <Color x:Key="Gray900">#111827</Color>
    
    <!-- Status Colors -->
    <Color x:Key="Success">#10B981</Color>
    <Color x:Key="SuccessLight">#D1FAE5</Color>
    <Color x:Key="Warning">#F59E0B</Color>
    <Color x:Key="WarningLight">#FEF3C7</Color>
    <Color x:Key="Danger">#EF4444</Color>
    <Color x:Key="DangerLight">#FEE2E2</Color>
    <Color x:Key="Info">#3B82F6</Color>
    <Color x:Key="InfoLight">#DBEAFE</Color>
    
    <!-- Surface Colors -->
    <Color x:Key="Surface">#FFFFFF</Color>
    <Color x:Key="SurfaceVariant">#F8FAFC</Color>
    <Color x:Key="OnSurface">#0F172A</Color>
    <Color x:Key="OnSurfaceVariant">#64748B</Color>
    
    <!-- Background Colors -->
    <Color x:Key="Background">#FFFFFF</Color>
    <Color x:Key="OnBackground">#0F172A</Color>
    
    <!-- Theme-aware colors -->
    <Color x:Key="TextPrimary">{AppThemeBinding Light={StaticResource Gray900}, Dark={StaticResource Gray100}}</Color>
    <Color x:Key="TextSecondary">{AppThemeBinding Light={StaticResource Gray600}, Dark={StaticResource Gray400}}</Color>
    <Color x:Key="TextDisabled">{AppThemeBinding Light={StaticResource Gray400}, Dark={StaticResource Gray600}}</Color>
    
    <Color x:Key="SurfaceColor">{AppThemeBinding Light={StaticResource Surface}, Dark={StaticResource Gray800}}</Color>
    <Color x:Key="BackgroundColor">{AppThemeBinding Light={StaticResource Background}, Dark={StaticResource Gray900}}</Color>
    
    <!-- Gradients -->
    <LinearGradientBrush x:Key="PrimaryGradient" StartPoint="0,0" EndPoint="1,1">
        <GradientStop Color="{StaticResource Primary}" Offset="0.0" />
        <GradientStop Color="{StaticResource PrimaryDark}" Offset="1.0" />
    </LinearGradientBrush>
    
    <LinearGradientBrush x:Key="SunsetGradient" StartPoint="0,0" EndPoint="1,1">
        <GradientStop Color="#FF8A80" Offset="0.0" />
        <GradientStop Color="#FF5722" Offset="0.5" />
        <GradientStop Color="#E91E63" Offset="1.0" />
    </LinearGradientBrush>

</ResourceDictionary>
```

### Typography.xaml

```xml
<!-- Resources/Styles/Typography.xaml -->
<?xml version="1.0" encoding="UTF-8" ?>
<ResourceDictionary 
    xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">

    <!-- Typography Scale (Tailwind-inspired) -->
    
    <!-- Display Styles -->
    <Style x:Key="Display1" TargetType="Label">
        <Setter Property="FontSize" Value="72" />
        <Setter Property="FontAttributes" Value="Bold" />
        <Setter Property="LineHeight" Value="1.1" />
        <Setter Property="TextColor" Value="{DynamicResource TextPrimary}" />
    </Style>
    
    <Style x:Key="Display2" TargetType="Label">
        <Setter Property="FontSize" Value="60" />
        <Setter Property="FontAttributes" Value="Bold" />
        <Setter Property="LineHeight" Value="1.1" />
        <Setter Property="TextColor" Value="{DynamicResource TextPrimary}" />
    </Style>
    
    <!-- Heading Styles -->
    <Style x:Key="H1" TargetType="Label">
        <Setter Property="FontSize" Value="48" />
        <Setter Property="FontAttributes" Value="Bold" />
        <Setter Property="LineHeight" Value="1.2" />
        <Setter Property="TextColor" Value="{DynamicResource TextPrimary}" />
    </Style>
    
    <Style x:Key="H2" TargetType="Label">
        <Setter Property="FontSize" Value="36" />
        <Setter Property="FontAttributes" Value="Bold" />
        <Setter Property="LineHeight" Value="1.25" />
        <Setter Property="TextColor" Value="{DynamicResource TextPrimary}" />
    </Style>
    
    <Style x:Key="H3" TargetType="Label">
        <Setter Property="FontSize" Value="30" />
        <Setter Property="FontAttributes" Value="Bold" />
        <Setter Property="LineHeight" Value="1.3" />
        <Setter Property="TextColor" Value="{DynamicResource TextPrimary}" />
    </Style>
    
    <Style x:Key="H4" TargetType="Label">
        <Setter Property="FontSize" Value="24" />
        <Setter Property="FontAttributes" Value="Bold" />
        <Setter Property="LineHeight" Value="1.35" />
        <Setter Property="TextColor" Value="{DynamicResource TextPrimary}" />
    </Style>
    
    <Style x:Key="H5" TargetType="Label">
        <Setter Property="FontSize" Value="20" />
        <Setter Property="FontAttributes" Value="Bold" />
        <Setter Property="LineHeight" Value="1.4" />
        <Setter Property="TextColor" Value="{DynamicResource TextPrimary}" />
    </Style>
    
    <Style x:Key="H6" TargetType="Label">
        <Setter Property="FontSize" Value="18" />
        <Setter Property="FontAttributes" Value="Bold" />
        <Setter Property="LineHeight" Value="1.45" />
        <Setter Property="TextColor" Value="{DynamicResource TextPrimary}" />
    </Style>
    
    <!-- Body Text Styles -->
    <Style x:Key="BodyLarge" TargetType="Label">
        <Setter Property="FontSize" Value="18" />
        <Setter Property="LineHeight" Value="1.6" />
        <Setter Property="TextColor" Value="{DynamicResource TextPrimary}" />
    </Style>
    
    <Style x:Key="Body" TargetType="Label">
        <Setter Property="FontSize" Value="16" />
        <Setter Property="LineHeight" Value="1.6" />
        <Setter Property="TextColor" Value="{DynamicResource TextPrimary}" />
    </Style>
    
    <Style x:Key="BodySmall" TargetType="Label">
        <Setter Property="FontSize" Value="14" />
        <Setter Property="LineHeight" Value="1.5" />
        <Setter Property="TextColor" Value="{DynamicResource TextSecondary}" />
    </Style>
    
    <!-- Caption and Label Styles -->
    <Style x:Key="Caption" TargetType="Label">
        <Setter Property="FontSize" Value="12" />
        <Setter Property="LineHeight" Value="1.4" />
        <Setter Property="TextColor" Value="{DynamicResource TextSecondary}" />
    </Style>
    
    <Style x:Key="Label" TargetType="Label">
        <Setter Property="FontSize" Value="14" />
        <Setter Property="FontAttributes" Value="Bold" />
        <Setter Property="TextColor" Value="{DynamicResource TextPrimary}" />
    </Style>
    
    <!-- Utility Text Styles -->
    <Style x:Key="Overline" TargetType="Label">
        <Setter Property="FontSize" Value="10" />
        <Setter Property="FontAttributes" Value="Bold" />
        <Setter Property="TextTransform" Value="Uppercase" />
        <Setter Property="LetterSpacing" Value="1.5" />
        <Setter Property="TextColor" Value="{DynamicResource TextSecondary}" />
    </Style>
    
    <!-- Link Styles -->
    <Style x:Key="Link" TargetType="Label">
        <Setter Property="FontSize" Value="16" />
        <Setter Property="TextColor" Value="{StaticResource Primary}" />
        <Setter Property="TextDecorations" Value="Underline" />
    </Style>
    
    <Style x:Key="LinkSmall" TargetType="Label">
        <Setter Property="FontSize" Value="14" />
        <Setter Property="TextColor" Value="{StaticResource Primary}" />
        <Setter Property="TextDecorations" Value="Underline" />
    </Style>

</ResourceDictionary>
```

---

## 🛠️ CSS-like Utility Classes

### Spacing Utilities

```xml
<!-- Resources/Styles/Layouts.xaml -->
<?xml version="1.0" encoding="UTF-8" ?>
<ResourceDictionary 
    xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">

    <!-- Spacing Scale: 0, 1, 2, 3, 4, 5, 6, 8, 10, 12, 16, 20, 24, 32, 40, 48, 64 -->
    <!-- Where: 1 = 4dp, 2 = 8dp, 3 = 12dp, 4 = 16dp, etc. -->
    
    <!-- Margin Utilities -->
    <Style x:Key="m-0" TargetType="View"><Setter Property="Margin" Value="0" /></Style>
    <Style x:Key="m-1" TargetType="View"><Setter Property="Margin" Value="4" /></Style>
    <Style x:Key="m-2" TargetType="View"><Setter Property="Margin" Value="8" /></Style>
    <Style x:Key="m-3" TargetType="View"><Setter Property="Margin" Value="12" /></Style>
    <Style x:Key="m-4" TargetType="View"><Setter Property="Margin" Value="16" /></Style>
    <Style x:Key="m-5" TargetType="View"><Setter Property="Margin" Value="20" /></Style>
    <Style x:Key="m-6" TargetType="View"><Setter Property="Margin" Value="24" /></Style>
    <Style x:Key="m-8" TargetType="View"><Setter Property="Margin" Value="32" /></Style>
    
    <!-- Directional Margins -->
    <Style x:Key="mt-4" TargetType="View"><Setter Property="Margin" Value="0,16,0,0" /></Style>
    <Style x:Key="mr-4" TargetType="View"><Setter Property="Margin" Value="0,0,16,0" /></Style>
    <Style x:Key="mb-4" TargetType="View"><Setter Property="Margin" Value="0,0,0,16" /></Style>
    <Style x:Key="ml-4" TargetType="View"><Setter Property="Margin" Value="16,0,0,0" /></Style>
    
    <Style x:Key="mx-4" TargetType="View"><Setter Property="Margin" Value="16,0" /></Style>
    <Style x:Key="my-4" TargetType="View"><Setter Property="Margin" Value="0,16" /></Style>
    
    <!-- Padding Utilities -->
    <Style x:Key="p-0" TargetType="Layout"><Setter Property="Padding" Value="0" /></Style>
    <Style x:Key="p-1" TargetType="Layout"><Setter Property="Padding" Value="4" /></Style>
    <Style x:Key="p-2" TargetType="Layout"><Setter Property="Padding" Value="8" /></Style>
    <Style x:Key="p-3" TargetType="Layout"><Setter Property="Padding" Value="12" /></Style>
    <Style x:Key="p-4" TargetType="Layout"><Setter Property="Padding" Value="16" /></Style>
    <Style x:Key="p-5" TargetType="Layout"><Setter Property="Padding" Value="20" /></Style>
    <Style x:Key="p-6" TargetType="Layout"><Setter Property="Padding" Value="24" /></Style>
    <Style x:Key="p-8" TargetType="Layout"><Setter Property="Padding" Value="32" /></Style>
    
    <!-- Width and Height Utilities -->
    <Style x:Key="w-full" TargetType="View"><Setter Property="HorizontalOptions" Value="FillAndExpand" /></Style>
    <Style x:Key="h-full" TargetType="View"><Setter Property="VerticalOptions" Value="FillAndExpand" /></Style>
    
    <!-- Flex Utilities -->
    <Style x:Key="flex" TargetType="View"><Setter Property="HorizontalOptions" Value="FillAndExpand" /></Style>
    <Style x:Key="flex-1" TargetType="View">
        <Setter Property="HorizontalOptions" Value="FillAndExpand" />
        <Setter Property="VerticalOptions" Value="FillAndExpand" />
    </Style>
    
    <!-- Alignment Utilities -->
    <Style x:Key="items-center" TargetType="Layout">
        <Setter Property="HorizontalOptions" Value="Center" />
    </Style>
    
    <Style x:Key="justify-center" TargetType="Layout">
        <Setter Property="VerticalOptions" Value="Center" />
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

</ResourceDictionary>
```

### Background and Border Utilities

```xml
<!-- Background Color Utilities -->
<Style x:Key="bg-primary" TargetType="VisualElement">
    <Setter Property="BackgroundColor" Value="{StaticResource Primary}" />
</Style>

<Style x:Key="bg-secondary" TargetType="VisualElement">
    <Setter Property="BackgroundColor" Value="{StaticResource Secondary}" />
</Style>

<Style x:Key="bg-success" TargetType="VisualElement">
    <Setter Property="BackgroundColor" Value="{StaticResource Success}" />
</Style>

<Style x:Key="bg-gray-100" TargetType="VisualElement">
    <Setter Property="BackgroundColor" Value="{StaticResource Gray100}" />
</Style>

<Style x:Key="bg-white" TargetType="VisualElement">
    <Setter Property="BackgroundColor" Value="White" />
</Style>

<!-- Border Radius Utilities -->
<Style x:Key="rounded-none" TargetType="Frame">
    <Setter Property="CornerRadius" Value="0" />
</Style>

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

<Style x:Key="rounded-full" TargetType="Frame">
    <Setter Property="CornerRadius" Value="999" />
</Style>

<!-- Shadow Utilities -->
<Style x:Key="shadow-sm" TargetType="Frame">
    <Setter Property="HasShadow" Value="True" />
</Style>

<Style x:Key="shadow" TargetType="Frame">
    <Setter Property="HasShadow" Value="True" />
    <Setter Property="Shadow">
        <Shadow Brush="Black" Offset="0,2" Radius="4" Opacity="0.1" />
    </Setter>
</Style>

<Style x:Key="shadow-lg" TargetType="Frame">
    <Setter Property="HasShadow" Value="True" />
    <Setter Property="Shadow">
        <Shadow Brush="Black" Offset="0,4" Radius="8" Opacity="0.15" />
    </Setter>
</Style>
```

---

## 🌓 Dynamic Theming

### Theme Manager Service

```csharp
// Services/IThemeService.cs
namespace MyApp.Services;

public enum AppTheme
{
    System,
    Light,
    Dark,
    HighContrast
}

public interface IThemeService
{
    AppTheme CurrentTheme { get; }
    event EventHandler<AppTheme> ThemeChanged;
    
    Task SetThemeAsync(AppTheme theme);
    Task InitializeAsync();
    void ApplyTheme(AppTheme theme);
}

// Services/ThemeService.cs
public class ThemeService : IThemeService
{
    private readonly IPreferencesService _preferencesService;
    private const string ThemeKey = "app_theme";
    
    public ThemeService(IPreferencesService preferencesService)
    {
        _preferencesService = preferencesService;
    }
    
    public AppTheme CurrentTheme { get; private set; } = AppTheme.System;
    
    public event EventHandler<AppTheme> ThemeChanged;
    
    public async Task InitializeAsync()
    {
        var savedTheme = _preferencesService.Get(ThemeKey, AppTheme.System);
        await SetThemeAsync(savedTheme);
    }
    
    public async Task SetThemeAsync(AppTheme theme)
    {
        CurrentTheme = theme;
        _preferencesService.Set(ThemeKey, theme);
        
        ApplyTheme(theme);
        ThemeChanged?.Invoke(this, theme);
        
        await Task.CompletedTask;
    }
    
    public void ApplyTheme(AppTheme theme)
    {
        var mergedDictionaries = Application.Current?.Resources?.MergedDictionaries;
        if (mergedDictionaries == null) return;
        
        // Remove existing theme
        var existingTheme = mergedDictionaries.FirstOrDefault(d => 
            d.Source?.OriginalString?.Contains("Theme") == true);
        if (existingTheme != null)
        {
            mergedDictionaries.Remove(existingTheme);
        }
        
        // Apply new theme
        ResourceDictionary newTheme = theme switch
        {
            AppTheme.Light => new Themes.LightTheme(),
            AppTheme.Dark => new Themes.DarkTheme(),
            AppTheme.HighContrast => new Themes.HighContrastTheme(),
            AppTheme.System => GetSystemTheme(),
            _ => new Themes.LightTheme()
        };
        
        mergedDictionaries.Add(newTheme);
        
        // Update app theme for system controls
        Application.Current.UserAppTheme = theme switch
        {
            AppTheme.Light => Microsoft.Maui.ApplicationModel.AppTheme.Light,
            AppTheme.Dark => Microsoft.Maui.ApplicationModel.AppTheme.Dark,
            AppTheme.System => Microsoft.Maui.ApplicationModel.AppTheme.Unspecified,
            _ => Microsoft.Maui.ApplicationModel.AppTheme.Unspecified
        };
    }
    
    private ResourceDictionary GetSystemTheme()
    {
        var systemTheme = Application.Current?.RequestedTheme;
        return systemTheme == Microsoft.Maui.ApplicationModel.AppTheme.Dark 
            ? new Themes.DarkTheme() 
            : new Themes.LightTheme();
    }
}
```

### Theme Resource Dictionaries

```xml
<!-- Resources/Themes/LightTheme.xaml -->
<?xml version="1.0" encoding="UTF-8" ?>
<ResourceDictionary 
    xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
    x:Class="MyApp.Themes.LightTheme">

    <!-- Light Theme Colors -->
    <Color x:Key="PrimaryColor">#6366F1</Color>
    <Color x:Key="OnPrimaryColor">#FFFFFF</Color>
    
    <Color x:Key="SurfaceColor">#FFFFFF</Color>
    <Color x:Key="OnSurfaceColor">#0F172A</Color>
    
    <Color x:Key="BackgroundColor">#FFFFFF</Color>
    <Color x:Key="OnBackgroundColor">#0F172A</Color>
    
    <Color x:Key="CardColor">#FFFFFF</Color>
    <Color x:Key="BorderColor">#E2E8F0</Color>
    
    <Color x:Key="TextPrimaryColor">#0F172A</Color>
    <Color x:Key="TextSecondaryColor">#64748B</Color>
    <Color x:Key="TextDisabledColor">#CBD5E1</Color>

</ResourceDictionary>

<!-- Resources/Themes/DarkTheme.xaml -->
<?xml version="1.0" encoding="UTF-8" ?>
<ResourceDictionary 
    xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
    x:Class="MyApp.Themes.DarkTheme">

    <!-- Dark Theme Colors -->
    <Color x:Key="PrimaryColor">#818CF8</Color>
    <Color x:Key="OnPrimaryColor">#000000</Color>
    
    <Color x:Key="SurfaceColor">#1E293B</Color>
    <Color x:Key="OnSurfaceColor">#F1F5F9</Color>
    
    <Color x:Key="BackgroundColor">#0F172A</Color>
    <Color x:Key="OnBackgroundColor">#F1F5F9</Color>
    
    <Color x:Key="CardColor">#334155</Color>
    <Color x:Key="BorderColor">#475569</Color>
    
    <Color x:Key="TextPrimaryColor">#F1F5F9</Color>
    <Color x:Key="TextSecondaryColor">#94A3B8</Color>
    <Color x:Key="TextDisabledColor">#64748B</Color>

</ResourceDictionary>
```

### Theme-Aware Components

```xml
<!-- Theme-aware card component -->
<Frame Style="{StaticResource Card}">
    <Frame.Style>
        <Style TargetType="Frame">
            <Setter Property="BackgroundColor" Value="{DynamicResource CardColor}" />
            <Setter Property="BorderColor" Value="{DynamicResource BorderColor}" />
            <Setter Property="CornerRadius" Value="12" />
            <Setter Property="Padding" Value="16" />
            <Setter Property="HasShadow" Value="True" />
        </Style>
    </Frame.Style>
    
    <VerticalStackLayout Spacing="12">
        <Label Text="Card Title" 
               TextColor="{DynamicResource TextPrimaryColor}" 
               Style="{StaticResource H3}" />
        <Label Text="Card content that adapts to theme changes automatically." 
               TextColor="{DynamicResource TextSecondaryColor}" 
               Style="{StaticResource Body}" />
    </VerticalStackLayout>
</Frame>

<!-- Theme selector -->
<VerticalStackLayout Spacing="8">
    <Label Text="Choose Theme" Style="{StaticResource H4}" />
    
    <CollectionView ItemsSource="{Binding AvailableThemes}" 
                    SelectedItem="{Binding SelectedTheme}"
                    SelectionMode="Single">
        <CollectionView.ItemTemplate>
            <DataTemplate>
                <Grid Padding="12" ColumnDefinitions="Auto,*,Auto">
                    <Ellipse Grid.Column="0" 
                             Fill="{Binding ThemeColor}" 
                             WidthRequest="24" 
                             HeightRequest="24" />
                    <Label Grid.Column="1" 
                           Text="{Binding Name}" 
                           VerticalOptions="Center" 
                           Margin="12,0" />
                    <Image Grid.Column="2" 
                           Source="check.png" 
                           WidthRequest="20" 
                           HeightRequest="20"
                           IsVisible="{Binding IsSelected}" />
                </Grid>
            </DataTemplate>
        </CollectionView.ItemTemplate>
    </CollectionView>
</VerticalStackLayout>
```

---

## 🎨 Design Systems

### Component Library Structure

```xml
<!-- Resources/Components/Buttons.xaml -->
<?xml version="1.0" encoding="UTF-8" ?>
<ResourceDictionary 
    xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">

    <!-- Button Base Style -->
    <Style x:Key="ButtonBase" TargetType="Button">
        <Setter Property="FontFamily" Value="OpenSansSemibold" />
        <Setter Property="FontSize" Value="16" />
        <Setter Property="FontAttributes" Value="Bold" />
        <Setter Property="CornerRadius" Value="8" />
        <Setter Property="Padding" Value="16,12" />
        <Setter Property="MinimumHeightRequest" Value="48" />
        <Setter Property="VisualStateManager.VisualStateGroups">
            <VisualStateGroupList>
                <VisualStateGroup x:Name="CommonStates">
                    <VisualState x:Name="Normal">
                        <VisualState.Setters>
                            <Setter Property="Scale" Value="1" />
                            <Setter Property="Opacity" Value="1" />
                        </VisualState.Setters>
                    </VisualState>
                    <VisualState x:Name="Pressed">
                        <VisualState.Setters>
                            <Setter Property="Scale" Value="0.96" />
                        </VisualState.Setters>
                    </VisualState>
                </VisualStateGroup>
            </VisualStateGroupList>
        </Setter>
    </Style>

    <!-- Primary Button -->
    <Style x:Key="PrimaryButton" TargetType="Button" BasedOn="{StaticResource ButtonBase}">
        <Setter Property="BackgroundColor" Value="{DynamicResource PrimaryColor}" />
        <Setter Property="TextColor" Value="{DynamicResource OnPrimaryColor}" />
    </Style>

    <!-- Secondary Button -->
    <Style x:Key="SecondaryButton" TargetType="Button" BasedOn="{StaticResource ButtonBase}">
        <Setter Property="BackgroundColor" Value="Transparent" />
        <Setter Property="TextColor" Value="{DynamicResource PrimaryColor}" />
        <Setter Property="BorderColor" Value="{DynamicResource PrimaryColor}" />
        <Setter Property="BorderWidth" Value="2" />
    </Style>

    <!-- Outline Button -->
    <Style x:Key="OutlineButton" TargetType="Button" BasedOn="{StaticResource ButtonBase}">
        <Setter Property="BackgroundColor" Value="Transparent" />
        <Setter Property="TextColor" Value="{DynamicResource TextPrimaryColor}" />
        <Setter Property="BorderColor" Value="{DynamicResource BorderColor}" />
        <Setter Property="BorderWidth" Value="1" />
    </Style>

    <!-- Ghost Button -->
    <Style x:Key="GhostButton" TargetType="Button" BasedOn="{StaticResource ButtonBase}">
        <Setter Property="BackgroundColor" Value="Transparent" />
        <Setter Property="TextColor" Value="{DynamicResource PrimaryColor}" />
        <Setter Property="BorderWidth" Value="0" />
    </Style>

    <!-- Icon Button -->
    <Style x:Key="IconButton" TargetType="Button">
        <Setter Property="BackgroundColor" Value="Transparent" />
        <Setter Property="CornerRadius" Value="24" />
        <Setter Property="WidthRequest" Value="48" />
        <Setter Property="HeightRequest" Value="48" />
        <Setter Property="Padding" Value="0" />
    </Style>

    <!-- Floating Action Button -->
    <Style x:Key="FAB" TargetType="Button">
        <Setter Property="BackgroundColor" Value="{DynamicResource PrimaryColor}" />
        <Setter Property="TextColor" Value="{DynamicResource OnPrimaryColor}" />
        <Setter Property="CornerRadius" Value="28" />
        <Setter Property="WidthRequest" Value="56" />
        <Setter Property="HeightRequest" Value="56" />
        <Setter Property="FontSize" Value="24" />
        <Setter Property="Shadow">
            <Shadow Brush="Black" Offset="0,4" Radius="8" Opacity="0.3" />
        </Setter>
    </Style>

    <!-- Button Sizes -->
    <Style x:Key="ButtonSmall" TargetType="Button" BasedOn="{StaticResource ButtonBase}">
        <Setter Property="FontSize" Value="14" />
        <Setter Property="Padding" Value="12,8" />
        <Setter Property="MinimumHeightRequest" Value="36" />
    </Style>

    <Style x:Key="ButtonLarge" TargetType="Button" BasedOn="{StaticResource ButtonBase}">
        <Setter Property="FontSize" Value="18" />
        <Setter Property="Padding" Value="20,16" />
        <Setter Property="MinimumHeightRequest" Value="56" />
    </Style>

</ResourceDictionary>
```

### Card Components

```xml
<!-- Resources/Components/Cards.xaml -->
<?xml version="1.0" encoding="UTF-8" ?>
<ResourceDictionary 
    xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">

    <!-- Base Card Style -->
    <Style x:Key="CardBase" TargetType="Frame">
        <Setter Property="BackgroundColor" Value="{DynamicResource CardColor}" />
        <Setter Property="BorderColor" Value="{DynamicResource BorderColor}" />
        <Setter Property="CornerRadius" Value="12" />
        <Setter Property="Padding" Value="16" />
        <Setter Property="HasShadow" Value="True" />
        <Setter Property="Shadow">
            <Shadow Brush="Black" Offset="0,2" Radius="8" Opacity="0.1" />
        </Setter>
    </Style>

    <!-- Elevated Card -->
    <Style x:Key="CardElevated" TargetType="Frame" BasedOn="{StaticResource CardBase}">
        <Setter Property="Shadow">
            <Shadow Brush="Black" Offset="0,4" Radius="12" Opacity="0.15" />
        </Setter>
    </Style>

    <!-- Outlined Card -->
    <Style x:Key="CardOutlined" TargetType="Frame" BasedOn="{StaticResource CardBase}">
        <Setter Property="BorderWidth" Value="1" />
        <Setter Property="HasShadow" Value="False" />
    </Style>

    <!-- Filled Card -->
    <Style x:Key="CardFilled" TargetType="Frame" BasedOn="{StaticResource CardBase}">
        <Setter Property="BackgroundColor" Value="{DynamicResource SurfaceColor}" />
        <Setter Property="HasShadow" Value="False" />
    </Style>

</ResourceDictionary>
```

### Form Components

```xml
<!-- Resources/Components/Forms.xaml -->
<?xml version="1.0" encoding="UTF-8" ?>
<ResourceDictionary 
    xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">

    <!-- Form Label -->
    <Style x:Key="FormLabel" TargetType="Label">
        <Setter Property="FontSize" Value="14" />
        <Setter Property="FontAttributes" Value="Bold" />
        <Setter Property="TextColor" Value="{DynamicResource TextPrimaryColor}" />
        <Setter Property="Margin" Value="0,0,0,4" />
    </Style>

    <!-- Form Entry -->
    <Style x:Key="FormEntry" TargetType="Entry">
        <Setter Property="BackgroundColor" Value="{DynamicResource SurfaceColor}" />
        <Setter Property="TextColor" Value="{DynamicResource TextPrimaryColor}" />
        <Setter Property="PlaceholderColor" Value="{DynamicResource TextSecondaryColor}" />
        <Setter Property="FontSize" Value="16" />
        <Setter Property="HeightRequest" Value="48" />
        <Setter Property="VisualStateManager.VisualStateGroups">
            <VisualStateGroupList>
                <VisualStateGroup x:Name="CommonStates">
                    <VisualState x:Name="Normal" />
                    <VisualState x:Name="Focused">
                        <VisualState.Setters>
                            <Setter Property="BorderColor" Value="{DynamicResource PrimaryColor}" />
                            <Setter Property="BorderWidth" Value="2" />
                        </VisualState.Setters>
                    </VisualState>
                </VisualStateGroup>
            </VisualStateGroupList>
        </Setter>
    </Style>

    <!-- Form Entry Container -->
    <Style x:Key="FormEntryContainer" TargetType="Frame">
        <Setter Property="BackgroundColor" Value="{DynamicResource SurfaceColor}" />
        <Setter Property="BorderColor" Value="{DynamicResource BorderColor}" />
        <Setter Property="BorderWidth" Value="1" />
        <Setter Property="CornerRadius" Value="8" />
        <Setter Property="Padding" Value="12,8" />
        <Setter Property="HasShadow" Value="False" />
    </Style>

    <!-- Error Text -->
    <Style x:Key="ErrorText" TargetType="Label">
        <Setter Property="FontSize" Value="12" />
        <Setter Property="TextColor" Value="{StaticResource Danger}" />
        <Setter Property="Margin" Value="0,4,0,0" />
    </Style>

    <!-- Helper Text -->
    <Style x:Key="HelperText" TargetType="Label">
        <Setter Property="FontSize" Value="12" />
        <Setter Property="TextColor" Value="{DynamicResource TextSecondaryColor}" />
        <Setter Property="Margin" Value="0,4,0,0" />
    </Style>

</ResourceDictionary>
```

---

## 🎯 Custom Controls Styling

### Custom Control Templates

```xml
<!-- Custom Button Template -->
<ControlTemplate x:Key="CustomButtonTemplate">
    <Frame BackgroundColor="{TemplateBinding BackgroundColor}"
           BorderColor="{TemplateBinding BorderColor}"
           CornerRadius="8"
           HasShadow="True"
           Padding="0">
        
        <Grid>
            <!-- Background gradient -->
            <RoundRectangle>
                <RoundRectangle.Fill>
                    <LinearGradientBrush StartPoint="0,0" EndPoint="0,1">
                        <GradientStop Color="{TemplateBinding BackgroundColor}" Offset="0.0" />
                        <GradientStop Color="#99{TemplateBinding BackgroundColor}" Offset="1.0" />
                    </LinearGradientBrush>
                </RoundRectangle.Fill>
            </RoundRectangle>
            
            <!-- Content presenter -->
            <ContentPresenter 
                HorizontalOptions="Center" 
                VerticalOptions="Center"
                Padding="{TemplateBinding Padding}" />
            
            <!-- Loading indicator -->
            <ActivityIndicator 
                IsVisible="{TemplateBinding IsEnabled, Converter={StaticResource InvertedBoolConverter}}"
                IsRunning="{TemplateBinding IsEnabled, Converter={StaticResource InvertedBoolConverter}}"
                Color="{TemplateBinding TextColor}"
                HorizontalOptions="Center"
                VerticalOptions="Center" />
        </Grid>
        
        <VisualStateManager.VisualStateGroups>
            <VisualStateGroup x:Name="CommonStates">
                <VisualState x:Name="Normal" />
                <VisualState x:Name="Pressed">
                    <VisualState.Setters>
                        <Setter TargetName="MainFrame" Property="Scale" Value="0.96" />
                    </VisualState.Setters>
                </VisualState>
                <VisualState x:Name="Disabled">
                    <VisualState.Setters>
                        <Setter TargetName="MainFrame" Property="Opacity" Value="0.6" />
                    </VisualState.Setters>
                </VisualState>
            </VisualStateGroup>
        </VisualStateManager.VisualStateGroups>
    </Frame>
</ControlTemplate>

<!-- Custom Entry Template -->
<ControlTemplate x:Key="CustomEntryTemplate">
    <Frame BackgroundColor="{DynamicResource SurfaceColor}"
           BorderColor="{DynamicResource BorderColor}"
           CornerRadius="8"
           Padding="12,8"
           HasShadow="False">
        
        <Grid ColumnDefinitions="*,Auto">
            <!-- Entry field -->
            <Entry Grid.Column="0"
                   Text="{TemplateBinding Text}"
                   Placeholder="{TemplateBinding Placeholder}"
                   TextColor="{DynamicResource TextPrimaryColor}"
                   BackgroundColor="Transparent"
                   BorderWidth="0" />
            
            <!-- Clear button -->
            <Button Grid.Column="1"
                    Text="✕"
                    BackgroundColor="Transparent"
                    TextColor="{DynamicResource TextSecondaryColor}"
                    WidthRequest="24"
                    HeightRequest="24"
                    FontSize="12"
                    IsVisible="{TemplateBinding Text, Converter={StaticResource StringToBoolConverter}}"
                    Command="{Binding ClearCommand, Source={x:Reference TemplatedParent}}" />
        </Grid>
    </Frame>
</ControlTemplate>
```

### Animated Controls

```xml
<!-- Animated Card with Hover Effect -->
<Style x:Key="AnimatedCard" TargetType="Frame">
    <Setter Property="BackgroundColor" Value="{DynamicResource CardColor}" />
    <Setter Property="CornerRadius" Value="12" />
    <Setter Property="Padding" Value="16" />
    <Setter Property="HasShadow" Value="True" />
    <Setter Property="Scale" Value="1" />
    
    <Setter Property="VisualStateManager.VisualStateGroups">
        <VisualStateGroupList>
            <VisualStateGroup x:Name="CommonStates">
                <VisualState x:Name="Normal">
                    <VisualState.Setters>
                        <Setter Property="Scale" Value="1" />
                        <Setter Property="Shadow">
                            <Shadow Brush="Black" Offset="0,2" Radius="4" Opacity="0.1" />
                        </Setter>
                    </VisualState.Setters>
                </VisualState>
                
                <VisualState x:Name="PointerOver">
                    <VisualState.Setters>
                        <Setter Property="Scale" Value="1.02" />
                        <Setter Property="Shadow">
                            <Shadow Brush="Black" Offset="0,8" Radius="16" Opacity="0.2" />
                        </Setter>
                    </VisualState.Setters>
                </VisualState>
            </VisualStateGroup>
        </VisualStateGroupList>
    </Setter>
</Style>

<!-- Floating Label Entry -->
<Grid>
    <Entry x:Name="FloatingEntry" 
           Text="{Binding Value}"
           BackgroundColor="Transparent"
           Margin="0,16,0,0" />
    
    <Label x:Name="FloatingLabel" 
           Text="Enter text"
           TextColor="{DynamicResource TextSecondaryColor}"
           VerticalOptions="Center"
           Margin="8,0">
        
        <Label.Triggers>
            <DataTrigger TargetType="Label" 
                         Binding="{Binding Source={x:Reference FloatingEntry}, Path=IsFocused}" 
                         Value="True">
                <DataTrigger.EnterActions>
                    <BeginAnimation>
                        <Animation Property="TranslationY" To="-20" Duration="200" />
                        <Animation Property="FontSize" To="12" Duration="200" />
                    </BeginAnimation>
                </DataTrigger.EnterActions>
                <DataTrigger.ExitActions>
                    <BeginAnimation>
                        <Animation Property="TranslationY" To="0" Duration="200" />
                        <Animation Property="FontSize" To="16" Duration="200" />
                    </BeginAnimation>
                </DataTrigger.ExitActions>
            </DataTrigger>
        </Label.Triggers>
    </Label>
</Grid>
```

---

## ⚡ Performance & Best Practices

### Styling Performance Tips

```csharp
// 1. Use StaticResource for static values
<Label TextColor="{StaticResource Primary}" />

// 2. Use DynamicResource for theme-aware values
<Label TextColor="{DynamicResource TextPrimaryColor}" />

// 3. Minimize resource lookups in loops
public void OptimizedListCreation()
{
    var primaryColor = (Color)Application.Current.Resources["Primary"];
    
    foreach (var item in items)
    {
        var label = new Label
        {
            Text = item.Name,
            TextColor = primaryColor // Use cached value
        };
    }
}

// 4. Use compiled bindings for better performance
[XamlCompilation(XamlCompilationOptions.Compile)]
public partial class OptimizedPage : ContentPage
{
    // Compiled XAML performs better
}
```

### Resource Organization Best Practices

```xml
<!-- App.xaml - Organize resources hierarchically -->
<Application.Resources>
    <ResourceDictionary>
        
        <!-- Merge dictionaries in order of dependency -->
        <ResourceDictionary.MergedDictionaries>
            <!-- 1. Base resources (colors, fonts) -->
            <ResourceDictionary Source="Resources/Styles/Colors.xaml" />
            <ResourceDictionary Source="Resources/Styles/Typography.xaml" />
            
            <!-- 2. Layout utilities -->
            <ResourceDictionary Source="Resources/Styles/Layouts.xaml" />
            
            <!-- 3. Component styles -->
            <ResourceDictionary Source="Resources/Components/Buttons.xaml" />
            <ResourceDictionary Source="Resources/Components/Cards.xaml" />
            <ResourceDictionary Source="Resources/Components/Forms.xaml" />
            
            <!-- 4. Theme (last to override) -->
            <ResourceDictionary Source="Resources/Themes/LightTheme.xaml" />
        </ResourceDictionary.MergedDictionaries>
        
    </ResourceDictionary>
</Application.Resources>
```

### CSS-like Workflow

```xml
<!-- Usage example showing CSS-like approach -->
<ScrollView>
    <StackLayout Style="{StaticResource p-4}"> <!-- padding: 16px -->
        
        <!-- Hero section -->
        <Frame Style="{StaticResource bg-primary}" 
               Style="{StaticResource rounded-lg}"
               Style="{StaticResource p-6}"> <!-- Multiple styles merged -->
            <Label Text="Welcome" 
                   Style="{StaticResource H1}"
                   Style="{StaticResource text-center}"
                   TextColor="White" />
        </Frame>
        
        <!-- Card grid -->
        <Grid Style="{StaticResource grid-cols-2}" 
              Style="{StaticResource gap-4}"
              Style="{StaticResource mt-6}">
            
            <Frame Grid.Column="0" Style="{StaticResource card}">
                <Label Text="Card 1" Style="{StaticResource H3}" />
            </Frame>
            
            <Frame Grid.Column="1" Style="{StaticResource card}">
                <Label Text="Card 2" Style="{StaticResource H3}" />
            </Frame>
            
        </Grid>
        
    </StackLayout>
</ScrollView>
```

---

## 🎯 Exercises

### Exercise 1: Theme System
สร้าง complete theme system ที่มี:
- Light และ Dark themes
- Theme switching functionality
- Theme-aware components
- Persistent theme selection

### Exercise 2: Component Library
สร้าง reusable component library:
- Button variants (Primary, Secondary, Outline, Ghost)
- Card components (Elevated, Outlined, Filled)
- Form controls พร้อม validation styles
- CSS-like utility classes

### Exercise 3: Design System
ออกแบบ design system สำหรับ brand:
- Color palette และ typography scale
- Spacing และ sizing system
- Component documentation
- Usage guidelines

---

## 📝 Quiz

1. **Resource Performance**: เมื่อไหร่ควรใช้ StaticResource vs DynamicResource?
2. **Style Inheritance**: วิธีการ inherit style และ override properties?
3. **Theme Management**: วิธีการสร้าง theme switching ที่มีประสิทธิภาพ?

<details>
<summary>เฉลย</summary>

1. StaticResource สำหรับค่าที่ไม่เปลี่ยน, DynamicResource สำหรับค่าที่อาจเปลี่ยนแปลง (themes)
2. ใช้ BasedOn="{StaticResource BaseStyle}" และ override ด้วย Setter ใหม่
3. ใช้ ResourceDictionary switching และ DynamicResource references

</details>

---

## 🎉 สรุป

ในบทเรียนนี้ คุณได้เรียนรู้:

✅ **Styling Fundamentals** - Styles, Resources, และ Visual States  
✅ **Resource Management** - Organization และ performance optimization  
✅ **CSS-like Utilities** - Spacing, layout, และ styling utilities  
✅ **Dynamic Theming** - Theme switching และ management  
✅ **Design Systems** - Component libraries และ design tokens  
✅ **Custom Controls** - Advanced styling และ templates  
✅ **Best Practices** - Performance และ maintainability

### Next Steps
🔜 **บทที่ 8**: Navigation Patterns - การนำทางแบบต่างๆ  
🔜 **บทที่ 9**: Advanced Layouts - Complex UI patterns

---

**Happy Styling! 🎨**

> บทเรียนต่อไป: [08-navigation-patterns.md](08-navigation-patterns.md)
