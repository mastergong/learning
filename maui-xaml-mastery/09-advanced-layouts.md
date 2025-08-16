# 📐 Chapter 9: Advanced Layouts & Responsive Design

> **หัวข้อ**: Advanced Layout Techniques & Responsive Design  
> **ระยะเวลา**: 3 ชั่วโมง  
> **ความยากง่าย**: ระดับกลาง  
> **Prerequisites**: บทที่ 1-8 (Foundation Phase)

## 🎯 Learning Objectives

หลังจากเรียนบทนี้แล้ว คุณจะสามารถ:

- ✅ สร้าง responsive layouts ที่ทำงานได้ทุกหน้าจอ
- ✅ ใช้ advanced layout techniques อย่างมืออาชีพ
- ✅ ปรับแต่ง layout ตาม platform และ orientation
- ✅ สร้าง complex UI ด้วย nested layouts
- ✅ เพิ่มประสิทธิภาพ layout performance

---

## 📱 Chapter Overview

### 🏗️ What We'll Build
สร้าง **"Responsive Dashboard App"** ที่มี:
- 📊 Multi-column responsive grid
- 📱 Adaptive navigation patterns
- 🔄 Dynamic layout switching
- 📈 Chart and data visualization
- 🎨 CSS-like responsive utilities

---

## 1. 🎨 CSS-like Layout Utilities

### 1.1 Responsive Grid System

```xml
<!-- d:\A-Work\learn from ai\maui-xaml-mastery\src\09-AdvancedLayouts\Styles\LayoutUtilities.xaml -->
<ResourceDictionary xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
                   xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">

    <!-- CSS-like Grid Utilities -->
    <Style x:Key="GridContainer" TargetType="Grid">
        <Setter Property="Padding" Value="16" />
        <Setter Property="RowSpacing" Value="12" />
        <Setter Property="ColumnSpacing" Value="12" />
    </Style>

    <!-- Responsive Column Definitions -->
    <Style x:Key="TwoColumnGrid" TargetType="Grid" BasedOn="{StaticResource GridContainer}">
        <Setter Property="ColumnDefinitions" Value="*, *" />
    </Style>

    <Style x:Key="ThreeColumnGrid" TargetType="Grid" BasedOn="{StaticResource GridContainer}">
        <Setter Property="ColumnDefinitions" Value="*, *, *" />
    </Style>

    <Style x:Key="AsymmetricGrid" TargetType="Grid" BasedOn="{StaticResource GridContainer}">
        <Setter Property="ColumnDefinitions" Value="2*, *, *" />
    </Style>

    <!-- Responsive Breakpoints -->
    <OnPlatform x:Key="ResponsiveColumns" x:TypeArguments="x:String">
        <On Platform="Android, iOS" Value="*" />
        <On Platform="WinUI" Value="*, *, *" />
        <On Platform="MacCatalyst" Value="*, *" />
    </OnPlatform>

    <!-- Spacing Utilities -->
    <Style x:Key="SpaceY8" TargetType="StackLayout">
        <Setter Property="Spacing" Value="8" />
    </Style>

    <Style x:Key="SpaceY16" TargetType="StackLayout">
        <Setter Property="Spacing" Value="16" />
    </Style>

    <Style x:Key="SpaceY24" TargetType="StackLayout">
        <Setter Property="Spacing" Value="24" />
    </Style>

</ResourceDictionary>
```

### 1.2 Flexbox-like Layout

```xml
<!-- FlexLayout with CSS-like properties -->
<FlexLayout x:Name="ResponsiveContainer"
           Direction="Row"
           Wrap="Wrap"
           JustifyContent="SpaceBetween"
           AlignItems="Center"
           AlignContent="Start">

    <!-- Flex Items with CSS-like flex properties -->
    <Frame FlexLayout.Grow="1" 
           FlexLayout.Basis="300">
        <Label Text="Flexible Item 1" />
    </Frame>

    <Frame FlexLayout.Grow="2" 
           FlexLayout.Basis="200">
        <Label Text="Flexible Item 2" />
    </Frame>

    <Frame FlexLayout.Grow="1" 
           FlexLayout.Basis="250">
        <Label Text="Flexible Item 3" />
    </Frame>

</FlexLayout>
```

---

