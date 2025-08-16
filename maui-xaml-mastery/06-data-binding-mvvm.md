# 🔗 บทที่ 6: Data Binding & MVVM
> **เชี่ยวชาญ Data Binding และ MVVM Pattern**

---

## 📋 สารบัญ
1. [Data Binding Fundamentals](#data-binding-fundamentals)
2. [MVVM Pattern Deep Dive](#mvvm-pattern-deep-dive)
3. [ObservableObject & Properties](#observableobject-properties)
4. [Commands & RelayCommand](#commands-relaycommand)
5. [Value Converters](#value-converters)
6. [Validation Patterns](#validation-patterns)
7. [Advanced Binding Scenarios](#advanced-binding-scenarios)
8. [Performance Optimization](#performance-optimization)

---

## 🎯 Learning Objectives

เมื่อจบบทเรียนนี้ คุณจะสามารถ:
- ✅ เข้าใจและใช้ Data Binding แบบต่างๆ
- ✅ สร้าง MVVM architecture ที่มีประสิทธิภาพ
- ✅ ใช้ ObservableObject และ ObservableProperty
- ✅ สร้างและใช้ Commands อย่างมีประสิทธิภาพ
- ✅ สร้าง Value Converters สำหรับ data transformation
- ✅ ใช้ Validation patterns ในการตรวจสอบข้อมูล
- ✅ Optimize performance ของ data binding

---

## 🔗 Data Binding Fundamentals

### Basic Data Binding

```xml
<!-- One-Way Binding (from source to target) -->
<Label Text="{Binding UserName}" />
<Label Text="{Binding FullName, StringFormat='Hello, {0}!'}" />

<!-- Two-Way Binding (bidirectional) -->
<Entry Text="{Binding Email, Mode=TwoWay}" />
<Switch IsToggled="{Binding IsEnabled, Mode=TwoWay}" />

<!-- One-Way to Source (from target to source) -->
<Entry Text="{Binding SearchQuery, Mode=OneWayToSource}" />

<!-- One-Time Binding (once only) -->
<Label Text="{Binding AppVersion, Mode=OneTime}" />
```

### Binding Context

```csharp
// Setting BindingContext in Code-Behind
public partial class MainPage : ContentPage
{
    public MainPage()
    {
        InitializeComponent();
        BindingContext = new MainPageViewModel();
    }
}

// Setting BindingContext in XAML
```

```xml
<ContentPage.BindingContext>
    <viewmodels:MainPageViewModel />
</ContentPage.BindingContext>
```

### Relative Binding

```xml
<!-- Binding to ancestor elements -->
<Grid x:Name="ParentGrid">
    <Grid.BindingContext>
        <viewmodels:ParentViewModel />
    </Grid.BindingContext>
    
    <StackLayout>
        <!-- Bind to parent Grid's BindingContext -->
        <Label Text="{Binding Source={x:Reference ParentGrid}, Path=BindingContext.ParentProperty}" />
        
        <!-- Bind to ancestor type -->
        <Button Command="{Binding Source={RelativeSource AncestorType={x:Type viewmodels:ParentViewModel}}, Path=ParentCommand}" />
        
        <!-- Self binding -->
        <Entry x:Name="SearchEntry" 
               Text="{Binding Source={x:Reference SearchEntry}, Path=Text, StringFormat='Searching for: {0}'}" />
    </StackLayout>
</Grid>
```

---

## 🏗️ MVVM Pattern Deep Dive

### MVVM Architecture

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│    View     │◄──►│  ViewModel  │◄──►│    Model    │
│   (XAML)    │    │ (Business   │    │   (Data)    │
│             │    │   Logic)    │    │             │
└─────────────┘    └─────────────┘    └─────────────┘
```

### Model Layer

```csharp
// Models/User.cs
namespace MyApp.Models;

public class User
{
    public int Id { get; set; }
    public string FirstName { get; set; } = string.Empty;
    public string LastName { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public DateTime DateOfBirth { get; set; }
    public bool IsActive { get; set; }
    public List<string> Tags { get; set; } = new();
    
    // Computed properties
    public string FullName => $"{FirstName} {LastName}";
    public int Age => DateTime.Now.Year - DateOfBirth.Year;
    public string InitialsInitials => $"{FirstName.FirstOrDefault()}{LastName.FirstOrDefault()}".ToUpper();
}

// Models/Product.cs
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public string ImageUrl { get; set; } = string.Empty;
    public string Category { get; set; } = string.Empty;
    public bool IsAvailable { get; set; } = true;
    public int Stock { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.Now;
}
```

### Service Layer

```csharp
// Services/IUserService.cs
namespace MyApp.Services;

public interface IUserService
{
    Task<List<User>> GetUsersAsync();
    Task<User?> GetUserByIdAsync(int id);
    Task<User> CreateUserAsync(User user);
    Task<User> UpdateUserAsync(User user);
    Task<bool> DeleteUserAsync(int id);
    Task<bool> ValidateEmailAsync(string email);
}

// Services/UserService.cs
public class UserService : IUserService
{
    private readonly HttpClient _httpClient;
    private readonly ILogger<UserService> _logger;
    
    public UserService(HttpClient httpClient, ILogger<UserService> logger)
    {
        _httpClient = httpClient;
        _logger = logger;
    }
    
    public async Task<List<User>> GetUsersAsync()
    {
        try
        {
            var response = await _httpClient.GetAsync("/api/users");
            response.EnsureSuccessStatusCode();
            
            var json = await response.Content.ReadAsStringAsync();
            return JsonSerializer.Deserialize<List<User>>(json) ?? new List<User>();
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error fetching users");
            return new List<User>();
        }
    }
    
    public async Task<User?> GetUserByIdAsync(int id)
    {
        try
        {
            var response = await _httpClient.GetAsync($"/api/users/{id}");
            if (!response.IsSuccessStatusCode) return null;
            
            var json = await response.Content.ReadAsStringAsync();
            return JsonSerializer.Deserialize<User>(json);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error fetching user {UserId}", id);
            return null;
        }
    }
    
    public async Task<bool> ValidateEmailAsync(string email)
    {
        if (string.IsNullOrWhiteSpace(email)) return false;
        
        try
        {
            var response = await _httpClient.GetAsync($"/api/users/validate-email?email={Uri.EscapeDataString(email)}");
            return response.IsSuccessStatusCode;
        }
        catch
        {
            return false;
        }
    }
}
```

---

## 📊 ObservableObject & Properties

### CommunityToolkit.Mvvm Setup

```csharp
// Install: CommunityToolkit.Mvvm
// ViewModels/BaseViewModel.cs
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;

namespace MyApp.ViewModels;

public partial class BaseViewModel : ObservableObject
{
    [ObservableProperty]
    private bool isBusy;
    
    [ObservableProperty]
    private string title = string.Empty;
    
    [ObservableProperty]
    private string errorMessage = string.Empty;
    
    [ObservableProperty]
    private bool hasError;
    
    // Loading state management
    protected async Task ExecuteAsync(Func<Task> operation, string? loadingMessage = null)
    {
        if (IsBusy) return;
        
        try
        {
            IsBusy = true;
            HasError = false;
            ErrorMessage = string.Empty;
            
            await operation();
        }
        catch (Exception ex)
        {
            HasError = true;
            ErrorMessage = ex.Message;
        }
        finally
        {
            IsBusy = false;
        }
    }
    
    protected async Task<T?> ExecuteAsync<T>(Func<Task<T>> operation)
    {
        if (IsBusy) return default(T);
        
        try
        {
            IsBusy = true;
            HasError = false;
            ErrorMessage = string.Empty;
            
            return await operation();
        }
        catch (Exception ex)
        {
            HasError = true;
            ErrorMessage = ex.Message;
            return default(T);
        }
        finally
        {
            IsBusy = false;
        }
    }
}
```

### Advanced ViewModel Example

```csharp
// ViewModels/UserListViewModel.cs
public partial class UserListViewModel : BaseViewModel
{
    private readonly IUserService _userService;
    private readonly INavigationService _navigationService;
    
    public UserListViewModel(IUserService userService, INavigationService navigationService)
    {
        _userService = userService;
        _navigationService = navigationService;
        Title = "Users";
    }
    
    [ObservableProperty]
    private ObservableCollection<UserViewModel> users = new();
    
    [ObservableProperty]
    private UserViewModel? selectedUser;
    
    [ObservableProperty]
    private string searchQuery = string.Empty;
    
    [ObservableProperty]
    private bool isRefreshing;
    
    [ObservableProperty]
    private int totalUsers;
    
    // Computed property using OnPropertyChanged
    public bool HasUsers => Users.Count > 0;
    
    // Property changed handler
    partial void OnSearchQueryChanged(string value)
    {
        // Debounce search
        _ = Task.Run(async () =>
        {
            await Task.Delay(300);
            if (SearchQuery == value) // Still the same query
            {
                await FilterUsersAsync();
            }
        });
    }
    
    partial void OnUsersChanged(ObservableCollection<UserViewModel> value)
    {
        OnPropertyChanged(nameof(HasUsers));
        TotalUsers = value.Count;
    }
    
    [RelayCommand]
    private async Task LoadUsersAsync()
    {
        await ExecuteAsync(async () =>
        {
            var users = await _userService.GetUsersAsync();
            var userViewModels = users.Select(u => new UserViewModel(u)).ToList();
            
            Users.Clear();
            foreach (var user in userViewModels)
            {
                Users.Add(user);
            }
        });
    }
    
    [RelayCommand]
    private async Task RefreshAsync()
    {
        IsRefreshing = true;
        try
        {
            await LoadUsersAsync();
        }
        finally
        {
            IsRefreshing = false;
        }
    }
    
    [RelayCommand]
    private async Task FilterUsersAsync()
    {
        if (string.IsNullOrWhiteSpace(SearchQuery))
        {
            await LoadUsersAsync();
            return;
        }
        
        await ExecuteAsync(async () =>
        {
            var allUsers = await _userService.GetUsersAsync();
            var filtered = allUsers
                .Where(u => u.FullName.Contains(SearchQuery, StringComparison.OrdinalIgnoreCase) ||
                           u.Email.Contains(SearchQuery, StringComparison.OrdinalIgnoreCase))
                .Select(u => new UserViewModel(u))
                .ToList();
            
            Users.Clear();
            foreach (var user in filtered)
            {
                Users.Add(user);
            }
        });
    }
    
    [RelayCommand]
    private async Task SelectUserAsync(UserViewModel user)
    {
        SelectedUser = user;
        await _navigationService.NavigateToAsync($"user-detail?id={user.Id}");
    }
    
    [RelayCommand]
    private async Task DeleteUserAsync(UserViewModel user)
    {
        var confirmed = await Application.Current?.MainPage?.DisplayAlert(
            "Confirm Delete",
            $"Are you sure you want to delete {user.FullName}?",
            "Yes", "No") ?? false;
        
        if (!confirmed) return;
        
        await ExecuteAsync(async () =>
        {
            var success = await _userService.DeleteUserAsync(user.Id);
            if (success)
            {
                Users.Remove(user);
            }
        });
    }
}

// ViewModels/UserViewModel.cs
public partial class UserViewModel : ObservableObject
{
    public UserViewModel(User user)
    {
        Id = user.Id;
        FirstName = user.FirstName;
        LastName = user.LastName;
        Email = user.Email;
        DateOfBirth = user.DateOfBirth;
        IsActive = user.IsActive;
    }
    
    public int Id { get; }
    
    [ObservableProperty]
    private string firstName = string.Empty;
    
    [ObservableProperty]
    private string lastName = string.Empty;
    
    [ObservableProperty]
    private string email = string.Empty;
    
    [ObservableProperty]
    private DateTime dateOfBirth = DateTime.Now.AddYears(-25);
    
    [ObservableProperty]
    private bool isActive = true;
    
    // Computed properties
    public string FullName => $"{FirstName} {LastName}";
    public string Initials => $"{FirstName.FirstOrDefault()}{LastName.FirstOrDefault()}".ToUpper();
    public int Age => DateTime.Now.Year - DateOfBirth.Year;
    public string AgeText => $"{Age} years old";
    
    // Update computed properties when names change
    partial void OnFirstNameChanged(string value) => OnPropertyChanged(nameof(FullName));
    partial void OnLastNameChanged(string value) => OnPropertyChanged(nameof(FullName));
    
    public User ToModel() => new()
    {
        Id = Id,
        FirstName = FirstName,
        LastName = LastName,
        Email = Email,
        DateOfBirth = DateOfBirth,
        IsActive = IsActive
    };
}
```

---

## ⚡ Commands & RelayCommand

### Basic Commands

```csharp
// Basic RelayCommand
[RelayCommand]
private void SimpleAction()
{
    // Simple action
    Debug.WriteLine("Button clicked!");
}

// Async RelayCommand
[RelayCommand]
private async Task LoadDataAsync()
{
    await Task.Delay(1000);
    // Load data logic
}

// Command with parameter
[RelayCommand]
private void DeleteItem(Product product)
{
    // Delete product logic
}

// Command with CanExecute
[RelayCommand(CanExecute = nameof(CanSaveUser))]
private async Task SaveUserAsync()
{
    // Save user logic
}

private bool CanSaveUser()
{
    return !string.IsNullOrWhiteSpace(FirstName) && 
           !string.IsNullOrWhiteSpace(Email) && 
           !IsBusy;
}
```

### Advanced Command Patterns

```csharp
// ViewModels/ProductFormViewModel.cs
public partial class ProductFormViewModel : BaseViewModel
{
    [ObservableProperty]
    private string name = string.Empty;
    
    [ObservableProperty]
    private string description = string.Empty;
    
    [ObservableProperty]
    private decimal price;
    
    [ObservableProperty]
    private bool isValid;
    
    // Command with complex CanExecute logic
    [RelayCommand(CanExecute = nameof(CanSaveProduct))]
    private async Task SaveProductAsync()
    {
        var product = new Product
        {
            Name = Name,
            Description = Description,
            Price = Price
        };
        
        await _productService.SaveProductAsync(product);
        await _navigationService.GoBackAsync();
    }
    
    private bool CanSaveProduct()
    {
        return !IsBusy && 
               !string.IsNullOrWhiteSpace(Name) && 
               !string.IsNullOrWhiteSpace(Description) && 
               Price > 0 && 
               IsValid;
    }
    
    // Update CanExecute when properties change
    partial void OnNameChanged(string value)
    {
        ValidateForm();
        SaveProductCommand.NotifyCanExecuteChanged();
    }
    
    partial void OnDescriptionChanged(string value)
    {
        ValidateForm();
        SaveProductCommand.NotifyCanExecuteChanged();
    }
    
    partial void OnPriceChanged(decimal value)
    {
        ValidateForm();
        SaveProductCommand.NotifyCanExecuteChanged();
    }
    
    private void ValidateForm()
    {
        IsValid = !string.IsNullOrWhiteSpace(Name) && 
                  !string.IsNullOrWhiteSpace(Description) && 
                  Price > 0;
    }
    
    // Command with confirmation
    [RelayCommand]
    private async Task DeleteProductAsync()
    {
        var confirmed = await Application.Current?.MainPage?.DisplayAlert(
            "Confirm Delete",
            "Are you sure you want to delete this product?",
            "Yes", "No") ?? false;
        
        if (confirmed)
        {
            await _productService.DeleteProductAsync(ProductId);
            await _navigationService.GoBackAsync();
        }
    }
    
    // Command with error handling
    [RelayCommand]
    private async Task LoadProductAsync(int productId)
    {
        try
        {
            IsBusy = true;
            var product = await _productService.GetProductByIdAsync(productId);
            
            if (product != null)
            {
                Name = product.Name;
                Description = product.Description;
                Price = product.Price;
            }
        }
        catch (Exception ex)
        {
            await Application.Current?.MainPage?.DisplayAlert("Error", ex.Message, "OK");
        }
        finally
        {
            IsBusy = false;
        }
    }
}
```

### Command Binding in XAML

```xml
<!-- Simple command binding -->
<Button Text="Save" Command="{Binding SaveCommand}" />

<!-- Command with parameter -->
<Button Text="Delete" 
        Command="{Binding DeleteCommand}" 
        CommandParameter="{Binding .}" />

<!-- Command binding in CollectionView -->
<CollectionView ItemsSource="{Binding Products}">
    <CollectionView.ItemTemplate>
        <DataTemplate x:DataType="models:Product">
            <Frame>
                <StackLayout>
                    <Label Text="{Binding Name}" />
                    <Button Text="Edit" 
                            Command="{Binding Source={RelativeSource AncestorType={x:Type viewmodels:ProductListViewModel}}, Path=EditProductCommand}"
                            CommandParameter="{Binding .}" />
                </StackLayout>
            </Frame>
        </DataTemplate>
    </CollectionView.ItemTemplate>
</CollectionView>

<!-- Command with loading state -->
<Button Text="{Binding IsBusy, Converter={StaticResource BoolToLoadingTextConverter}}"
        Command="{Binding LoadDataCommand}"
        IsEnabled="{Binding IsBusy, Converter={StaticResource InvertedBoolConverter}}" />
```

---

## 🔄 Value Converters

### Basic Converters

```csharp
// Converters/BoolToColorConverter.cs
using System.Globalization;

namespace MyApp.Converters;

public class BoolToColorConverter : IValueConverter
{
    public Color TrueColor { get; set; } = Colors.Green;
    public Color FalseColor { get; set; } = Colors.Red;
    
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if (value is bool boolValue)
        {
            return boolValue ? TrueColor : FalseColor;
        }
        return FalseColor;
    }
    
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        throw new NotImplementedException();
    }
}

// Converters/InvertedBoolConverter.cs
public class InvertedBoolConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        return value is bool boolValue ? !boolValue : false;
    }
    
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        return value is bool boolValue ? !boolValue : false;
    }
}

// Converters/StringToUpperConverter.cs
public class StringToUpperConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        return value?.ToString()?.ToUpper() ?? string.Empty;
    }
    
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        return value?.ToString() ?? string.Empty;
    }
}
```

### Advanced Converters

```csharp
// Converters/BytesToImageConverter.cs
public class BytesToImageConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if (value is byte[] bytes && bytes.Length > 0)
        {
            return ImageSource.FromStream(() => new MemoryStream(bytes));
        }
        return null;
    }
    
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        throw new NotImplementedException();
    }
}

