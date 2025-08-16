# 🎭 Chapter 11: Animations & Visual Effects

> **หัวข้อ**: Advanced Animations & Interactive Visual Effects  
> **ระยะเวลา**: 3 ชั่วโมง  
> **ความยากง่าย**: ระดับกลาง-สูง  
> **Prerequisites**: บทที่ 1-10

## 🎯 Learning Objectives

หลังจากเรียนบทนี้แล้ว คุณจะสามารถ:

- ✅ สร้าง smooth animations ที่ enhance UX
- ✅ ใช้ Lottie animations สำหรับ complex effects
- ✅ สร้าง interactive transitions และ micro-interactions
- ✅ เพิ่มประสิทธิภาพ animation performance
- ✅ ใช้ CSS-like animation approaches

---

## 📱 Chapter Overview

### 🏗️ What We'll Build
สร้าง **"Interactive Animation Showcase"** ที่มี:
- 🎨 Custom loading animations
- 🔄 Page transition effects
- 📱 Interactive UI animations
- ✨ Micro-interactions
- 🎭 Complex Lottie animations

---

## 1. 🎨 Basic Animation Fundamentals

### 1.1 CSS-like Animation Utilities

```xml
<!-- d:\A-Work\learn from ai\maui-xaml-mastery\src\11-Animations\Styles\AnimationUtilities.xaml -->
<ResourceDictionary xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
                   xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">

    <!-- CSS-like Animation Duration Constants -->
    <x:Double x:Key="Duration150">150</x:Double>
    <x:Double x:Key="Duration300">300</x:Double>
    <x:Double x:Key="Duration500">500</x:Double>
    <x:Double x:Key="Duration700">700</x:Double>

    <!-- Easing Functions (CSS-like) -->
    <x:String x:Key="EaseIn">CubicIn</x:String>
    <x:String x:Key="EaseOut">CubicOut</x:String>
    <x:String x:Key="EaseInOut">CubicInOut</x:String>
    <x:String x:Key="EaseBounce">BounceOut</x:String>

    <!-- Animation Triggers -->
    <Style x:Key="FadeInTrigger" TargetType="VisualElement">
        <Style.Triggers>
            <Trigger TargetType="VisualElement" Property="IsVisible" Value="True">
                <Trigger.EnterActions>
                    <TriggerAction>
                        <TriggerAction.Action>
                            <Storyboard>
                                <DoubleAnimation 
                                    Storyboard.TargetProperty="Opacity"
                                    From="0" 
                                    To="1" 
                                    Duration="0:0:0.3"
                                    Easing="{x:Static Easing.CubicOut}" />
                            </Storyboard>
                        </TriggerAction.Action>
                    </TriggerAction>
                </Trigger.EnterActions>
            </Trigger>
        </Style.Triggers>
    </Style>

</ResourceDictionary>
```

### 1.2 Animation Extension Methods

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\11-Animations\Extensions\AnimationExtensions.cs
using Microsoft.Maui.Controls;

namespace Animations.Extensions;

public static class AnimationExtensions
{
    #region Fade Animations (CSS-like)
    
    /// <summary>
    /// Fade in animation - like CSS fadeIn
    /// </summary>
    public static Task<bool> FadeInAsync(this VisualElement element, uint duration = 300, Easing easing = null)
    {
        element.Opacity = 0;
        element.IsVisible = true;
        return element.FadeTo(1, duration, easing ?? Easing.CubicOut);
    }

    /// <summary>
    /// Fade out animation - like CSS fadeOut
    /// </summary>
    public static async Task<bool> FadeOutAsync(this VisualElement element, uint duration = 300, Easing easing = null)
    {
        var result = await element.FadeTo(0, duration, easing ?? Easing.CubicIn);
        element.IsVisible = false;
        return result;
    }

    /// <summary>
    /// Fade toggle - like CSS animation toggle
    /// </summary>
    public static Task<bool> FadeToggleAsync(this VisualElement element, uint duration = 300)
    {
        return element.Opacity > 0.5 
            ? element.FadeOutAsync(duration) 
            : element.FadeInAsync(duration);
    }

    #endregion

    #region Scale Animations (CSS-like transform: scale)

    /// <summary>
    /// Scale in animation - like CSS scaleIn
    /// </summary>
    public static async Task<bool> ScaleInAsync(this VisualElement element, double scale = 1.0, uint duration = 300, Easing easing = null)
    {
        element.Scale = 0;
        element.IsVisible = true;
        return await element.ScaleTo(scale, duration, easing ?? Easing.BounceOut);
    }

    /// <summary>
    /// Scale out animation - like CSS scaleOut
    /// </summary>
    public static async Task<bool> ScaleOutAsync(this VisualElement element, uint duration = 300, Easing easing = null)
    {
        var result = await element.ScaleTo(0, duration, easing ?? Easing.CubicIn);
        element.IsVisible = false;
        element.Scale = 1; // Reset for next use
        return result;
    }

    /// <summary>
    /// Pulse animation - like CSS pulse effect
    /// </summary>
    public static async Task PulseAsync(this VisualElement element, double scale = 1.1, uint duration = 150, int cycles = 1)
    {
        for (int i = 0; i < cycles; i++)
        {
            await element.ScaleTo(scale, duration / 2, Easing.CubicOut);
            await element.ScaleTo(1.0, duration / 2, Easing.CubicIn);
        }
    }

