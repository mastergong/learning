# 🌐 Chapter 19: Accessibility & Localization

- ทำความเข้าใจและใช้งาน Accessibility APIs ใน .NET MAUI
- สร้าง UI ที่เข้าถึงได้สำหรับผู้ใช้ที่มีความต้องการพิเศษ
- ใช้งาน Internationalization (i18n) และ Localization (l10n)
- จัดการ Right-to-Left (RTL) languages และ cultural adaptations
- ทดสอบและตรวจสอบ accessibility ของแอปพลิเคชัน

---

## 1. 🔍 Accessibility Fundamentals

### 1.1 Why Accessibility Matters

การพัฒนา accessible applications ไม่เพียงแต่เป็นสิ่งที่ถูกต้องทางจริยธรรมเท่านั้น แต่ยังเป็นข้อกำหนดทางกฎหมายในหลายประเทศ ตามมาตรฐาน WCAG (Web Content Accessibility Guidelines) และ Section 508 การทำให้แอปของคุณเข้าถึงได้จะขยายฐานผู้ใช้และสร้างประสบการณ์ที่ดีขึ้นสำหรับผู้ใช้ทุกคน

### 1.2 Accessibility Standards ใน Mobile Applications

.NET MAUI รองรับมาตรฐาน accessibility หลัก ๆ เหล่านี้:

- **WCAG 2.1** - Web Content Accessibility Guidelines
- **Section 508** - US Federal requirements
- **EN 301 549** - European standard
- **iOS Accessibility** - Apple's guidelines
- **Android Accessibility** - Google's guidelines

### 1.3 Configuring Semantic Properties in XAML

```xml
<VerticalStackLayout Spacing="25" Padding="30,0" 
                     VerticalOptions="Center">
    <!-- SemanticProperties สำหรับการอ่านออกเสียงที่มีความหมาย -->
    <Image Source="dotnet_bot.png"
           SemanticProperties.Description="The .NET Bot mascot" 
           HeightRequest="200"
           HorizontalOptions="Center" />

    <Label Text="Hello, World!"
           SemanticProperties.HeadingLevel="Level1"
           SemanticProperties.Description="Welcome to the app"
           FontSize="32"
           HorizontalOptions="Center" />

    <Button Text="Click Me"
            SemanticProperties.Hint="Performs the main action"
            AutomationProperties.IsInAccessibleTree="True" 
            HorizontalOptions="Center" />
</VerticalStackLayout>
```

### 1.4 AutomationProperties API

```csharp
// In code-behind or ViewModel
public void ConfigureAccessibility()
{
    // Setting essential automation properties
    myButton.SetValue(AutomationProperties.NameProperty, "Submit Form");
    myButton.SetValue(AutomationProperties.HelpTextProperty, "Tap to submit the form");
    myButton.SetValue(AutomationProperties.IsInAccessibleTreeProperty, true);
    
    // Setting automation Id for testing
    myButton.SetValue(AutomationProperties.AutomationIdProperty, "SubmitButton");
    
    // Grouping elements for screen readers
    myTextField.SetValue(AutomationProperties.LabeledByProperty, myLabel);
    
    // Setting a live region (announces changes)
    statusMessage.SetValue(AutomationProperties.LiveSettingProperty, "Polite");
    
    // Hiding decorative elements from screen readers
    decorativeImage.SetValue(AutomationProperties.IsInAccessibleTreeProperty, false);
}
```

---

## 2. 🖥️ Screen Reader Support

### 2.1 Testing with Screen Readers

การทดสอบกับ screen reader เป็นสิ่งสำคัญสำหรับการสร้างแอปที่เข้าถึงได้ แต่ละแพลตฟอร์มมี screen reader เฉพาะของตัวเอง:

- **Android**: TalkBack
- **iOS**: VoiceOver
- **Windows**: Narrator

### 2.2 Ensuring Text-to-Speech Quality

```csharp
// ให้ข้อมูลที่มีความหมายกับ screen readers
public class AccessibilityHelper
{
    public static void ConfigureElementForScreenReader(VisualElement element, string name, string helpText)
    {
        AutomationProperties.SetName(element, name);
        AutomationProperties.SetHelpText(element, helpText);
        AutomationProperties.SetIsInAccessibleTree(element, true);
    }
    
    public static void AnnounceScreenChange(string announcement)
    {
        // Announce page changes or important updates
        #if ANDROID
        var context = Platform.CurrentActivity;
        if (context != null)
        {
            var manager = context.GetSystemService(Android.Content.Context.AccessibilityService) 
                as Android.Views.Accessibility.AccessibilityManager;
            if (manager != null && manager.IsEnabled)
            {
                var e = new Android.Views.Accessibility.AccessibilityEvent();
                e.EventType = EventTypes.Announcement;
                e.Text.Add(new Java.Lang.String(announcement));
                manager.SendAccessibilityEvent(e);
            }
        }
        #elif IOS
        UIKit.UIAccessibility.PostNotification(
            UIKit.UIAccessibilityPostNotification.Announcement, 
            new Foundation.NSString(announcement));
        #elif WINDOWS
        // Windows screen reader announcement code
        #endif
    }
}
```

### 2.3 Focus Management

```csharp
public class FocusHelper
{
    // จัดการลำดับ focus ในฟอร์ม
    public static void SetupTabOrder(List<View> elements)
    {
        for (int i = 0; i < elements.Count; i++)
        {
            var current = elements[i];
            if (i < elements.Count - 1)
            {
                var next = elements[i + 1];
                current.SetValue(AutomationProperties.NextProperty, next);
            }
        }
    }
    
    // ย้าย focus ไปยัง element ที่สำคัญ (เช่น หลังจาก navigation หรือเปิด dialog)
    public static async Task SetFocusToElement(VisualElement element, bool animateFocus = true)
    {
        await Task.Delay(300); // รอให้ UI render
        
        if (animateFocus)
        {
            // เพิ่มเอฟเฟกต์ animation focus เพื่อช่วยผู้ใช้
            var originalScale = element.Scale;
            await element.ScaleTo(1.05, 150);
            await element.ScaleTo(originalScale, 100);
        }
        
        element.Focus();
    }
}
```