// Converters/ItemsCountToVisibilityConverter.cs
public class ItemsCountToVisibilityConverter : IValueConverter
{
    public bool InvertVisibility { get; set; } = false;
    
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        var hasItems = false;
        
        if (value is int count)
        {
            hasItems = count > 0;
        }
        else if (value is ICollection collection)
        {
            hasItems = collection.Count > 0;
        }
        else if (value is IEnumerable enumerable)
        {
            hasItems = enumerable.Cast<object>().Any();
        }
        
        return InvertVisibility ? !hasItems : hasItems;
    }
    
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        throw new NotImplementedException();
    }
}

// Converters/PriceToFormattedStringConverter.cs
public class PriceToFormattedStringConverter : IValueConverter
{
    public string CurrencySymbol { get; set; } = "$";
    public int DecimalPlaces { get; set; } = 2;
    
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if (value is decimal price)
        {
            return $"{CurrencySymbol}{price:F2}";
        }
        else if (value is double doublePrice)
        {
            return $"{CurrencySymbol}{doublePrice:F2}";
        }
        else if (decimal.TryParse(value?.ToString(), out var parsedPrice))
        {
            return $"{CurrencySymbol}{parsedPrice:F2}";
        }
        
        return $"{CurrencySymbol}0.00";
    }
    
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        var str = value?.ToString()?.Replace(CurrencySymbol, "").Trim();
        return decimal.TryParse(str, out var result) ? result : 0m;
    }
}
```

### Multi-Value Converters

```csharp
// Converters/FullNameConverter.cs
public class FullNameConverter : IMultiValueConverter
{
    public object Convert(object[] values, Type targetType, object parameter, CultureInfo culture)
    {
        if (values?.Length >= 2)
        {
            var firstName = values[0]?.ToString() ?? "";
            var lastName = values[1]?.ToString() ?? "";
            return $"{firstName} {lastName}".Trim();
        }
        return string.Empty;
    }
    
