# 📱 Chapter 12: Platform-Specific Features

> **หัวข้อ**: Native Platform Integration & Platform-Specific UI  
> **ระยะเวลา**: 3.5 ชั่วโมง  
> **ความยากง่าย**: ระดับกลาง-สูง  
> **Prerequisites**: บทที่ 1-11

## 🎯 Learning Objectives

หลังจากเรียนบทนี้แล้ว คุณจะสามารถ:

- ✅ ใช้ platform-specific features ใน iOS, Android, Windows
- ✅ สร้าง adaptive UI ตาม platform design guidelines
- ✅ เข้าถึง native hardware features
- ✅ ปรับแต่ง performance ตาม platform
- ✅ ใช้ platform handlers และ custom renderers

---

## 📱 Chapter Overview

### 🏗️ What We'll Build
สร้าง **"Cross-Platform Feature Demo App"** ที่มี:
- 📷 Camera และ photo capture
- 📍 Location services และ maps
- 📳 Push notifications
- 🔐 Biometric authentication
- 💾 Platform-specific storage

---

## 1. 🎨 Platform-Specific UI Patterns

### 1.1 Material Design vs Cupertino vs Fluent Design

```xml
<!-- d:\A-Work\learn from ai\maui-xaml-mastery\src\12-PlatformFeatures\Styles\PlatformSpecificStyles.xaml -->
<ResourceDictionary xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
                   xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">

    <!-- Android Material Design Styles -->
    <Style x:Key="MaterialButton" TargetType="Button">
        <Setter Property="BackgroundColor" Value="#2196F3" />
        <Setter Property="TextColor" Value="White" />
        <Setter Property="CornerRadius" Value="4" />
        <Setter Property="FontSize" Value="14" />
        <Setter Property="FontAttributes" Value="Bold" />
        <Setter Property="Padding" Value="16,12" />
        <Setter Property="TextTransform" Value="Uppercase" />
    </Style>

    <!-- iOS Cupertino Styles -->
    <Style x:Key="CupertinoButton" TargetType="Button">
        <Setter Property="BackgroundColor" Value="#007AFF" />
        <Setter Property="TextColor" Value="White" />
        <Setter Property="CornerRadius" Value="8" />
        <Setter Property="FontSize" Value="17" />
        <Setter Property="FontAttributes" Value="None" />
        <Setter Property="Padding" Value="20,12" />
        <Setter Property="TextTransform" Value="None" />
    </Style>

    <!-- Windows Fluent Design Styles -->
    <Style x:Key="FluentButton" TargetType="Button">
        <Setter Property="BackgroundColor" Value="#0078D4" />
        <Setter Property="TextColor" Value="White" />
        <Setter Property="CornerRadius" Value="2" />
        <Setter Property="FontSize" Value="14" />
        <Setter Property="FontAttributes" Value="None" />
        <Setter Property="Padding" Value="32,8" />
        <Setter Property="TextTransform" Value="None" />
    </Style>

    <!-- Platform-Specific Button Style -->
    <Style x:Key="PlatformButton" TargetType="Button">
        <Setter Property="Style">
            <Setter.Value>
                <OnPlatform x:TypeArguments="Style">
                    <On Platform="Android" Value="{StaticResource MaterialButton}" />
                    <On Platform="iOS" Value="{StaticResource CupertinoButton}" />
                    <On Platform="WinUI" Value="{StaticResource FluentButton}" />
                </OnPlatform>
            </Setter.Value>
        </Setter>
    </Style>

    <!-- Navigation Bar Styles -->
    <Style x:Key="MaterialNavigationPage" TargetType="NavigationPage">
        <Setter Property="BarBackgroundColor" Value="#2196F3" />
        <Setter Property="BarTextColor" Value="White" />
    </Style>

    <Style x:Key="CupertinoNavigationPage" TargetType="NavigationPage">
        <Setter Property="BarBackgroundColor" Value="#F8F8F8" />
        <Setter Property="BarTextColor" Value="#007AFF" />
    </Style>

    <Style x:Key="FluentNavigationPage" TargetType="NavigationPage">
        <Setter Property="BarBackgroundColor" Value="#F3F2F1" />
        <Setter Property="BarTextColor" Value="#323130" />
    </Style>

    <!-- Platform-Specific Navigation -->
    <Style x:Key="PlatformNavigationPage" TargetType="NavigationPage">
        <Setter Property="Style">
            <Setter.Value>
                <OnPlatform x:TypeArguments="Style">
                    <On Platform="Android" Value="{StaticResource MaterialNavigationPage}" />
                    <On Platform="iOS" Value="{StaticResource CupertinoNavigationPage}" />
                    <On Platform="WinUI" Value="{StaticResource FluentNavigationPage}" />
                </OnPlatform>
            </Setter.Value>
        </Setter>
    </Style>

</ResourceDictionary>
```

### 1.2 Adaptive Layout Components