---

## 3. 🌍 Internationalization & Localization (i18n/l10n)

### 3.1 Setting Up Resources Files

```xml
<!-- Project file structure -->
Resources/
  ├── Strings/
  │   ├── AppResources.resx        (Default - English)
  │   ├── AppResources.th-TH.resx  (Thai)
  │   ├── AppResources.ja.resx     (Japanese)
  │   └── AppResources.de.resx     (German)
  │
  └── Images/
      ├── Default/
      ├── th-TH/
      ├── ja/
      └── de/
```

```xml
<!-- AppResources.resx -->
<data name="WelcomeMessage" xml:space="preserve">
  <value>Welcome to our app!</value>
  <comment>Main welcome message</comment>
</data>

<data name="Greeting" xml:space="preserve">
  <value>Hello, {0}!</value>
  <comment>Personal greeting with name parameter</comment>
</data>

<!-- AppResources.th-TH.resx -->
<data name="WelcomeMessage" xml:space="preserve">
  <value>ยินดีต้อนรับสู่แอปพลิเคชันของเรา!</value>
  <comment>Main welcome message</comment>
</data>

<data name="Greeting" xml:space="preserve">
  <value>สวัสดี, {0}!</value>
  <comment>Personal greeting with name parameter</comment>
</data>
```

### 3.2 Creating Resource Manager

```csharp
// LocalizationResourceManager.cs
public class LocalizationResourceManager : INotifyPropertyChanged
{
    private static readonly Lazy<LocalizationResourceManager> _instance =
        new Lazy<LocalizationResourceManager>(() => new LocalizationResourceManager());
    
    public static LocalizationResourceManager Instance => _instance.Value;
    
    private readonly ResourceManager _resourceManager;
    private CultureInfo _currentCulture;
    
    public event PropertyChangedEventHandler PropertyChanged;
    
    private LocalizationResourceManager()
    {
        _resourceManager = AppResources.ResourceManager;
        _currentCulture = Thread.CurrentThread.CurrentUICulture;
    }
    
    public string GetString(string key)
    {
        return _resourceManager.GetString(key, _currentCulture) ?? key;
    }
    
    public string GetString(string key, params object[] formatArgs)
    {
        string value = GetString(key);
        if (!string.IsNullOrEmpty(value) && formatArgs != null)
            return string.Format(value, formatArgs);
        return value;
    }
    
    public CultureInfo CurrentCulture
    {
        get => _currentCulture;
        set
        {
            if (_currentCulture != value)
            {
                _currentCulture = value;
                Thread.CurrentThread.CurrentUICulture = value;
                
                // Save selected culture for app restarts
                Preferences.Default.Set("AppCulture", value.Name);
                
                // Notify all bindings
                PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(null));
            }
        }
    }
    
    public void SetCulture(string cultureName)
    {
        CurrentCulture = CultureInfo.GetCultureInfo(cultureName);
    }
}
```

### 3.3 Creating XAML Markup Extension

```csharp
// TranslateExtension.cs
[ContentProperty(nameof(Key))]
public class TranslateExtension : IMarkupExtension
{
    public string Key { get; set; }
    public object[] StringFormat { get; set; }
    
    public object ProvideValue(IServiceProvider serviceProvider)
    {
        if (string.IsNullOrEmpty(Key))
            return string.Empty;
        
        var localizationManager = LocalizationResourceManager.Instance;
        var translation = StringFormat != null 
            ? localizationManager.GetString(Key, StringFormat)
            : localizationManager.GetString(Key);
            
        return translation;
    }
}
```

### 3.4 Using the Localization in XAML

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:local="clr-namespace:YourApp"
             x:Class="YourApp.MainPage">
    
    <VerticalStackLayout Padding="20">
        <!-- Static localized text -->
        <Label Text="{local:Translate WelcomeMessage}" 
               FontSize="24" 
               HorizontalOptions="Center" />
        
        <!-- Localized text with parameters -->
        <Label Text="{local:Translate Greeting, {Binding Username}}" 
               FontSize="18" 
               Margin="0,20,0,0" />
        
        <!-- Culture selection picker -->
        <Picker Title="Select Language" 
                ItemsSource="{Binding AvailableCultures}"
                SelectedItem="{Binding SelectedCulture}"
                DisplayMemberPath="DisplayName"
                Margin="0,30,0,0" />
    </VerticalStackLayout>
</ContentPage>
```

### 3.5 Language Selection ViewModel

```csharp
// LanguageViewModel.cs
public class LanguageViewModel : ObservableObject
{
    private readonly List<CultureInfo> _availableCultures = new List<CultureInfo>
    {
        new CultureInfo("en-US"),    // English
        new CultureInfo("th-TH"),    // Thai
        new CultureInfo("ja"),       // Japanese
        new CultureInfo("de")        // German
    };
    
    private CultureDisplay _cultureDisplay = CultureDisplay.NativeName;
    private CultureInfo _selectedCulture;
    
    public LanguageViewModel()
    {
        // Load saved culture or default
        string savedCulture = Preferences.Default.Get("AppCulture", "en-US");
        _selectedCulture = _availableCultures.FirstOrDefault(c => c.Name == savedCulture) 
            ?? _availableCultures.First();
            
        LocalizationResourceManager.Instance.CurrentCulture = _selectedCulture;
    }
    