## 2. 📱 Adaptive Layout Patterns

### 2.1 Responsive Dashboard Layout

```xml
<!-- d:\A-Work\learn from ai\maui-xaml-mastery\src\09-AdvancedLayouts\Views\ResponsiveDashboard.xaml -->
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="AdvancedLayouts.Views.ResponsiveDashboard"
             Title="Responsive Dashboard">

    <ContentPage.Resources>
        <ResourceDictionary>
            <Style x:Key="DashboardCard" TargetType="Frame">
                <Setter Property="BackgroundColor" Value="{AppThemeBinding Light={StaticResource White}, Dark={StaticResource Gray900}}" />
                <Setter Property="BorderColor" Value="{AppThemeBinding Light={StaticResource Gray200}, Dark={StaticResource Gray700}}" />
                <Setter Property="CornerRadius" Value="12" />
                <Setter Property="Padding" Value="16" />
                <Setter Property="Margin" Value="0,0,0,16" />
                <Setter Property="HasShadow" Value="True" />
            </Style>
        </ResourceDictionary>
    </ContentPage.Resources>

    <ScrollView>
        <Grid x:Name="MainGrid" 
              Style="{StaticResource GridContainer}">
            
            <!-- Auto-adjusting columns based on screen size -->
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="{Binding ColumnWidth1}" />
                <ColumnDefinition Width="{Binding ColumnWidth2}" />
                <ColumnDefinition Width="{Binding ColumnWidth3}" />
            </Grid.ColumnDefinitions>

            <!-- Header Section (Full Width) -->
            <StackLayout Grid.ColumnSpan="3" 
                        Style="{StaticResource SpaceY16}">
                
                <Label Text="Dashboard Overview" 
                       FontSize="28" 
                       FontAttributes="Bold"
                       HorizontalOptions="Center" />
                
                <!-- Stats Cards Row -->
                <FlexLayout Direction="Row" 
                           Wrap="Wrap" 
                           JustifyContent="SpaceEvenly">
                    
                    <Frame Style="{StaticResource DashboardCard}"
                           FlexLayout.Basis="200">
                        <StackLayout>
                            <Label Text="Total Sales" 
                                   FontSize="14" 
                                   TextColor="{StaticResource Gray600}" />
                            <Label Text="$124,500" 
                                   FontSize="24" 
                                   FontAttributes="Bold"
                                   TextColor="{StaticResource Primary}" />
                            <Label Text="+12.5%" 
                                   FontSize="12" 
                                   TextColor="{StaticResource Success}" />
                        </StackLayout>
                    </Frame>

                    <Frame Style="{StaticResource DashboardCard}"
                           FlexLayout.Basis="200">
                        <StackLayout>
                            <Label Text="New Users" 
                                   FontSize="14" 
                                   TextColor="{StaticResource Gray600}" />
                            <Label Text="2,341" 
                                   FontSize="24" 
                                   FontAttributes="Bold"
                                   TextColor="{StaticResource Secondary}" />
                            <Label Text="+5.2%" 
                                   FontSize="12" 
                                   TextColor="{StaticResource Success}" />
                        </StackLayout>
                    </Frame>

                    <Frame Style="{StaticResource DashboardCard}"
                           FlexLayout.Basis="200">
                        <StackLayout>
                            <Label Text="Active Sessions" 
                                   FontSize="14" 
                                   TextColor="{StaticResource Gray600}" />
                            <Label Text="892" 
                                   FontSize="24" 
                                   FontAttributes="Bold"
                                   TextColor="{StaticResource Tertiary}" />
                            <Label Text="-2.1%" 
                                   FontSize="12" 
                                   TextColor="{StaticResource Danger}" />
                        </StackLayout>
                    </Frame>

                </FlexLayout>
            </StackLayout>

            <!-- Main Content Area -->
            <Frame Grid.Row="1" Grid.Column="0" Grid.ColumnSpan="2"
                   Style="{StaticResource DashboardCard}">
                <StackLayout>
                    <Label Text="Revenue Chart" 
                           FontSize="18" 
                           FontAttributes="Bold" />
                    
                    <!-- Placeholder for Chart -->
                    <Rectangle HeightRequest="300"
                              Fill="{StaticResource Gray100}"
                              Stroke="{StaticResource Gray300}"
                              StrokeThickness="1" />
                    
                    <Label Text="Chart will be implemented in later chapters"
                           HorizontalOptions="Center"
                           TextColor="{StaticResource Gray500}" />
                </StackLayout>
            </Frame>

            <!-- Sidebar Content -->
            <StackLayout Grid.Row="1" Grid.Column="2"
                        Style="{StaticResource SpaceY16}">
                
                <Frame Style="{StaticResource DashboardCard}">
                    <StackLayout>
                        <Label Text="Recent Activity" 
                               FontSize="16" 
                               FontAttributes="Bold" />
                        
                        <CollectionView ItemsSource="{Binding RecentActivities}"
                                       HeightRequest="200">
                            <CollectionView.ItemTemplate>
                                <DataTemplate>
                                    <Grid Padding="0,8">
                                        <Grid.ColumnDefinitions>
                                            <ColumnDefinition Width="Auto" />
                                            <ColumnDefinition Width="*" />
                                        </Grid.ColumnDefinitions>
                                        
                                        <Ellipse Grid.Column="0"
                                                WidthRequest="8"
                                                HeightRequest="8"
                                                Fill="{StaticResource Primary}"
                                                VerticalOptions="Center" />
                                        
                                        <StackLayout Grid.Column="1" Margin="12,0,0,0">
                                            <Label Text="{Binding Title}" 
                                                   FontSize="14" />
                                            <Label Text="{Binding Time}" 
                                                   FontSize="12" 
                                                   TextColor="{StaticResource Gray500}" />
                                        </StackLayout>
                                    </Grid>
                                </DataTemplate>
                            </CollectionView.ItemTemplate>
                        </CollectionView>
                    </StackLayout>
                </Frame>

                <Frame Style="{StaticResource DashboardCard}">
                    <StackLayout>
                        <Label Text="Quick Actions" 
                               FontSize="16" 
                               FontAttributes="Bold" />
                        
                        <Button Text="Export Data" 
                                Style="{StaticResource PrimaryButton}"
                                Margin="0,8,0,0" />
                        
                        <Button Text="Generate Report" 
                                Style="{StaticResource SecondaryButton}"
                                Margin="0,8,0,0" />
                        
                        <Button Text="Settings" 
                                Style="{StaticResource OutlineButton}"
                                Margin="0,8,0,0" />
                    </StackLayout>
                </Frame>

            </StackLayout>

        </Grid>
    </ScrollView>

</ContentPage>
```