```xml
<!-- d:\A-Work\learn from ai\maui-xaml-mastery\src\12-PlatformFeatures\Views\AdaptiveLayoutPage.xaml -->
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="PlatformFeatures.Views.AdaptiveLayoutPage"
             Title="Platform Adaptive UI">

    <ContentPage.Resources>
        <ResourceDictionary>
            <!-- Android FAB Style -->
            <Style x:Key="AndroidFAB" TargetType="Button">
                <Setter Property="WidthRequest" Value="56" />
                <Setter Property="HeightRequest" Value="56" />
                <Setter Property="CornerRadius" Value="28" />
                <Setter Property="BackgroundColor" Value="#FF4081" />
                <Setter Property="TextColor" Value="White" />
                <Setter Property="FontSize" Value="24" />
                <Setter Property="HorizontalOptions" Value="End" />
                <Setter Property="VerticalOptions" Value="End" />
                <Setter Property="Margin" Value="0,0,16,16" />
            </Style>

            <!-- iOS Tab Bar Style -->
            <Style x:Key="iOSTabBar" TargetType="Grid">
                <Setter Property="BackgroundColor" Value="#F8F8F8" />
                <Setter Property="HeightRequest" Value="83" />
                <Setter Property="VerticalOptions" Value="End" />
            </Style>
        </ResourceDictionary>
    </ContentPage.Resources>

    <Grid>
        <!-- Main Content -->
        <ScrollView>
            <StackLayout Padding="20" Spacing="20">

                <!-- Platform-Specific Header -->
                <Frame>
                    <StackLayout>
                        <Label Text="Platform Detection" 
                               FontSize="18" 
                               FontAttributes="Bold" />
                        
                        <Label x:Name="PlatformInfoLabel" 
                               FontSize="14" />
                    </StackLayout>
                </Frame>

                <!-- Platform-Specific Buttons -->
                <StackLayout>
                    <Label Text="Platform-Specific Buttons" 
                           FontSize="16" 
                           FontAttributes="Bold" />
                    
                    <Button Text="Primary Action" 
                            Style="{StaticResource PlatformButton}"
                            Clicked="OnPrimaryActionClicked" />
                    
                    <!-- Android-specific button -->
                    <Button Text="Material Design Button" 
                            Style="{StaticResource MaterialButton}"
                            IsVisible="{OnPlatform Android=True, Default=False}" />
                    
                    <!-- iOS-specific button -->
                    <Button Text="Cupertino Button" 
                            Style="{StaticResource CupertinoButton}"
                            IsVisible="{OnPlatform iOS=True, Default=False}" />
                    
                    <!-- Windows-specific button -->
                    <Button Text="Fluent Design Button" 
                            Style="{StaticResource FluentButton}"
                            IsVisible="{OnPlatform WinUI=True, Default=False}" />
                </StackLayout>

                <!-- Platform-Specific Lists -->
                <StackLayout>
                    <Label Text="Platform-Specific List Styles" 
                           FontSize="16" 
                           FontAttributes="Bold" />
                    
                    <CollectionView x:Name="ItemsCollectionView" 
                                   HeightRequest="200">
                        <CollectionView.ItemTemplate>
                            <DataTemplate>
                                <!-- Android Material List Item -->
                                <Grid IsVisible="{OnPlatform Android=True, Default=False}"
                                      Padding="16,12"
                                      ColumnDefinitions="Auto, *, Auto">
                                    
                                    <Ellipse Grid.Column="0"
                                            WidthRequest="40"
                                            HeightRequest="40"
                                            Fill="#2196F3" />
                                    
                                    <StackLayout Grid.Column="1" 
                                                Margin="16,0,0,0"
                                                VerticalOptions="Center">
                                        <Label Text="{Binding Title}" 
                                               FontSize="16" />
                                        <Label Text="{Binding Subtitle}" 
                                               FontSize="14" 
                                               TextColor="Gray" />
                                    </StackLayout>
                                    
                                    <Label Grid.Column="2"
                                           Text="chevron_right"
                                           FontFamily="MaterialIcons"
                                           VerticalOptions="Center" />
                                </Grid>

                                <!-- iOS Cupertino List Item -->
                                <Grid IsVisible="{OnPlatform iOS=True, Default=False}"
                                      Padding="15,11"
                                      ColumnDefinitions="Auto, *, Auto"
                                      BackgroundColor="White">
                                    
                                    <Image Grid.Column="0"
                                           Source="{Binding Icon}"
                                           WidthRequest="29"
                                           HeightRequest="29" />
                                    
                                    <Label Grid.Column="1"
                                           Text="{Binding Title}"
                                           FontSize="17"
                                           Margin="15,0,0,0"
                                           VerticalOptions="Center" />
                                    
                                    <Label Grid.Column="2"
                                           Text="›"
                                           FontSize="18"
                                           TextColor="#C7C7CC"
                                           VerticalOptions="Center" />
                                </Grid>

                                <!-- Windows Fluent List Item -->
                                <Grid IsVisible="{OnPlatform WinUI=True, Default=False}"
                                      Padding="12,8"
                                      ColumnDefinitions="Auto, *, Auto">
                                    
                                    <Rectangle Grid.Column="0"
                                              WidthRequest="32"
                                              HeightRequest="32"
                                              Fill="#0078D4" />
                                    
                                    <StackLayout Grid.Column="1" 
                                                Margin="12,0,0,0"
                                                VerticalOptions="Center">
                                        <Label Text="{Binding Title}" 
                                               FontSize="14" />
                                        <Label Text="{Binding Subtitle}" 
                                               FontSize="12" 
                                               TextColor="#605E5C" />
                                    </StackLayout>
                                </Grid>
                            </DataTemplate>
                        </CollectionView.ItemTemplate>
                    </CollectionView>
                </StackLayout>

                <!-- Platform-Specific Input Controls -->
                <StackLayout>
                    <Label Text="Platform-Specific Input Controls" 
                           FontSize="16" 
                           FontAttributes="Bold" />
                    
                    <!-- Android Material Entry -->
                    <Frame IsVisible="{OnPlatform Android=True, Default=False}"
                           Padding="0"
                           BorderColor="#2196F3"
                           BackgroundColor="Transparent">
                        <Entry Placeholder="Material Design Entry"
                               BackgroundColor="Transparent" />
                    </Frame>
                    
                    <!-- iOS Cupertino Entry -->
                    <Frame IsVisible="{OnPlatform iOS=True, Default=False}"
                           Padding="8"
                           CornerRadius="8"
                           BorderColor="#C7C7CC"
                           BackgroundColor="White">
                        <Entry Placeholder="Cupertino Entry"
                               BackgroundColor="Transparent" />
                    </Frame>
                    
                    <!-- Windows Fluent Entry -->
                    <Frame IsVisible="{OnPlatform WinUI=True, Default=False}"
                           Padding="8"
                           CornerRadius="2"
                           BorderColor="#8A8886"
                           BackgroundColor="White">
                        <Entry Placeholder="Fluent Design Entry"
                               BackgroundColor="Transparent" />
                    </Frame>
                </StackLayout>

            </StackLayout>
        </ScrollView>

        <!-- Android FAB -->
        <Button Text="+"
                Style="{StaticResource AndroidFAB}"
                IsVisible="{OnPlatform Android=True, Default=False}"
                Clicked="OnFABClicked" />

        <!-- iOS Tab Bar -->
        <Grid Style="{StaticResource iOSTabBar}"
              IsVisible="{OnPlatform iOS=True, Default=False}"
              ColumnDefinitions="*, *, *, *">
            
            <Button Grid.Column="0" 
                    Text="Home" 
                    BackgroundColor="Transparent"
                    TextColor="#007AFF" />
            <Button Grid.Column="1" 
                    Text="Search" 
                    BackgroundColor="Transparent"
                    TextColor="Gray" />
            <Button Grid.Column="2" 
                    Text="Settings" 
                    BackgroundColor="Transparent"
                    TextColor="Gray" />
            <Button Grid.Column="3" 
                    Text="Profile" 
                    BackgroundColor="Transparent"
                    TextColor="Gray" />
        </Grid>

    </Grid>

</ContentPage>
```