    public List<CultureViewModel> AvailableCultures => _availableCultures
        .Select(c => new CultureViewModel
        {
            CultureInfo = c,
            DisplayName = _cultureDisplay == CultureDisplay.NativeName 
                ? c.NativeName 
                : c.DisplayName
        })
        .ToList();
        
    public CultureViewModel SelectedCulture
    {
        get => AvailableCultures.FirstOrDefault(c => 
            c.CultureInfo.Name == _selectedCulture?.Name);
        set
        {
            if (value?.CultureInfo != _selectedCulture)
            {
                _selectedCulture = value?.CultureInfo;
                LocalizationResourceManager.Instance.CurrentCulture = _selectedCulture;
                OnPropertyChanged(nameof(SelectedCulture));
            }
        }
    }
    
    public CultureDisplay CultureDisplayMode
    {
        get => _cultureDisplay;
        set
        {
            if (_cultureDisplay != value)
            {
                _cultureDisplay = value;
                OnPropertyChanged(nameof(CultureDisplayMode));
                OnPropertyChanged(nameof(AvailableCultures));
            }
        }
    }
}

public class CultureViewModel
{
    public CultureInfo CultureInfo { get; set; }
    public string DisplayName { get; set; }
    
    public override string ToString() => DisplayName;
}

public enum CultureDisplay
{
    DisplayName,   // "English (United States)"
    NativeName     // "English (United States)" or "ไทย (ไทย)" etc.
}
```

---

## 4. 🔄 Right-to-Left (RTL) Support

### 4.1 Enabling RTL Support

```xml
<!-- In AppShell.xaml or App.xaml -->
<Application xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="YourApp.App"
             FlowDirection="MatchParent">
    <!-- FlowDirection can be:
         - MatchParent: Follow the parent/system setting
         - LeftToRight: Force LTR
         - RightToLeft: Force RTL -->
</Application>
```

```csharp
// In App.xaml.cs
public partial class App : Application
{
    public App()
    {
        InitializeComponent();
        
        // Set flow direction based on current culture
        UpdateFlowDirection();
        
        MainPage = new AppShell();
    }
    
    private void UpdateFlowDirection()
    {
        var culture = LocalizationResourceManager.Instance.CurrentCulture;
        bool isRightToLeft = culture.TextInfo.IsRightToLeft;
        
        FlowDirection = isRightToLeft ? FlowDirection.RightToLeft : FlowDirection.LeftToRight;
    }
}
```

### 4.2 RTL-Aware Layout & Margins

```xml
<!-- Layout ที่รองรับ RTL & LTR -->
<Grid Padding="{OnIdiom Phone='20,10', Tablet='40,20'}"
      FlowDirection="{Binding Source={x:Static local:Settings.FlowDirection}}">
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="Auto" />
        <ColumnDefinition Width="*" />
    </Grid.ColumnDefinitions>
    
    <!-- ปุ่ม Back จะอยู่ด้านซ้ายใน LTR และด้านขวาใน RTL โดยอัตโนมัติ -->
    <Button Grid.Column="0" 
            Text="Back"
            SemanticProperties.Hint="Go back to previous page"
            AutomationProperties.IsInAccessibleTree="True" />
            
    <!-- ข้อความจะจัดเรียงตามทิศทางภาษาโดยอัตโนมัติ -->
    <Label Grid.Column="1"
           Text="{local:Translate PageTitle}"
           HorizontalOptions="Fill"
           HorizontalTextAlignment="Start" /> <!-- Start จะปรับเป็นซ้ายหรือขวาตาม FlowDirection -->
</Grid>
```

### 4.3 RTL-Safe Alignment & Padding

```csharp
// RTLHelper.cs - Helper class for RTL layouts
public static class RTLHelper
{
    public static LayoutOptions GetStartOptions(bool isRTL = false)
    {
        return isRTL ? LayoutOptions.End : LayoutOptions.Start;
    }
    
    public static LayoutOptions GetEndOptions(bool isRTL = false)
    {
        return isRTL ? LayoutOptions.Start : LayoutOptions.End;
    }
    
    public static Thickness GetStartPadding(double value, bool isRTL = false)
    {
        return isRTL ? new Thickness(0, 0, value, 0) : new Thickness(value, 0, 0, 0);
    }
    
    public static Thickness GetEndPadding(double value, bool isRTL = false)
    {
        return isRTL ? new Thickness(value, 0, 0, 0) : new Thickness(0, 0, value, 0);
    }
    
    public static TextAlignment GetStartAlignment(bool isRTL = false)
    {
        return isRTL ? TextAlignment.End : TextAlignment.Start;
    }
}
```

### 4.4 RTL Images & Icons

```xml
<!-- Icon ที่ปรับตาม RTL -->
<Grid>
    <Image Source="{local:DirectionalImage 'arrow_forward.png', 'arrow_back.png'}"
           HeightRequest="24" />
</Grid>
```

```csharp
// DirectionalImageExtension.cs
[ContentProperty(nameof(LtrSource))]
public class DirectionalImageExtension : IMarkupExtension<ImageSource>
{
    public string LtrSource { get; set; }
    public string RtlSource { get; set; }
    
    public ImageSource ProvideValue(IServiceProvider serviceProvider)
    {
        if (string.IsNullOrEmpty(LtrSource))
            return null;
            
        bool isRightToLeft = CultureInfo.CurrentCulture.TextInfo.IsRightToLeft;
        string sourceToUse = isRightToLeft && !string.IsNullOrEmpty(RtlSource) 
            ? RtlSource 
            : LtrSource;
            
        return ImageSource.FromFile(sourceToUse);
    }
    
