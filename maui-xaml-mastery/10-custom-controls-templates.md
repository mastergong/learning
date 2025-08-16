# 🎨 Chapter 10: Custom Controls & Templates

> **หัวข้อ**: Building Reusable Custom Controls & Advanced Templates  
> **ระยะเวลา**: 3.5 ชั่วโมง  
> **ความยากง่าย**: ระดับกลาง-สูง  
> **Prerequisites**: บทที่ 1-9

## 🎯 Learning Objectives

หลังจากเรียนบทนี้แล้ว คุณจะสามารถ:

- ✅ สร้าง custom controls ที่ reusable และ maintainable
- ✅ ใช้ ControlTemplate และ DataTemplate อย่างมืออาชีพ
- ✅ สร้าง design system ด้วย custom controls
- ✅ เพิ่มประสิทธิภาพและ accessibility ให้ controls
- ✅ ทำ templating ระดับสูงด้วย CSS-like approaches

---

## 📱 Chapter Overview

### 🏗️ What We'll Build
สร้าง **"Custom UI Component Library"** ที่มี:
- 🎨 Reusable design system components
- 🔘 Custom button variations
- 📝 Advanced input controls
- 📊 Chart and data visualization controls
- 🎭 Animated UI components

---

## 1. 🔧 Custom Control Fundamentals

### 1.1 Basic Custom Control Structure

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\10-CustomControls\Controls\CustomButton.cs
using Microsoft.Maui.Controls;

namespace CustomControls.Controls;

public class CustomButton : ContentView
{
    #region Bindable Properties
    
    public static readonly BindableProperty TextProperty =
        BindableProperty.Create(nameof(Text), typeof(string), typeof(CustomButton), 
            string.Empty, propertyChanged: OnTextChanged);

    public static readonly BindableProperty IconProperty =
        BindableProperty.Create(nameof(Icon), typeof(string), typeof(CustomButton), 
            string.Empty);

    public static readonly BindableProperty ButtonTypeProperty =
        BindableProperty.Create(nameof(ButtonType), typeof(ButtonType), typeof(CustomButton), 
            ButtonType.Primary, propertyChanged: OnButtonTypeChanged);

    public static readonly BindableProperty CommandProperty =
        BindableProperty.Create(nameof(Command), typeof(ICommand), typeof(CustomButton));

    public static readonly BindableProperty CommandParameterProperty =
        BindableProperty.Create(nameof(CommandParameter), typeof(object), typeof(CustomButton));

    #endregion

    #region Properties

    public string Text
    {
        get => (string)GetValue(TextProperty);
        set => SetValue(TextProperty, value);
    }

    public string Icon
    {
        get => (string)GetValue(IconProperty);
        set => SetValue(IconProperty, value);
    }

    public ButtonType ButtonType
    {
        get => (ButtonType)GetValue(ButtonTypeProperty);
        set => SetValue(ButtonTypeProperty, value);
    }

    public ICommand Command
    {
        get => (ICommand)GetValue(CommandProperty);
        set => SetValue(CommandProperty, value);
    }

    public object CommandParameter
    {
        get => GetValue(CommandParameterProperty);
        set => SetValue(CommandParameterProperty, value);
    }

    #endregion

    private Frame _containerFrame;
    private Grid _contentGrid;
    private Label _iconLabel;
    private Label _textLabel;

    public CustomButton()
    {
        CreateControl();
        ApplyButtonType();
    }

    private void CreateControl()
    {
        _containerFrame = new Frame
        {
            Padding = new Thickness(16, 12),
            CornerRadius = 8,
            HasShadow = false,
            BorderColor = Colors.Transparent
        };

        _contentGrid = new Grid
        {
            ColumnDefinitions =
            {
                new ColumnDefinition { Width = GridLength.Auto },
                new ColumnDefinition { Width = GridLength.Star }
            },
            ColumnSpacing = 8
        };

        _iconLabel = new Label
        {
            FontFamily = "MaterialIcons",
            FontSize = 18,
            VerticalOptions = LayoutOptions.Center,
            IsVisible = false
        };

        _textLabel = new Label
        {
            FontSize = 16,
            FontAttributes = FontAttributes.Bold,
            VerticalOptions = LayoutOptions.Center,
            HorizontalOptions = LayoutOptions.Center
        };

        _contentGrid.Children.Add(_iconLabel);
        Grid.SetColumn(_iconLabel, 0);
        
        _contentGrid.Children.Add(_textLabel);
        Grid.SetColumn(_textLabel, 1);

        _containerFrame.Content = _contentGrid;
        Content = _containerFrame;

        // Add tap gesture
        var tapGesture = new TapGestureRecognizer();
        tapGesture.Tapped += OnTapped;
        GestureRecognizers.Add(tapGesture);
    }

    private static void OnTextChanged(BindableObject bindable, object oldValue, object newValue)
    {
        if (bindable is CustomButton button)
        {
            button._textLabel.Text = newValue?.ToString() ?? string.Empty;
        }
    }

    private static void OnButtonTypeChanged(BindableObject bindable, object oldValue, object newValue)
    {
        if (bindable is CustomButton button)
        {
            button.ApplyButtonType();
        }
    }