---

## 2. 📷 Camera & Media Features

### 2.1 Camera Service Implementation

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\12-PlatformFeatures\Services\ICameraService.cs
namespace PlatformFeatures.Services;

public interface ICameraService
{
    Task<bool> IsCameraAvailableAsync();
    Task<FileResult> TakePhotoAsync(CameraOptions options = null);
    Task<FileResult> PickPhotoAsync();
    Task<List<FileResult>> PickMultiplePhotosAsync();
}

public class CameraOptions
{
    public int Quality { get; set; } = 80; // 0-100
    public PhotoSize PhotoSize { get; set; } = PhotoSize.Medium;
    public bool AllowCropping { get; set; } = false;
}

public enum PhotoSize
{
    Small,
    Medium,
    Large,
    Custom
}
```

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\12-PlatformFeatures\Services\CameraService.cs
using Microsoft.Maui.Media;

namespace PlatformFeatures.Services;

public class CameraService : ICameraService
{
    public async Task<bool> IsCameraAvailableAsync()
    {
        try
        {
            return MediaPicker.Default.IsCaptureSupported;
        }
        catch
        {
            return false;
        }
    }

    public async Task<FileResult> TakePhotoAsync(CameraOptions options = null)
    {
        try
        {
            var photo = await MediaPicker.Default.CapturePhotoAsync(new MediaPickerOptions
            {
                Title = "Take a photo"
            });

            if (photo != null && options != null)
            {
                return await ProcessPhotoAsync(photo, options);
            }

            return photo;
        }
        catch (Exception ex)
        {
            // Log exception
            System.Diagnostics.Debug.WriteLine($"Camera error: {ex.Message}");
            return null;
        }
    }

    public async Task<FileResult> PickPhotoAsync()
    {
        try
        {
            var photo = await MediaPicker.Default.PickPhotoAsync(new MediaPickerOptions
            {
                Title = "Select a photo"
            });

            return photo;
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"Photo picker error: {ex.Message}");
            return null;
        }
    }

    public async Task<List<FileResult>> PickMultiplePhotosAsync()
    {
        try
        {
            // Note: Multiple photo picking might require platform-specific implementation
            var photos = new List<FileResult>();
            
            // For now, we'll use single photo picker in a loop
            // In a real app, you'd use platform-specific APIs for multiple selection
            var photo = await PickPhotoAsync();
            if (photo != null)
            {
                photos.Add(photo);
            }

            return photos;
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"Multiple photo picker error: {ex.Message}");
            return new List<FileResult>();
        }
    }

    private async Task<FileResult> ProcessPhotoAsync(FileResult photo, CameraOptions options)
    {
        // This would include photo processing logic like:
        // - Resizing based on PhotoSize
        // - Compression based on Quality
        // - Cropping if AllowCropping is true
        
        // For now, return the original photo
        return photo;
    }
}
```