    object IMarkupExtension.ProvideValue(IServiceProvider serviceProvider)
    {
        return ProvideValue(serviceProvider);
    }
}
```

---

## 5. 📱 Cultural Adaptations

### 5.1 Date & Time Formatting

```csharp
// DateTime formatter with cultural awareness
public class CultureAwareDateFormatter
{
    public static string FormatDate(DateTime dateTime, string format = null)
    {
        var culture = LocalizationResourceManager.Instance.CurrentCulture;
        
        if (string.IsNullOrEmpty(format))
            return dateTime.ToString(culture);
        
        return dateTime.ToString(format, culture);
    }
    
    public static string GetRelativeDate(DateTime dateTime)
    {
        var culture = LocalizationResourceManager.Instance.CurrentCulture;
        var diff = DateTime.Now - dateTime;
        
        // Cultural-specific relative time formatting
        if (diff.TotalDays < 1)
        {
            if (diff.TotalHours < 1)
                return string.Format(
                    LocalizationResourceManager.Instance.GetString("MinutesAgo"), 
                    (int)diff.TotalMinutes);
                
            return string.Format(
                LocalizationResourceManager.Instance.GetString("HoursAgo"), 
                (int)diff.TotalHours);
        }
        
        if (diff.TotalDays < 7)
            return string.Format(
                LocalizationResourceManager.Instance.GetString("DaysAgo"), 
                (int)diff.TotalDays);
                
        return dateTime.ToString("d", culture);
    }
}
```

### 5.2 Number & Currency Formatting

```csharp
// CurrencyFormatter.cs
public class CurrencyFormatter
{
    public static string FormatCurrency(decimal amount, string currencyCode = null)
    {
        var culture = LocalizationResourceManager.Instance.CurrentCulture;
        
        if (string.IsNullOrEmpty(currencyCode))
            return amount.ToString("C", culture);
        
        var regionInfo = new RegionInfo(culture.Name);
        
        // Override currency if specified
        if (!string.IsNullOrEmpty(currencyCode))
        {
            var customCulture = (CultureInfo)culture.Clone();
            var region = new RegionInfo(currencyCode);
            customCulture.NumberFormat.CurrencySymbol = region.CurrencySymbol;
            
            return amount.ToString("C", customCulture);
        }
        
        return amount.ToString("C", culture);
    }
    
    public static string FormatNumber(double number, int decimals = 2)
    {
        var culture = LocalizationResourceManager.Instance.CurrentCulture;
        return number.ToString($"N{decimals}", culture);
    }
    
    public static string FormatPercentage(double value)
    {
        var culture = LocalizationResourceManager.Instance.CurrentCulture;
        return value.ToString("P", culture);
    }
}
```

### 5.3 Name & Address Formatting

```csharp
// Cultural-aware name formatter
public class NameFormatter
{
    public static string FormatName(string firstName, string lastName, string middleName = null)
    {
        var culture = LocalizationResourceManager.Instance.CurrentCulture;
        
        // Western style: Given name + Family name
        if (culture.Name.StartsWith("en") || 
            culture.Name.StartsWith("de") || 
            culture.Name.StartsWith("fr"))
        {
            string middlePart = string.IsNullOrEmpty(middleName) ? "" : $" {middleName}";
            return $"{firstName}{middlePart} {lastName}";
        }
        
        // East Asian style: Family name + Given name
        if (culture.Name.StartsWith("zh") || 
            culture.Name.StartsWith("ja") || 
            culture.Name.StartsWith("ko"))
        {
            return $"{lastName}{firstName}";
        }
        
        // Thai style: Just the first name in most casual contexts
        if (culture.Name.StartsWith("th"))
        {
            return firstName;
        }
        
        // Default to Western style
        return $"{firstName} {lastName}";
    }
    
    public static string FormatAddress(Address address)
    {
        var culture = LocalizationResourceManager.Instance.CurrentCulture;
        var formatter = new AddressFormatter(culture.Name);
        return formatter.Format(address);
    }
    
    private class AddressFormatter
    {
        private readonly string _cultureCode;
        
        public AddressFormatter(string cultureCode)
        {
            _cultureCode = cultureCode;
        }
        
        public string Format(Address address)
        {
            // Different countries have different address formats
            if (_cultureCode.StartsWith("us") || _cultureCode.StartsWith("en"))
            {
                // US format: street, city, state ZIP
                return $"{address.Street}\n{address.City}, {address.State} {address.PostalCode}";
            }
            
            if (_cultureCode.StartsWith("jp"))
            {
                // Japan format: postal code, prefecture, city, street
                return $"〒{address.PostalCode}\n{address.State}{address.City}\n{address.Street}";
            }
            
            // Add more country-specific formats as needed
            
            // Default format
            return $"{address.Street}\n{address.PostalCode} {address.City}\n{address.State}";
        }
    }
}

public class Address
{
    public string Street { get; set; }
    public string City { get; set; }
    public string State { get; set; }
    public string PostalCode { get; set; }
    public string Country { get; set; }
}
```

---

## 6. 🧪 Accessibility Testing

### 6.1 Automated Accessibility Testing

```csharp
// AccessibilityTestingHelper.cs
public class AccessibilityTestingHelper
{
    public static List<AccessibilityIssue> AnalyzeScreen(Page page)
    {
        var issues = new List<AccessibilityIssue>();
        
        // Recursively analyze all elements
        AnalyzeElement(page, issues);
        
        return issues;
    }
    