    private void ApplyButtonType()
    {
        switch (ButtonType)
        {
            case ButtonType.Primary:
                _containerFrame.BackgroundColor = Application.Current.RequestedTheme == AppTheme.Dark 
                    ? Color.FromArgb("#3B82F6") : Color.FromArgb("#2563EB");
                _textLabel.TextColor = Colors.White;
                _iconLabel.TextColor = Colors.White;
                break;

            case ButtonType.Secondary:
                _containerFrame.BackgroundColor = Application.Current.RequestedTheme == AppTheme.Dark 
                    ? Color.FromArgb("#374151") : Color.FromArgb("#6B7280");
                _textLabel.TextColor = Colors.White;
                _iconLabel.TextColor = Colors.White;
                break;

            case ButtonType.Outline:
                _containerFrame.BackgroundColor = Colors.Transparent;
                _containerFrame.BorderColor = Application.Current.RequestedTheme == AppTheme.Dark 
                    ? Color.FromArgb("#3B82F6") : Color.FromArgb("#2563EB");
                _textLabel.TextColor = Application.Current.RequestedTheme == AppTheme.Dark 
                    ? Color.FromArgb("#3B82F6") : Color.FromArgb("#2563EB");
                _iconLabel.TextColor = _textLabel.TextColor;
                break;

            case ButtonType.Ghost:
                _containerFrame.BackgroundColor = Colors.Transparent;
                _containerFrame.BorderColor = Colors.Transparent;
                _textLabel.TextColor = Application.Current.RequestedTheme == AppTheme.Dark 
                    ? Color.FromArgb("#3B82F6") : Color.FromArgb("#2563EB");
                _iconLabel.TextColor = _textLabel.TextColor;
                break;
        }
    }

    private void OnTapped(object sender, EventArgs e)
    {
        // Add visual feedback
        this.ScaleTo(0.95, 50).ContinueWith(t => 
        {
            Device.BeginInvokeOnMainThread(() =>
            {
                this.ScaleTo(1.0, 50);
            });
        });

        if (Command?.CanExecute(CommandParameter) == true)
        {
            Command.Execute(CommandParameter);
        }
    }
}

public enum ButtonType
{
    Primary,
    Secondary,
    Outline,
    Ghost
}
```

### 1.2 Custom Control Template

```xml
<!-- d:\A-Work\learn from ai\maui-xaml-mastery\src\10-CustomControls\Styles\CustomButtonTemplate.xaml -->
<ResourceDictionary xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
                   xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
                   xmlns:controls="clr-namespace:CustomControls.Controls">

    <!-- CSS-like Button Utilities -->
    <Style x:Key="BaseButton" TargetType="controls:CustomButton">
        <Setter Property="HeightRequest" Value="48" />
        <Setter Property="HorizontalOptions" Value="FillAndExpand" />
        <Setter Property="VerticalOptions" Value="Center" />
    </Style>

    <!-- Primary Button - like .btn-primary -->
    <Style x:Key="PrimaryButton" TargetType="controls:CustomButton" BasedOn="{StaticResource BaseButton}">
        <Setter Property="ButtonType" Value="Primary" />
    </Style>

    <!-- Secondary Button - like .btn-secondary -->
    <Style x:Key="SecondaryButton" TargetType="controls:CustomButton" BasedOn="{StaticResource BaseButton}">
        <Setter Property="ButtonType" Value="Secondary" />
    </Style>

    <!-- Outline Button - like .btn-outline -->
    <Style x:Key="OutlineButton" TargetType="controls:CustomButton" BasedOn="{StaticResource BaseButton}">
        <Setter Property="ButtonType" Value="Outline" />
    </Style>

    <!-- Ghost Button - like .btn-ghost -->
    <Style x:Key="GhostButton" TargetType="controls:CustomButton" BasedOn="{StaticResource BaseButton}">
        <Setter Property="ButtonType" Value="Ghost" />
    </Style>

    <!-- Size Variations - like .btn-sm, .btn-lg -->
    <Style x:Key="SmallButton" TargetType="controls:CustomButton">
        <Setter Property="HeightRequest" Value="32" />
        <Setter Property="HorizontalOptions" Value="Center" />
    </Style>

    <Style x:Key="LargeButton" TargetType="controls:CustomButton">
        <Setter Property="HeightRequest" Value="56" />
        <Setter Property="HorizontalOptions" Value="FillAndExpand" />
    </Style>