### 2.2 Camera Demo Page

```xml
<!-- d:\A-Work\learn from ai\maui-xaml-mastery\src\12-PlatformFeatures\Views\CameraPage.xaml -->
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="PlatformFeatures.Views.CameraPage"
             Title="Camera Features">

    <Grid RowDefinitions="*, Auto">
        
        <!-- Image Display Area -->
        <ScrollView Grid.Row="0" Padding="20">
            <StackLayout Spacing="20">
                
                <!-- Captured/Selected Image -->
                <Frame x:Name="ImageFrame"
                       BackgroundColor="LightGray"
                       HasShadow="True"
                       CornerRadius="10"
                       HeightRequest="300"
                       IsVisible="False">
                    <Image x:Name="CapturedImage"
                           Aspect="AspectFit" />
                </Frame>

                <!-- Placeholder when no image -->
                <Frame x:Name="PlaceholderFrame"
                       BackgroundColor="#F5F5F5"
                       HasShadow="False"
                       CornerRadius="10"
                       HeightRequest="300">
                    <StackLayout VerticalOptions="Center" 
                                HorizontalOptions="Center">
                        <Label Text="📷"
                               FontSize="48"
                               HorizontalOptions="Center" />
                        <Label Text="No image selected"
                               FontSize="16"
                               TextColor="Gray"
                               HorizontalOptions="Center" />
                    </StackLayout>
                </Frame>

                <!-- Image Info -->
                <StackLayout x:Name="ImageInfoContainer"
                            IsVisible="False"
                            Spacing="10">
                    
                    <Label Text="Image Information"
                           FontSize="18"
                           FontAttributes="Bold" />
                    
                    <Grid ColumnDefinitions="Auto, *" 
                          RowDefinitions="Auto, Auto, Auto, Auto"
                          ColumnSpacing="10"
                          RowSpacing="5">
                        
                        <Label Grid.Row="0" Grid.Column="0" 
                               Text="File Name:" 
                               FontAttributes="Bold" />
                        <Label Grid.Row="0" Grid.Column="1" 
                               x:Name="FileNameLabel" />
                        
                        <Label Grid.Row="1" Grid.Column="0" 
                               Text="File Size:" 
                               FontAttributes="Bold" />
                        <Label Grid.Row="1" Grid.Column="1" 
                               x:Name="FileSizeLabel" />
                        
                        <Label Grid.Row="2" Grid.Column="0" 
                               Text="Dimensions:" 
                               FontAttributes="Bold" />
                        <Label Grid.Row="2" Grid.Column="1" 
                               x:Name="DimensionsLabel" />
                        
                        <Label Grid.Row="3" Grid.Column="0" 
                               Text="Content Type:" 
                               FontAttributes="Bold" />
                        <Label Grid.Row="3" Grid.Column="1" 
                               x:Name="ContentTypeLabel" />
                    </Grid>
                </StackLayout>

            </StackLayout>
        </ScrollView>

        <!-- Camera Controls -->
        <StackLayout Grid.Row="1" 
                     Padding="20"
                     Spacing="15"
                     BackgroundColor="{AppThemeBinding Light=#F8F9FA, Dark=#212529}">
            
            <!-- Camera Options -->
            <StackLayout Orientation="Horizontal" 
                        HorizontalOptions="Center"
                        Spacing="20">
                
                <StackLayout>
                    <Label Text="Quality" 
                           HorizontalOptions="Center"
                           FontSize="12" />
                    <Slider x:Name="QualitySlider"
                            Minimum="10"
                            Maximum="100"
                            Value="80"
                            WidthRequest="100" />
                    <Label x:Name="QualityLabel"
                           Text="80%"
                           HorizontalOptions="Center"
                           FontSize="10" />
                </StackLayout>

                <StackLayout>
                    <Label Text="Size" 
                           HorizontalOptions="Center"
                           FontSize="12" />
                    <Picker x:Name="SizePicker"
                            Title="Photo Size"
                            WidthRequest="100">
                        <Picker.ItemsSource>
                            <x:Array Type="{x:Type x:String}">
                                <x:String>Small</x:String>
                                <x:String>Medium</x:String>
                                <x:String>Large</x:String>
                            </x:Array>
                        </Picker.ItemsSource>
                    </Picker>
                </StackLayout>

                <StackLayout>
                    <Label Text="Cropping" 
                           HorizontalOptions="Center"
                           FontSize="12" />
                    <Switch x:Name="CroppingSwitch"
                            HorizontalOptions="Center" />
                </StackLayout>

            </StackLayout>

            <!-- Action Buttons -->
            <Grid ColumnDefinitions="*, *, *" 
                  ColumnSpacing="10">
                
                <Button Grid.Column="0"
                        Text="📷 Take Photo"
                        Style="{StaticResource PrimaryButton}"
                        Clicked="OnTakePhotoClicked" />
                
                <Button Grid.Column="1"
                        Text="🖼️ Pick Photo"
                        Style="{StaticResource SecondaryButton}"
                        Clicked="OnPickPhotoClicked" />
                
                <Button Grid.Column="2"
                        Text="🗑️ Clear"
                        Style="{StaticResource OutlineButton}"
                        Clicked="OnClearClicked" />

            </Grid>

            <!-- Platform-Specific Features -->
            <StackLayout IsVisible="{OnPlatform Android=True, iOS=True, Default=False}">
                <Button Text="📱 Multiple Photos (Mobile Only)"
                        Style="{StaticResource GhostButton}"
                        Clicked="OnPickMultipleClicked" />
            </StackLayout>

        </StackLayout>

    </Grid>

</ContentPage>
```

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\12-PlatformFeatures\Views\CameraPage.xaml.cs
using PlatformFeatures.Services;