    private static void AnalyzeElement(Element element, List<AccessibilityIssue> issues)
    {
        // Skip invisible elements
        if (element is VisualElement visual && !visual.IsVisible)
            return;
        
        // Check accessibility properties
        if (element is VisualElement ve)
        {
            // Check for missing automation properties
            if (ve is Button || ve is Entry || ve is SearchBar || ve is Slider)
            {
                bool hasName = AutomationProperties.GetName(ve) != null;
                bool hasHelpText = AutomationProperties.GetHelpText(ve) != null;
                bool isInTree = AutomationProperties.GetIsInAccessibleTree(ve) ?? true;
                
                if (isInTree && !hasName)
                {
                    issues.Add(new AccessibilityIssue
                    {
                        Element = ve,
                        IssueType = AccessibilityIssueType.MissingName,
                        Description = "Interactive element is missing an accessible name"
                    });
                }
                
                if (isInTree && !hasHelpText)
                {
                    issues.Add(new AccessibilityIssue
                    {
                        Element = ve,
                        IssueType = AccessibilityIssueType.MissingHelpText,
                        Description = "Interactive element is missing help text"
                    });
                }
            }
            
            // Check for small touch targets
            if (ve is Button or ImageButton or TapGestureRecognizer)
            {
                if (ve.Width < 44 || ve.Height < 44)
                {
                    issues.Add(new AccessibilityIssue
                    {
                        Element = ve,
                        IssueType = AccessibilityIssueType.SmallTouchTarget,
                        Description = "Touch target is smaller than 44x44, which is difficult for users with motor impairments"
                    });
                }
            }
            
            // Check for color contrast
            if (ve is Label label)
            {
                // This would require color analysis
                // Simplified placeholder for demonstration
            }
        }
        
        // Recursively check child elements
        if (element is IElementController controller)
        {
            foreach (var child in controller.LogicalChildren)
            {
                if (child is Element childElement)
                {
                    AnalyzeElement(childElement, issues);
                }
            }
        }
    }
}

public class AccessibilityIssue
{
    public Element Element { get; set; }
    public AccessibilityIssueType IssueType { get; set; }
    public string Description { get; set; }
}

public enum AccessibilityIssueType
{
    MissingName,
    MissingHelpText,
    SmallTouchTarget,
    LowContrast,
    MissingLabel,
    DuplicateLabels
}
```

### 6.2 Visual Accessibility Checker

```csharp
// AccessibilityCheckerPage.xaml.cs
public partial class AccessibilityCheckerPage : ContentPage
{
    public ObservableCollection<AccessibilityIssue> Issues { get; } = new();
    
    public AccessibilityCheckerPage()
    {
        InitializeComponent();
        BindingContext = this;
    }
    
    public async void ScanCurrentPage_Clicked(object sender, EventArgs e)
    {
        var mainPage = Application.Current.MainPage;
        var issues = AccessibilityTestingHelper.AnalyzeScreen(mainPage);
        
        Issues.Clear();
        foreach (var issue in issues)
        {
            Issues.Add(issue);
        }
        
        // Report summary
        await DisplayAlert(
            "Accessibility Scan Complete", 
            $"Found {issues.Count} potential issues", 
            "OK");
    }
    
    public async void ViewAccessibilityReport_Clicked(object sender, EventArgs e)
    {
        if (Issues.Count == 0)
        {
            await DisplayAlert("No Issues", "No accessibility issues were found or scan not performed yet.", "OK");
            return;
        }
        
        var report = new StringBuilder();
        report.AppendLine("# Accessibility Issues Report");
        report.AppendLine();
        
        foreach (var issueGroup in Issues.GroupBy(i => i.IssueType))
        {
            report.AppendLine($"## {issueGroup.Key} ({issueGroup.Count()})");
            report.AppendLine();
            
            foreach (var issue in issueGroup)
            {
                report.AppendLine($"- {issue.Description}");
                
                if (issue.Element is VisualElement ve)
                {
                    report.AppendLine($"  - Element: {ve.GetType().Name}");
                    
                    if (AutomationProperties.GetAutomationId(ve) is string id)
                    {
                        report.AppendLine($"  - AutomationId: {id}");
                    }
                }
                
                report.AppendLine();
            }
        }
        
        await Navigation.PushAsync(new AccessibilityReportPage(report.ToString()));
    }
}
```

### 6.3 Live Accessibility Overlay

```csharp
// AccessibilityOverlayHandler.cs - Visual overlay for checking accessibility
public class AccessibilityOverlay : ContentView
{
    private readonly AbsoluteLayout _rootLayout = new AbsoluteLayout();
    
    public AccessibilityOverlay()
    {
        Opacity = 0.7;
        BackgroundColor = Colors.Transparent;
        Content = _rootLayout;
        IsVisible = false;
    }
    
    public void Analyze(Page page)
    {
        _rootLayout.Children.Clear();
        IsVisible = true;
        
        AnalyzeElement(page);
    }
    
    private void AnalyzeElement(Element element)
    {
        if (element is VisualElement ve)
        {
            // Get global position and bounds
            var bounds = GetGlobalBounds(ve);
            
            // Skip elements with no size
            if (bounds.Width <= 0 || bounds.Height <= 0)
                return;
            
            // Create visual indicator for touch target size
            if (ve is Button or ImageButton or TapGestureRecognizer)
            {
                var touchIndicator = new BoxView
                {
                    Color = bounds.Width < 44 || bounds.Height < 44 ? Colors.Red : Colors.Green,
                    Opacity = 0.3,
                    CornerRadius = 5
                };
                
                _rootLayout.Children.Add(touchIndicator, bounds);
            }
            
            // Create visual indicator for accessibility properties
            bool hasName = AutomationProperties.GetName(ve) != null;
            bool isInTree = AutomationProperties.GetIsInAccessibleTree(ve) ?? true;
            
            if (ve is Button or Entry or SearchBar or Slider)
            {
                if (isInTree && !hasName)
                {
                    var warningIndicator = new BoxView
                    {
                        Color = Colors.Orange,
                        Opacity = 0.5,
                        CornerRadius = 5
                    };
                    
                    _rootLayout.Children.Add(warningIndicator, bounds);
                }
            }
        }
        
        // Recursively check child elements
        if (element is IElementController controller)
        {
            foreach (var child in controller.LogicalChildren)
            {
                if (child is Element childElement)
                {
                    AnalyzeElement(childElement);
                }
            }
        }
    }
    