    public object[] ConvertBack(object value, Type[] targetTypes, object parameter, CultureInfo culture)
    {
        var fullName = value?.ToString() ?? "";
        var parts = fullName.Split(' ', 2);
        return new object[] 
        { 
            parts.Length > 0 ? parts[0] : "", 
            parts.Length > 1 ? parts[1] : "" 
        };
    }
}

// Converters/BooleanAndConverter.cs
public class BooleanAndConverter : IMultiValueConverter
{
    public object Convert(object[] values, Type targetType, object parameter, CultureInfo culture)
    {
        return values?.All(v => v is bool b && b) ?? false;
    }
    
    public object[] ConvertBack(object value, Type[] targetTypes, object parameter, CultureInfo culture)
    {
        throw new NotImplementedException();
    }
}
```

### Converter Usage in XAML

```xml
<!-- Define converters in resources -->
<ContentPage.Resources>
    <ResourceDictionary>
        <converters:BoolToColorConverter x:Key="BoolToColorConverter" 
                                       TrueColor="Green" 
                                       FalseColor="Red" />
        <converters:InvertedBoolConverter x:Key="InvertedBoolConverter" />
        <converters:ItemsCountToVisibilityConverter x:Key="ItemsCountToVisibilityConverter" />
        <converters:PriceToFormattedStringConverter x:Key="PriceConverter" CurrencySymbol="$" />
        <converters:FullNameConverter x:Key="FullNameConverter" />
    </ResourceDictionary>