</ResourceDictionary>
```

---

## 2. 📝 Advanced Input Controls

### 2.1 Custom Input Field with Validation

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\10-CustomControls\Controls\CustomEntry.cs
using System.ComponentModel;

namespace CustomControls.Controls;

public class CustomEntry : ContentView
{
    #region Bindable Properties

    public static readonly BindableProperty LabelTextProperty =
        BindableProperty.Create(nameof(LabelText), typeof(string), typeof(CustomEntry));

    public static readonly BindableProperty PlaceholderProperty =
        BindableProperty.Create(nameof(Placeholder), typeof(string), typeof(CustomEntry));

    public static readonly BindableProperty TextProperty =
        BindableProperty.Create(nameof(Text), typeof(string), typeof(CustomEntry), 
            defaultBindingMode: BindingMode.TwoWay);

    public static readonly BindableProperty IsPasswordProperty =
        BindableProperty.Create(nameof(IsPassword), typeof(bool), typeof(CustomEntry));

    public static readonly BindableProperty ValidationRulesProperty =
        BindableProperty.Create(nameof(ValidationRules), typeof(List<IValidationRule>), typeof(CustomEntry));

    public static readonly BindableProperty ErrorMessageProperty =
        BindableProperty.Create(nameof(ErrorMessage), typeof(string), typeof(CustomEntry));

    public static readonly BindableProperty IsValidProperty =
        BindableProperty.Create(nameof(IsValid), typeof(bool), typeof(CustomEntry), true);

    #endregion

    #region Properties

    public string LabelText
    {
        get => (string)GetValue(LabelTextProperty);
        set => SetValue(LabelTextProperty, value);
    }

    public string Placeholder
    {
        get => (string)GetValue(PlaceholderProperty);
        set => SetValue(PlaceholderProperty, value);
    }

    public string Text
    {
        get => (string)GetValue(TextProperty);
        set => SetValue(TextProperty, value);
    }

    public bool IsPassword
    {
        get => (bool)GetValue(IsPasswordProperty);
        set => SetValue(IsPasswordProperty, value);
    }

    public List<IValidationRule> ValidationRules
    {
        get => (List<IValidationRule>)GetValue(ValidationRulesProperty);
        set => SetValue(ValidationRulesProperty, value);
    }

    public string ErrorMessage
    {
        get => (string)GetValue(ErrorMessageProperty);
        set => SetValue(ErrorMessageProperty, value);
    }

    public bool IsValid
    {
        get => (bool)GetValue(IsValidProperty);
        set => SetValue(IsValidProperty, value);
    }

    #endregion

    private StackLayout _mainContainer;
    private Label _labelLabel;
    private Frame _entryFrame;
    private Entry _entry;
    private Label _errorLabel;

    public CustomEntry()
    {
        ValidationRules = new List<IValidationRule>();
        CreateControl();
        BindEvents();
    }

    private void CreateControl()
    {
        _mainContainer = new StackLayout
        {
            Spacing = 8
        };

        // Label
        _labelLabel = new Label
        {
            FontSize = 14,
            FontAttributes = FontAttributes.Bold,
            TextColor = Application.Current.RequestedTheme == AppTheme.Dark 
                ? Color.FromArgb("#E5E7EB") : Color.FromArgb("#374151")
        };

        // Entry container with border
        _entryFrame = new Frame
        {
            Padding = new Thickness(12, 8),
            CornerRadius = 8,
            BorderColor = Application.Current.RequestedTheme == AppTheme.Dark 
                ? Color.FromArgb("#4B5563") : Color.FromArgb("#D1D5DB"),
            BackgroundColor = Application.Current.RequestedTheme == AppTheme.Dark 
                ? Color.FromArgb("#111827") : Colors.White,
            HasShadow = false
        };

        _entry = new Entry
        {
            BackgroundColor = Colors.Transparent,
            FontSize = 16
        };

        // Error message
        _errorLabel = new Label
        {
            FontSize = 12,
            TextColor = Color.FromArgb("#EF4444"),
            IsVisible = false
        };

        _entryFrame.Content = _entry;

        _mainContainer.Children.Add(_labelLabel);
        _mainContainer.Children.Add(_entryFrame);
        _mainContainer.Children.Add(_errorLabel);

        Content = _mainContainer;
    }

    private void BindEvents()
    {
        _entry.TextChanged += OnTextChanged;
        _entry.Unfocused += OnUnfocused;

        PropertyChanged += OnPropertyChanged;
    }

    private void OnPropertyChanged(object sender, PropertyChangedEventArgs e)
    {
        switch (e.PropertyName)
        {
            case nameof(LabelText):
                _labelLabel.Text = LabelText;
                _labelLabel.IsVisible = !string.IsNullOrEmpty(LabelText);
                break;
            case nameof(Placeholder):
                _entry.Placeholder = Placeholder;
                break;
            case nameof(Text):
                if (_entry.Text != Text)
                    _entry.Text = Text;
                break;
            case nameof(IsPassword):
                _entry.IsPassword = IsPassword;
                break;
            case nameof(ErrorMessage):
                _errorLabel.Text = ErrorMessage;
                _errorLabel.IsVisible = !string.IsNullOrEmpty(ErrorMessage);
                break;
            case nameof(IsValid):
                UpdateValidationState();
                break;
        }
    }

    private void OnTextChanged(object sender, TextChangedEventArgs e)
    {
        Text = e.NewTextValue;
        ValidateText();
    }

    private void OnUnfocused(object sender, FocusEventArgs e)
    {
        ValidateText();
    }

    private void ValidateText()
    {
        if (ValidationRules == null || !ValidationRules.Any())
        {
            IsValid = true;
            ErrorMessage = string.Empty;
            return;
        }

        foreach (var rule in ValidationRules)
        {
            if (!rule.Check(Text))
            {
                IsValid = false;
                ErrorMessage = rule.ValidationMessage;
                return;
            }
        }

        IsValid = true;
        ErrorMessage = string.Empty;
    }

    private void UpdateValidationState()
    {
        if (IsValid)
        {
            _entryFrame.BorderColor = Application.Current.RequestedTheme == AppTheme.Dark 
                ? Color.FromArgb("#4B5563") : Color.FromArgb("#D1D5DB");
        }
        else
        {
            _entryFrame.BorderColor = Color.FromArgb("#EF4444");
        }
    }
}

// Validation Rule Interface
public interface IValidationRule
{
    string ValidationMessage { get; set; }
    bool Check(string value);
}

// Example Validation Rules
public class RequiredRule : IValidationRule
{
    public string ValidationMessage { get; set; } = "This field is required";

    public bool Check(string value)
    {
        return !string.IsNullOrWhiteSpace(value);
    }
}

public class EmailRule : IValidationRule
{
    public string ValidationMessage { get; set; } = "Please enter a valid email address";

    public bool Check(string value)
    {
        if (string.IsNullOrWhiteSpace(value))
            return true; // Let required rule handle empty values

        return System.Text.RegularExpressions.Regex.IsMatch(value,
            @"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$");
    }
}
```

### 2.2 Advanced Select Control

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\10-CustomControls\Controls\CustomPicker.cs
namespace CustomControls.Controls;

public class CustomPicker : ContentView
{
    #region Bindable Properties

    public static readonly BindableProperty ItemsSourceProperty =
        BindableProperty.Create(nameof(ItemsSource), typeof(IList), typeof(CustomPicker));