namespace PlatformFeatures.Views;

public partial class CameraPage : ContentPage
{
    private readonly ICameraService _cameraService;

    public CameraPage(ICameraService cameraService)
    {
        InitializeComponent();
        _cameraService = cameraService;
        
        SetupUI();
    }

    private void SetupUI()
    {
        // Setup quality slider
        QualitySlider.ValueChanged += (s, e) =>
        {
            QualityLabel.Text = $"{(int)e.NewValue}%";
        };

        // Set default values
        SizePicker.SelectedIndex = 1; // Medium
        QualitySlider.Value = 80;
    }

    private async void OnTakePhotoClicked(object sender, EventArgs e)
    {
        try
        {
            var button = (Button)sender;
            button.IsEnabled = false;
            button.Text = "📷 Taking...";

            // Check camera availability
            if (!await _cameraService.IsCameraAvailableAsync())
            {
                await DisplayAlert("Error", "Camera is not available on this device", "OK");
                return;
            }

            // Get camera options
            var options = new CameraOptions
            {
                Quality = (int)QualitySlider.Value,
                PhotoSize = GetSelectedPhotoSize(),
                AllowCropping = CroppingSwitch.IsToggled
            };

            // Take photo
            var result = await _cameraService.TakePhotoAsync(options);
            
            if (result != null)
            {
                await DisplayImageAsync(result);
            }
        }
        catch (Exception ex)
        {
            await DisplayAlert("Error", $"Failed to take photo: {ex.Message}", "OK");
        }
        finally
        {
            var button = (Button)sender;
            button.IsEnabled = true;
            button.Text = "📷 Take Photo";
        }
    }

    private async void OnPickPhotoClicked(object sender, EventArgs e)
    {
        try
        {
            var button = (Button)sender;
            button.IsEnabled = false;
            button.Text = "🖼️ Picking...";

            var result = await _cameraService.PickPhotoAsync();
            
            if (result != null)
            {
                await DisplayImageAsync(result);
            }
        }
        catch (Exception ex)
        {
            await DisplayAlert("Error", $"Failed to pick photo: {ex.Message}", "OK");
        }
        finally
        {
            var button = (Button)sender;
            button.IsEnabled = true;
            button.Text = "🖼️ Pick Photo";
        }
    }

    private async void OnPickMultipleClicked(object sender, EventArgs e)
    {
        try
        {
            var button = (Button)sender;
            button.IsEnabled = false;
            button.Text = "📱 Picking...";

            var results = await _cameraService.PickMultiplePhotosAsync();
            
            if (results.Any())
            {
                // For now, just display the first image
                await DisplayImageAsync(results.First());
                
                if (results.Count > 1)
                {
                    await DisplayAlert("Info", $"Selected {results.Count} photos. Displaying the first one.", "OK");
                }
            }
        }
        catch (Exception ex)
        {
            await DisplayAlert("Error", $"Failed to pick photos: {ex.Message}", "OK");
        }
        finally
        {
            var button = (Button)sender;
            button.IsEnabled = true;
            button.Text = "📱 Multiple Photos (Mobile Only)";
        }
    }

    private void OnClearClicked(object sender, EventArgs e)
    {
        ClearImage();
    }

    private async Task DisplayImageAsync(FileResult fileResult)
    {
        if (fileResult == null) return;

        try
        {
            // Display the image
            var stream = await fileResult.OpenReadAsync();
            CapturedImage.Source = ImageSource.FromStream(() => stream);

            // Show image container and hide placeholder
            ImageFrame.IsVisible = true;
            PlaceholderFrame.IsVisible = false;
            ImageInfoContainer.IsVisible = true;

            // Update image info
            await UpdateImageInfoAsync(fileResult);

            // Add some animation
            await ImageFrame.ScaleTo(0.8, 0);
            await ImageFrame.ScaleTo(1.0, 250, Easing.BounceOut);
        }
        catch (Exception ex)
        {
            await DisplayAlert("Error", $"Failed to display image: {ex.Message}", "OK");
        }
    }