</ContentPage.Resources>

<!-- Using converters -->
<StackLayout>
    <!-- Bool to Color -->
    <Label Text="Status" 
           TextColor="{Binding IsActive, Converter={StaticResource BoolToColorConverter}}" />
    
    <!-- Inverted Bool -->
    <ActivityIndicator IsVisible="{Binding IsBusy}" 
                       IsRunning="{Binding IsBusy}" />
    <Button Text="Load" 
            IsEnabled="{Binding IsBusy, Converter={StaticResource InvertedBoolConverter}}" />
    
    <!-- Items count to visibility -->
    <Label Text="No items found" 
           IsVisible="{Binding Items.Count, Converter={StaticResource ItemsCountToVisibilityConverter}, ConverterParameter=Invert}" />
    
    <!-- Price formatting -->
    <Label Text="{Binding Price, Converter={StaticResource PriceConverter}}" />
    
    <!-- Multi-value converter -->
    <Label>
        <Label.Text>
            <MultiBinding Converter="{StaticResource FullNameConverter}">
                <Binding Path="FirstName" />
                <Binding Path="LastName" />
            </MultiBinding>
        </Label.Text>
    </Label>
</StackLayout>
```

---

## ✅ Validation Patterns

### FluentValidation Integration

```csharp
// Install: FluentValidation
// ViewModels/UserFormViewModel.cs
using FluentValidation;