### 2.2 Responsive Dashboard ViewModel

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\09-AdvancedLayouts\ViewModels\ResponsiveDashboardViewModel.cs
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;
using System.Collections.ObjectModel;

namespace AdvancedLayouts.ViewModels;

public partial class ResponsiveDashboardViewModel : ObservableObject
{
    [ObservableProperty]
    private string columnWidth1 = "*";
    
    [ObservableProperty]
    private string columnWidth2 = "*";
    
    [ObservableProperty]
    private string columnWidth3 = "*";

    [ObservableProperty]
    private ObservableCollection<ActivityItem> recentActivities = new();

    public ResponsiveDashboardViewModel()
    {
        LoadData();
        UpdateLayoutForCurrentSize();
    }

    private void LoadData()
    {
        RecentActivities.Add(new ActivityItem("New order received", "2 minutes ago"));
        RecentActivities.Add(new ActivityItem("User registered", "5 minutes ago"));
        RecentActivities.Add(new ActivityItem("Payment processed", "8 minutes ago"));
        RecentActivities.Add(new ActivityItem("Report generated", "12 minutes ago"));
        RecentActivities.Add(new ActivityItem("Backup completed", "15 minutes ago"));
    }

    public void UpdateLayoutForCurrentSize()
    {
        var displayInfo = DeviceDisplay.Current.MainDisplayInfo;
        var width = displayInfo.Width / displayInfo.Density;

        if (width < 768) // Mobile
        {
            ColumnWidth1 = "*";
            ColumnWidth2 = "0";
            ColumnWidth3 = "0";
        }
        else if (width < 1024) // Tablet
        {
            ColumnWidth1 = "2*";
            ColumnWidth2 = "*";
            ColumnWidth3 = "0";
        }
        else // Desktop
        {
            ColumnWidth1 = "2*";
            ColumnWidth2 = "*";
            ColumnWidth3 = "*";
        }
    }