    private async Task UpdateImageInfoAsync(FileResult fileResult)
    {
        try
        {
            FileNameLabel.Text = fileResult.FileName;
            ContentTypeLabel.Text = fileResult.ContentType ?? "Unknown";

            // Get file size
            using var stream = await fileResult.OpenReadAsync();
            var sizeInBytes = stream.Length;
            var sizeInKB = sizeInBytes / 1024.0;
            var sizeInMB = sizeInKB / 1024.0;

            if (sizeInMB >= 1)
                FileSizeLabel.Text = $"{sizeInMB:F2} MB";
            else
                FileSizeLabel.Text = $"{sizeInKB:F0} KB";

            // For dimensions, we'd need to use platform-specific code or image processing library
            DimensionsLabel.Text = "Loading...";
            
            // Simulate dimension detection (in a real app, use ImageSharp or similar)
            Device.StartTimer(TimeSpan.FromSeconds(1), () =>
            {
                Device.BeginInvokeOnMainThread(() =>
                {
                    DimensionsLabel.Text = "Unknown"; // Replace with actual dimensions
                });
                return false;
            });
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"Error updating image info: {ex.Message}");
        }
    }

    private void ClearImage()
    {
        CapturedImage.Source = null;
        ImageFrame.IsVisible = false;
        PlaceholderFrame.IsVisible = true;
        ImageInfoContainer.IsVisible = false;
    }

    private PhotoSize GetSelectedPhotoSize()
    {
        return SizePicker.SelectedIndex switch
        {
            0 => PhotoSize.Small,
            1 => PhotoSize.Medium,
            2 => PhotoSize.Large,
            _ => PhotoSize.Medium
        };
    }
}
```

---

## 3. 📍 Location Services

### 3.1 Location Service Implementation

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\12-PlatformFeatures\Services\ILocationService.cs
namespace PlatformFeatures.Services;

public interface ILocationService
{
    Task<bool> IsLocationAvailableAsync();
    Task<Location> GetCurrentLocationAsync();
    Task<Location> GetLastKnownLocationAsync();
    Task<bool> RequestLocationPermissionAsync();
    Task<List<Placemark>> GetPlacemarksAsync(double latitude, double longitude);
    Task<List<Location>> GetLocationsAsync(string address);
}
```

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\12-PlatformFeatures\Services\LocationService.cs
using Microsoft.Maui.Devices.Sensors;

namespace PlatformFeatures.Services;

public class LocationService : ILocationService
{
    public async Task<bool> IsLocationAvailableAsync()
    {
        try
        {
            var status = await Permissions.CheckStatusAsync<Permissions.LocationWhenInUse>();
            return status == PermissionStatus.Granted;
        }
        catch
        {
            return false;
        }
    }

    public async Task<bool> RequestLocationPermissionAsync()
    {
        try
        {
            var status = await Permissions.RequestAsync<Permissions.LocationWhenInUse>();
            return status == PermissionStatus.Granted;
        }
        catch
        {
            return false;
        }
    }

    public async Task<Location> GetCurrentLocationAsync()
    {
        try
        {
            var request = new GeolocationRequest
            {
                DesiredAccuracy = GeolocationAccuracy.Medium,
                Timeout = TimeSpan.FromSeconds(10)
            };

            var location = await Geolocation.Default.GetLocationAsync(request);
            return location;
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"Location error: {ex.Message}");
            return null;
        }
    }

    public async Task<Location> GetLastKnownLocationAsync()
    {
        try
        {
            var location = await Geolocation.Default.GetLastKnownLocationAsync();
            return location;
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"Last known location error: {ex.Message}");
            return null;
        }
    }

    public async Task<List<Placemark>> GetPlacemarksAsync(double latitude, double longitude)
    {
        try
        {
            var placemarks = await Geocoding.Default.GetPlacemarksAsync(latitude, longitude);
            return placemarks?.ToList() ?? new List<Placemark>();
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"Geocoding error: {ex.Message}");
            return new List<Placemark>();
        }
    }

    public async Task<List<Location>> GetLocationsAsync(string address)
    {
        try
        {
            var locations = await Geocoding.Default.GetLocationsAsync(address);
            return locations?.ToList() ?? new List<Location>();
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"Address geocoding error: {ex.Message}");
            return new List<Location>();
        }
    }
}
```

---

## 4. 🔐 Biometric Authentication

### 4.1 Biometric Service

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\12-PlatformFeatures\Services\IBiometricService.cs
namespace PlatformFeatures.Services;

public interface IBiometricService
{
    Task<bool> IsAvailableAsync();
    Task<BiometricAuthenticationResult> AuthenticateAsync(string reason);
    Task<BiometricType> GetAvailableBiometricTypesAsync();
}

public enum BiometricType
{
    None,
    Fingerprint,
    Face,
    Voice,
    Multiple
}

public class BiometricAuthenticationResult
{
    public bool IsSuccessful { get; set; }
    public string ErrorMessage { get; set; }
    public BiometricType AuthenticationType { get; set; }
}
```

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\12-PlatformFeatures\Services\BiometricService.cs
#if ANDROID
using AndroidX.Biometric;
using Android.Content;
#endif