public partial class UserFormViewModel : BaseViewModel
{
    private readonly IValidator<UserFormViewModel> _validator;
    
    public UserFormViewModel(IValidator<UserFormViewModel> validator)
    {
        _validator = validator;
    }
    
    [ObservableProperty]
    private string firstName = string.Empty;
    
    [ObservableProperty]
    private string lastName = string.Empty;
    
    [ObservableProperty]
    private string email = string.Empty;
    
    [ObservableProperty]
    private DateTime dateOfBirth = DateTime.Now.AddYears(-25);
    
    [ObservableProperty]
    private Dictionary<string, List<string>> errors = new();
    
    [ObservableProperty]
    private bool isValid = false;
    
    // Property change handlers with validation
    partial void OnFirstNameChanged(string value) => ValidateProperty(nameof(FirstName));
    partial void OnLastNameChanged(string value) => ValidateProperty(nameof(LastName));
    partial void OnEmailChanged(string value) => ValidateProperty(nameof(Email));
    partial void OnDateOfBirthChanged(DateTime value) => ValidateProperty(nameof(DateOfBirth));
    
    private void ValidateProperty(string propertyName)
    {
        var result = _validator.Validate(this);
        
        // Clear existing errors for this property
        if (Errors.ContainsKey(propertyName))
        {
            Errors.Remove(propertyName);
        }
        
        // Add new errors for this property
        var propertyErrors = result.Errors
            .Where(e => e.PropertyName == propertyName)
            .Select(e => e.ErrorMessage)
            .ToList();
        
        if (propertyErrors.Any())
        {
            Errors[propertyName] = propertyErrors;
        }
        
        IsValid = !result.Errors.Any();
        OnPropertyChanged(nameof(Errors));
        
        // Update command can execute
        SaveUserCommand.NotifyCanExecuteChanged();
    }
    