    #endregion

    #region Slide Animations (CSS-like transform: translate)

    /// <summary>
    /// Slide in from left - like CSS slideInLeft
    /// </summary>
    public static async Task<bool> SlideInFromLeftAsync(this VisualElement element, uint duration = 300, Easing easing = null)
    {
        element.TranslationX = -element.Width;
        element.IsVisible = true;
        return await element.TranslateTo(0, element.TranslationY, duration, easing ?? Easing.CubicOut);
    }

    /// <summary>
    /// Slide in from right - like CSS slideInRight
    /// </summary>
    public static async Task<bool> SlideInFromRightAsync(this VisualElement element, uint duration = 300, Easing easing = null)
    {
        element.TranslationX = element.Width;
        element.IsVisible = true;
        return await element.TranslateTo(0, element.TranslationY, duration, easing ?? Easing.CubicOut);
    }

    /// <summary>
    /// Slide out to left - like CSS slideOutLeft
    /// </summary>
    public static async Task<bool> SlideOutToLeftAsync(this VisualElement element, uint duration = 300, Easing easing = null)
    {
        var result = await element.TranslateTo(-element.Width, element.TranslationY, duration, easing ?? Easing.CubicIn);
        element.IsVisible = false;
        element.TranslationX = 0; // Reset for next use
        return result;
    }

    #endregion

    #region Rotation Animations

    /// <summary>
    /// Rotate animation - like CSS rotate
    /// </summary>
    public static Task<bool> RotateAsync(this VisualElement element, double rotation, uint duration = 300, Easing easing = null)
    {
        return element.RotateTo(rotation, duration, easing ?? Easing.CubicOut);
    }

    /// <summary>
    /// Spin animation - continuous rotation
    /// </summary>
    public static async Task SpinAsync(this VisualElement element, int rotations = 1, uint duration = 1000, Easing easing = null)
    {
        var totalRotation = rotations * 360;
        await element.RotateTo(totalRotation, duration, easing ?? Easing.Linear);
        element.Rotation = 0; // Reset
    }

    #endregion

    #region Complex Animations

    /// <summary>
    /// Bounce in animation - like CSS bounceIn
    /// </summary>
    public static async Task<bool> BounceInAsync(this VisualElement element, uint duration = 600)
    {
        element.Scale = 0;
        element.IsVisible = true;

        await Task.WhenAll(
            element.FadeTo(1, duration / 4, Easing.CubicOut),
            element.ScaleTo(1.1, duration / 4, Easing.CubicOut)
        );

        await element.ScaleTo(0.9, duration / 4, Easing.CubicInOut);
        await element.ScaleTo(1.0, duration / 2, Easing.BounceOut);

        return true;
    }

    /// <summary>
    /// Shake animation - like CSS shake
    /// </summary>
    public static async Task ShakeAsync(this VisualElement element, uint duration = 500, double intensity = 10)
    {
        var originalX = element.TranslationX;
        var shakeCount = 8;
        var shakeDuration = duration / (uint)shakeCount;

        for (int i = 0; i < shakeCount; i++)
        {
            var offset = (i % 2 == 0) ? intensity : -intensity;
            await element.TranslateTo(originalX + offset, element.TranslationY, shakeDuration, Easing.Linear);
        }

        await element.TranslateTo(originalX, element.TranslationY, shakeDuration, Easing.CubicOut);
    }

    /// <summary>
    /// Heartbeat animation - like CSS heartBeat
    /// </summary>
    public static async Task HeartBeatAsync(this VisualElement element, uint duration = 1000, int beats = 2)
    {
        var beatDuration = duration / (uint)(beats * 2);

        for (int i = 0; i < beats; i++)
        {
            await element.ScaleTo(1.3, beatDuration, Easing.CubicOut);
            await element.ScaleTo(1.0, beatDuration, Easing.CubicIn);
        }
    }

    #endregion

    #region Chaining Utilities

    /// <summary>
    /// Chain multiple animations in sequence
    /// </summary>
    public static async Task ChainAsync(this VisualElement element, params Func<Task>[] animations)
    {
        foreach (var animation in animations)
        {
            await animation();
        }
    }

    /// <summary>
    /// Run multiple animations in parallel
    /// </summary>
    public static Task ParallelAsync(this VisualElement element, params Func<Task>[] animations)
    {
        return Task.WhenAll(animations.Select(a => a()));
    }