    public static readonly BindableProperty SelectedItemProperty =
        BindableProperty.Create(nameof(SelectedItem), typeof(object), typeof(CustomPicker),
            defaultBindingMode: BindingMode.TwoWay);

    public static readonly BindableProperty DisplayMemberPathProperty =
        BindableProperty.Create(nameof(DisplayMemberPath), typeof(string), typeof(CustomPicker));

    public static readonly BindableProperty PlaceholderProperty =
        BindableProperty.Create(nameof(Placeholder), typeof(string), typeof(CustomPicker));

    #endregion

    #region Properties

    public IList ItemsSource
    {
        get => (IList)GetValue(ItemsSourceProperty);
        set => SetValue(ItemsSourceProperty, value);
    }

    public object SelectedItem
    {
        get => GetValue(SelectedItemProperty);
        set => SetValue(SelectedItemProperty, value);
    }

    public string DisplayMemberPath
    {
        get => (string)GetValue(DisplayMemberPathProperty);
        set => SetValue(DisplayMemberPathProperty, value);
    }

    public string Placeholder
    {
        get => (string)GetValue(PlaceholderProperty);
        set => SetValue(PlaceholderProperty, value);
    }

    #endregion

    private Frame _containerFrame;
    private Grid _contentGrid;
    private Label _displayLabel;
    private Label _iconLabel;
    private CollectionView _dropdownCollectionView;
    private bool _isDropdownOpen = false;

    public CustomPicker()
    {
        CreateControl();
        BindEvents();
    }

    private void CreateControl()
    {
        var mainContainer = new StackLayout();

        // Main picker button
        _containerFrame = new Frame
        {
            Padding = new Thickness(12, 16),
            CornerRadius = 8,
            BorderColor = Application.Current.RequestedTheme == AppTheme.Dark 
                ? Color.FromArgb("#4B5563") : Color.FromArgb("#D1D5DB"),
            BackgroundColor = Application.Current.RequestedTheme == AppTheme.Dark 
                ? Color.FromArgb("#111827") : Colors.White,
            HasShadow = false
        };

        _contentGrid = new Grid
        {
            ColumnDefinitions =
            {
                new ColumnDefinition { Width = GridLength.Star },
                new ColumnDefinition { Width = GridLength.Auto }
            }
        };

        _displayLabel = new Label
        {
            FontSize = 16,
            VerticalOptions = LayoutOptions.Center,
            TextColor = Application.Current.RequestedTheme == AppTheme.Dark 
                ? Color.FromArgb("#9CA3AF") : Color.FromArgb("#6B7280")
        };

        _iconLabel = new Label
        {
            Text = "▼",
            FontSize = 12,
            VerticalOptions = LayoutOptions.Center,
            HorizontalOptions = LayoutOptions.End,
            TextColor = Application.Current.RequestedTheme == AppTheme.Dark 
                ? Color.FromArgb("#9CA3AF") : Color.FromArgb("#6B7280")
        };

        _contentGrid.Children.Add(_displayLabel);
        Grid.SetColumn(_displayLabel, 0);

        _contentGrid.Children.Add(_iconLabel);
        Grid.SetColumn(_iconLabel, 1);

        _containerFrame.Content = _contentGrid;

        // Dropdown list
        _dropdownCollectionView = new CollectionView
        {
            IsVisible = false,
            BackgroundColor = Application.Current.RequestedTheme == AppTheme.Dark 
                ? Color.FromArgb("#111827") : Colors.White,
            SelectionMode = SelectionMode.Single
        };

        _dropdownCollectionView.ItemTemplate = new DataTemplate(() =>
        {
            var label = new Label
            {
                FontSize = 16,
                Padding = new Thickness(12, 16),
                VerticalOptions = LayoutOptions.Center
            };

            label.SetBinding(Label.TextProperty, new Binding("."));
            
            return new ViewCell { View = label };
        });

        mainContainer.Children.Add(_containerFrame);
        mainContainer.Children.Add(_dropdownCollectionView);

        Content = mainContainer;

        // Add tap gesture
        var tapGesture = new TapGestureRecognizer();
        tapGesture.Tapped += OnTapped;
        _containerFrame.GestureRecognizers.Add(tapGesture);
    }

    private void BindEvents()
    {
        PropertyChanged += OnPropertyChanged;
        _dropdownCollectionView.SelectionChanged += OnSelectionChanged;
    }

    private void OnPropertyChanged(object sender, PropertyChangedEventArgs e)
    {
        switch (e.PropertyName)
        {
            case nameof(ItemsSource):
                _dropdownCollectionView.ItemsSource = ItemsSource;
                break;
            case nameof(SelectedItem):
                UpdateDisplayText();
                break;
            case nameof(Placeholder):
                if (SelectedItem == null)
                    _displayLabel.Text = Placeholder;
                break;
        }
    }

    private void OnTapped(object sender, EventArgs e)
    {
        ToggleDropdown();
    }

    private void OnSelectionChanged(object sender, SelectionChangedEventArgs e)
    {
        if (e.CurrentSelection.FirstOrDefault() is object selectedItem)
        {
            SelectedItem = selectedItem;
            CloseDropdown();
        }
    }

    private void ToggleDropdown()
    {
        if (_isDropdownOpen)
            CloseDropdown();
        else
            OpenDropdown();
    }

    private void OpenDropdown()
    {
        _dropdownCollectionView.IsVisible = true;
        _isDropdownOpen = true;
        _iconLabel.Text = "▲";
    }

    private void CloseDropdown()
    {
        _dropdownCollectionView.IsVisible = false;
        _isDropdownOpen = false;
        _iconLabel.Text = "▼";
    }