    [RelayCommand(CanExecute = nameof(CanSaveUser))]
    private async Task SaveUserAsync()
    {
        // Full validation before save
        var result = _validator.Validate(this);
        if (!result.IsValid)
        {
            // Display validation errors
            var errorMessage = string.Join("\n", result.Errors.Select(e => e.ErrorMessage));
            await Application.Current?.MainPage?.DisplayAlert("Validation Error", errorMessage, "OK");
            return;
        }
        
        // Save user logic
        var user = new User
        {
            FirstName = FirstName,
            LastName = LastName,
            Email = Email,
            DateOfBirth = DateOfBirth
        };
        
        await _userService.SaveUserAsync(user);
    }
    
    private bool CanSaveUser() => IsValid && !IsBusy;
    
    public string GetFirstErrorForProperty(string propertyName)
    {
        return Errors.TryGetValue(propertyName, out var errors) ? errors.FirstOrDefault() : string.Empty;
    }
    
    public bool HasErrorForProperty(string propertyName)
    {
        return Errors.ContainsKey(propertyName) && Errors[propertyName].Any();
    }
}

// Validators/UserFormViewModelValidator.cs
public class UserFormViewModelValidator : AbstractValidator<UserFormViewModel>
{
    public UserFormViewModelValidator()
    {
        RuleFor(x => x.FirstName)
            .NotEmpty().WithMessage("First name is required")
            .Length(2, 50).WithMessage("First name must be between 2 and 50 characters");
        
        RuleFor(x => x.LastName)
            .NotEmpty().WithMessage("Last name is required")
            .Length(2, 50).WithMessage("Last name must be between 2 and 50 characters");
        
        RuleFor(x => x.Email)
            .NotEmpty().WithMessage("Email is required")
            .EmailAddress().WithMessage("Please enter a valid email address");
        
        RuleFor(x => x.DateOfBirth)
            .LessThan(DateTime.Now.AddYears(-13)).WithMessage("Must be at least 13 years old")
            .GreaterThan(DateTime.Now.AddYears(-120)).WithMessage("Please enter a valid birth date");
    }
}
```

### Validation in XAML

```xml
<!-- User form with validation -->
<ScrollView>
    <StackLayout Spacing="16" Padding="20">
        <!-- First Name -->
        <StackLayout Spacing="4">
            <Label Text="First Name" Style="{StaticResource FormLabel}" />
            <Entry Text="{Binding FirstName, Mode=TwoWay}" 
                   Placeholder="Enter first name" />
            <Label Text="{Binding FirstNameError}" 
                   TextColor="{StaticResource Danger}" 
                   FontSize="12"
                   IsVisible="{Binding HasFirstNameError}" />
        </StackLayout>
        
        <!-- Last Name -->
        <StackLayout Spacing="4">
            <Label Text="Last Name" Style="{StaticResource FormLabel}" />
            <Entry Text="{Binding LastName, Mode=TwoWay}" 
                   Placeholder="Enter last name" />
            <Label Text="{Binding LastNameError}" 
                   TextColor="{StaticResource Danger}" 
                   FontSize="12"
                   IsVisible="{Binding HasLastNameError}" />
        </StackLayout>
        
        <!-- Email -->
        <StackLayout Spacing="4">
            <Label Text="Email" Style="{StaticResource FormLabel}" />
            <Entry Text="{Binding Email, Mode=TwoWay}" 
                   Placeholder="Enter email address" 
                   Keyboard="Email" />
            <Label Text="{Binding EmailError}" 
                   TextColor="{StaticResource Danger}" 
                   FontSize="12"
                   IsVisible="{Binding HasEmailError}" />
        </StackLayout>
        
        <!-- Date of Birth -->
        <StackLayout Spacing="4">
            <Label Text="Date of Birth" Style="{StaticResource FormLabel}" />
            <DatePicker Date="{Binding DateOfBirth, Mode=TwoWay}" />
            <Label Text="{Binding DateOfBirthError}" 
                   TextColor="{StaticResource Danger}" 
                   FontSize="12"
                   IsVisible="{Binding HasDateOfBirthError}" />
        </StackLayout>
        
        <!-- Submit Button -->
        <Button Text="Save User" 
                Command="{Binding SaveUserCommand}" 
                Style="{StaticResource PrimaryButton}" 
                IsEnabled="{Binding IsValid}" />
    </StackLayout>