    #endregion
}
```

---

## 2. 🔄 Page Transitions & Navigation Animations

### 2.1 Custom Page Transition

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\11-Animations\Services\PageTransitionService.cs
using Microsoft.Maui.Controls;

namespace Animations.Services;

public class PageTransitionService
{
    public enum TransitionType
    {
        SlideLeft,
        SlideRight,
        SlideUp,
        SlideDown,
        Fade,
        Scale,
        Flip
    }

    public static async Task NavigateWithTransitionAsync(
        INavigation navigation, 
        Page targetPage, 
        TransitionType transition = TransitionType.SlideLeft,
        uint duration = 300)
    {
        // Setup initial state for incoming page
        await SetupIncomingPageAsync(targetPage, transition);
        
        // Get current page for outgoing transition
        var currentPage = navigation.NavigationStack.LastOrDefault();
        
        // Navigate to new page
        await navigation.PushAsync(targetPage, false);
        
        // Perform transition animations
        var incomingTask = AnimateIncomingPageAsync(targetPage, transition, duration);
        var outgoingTask = currentPage != null 
            ? AnimateOutgoingPageAsync(currentPage, transition, duration)
            : Task.CompletedTask;
            
        await Task.WhenAll(incomingTask, outgoingTask);
    }

    private static async Task SetupIncomingPageAsync(Page page, TransitionType transition)
    {
        switch (transition)
        {
            case TransitionType.SlideLeft:
                page.TranslationX = page.Width;
                break;
            case TransitionType.SlideRight:
                page.TranslationX = -page.Width;
                break;
            case TransitionType.SlideUp:
                page.TranslationY = page.Height;
                break;
            case TransitionType.SlideDown:
                page.TranslationY = -page.Height;
                break;
            case TransitionType.Fade:
                page.Opacity = 0;
                break;
            case TransitionType.Scale:
                page.Scale = 0;
                break;
            case TransitionType.Flip:
                page.RotationY = -90;
                break;
        }
    }

    private static async Task AnimateIncomingPageAsync(Page page, TransitionType transition, uint duration)
    {
        switch (transition)
        {
            case TransitionType.SlideLeft:
            case TransitionType.SlideRight:
                await page.TranslateTo(0, 0, duration, Easing.CubicOut);
                break;
            case TransitionType.SlideUp:
            case TransitionType.SlideDown:
                await page.TranslateTo(0, 0, duration, Easing.CubicOut);
                break;
            case TransitionType.Fade:
                await page.FadeTo(1, duration, Easing.CubicOut);
                break;
            case TransitionType.Scale:
                await page.ScaleTo(1, duration, Easing.BounceOut);
                break;
            case TransitionType.Flip:
                await page.RotateYTo(0, duration, Easing.CubicOut);
                break;
        }
    }

    private static async Task AnimateOutgoingPageAsync(Page page, TransitionType transition, uint duration)
    {
        switch (transition)
        {
            case TransitionType.SlideLeft:
                await page.TranslateTo(-page.Width, 0, duration, Easing.CubicIn);
                break;
            case TransitionType.SlideRight:
                await page.TranslateTo(page.Width, 0, duration, Easing.CubicIn);
                break;
            case TransitionType.SlideUp:
                await page.TranslateTo(0, -page.Height, duration, Easing.CubicIn);
                break;
            case TransitionType.SlideDown:
                await page.TranslateTo(0, page.Height, duration, Easing.CubicIn);
                break;
            case TransitionType.Fade:
                await page.FadeTo(0, duration, Easing.CubicIn);
                break;
            case TransitionType.Scale:
                await page.ScaleTo(0, duration, Easing.CubicIn);
                break;
            case TransitionType.Flip:
                await page.RotateYTo(90, duration, Easing.CubicIn);
                break;
        }
    }
}
```

### 2.2 Animated NavigationPage

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\11-Animations\Pages\AnimatedNavigationPage.cs
namespace Animations.Pages;

public class AnimatedNavigationPage : NavigationPage
{
    public AnimatedNavigationPage(Page root) : base(root)
    {
        SetupAnimations();
    }

    private void SetupAnimations()
    {
        Pushed += OnPagePushed;
        Popped += OnPagePopped;
    }

    private async void OnPagePushed(object sender, NavigationEventArgs e)
    {
        if (e.Page != null)
        {
            await e.Page.SlideInFromRightAsync(250);
        }
    }