    private void UpdateDisplayText()
    {
        if (SelectedItem == null)
        {
            _displayLabel.Text = Placeholder ?? "Select an option";
            _displayLabel.TextColor = Application.Current.RequestedTheme == AppTheme.Dark 
                ? Color.FromArgb("#9CA3AF") : Color.FromArgb("#6B7280");
        }
        else
        {
            string displayText = SelectedItem.ToString();
            
            if (!string.IsNullOrEmpty(DisplayMemberPath))
            {
                var property = SelectedItem.GetType().GetProperty(DisplayMemberPath);
                displayText = property?.GetValue(SelectedItem)?.ToString() ?? displayText;
            }

            _displayLabel.Text = displayText;
            _displayLabel.TextColor = Application.Current.RequestedTheme == AppTheme.Dark 
                ? Color.FromArgb("#E5E7EB") : Color.FromArgb("#111827");
        }
    }
}
```

---

## 3. 📊 Data Visualization Controls

### 3.1 Custom Chart Control

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\10-CustomControls\Controls\SimpleChart.cs
using Microsoft.Maui.Graphics;

namespace CustomControls.Controls;

public class SimpleChart : GraphicsView
{
    #region Bindable Properties

    public static readonly BindableProperty DataPointsProperty =
        BindableProperty.Create(nameof(DataPoints), typeof(List<ChartDataPoint>), typeof(SimpleChart),
            propertyChanged: OnDataPointsChanged);

    public static readonly BindableProperty ChartTypeProperty =
        BindableProperty.Create(nameof(ChartType), typeof(ChartType), typeof(SimpleChart), 
            ChartType.Line, propertyChanged: OnChartTypeChanged);

    public static readonly BindableProperty PrimaryColorProperty =
        BindableProperty.Create(nameof(PrimaryColor), typeof(Color), typeof(SimpleChart), 
            Colors.Blue, propertyChanged: OnStyleChanged);

    #endregion

    #region Properties

    public List<ChartDataPoint> DataPoints
    {
        get => (List<ChartDataPoint>)GetValue(DataPointsProperty);
        set => SetValue(DataPointsProperty, value);
    }

    public ChartType ChartType
    {
        get => (ChartType)GetValue(ChartTypeProperty);
        set => SetValue(ChartTypeProperty, value);
    }

    public Color PrimaryColor
    {
        get => (Color)GetValue(PrimaryColorProperty);
        set => SetValue(PrimaryColorProperty, value);
    }

    #endregion

    public SimpleChart()
    {
        Drawable = new ChartDrawable(this);
        DataPoints = new List<ChartDataPoint>();
    }

    private static void OnDataPointsChanged(BindableObject bindable, object oldValue, object newValue)
    {
        if (bindable is SimpleChart chart)
        {
            chart.Invalidate();
        }
    }

    private static void OnChartTypeChanged(BindableObject bindable, object oldValue, object newValue)
    {
        if (bindable is SimpleChart chart)
        {
            chart.Invalidate();
        }
    }

    private static void OnStyleChanged(BindableObject bindable, object oldValue, object newValue)
    {
        if (bindable is SimpleChart chart)
        {
            chart.Invalidate();
        }
    }
}

public class ChartDrawable : IDrawable
{
    private readonly SimpleChart _chart;

    public ChartDrawable(SimpleChart chart)
    {
        _chart = chart;
    }

    public void Draw(ICanvas canvas, RectF dirtyRect)
    {
        if (_chart.DataPoints == null || !_chart.DataPoints.Any())
            return;

        canvas.FillColor = Colors.White;
        canvas.FillRectangle(dirtyRect);

        var padding = 40f;
        var chartRect = new RectF(
            dirtyRect.X + padding,
            dirtyRect.Y + padding,
            dirtyRect.Width - (padding * 2),
            dirtyRect.Height - (padding * 2)
        );

        DrawAxes(canvas, chartRect);
        
        switch (_chart.ChartType)
        {
            case ChartType.Line:
                DrawLineChart(canvas, chartRect);
                break;
            case ChartType.Bar:
                DrawBarChart(canvas, chartRect);
                break;
        }

        DrawLabels(canvas, chartRect, dirtyRect);
    }

    private void DrawAxes(ICanvas canvas, RectF chartRect)
    {
        canvas.StrokeColor = Colors.Gray;
        canvas.StrokeSize = 1;

        // Y-axis
        canvas.DrawLine(chartRect.X, chartRect.Y, chartRect.X, chartRect.Bottom);
        
        // X-axis
        canvas.DrawLine(chartRect.X, chartRect.Bottom, chartRect.Right, chartRect.Bottom);
    }

    private void DrawLineChart(ICanvas canvas, RectF chartRect)
    {
        if (_chart.DataPoints.Count < 2) return;

        var points = GetScaledPoints(chartRect);
        
        canvas.StrokeColor = _chart.PrimaryColor;
        canvas.StrokeSize = 3;

        var path = new PathF();
        path.MoveTo(points[0].X, points[0].Y);

        for (int i = 1; i < points.Count; i++)
        {
            path.LineTo(points[i].X, points[i].Y);
        }

        canvas.DrawPath(path);

        // Draw data points
        canvas.FillColor = _chart.PrimaryColor;
        foreach (var point in points)
        {
            canvas.FillCircle(point.X, point.Y, 4);
        }
    }

    private void DrawBarChart(ICanvas canvas, RectF chartRect)
    {
        var maxValue = _chart.DataPoints.Max(p => p.Value);
        var barWidth = chartRect.Width / _chart.DataPoints.Count * 0.8f;
        var barSpacing = chartRect.Width / _chart.DataPoints.Count * 0.2f;

        canvas.FillColor = _chart.PrimaryColor;

        for (int i = 0; i < _chart.DataPoints.Count; i++)
        {
            var dataPoint = _chart.DataPoints[i];
            var barHeight = (float)(dataPoint.Value / maxValue * chartRect.Height);
            
            var barX = chartRect.X + (i * (barWidth + barSpacing)) + (barSpacing / 2);
            var barY = chartRect.Bottom - barHeight;

            canvas.FillRectangle(barX, barY, barWidth, barHeight);
        }
    }

    private List<PointF> GetScaledPoints(RectF chartRect)
    {
        var points = new List<PointF>();
        var maxValue = _chart.DataPoints.Max(p => p.Value);
        var minValue = _chart.DataPoints.Min(p => p.Value);
        var valueRange = maxValue - minValue;

        for (int i = 0; i < _chart.DataPoints.Count; i++)
        {
            var dataPoint = _chart.DataPoints[i];
            
            var x = chartRect.X + (i / (float)(_chart.DataPoints.Count - 1)) * chartRect.Width;
            var y = chartRect.Bottom - ((dataPoint.Value - minValue) / valueRange * chartRect.Height);
            
            points.Add(new PointF(x, y));
        }

        return points;
    }

    private void DrawLabels(ICanvas canvas, RectF chartRect, RectF dirtyRect)
    {
        canvas.FontColor = Colors.Black;
        canvas.FontSize = 12;

        // Y-axis labels
        var maxValue = _chart.DataPoints.Max(p => p.Value);
        var steps = 5;
        
        for (int i = 0; i <= steps; i++)
        {
            var value = (maxValue / steps) * i;
            var y = chartRect.Bottom - (i / (float)steps) * chartRect.Height;
            
            canvas.DrawString(
                value.ToString("F0"),
                chartRect.X - 30,
                y - 6,
                HorizontalAlignment.Right
            );
        }

        // X-axis labels (if labels are provided)
        for (int i = 0; i < _chart.DataPoints.Count; i++)
        {
            var dataPoint = _chart.DataPoints[i];
            if (!string.IsNullOrEmpty(dataPoint.Label))
            {
                var x = chartRect.X + (i / (float)(_chart.DataPoints.Count - 1)) * chartRect.Width;
                
                canvas.DrawString(
                    dataPoint.Label,
                    x,
                    chartRect.Bottom + 15,
                    HorizontalAlignment.Center
                );
            }
        }
    }
}

public class ChartDataPoint
{
    public double Value { get; set; }
    public string Label { get; set; }

    public ChartDataPoint(double value, string label = "")
    {
        Value = value;
        Label = label;
    }
}

public enum ChartType
{
    Line,
    Bar
}
```