</ScrollView>
```

---

## 🚀 Performance Optimization

### Binding Performance Tips

```csharp
// Use ObservableProperty instead of manual PropertyChanged
[ObservableProperty]
private string name = string.Empty;

// Instead of:
private string _name = string.Empty;
public string Name
{
    get => _name;
    set
    {
        if (_name != value)
        {
            _name = value;
            OnPropertyChanged();
        }
    }
}

// Optimize collections with ObservableCollection
public ObservableCollection<UserViewModel> Users { get; } = new();

// Batch updates for better performance
public void UpdateUsers(List<User> users)
{
    Users.Clear();
    foreach (var user in users)
    {
        Users.Add(new UserViewModel(user));
    }
}

// Use computed properties sparingly
public string FullName => $"{FirstName} {LastName}";

// For expensive computations, cache the result
private string? _expensiveComputation;
public string ExpensiveComputation
{
    get
    {
        if (_expensiveComputation == null)
        {
            _expensiveComputation = DoExpensiveCalculation();
        }
        return _expensiveComputation;
    }
}

private void InvalidateExpensiveComputation()
{
    _expensiveComputation = null;
    OnPropertyChanged(nameof(ExpensiveComputation));
}
```

### CollectionView Optimization

```xml
<!-- Optimize CollectionView performance -->
<CollectionView ItemsSource="{Binding Items}"
                ItemsUpdatingScrollMode="KeepItemsInView"
                RemainingItemsThreshold="10"
                RemainingItemsThresholdReachedCommand="{Binding LoadMoreCommand}">
    
    <!-- Use DataTemplateSelector for different item types -->
    <CollectionView.ItemTemplate>
        <DataTemplate>
            <Grid>
                <!-- Minimize the view hierarchy -->
                <Label Text="{Binding Name}" />
                <Label Text="{Binding Description}" Grid.Row="1" />
            </Grid>
        </DataTemplate>
    </CollectionView.ItemTemplate>
