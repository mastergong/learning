# 📐 บทที่ 5: Layouts & Controls
> **เชี่ยวชาญการจัดวาง UI และการใช้ Controls ต่างๆ**

---

## 📋 สารบัญ
1. [Layout Fundamentals](#layout-fundamentals)
2. [StackLayout & VerticalStackLayout](#stacklayout-verticalstacklayout)
3. [Grid Layout System](#grid-layout-system)
4. [FlexLayout - CSS Flexbox](#flexlayout-css-flexbox)
5. [AbsoluteLayout & RelativeLayout](#absolutelayout-relativelayout)
6. [Essential Controls](#essential-controls)
7. [Input Controls](#input-controls)
8. [Collection Views](#collection-views)
9. [CSS-like Layout Patterns](#css-like-layout-patterns)

---

## 🎯 Learning Objectives

เมื่อจบบทเรียนนี้ คุณจะสามารถ:
- ✅ เข้าใจและใช้ Layout systems ต่างๆ ใน MAUI
- ✅ สร้าง responsive layouts ด้วย Grid และ FlexLayout
- ✅ ใช้ Controls พื้นฐานและขั้นสูง
- ✅ จัดการ Collections และ Lists อย่างมีประสิทธิภาพ
- ✅ ประยุกต์ใช้ CSS-like patterns ใน XAML
- ✅ สร้าง complex UI layouts ที่ responsive

---

## 🏗️ Layout Fundamentals

### Layout Hierarchy

```xml
<!-- Layout Hierarchy Structure -->
<ContentPage>
    <ScrollView> <!-- Scrollable container -->
        <StackLayout> <!-- Main container -->
            <Grid> <!-- Complex layouts -->
                <FlexLayout> <!-- CSS Flexbox-like -->
                    <Frame> <!-- Visual container -->
                        <Label /> <!-- Content -->
                    </Frame>
                </FlexLayout>
            </Grid>
        </StackLayout>
    </ScrollView>
</ContentPage>
```

### Layout Properties

```xml
<!-- Common Layout Properties -->
<StackLayout
    Spacing="16"                    <!-- Space between children -->
    Padding="20"                    <!-- Internal padding -->
    Margin="10"                     <!-- External margin -->
    HorizontalOptions="FillAndExpand" <!-- Horizontal alignment -->
    VerticalOptions="Center"        <!-- Vertical alignment -->
    BackgroundColor="LightGray">    <!-- Background color -->
    
    <!-- Children here -->
    
</StackLayout>
```

---

## 📚 StackLayout & VerticalStackLayout

### Basic StackLayout

```xml
<!-- Traditional StackLayout -->
<StackLayout Orientation="Vertical" Spacing="12">
    <Label Text="Header" Style="{StaticResource H1}" />
    <Label Text="Subtitle" Style="{StaticResource Body}" />
    <Button Text="Action" Style="{StaticResource PrimaryButton}" />
</StackLayout>

<!-- Horizontal StackLayout -->
<StackLayout Orientation="Horizontal" Spacing="8">
    <Image Source="icon.png" WidthRequest="24" HeightRequest="24" />
    <Label Text="Icon with text" VerticalOptions="Center" />
</StackLayout>
```

### Modern VerticalStackLayout

```xml
<!-- .NET 6+ VerticalStackLayout (Recommended) -->
<VerticalStackLayout Spacing="16" Padding="20">
    <!-- Hero Section -->
    <Frame BackgroundColor="{StaticResource Primary}" CornerRadius="12">
        <VerticalStackLayout Spacing="12" Padding="16">
            <Label 
                Text="Welcome"
                Style="{StaticResource H1}"
                TextColor="White"
                HorizontalOptions="Center" />
            <Label 
                Text="Modern MAUI Application"
                Style="{StaticResource Body}"
                TextColor="White"
                HorizontalOptions="Center" />
        </VerticalStackLayout>
    </Frame>
    
    <!-- Content Cards -->
    <Frame CornerRadius="8" HasShadow="True">
        <VerticalStackLayout Spacing="8" Padding="12">
            <Label Text="Card Title" Style="{StaticResource H2}" />
            <Label Text="Card content goes here..." Style="{StaticResource Body}" />
            <HorizontalStackLayout Spacing="8" HorizontalOptions="End">
                <Button Text="Cancel" Style="{StaticResource SecondaryButton}" />
                <Button Text="Confirm" Style="{StaticResource PrimaryButton}" />
            </HorizontalStackLayout>
        </VerticalStackLayout>
    </Frame>
</VerticalStackLayout>
```

### CSS-like Stack Patterns

```xml
<!-- CSS Flexbox-like behavior -->
<VerticalStackLayout Spacing="0">
    <!-- Header (fixed) -->
    <Frame BackgroundColor="{StaticResource Primary}" CornerRadius="0">
        <Label 
            Text="Header" 
            Padding="16" 
            TextColor="White"
            HorizontalOptions="Center" />
    </Frame>
    
    <!-- Main Content (expandable) -->
    <ScrollView VerticalOptions="FillAndExpand">
        <VerticalStackLayout Spacing="16" Padding="16">
            <!-- Dynamic content -->
            <Label Text="Content 1" />
            <Label Text="Content 2" />
            <Label Text="Content 3" />
        </VerticalStackLayout>
    </ScrollView>
    
    <!-- Footer (fixed) -->
    <Frame BackgroundColor="{StaticResource Gray100}" CornerRadius="0">
        <HorizontalStackLayout Spacing="16" Padding="16" HorizontalOptions="Center">
            <Button Text="Back" Style="{StaticResource SecondaryButton}" />
            <Button Text="Next" Style="{StaticResource PrimaryButton}" />
        </HorizontalStackLayout>
    </Frame>
</VerticalStackLayout>
```

---

## 🎨 Grid Layout System

### Basic Grid Structure

```xml
<!-- CSS Grid-like Layout -->
<Grid>
    <!-- Define columns and rows -->
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="*" />      <!-- Flexible -->
        <ColumnDefinition Width="2*" />     <!-- 2x flexible -->
        <ColumnDefinition Width="100" />    <!-- Fixed -->
        <ColumnDefinition Width="Auto" />   <!-- Content size -->
    </Grid.ColumnDefinitions>
    
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto" />     <!-- Header -->
        <RowDefinition Height="*" />        <!-- Main content -->
        <RowDefinition Height="60" />       <!-- Footer -->
    </Grid.RowDefinitions>
    
    <!-- Place items in grid -->
    <Label Grid.Row="0" Grid.Column="0" Grid.ColumnSpan="4" 
           Text="Header" BackgroundColor="{StaticResource Primary}" />
    
    <Label Grid.Row="1" Grid.Column="0" Text="Sidebar" />
    <Label Grid.Row="1" Grid.Column="1" Text="Main Content" />
    <Label Grid.Row="1" Grid.Column="2" Text="Side Panel" />
    
    <Button Grid.Row="2" Grid.Column="0" Grid.ColumnSpan="4" 
            Text="Footer Button" />
</Grid>
```

### Responsive Dashboard Layout

```xml
<!-- Responsive Dashboard Grid -->
<ScrollView>
    <Grid Padding="16" RowSpacing="16" ColumnSpacing="16">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="*" />
            <ColumnDefinition Width="*" />
            <ColumnDefinition Width="*" />
        </Grid.ColumnDefinitions>
        
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto" />
            <RowDefinition Height="200" />
            <RowDefinition Height="200" />
            <RowDefinition Height="Auto" />
        </Grid.RowDefinitions>
        
        <!-- Header spanning all columns -->
        <Frame Grid.Row="0" Grid.ColumnSpan="3" 
               BackgroundColor="{StaticResource Primary}" 
               CornerRadius="12">
            <VerticalStackLayout Spacing="8" Padding="20">
                <Label Text="Dashboard" 
                       Style="{StaticResource H1}" 
                       TextColor="White" />
                <Label Text="Welcome back, User!" 
                       Style="{StaticResource Body}" 
                       TextColor="White" />
            </VerticalStackLayout>
        </Frame>
        
        <!-- Stats Cards -->
        <Frame Grid.Row="1" Grid.Column="0" CornerRadius="8" HasShadow="True">
            <VerticalStackLayout Spacing="8" Padding="16">
                <Label Text="Users" Style="{StaticResource Body}" />
                <Label Text="1,234" Style="{StaticResource H1}" TextColor="{StaticResource Primary}" />
                <Label Text="+12% this month" FontSize="12" TextColor="{StaticResource Success}" />
            </VerticalStackLayout>
        </Frame>
        
        <Frame Grid.Row="1" Grid.Column="1" CornerRadius="8" HasShadow="True">
            <VerticalStackLayout Spacing="8" Padding="16">
                <Label Text="Revenue" Style="{StaticResource Body}" />
                <Label Text="$45.2K" Style="{StaticResource H1}" TextColor="{StaticResource Primary}" />
                <Label Text="+8% this month" FontSize="12" TextColor="{StaticResource Success}" />
            </VerticalStackLayout>
        </Frame>
        
        <Frame Grid.Row="1" Grid.Column="2" CornerRadius="8" HasShadow="True">
            <VerticalStackLayout Spacing="8" Padding="16">
                <Label Text="Orders" Style="{StaticResource Body}" />
                <Label Text="892" Style="{StaticResource H1}" TextColor="{StaticResource Primary}" />
                <Label Text="-3% this month" FontSize="12" TextColor="{StaticResource Danger}" />
            </VerticalStackLayout>
        </Frame>
        
        <!-- Chart section spanning 2 columns -->
        <Frame Grid.Row="2" Grid.ColumnSpan="2" CornerRadius="8" HasShadow="True">
            <VerticalStackLayout Spacing="12" Padding="16">
                <Label Text="Performance Chart" Style="{StaticResource H2}" />
                <Rectangle Fill="{StaticResource Gray200}" HeightRequest="120" />
                <Label Text="Chart placeholder" 
                       HorizontalOptions="Center" 
                       VerticalOptions="Center" />
            </VerticalStackLayout>
        </Frame>
        
        <!-- Recent Activity -->
        <Frame Grid.Row="2" Grid.Column="2" CornerRadius="8" HasShadow="True">
            <VerticalStackLayout Spacing="8" Padding="16">
                <Label Text="Recent Activity" Style="{StaticResource H2}" />
                <VerticalStackLayout Spacing="4">
                    <Label Text="New user registered" FontSize="12" />
                    <Label Text="Order #1234 completed" FontSize="12" />
                    <Label Text="Payment received" FontSize="12" />
                </VerticalStackLayout>
            </VerticalStackLayout>
        </Frame>
        
        <!-- Actions row -->
        <HorizontalStackLayout Grid.Row="3" Grid.ColumnSpan="3" 
                               Spacing="12" HorizontalOptions="Center">
            <Button Text="Export Data" Style="{StaticResource SecondaryButton}" />
            <Button Text="Generate Report" Style="{StaticResource PrimaryButton}" />
            <Button Text="Settings" Style="{StaticResource SecondaryButton}" />
        </HorizontalStackLayout>
    </Grid>
</ScrollView>
```

### Grid Utilities (CSS-like)

```xml
<!-- CSS Grid-like utility styles -->
<ResourceDictionary>
    <!-- Grid Template Styles -->
    <Style x:Key="grid-cols-1" TargetType="Grid">
        <Setter Property="ColumnDefinitions" Value="*" />
    </Style>
    
    <Style x:Key="grid-cols-2" TargetType="Grid">
        <Setter Property="ColumnDefinitions" Value="*,*" />
    </Style>
    
    <Style x:Key="grid-cols-3" TargetType="Grid">
        <Setter Property="ColumnDefinitions" Value="*,*,*" />
    </Style>
    
    <Style x:Key="grid-cols-4" TargetType="Grid">
        <Setter Property="ColumnDefinitions" Value="*,*,*,*" />
    </Style>
    
    <!-- Gap utilities -->
    <Style x:Key="gap-2" TargetType="Grid">
        <Setter Property="RowSpacing" Value="8" />
        <Setter Property="ColumnSpacing" Value="8" />
    </Style>
    
    <Style x:Key="gap-4" TargetType="Grid">
        <Setter Property="RowSpacing" Value="16" />
        <Setter Property="ColumnSpacing" Value="16" />
    </Style>
</ResourceDictionary>

<!-- Usage -->
<Grid Style="{StaticResource grid-cols-3}" Style="{StaticResource gap-4}">
    <Frame Grid.Column="0">
        <Label Text="Column 1" />
    </Frame>
    <Frame Grid.Column="1">
        <Label Text="Column 2" />
    </Frame>
    <Frame Grid.Column="2">
        <Label Text="Column 3" />
    </Frame>
</Grid>
```

---

## 🔄 FlexLayout - CSS Flexbox

### Basic FlexLayout

```xml
<!-- CSS Flexbox equivalent -->
<FlexLayout 
    Direction="Row"              <!-- flex-direction: row -->
    Wrap="Wrap"                 <!-- flex-wrap: wrap -->
    JustifyContent="SpaceEvenly" <!-- justify-content: space-evenly -->
    AlignItems="Center"         <!-- align-items: center -->
    AlignContent="Start">       <!-- align-content: flex-start -->
    
    <Label Text="Item 1" FlexLayout.Grow="1" />
    <Label Text="Item 2" FlexLayout.Grow="2" />
    <Label Text="Item 3" FlexLayout.Shrink="1" />
</FlexLayout>
```

### Responsive Card Layout

```xml
<!-- Responsive card grid using FlexLayout -->
<ScrollView>
    <FlexLayout 
        Direction="Row" 
        Wrap="Wrap" 
        JustifyContent="SpaceAround"
        Padding="16">
        
        <!-- Card items that wrap responsively -->
        <Frame FlexLayout.Basis="300" 
               FlexLayout.Grow="1" 
               Margin="8" 
               CornerRadius="12" 
               HasShadow="True">
            <VerticalStackLayout Spacing="12" Padding="16">
                <Image Source="product1.jpg" Aspect="AspectFill" HeightRequest="150" />
                <Label Text="Product Name" Style="{StaticResource H2}" />
                <Label Text="Product description here..." Style="{StaticResource Body}" />
                <HorizontalStackLayout Spacing="8" HorizontalOptions="End">
                    <Label Text="$99.99" Style="{StaticResource H2}" TextColor="{StaticResource Primary}" />
                    <Button Text="Buy" Style="{StaticResource PrimaryButton}" />
                </HorizontalStackLayout>
            </VerticalStackLayout>
        </Frame>
        
        <!-- More cards... -->
        <Frame FlexLayout.Basis="300" FlexLayout.Grow="1" Margin="8" CornerRadius="12" HasShadow="True">
            <VerticalStackLayout Spacing="12" Padding="16">
                <Image Source="product2.jpg" Aspect="AspectFill" HeightRequest="150" />
                <Label Text="Another Product" Style="{StaticResource H2}" />
                <Label Text="Another description..." Style="{StaticResource Body}" />
                <HorizontalStackLayout Spacing="8" HorizontalOptions="End">
                    <Label Text="$149.99" Style="{StaticResource H2}" TextColor="{StaticResource Primary}" />
                    <Button Text="Buy" Style="{StaticResource PrimaryButton}" />
                </HorizontalStackLayout>
            </VerticalStackLayout>
        </Frame>
    </FlexLayout>
</ScrollView>
```

### FlexLayout Patterns

```xml
<!-- Navigation Bar with FlexLayout -->
<Frame BackgroundColor="{StaticResource Primary}" CornerRadius="0">
    <FlexLayout 
        Direction="Row" 
        JustifyContent="SpaceBetween" 
        AlignItems="Center"
        Padding="16">
        
        <!-- Logo/Brand -->
        <HorizontalStackLayout Spacing="8">
            <Image Source="logo.png" WidthRequest="32" HeightRequest="32" />
            <Label Text="Brand" 
                   Style="{StaticResource H2}" 
                   TextColor="White" 
                   VerticalOptions="Center" />
        </HorizontalStackLayout>
        
        <!-- Navigation Items (grow to fill space) -->
        <FlexLayout 
            Direction="Row" 
            FlexLayout.Grow="1" 
            JustifyContent="Center" 
            Spacing="20">
            <Button Text="Home" BackgroundColor="Transparent" TextColor="White" />
            <Button Text="About" BackgroundColor="Transparent" TextColor="White" />
            <Button Text="Contact" BackgroundColor="Transparent" TextColor="White" />
        </FlexLayout>
        
        <!-- Actions -->
        <HorizontalStackLayout Spacing="8">
            <Button Text="Login" Style="{StaticResource SecondaryButton}" />
            <Button Text="Sign Up" BackgroundColor="White" TextColor="{StaticResource Primary}" />
        </HorizontalStackLayout>
    </FlexLayout>
</Frame>

<!-- Center content (CSS: justify-content: center, align-items: center) -->
<FlexLayout 
    Direction="Column" 
    JustifyContent="Center" 
    AlignItems="Center" 
    VerticalOptions="FillAndExpand">
    
    <Image Source="empty_state.png" WidthRequest="200" HeightRequest="200" />
    <Label Text="No items found" Style="{StaticResource H2}" />
    <Label Text="Try adjusting your search criteria" Style="{StaticResource Body}" />
    <Button Text="Refresh" Style="{StaticResource PrimaryButton}" Margin="0,20,0,0" />
</FlexLayout>
```

---

## 📍 AbsoluteLayout & RelativeLayout

### AbsoluteLayout

```xml
<!-- Absolute positioning for complex overlays -->
<AbsoluteLayout>
    <!-- Background image -->
    <Image Source="background.jpg" 
           AbsoluteLayout.LayoutBounds="0,0,1,1" 
           AbsoluteLayout.LayoutFlags="All" 
           Aspect="AspectFill" />
    
    <!-- Overlay -->
    <BoxView Color="Black" 
             Opacity="0.5" 
             AbsoluteLayout.LayoutBounds="0,0,1,1" 
             AbsoluteLayout.LayoutFlags="All" />
    
    <!-- Centered content -->
    <VerticalStackLayout 
        AbsoluteLayout.LayoutBounds="0.5,0.5,AutoSize,AutoSize" 
        AbsoluteLayout.LayoutFlags="PositionProportional"
        Spacing="16">
        <Label Text="Welcome" 
               Style="{StaticResource H1}" 
               TextColor="White" 
               HorizontalOptions="Center" />
        <Button Text="Get Started" Style="{StaticResource PrimaryButton}" />
    </VerticalStackLayout>
    
    <!-- Floating action button -->
    <Button Text="+" 
            BackgroundColor="{StaticResource Primary}" 
            TextColor="White" 
            CornerRadius="28" 
            WidthRequest="56" 
            HeightRequest="56" 
            AbsoluteLayout.LayoutBounds="0.9,0.9,AutoSize,AutoSize" 
            AbsoluteLayout.LayoutFlags="PositionProportional" />
</AbsoluteLayout>
```

---

## 🎛️ Essential Controls

### Text Controls

```xml
<!-- Text Display Controls -->
<VerticalStackLayout Spacing="16" Padding="20">
    <!-- Labels with different styles -->
    <Label Text="Main Heading" Style="{StaticResource H1}" />
    <Label Text="Subheading" Style="{StaticResource H2}" />
    <Label Text="Body text with multiple lines. This demonstrates how text wraps in a Label control." 
           Style="{StaticResource Body}" />
    
    <!-- Formatted text -->
    <Label>
        <Label.FormattedText>
            <FormattedString>
                <Span Text="Bold " FontAttributes="Bold" />
                <Span Text="Italic " FontAttributes="Italic" />
                <Span Text="Colored" TextColor="{StaticResource Primary}" />
            </FormattedString>
        </Label.FormattedText>
    </Label>
    
    <!-- Hyperlink-like label -->
    <Label Text="Learn more" 
           TextColor="{StaticResource Primary}" 
           TextDecorations="Underline">
        <Label.GestureRecognizers>
            <TapGestureRecognizer Tapped="OnLearnMoreTapped" />
        </Label.GestureRecognizers>
    </Label>
</VerticalStackLayout>
```

### Button Variants

```xml
<!-- Button Styles and Variants -->
<VerticalStackLayout Spacing="12" Padding="20">
    <!-- Primary buttons -->
    <Button Text="Primary Button" Style="{StaticResource PrimaryButton}" />
    
    <!-- Secondary buttons -->
    <Button Text="Secondary Button" Style="{StaticResource SecondaryButton}" />
    
    <!-- Icon buttons -->
    <Button>
        <Button.ContentLayout>
            <Button.ContentLayout>
                <FontImageSource Glyph="&#xE109;" 
                                 FontFamily="FluentSystemIcons" 
                                 Color="White" />
            </Button.ContentLayout>
        </Button.ContentLayout>
    </Button>
    
    <!-- Custom styled button -->
    <Button Text="Custom Button">
        <Button.Style>
            <Style TargetType="Button">
                <Setter Property="BackgroundColor" Value="{StaticResource Success}" />
                <Setter Property="TextColor" Value="White" />
                <Setter Property="CornerRadius" Value="25" />
                <Setter Property="Padding" Value="20,12" />
                <Setter Property="FontAttributes" Value="Bold" />
                <Setter Property="Shadow">
                    <Shadow Brush="Gray" Offset="2,2" Radius="5" Opacity="0.3" />
                </Setter>
            </Style>
        </Button.Style>
    </Button>
    
    <!-- Button group -->
    <HorizontalStackLayout Spacing="8" HorizontalOptions="Center">
        <Button Text="Option 1" Style="{StaticResource SecondaryButton}" />
        <Button Text="Option 2" Style="{StaticResource PrimaryButton}" />
        <Button Text="Option 3" Style="{StaticResource SecondaryButton}" />
    </HorizontalStackLayout>
</VerticalStackLayout>
```

### Image Controls

```xml
<!-- Image Controls and Variants -->
<VerticalStackLayout Spacing="16" Padding="20">
    <!-- Basic image -->
    <Image Source="sample.jpg" 
           Aspect="AspectFit" 
           HeightRequest="200" />
    
    <!-- Circular image (profile picture) -->
    <Frame CornerRadius="50" 
           WidthRequest="100" 
           HeightRequest="100" 
           IsClippedToBounds="True" 
           HorizontalOptions="Center">
        <Image Source="profile.jpg" Aspect="AspectFill" />
    </Frame>
    
    <!-- Image with overlay -->
    <Grid>
        <Image Source="banner.jpg" Aspect="AspectFill" HeightRequest="200" />
        <BoxView Color="Black" Opacity="0.4" />
        <Label Text="Image Overlay Text" 
               TextColor="White" 
               Style="{StaticResource H2}" 
               HorizontalOptions="Center" 
               VerticalOptions="Center" />
    </Grid>
    
    <!-- Icon using FontImageSource -->
    <Image WidthRequest="24" HeightRequest="24">
        <Image.Source>
            <FontImageSource Glyph="&#xE115;" 
                             FontFamily="FluentSystemIcons" 
                             Color="{StaticResource Primary}" />
        </Image.Source>
    </Image>
</VerticalStackLayout>
```

---

## 📝 Input Controls

### Entry and Editor

```xml
<!-- Input Controls -->
<VerticalStackLayout Spacing="16" Padding="20">
    <!-- Basic Entry -->
    <VerticalStackLayout Spacing="4">
        <Label Text="Email Address" Style="{StaticResource Body}" />
        <Entry Placeholder="Enter your email" 
               Keyboard="Email" 
               Text="{Binding Email}" />
    </VerticalStackLayout>
    
    <!-- Password Entry -->
    <VerticalStackLayout Spacing="4">
        <Label Text="Password" Style="{StaticResource Body}" />
        <Entry Placeholder="Enter password" 
               IsPassword="True" 
               Text="{Binding Password}" />
    </VerticalStackLayout>
    
    <!-- Styled Entry -->
    <VerticalStackLayout Spacing="4">
        <Label Text="Search" Style="{StaticResource Body}" />
        <Frame BackgroundColor="{StaticResource Gray100}" 
               CornerRadius="8" 
               Padding="0" 
               HasShadow="False">
            <Entry Placeholder="Search..." 
                   BackgroundColor="Transparent" 
                   BorderWidth="0" />
        </Frame>
    </VerticalStackLayout>
    
    <!-- Multi-line Editor -->
    <VerticalStackLayout Spacing="4">
        <Label Text="Comments" Style="{StaticResource Body}" />
        <Frame BackgroundColor="{StaticResource Gray100}" 
               CornerRadius="8" 
               Padding="8" 
               HasShadow="False">
            <Editor Placeholder="Enter your comments..." 
                    HeightRequest="100" 
                    BackgroundColor="Transparent" 
                    Text="{Binding Comments}" />
        </Frame>
    </VerticalStackLayout>
</VerticalStackLayout>
```

### Picker and DatePicker

```xml
<!-- Selection Controls -->
<VerticalStackLayout Spacing="16" Padding="20">
    <!-- Picker -->
    <VerticalStackLayout Spacing="4">
        <Label Text="Select Country" Style="{StaticResource Body}" />
        <Picker Title="Choose a country" 
                ItemsSource="{Binding Countries}" 
                SelectedItem="{Binding SelectedCountry}" />
    </VerticalStackLayout>
    
    <!-- DatePicker -->
    <VerticalStackLayout Spacing="4">
        <Label Text="Birth Date" Style="{StaticResource Body}" />
        <DatePicker Date="{Binding BirthDate}" 
                    MinimumDate="1900-01-01" 
                    MaximumDate="{x:Static sys:DateTime.Now}" />
    </VerticalStackLayout>
    
    <!-- TimePicker -->
    <VerticalStackLayout Spacing="4">
        <Label Text="Appointment Time" Style="{StaticResource Body}" />
        <TimePicker Time="{Binding AppointmentTime}" />
    </VerticalStackLayout>
    
    <!-- Switch -->
    <HorizontalStackLayout Spacing="12">
        <Switch IsToggled="{Binding NotificationsEnabled}" />
        <Label Text="Enable Notifications" 
               Style="{StaticResource Body}" 
               VerticalOptions="Center" />
    </HorizontalStackLayout>
    
    <!-- Slider -->
    <VerticalStackLayout Spacing="4">
        <Label Text="{Binding Volume, StringFormat='Volume: {0:F0}%'}" 
               Style="{StaticResource Body}" />
        <Slider Minimum="0" 
                Maximum="100" 
                Value="{Binding Volume}" 
                ThumbColor="{StaticResource Primary}" 
                MinimumTrackColor="{StaticResource Primary}" />
    </VerticalStackLayout>
    
    <!-- Stepper -->
    <HorizontalStackLayout Spacing="12">
        <Label Text="Quantity:" Style="{StaticResource Body}" VerticalOptions="Center" />
        <Stepper Minimum="1" 
                 Maximum="10" 
                 Value="{Binding Quantity}" 
                 Increment="1" />
        <Label Text="{Binding Quantity}" 
               Style="{StaticResource Body}" 
               VerticalOptions="Center" />
    </HorizontalStackLayout>
</VerticalStackLayout>
```

---

## 📋 Collection Views

### Basic CollectionView

```xml
<!-- Modern CollectionView -->
<CollectionView ItemsSource="{Binding Products}" 
                SelectionMode="Single" 
                SelectedItem="{Binding SelectedProduct}">
    
    <!-- Item Template -->
    <CollectionView.ItemTemplate>
        <DataTemplate x:DataType="models:Product">
            <Frame Margin="8" 
                   CornerRadius="12" 
                   HasShadow="True" 
                   BackgroundColor="White">
                <Grid Padding="12" ColumnDefinitions="Auto,*,Auto" ColumnSpacing="12">
                    <!-- Product Image -->
                    <Frame Grid.Column="0" 
                           CornerRadius="8" 
                           WidthRequest="60" 
                           HeightRequest="60" 
                           IsClippedToBounds="True">
                        <Image Source="{Binding ImageUrl}" Aspect="AspectFill" />
                    </Frame>
                    
                    <!-- Product Info -->
                    <VerticalStackLayout Grid.Column="1" Spacing="4" VerticalOptions="Center">
                        <Label Text="{Binding Name}" Style="{StaticResource H3}" />
                        <Label Text="{Binding Description}" 
                               Style="{StaticResource Body}" 
                               LineBreakMode="TailTruncation" />
                        <Label Text="{Binding Price, StringFormat='{0:C}'}" 
                               Style="{StaticResource H3}" 
                               TextColor="{StaticResource Primary}" />
                    </VerticalStackLayout>
                    
                    <!-- Actions -->
                    <Button Grid.Column="2" 
                            Text="Add" 
                            Style="{StaticResource PrimaryButton}" 
                            Command="{Binding Source={RelativeSource AncestorType={x:Type viewmodels:MainViewModel}}, Path=AddToCartCommand}" 
                            CommandParameter="{Binding .}" />
                </Grid>
            </Frame>
        </DataTemplate>
    </CollectionView.ItemTemplate>
    
    <!-- Empty View -->
    <CollectionView.EmptyView>
        <VerticalStackLayout Spacing="16" HorizontalOptions="Center" VerticalOptions="Center">
            <Image Source="empty_cart.png" WidthRequest="100" HeightRequest="100" />
            <Label Text="No products found" Style="{StaticResource H2}" />
            <Label Text="Try adjusting your search" Style="{StaticResource Body}" />
            <Button Text="Refresh" Style="{StaticResource PrimaryButton}" />
        </VerticalStackLayout>
    </CollectionView.EmptyView>
    
    <!-- Header -->
    <CollectionView.Header>
        <Frame BackgroundColor="{StaticResource Primary}" CornerRadius="0">
            <Label Text="Featured Products" 
                   Style="{StaticResource H2}" 
                   TextColor="White" 
                   Padding="16" />
        </Frame>
    </CollectionView.Header>
    
    <!-- Footer -->
    <CollectionView.Footer>
        <Button Text="Load More" 
                Style="{StaticResource SecondaryButton}" 
                Margin="16" 
                Command="{Binding LoadMoreCommand}" />
    </CollectionView.Footer>
</CollectionView>
```

### Grid Layout CollectionView

```xml
<!-- Grid layout for image gallery -->
<CollectionView ItemsSource="{Binding Photos}">
    <CollectionView.ItemsLayout>
        <GridItemsLayout Orientation="Vertical" Span="2" VerticalItemSpacing="8" HorizontalItemSpacing="8" />
    </CollectionView.ItemsLayout>
    
    <CollectionView.ItemTemplate>
        <DataTemplate x:DataType="models:Photo">
            <Frame CornerRadius="12" IsClippedToBounds="True" HasShadow="True">
                <Grid>
                    <Image Source="{Binding Url}" Aspect="AspectFill" HeightRequest="150" />
                    <BoxView Color="Black" Opacity="0.3" />
                    <Label Text="{Binding Title}" 
                           TextColor="White" 
                           FontAttributes="Bold" 
                           Margin="8" 
                           VerticalOptions="End" />
                </Grid>
            </Frame>
        </DataTemplate>
    </CollectionView.ItemTemplate>
</CollectionView>
```

---

## 🎨 CSS-like Layout Patterns

### Container Patterns

```xml
<!-- CSS Container Pattern -->
<ResourceDictionary>
    <!-- Container utilities -->
    <Style x:Key="container" TargetType="StackLayout">
        <Setter Property="Padding" Value="16" />
        <Setter Property="Spacing" Value="16" />
    </Style>
    
    <Style x:Key="container-fluid" TargetType="StackLayout">
        <Setter Property="Padding" Value="8" />
        <Setter Property="Spacing" Value="8" />
    </Style>
    
    <!-- Card pattern -->
    <Style x:Key="card" TargetType="Frame">
        <Setter Property="BackgroundColor" Value="White" />
        <Setter Property="CornerRadius" Value="12" />
        <Setter Property="HasShadow" Value="True" />
        <Setter Property="Padding" Value="16" />
        <Setter Property="Margin" Value="8" />
    </Style>
    
    <!-- Flexbox patterns -->
    <Style x:Key="flex-row" TargetType="FlexLayout">
        <Setter Property="Direction" Value="Row" />
    </Style>
    
    <Style x:Key="flex-column" TargetType="FlexLayout">
        <Setter Property="Direction" Value="Column" />
    </Style>
    
    <Style x:Key="justify-center" TargetType="FlexLayout">
        <Setter Property="JustifyContent" Value="Center" />
    </Style>
    
    <Style x:Key="items-center" TargetType="FlexLayout">
        <Setter Property="AlignItems" Value="Center" />
    </Style>
</ResourceDictionary>
```

### Component Patterns

```xml
<!-- Reusable component patterns -->
<ContentView x:Class="Components.ProductCard">
    <Frame Style="{StaticResource card}">
        <VerticalStackLayout Spacing="12">
            <Frame CornerRadius="8" IsClippedToBounds="True" HeightRequest="120">
                <Image Source="{Binding ImageUrl}" Aspect="AspectFill" />
            </Frame>
            
            <Label Text="{Binding Name}" Style="{StaticResource H3}" />
            <Label Text="{Binding Description}" 
                   Style="{StaticResource Body}" 
                   LineBreakMode="TailTruncation" 
                   MaxLines="2" />
            
            <HorizontalStackLayout Spacing="8" HorizontalOptions="SpaceBetween">
                <Label Text="{Binding Price, StringFormat='{0:C}'}" 
                       Style="{StaticResource H3}" 
                       TextColor="{StaticResource Primary}" 
                       VerticalOptions="Center" />
                <Button Text="Add to Cart" 
                        Style="{StaticResource PrimaryButton}" 
                        Command="{Binding AddCommand}" />
            </HorizontalStackLayout>
        </VerticalStackLayout>
    </Frame>
</ContentView>

<!-- Usage of component -->
<CollectionView ItemsSource="{Binding Products}">
    <CollectionView.ItemTemplate>
        <DataTemplate>
            <components:ProductCard BindingContext="{Binding .}" />
        </DataTemplate>
    </CollectionView.ItemTemplate>
</CollectionView>
```

---

## 🎯 Exercises

### Exercise 1: Responsive Dashboard
สร้าง responsive dashboard ที่มี:
- Header section
- 4 stats cards ใน grid layout
- Chart section
- Recent activity list

### Exercise 2: Product Gallery
สร้าง product gallery ด้วย:
- CollectionView with grid layout
- Product cards พร้อม image และ info
- Search และ filter functionality
- Empty state

### Exercise 3: Form Layout
สร้าง registration form ที่มี:
- StackLayout สำหรับ form fields
- Input validation
- Submit button
- Loading states

---

## 📝 Quiz

1. **Grid Layout**: Grid.ColumnDefinitions="*,2*,100" หมายความว่าอย่างไร?
2. **FlexLayout**: JustifyContent="SpaceBetween" ทำหน้าที่อะไร?
3. **CollectionView**: ItemsLayout ใช้สำหรับอะไร?

<details>
<summary>เฉลย</summary>

1. Column แรกขนาด flexible, column ที่ 2 ขนาด 2 เท่าของ column แรก, column ที่ 3 ขนาดคงที่ 100 units
2. จัดวาง items โดยให้ space ที่เหลือกระจายระหว่าง items
3. กำหนดวิธีการจัดวาง items ใน collection (เช่น Grid, Linear)

</details>

---

## 🎉 สรุป

ในบทเรียนนี้ คุณได้เรียนรู้:

✅ **Layout Systems** - StackLayout, Grid, FlexLayout, AbsoluteLayout  
✅ **Essential Controls** - Labels, Buttons, Images, Input controls  
✅ **Collection Views** - CollectionView patterns และ customization  
✅ **CSS-like Patterns** - Utility classes และ responsive design  
✅ **Component Architecture** - การสร้าง reusable components

### Next Steps
🔜 **บทที่ 6**: Data Binding & MVVM - ลงลึกใน data binding patterns  
🔜 **บทที่ 7**: Styling & Theming - การจัดรูปแบบและธีมขั้นสูง  
🔜 **บทที่ 8**: Navigation Patterns - การนำทางแบบต่างๆ

---

**Happy Coding! 🚀**

> บทเรียนต่อไป: [06-data-binding-mvvm.md](06-data-binding-mvvm.md)