    [RelayCommand]
    private async Task RefreshData()
    {
        // Simulate data refresh
        await Task.Delay(1000);
        LoadData();
    }
}

public class ActivityItem
{
    public string Title { get; set; }
    public string Time { get; set; }

    public ActivityItem(string title, string time)
    {
        Title = title;
        Time = time;
    }
}
```

---

## 3. 🔄 Advanced Layout Behaviors

### 3.1 Orientation-Aware Layout

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\09-AdvancedLayouts\Behaviors\OrientationAwareLayoutBehavior.cs
using Microsoft.Maui.Controls;

namespace AdvancedLayouts.Behaviors;

public class OrientationAwareLayoutBehavior : Behavior<Grid>
{
    public static readonly BindableProperty PortraitTemplateProperty =
        BindableProperty.Create(nameof(PortraitTemplate), typeof(DataTemplate), typeof(OrientationAwareLayoutBehavior));

    public static readonly BindableProperty LandscapeTemplateProperty =
        BindableProperty.Create(nameof(LandscapeTemplate), typeof(DataTemplate), typeof(OrientationAwareLayoutBehavior));

    public DataTemplate PortraitTemplate
    {
        get => (DataTemplate)GetValue(PortraitTemplateProperty);
        set => SetValue(PortraitTemplateProperty, value);
    }

    public DataTemplate LandscapeTemplate
    {
        get => (DataTemplate)GetValue(LandscapeTemplateProperty);
        set => SetValue(LandscapeTemplateProperty, value);
    }

    private Grid _attachedGrid;

    protected override void OnAttachedTo(Grid bindable)
    {
        _attachedGrid = bindable;
        DeviceDisplay.Current.MainDisplayInfoChanged += OnDisplayInfoChanged;
        UpdateLayout();
        base.OnAttachedTo(bindable);
    }

    protected override void OnDetachingFrom(Grid bindable)
    {
        DeviceDisplay.Current.MainDisplayInfoChanged -= OnDisplayInfoChanged;
        _attachedGrid = null;
        base.OnDetachingFrom(bindable);
    }

    private void OnDisplayInfoChanged(object sender, DisplayInfoChangedEventArgs e)
    {
        UpdateLayout();
    }

    private void UpdateLayout()
    {
        if (_attachedGrid == null) return;

        var displayInfo = DeviceDisplay.Current.MainDisplayInfo;
        var isLandscape = displayInfo.Width > displayInfo.Height;

        var template = isLandscape ? LandscapeTemplate : PortraitTemplate;
        
        if (template != null)
        {
            _attachedGrid.Children.Clear();
            
            if (template.CreateContent() is View content)
            {
                _attachedGrid.Children.Add(content);
            }
        }
    }
}
```

### 3.2 Adaptive Container

```xml
<!-- Usage Example -->
<Grid>
    <Grid.Behaviors>
        <behaviors:OrientationAwareLayoutBehavior>
            <behaviors:OrientationAwareLayoutBehavior.PortraitTemplate>
                <DataTemplate>
                    <StackLayout Orientation="Vertical" Spacing="16">
                        <Label Text="Portrait Layout" />
                        <Frame BackgroundColor="{StaticResource Primary}">
                            <Label Text="Content Block 1" TextColor="White" />
                        </Frame>
                        <Frame BackgroundColor="{StaticResource Secondary}">
                            <Label Text="Content Block 2" TextColor="White" />
                        </Frame>
                    </StackLayout>
                </DataTemplate>
            </behaviors:OrientationAwareLayoutBehavior.PortraitTemplate>
            
            <behaviors:OrientationAwareLayoutBehavior.LandscapeTemplate>
                <DataTemplate>
                    <Grid ColumnDefinitions="*, *" ColumnSpacing="16">
                        <StackLayout Grid.Column="0">
                            <Label Text="Landscape Layout" />
                            <Frame BackgroundColor="{StaticResource Primary}">
                                <Label Text="Left Panel" TextColor="White" />
                            </Frame>
                        </StackLayout>
                        <Frame Grid.Column="1" BackgroundColor="{StaticResource Secondary}">
                            <Label Text="Right Panel" TextColor="White" />
                        </Frame>
                    </Grid>
                </DataTemplate>
            </behaviors:OrientationAwareLayoutBehavior.LandscapeTemplate>
        </behaviors:OrientationAwareLayoutBehavior>
    </Grid.Behaviors>
</Grid>
```