    private async void OnPagePopped(object sender, NavigationEventArgs e)
    {
        if (e.Page != null)
        {
            await e.Page.SlideOutToRightAsync(250);
        }
    }
}
```

---

## 3. 🎨 Lottie Animations Integration

### 3.1 Lottie Animation Control

```xml
<!-- d:\A-Work\learn from ai\maui-xaml-mastery\src\11-Animations\Views\LottieAnimationsPage.xaml -->
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:lottie="clr-namespace:SkiaSharp.Extended.UI.Controls;assembly=SkiaSharp.Extended.UI"
             x:Class="Animations.Views.LottieAnimationsPage"
             Title="Lottie Animations">

    <ScrollView>
        <StackLayout Padding="20" Spacing="20">

            <!-- Loading Animation -->
            <Frame Style="{StaticResource card}">
                <StackLayout>
                    <Label Text="Loading Animation" 
                           Style="{StaticResource text-lg}" 
                           StyleClass="font-bold" />
                    
                    <lottie:SKLottieView x:Name="LoadingAnimation"
                                        Source="loading.json"
                                        RepeatCount="-1"
                                        HeightRequest="100"
                                        WidthRequest="100"
                                        HorizontalOptions="Center" />
                    
                    <StackLayout Orientation="Horizontal" HorizontalOptions="Center">
                        <Button Text="Play" 
                                Clicked="OnPlayClicked"
                                Style="{StaticResource PrimaryButton}" />
                        <Button Text="Pause" 
                                Clicked="OnPauseClicked"
                                Style="{StaticResource SecondaryButton}" />
                        <Button Text="Stop" 
                                Clicked="OnStopClicked"
                                Style="{StaticResource OutlineButton}" />
                    </StackLayout>
                </StackLayout>
            </Frame>

            <!-- Success Animation -->
            <Frame Style="{StaticResource card}">
                <StackLayout>
                    <Label Text="Success Animation" 
                           Style="{StaticResource text-lg}" 
                           StyleClass="font-bold" />
                    
                    <lottie:SKLottieView x:Name="SuccessAnimation"
                                        Source="success.json"
                                        RepeatCount="1"
                                        HeightRequest="120"
                                        WidthRequest="120"
                                        HorizontalOptions="Center" />
                    
                    <Button Text="Trigger Success" 
                            Clicked="OnSuccessClicked"
                            Style="{StaticResource PrimaryButton}" />
                </StackLayout>
            </Frame>

            <!-- Interactive Animation -->
            <Frame Style="{StaticResource card}">
                <StackLayout>
                    <Label Text="Interactive Animation" 
                           Style="{StaticResource text-lg}" 
                           StyleClass="font-bold" />
                    
                    <lottie:SKLottieView x:Name="InteractiveAnimation"
                                        Source="interactive.json"
                                        RepeatCount="0"
                                        HeightRequest="150"
                                        WidthRequest="150"
                                        HorizontalOptions="Center">
                        <lottie:SKLottieView.GestureRecognizers>
                            <TapGestureRecognizer Tapped="OnInteractiveAnimationTapped" />
                        </lottie:SKLottieView.GestureRecognizers>
                    </lottie:SKLottieView>
                    
                    <Slider x:Name="AnimationSlider"
                            Minimum="0"
                            Maximum="1"
                            Value="0"
                            ValueChanged="OnAnimationSliderChanged" />
                    
                    <Label Text="Drag slider to control animation frame" 
                           HorizontalOptions="Center"
                           Style="{StaticResource text-sm}"
                           StyleClass="text-gray-500" />
                </StackLayout>
            </Frame>

            <!-- Morphing Animation -->
            <Frame Style="{StaticResource card}">
                <StackLayout>
                    <Label Text="Morphing Animation" 
                           Style="{StaticResource text-lg}" 
                           StyleClass="font-bold" />
                    
                    <lottie:SKLottieView x:Name="MorphingAnimation"
                                        Source="morphing.json"
                                        RepeatCount="-1"
                                        HeightRequest="200"
                                        WidthRequest="200"
                                        HorizontalOptions="Center" />
                    
                    <StackLayout Orientation="Horizontal" HorizontalOptions="Center">
                        <Button Text="Slow" 
                                Clicked="OnSlowClicked"
                                Style="{StaticResource GhostButton}" />
                        <Button Text="Normal" 
                                Clicked="OnNormalClicked"
                                Style="{StaticResource GhostButton}" />
                        <Button Text="Fast" 
                                Clicked="OnFastClicked"
                                Style="{StaticResource GhostButton}" />
                    </StackLayout>
                </StackLayout>
            </Frame>

        </StackLayout>
    </ScrollView>

</ContentPage>
```

### 3.2 Lottie Animation Code-Behind

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\11-Animations\Views\LottieAnimationsPage.xaml.cs
using SkiaSharp.Extended.UI.Controls;

namespace Animations.Views;

public partial class LottieAnimationsPage : ContentPage
{
    public LottieAnimationsPage()
    {
        InitializeComponent();
        SetupAnimations();
    }

    private void SetupAnimations()
    {
        // Auto-start loading animation
        LoadingAnimation.IsAnimationEnabled = true;
        
        // Setup success animation completion
        SuccessAnimation.AnimationCompleted += OnSuccessAnimationCompleted;
    }

    #region Loading Animation Controls

    private void OnPlayClicked(object sender, EventArgs e)
    {
        LoadingAnimation.IsAnimationEnabled = true;
    }

    private void OnPauseClicked(object sender, EventArgs e)
    {
        LoadingAnimation.IsAnimationEnabled = false;
    }

    private void OnStopClicked(object sender, EventArgs e)
    {
        LoadingAnimation.IsAnimationEnabled = false;
        LoadingAnimation.Progress = 0;
    }

    #endregion

    #region Success Animation

    private async void OnSuccessClicked(object sender, EventArgs e)
    {
        // Reset animation
        SuccessAnimation.Progress = 0;
        SuccessAnimation.IsAnimationEnabled = true;
        
        // Add some visual feedback
        var button = (Button)sender;
        await button.ScaleTo(0.9, 50);
        await button.ScaleTo(1.0, 50);
    }

    private async void OnSuccessAnimationCompleted(object sender, EventArgs e)
    {
        // Show completion feedback
        await DisplayAlert("Success!", "Animation completed successfully!", "OK");
    }

    #endregion

    #region Interactive Animation

    private async void OnInteractiveAnimationTapped(object sender, EventArgs e)
    {
        // Trigger a quick animation sequence
        InteractiveAnimation.IsAnimationEnabled = true;
        
        // Add haptic feedback
        try
        {
#if ANDROID || IOS
            Microsoft.Maui.Authentication.WebAuthenticatorResult.HapticFeedback.Perform(Microsoft.Maui.Authentication.WebAuthenticatorResult.HapticFeedbackType.Click);
#endif
        }
        catch { /* Ignore haptic feedback errors */ }
        
        await Task.Delay(1000); // Let animation play for 1 second
        InteractiveAnimation.IsAnimationEnabled = false;
    }

    private void OnAnimationSliderChanged(object sender, ValueChangedEventArgs e)
    {
        // Control animation frame manually
        InteractiveAnimation.Progress = (float)e.NewValue;
        InteractiveAnimation.IsAnimationEnabled = false;
    }

    #endregion

    #region Morphing Animation Speed Controls

    private void OnSlowClicked(object sender, EventArgs e)
    {
        MorphingAnimation.AnimationSpeed = 0.5f;
    }

    private void OnNormalClicked(object sender, EventArgs e)
    {
        MorphingAnimation.AnimationSpeed = 1.0f;
    }

    private void OnFastClicked(object sender, EventArgs e)
    {
        MorphingAnimation.AnimationSpeed = 2.0f;
    }

    #endregion
}
```