namespace PlatformFeatures.Services;

public class BiometricService : IBiometricService
{
    public async Task<bool> IsAvailableAsync()
    {
        try
        {
#if ANDROID
            return await CheckAndroidBiometricAsync();
#elif IOS
            return await CheckiOSBiometricAsync();
#else
            return false;
#endif
        }
        catch
        {
            return false;
        }
    }

    public async Task<BiometricAuthenticationResult> AuthenticateAsync(string reason)
    {
        try
        {
#if ANDROID
            return await AuthenticateAndroidAsync(reason);
#elif IOS
            return await AuthenticateiOSAsync(reason);
#else
            return new BiometricAuthenticationResult
            {
                IsSuccessful = false,
                ErrorMessage = "Biometric authentication not supported on this platform"
            };
#endif
        }
        catch (Exception ex)
        {
            return new BiometricAuthenticationResult
            {
                IsSuccessful = false,
                ErrorMessage = ex.Message
            };
        }
    }

    public async Task<BiometricType> GetAvailableBiometricTypesAsync()
    {
        try
        {
#if ANDROID
            return await GetAndroidBiometricTypesAsync();
#elif IOS
            return await GetiOSBiometricTypesAsync();
#else
            return BiometricType.None;
#endif
        }
        catch
        {
            return BiometricType.None;
        }
    }

#if ANDROID
    private async Task<bool> CheckAndroidBiometricAsync()
    {
        var context = Platform.CurrentActivity ?? Android.App.Application.Context;
        return BiometricManager.From(context).CanAuthenticate(BiometricManager.Authenticators.BiometricWeak) == BiometricManager.BiometricSuccess;
    }

    private async Task<BiometricAuthenticationResult> AuthenticateAndroidAsync(string reason)
    {
        var tcs = new TaskCompletionSource<BiometricAuthenticationResult>();
        
        var context = Platform.CurrentActivity as AndroidX.Fragment.App.FragmentActivity;
        if (context == null)
        {
            return new BiometricAuthenticationResult
            {
                IsSuccessful = false,
                ErrorMessage = "Unable to get current activity"
            };
        }

        var biometricPrompt = new BiometricPrompt(context, 
            ContextCompat.GetMainExecutor(context), 
            new AuthenticationCallback(tcs));

        var promptInfo = new BiometricPrompt.PromptInfo.Builder()
            .SetTitle("Biometric Authentication")
            .SetSubtitle(reason)
            .SetNegativeButtonText("Cancel")
            .Build();

        biometricPrompt.Authenticate(promptInfo);
        
        return await tcs.Task;
    }

    private async Task<BiometricType> GetAndroidBiometricTypesAsync()
    {
        var context = Platform.CurrentActivity ?? Android.App.Application.Context;
        var biometricManager = BiometricManager.From(context);
        
        switch (biometricManager.CanAuthenticate(BiometricManager.Authenticators.BiometricWeak))
        {
            case BiometricManager.BiometricSuccess:
                return BiometricType.Multiple; // Android doesn't distinguish between types easily
            default:
                return BiometricType.None;
        }
    }

    private class AuthenticationCallback : BiometricPrompt.AuthenticationCallback
    {
        private readonly TaskCompletionSource<BiometricAuthenticationResult> _tcs;

        public AuthenticationCallback(TaskCompletionSource<BiometricAuthenticationResult> tcs)
        {
            _tcs = tcs;
        }

        public override void OnAuthenticationSucceeded(BiometricPrompt.AuthenticationResult result)
        {
            base.OnAuthenticationSucceeded(result);
            _tcs.SetResult(new BiometricAuthenticationResult
            {
                IsSuccessful = true,
                AuthenticationType = BiometricType.Multiple
            });
        }

        public override void OnAuthenticationError(int errorCode, Java.Lang.ICharSequence errString)
        {
            base.OnAuthenticationError(errorCode, errString);
            _tcs.SetResult(new BiometricAuthenticationResult
            {
                IsSuccessful = false,
                ErrorMessage = errString?.ToString() ?? "Authentication error"
            });
        }

        public override void OnAuthenticationFailed()
        {
            base.OnAuthenticationFailed();
            _tcs.SetResult(new BiometricAuthenticationResult
            {
                IsSuccessful = false,
                ErrorMessage = "Authentication failed"
            });
        }
    }
#endif

#if IOS
    private async Task<bool> CheckiOSBiometricAsync()
    {
        var context = new LocalAuthentication.LAContext();
        var error = new Foundation.NSError();
        return context.CanEvaluatePolicy(LocalAuthentication.LAPolicy.DeviceOwnerAuthenticationWithBiometrics, out error);
    }

    private async Task<BiometricAuthenticationResult> AuthenticateiOSAsync(string reason)
    {
        var context = new LocalAuthentication.LAContext();
        
        var result = await context.EvaluatePolicyAsync(
            LocalAuthentication.LAPolicy.DeviceOwnerAuthenticationWithBiometrics,
            reason);

        return new BiometricAuthenticationResult
        {
            IsSuccessful = result.Item1,
            ErrorMessage = result.Item2?.LocalizedDescription,
            AuthenticationType = await GetiOSBiometricTypesAsync()
        };
    }