---

## 4. 🎨 Complex Layout Patterns

### 4.1 Masonry Layout (Pinterest-style)

```xml
<!-- d:\A-Work\learn from ai\maui-xaml-mastery\src\09-AdvancedLayouts\Views\MasonryLayout.xaml -->
<ScrollView>
    <FlexLayout x:Name="MasonryContainer"
               Direction="Row"
               Wrap="Wrap"
               JustifyContent="Start"
               AlignContent="Start">
        
        <!-- Dynamic items will be added programmatically -->
        
    </FlexLayout>
</ScrollView>
```

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\09-AdvancedLayouts\Views\MasonryLayout.xaml.cs
public partial class MasonryLayout : ContentView
{
    private readonly Random _random = new();
    
    public MasonryLayout()
    {
        InitializeComponent();
        CreateMasonryItems();
    }

    private void CreateMasonryItems()
    {
        var colors = new[]
        {
            Colors.Red, Colors.Blue, Colors.Green, Colors.Orange,
            Colors.Purple, Colors.Teal, Colors.Pink, Colors.Brown
        };

        for (int i = 0; i < 20; i++)
        {
            var height = _random.Next(100, 300);
            var width = 150; // Fixed width for masonry effect
            
            var frame = new Frame
            {
                BackgroundColor = colors[i % colors.Length],
                WidthRequest = width,
                HeightRequest = height,
                Margin = new Thickness(4),
                CornerRadius = 8,
                Content = new Label
                {
                    Text = $"Item {i + 1}",
                    TextColor = Colors.White,
                    HorizontalOptions = LayoutOptions.Center,
                    VerticalOptions = LayoutOptions.Center,
                    FontAttributes = FontAttributes.Bold
                }
            };

            // Set FlexLayout properties for masonry effect
            FlexLayout.SetBasis(frame, width);
            FlexLayout.SetGrow(frame, 0);
            FlexLayout.SetShrink(frame, 0);

            MasonryContainer.Children.Add(frame);
        }
    }
}
```

### 4.2 Card-based Layout with Drag & Drop

```xml
<!-- d:\A-Work\learn from ai\maui-xaml-mastery\src\09-AdvancedLayouts\Views\DraggableCardLayout.xaml -->
<Grid>
    <CollectionView x:Name="CardsCollectionView"
                   ItemsSource="{Binding Cards}"
                   SelectionMode="None">
        
        <CollectionView.ItemsLayout>
            <GridItemsLayout Orientation="Vertical" 
                            Span="2" 
                            HorizontalItemSpacing="12"
                            VerticalItemSpacing="12" />
        </CollectionView.ItemsLayout>

        <CollectionView.ItemTemplate>
            <DataTemplate>
                <Frame Style="{StaticResource DashboardCard}"
                       HeightRequest="{Binding Height}">
                    
                    <Frame.GestureRecognizers>
                        <TapGestureRecognizer Command="{Binding Source={x:Reference CardsCollectionView}, Path=BindingContext.SelectCardCommand}"
                                             CommandParameter="{Binding}" />
                        <PanGestureRecognizer PanUpdated="OnCardPanUpdated" />
                    </Frame.GestureRecognizers>

                    <Grid>
                        <Grid.RowDefinitions>
                            <RowDefinition Height="Auto" />
                            <RowDefinition Height="*" />
                            <RowDefinition Height="Auto" />
                        </Grid.RowDefinitions>

                        <!-- Card Header -->
                        <StackLayout Grid.Row="0" Orientation="Horizontal">
                            <Label Text="{Binding Icon}" 
                                   FontFamily="MaterialIcons"
                                   FontSize="24"
                                   TextColor="{Binding IconColor}" />
                            <Label Text="{Binding Title}" 
                                   FontSize="16"
                                   FontAttributes="Bold"
                                   VerticalOptions="Center" />
                        </StackLayout>

                        <!-- Card Content -->
                        <Label Grid.Row="1"
                               Text="{Binding Content}"
                               FontSize="14"
                               TextColor="{StaticResource Gray600}"
                               Margin="0,8" />

                        <!-- Card Footer -->
                        <StackLayout Grid.Row="2" Orientation="Horizontal">
                            <Label Text="{Binding Category}" 
                                   FontSize="12"
                                   TextColor="{StaticResource Gray500}" />
                            <Label Text="{Binding LastUpdated, StringFormat='Updated {0:MMM dd}'}" 
                                   FontSize="12"
                                   TextColor="{StaticResource Gray500}"
                                   HorizontalOptions="EndAndExpand" />
                        </StackLayout>

                        <!-- Drag Indicator -->
                        <BoxView Grid.RowSpan="3"
                                IsVisible="{Binding IsDragging}"
                                Color="{StaticResource Primary}"
                                Opacity="0.1" />

                    </Grid>
                </Frame>
            </DataTemplate>
        </CollectionView.ItemTemplate>
    </CollectionView>