---

## 4. ✨ Micro-interactions & UI Feedback

### 4.1 Interactive Button Effects

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\11-Animations\Behaviors\ButtonAnimationBehavior.cs
using Microsoft.Maui.Controls;

namespace Animations.Behaviors;

public class ButtonAnimationBehavior : Behavior<Button>
{
    public enum AnimationType
    {
        Scale,
        Bounce,
        Pulse,
        Shake,
        Glow
    }

    public static readonly BindableProperty AnimationTypeProperty =
        BindableProperty.Create(nameof(AnimationType), typeof(AnimationType), typeof(ButtonAnimationBehavior), AnimationType.Scale);

    public AnimationType AnimationType
    {
        get => (AnimationType)GetValue(AnimationTypeProperty);
        set => SetValue(AnimationTypeProperty, value);
    }

    private Button _button;

    protected override void OnAttachedTo(Button bindable)
    {
        _button = bindable;
        _button.Pressed += OnButtonPressed;
        _button.Released += OnButtonReleased;
        _button.Clicked += OnButtonClicked;
        base.OnAttachedTo(bindable);
    }

    protected override void OnDetachingFrom(Button bindable)
    {
        _button.Pressed -= OnButtonPressed;
        _button.Released -= OnButtonReleased;
        _button.Clicked -= OnButtonClicked;
        _button = null;
        base.OnDetachingFrom(bindable);
    }

    private async void OnButtonPressed(object sender, EventArgs e)
    {
        switch (AnimationType)
        {
            case AnimationType.Scale:
                await _button.ScaleTo(0.95, 50, Easing.CubicOut);
                break;
            case AnimationType.Bounce:
                await _button.ScaleTo(0.9, 50, Easing.CubicOut);
                break;
        }
    }

    private async void OnButtonReleased(object sender, EventArgs e)
    {
        switch (AnimationType)
        {
            case AnimationType.Scale:
                await _button.ScaleTo(1.0, 50, Easing.CubicOut);
                break;
            case AnimationType.Bounce:
                await _button.ScaleTo(1.1, 50, Easing.CubicOut);
                await _button.ScaleTo(1.0, 100, Easing.BounceOut);
                break;
        }
    }

    private async void OnButtonClicked(object sender, EventArgs e)
    {
        switch (AnimationType)
        {
            case AnimationType.Pulse:
                await _button.PulseAsync(1.1, 150, 2);
                break;
            case AnimationType.Shake:
                await _button.ShakeAsync(300, 5);
                break;
            case AnimationType.Glow:
                await CreateGlowEffect();
                break;
        }
    }

    private async Task CreateGlowEffect()
    {
        var originalBackground = _button.BackgroundColor;
        var glowColor = Colors.LightBlue;
        
        // Create glow effect by changing background color
        await _button.ColorTo(originalBackground, glowColor, c => _button.BackgroundColor = c, 150);
        await _button.ColorTo(glowColor, originalBackground, c => _button.BackgroundColor = c, 150);
    }
}

// Extension for color animation
public static class ColorAnimationExtension
{
    public static Task<bool> ColorTo(this VisualElement element, Color fromColor, Color toColor, Action<Color> callback, uint duration = 250, Easing easing = null)
    {
        Func<double, Color> transform = (t) =>
            Color.FromRgba(
                fromColor.Red + t * (toColor.Red - fromColor.Red),
                fromColor.Green + t * (toColor.Green - fromColor.Green),
                fromColor.Blue + t * (toColor.Blue - fromColor.Blue),
                fromColor.Alpha + t * (toColor.Alpha - fromColor.Alpha)
            );

        return ColorAnimation(element, "ColorTo", transform, callback, duration, easing);
    }