    private Rect GetGlobalBounds(VisualElement element)
    {
        // Get global bounds for the element
        // This is a simplified implementation
        return new Rect(0, 0, element.Width, element.Height);
    }
    
    public void Hide()
    {
        IsVisible = false;
        _rootLayout.Children.Clear();
    }
}
```

---

## 7. 📊 Combined Accessibility & Localization Strategy

### 7.1 Creating a Complete i18n/a11y Service

```csharp
// AppServices/AccessibilityLocalizationService.cs
public class AccessibilityLocalizationService : IAccessibilityLocalizationService
{
    private readonly LocalizationResourceManager _localizationManager;
    
    public AccessibilityLocalizationService()
    {
        _localizationManager = LocalizationResourceManager.Instance;
        
        // Restore user preferences
        LoadUserPreferences();
    }
    
    public CultureInfo CurrentCulture => _localizationManager.CurrentCulture;
    
    public bool IsRightToLeft => CurrentCulture.TextInfo.IsRightToLeft;
    
    public FlowDirection FlowDirection => IsRightToLeft ? 
        FlowDirection.RightToLeft : FlowDirection.LeftToRight;
        
    public void ChangeCulture(string cultureName)
    {
        _localizationManager.SetCulture(cultureName);
        SaveUserPreferences();
    }
    
    // เพิ่มการรองรับคุณสมบัติ Accessibility
    public bool IsScreenReaderEnabled 
    {
        get
        {
            bool defaultValue = false;
            
            #if ANDROID
            var context = Android.App.Application.Context;
            var manager = context.GetSystemService(Android.Content.Context.AccessibilityService) 
                as Android.Views.Accessibility.AccessibilityManager;
            return manager?.IsEnabled ?? defaultValue;
            #elif IOS
            return UIKit.UIAccessibility.IsVoiceOverRunning;
            #elif WINDOWS
            // Use Windows API to check if Narrator is running
            return defaultValue;
            #else
            return defaultValue;
            #endif
        }
    }
    
    public bool IsHighContrastEnabled
    {
        get
        {
            bool defaultValue = false;
            
            #if ANDROID
            return defaultValue; // Need to check system settings
            #elif IOS
            return UIKit.UIAccessibility.IsHighContrastEnabled;
            #elif WINDOWS
            // Use Windows API to check high contrast mode
            return defaultValue;
            #else
            return defaultValue;
            #endif
        }
    }
    
    public float TextSizeScale
    {
        get
        {
            float defaultValue = 1.0f;
            
            #if ANDROID
            var context = Android.App.Application.Context;
            return context.Resources.Configuration.FontScale;
            #elif IOS
            return (float)UIKit.UIApplication.SharedApplication.PreferredContentSizeCategory.GetHashCode() / 4f;
            #elif WINDOWS
            // Use Windows API to get text size scale
            return defaultValue;
            #else
            return defaultValue;
            #endif
        }
    }
    
    public void AnnounceScreenChange(string announcement)
    {
        if (string.IsNullOrEmpty(announcement))
            return;
            
        #if ANDROID
        var context = Platform.CurrentActivity;
        if (context != null)
        {
            var manager = context.GetSystemService(Android.Content.Context.AccessibilityService) 
                as Android.Views.Accessibility.AccessibilityManager;
            if (manager?.IsEnabled == true)
            {
                var e = new Android.Views.Accessibility.AccessibilityEvent();
                e.EventType = EventTypes.Announcement;
                e.Text.Add(new Java.Lang.String(announcement));
                manager.SendAccessibilityEvent(e);
            }
        }
        #elif IOS
        UIKit.UIAccessibility.PostNotification(
            UIKit.UIAccessibilityPostNotification.Announcement, 
            new Foundation.NSString(announcement));
        #elif WINDOWS
        // Use Windows API for screen reader announcement
        #endif
    }
    
    // Helper methods for formatting with cultural awareness
    public string FormatCurrency(decimal amount) => 
        CurrencyFormatter.FormatCurrency(amount);
        
    public string FormatDate(DateTime date) => 
        CultureAwareDateFormatter.FormatDate(date);
        
    public string FormatRelativeTime(DateTime date) =>
        CultureAwareDateFormatter.GetRelativeDate(date);
        
    public string FormatName(string firstName, string lastName, string middleName = null) =>
        NameFormatter.FormatName(firstName, lastName, middleName);
        
    // Save and load user preferences
    private void SaveUserPreferences()
    {
        Preferences.Default.Set("AppCulture", CurrentCulture.Name);
    }
    
    private void LoadUserPreferences()
    {
        string savedCulture = Preferences.Default.Get("AppCulture", "en-US");
        _localizationManager.SetCulture(savedCulture);
    }
}

// Interface for the service
public interface IAccessibilityLocalizationService
{
    CultureInfo CurrentCulture { get; }
    bool IsRightToLeft { get; }
    FlowDirection FlowDirection { get; }
    bool IsScreenReaderEnabled { get; }
    bool IsHighContrastEnabled { get; }
    float TextSizeScale { get; }
    
    void ChangeCulture(string cultureName);
    void AnnounceScreenChange(string announcement);
    string FormatCurrency(decimal amount);
    string FormatDate(DateTime date);
    string FormatRelativeTime(DateTime date);
    string FormatName(string firstName, string lastName, string middleName = null);
}
```

### 7.2 Registering the Service

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
                fonts.AddFont("OpenSans-Semibold.ttf", "OpenSansSemibold");
            })
            .RegisterAppServices();

        return builder.Build();
    }
    
    private static MauiAppBuilder RegisterAppServices(this MauiAppBuilder builder)
    {
        // Register singleton services
        builder.Services.AddSingleton<IAccessibilityLocalizationService, AccessibilityLocalizationService>();
        
        // Register view models that depend on the service
        builder.Services.AddTransient<SettingsViewModel>();
        
        // Register pages
        builder.Services.AddTransient<SettingsPage>();
        
        return builder;
    }
}
```