</Grid>
```

---

## 5. ⚡ Layout Performance Optimization

### 5.1 Virtualization Strategies

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\09-AdvancedLayouts\Optimizations\VirtualizedLayoutManager.cs
using System.Collections.ObjectModel;

namespace AdvancedLayouts.Optimizations;

public class VirtualizedLayoutManager
{
    private readonly ObservableCollection<object> _visibleItems = new();
    private readonly List<object> _allItems = new();
    private int _viewportStartIndex = 0;
    private int _viewportSize = 10;

    public ObservableCollection<object> VisibleItems => _visibleItems;

    public void Initialize(IEnumerable<object> items)
    {
        _allItems.Clear();
        _allItems.AddRange(items);
        UpdateVisibleItems();
    }

    public void OnScrollPositionChanged(double scrollY, double itemHeight)
    {
        var newStartIndex = Math.Max(0, (int)(scrollY / itemHeight) - 2);
        
        if (newStartIndex != _viewportStartIndex)
        {
            _viewportStartIndex = newStartIndex;
            UpdateVisibleItems();
        }
    }

    private void UpdateVisibleItems()
    {
        _visibleItems.Clear();
        
        var endIndex = Math.Min(_allItems.Count, _viewportStartIndex + _viewportSize);
        
        for (int i = _viewportStartIndex; i < endIndex; i++)
        {
            _visibleItems.Add(_allItems[i]);
        }
    }
}
```

### 5.2 Layout Caching

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\09-AdvancedLayouts\Optimizations\LayoutCache.cs
using System.Collections.Concurrent;

namespace AdvancedLayouts.Optimizations;

public class LayoutCache
{
    private readonly ConcurrentDictionary<string, Size> _measureCache = new();
    private readonly ConcurrentDictionary<string, Rect> _arrangeCache = new();

    public Size? GetCachedMeasure(string key)
    {
        return _measureCache.TryGetValue(key, out var size) ? size : null;
    }

    public void CacheMeasure(string key, Size size)
    {
        _measureCache[key] = size;
    }

    public Rect? GetCachedArrange(string key)
    {
        return _arrangeCache.TryGetValue(key, out var rect) ? rect : null;
    }

    public void CacheArrange(string key, Rect rect)
    {
        _arrangeCache[key] = rect;
    }

    public void ClearCache()
    {
        _measureCache.Clear();
        _arrangeCache.Clear();
    }
}
```

---

## 6. 🧪 Practical Exercises

### Exercise 1: Responsive Product Grid
สร้าง product grid ที่ปรับจำนวน columns ตามขนาดหน้าจอ:
- Mobile: 1 column
- Tablet: 2 columns  
- Desktop: 3-4 columns

### Exercise 2: Dashboard Widget System
สร้างระบบ dashboard ที่ widgets สามารถ:
- Drag & drop เพื่อเรียงลำดับใหม่
- Resize ตามต้องการ
- Hide/show ตามการตั้งค่า

### Exercise 3: Adaptive Navigation
สร้าง navigation ที่เปลี่ยนแปลงตาม:
- Screen orientation
- Platform (iOS vs Android vs Windows)
- User preferences

---

## 7. 📝 Best Practices & Tips

### 🎯 Layout Performance Tips

```csharp
// ✅ DO: Use appropriate layout for the task
// Grid for structured layouts
// StackLayout for simple sequential layouts  
// FlexLayout for responsive designs