    private async Task<BiometricType> GetiOSBiometricTypesAsync()
    {
        var context = new LocalAuthentication.LAContext();
        
        switch (context.BiometryType)
        {
            case LocalAuthentication.LABiometryType.FaceId:
                return BiometricType.Face;
            case LocalAuthentication.LABiometryType.TouchId:
                return BiometricType.Fingerprint;
            default:
                return BiometricType.None;
        }
    }
#endif
}
```

---

## 5. 🧪 Practical Exercises

### Exercise 1: Platform-Specific Navigation
สร้าง navigation system ที่ปรับตาม platform:
- Android: Navigation Drawer + FAB
- iOS: Tab Bar + Navigation Controller
- Windows: NavigationView + CommandBar

### Exercise 2: Adaptive Media Player
สร้าง media player ที่มี:
- Platform-specific controls
- Background audio support
- Lock screen controls
- Adaptive UI layouts

### Exercise 3: Cross-Platform Notifications
สร้างระบบ notifications ที่:
- ใช้ platform-specific notification styles
- Support rich notifications
- Handle notification interactions
- Background processing

---

## 6. 📝 Best Practices & Performance

### 🎯 Platform-Specific Development Tips

```csharp
// ✅ DO: Use OnPlatform for platform-specific values
<Button BackgroundColor="{OnPlatform Android=#2196F3, iOS=#007AFF, WinUI=#0078D4}" />

// ✅ DO: Check platform capabilities before using features
public async Task<bool> CanUseFeatureAsync()
{
    if (DeviceInfo.Platform == DevicePlatform.Android)
    {
        // Check Android-specific requirements
        return await CheckAndroidCapabilityAsync();
    }
    else if (DeviceInfo.Platform == DevicePlatform.iOS)
    {
        // Check iOS-specific requirements
        return await CheckiOSCapabilityAsync();
    }
    
    return false;
}

// ✅ DO: Handle permissions properly
public async Task<bool> RequestPermissionsAsync()
{
    var status = await Permissions.CheckStatusAsync<Permissions.Camera>();
    
    if (status != PermissionStatus.Granted)
    {
        status = await Permissions.RequestAsync<Permissions.Camera>();
    }
    
    if (status != PermissionStatus.Granted)
    {
        // Show explanation to user
        await ShowPermissionExplanationAsync();
        return false;
    }
    
    return true;
}

// ✅ DO: Use dependency injection for platform services
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

        // Register platform-specific services
        builder.Services.AddSingleton<ICameraService, CameraService>();
        builder.Services.AddSingleton<ILocationService, LocationService>();
        builder.Services.AddSingleton<IBiometricService, BiometricService>();

        return builder.Build();
    }
}

// ❌ DON'T: Assume all platforms support the same features
// Wrong:
await MediaPicker.CapturePhotoAsync(); // Might not work on all platforms

// Right:
if (MediaPicker.Default.IsCaptureSupported)
{
    await MediaPicker.Default.CapturePhotoAsync();
}
```

---

## 7. 🎯 Quiz & Assessment

### Quiz Questions

1. **Platform Design**: Material Design, Cupertino และ Fluent Design มีหลักการออกแบบที่แตกต่างกันอย่างไร?

2. **Permission Handling**: วิธีที่ดีที่สุดในการ handle permissions ข้าม platform คืออะไร?

3. **Native Features**: จะเข้าถึง platform-specific hardware features ได้อย่างไร?

4. **Performance**: เทคนิคไหนช่วยเพิ่มประสิทธิภาพแอปข้าม platform?

### Practical Assessment
สร้าง **"Cross-Platform Feature App"** ที่มี:
- ✅ Platform-adaptive UI design
- ✅ Camera and photo features
- ✅ Location services integration
- ✅ Biometric authentication
- ✅ Platform-specific optimizations

---

## 8. 🚀 Next Steps

### Advanced Topics to Explore
- **Custom Handlers**: สร้าง custom platform handlers
- **Native Library Integration**: ใช้ native libraries ใน MAUI
- **Platform-Specific Performance**: ปรับแต่ง performance per platform
- **Advanced Hardware Features**: เข้าถึง specialized hardware

### Chapter 13 Preview
หัวข้อต่อไป: **"Data Management & Storage"**
- Local database integration
- Cloud data synchronization
- Offline-first architecture
- Data caching strategies

---

## 📚 Additional Resources

### 🔗 Useful Links
- [Platform-Specific Features Guide](https://docs.microsoft.com/dotnet/maui/platform-integration/)
- [Camera and Media](https://docs.microsoft.com/dotnet/maui/platform-integration/device-media)
- [Geolocation](https://docs.microsoft.com/dotnet/maui/platform-integration/device/geolocation)

### 📖 Recommended Reading
- "Cross-Platform Mobile Development Best Practices"
- "Platform-Specific UI Design Guidelines"
- "Mobile Security and Privacy Implementation"

---

*หมายเหตุ: บทนี้เน้นการใช้ platform-specific features อย่างมีประสิทธิภาพและการสร้าง adaptive UI ที่เหมาะสมกับแต่ละ platform*
