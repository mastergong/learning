# 🧭 บทที่ 8: Navigation Patterns
> **เชี่ยวชาญการนำทางและ Navigation Patterns**

---

## 📋 สารบัญ
1. [Navigation Fundamentals](#navigation-fundamentals)
2. [Shell Navigation](#shell-navigation)
3. [Page-based Navigation](#page-based-navigation)
4. [Modal Navigation](#modal-navigation)
5. [Tabbed Navigation](#tabbed-navigation)
6. [Master-Detail Navigation](#master-detail-navigation)
7. [Deep Linking](#deep-linking)
8. [Navigation State Management](#navigation-state-management)

---

## 🎯 Learning Objectives

เมื่อจบบทเรียนนี้ คุณจะสามารถ:
- ✅ เข้าใจ Navigation patterns ต่างๆ ใน MAUI
- ✅ ใช้ Shell Navigation อย่างมีประสิทธิภาพ
- ✅ สร้าง complex navigation flows
- ✅ จัดการ Navigation state และ parameters
- ✅ ใช้ Deep linking และ URL-based navigation
- ✅ สร้าง responsive navigation patterns
- ✅ Optimize navigation performance

---

## 🧭 Navigation Fundamentals

### Navigation Types in MAUI

```csharp
// 1. Shell Navigation (Recommended)
await Shell.Current.GoToAsync("//main/details");

// 2. NavigationPage Stack
await Navigation.PushAsync(new DetailsPage());

// 3. Modal Navigation
await Navigation.PushModalAsync(new ModalPage());

// 4. Application MainPage
Application.Current.MainPage = new AppShell();
```

### Navigation Service Interface

```csharp
// Services/INavigationService.cs
namespace MyApp.Services;

public interface INavigationService
{
    // Shell Navigation
    Task GoToAsync(string route, bool animate = true);
    Task GoToAsync(string route, Dictionary<string, object> parameters, bool animate = true);
    Task GoBackAsync(bool animate = true);
    
    // Modal Navigation
    Task PushModalAsync<T>(bool animate = true) where T : Page, new();
    Task PushModalAsync<T>(Dictionary<string, object> parameters, bool animate = true) where T : Page, new();
    Task PopModalAsync(bool animate = true);
    
    // Page Navigation
    Task PushAsync<T>(bool animate = true) where T : Page, new();
    Task PushAsync<T>(Dictionary<string, object> parameters, bool animate = true) where T : Page, new();
    Task PopAsync(bool animate = true);
    Task PopToRootAsync(bool animate = true);
    
    // Navigation State
    bool CanGoBack { get; }
    string CurrentRoute { get; }
    int NavigationStackCount { get; }
}

// Services/NavigationService.cs
public class NavigationService : INavigationService
{
    public async Task GoToAsync(string route, bool animate = true)
    {
        await Shell.Current.GoToAsync(route, animate);
    }
    
    public async Task GoToAsync(string route, Dictionary<string, object> parameters, bool animate = true)
    {
        await Shell.Current.GoToAsync(route, animate, parameters);
    }
    
    public async Task GoBackAsync(bool animate = true)
    {
        await Shell.Current.GoToAsync("..", animate);
    }
    
    public async Task PushModalAsync<T>(bool animate = true) where T : Page, new()
    {
        var page = new T();
        await Application.Current.MainPage.Navigation.PushModalAsync(page, animate);
    }
    
    public async Task PushModalAsync<T>(Dictionary<string, object> parameters, bool animate = true) where T : Page, new()
    {
        var page = new T();
        
        // Pass parameters if page implements IQueryPropertyBag or has QueryProperty attributes
        if (page is IQueryAttributable queryPage)
        {
            queryPage.ApplyQueryAttributes(parameters);
        }
        
        await Application.Current.MainPage.Navigation.PushModalAsync(page, animate);
    }
    
    public async Task PopModalAsync(bool animate = true)
    {
        await Application.Current.MainPage.Navigation.PopModalAsync(animate);
    }
    
    public async Task PushAsync<T>(bool animate = true) where T : Page, new()
    {
        var page = new T();
        await Shell.Current.Navigation.PushAsync(page, animate);
    }
    
    public async Task PushAsync<T>(Dictionary<string, object> parameters, bool animate = true) where T : Page, new()
    {
        var page = new T();
        
        if (page is IQueryAttributable queryPage)
        {
            queryPage.ApplyQueryAttributes(parameters);
        }
        
        await Shell.Current.Navigation.PushAsync(page, animate);
    }
    
    public async Task PopAsync(bool animate = true)
    {
        await Shell.Current.Navigation.PopAsync(animate);
    }
    
    public async Task PopToRootAsync(bool animate = true)
    {
        await Shell.Current.Navigation.PopToRootAsync(animate);
    }
    
    public bool CanGoBack => Shell.Current.Navigation.NavigationStack.Count > 1;
    
    public string CurrentRoute => Shell.Current.CurrentState.Location.ToString();
    
    public int NavigationStackCount => Shell.Current.Navigation.NavigationStack.Count;
}
```

---

## 🐚 Shell Navigation

### Basic Shell Structure

```xml
<!-- AppShell.xaml -->
<?xml version="1.0" encoding="UTF-8" ?>
<Shell
    x:Class="MyApp.AppShell"
    xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
    xmlns:local="clr-namespace:MyApp"
    Title="MyApp"
    FlyoutBehavior="Flyout">

    <!-- Shell Resources for Styling -->
    <Shell.Resources>
        <ResourceDictionary>
            <Style x:Key="BaseStyle" TargetType="Element">
                <Setter Property="Shell.BackgroundColor" Value="{AppThemeBinding Light={StaticResource White}, Dark={StaticResource Black}}" />
                <Setter Property="Shell.ForegroundColor" Value="{AppThemeBinding Light={StaticResource Primary}, Dark={StaticResource White}}" />
                <Setter Property="Shell.TitleColor" Value="{AppThemeBinding Light={StaticResource Primary}, Dark={StaticResource White}}" />
            </Style>
            <Style TargetType="TabBar" BasedOn="{StaticResource BaseStyle}" />
            <Style TargetType="FlyoutItem" BasedOn="{StaticResource BaseStyle}" />
        </ResourceDictionary>
    </Shell.Resources>

    <!-- Flyout Header -->
    <Shell.FlyoutHeader>
        <Grid BackgroundColor="{StaticResource Primary}" HeightRequest="200">
            <Grid.RowDefinitions>
                <RowDefinition Height="*" />
                <RowDefinition Height="Auto" />
            </Grid.RowDefinitions>
            
            <Image Grid.Row="0" 
                   Source="flyout_background.png" 
                   Aspect="AspectFill" />
                   
            <StackLayout Grid.Row="1" 
                         Orientation="Horizontal" 
                         Padding="20,10">
                <Frame CornerRadius="30" 
                       WidthRequest="60" 
                       HeightRequest="60" 
                       IsClippedToBounds="True">
                    <Image Source="user_avatar.png" Aspect="AspectFill" />
                </Frame>
                
                <StackLayout VerticalOptions="Center" Margin="15,0">
                    <Label Text="{Binding UserName}" 
                           TextColor="White" 
                           FontSize="16" 
                           FontAttributes="Bold" />
                    <Label Text="{Binding UserEmail}" 
                           TextColor="White" 
                           FontSize="14" 
                           Opacity="0.8" />
                </StackLayout>
            </StackLayout>
        </Grid>
    </Shell.FlyoutHeader>

    <!-- Flyout Footer -->
    <Shell.FlyoutFooter>
        <StackLayout Padding="20">
            <Button Text="Settings" 
                    Style="{StaticResource SecondaryButton}"
                    Command="{Binding NavigateToSettingsCommand}" />
            <Button Text="Sign Out" 
                    Style="{StaticResource OutlineButton}"
                    Command="{Binding SignOutCommand}" />
        </StackLayout>
    </Shell.FlyoutFooter>

    <!-- Main Navigation Structure -->
    
    <!-- Tab Bar Navigation -->
    <TabBar>
        <!-- Home Tab -->
        <ShellContent
            Title="Home"
            Icon="home.png"
            Route="home"
            ContentTemplate="{DataTemplate local:HomePage}" />
        
        <!-- Explore Tab -->
        <ShellContent
            Title="Explore"
            Icon="explore.png"
            Route="explore"
            ContentTemplate="{DataTemplate local:ExplorePage}" />
        
        <!-- Profile Tab -->
        <ShellContent
            Title="Profile"
            Icon="profile.png"
            Route="profile"
            ContentTemplate="{DataTemplate local:ProfilePage}" />
    </TabBar>

    <!-- Flyout Items -->
    <FlyoutItem Title="Dashboard" 
                Icon="dashboard.png"
                Route="dashboard">
        <ShellContent ContentTemplate="{DataTemplate local:DashboardPage}" />
    </FlyoutItem>
    
    <FlyoutItem Title="Products" 
                Icon="products.png"
                Route="products">
        <ShellContent ContentTemplate="{DataTemplate local:ProductsPage}" />
    </FlyoutItem>
    
    <FlyoutItem Title="Orders" 
                Icon="orders.png"
                Route="orders">
        <ShellContent ContentTemplate="{DataTemplate local:OrdersPage}" />
    </FlyoutItem>
    
    <!-- Separators and Menu Items -->
    <MenuItem Text="Help" 
              IconImageSource="help.png"
              Command="{Binding HelpCommand}" />
    
    <MenuItem Text="About" 
              IconImageSource="info.png"
              Command="{Binding AboutCommand}" />

</Shell>
```

### Shell Navigation Code-Behind

```csharp
// AppShell.xaml.cs
namespace MyApp;

public partial class AppShell : Shell
{
    public AppShell()
    {
        InitializeComponent();
        
        // Register routes for navigation
        RegisterRoutes();
        
        // Set up binding context
        BindingContext = new AppShellViewModel();
    }
    
    private void RegisterRoutes()
    {
        // Register pages for navigation
        Routing.RegisterRoute("product-detail", typeof(ProductDetailPage));
        Routing.RegisterRoute("user-profile", typeof(UserProfilePage));
        Routing.RegisterRoute("settings", typeof(SettingsPage));
        Routing.RegisterRoute("order-detail", typeof(OrderDetailPage));
        Routing.RegisterRoute("add-product", typeof(AddProductPage));
        Routing.RegisterRoute("edit-product", typeof(EditProductPage));
        
        // Nested routes
        Routing.RegisterRoute("products/categories", typeof(CategoriesPage));
        Routing.RegisterRoute("products/categories/detail", typeof(CategoryDetailPage));
    }
}

// ViewModels/AppShellViewModel.cs
public partial class AppShellViewModel : BaseViewModel
{
    private readonly IAuthService _authService;
    private readonly INavigationService _navigationService;
    
    public AppShellViewModel(IAuthService authService, INavigationService navigationService)
    {
        _authService = authService;
        _navigationService = navigationService;
    }
    
    [ObservableProperty]
    private string userName = "John Doe";
    
    [ObservableProperty]
    private string userEmail = "john@example.com";
    
    [RelayCommand]
    private async Task NavigateToSettings()
    {
        await _navigationService.GoToAsync("settings");
    }
    
    [RelayCommand]
    private async Task SignOut()
    {
        var confirmed = await Application.Current.MainPage.DisplayAlert(
            "Sign Out", 
            "Are you sure you want to sign out?", 
            "Yes", "No");
        
        if (confirmed)
        {
            await _authService.SignOutAsync();
            Application.Current.MainPage = new LoginPage();
        }
    }
    
    [RelayCommand]
    private async Task Help()
    {
        await Browser.OpenAsync("https://docs.microsoft.com/dotnet/maui/");
    }
    
    [RelayCommand]
    private async Task About()
    {
        await _navigationService.PushModalAsync<AboutPage>();
    }
}
```

### Advanced Shell Navigation

```csharp
// Advanced navigation patterns
public class AdvancedNavigationExamples
{
    private readonly INavigationService _navigation;
    
    public AdvancedNavigationExamples(INavigationService navigation)
    {
        _navigation = navigation;
    }
    
    // Absolute navigation (from root)
    public async Task NavigateToProductDetail(int productId)
    {
        await _navigation.GoToAsync($"//products/product-detail?id={productId}");
    }
    
    // Relative navigation
    public async Task NavigateToEditProduct(int productId)
    {
        await _navigation.GoToAsync($"edit-product?id={productId}");
    }
    
    // Navigation with complex parameters
    public async Task NavigateWithParameters()
    {
        var parameters = new Dictionary<string, object>
        {
            ["product"] = new Product { Id = 1, Name = "Sample Product" },
            ["mode"] = "edit",
            ["returnTo"] = "products"
        };
        
        await _navigation.GoToAsync("product-detail", parameters);
    }
    
    // Conditional navigation
    public async Task ConditionalNavigation(bool isAuthenticated)
    {
        if (isAuthenticated)
        {
            await _navigation.GoToAsync("//main/dashboard");
        }
        else
        {
            await _navigation.GoToAsync("//login");
        }
    }
    
    // Navigation with animation control
    public async Task NavigateWithoutAnimation()
    {
        await _navigation.GoToAsync("settings", animate: false);
    }
    
    // Back navigation
    public async Task GoBack()
    {
        if (_navigation.CanGoBack)
        {
            await _navigation.GoBackAsync();
        }
        else
        {
            // Handle case where can't go back (e.g., go to home)
            await _navigation.GoToAsync("//home");
        }
    }
}
```

---

## 📄 Page-based Navigation

### NavigationPage Stack

```csharp
// Traditional NavigationPage approach
public class PageNavigationExamples
{
    // Set up NavigationPage
    public void SetupNavigationPage()
    {
        var mainPage = new NavigationPage(new HomePage())
        {
            BarBackgroundColor = Colors.Primary,
            BarTextColor = Colors.White
        };
        
        Application.Current.MainPage = mainPage;
    }
    
    // Push page to stack
    public async Task PushPage()
    {
        var detailPage = new ProductDetailPage();
        await Navigation.PushAsync(detailPage);
    }
    
    // Push with parameters
    public async Task PushPageWithData(Product product)
    {
        var detailPage = new ProductDetailPage();
        detailPage.BindingContext = new ProductDetailViewModel(product);
        await Navigation.PushAsync(detailPage);
    }
    
    // Pop page from stack
    public async Task PopPage()
    {
        if (Navigation.NavigationStack.Count > 1)
        {
            await Navigation.PopAsync();
        }
    }
    
    // Pop to root
    public async Task PopToRoot()
    {
        await Navigation.PopToRootAsync();
    }
    
    // Insert page into stack
    public void InsertPage()
    {
        var intermediatePage = new IntermediatePage();
        Navigation.InsertPageBefore(intermediatePage, Navigation.NavigationStack.Last());
    }
    
    // Remove page from stack
    public void RemovePage()
    {
        var pageToRemove = Navigation.NavigationStack[Navigation.NavigationStack.Count - 2];
        Navigation.RemovePage(pageToRemove);
    }
}
```

### Custom Navigation Animations

```csharp
// Custom page transitions
public class CustomNavigationPage : NavigationPage
{
    public CustomNavigationPage(Page root) : base(root)
    {
        // Custom push animation
        this.Pushed += OnPagePushed;
        this.Popped += OnPagePopped;
    }
    
    private async void OnPagePushed(object sender, NavigationEventArgs e)
    {
        var page = e.Page;
        page.Opacity = 0;
        page.Scale = 0.8;
        
        await Task.WhenAll(
            page.FadeTo(1, 300),
            page.ScaleTo(1, 300, Easing.CubicOut)
        );
    }
    
    private async void OnPagePopped(object sender, NavigationEventArgs e)
    {
        var page = e.Page;
        
        await Task.WhenAll(
            page.FadeTo(0, 200),
            page.ScaleTo(0.8, 200, Easing.CubicIn)
        );
    }
}

// Page with custom transitions
public partial class AnimatedPage : ContentPage
{
    public AnimatedPage()
    {
        InitializeComponent();
    }
    
    protected override async void OnAppearing()
    {
        base.OnAppearing();
        
        // Slide in from right
        await this.TranslateTo(Width, 0, 0);
        await this.TranslateTo(0, 0, 300, Easing.CubicOut);
    }
    
    public async Task SlideOut()
    {
        // Slide out to left
        await this.TranslateTo(-Width, 0, 300, Easing.CubicIn);
    }
}
```

---

## 📝 Modal Navigation

### Modal Patterns

```csharp
// Modal navigation service
public class ModalNavigationService
{
    // Basic modal
    public async Task ShowModal<T>() where T : Page, new()
    {
        var page = new T();
        await Application.Current.MainPage.Navigation.PushModalAsync(page);
    }
    
    // Modal with parameters
    public async Task ShowModal<T>(Dictionary<string, object> parameters) where T : Page, new()
    {
        var page = new T();
        
        if (page is IQueryAttributable queryPage)
        {
            queryPage.ApplyQueryAttributes(parameters);
        }
        
        await Application.Current.MainPage.Navigation.PushModalAsync(page);
    }
    
    // Modal with result
    public async Task<T> ShowModalForResult<TPage, T>() 
        where TPage : Page, IModalWithResult<T>, new()
    {
        var page = new TPage();
        var tcs = new TaskCompletionSource<T>();
        
        page.ModalResult += (sender, result) => tcs.SetResult(result);
        
        await Application.Current.MainPage.Navigation.PushModalAsync(page);
        
        return await tcs.Task;
    }
    
    // Dismiss modal
    public async Task DismissModal()
    {
        await Application.Current.MainPage.Navigation.PopModalAsync();
    }
}

// Modal with result interface
public interface IModalWithResult<T>
{
    event EventHandler<T> ModalResult;
}

// Example modal page
public partial class EditProductModal : ContentPage, IModalWithResult<Product>
{
    public event EventHandler<Product> ModalResult;
    
    public EditProductModal()
    {
        InitializeComponent();
    }
    
    private async void OnSaveClicked(object sender, EventArgs e)
    {
        var product = GetEditedProduct();
        ModalResult?.Invoke(this, product);
        await Navigation.PopModalAsync();
    }
    
    private async void OnCancelClicked(object sender, EventArgs e)
    {
        ModalResult?.Invoke(this, null);
        await Navigation.PopModalAsync();
    }
    
    private Product GetEditedProduct()
    {
        // Return edited product
        return new Product(); // Simplified
    }
}
```

### Modal XAML Examples

```xml
<!-- Modal Page with Header -->
<ContentPage x:Class="MyApp.Modals.EditProductModal"
             Title="Edit Product">
    
    <!-- Navigation Bar -->
    <ContentPage.ToolbarItems>
        <ToolbarItem Text="Cancel" 
                     Command="{Binding CancelCommand}" 
                     Order="Primary" 
                     Priority="0" />
        <ToolbarItem Text="Save" 
                     Command="{Binding SaveCommand}" 
                     Order="Primary" 
                     Priority="1" />
    </ContentPage.ToolbarItems>
    
    <ScrollView>
        <StackLayout Padding="20" Spacing="16">
            
            <!-- Form content -->
            <Entry Placeholder="Product Name" 
                   Text="{Binding ProductName}" />
            
            <Editor Placeholder="Description" 
                    Text="{Binding Description}" 
                    HeightRequest="100" />
            
            <Entry Placeholder="Price" 
                   Text="{Binding Price}" 
                   Keyboard="Numeric" />
            
            <!-- Action buttons for mobile -->
            <StackLayout Orientation="Horizontal" 
                         HorizontalOptions="End"
                         Spacing="8"
                         IsVisible="{OnPlatform Default=False, Phone=True}">
                <Button Text="Cancel" 
                        Style="{StaticResource SecondaryButton}"
                        Command="{Binding CancelCommand}" />
                <Button Text="Save" 
                        Style="{StaticResource PrimaryButton}"
                        Command="{Binding SaveCommand}" />
            </StackLayout>
            
        </StackLayout>
    </ScrollView>
    
</ContentPage>

<!-- Full Screen Modal -->
<ContentPage x:Class="MyApp.Modals.ImageViewerModal"
             BackgroundColor="Black">
    
    <Grid>
        <!-- Close button -->
        <Button Text="✕" 
                BackgroundColor="Transparent"
                TextColor="White"
                FontSize="24"
                WidthRequest="48"
                HeightRequest="48"
                HorizontalOptions="End"
                VerticalOptions="Start"
                Margin="16"
                Command="{Binding CloseCommand}" />
        
        <!-- Image -->
        <Image Source="{Binding ImageSource}" 
               Aspect="AspectFit"
               VerticalOptions="Center"
               HorizontalOptions="Center" />
        
        <!-- Image info -->
        <StackLayout VerticalOptions="End" 
                     BackgroundColor="#80000000"
                     Padding="16">
            <Label Text="{Binding ImageTitle}" 
                   TextColor="White" 
                   FontSize="18" 
                   FontAttributes="Bold" />
            <Label Text="{Binding ImageDescription}" 
                   TextColor="White" 
                   FontSize="14" />
        </StackLayout>
    </Grid>
    
</ContentPage>
```

---

## 📑 Tabbed Navigation

### TabbedPage Implementation

```xml
<!-- TabbedPage XAML -->
<TabbedPage x:Class="MyApp.MainTabbedPage"
            xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
            xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
            xmlns:local="clr-namespace:MyApp"
            Title="Main">
    
    <!-- Home Tab -->
    <NavigationPage Title="Home" IconImageSource="home.png">
        <x:Arguments>
            <local:HomePage />
        </x:Arguments>
    </NavigationPage>
    
    <!-- Search Tab -->
    <NavigationPage Title="Search" IconImageSource="search.png">
        <x:Arguments>
            <local:SearchPage />
        </x:Arguments>
    </NavigationPage>
    
    <!-- Notifications Tab with Badge -->
    <NavigationPage Title="Notifications" IconImageSource="notifications.png">
        <x:Arguments>
            <local:NotificationsPage />
        </x:Arguments>
    </NavigationPage>
    
    <!-- Profile Tab -->
    <NavigationPage Title="Profile" IconImageSource="profile.png">
        <x:Arguments>
            <local:ProfilePage />
        </x:Arguments>
    </NavigationPage>
    
</TabbedPage>
```

### Custom Tab Bar

```xml
<!-- Custom Tab Bar using Shell -->
<Shell x:Class="MyApp.CustomTabShell">
    
    <Shell.Resources>
        <Style TargetType="TabBar">
            <Setter Property="Shell.TabBarBackgroundColor" Value="White" />
            <Setter Property="Shell.TabBarTitleColor" Value="Gray" />
            <Setter Property="Shell.TabBarUnselectedColor" Value="LightGray" />
        </Style>
    </Shell.Resources>
    
    <TabBar>
        <!-- Custom tab with notification badge -->
        <Tab Title="Home" Icon="home.png">
            <ShellContent ContentTemplate="{DataTemplate local:HomePage}" />
        </Tab>
        
        <Tab Title="Search" Icon="search.png">
            <ShellContent ContentTemplate="{DataTemplate local:SearchPage}" />
        </Tab>
        
        <Tab Title="Notifications" Icon="notifications.png">
            <ShellContent ContentTemplate="{DataTemplate local:NotificationsPage}">
                <!-- Custom tab content with badge -->
                <ShellContent.Icon>
                    <FontImageSource Glyph="🔔" 
                                     FontFamily="Segoe UI Emoji" 
                                     Color="Gray" />
                </ShellContent.Icon>
            </ShellContent>
        </Tab>
        
        <Tab Title="Profile" Icon="profile.png">
            <ShellContent ContentTemplate="{DataTemplate local:ProfilePage}" />
        </Tab>
    </TabBar>
    
</Shell>
```

### Dynamic Tab Management

```csharp
// Dynamic tab management
public class DynamicTabManager
{
    public void AddTab(string title, string icon, Type pageType)
    {
        var tabBar = (TabBar)Shell.Current.Items.FirstOrDefault(i => i is TabBar);
        
        if (tabBar != null)
        {
            var newTab = new Tab
            {
                Title = title,
                Icon = icon
            };
            
            var shellContent = new ShellContent
            {
                ContentTemplate = new DataTemplate(pageType)
            };
            
            newTab.Items.Add(shellContent);
            tabBar.Items.Add(newTab);
        }
    }
    
    public void RemoveTab(string title)
    {
        var tabBar = (TabBar)Shell.Current.Items.FirstOrDefault(i => i is TabBar);
        var tabToRemove = tabBar?.Items.FirstOrDefault(t => t.Title == title);
        
        if (tabToRemove != null)
        {
            tabBar.Items.Remove(tabToRemove);
        }
    }
    
    public void UpdateTabBadge(string tabTitle, int badgeCount)
    {
        // Implementation for tab badges
        // This would typically involve custom renderers or handlers
    }
}
```

---

## 📱 Master-Detail Navigation

### Flyout Navigation

```xml
<!-- FlyoutPage XAML -->
<FlyoutPage x:Class="MyApp.MainFlyoutPage"
            xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
            xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
            xmlns:local="clr-namespace:MyApp">
    
    <!-- Flyout (Master) -->
    <FlyoutPage.Flyout>
        <ContentPage Title="Menu">
            <StackLayout>
                <!-- Flyout Header -->
                <Grid BackgroundColor="{StaticResource Primary}" HeightRequest="200">
                    <StackLayout VerticalOptions="Center" Padding="20">
                        <Image Source="user_avatar.png" 
                               WidthRequest="80" 
                               HeightRequest="80" 
                               Aspect="AspectFill" />
                        <Label Text="John Doe" 
                               TextColor="White" 
                               FontSize="18" 
                               FontAttributes="Bold" />
                        <Label Text="john@example.com" 
                               TextColor="White" 
                               FontSize="14" />
                    </StackLayout>
                </Grid>
                
                <!-- Menu Items -->
                <CollectionView ItemsSource="{Binding MenuItems}">
                    <CollectionView.ItemTemplate>
                        <DataTemplate>
                            <Grid Padding="16,12" ColumnDefinitions="Auto,*">
                                <Grid.GestureRecognizers>
                                    <TapGestureRecognizer Command="{Binding NavigateCommand}" />
                                </Grid.GestureRecognizers>
                                
                                <Image Grid.Column="0" 
                                       Source="{Binding Icon}" 
                                       WidthRequest="24" 
                                       HeightRequest="24" />
                                <Label Grid.Column="1" 
                                       Text="{Binding Title}" 
                                       VerticalOptions="Center" 
                                       Margin="16,0,0,0" />
                            </Grid>
                        </DataTemplate>
                    </CollectionView.ItemTemplate>
                </CollectionView>
            </StackLayout>
        </ContentPage>
    </FlyoutPage.Flyout>
    
    <!-- Detail -->
    <FlyoutPage.Detail>
        <NavigationPage>
            <x:Arguments>
                <local:HomePage />
            </x:Arguments>
        </NavigationPage>
    </FlyoutPage.Detail>
    
</FlyoutPage>
```

### Responsive Master-Detail

```csharp
// Responsive master-detail behavior
public class ResponsiveFlyoutPage : FlyoutPage
{
    public ResponsiveFlyoutPage()
    {
        InitializeComponent();
        
        // Set flyout behavior based on device
        SetFlyoutBehavior();
        
        // Handle orientation changes
        DeviceDisplay.MainDisplayInfoChanged += OnDisplayInfoChanged;
    }
    
    private void SetFlyoutBehavior()
    {
        var width = DeviceDisplay.MainDisplayInfo.Width;
        var density = DeviceDisplay.MainDisplayInfo.Density;
        var widthInDp = width / density;
        
        // Desktop/Tablet: Show flyout as split view
        if (widthInDp >= 1024)
        {
            FlyoutLayoutBehavior = FlyoutLayoutBehavior.Split;
        }
        // Mobile: Show flyout as overlay
        else
        {
            FlyoutLayoutBehavior = FlyoutLayoutBehavior.Popover;
        }
    }
    
    private void OnDisplayInfoChanged(object sender, DisplayInfoChangedEventArgs e)
    {
        SetFlyoutBehavior();
    }
    
    protected override void OnDisappearing()
    {
        DeviceDisplay.MainDisplayInfoChanged -= OnDisplayInfoChanged;
        base.OnDisappearing();
    }
}
```

---

## 🔗 Deep Linking

### URL Routing Setup

```csharp
// AppShell route registration
public partial class AppShell : Shell
{
    public AppShell()
    {
        InitializeComponent();
        RegisterRoutes();
    }
    
    private void RegisterRoutes()
    {
        // Basic routes
        Routing.RegisterRoute("product", typeof(ProductDetailPage));
        Routing.RegisterRoute("user", typeof(UserProfilePage));
        Routing.RegisterRoute("settings", typeof(SettingsPage));
        
        // Nested routes
        Routing.RegisterRoute("products/category", typeof(CategoryPage));
        Routing.RegisterRoute("products/category/item", typeof(ProductDetailPage));
        
        // Parameterized routes
        Routing.RegisterRoute("product/{id}", typeof(ProductDetailPage));
        Routing.RegisterRoute("user/{userId}/profile", typeof(UserProfilePage));
        
        // Complex routes
        Routing.RegisterRoute("shop/{category}/{subcategory}/{productId}", typeof(ProductDetailPage));
    }
}

// Query parameters handling
[QueryProperty(nameof(ProductId), "id")]
[QueryProperty(nameof(Category), "category")]
[QueryProperty(nameof(Mode), "mode")]
public partial class ProductDetailPage : ContentPage, IQueryAttributable
{
    public string ProductId { get; set; }
    public string Category { get; set; }
    public string Mode { get; set; }
    
    public ProductDetailPage()
    {
        InitializeComponent();
    }
    
    public void ApplyQueryAttributes(IDictionary<string, object> query)
    {
        // Handle complex objects passed as parameters
        if (query.TryGetValue("product", out var productObj) && productObj is Product product)
        {
            BindingContext = new ProductDetailViewModel(product);
        }
        
        if (query.TryGetValue("highlightSection", out var section))
        {
            HighlightSection(section.ToString());
        }
    }
    
    private void HighlightSection(string section)
    {
        // Highlight specific section based on deep link
    }
}
```

### Custom URL Handling

```csharp
// App.xaml.cs - URL handling
public partial class App : Application
{
    public App()
    {
        InitializeComponent();
        MainPage = new AppShell();
    }
    
    protected override async void OnStart()
    {
        // Handle app launch with URL
        var url = await GetLaunchUrl();
        if (!string.IsNullOrEmpty(url))
        {
            await HandleDeepLink(url);
        }
    }
    
    protected override async void OnAppLinkRequestReceived(Uri uri)
    {
        base.OnAppLinkRequestReceived(uri);
        await HandleDeepLink(uri.ToString());
    }
    
    private async Task HandleDeepLink(string url)
    {
        try
        {
            var uri = new Uri(url);
            var route = ParseRoute(uri);
            var parameters = ParseParameters(uri);
            
            if (!string.IsNullOrEmpty(route))
            {
                await Shell.Current.GoToAsync(route, parameters);
            }
        }
        catch (Exception ex)
        {
            // Handle invalid URLs gracefully
            await Shell.Current.DisplayAlert("Error", "Invalid link", "OK");
        }
    }
    
    private string ParseRoute(Uri uri)
    {
        // Parse URI to Shell route
        var segments = uri.Segments.Where(s => s != "/").Select(s => s.TrimEnd('/')).ToArray();
        
        return segments.Length switch
        {
            1 when segments[0] == "product" => "product",
            2 when segments[0] == "product" => $"product?id={segments[1]}",
            3 when segments[0] == "user" && segments[2] == "profile" => $"user?userId={segments[1]}",
            _ => string.Empty
        };
    }
    
    private Dictionary<string, object> ParseParameters(Uri uri)
    {
        var parameters = new Dictionary<string, object>();
        
        // Parse query string
        var queryParams = Microsoft.AspNetCore.WebUtilities.QueryHelpers.ParseQuery(uri.Query);
        foreach (var param in queryParams)
        {
            parameters[param.Key] = param.Value.ToString();
        }
        
        return parameters;
    }
    
    private async Task<string> GetLaunchUrl()
    {
        // Platform-specific implementation to get launch URL
        return string.Empty;
    }
}

// Deep link service
public interface IDeepLinkService
{
    Task<bool> CanHandle(string url);
    Task HandleDeepLink(string url);
    string GenerateDeepLink(string route, Dictionary<string, object> parameters = null);
}

public class DeepLinkService : IDeepLinkService
{
    private readonly INavigationService _navigationService;
    
    public DeepLinkService(INavigationService navigationService)
    {
        _navigationService = navigationService;
    }
    
    public async Task<bool> CanHandle(string url)
    {
        try
        {
            var uri = new Uri(url);
            return uri.Host == "myapp.com" || uri.Scheme == "myapp";
        }
        catch
        {
            return false;
        }
    }
    
    public async Task HandleDeepLink(string url)
    {
        var uri = new Uri(url);
        
        // Route handling logic
        var path = uri.AbsolutePath.TrimStart('/');
        var segments = path.Split('/');
        
        switch (segments[0])
        {
            case "product":
                await HandleProductLink(segments, uri.Query);
                break;
            case "user":
                await HandleUserLink(segments, uri.Query);
                break;
            case "share":
                await HandleShareLink(segments, uri.Query);
                break;
            default:
                await _navigationService.GoToAsync("//home");
                break;
        }
    }
    
    private async Task HandleProductLink(string[] segments, string query)
    {
        if (segments.Length > 1)
        {
            var productId = segments[1];
            await _navigationService.GoToAsync($"product?id={productId}{query}");
        }
    }
    
    private async Task HandleUserLink(string[] segments, string query)
    {
        if (segments.Length > 1)
        {
            var userId = segments[1];
            await _navigationService.GoToAsync($"user?userId={userId}{query}");
        }
    }
    
    private async Task HandleShareLink(string[] segments, string query)
    {
        // Handle shared content
        await _navigationService.GoToAsync($"share{query}");
    }
    
    public string GenerateDeepLink(string route, Dictionary<string, object> parameters = null)
    {
        var baseUrl = "https://myapp.com";
        var url = $"{baseUrl}/{route}";
        
        if (parameters?.Any() == true)
        {
            var queryString = string.Join("&", 
                parameters.Select(kvp => $"{kvp.Key}={Uri.EscapeDataString(kvp.Value?.ToString() ?? "")}"));
            url += $"?{queryString}";
        }
        
        return url;
    }
}
```

---

## 🗂️ Navigation State Management

### Navigation State Service

```csharp
// Navigation state management
public interface INavigationStateService
{
    NavigationState CurrentState { get; }
    event EventHandler<NavigationState> StateChanged;
    
    void SaveState(string key, object value);
    T GetState<T>(string key, T defaultValue = default);
    void ClearState();
    
    void SaveNavigationStack();
    Task RestoreNavigationStack();
}

public class NavigationState
{
    public string CurrentRoute { get; set; }
    public Dictionary<string, object> Parameters { get; set; } = new();
    public List<string> NavigationStack { get; set; } = new();
    public Dictionary<string, object> PageStates { get; set; } = new();
}

public class NavigationStateService : INavigationStateService
{
    private readonly IPreferencesService _preferences;
    private NavigationState _currentState;
    
    public NavigationStateService(IPreferencesService preferences)
    {
        _preferences = preferences;
        _currentState = new NavigationState();
    }
    
    public NavigationState CurrentState => _currentState;
    
    public event EventHandler<NavigationState> StateChanged;
    
    public void SaveState(string key, object value)
    {
        _currentState.PageStates[key] = value;
        StateChanged?.Invoke(this, _currentState);
    }
    
    public T GetState<T>(string key, T defaultValue = default)
    {
        if (_currentState.PageStates.TryGetValue(key, out var value) && value is T typedValue)
        {
            return typedValue;
        }
        return defaultValue;
    }
    
    public void ClearState()
    {
        _currentState.PageStates.Clear();
        StateChanged?.Invoke(this, _currentState);
    }
    
    public void SaveNavigationStack()
    {
        var stack = Shell.Current.Navigation.NavigationStack
            .Select(page => page.GetType().Name)
            .ToList();
        
        _currentState.NavigationStack = stack;
        _currentState.CurrentRoute = Shell.Current.CurrentState.Location.ToString();
        
        var stateJson = JsonSerializer.Serialize(_currentState);
        _preferences.Set("navigation_state", stateJson);
    }
    
    public async Task RestoreNavigationStack()
    {
        var stateJson = _preferences.Get("navigation_state", string.Empty);
        if (string.IsNullOrEmpty(stateJson)) return;
        
        try
        {
            var state = JsonSerializer.Deserialize<NavigationState>(stateJson);
            if (state != null)
            {
                _currentState = state;
                
                // Restore navigation stack if needed
                if (!string.IsNullOrEmpty(state.CurrentRoute))
                {
                    await Shell.Current.GoToAsync(state.CurrentRoute);
                }
                
                StateChanged?.Invoke(this, _currentState);
            }
        }
        catch (Exception ex)
        {
            // Handle deserialization errors
            Debug.WriteLine($"Failed to restore navigation state: {ex.Message}");
        }
    }
}
```

### Page State Persistence

```csharp
// Base page with state management
public abstract class StatefulPage : ContentPage
{
    protected INavigationStateService NavigationState { get; }
    
    protected StatefulPage(INavigationStateService navigationState)
    {
        NavigationState = navigationState;
    }
    
    protected override void OnAppearing()
    {
        base.OnAppearing();
        RestorePageState();
    }
    
    protected override void OnDisappearing()
    {
        base.OnDisappearing();
        SavePageState();
    }
    
    protected virtual void RestorePageState()
    {
        // Override in derived classes
    }
    
    protected virtual void SavePageState()
    {
        // Override in derived classes
    }
    
    protected void SaveState(string key, object value)
    {
        var pageKey = $"{GetType().Name}_{key}";
        NavigationState.SaveState(pageKey, value);
    }
    
    protected T GetState<T>(string key, T defaultValue = default)
    {
        var pageKey = $"{GetType().Name}_{key}";
        return NavigationState.GetState(pageKey, defaultValue);
    }
}

// Example stateful page
public partial class ProductListPage : StatefulPage
{
    public ProductListPage(INavigationStateService navigationState) : base(navigationState)
    {
        InitializeComponent();
    }
    
    protected override void RestorePageState()
    {
        // Restore scroll position
        var scrollPosition = GetState("ScrollPosition", 0.0);
        MainScrollView.ScrollToAsync(0, scrollPosition, false);
        
        // Restore search query
        var searchQuery = GetState("SearchQuery", string.Empty);
        SearchEntry.Text = searchQuery;
        
        // Restore selected filters
        var selectedFilters = GetState<List<string>>("SelectedFilters", new List<string>());
        ApplyFilters(selectedFilters);
    }
    
    protected override void SavePageState()
    {
        // Save scroll position
        SaveState("ScrollPosition", MainScrollView.ScrollY);
        
        // Save search query
        SaveState("SearchQuery", SearchEntry.Text);
        
        // Save selected filters
        SaveState("SelectedFilters", GetSelectedFilters());
    }
    
    private void ApplyFilters(List<string> filters)
    {
        // Apply filters to the view
    }
    
    private List<string> GetSelectedFilters()
    {
        // Get currently selected filters
        return new List<string>();
    }
}
```

---

## 🎯 Exercises

### Exercise 1: Complete Navigation System
สร้าง navigation system ที่มี:
- Shell navigation พร้อม flyout และ tabs
- Modal navigation สำหรับ forms
- Deep linking support
- Navigation state persistence

### Exercise 2: Advanced Routing
สร้าง advanced routing system:
- Parameterized routes
- Conditional navigation based on user roles
- Custom navigation animations
- Breadcrumb navigation

### Exercise 3: Responsive Navigation
สร้าง responsive navigation:
- Different patterns สำหรับ phone/tablet/desktop
- Adaptive flyout behavior
- Context-aware navigation
- Navigation analytics

---

## 📝 Quiz

1. **Shell vs NavigationPage**: ข้อดีของ Shell Navigation เมื่อเทียบกับ NavigationPage?
2. **Deep Linking**: วิธีการ handle complex deep links พร้อม parameters?
3. **Modal Navigation**: เมื่อไหร่ควรใช้ Modal vs Page navigation?

<details>
<summary>เฉลย</summary>

1. Shell ให้ unified navigation, better performance, built-in tab/flyout support, และ URL-based routing
2. ใช้ QueryProperty attributes, IQueryAttributable interface, และ route parameter parsing
3. Modal สำหรับ temporary tasks, forms, confirmations; Page navigation สำหรับ main app flow

</details>

---

## 🎉 สรุป

ในบทเรียนนี้ คุณได้เรียนรู้:

✅ **Navigation Fundamentals** - Types และ patterns ต่างๆ  
✅ **Shell Navigation** - Modern navigation approach  
✅ **Page Navigation** - Traditional stack-based navigation  
✅ **Modal Patterns** - Temporary UI และ forms  
✅ **Tabbed Navigation** - Tab-based organization  
✅ **Master-Detail** - Responsive flyout patterns  
✅ **Deep Linking** - URL-based navigation  
✅ **State Management** - Navigation state persistence

### Next Steps
🔜 **บทที่ 9**: Advanced Layouts - Complex UI patterns และ responsive design  
🔜 **บทที่ 10**: Custom Controls - สร้าง custom controls และ behaviors

---

**Happy Navigating! 🧭**

> บทเรียนต่อไป: [09-advanced-layouts.md](09-advanced-layouts.md)