    private static Task<bool> ColorAnimation(VisualElement element, string name, Func<double, Color> transform, Action<Color> callback, uint duration, Easing easing)
    {
        easing = easing ?? Easing.Linear;
        var taskCompletionSource = new TaskCompletionSource<bool>();

        element.Animate<Color>(name, transform, callback, 16, duration, easing, (v, c) => taskCompletionSource.SetResult(c));
        return taskCompletionSource.Task;
    }
}
```

### 4.2 Loading State Animations

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\11-Animations\Controls\LoadingButton.cs
namespace Animations.Controls;

public class LoadingButton : ContentView
{
    #region Bindable Properties

    public static readonly BindableProperty TextProperty =
        BindableProperty.Create(nameof(Text), typeof(string), typeof(LoadingButton), "Button");

    public static readonly BindableProperty IsLoadingProperty =
        BindableProperty.Create(nameof(IsLoading), typeof(bool), typeof(LoadingButton), false, propertyChanged: OnIsLoadingChanged);

    public static readonly BindableProperty CommandProperty =
        BindableProperty.Create(nameof(Command), typeof(ICommand), typeof(LoadingButton));

    public string Text
    {
        get => (string)GetValue(TextProperty);
        set => SetValue(TextProperty, value);
    }

    public bool IsLoading
    {
        get => (bool)GetValue(IsLoadingProperty);
        set => SetValue(IsLoadingProperty, value);
    }

    public ICommand Command
    {
        get => (ICommand)GetValue(CommandProperty);
        set => SetValue(CommandProperty, value);
    }

    #endregion

    private Frame _buttonFrame;
    private Grid _contentGrid;
    private Label _textLabel;
    private ActivityIndicator _loadingIndicator;
    private StackLayout _dotsContainer;

    public LoadingButton()
    {
        CreateControl();
        BindEvents();
    }

    private void CreateControl()
    {
        _buttonFrame = new Frame
        {
            Padding = new Thickness(20, 12),
            CornerRadius = 8,
            BackgroundColor = Colors.Blue,
            BorderColor = Colors.Transparent,
            HasShadow = false
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

        _loadingIndicator = new ActivityIndicator
        {
            Color = Colors.White,
            IsVisible = false,
            VerticalOptions = LayoutOptions.Center
        };

        _dotsContainer = new StackLayout
        {
            Orientation = StackOrientation.Horizontal,
            Spacing = 4,
            IsVisible = false,
            VerticalOptions = LayoutOptions.Center
        };

        // Create animated dots
        for (int i = 0; i < 3; i++)
        {
            var dot = new Ellipse
            {
                WidthRequest = 6,
                HeightRequest = 6,
                Fill = new SolidColorBrush(Colors.White)
            };
            _dotsContainer.Children.Add(dot);
        }

        _textLabel = new Label
        {
            Text = Text,
            TextColor = Colors.White,
            FontAttributes = FontAttributes.Bold,
            VerticalOptions = LayoutOptions.Center,
            HorizontalOptions = LayoutOptions.Center
        };

        _contentGrid.Children.Add(_loadingIndicator);
        Grid.SetColumn(_loadingIndicator, 0);

        _contentGrid.Children.Add(_dotsContainer);
        Grid.SetColumn(_dotsContainer, 0);

        _contentGrid.Children.Add(_textLabel);
        Grid.SetColumn(_textLabel, 1);

        _buttonFrame.Content = _contentGrid;
        Content = _buttonFrame;

        // Add tap gesture
        var tapGesture = new TapGestureRecognizer();
        tapGesture.Tapped += OnTapped;
        GestureRecognizers.Add(tapGesture);
    }

    private void BindEvents()
    {
        PropertyChanged += (s, e) =>
        {
            if (e.PropertyName == nameof(Text))
            {
                _textLabel.Text = Text;
            }
        };
    }

    private static void OnIsLoadingChanged(BindableObject bindable, object oldValue, object newValue)
    {
        if (bindable is LoadingButton button)
        {
            button.UpdateLoadingState((bool)newValue);
        }
    }

    private async void UpdateLoadingState(bool isLoading)
    {
        if (isLoading)
        {
            await StartLoadingAnimation();
        }
        else
        {
            await StopLoadingAnimation();
        }
    }

    private async Task StartLoadingAnimation()
    {
        // Fade out text and show loading indicator
        await Task.WhenAll(
            _textLabel.FadeTo(0.3, 150),
            _textLabel.ScaleTo(0.9, 150)
        );

        _dotsContainer.IsVisible = true;
        
        // Start dots animation
        _ = AnimateDotsAsync();

        IsEnabled = false;
    }

    private async Task StopLoadingAnimation()
    {
        _dotsContainer.IsVisible = false;
        
        // Fade in text
        await Task.WhenAll(
            _textLabel.FadeTo(1.0, 150),
            _textLabel.ScaleTo(1.0, 150)
        );

        IsEnabled = true;
    }

    private async Task AnimateDotsAsync()
    {
        while (IsLoading && _dotsContainer.IsVisible)
        {
            foreach (var dot in _dotsContainer.Children.OfType<Ellipse>())
            {
                if (!IsLoading) break;
                
                _ = Task.Run(async () =>
                {
                    await Device.InvokeOnMainThreadAsync(async () =>
                    {
                        await dot.ScaleTo(1.5, 200, Easing.CubicOut);
                        await dot.ScaleTo(1.0, 200, Easing.CubicIn);
                    });
                });
                
                await Task.Delay(100);
            }
            
            await Task.Delay(300);
        }
    }

    private async void OnTapped(object sender, EventArgs e)
    {
        if (IsLoading) return;

        // Button press animation
        await this.ScaleTo(0.95, 50);
        await this.ScaleTo(1.0, 50);

        if (Command?.CanExecute(null) == true)
        {
            Command.Execute(null);
        }
    }
}
```

---

## 5. 🎬 Complex Animation Sequences

### 5.1 Staggered List Animation

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\11-Animations\Controls\AnimatedCollectionView.cs
using System.Collections.Specialized;

namespace Animations.Controls;

public class AnimatedCollectionView : CollectionView
{
    public static readonly BindableProperty StaggerDelayProperty =
        BindableProperty.Create(nameof(StaggerDelay), typeof(int), typeof(AnimatedCollectionView), 100);