### 7.3 Using the Service in Views

```xml
<!-- SettingsPage.xaml -->
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:local="clr-namespace:YourApp"
             xmlns:vm="clr-namespace:YourApp.ViewModels"
             x:Class="YourApp.Views.SettingsPage"
             x:DataType="vm:SettingsViewModel"
             FlowDirection="{Binding FlowDirection}">
    
    <ScrollView>
        <VerticalStackLayout Padding="20" Spacing="15">
            <Label Text="{local:Translate SettingsTitle}"
                   FontSize="24"
                   HorizontalOptions="Center" />
                   
            <!-- Language Selection -->
            <Frame BorderColor="{StaticResource BorderColor}" Padding="15">
                <VerticalStackLayout Spacing="10">
                    <Label Text="{local:Translate LanguageSettings}"
                           FontAttributes="Bold" />
                           
                    <Picker Title="{local:Translate SelectLanguage}"
                            ItemsSource="{Binding AvailableLanguages}"
                            SelectedItem="{Binding SelectedLanguage}"
                            HorizontalOptions="Fill" />
                </VerticalStackLayout>
            </Frame>
            
            <!-- Accessibility Settings -->
            <Frame BorderColor="{StaticResource BorderColor}" Padding="15">
                <VerticalStackLayout Spacing="10">
                    <Label Text="{local:Translate AccessibilitySettings}"
                           FontAttributes="Bold" />
                           
                    <!-- Text Size -->
                    <Label Text="{local:Translate TextSize}" />
                    <Slider Value="{Binding TextSizeScale}"
                            Minimum="0.8"
                            Maximum="1.5"
                            SemanticProperties.Hint="{local:Translate TextSizeHint}" />
                            
                    <!-- High Contrast Mode -->
                    <CheckBox IsChecked="{Binding UseHighContrast}"
                              SemanticProperties.Description="{local:Translate HighContrastMode}" />
                    <Label Text="{local:Translate HighContrastMode}"
                           VerticalOptions="Center" />
                            
                    <!-- Screen Reader Info -->
                    <Label Text="{local:Translate ScreenReaderActive}"
                           IsVisible="{Binding IsScreenReaderActive}" />
                </VerticalStackLayout>
            </Frame>
            
            <!-- Display Examples of Cultural Formatting -->
            <Frame BorderColor="{StaticResource BorderColor}" Padding="15">
                <VerticalStackLayout Spacing="10">
                    <Label Text="{local:Translate FormattingExamples}"
                           FontAttributes="Bold" />
                           
                    <Label Text="{Binding CurrentDateFormatted}" />
                    <Label Text="{Binding CurrencyFormatted}" />
                    <Label Text="{Binding NameFormatted}" />
                </VerticalStackLayout>
            </Frame>
            
            <!-- Accessibility Checker Tool -->
            <Button Text="{local:Translate RunAccessibilityCheck}"
                    Command="{Binding RunAccessibilityCheckCommand}"
                    BackgroundColor="{StaticResource PrimaryColor}"
                    TextColor="White"
                    HorizontalOptions="Center" />
        </VerticalStackLayout>
    </ScrollView>
</ContentPage>
```

### 7.4 Settings ViewModel

```csharp
// SettingsViewModel.cs
public class SettingsViewModel : ObservableObject
{
    private readonly IAccessibilityLocalizationService _accessibilityLocalizationService;
    private float _textSizeScale = 1.0f;
    private bool _useHighContrast;
    private string _selectedLanguage;
    
    public SettingsViewModel(IAccessibilityLocalizationService accessibilityLocalizationService)
    {
        _accessibilityLocalizationService = accessibilityLocalizationService;
        
        // Initialize properties from the service
        _textSizeScale = _accessibilityLocalizationService.TextSizeScale;
        _useHighContrast = _accessibilityLocalizationService.IsHighContrastEnabled;
        _selectedLanguage = _accessibilityLocalizationService.CurrentCulture.Name;
        
        // Initialize commands
        RunAccessibilityCheckCommand = new Command(RunAccessibilityCheck);
        
        // Create available languages list
        AvailableLanguages = new List<string>
        {
            "en-US", // English (United States)
            "th-TH", // Thai
            "ja",    // Japanese
            "de",    // German
            "ar",    // Arabic (RTL example)
            "fr"     // French
        };
    }
    
    public List<string> AvailableLanguages { get; }
    
    public string SelectedLanguage
    {
        get => _selectedLanguage;
        set
        {
            if (SetProperty(ref _selectedLanguage, value))
            {
                _accessibilityLocalizationService.ChangeCulture(value);
                OnPropertyChanged(nameof(FlowDirection));
                OnPropertyChanged(nameof(CurrentDateFormatted));
                OnPropertyChanged(nameof(CurrencyFormatted));
                OnPropertyChanged(nameof(NameFormatted));
            }
        }
    }
    
    public float TextSizeScale
    {
        get => _textSizeScale;
        set => SetProperty(ref _textSizeScale, value);
    }
    
    public bool UseHighContrast
    {
        get => _useHighContrast;
        set => SetProperty(ref _useHighContrast, value);
    }
    
    public bool IsScreenReaderActive => _accessibilityLocalizationService.IsScreenReaderEnabled;
    
    public FlowDirection FlowDirection => _accessibilityLocalizationService.FlowDirection;
    
    // Cultural formatting examples
    public string CurrentDateFormatted => _accessibilityLocalizationService.FormatDate(DateTime.Now);
    
    public string CurrencyFormatted => _accessibilityLocalizationService.FormatCurrency(1234.56m);
    
    public string NameFormatted => _accessibilityLocalizationService.FormatName("John", "Smith");
    
    // Commands
    public ICommand RunAccessibilityCheckCommand { get; }
    
    private async void RunAccessibilityCheck()
    {
        await Application.Current.MainPage.DisplayAlert(
            "Accessibility Check",
            "Scanning UI elements for accessibility issues...",
            "OK");
            
        // This would normally trigger the accessibility checker
    }
}
```