---

## 4. 🎭 Advanced Templating

### 4.1 Dynamic DataTemplate Selector

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\10-CustomControls\Templates\DynamicDataTemplateSelector.cs
namespace CustomControls.Templates;

public class DynamicDataTemplateSelector : DataTemplateSelector
{
    public DataTemplate CardTemplate { get; set; }
    public DataTemplate ListTemplate { get; set; }
    public DataTemplate GridTemplate { get; set; }

    protected override DataTemplate OnSelectTemplate(object item, BindableObject container)
    {
        if (item is TemplateItem templateItem)
        {
            return templateItem.ViewType switch
            {
                ViewType.Card => CardTemplate,
                ViewType.List => ListTemplate,
                ViewType.Grid => GridTemplate,
                _ => CardTemplate
            };
        }

        return CardTemplate;
    }
}

public class TemplateItem
{
    public string Title { get; set; }
    public string Description { get; set; }
    public string ImageUrl { get; set; }
    public ViewType ViewType { get; set; }
    public Dictionary<string, object> Properties { get; set; } = new();
}

public enum ViewType
{
    Card,
    List,
    Grid
}
```

### 4.2 Adaptive Control Template

```xml
<!-- d:\A-Work\learn from ai\maui-xaml-mastery\src\10-CustomControls\Templates\AdaptiveTemplates.xaml -->
<ResourceDictionary xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
                   xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
                   xmlns:templates="clr-namespace:CustomControls.Templates">

    <!-- Card Template for larger screens -->
    <DataTemplate x:Key="CardTemplate">
        <Frame Padding="16" 
               Margin="8" 
               CornerRadius="12"
               BackgroundColor="{AppThemeBinding Light={StaticResource White}, Dark={StaticResource Gray900}}"
               BorderColor="{AppThemeBinding Light={StaticResource Gray200}, Dark={StaticResource Gray700}}">
            
            <Grid RowDefinitions="Auto, *, Auto" RowSpacing="12">
                
                <!-- Header -->
                <StackLayout Grid.Row="0" Orientation="Horizontal">
                    <Image Source="{Binding ImageUrl}" 
                           WidthRequest="40" 
                           HeightRequest="40"
                           Aspect="AspectFill">
                        <Image.Clip>
                            <EllipseGeometry Center="20,20" RadiusX="20" RadiusY="20" />
                        </Image.Clip>
                    </Image>
                    <StackLayout Margin="12,0,0,0" VerticalOptions="Center">
                        <Label Text="{Binding Title}" 
                               FontSize="16" 
                               FontAttributes="Bold" />
                        <Label Text="{Binding Description}" 
                               FontSize="12" 
                               TextColor="{StaticResource Gray600}" />
                    </StackLayout>
                </StackLayout>

                <!-- Content -->
                <StackLayout Grid.Row="1">
                    <Label Text="Card content area" 
                           FontSize="14" 
                           TextColor="{StaticResource Gray700}" />
                </StackLayout>

                <!-- Actions -->
                <StackLayout Grid.Row="2" Orientation="Horizontal" HorizontalOptions="End">
                    <Button Text="Action 1" Style="{StaticResource GhostButton}" />
                    <Button Text="Action 2" Style="{StaticResource OutlineButton}" />
                </StackLayout>

            </Grid>
        </Frame>
    </DataTemplate>

    <!-- List Template for compact display -->
    <DataTemplate x:Key="ListTemplate">
        <Grid Padding="16,8" 
              ColumnDefinitions="Auto, *, Auto" 
              ColumnSpacing="12">
            
            <Image Grid.Column="0"
                   Source="{Binding ImageUrl}" 
                   WidthRequest="32" 
                   HeightRequest="32"
                   Aspect="AspectFill"
                   VerticalOptions="Center">
                <Image.Clip>
                    <EllipseGeometry Center="16,16" RadiusX="16" RadiusY="16" />
                </Image.Clip>
            </Image>

            <StackLayout Grid.Column="1" VerticalOptions="Center">
                <Label Text="{Binding Title}" 
                       FontSize="14" 
                       FontAttributes="Bold" />
                <Label Text="{Binding Description}" 
                       FontSize="12" 
                       TextColor="{StaticResource Gray600}" />
            </StackLayout>

            <Label Grid.Column="2"
                   Text="›" 
                   FontSize="16" 
                   TextColor="{StaticResource Gray400}"
                   VerticalOptions="Center" />

        </Grid>
    </DataTemplate>

    <!-- Grid Template for grid layouts -->
    <DataTemplate x:Key="GridTemplate">
        <Frame Padding="12" 
               Margin="4" 
               CornerRadius="8"
               BackgroundColor="{AppThemeBinding Light={StaticResource White}, Dark={StaticResource Gray900}}">
            
            <StackLayout>
                <Image Source="{Binding ImageUrl}" 
                       HeightRequest="80"
                       Aspect="AspectFill" />
                
                <Label Text="{Binding Title}" 
                       FontSize="12" 
                       FontAttributes="Bold"
                       HorizontalOptions="Center"
                       Margin="0,8,0,0" />
            </StackLayout>

        </Frame>
    </DataTemplate>

    <!-- Dynamic Template Selector -->
    <templates:DynamicDataTemplateSelector x:Key="DynamicTemplateSelector"
                                         CardTemplate="{StaticResource CardTemplate}"
                                         ListTemplate="{StaticResource ListTemplate}"
                                         GridTemplate="{StaticResource GridTemplate}" />