    public static readonly BindableProperty AnimationTypeProperty =
        BindableProperty.Create(nameof(AnimationType), typeof(ListAnimationType), typeof(AnimatedCollectionView), ListAnimationType.FadeIn);

    public int StaggerDelay
    {
        get => (int)GetValue(StaggerDelayProperty);
        set => SetValue(StaggerDelayProperty, value);
    }

    public ListAnimationType AnimationType
    {
        get => (ListAnimationType)GetValue(AnimationTypeProperty);
        set => SetValue(AnimationTypeProperty, value);
    }

    protected override void OnItemsSourceChanged(IEnumerable oldValue, IEnumerable newValue)
    {
        base.OnItemsSourceChanged(oldValue, newValue);
        
        if (newValue != null)
        {
            Device.BeginInvokeOnMainThread(async () =>
            {
                await Task.Delay(100); // Allow layout to complete
                await AnimateItemsInAsync();
            });
        }
    }

    private async Task AnimateItemsInAsync()
    {
        var items = GetVisibleItems();
        
        // Initially hide all items
        foreach (var item in items)
        {
            await SetInitialState(item);
        }

        // Animate items in with stagger
        for (int i = 0; i < items.Count; i++)
        {
            var item = items[i];
            _ = AnimateItemAsync(item, i);
            
            if (i < items.Count - 1)
            {
                await Task.Delay(StaggerDelay);
            }
        }
    }

    private async Task SetInitialState(VisualElement item)
    {
        switch (AnimationType)
        {
            case ListAnimationType.FadeIn:
                item.Opacity = 0;
                break;
            case ListAnimationType.SlideUp:
                item.Opacity = 0;
                item.TranslationY = 50;
                break;
            case ListAnimationType.ScaleIn:
                item.Opacity = 0;
                item.Scale = 0.8;
                break;
            case ListAnimationType.SlideLeft:
                item.Opacity = 0;
                item.TranslationX = 100;
                break;
        }
    }

    private async Task AnimateItemAsync(VisualElement item, int index)
    {
        switch (AnimationType)
        {
            case ListAnimationType.FadeIn:
                await item.FadeTo(1, 300, Easing.CubicOut);
                break;
                
            case ListAnimationType.SlideUp:
                await Task.WhenAll(
                    item.FadeTo(1, 300, Easing.CubicOut),
                    item.TranslateTo(0, 0, 300, Easing.CubicOut)
                );
                break;
                
            case ListAnimationType.ScaleIn:
                await Task.WhenAll(
                    item.FadeTo(1, 300, Easing.CubicOut),
                    item.ScaleTo(1, 300, Easing.BounceOut)
                );
                break;
                
            case ListAnimationType.SlideLeft:
                await Task.WhenAll(
                    item.FadeTo(1, 300, Easing.CubicOut),
                    item.TranslateTo(0, 0, 300, Easing.CubicOut)
                );
                break;
        }
    }

    private List<VisualElement> GetVisibleItems()
    {
        var items = new List<VisualElement>();
        
        // This is a simplified approach - in a real implementation,
        // you would need to get the actual visible items from the CollectionView
        if (ItemsSource != null)
        {
            foreach (var dataItem in ItemsSource.Cast<object>().Take(10)) // First 10 items as example
            {
                // In practice, you'd get the actual visual elements
                // This is conceptual code
            }
        }
        
        return items;
    }
}

public enum ListAnimationType
{
    FadeIn,
    SlideUp,
    ScaleIn,
    SlideLeft
}
```

### 5.2 Page Load Animation Sequence

```csharp
// d:\A-Work\learn from ai\maui-xaml-mastery\src\11-Animations\Views\AnimatedHomePage.xaml.cs
namespace Animations.Views;

public partial class AnimatedHomePage : ContentPage
{
    public AnimatedHomePage()
    {
        InitializeComponent();
    }

    protected override async void OnAppearing()
    {
        base.OnAppearing();
        await PlayPageLoadAnimation();
    }

    private async Task PlayPageLoadAnimation()
    {
        // Set initial state - all elements hidden
        HeaderLabel.Opacity = 0;
        HeaderLabel.TranslationY = -50;
        
        WelcomeCard.Opacity = 0;
        WelcomeCard.Scale = 0.8;
        
        foreach (var button in ButtonsContainer.Children.OfType<Button>())
        {
            button.Opacity = 0;
            button.TranslationX = 100;
        }

        // Animation sequence
        await Task.Delay(200); // Small delay for page to settle

        // 1. Animate header
        await Task.WhenAll(
            HeaderLabel.FadeTo(1, 400, Easing.CubicOut),
            HeaderLabel.TranslateTo(0, 0, 400, Easing.CubicOut)
        );

        // 2. Animate welcome card
        await Task.WhenAll(
            WelcomeCard.FadeTo(1, 500, Easing.CubicOut),
            WelcomeCard.ScaleTo(1, 500, Easing.BounceOut)
        );

        // 3. Animate buttons with stagger
        var buttons = ButtonsContainer.Children.OfType<Button>().ToList();
        for (int i = 0; i < buttons.Count; i++)
        {
            var button = buttons[i];
            
            _ = Task.WhenAll(
                button.FadeTo(1, 300, Easing.CubicOut),
                button.TranslateTo(0, 0, 300, Easing.CubicOut)
            );
            
            if (i < buttons.Count - 1)
            {
                await Task.Delay(100); // Stagger delay
            }
        }

        // 4. Final flourish - subtle pulse animation
        await Task.Delay(500);
        await WelcomeCard.PulseAsync(1.05, 200, 1);
    }
}
```

---

## 6. 🧪 Practical Exercises

### Exercise 1: Custom Progress Animation
สร้าง animated progress bar พร้อม:
- Smooth progress updates
- Color transitions
- Text animations
- Success/error states

### Exercise 2: Interactive Card Deck
สร้าง card interface ที่มี:
- Swipe animations
- Stack rearrangement
- Physics-based movements
- Spring animations

### Exercise 3: Animated Form Validation
สร้าง form ที่มี:
- Real-time validation animations
- Error state transitions
- Success celebrations
- Input focus effects

---

## 7. 📝 Best Practices & Performance

### 🎯 Animation Performance Tips

```csharp
// ✅ DO: Use hardware acceleration
await element.ScaleTo(1.2, 250, Easing.CubicOut);