</CollectionView>

<!-- Use virtualization for large lists -->
<CollectionView ItemsSource="{Binding LargeItemsList}"
                CachingStrategy="RecycleElement">
    <!-- Item template -->
</CollectionView>
```

---

## 🎯 Exercises

### Exercise 1: User Profile Form
สร้าง user profile form ที่มี:
- Data binding สำหรับ form fields
- FluentValidation สำหรับ validation
- Commands สำหรับ save/cancel
- Value converters สำหรับ formatting

### Exercise 2: Product Catalog
สร้าง product catalog ที่มี:
- ObservableCollection สำหรับ products
- Search functionality with data binding
- Price formatting converters
- Add to cart commands

### Exercise 3: Settings Page
สร้าง settings page ที่มี:
- Two-way binding สำหรับ preferences
- Bool to visibility converters
- Command patterns สำหรับ actions
- Validation for settings values

---

## 📝 Quiz

1. **MVVM Pattern**: ViewModel ไม่ควรมี reference ไปยัง View เพราะอะไร?
2. **ObservableProperty**: ข้อดีของ `[ObservableProperty]` เมื่อเทียบกับการ implement PropertyChanged manually?
3. **Value Converters**: เมื่อไหร่ควรใช้ IValueConverter vs IMultiValueConverter?

<details>
<summary>เฉลย</summary>

1. เพื่อให้ ViewModel สามารถ unit test ได้ และใช้ร่วมกับ View หลายๆ แบบได้
2. ลดโค้ด boilerplate, ลดความผิดพลาด, และ performance ดีกว่า
3. IValueConverter สำหรับ transform ค่าเดียว, IMultiValueConverter สำหรับ combine หลายค่า

</details>

---

## 🎉 สรุป

ในบทเรียนนี้ คุณได้เรียนรู้:

✅ **Data Binding** - One-way, Two-way, และ advanced binding scenarios  
✅ **MVVM Architecture** - Model, View, ViewModel separation  
✅ **ObservableObject** - Modern property change notification  
✅ **Commands** - RelayCommand patterns และ advanced scenarios  
✅ **Value Converters** - Data transformation และ formatting  
✅ **Validation** - FluentValidation integration  
✅ **Performance** - Optimization techniques

### Next Steps
🔜 **บทที่ 7**: Styling & Theming - การจัดรูปแบบและธีมขั้นสูง  
🔜 **บทที่ 8**: Navigation Patterns - การนำทางแบบต่างๆ

---

**Happy Coding! 🚀**

> บทเรียนต่อไป: [07-styling-and-theming.md](07-styling-and-theming.md)