</ResourceDictionary>
```

---

## 5. 🎨 CSS-like Component System

### 5.1 Design System Components

```xml
<!-- d:\A-Work\learn from ai\maui-xaml-mastery\src\10-CustomControls\Styles\DesignSystem.xaml -->
<ResourceDictionary xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
                   xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">

    <!-- CSS-like Utility Classes -->
    
    <!-- Spacing Utilities (like Tailwind's p-*, m-*) -->
    <Style x:Key="p-2" TargetType="Layout">
        <Setter Property="Padding" Value="8" />
    </Style>

    <Style x:Key="p-4" TargetType="Layout">
        <Setter Property="Padding" Value="16" />
    </Style>

    <Style x:Key="p-6" TargetType="Layout">
        <Setter Property="Padding" Value="24" />
    </Style>

    <Style x:Key="m-2" TargetType="View">
        <Setter Property="Margin" Value="8" />
    </Style>

    <Style x:Key="m-4" TargetType="View">
        <Setter Property="Margin" Value="16" />
    </Style>

    <!-- Text Utilities -->
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

    <Style x:Key="font-bold" TargetType="Label">
        <Setter Property="FontAttributes" Value="Bold" />
    </Style>

    <!-- Color Utilities -->
    <Style x:Key="text-gray-500" TargetType="Label">
        <Setter Property="TextColor" Value="{StaticResource Gray500}" />
    </Style>

    <Style x:Key="text-gray-700" TargetType="Label">
        <Setter Property="TextColor" Value="{StaticResource Gray700}" />
    </Style>

    <Style x:Key="text-primary" TargetType="Label">
        <Setter Property="TextColor" Value="{StaticResource Primary}" />
    </Style>

    <!-- Background Utilities -->
    <Style x:Key="bg-white" TargetType="VisualElement">
        <Setter Property="BackgroundColor" Value="{StaticResource White}" />
    </Style>

    <Style x:Key="bg-gray-50" TargetType="VisualElement">
        <Setter Property="BackgroundColor" Value="{StaticResource Gray50}" />
    </Style>

    <Style x:Key="bg-primary" TargetType="VisualElement">
        <Setter Property="BackgroundColor" Value="{StaticResource Primary}" />
    </Style>

    <!-- Border Utilities -->
    <Style x:Key="border" TargetType="Frame">
        <Setter Property="BorderColor" Value="{StaticResource Gray200}" />
        <Setter Property="HasShadow" Value="False" />
    </Style>

    <Style x:Key="border-gray-300" TargetType="Frame">
        <Setter Property="BorderColor" Value="{StaticResource Gray300}" />
    </Style>

    <Style x:Key="rounded" TargetType="Frame">
        <Setter Property="CornerRadius" Value="4" />
    </Style>

    <Style x:Key="rounded-lg" TargetType="Frame">
        <Setter Property="CornerRadius" Value="8" />
    </Style>

    <Style x:Key="rounded-xl" TargetType="Frame">
        <Setter Property="CornerRadius" Value="12" />
    </Style>

    <!-- Shadow Utilities -->
    <Style x:Key="shadow" TargetType="Frame">
        <Setter Property="HasShadow" Value="True" />
    </Style>

    <Style x:Key="shadow-none" TargetType="Frame">
        <Setter Property="HasShadow" Value="False" />
    </Style>

    <!-- Component Combinations (like CSS components) -->
    <Style x:Key="card" TargetType="Frame" BasedOn="{StaticResource bg-white}">
        <Setter Property="Padding" Value="16" />
        <Setter Property="CornerRadius" Value="8" />
        <Setter Property="BorderColor" Value="{StaticResource Gray200}" />
        <Setter Property="HasShadow" Value="True" />
    </Style>

    <Style x:Key="alert" TargetType="Frame">
        <Setter Property="Padding" Value="12" />
        <Setter Property="CornerRadius" Value="6" />
        <Setter Property="BorderColor" Value="{StaticResource Primary}" />
        <Setter Property="BackgroundColor" Value="{StaticResource PrimaryLight}" />
    </Style>

</ResourceDictionary>
```

---

## 6. 🧪 Practical Exercises

### Exercise 1: Custom Rating Control
สร้าง star rating control ที่มี:
- Interactive star selection
- Half-star support
- Custom styling options
- Accessibility support

### Exercise 2: Advanced Form Builder
สร้างระบบ form ที่มี:
- Dynamic field generation
- Validation integration
- CSS-like styling
- Responsive layout

### Exercise 3: Data Visualization Suite
สร้าง chart library ที่มี:
- Multiple chart types
- Interactive features
- Animation support
- Theme integration

---

## 7. 📝 Best Practices & Tips

### 🎯 Custom Control Best Practices

```csharp
// ✅ DO: Use proper bindable properties
public static readonly BindableProperty TextProperty =
    BindableProperty.Create(nameof(Text), typeof(string), typeof(CustomControl),
        defaultValue: string.Empty,
        defaultBindingMode: BindingMode.TwoWay,
        propertyChanged: OnTextChanged);

// ✅ DO: Handle property changes properly
private static void OnTextChanged(BindableObject bindable, object oldValue, object newValue)
{
    if (bindable is CustomControl control)
    {
        control.UpdateTextDisplay();
    }
}

// ✅ DO: Provide good defaults
public CustomControl()
{
    SetupDefaultStyles();
    InitializeComponents();
}

// ✅ DO: Support theming
private void ApplyTheme()
{
    var isDark = Application.Current.RequestedTheme == AppTheme.Dark;
    BackgroundColor = isDark ? DarkBackground : LightBackground;
}

// ❌ DON'T: Hardcode values
// Wrong:
BackgroundColor = Colors.Blue;

// Right:
BackgroundColor = Application.Current.Resources["PrimaryColor"] as Color;
```

### 🔧 Performance Optimization

```csharp
// ✅ DO: Use efficient layouts
public class OptimizedCustomControl : ContentView
{
    private readonly Dictionary<string, object> _cache = new();
    
    protected override Size MeasureOverride(double widthConstraint, double heightConstraint)
    {
        var key = $"{widthConstraint}-{heightConstraint}";
        
        if (_cache.TryGetValue(key, out var cachedSize))
            return (Size)cachedSize;
            
        var size = base.MeasureOverride(widthConstraint, heightConstraint);
        _cache[key] = size;
        
        return size;
    }
}

// ✅ DO: Minimize property change events
private bool _isUpdating = false;

private void UpdateMultipleProperties()
{
    _isUpdating = true;
    
    Property1 = value1;
    Property2 = value2;
    Property3 = value3;
    
    _isUpdating = false;
    RefreshDisplay();
}
```

---

## 8. 🎯 Quiz & Assessment

### Quiz Questions

1. **Control Architecture**: เมื่อไหร่ควรใช้ ContentView vs สร้าง control จาก scratch?

2. **Templating**: ความแตกต่างระหว่าง ControlTemplate และ DataTemplate คืออะไร?

3. **Performance**: วิธีไหนดีที่สุดในการเพิ่มประสิทธิภาพ custom control?

4. **CSS-like Design**: จะออกแบบ utility class system เหมือน Tailwind ใน XAML ได้อย่างไร?

### Practical Assessment
สร้าง **"Component Library Dashboard"** ที่มี:
- ✅ Custom input controls พร้อม validation
- ✅ Advanced chart components
- ✅ Responsive layout system
- ✅ CSS-like utility classes
- ✅ Theme switching support

---

## 9. 🚀 Next Steps

### Advanced Topics to Explore
- **Platform-Specific Controls**: ใช้ handlers สำหรับ native features
- **Performance Profiling**: วัดและปรับแต่ง control performance
- **Accessibility Features**: เพิ่ม screen reader support
- **Animation Integration**: สร้าง animated custom controls

### Chapter 11 Preview
หัวข้อต่อไป: **"Animations & Visual Effects"**
- Advanced animation techniques
- Custom transitions และ effects
- Performance-optimized animations
- Interactive visual feedback

---

## 📚 Additional Resources

### 🔗 Useful Links
- [Custom Controls Guide](https://docs.microsoft.com/dotnet/maui/user-interface/controls/custom)
- [Control Templates](https://docs.microsoft.com/dotnet/maui/fundamentals/control-templates)
- [Data Templates](https://docs.microsoft.com/dotnet/maui/fundamentals/datatemplate)

### 📖 Recommended Reading
- "Building Custom Controls in .NET MAUI"
- "Advanced XAML Templating Techniques"
- "Mobile UI Component Design Patterns"

---

*หมายเหตุ: บทนี้เน้นการสร้าง custom controls ที่ reusable, maintainable และมี performance ดี พร้อมทั้งใช้ CSS-like approaches สำหรับการ styling*