// ✅ DO: Batch animations when possible
await Task.WhenAll(
    element.FadeTo(1, 300),
    element.ScaleTo(1, 300),
    element.TranslateTo(0, 0, 300)
);

// ❌ DON'T: Chain too many sequential animations
// This creates janky movement
await element.FadeTo(1, 100);
await element.ScaleTo(1.1, 100);
await element.ScaleTo(1, 100);
await element.TranslateTo(10, 0, 100);

// ✅ DO: Use appropriate durations
// Fast feedback: 100-200ms
// UI transitions: 200-300ms
// Page transitions: 300-500ms
// Attention-getting: 500-1000ms

// ✅ DO: Clean up long-running animations
private CancellationTokenSource _animationCancellation;

private async Task StartContinuousAnimation()
{
    _animationCancellation = new CancellationTokenSource();
    
    try
    {
        while (!_animationCancellation.Token.IsCancellationRequested)
        {
            await element.RotateTo(360, 2000, Easing.Linear);
            element.Rotation = 0;
        }
    }
    catch (OperationCanceledException)
    {
        // Animation was cancelled
    }
}

protected override void OnDisappearing()
{
    _animationCancellation?.Cancel();
    base.OnDisappearing();
}
```

### 🔧 CSS-like Animation Standards

```xml
<!-- Define consistent animation durations like CSS -->
<x:Double x:Key="AnimationFast">150</x:Double>
<x:Double x:Key="AnimationNormal">300</x:Double>
<x:Double x:Key="AnimationSlow">500</x:Double>

<!-- Create reusable animation styles -->
<Style x:Key="FadeInAnimation" TargetType="VisualElement">
    <Style.Triggers>
        <Trigger TargetType="VisualElement" Property="IsVisible" Value="True">
            <Trigger.EnterActions>
                <BeginStoryboard>
                    <Storyboard>
                        <DoubleAnimation 
                            Storyboard.TargetProperty="Opacity"
                            From="0" To="1" 
                            Duration="0:0:0.3" />
                    </Storyboard>
                </BeginStoryboard>
            </Trigger.EnterActions>
        </Trigger>
    </Style.Triggers>
</Style>
```

---

## 8. 🎯 Quiz & Assessment

### Quiz Questions

1. **Animation Performance**: วิธีไหนดีที่สุดในการเพิ่มประสิทธิภาพ animations?

2. **Lottie Integration**: ข้อดีของการใช้ Lottie แทน traditional animations คืออะไร?

3. **Micro-interactions**: หลักการออกแบบ micro-interactions ที่ดีมีอะไรบ้าง?

4. **CSS-like Animations**: จะสร้าง animation system ที่เหมือน CSS ใน MAUI ได้อย่างไร?

### Practical Assessment
สร้าง **"Animated E-commerce App"** ที่มี:
- ✅ Smooth page transitions
- ✅ Product card animations
- ✅ Loading state animations
- ✅ Cart interaction effects
- ✅ Success/error feedback animations

---

## 9. 🚀 Next Steps

### Advanced Topics to Explore
- **Physics-based Animations**: ใช้ spring animations และ realistic physics
- **3D Animations**: สร้าง 3D effects และ transformations
- **Performance Optimization**: Advanced techniques สำหรับ smooth 60fps
- **Custom Animation Engines**: สร้าง animation framework ของตัวเอง

### Chapter 12 Preview
หัวข้อต่อไป: **"Platform-Specific Features"**
- Native platform integrations
- Platform-specific UI patterns
- Hardware feature access
- Performance optimizations per platform

---

## 📚 Additional Resources

### 🔗 Useful Links
- [MAUI Animations Guide](https://docs.microsoft.com/dotnet/maui/user-interface/animation)
- [Lottie for .NET MAUI](https://github.com/Baseflow/LottieXamarin)
- [Animation Design Principles](https://material.io/design/motion)

### 📖 Recommended Reading
- "Animation at Work" by Rachel Nabors
- "UI Animation Microinteractions" 
- "Mobile Animation Design Patterns"

---

*หมายเหตุ: บทนี้เน้นการสร้าง animations ที่สวยงาม, smooth และเพิ่ม user experience โดยใช้เทคนิคสมัยใหม่และ performance optimization*