// ✅ DO: Minimize layout nesting
<Grid>
    <Label Text="Simple is better" />
</Grid>

// ❌ DON'T: Over-nest layouts
<StackLayout>
    <Grid>
        <StackLayout>
            <Label Text="Too much nesting" />
        </StackLayout>
    </Grid>
</StackLayout>

// ✅ DO: Use virtualization for large lists
<CollectionView ItemsSource="{Binding LargeDataSet}">
    <!-- Automatically virtualizes items -->
</CollectionView>

// ✅ DO: Cache layout calculations
public class OptimizedLayout : Layout
{
    private Size? _cachedMeasureResult;
    
    protected override Size MeasureOverride(double widthConstraint, double heightConstraint)
    {
        var key = $"{widthConstraint}-{heightConstraint}";
        if (_cachedMeasureResult.HasValue)
            return _cachedMeasureResult.Value;
            
        // Perform measure...
        return result;
    }
}
```

### 🔧 CSS-like Best Practices

```xml
<!-- ✅ DO: Use semantic class names -->
<Style x:Key="CardContainer" TargetType="Frame">
    <Setter Property="BackgroundColor" Value="{StaticResource SurfaceColor}" />
    <Setter Property="Padding" Value="16" />
    <Setter Property="CornerRadius" Value="8" />
</Style>

<!-- ✅ DO: Create responsive utilities -->
<Style x:Key="ResponsiveGrid" TargetType="Grid">
    <Setter Property="ColumnDefinitions">
        <Setter.Value>
            <OnPlatform x:TypeArguments="x:String">
                <On Platform="Android,iOS" Value="*" />
                <On Platform="WinUI" Value="*, *, *" />
            </OnPlatform>
        </Setter.Value>
    </Setter>
</Style>

<!-- ✅ DO: Use consistent spacing -->
<Style x:Key="SectionSpacing" TargetType="StackLayout">
    <Setter Property="Spacing" Value="{StaticResource SpacingLarge}" />
</Style>
```

---

## 8. 🎯 Quiz & Assessment

### Quiz Questions

1. **Layout Selection**: เมื่อไหร่ควรใช้ FlexLayout แทน Grid?

2. **Performance**: วิธีไหนดีที่สุดในการเพิ่มประสิทธิภาพ layout ที่มี items เยอะ?

3. **Responsive Design**: เทคนิคไหนใช้สร้าง responsive columns ใน MAUI?

4. **CSS-like Approach**: จะสร้าง utility classes เหมือน Tailwind CSS ใน XAML ได้อย่างไร?

### Practical Assessment
สร้างแอป "Responsive News Reader" ที่มี:
- ✅ Adaptive layout สำหรับ mobile/tablet/desktop
- ✅ Card-based article layout
- ✅ Smooth scrolling performance
- ✅ CSS-like styling system

---

## 9. 🚀 Next Steps

### Advanced Topics to Explore
- **Custom Layout Engines**: สร้าง layout engine ของตัวเอง
- **Animation Integration**: รวม animations เข้ากับ layouts
- **Platform-Specific Layouts**: ใช้ platform renderers
- **Performance Profiling**: วัดและปรับแต่ง layout performance

### Chapter 10 Preview
หัวข้อต่อไป: **"Custom Controls & Templates"**
- สร้าง reusable custom controls
- Advanced templating techniques  
- Control styling และ theming
- Performance optimization

---

## 📚 Additional Resources

### 🔗 Useful Links
- [MAUI Layouts Documentation](https://docs.microsoft.com/dotnet/maui/user-interface/layouts/)
- [FlexLayout Guide](https://docs.microsoft.com/dotnet/maui/user-interface/layouts/flexlayout)
- [Responsive Design Patterns](https://docs.microsoft.com/dotnet/maui/user-interface/responsive-design)

### 📖 Recommended Reading
- "Mobile App UI/UX Design Patterns"
- "CSS Grid และ Flexbox สำหรับ Mobile"
- "Performance Optimization in Mobile Apps"

---

*หมายเหตุ: บทนี้เน้นการสร้าง responsive และ performant layouts ที่ทำงานได้ดีทุก platform โดยใช้หรือองค์ประกอบสมัยใหม่และ CSS-like approaches*