---

## 8. 🧩 Practical Exercise

### Accessibility & Localization Challenge

ในแบบฝึกหัดนี้ คุณจะปรับปรุง XAML application ให้มี accessibility และ localization ที่สมบูรณ์:

1. **Setup RESX Files:**
   - สร้างไฟล์ AppResources.resx สำหรับ default language (English)
   - สร้างไฟล์ AppResources.th-TH.resx สำหรับภาษาไทย
   - เพิ่ม key-value pairs สำหรับ strings ทั้งหมดในแอพ

2. **Implement LocalizationManager:**
   - สร้าง LocalizationManager service ที่รองรับการเปลี่ยนภาษา
   - สร้าง XAML markup extension สำหรับการใช้งานใน views

3. **Add Accessibility Features:**
   - เพิ่ม SemanticProperties และ AutomationProperties ให้กับ controls ทั้งหมด
   - ตรวจสอบการทำงานกับ screen reader

4. **Add RTL Support:**
   - ปรับปรุง layouts ให้รองรับภาษาที่อ่านจากขวาไปซ้าย (RTL)
   - ทดสอบกับภาษาอาหรับ

5. **Create Settings Page:**
   - สร้างหน้าตั้งค่าที่ผู้ใช้สามารถเปลี่ยนภาษาได้
   - แสดงตัวอย่างการจัดรูปแบบวันที่ เงิน และชื่อตามวัฒนธรรม

### Model Solution:

```csharp
// AppResources.resx
<data name="WelcomeMessage" xml:space="preserve">
  <value>Welcome to the Accessible App!</value>
</data>
<data name="SettingsTitle" xml:space="preserve">
  <value>App Settings</value>
</data>

// AppResources.th-TH.resx
<data name="WelcomeMessage" xml:space="preserve">
  <value>ยินดีต้อนรับสู่แอปพลิเคชันที่เข้าถึงได้!</value>
</data>
<data name="SettingsTitle" xml:space="preserve">
  <value>ตั้งค่าแอปพลิเคชัน</value>
</data>
```

```csharp
public interface ILocalizeService
{
    // The current culture
    CultureInfo CurrentCulture { get; }
    
    // Get localized string
    string GetString(string key);
    
    // Set current culture
    void SetCulture(string cultureName);
}

public class LocalizeService : ILocalizeService, INotifyPropertyChanged
{
    // Implementation
}

// Use in XAML:
<Label Text="{local:Translate WelcomeMessage}" 
       SemanticProperties.HeadingLevel="Level1"
       AutomationProperties.IsInAccessibleTree="True" />
```

---

## 9. 📚 Summary

ในบทนี้เราได้เรียนรู้:

### 🔑 Key Concepts
- **Accessibility**: การออกแบบแอปพลิเคชันให้ทุกคนสามารถใช้งานได้ รวมถึงผู้ใช้ที่มีความบกพร่องทางการมองเห็น การได้ยิน หรือความสามารถทางร่างกาย
- **Internationalization (i18n)**: การออกแบบแอปพลิเคชันให้รองรับหลายภาษาและวัฒนธรรม
- **Localization (l10n)**: การปรับแต่งแอปพลิเคชันให้เหมาะสมกับภาษาและวัฒนธรรมเฉพาะ
- **RTL Support**: การรองรับภาษาที่อ่านจากขวาไปซ้าย เช่น ภาษาอาหรับ ฮีบรู และอูรดู

### 🧰 Techniques & APIs
- **SemanticProperties**: การกำหนดความหมายให้กับ UI elements สำหรับ screen readers
- **AutomationProperties**: การปรับแต่ง accessibility ขั้นสูง
- **ResourceManager**: การจัดการ localized strings และ resources
- **FlowDirection**: การควบคุมทิศทางการแสดงผล UI สำหรับ RTL/LTR

### 📊 Cultural Considerations
- **Date & Time Formatting**: การแสดงวันเวลาตามรูปแบบของแต่ละวัฒนธรรม
- **Number & Currency**: การแสดงตัวเลขและสกุลเงินให้เหมาะสมกับแต่ละภูมิภาค
- **Name & Address Formats**: การจัดรูปแบบชื่อและที่อยู่ตามธรรมเนียมของแต่ละประเทศ

### 🧪 Testing
- **Accessibility Testing**: การทดสอบแอปพลิเคชันกับ screen readers และ tools อื่น ๆ
- **Automation**: การสร้าง tools สำหรับตรวจสอบ accessibility issues
- **Visual Inspection**: การตรวจสอบด้วยตาเพื่อหาปัญหาที่เห็นได้ชัด

การพัฒนาแอปพลิเคชันที่เข้าถึงได้และรองรับหลากหลายวัฒนธรรมไม่เพียงแต่ช่วยให้ผู้ใช้ทุกคนสามารถใช้งานได้ แต่ยังเป็นการเพิ่มฐานผู้ใช้และสร้างประสบการณ์ที่ดีขึ้นสำหรับผู้ใช้ทั่วโลก การใส่ใจในรายละเอียดเหล่านี้จะทำให้แอปของคุณโดดเด่นและเป็นมืออาชีพมากขึ้น

**Next Chapter Preview:** ในบทต่อไป เราจะเจาะลึกไปที่ **Enterprise Features** และ **Security Practices** ที่จำเป็นสำหรับการพัฒนาแอปพลิเคชันระดับองค์กรด้วย .NET MAUI! 🔐
