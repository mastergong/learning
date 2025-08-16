# MAUI XAML Fundamentals: การทำงานกับ Grid และ FlexLayout
## การจัดการ Layout แบบยืดหยุ่นด้วย Grid และ FlexLayout

### 🎯 จุดประสงค์การเรียนรู้

เมื่อเสร็จสิ้นบทเรียนนี้ คุณจะสามารถ:
- เข้าใจหลักการทำงานของ Grid Layout และ FlexLayout อย่างลึกซึ้ง
- สร้างและกำหนดค่า Grid ด้วย Row และ Column Definitions
- ใช้งาน Grid.Row, Grid.Column, Grid.RowSpan และ Grid.ColumnSpan อย่างถูกต้อง
- สร้าง Responsive Layout ด้วย Star Sizing และ Grid Units
- ใช้งาน FlexLayout สำหรับ Flexible UI
- ผสมผสาน Grid และ FlexLayout เพื่อสร้าง UI ที่ซับซ้อน

## 📋 Grid Layout ใน MAUI

### **Grid คืออะไร?**

Grid เป็น Layout Container ที่จัดเรียง UI Elements ในรูปแบบตาราง (rows และ columns) ทำให้สามารถจัดวาง elements ในตำแหน่งที่ต้องการได้อย่างแม่นยำ

```xml
<Grid>
    <!-- Elements inside Grid -->
</Grid>
```

### **การกำหนดแถวและคอลัมน์**

Grid ใน MAUI สามารถกำหนด rows และ columns ได้ด้วย `RowDefinitions` และ `ColumnDefinitions`

#### **Explicit Row/Column Definitions**

```xml
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="50" />           <!-- Absolute height (50 device-independent units) -->
        <RowDefinition Height="Auto" />         <!-- Auto height (sized to content) -->
        <RowDefinition Height="*" />            <!-- Star/Proportional height (takes remaining space) -->
        <RowDefinition Height="2*" />           <!-- Double proportion (takes twice as much space as *) -->
    </Grid.RowDefinitions>
    
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="100" />        <!-- Absolute width -->
        <ColumnDefinition Width="Auto" />       <!-- Auto width -->
        <ColumnDefinition Width="*" />          <!-- Star/Proportional width -->
        <ColumnDefinition Width="3*" />         <!-- Triple proportion -->
    </Grid.ColumnDefinitions>
    
    <!-- Grid content goes here -->
</Grid>
```

#### **Concise RowDefinitions/ColumnDefinitions**

```xml
<!-- Concise syntax with strings -->
<Grid
    RowDefinitions="50,Auto,*,2*"
    ColumnDefinitions="100,Auto,*,3*">
    
    <!-- Grid content goes here -->
</Grid>
```

### **จัดวางองค์ประกอบใน Grid**

ใช้ attached properties เพื่อระบุตำแหน่งขององค์ประกอบใน Grid

#### **Grid Attached Properties**

```xml
<Grid RowDefinitions="Auto,*,Auto"
      ColumnDefinitions="*,2*,*">
      
    <!-- Row 0, Column 0 (implicit) -->
    <Label Text="Top Left" />
    
    <!-- Row 0, Column 1 -->
    <Label Text="Top Center" 
           Grid.Column="1" />
    
    <!-- Row 0, Column 2 -->
    <Label Text="Top Right" 
           Grid.Column="2" />
    
    <!-- Row 1, Column 0 -->
    <Label Text="Middle Left" 
           Grid.Row="1" />
    
    <!-- Row 1, Column 1 -->
    <Label Text="Center" 
           Grid.Row="1" 
           Grid.Column="1" />
    
    <!-- Row 1, Column 2 -->
    <Label Text="Middle Right" 
           Grid.Row="1" 
           Grid.Column="2" />
    
    <!-- Row 2, Column 0 -->
    <Label Text="Bottom Left" 
           Grid.Row="2" />
    
    <!-- Row 2, Column 1 -->
    <Label Text="Bottom Center" 
           Grid.Row="2" 
           Grid.Column="1" />
    
    <!-- Row 2, Column 2 -->
    <Label Text="Bottom Right" 
           Grid.Row="2" 
           Grid.Column="2" />
</Grid>
```

### **การขยายองค์ประกอบให้ครอบคลุมหลาย Cells**

ใช้ `Grid.RowSpan` และ `Grid.ColumnSpan` เพื่อให้ element ครอบคลุมหลาย cells

```xml
<Grid RowDefinitions="Auto,*,Auto"
      ColumnDefinitions="*,2*,*">
      
    <!-- Spans across all 3 columns in the first row -->
    <Label Text="Header" 
           Grid.ColumnSpan="3" 
           HorizontalOptions="Center" />
    
    <!-- Spans across the first 2 columns in the second row -->
    <Image Source="banner.jpg"
           Aspect="AspectFill"
           Grid.Row="1"
           Grid.ColumnSpan="2" />
    
    <!-- Spans across 2 rows in the last column -->
    <BoxView Color="LightBlue"
             Grid.Row="1"
             Grid.Column="2"
             Grid.RowSpan="2" />
    
    <!-- Spans across the first 2 columns in the last row -->
    <Button Text="Submit"
            Grid.Row="2"
            Grid.ColumnSpan="2" />
</Grid>
```

### **Padding และ Spacing**

```xml
<Grid RowDefinitions="Auto,*,Auto"
      ColumnDefinitions="*,2*,*"
      Padding="20"                   <!-- Padding around the entire Grid -->
      RowSpacing="10"                <!-- Space between rows -->
      ColumnSpacing="15">            <!-- Space between columns -->
    
    <!-- Grid content goes here -->
</Grid>
```

## 🧩 Advanced Grid Techniques

### **Nested Grids**

คุณสามารถซ้อน Grid ใน Grid เพื่อสร้าง layouts ที่ซับซ้อน

```xml
<Grid RowDefinitions="Auto,*,Auto"
      ColumnDefinitions="*,*">
      
    <!-- Outer Grid Header -->
    <Label Text="Main Header" 
           Grid.ColumnSpan="2" 
           HorizontalOptions="Center" />
    
    <!-- Nested Grid in Cell (1,0) -->
    <Grid Grid.Row="1" Grid.Column="0"
          RowDefinitions="*,*"
          ColumnDefinitions="*,*"
          Margin="5">
          
        <Label Text="Nested (0,0)" />
        <Label Text="Nested (0,1)" Grid.Column="1" />
        <Label Text="Nested (1,0)" Grid.Row="1" />
        <Label Text="Nested (1,1)" Grid.Row="1" Grid.Column="1" />
    </Grid>
    
    <!-- Content in Cell (1,1) -->
    <StackLayout Grid.Row="1" Grid.Column="1">
        <Label Text="Regular content" />
        <Button Text="Click me" />
    </StackLayout>
    
    <!-- Outer Grid Footer -->
    <Label Text="Footer" 
           Grid.Row="2" 
           Grid.ColumnSpan="2" 
           HorizontalOptions="Center" />
</Grid>
```

### **Responsive Design ด้วย Star Sizing**

ใช้ Star Sizing (`*`) เพื่อสร้าง responsive layouts

```xml
<Grid RowDefinitions="Auto,*,Auto"
      ColumnDefinitions="1*,2*,1*">
      
    <!-- Header -->
    <Label Text="Responsive Header" 
           Grid.ColumnSpan="3" 
           HorizontalOptions="Center" />
    
    <!-- Sidebar -->
    <StackLayout Grid.Row="1" Grid.Column="0"
                 BackgroundColor="LightGray">
        <Label Text="Sidebar" />
        <Button Text="Menu 1" />
        <Button Text="Menu 2" />
        <Button Text="Menu 3" />
    </StackLayout>
    
    <!-- Main Content - Takes twice as much space -->
    <ScrollView Grid.Row="1" Grid.Column="1">
        <StackLayout>
            <Label Text="Main Content Area" FontSize="Large" />
            <Label Text="This area takes up twice as much space as the sidebars because of the 2* column definition." />
            <!-- More content -->
        </StackLayout>
    </ScrollView>
    
    <!-- Right Sidebar -->
    <StackLayout Grid.Row="1" Grid.Column="2"
                 BackgroundColor="LightGreen">
        <Label Text="Right Sidebar" />
        <BoxView Color="Green" HeightRequest="50" />
        <BoxView Color="Blue" HeightRequest="50" />
    </StackLayout>
    
    <!-- Footer -->
    <Label Text="Footer" 
           Grid.Row="2" 
           Grid.ColumnSpan="3" 
           HorizontalOptions="Center" />
</Grid>
```

### **ปรับแต่ง Grid สำหรับ Orientation Changes**

```csharp
// ในไฟล์ Code-behind
protected override void OnSizeAllocated(double width, double height)
{
    base.OnSizeAllocated(width, height);
    
    // ตรวจสอบ orientation ของอุปกรณ์
    bool isLandscape = width > height;
    
    if (isLandscape)
    {
        // ปรับ layout สำหรับ landscape mode
        MainGrid.RowDefinitions = new RowDefinitionCollection
        {
            new RowDefinition { Height = new GridLength(1, GridUnitType.Star) }
        };
        
        MainGrid.ColumnDefinitions = new ColumnDefinitionCollection
        {
            new ColumnDefinition { Width = new GridLength(1, GridUnitType.Star) },
            new ColumnDefinition { Width = new GridLength(2, GridUnitType.Star) }
        };
        
        // ปรับตำแหน่งองค์ประกอบ
        Grid.SetRow(NavigationPanel, 0);
        Grid.SetColumn(NavigationPanel, 0);
        
        Grid.SetRow(ContentArea, 0);
        Grid.SetColumn(ContentArea, 1);
    }
    else
    {
        // ปรับ layout สำหรับ portrait mode
        MainGrid.RowDefinitions = new RowDefinitionCollection
        {
            new RowDefinition { Height = new GridLength(1, GridUnitType.Auto) },
            new RowDefinition { Height = new GridLength(1, GridUnitType.Star) }
        };
        
        MainGrid.ColumnDefinitions = new ColumnDefinitionCollection
        {
            new ColumnDefinition { Width = new GridLength(1, GridUnitType.Star) }
        };
        
        // ปรับตำแหน่งองค์ประกอบ
        Grid.SetRow(NavigationPanel, 0);
        Grid.SetColumn(NavigationPanel, 0);
        
        Grid.SetRow(ContentArea, 1);
        Grid.SetColumn(ContentArea, 0);
    }
}
```

### **Absolute vs Auto vs Star Sizing**

เลือกใช้ sizing mode ที่เหมาะสม:

- **Absolute** (เช่น `50`) - กำหนดขนาดแน่นอน (device-independent units)
  - ใช้เมื่อ: ต้องการขนาดที่แน่นอนไม่ว่าอุปกรณ์จะมีขนาดเท่าใด
  - ตัวอย่าง: ส่วนหัวที่มีความสูงคงที่

- **Auto** - ขนาดจะปรับตามเนื้อหา
  - ใช้เมื่อ: ต้องการให้ row/column พอดีกับเนื้อหาภายใน
  - ตัวอย่าง: แถวที่มีข้อความที่ความยาวไม่แน่นอน

- **Star** (`*`) - ขนาดเป็นสัดส่วนของพื้นที่ที่เหลือ
  - ใช้เมื่อ: ต้องการให้ element ขยายหรือหดตัวตามพื้นที่ที่มี
  - ตัวอย่าง: พื้นที่แสดงเนื้อหาหลักที่ควรใช้พื้นที่ให้มากที่สุด

```xml
<!-- Example showing all three sizing modes -->
<Grid RowDefinitions="60,Auto,*"
      ColumnDefinitions="100,Auto,*,2*">
    
    <!-- Fixed header area (60 units tall) -->
    <BoxView Grid.Row="0" Grid.ColumnSpan="4" Color="Navy" />
    
    <!-- Auto-sized row based on content -->
    <Label Grid.Row="1" Grid.Column="0" Text="Fixed width column" />
    <Label Grid.Row="1" Grid.Column="1" Text="This column will adjust to fit this text" />
    <Label Grid.Row="1" Grid.Column="2" Text="Flexible column" />
    <Label Grid.Row="1" Grid.Column="3" Text="This column takes twice the space of column 2" />
    
    <!-- Star-sized row that takes remaining space -->
    <BoxView Grid.Row="2" Grid.Column="0" Color="LightGray" />
    <BoxView Grid.Row="2" Grid.Column="1" Color="LightBlue" />
    <BoxView Grid.Row="2" Grid.Column="2" Color="LightGreen" />
    <BoxView Grid.Row="2" Grid.Column="3" Color="LightPink" />
</Grid>
```

## 🌟 FlexLayout ใน MAUI

### **FlexLayout คืออะไร?**

FlexLayout เป็นการนำแนวคิด CSS Flexbox มาใช้ใน MAUI ทำให้สามารถจัดวาง elements แบบยืดหยุ่นตามแกนหลัก (Main Axis) และแกนรอง (Cross Axis)

```xml
<FlexLayout>
    <!-- Elements inside FlexLayout -->
</FlexLayout>
```

### **FlexLayout Properties หลัก**

```xml
<FlexLayout Direction="Row"                  <!-- Main axis direction -->
           Wrap="Wrap"                        <!-- Whether items can wrap to next line -->
           JustifyContent="SpaceEvenly"       <!-- Main axis alignment -->
           AlignItems="Center"                <!-- Cross axis alignment -->
           AlignContent="SpaceAround">        <!-- Alignment when multiple lines -->
    
    <!-- FlexLayout content goes here -->
</FlexLayout>
```

### **Direction Property**

FlexLayout มี 4 ทิศทางพื้นฐาน:

```xml
<FlexLayout Direction="Row">                  <!-- Left to Right (Default) -->
    <Label Text="Item 1" />
    <Label Text="Item 2" />
    <Label Text="Item 3" />
</FlexLayout>

<FlexLayout Direction="RowReverse">           <!-- Right to Left -->
    <Label Text="Item 1" />
    <Label Text="Item 2" />
    <Label Text="Item 3" />
</FlexLayout>

<FlexLayout Direction="Column">               <!-- Top to Bottom -->
    <Label Text="Item 1" />
    <Label Text="Item 2" />
    <Label Text="Item 3" />
</FlexLayout>

<FlexLayout Direction="ColumnReverse">        <!-- Bottom to Top -->
    <Label Text="Item 1" />
    <Label Text="Item 2" />
    <Label Text="Item 3" />
</FlexLayout>
```

### **JustifyContent Property**

ควบคุมการจัดวาง items ตามแกนหลัก (main axis):

```xml
<FlexLayout Direction="Row" JustifyContent="Start">
    <!-- Items aligned to start -->
</FlexLayout>

<FlexLayout Direction="Row" JustifyContent="Center">
    <!-- Items centered -->
</FlexLayout>

<FlexLayout Direction="Row" JustifyContent="End">
    <!-- Items aligned to end -->
</FlexLayout>

<FlexLayout Direction="Row" JustifyContent="SpaceBetween">
    <!-- Equal space between items, no space at edges -->
</FlexLayout>

<FlexLayout Direction="Row" JustifyContent="SpaceAround">
    <!-- Equal space around items, half space at edges -->
</FlexLayout>

<FlexLayout Direction="Row" JustifyContent="SpaceEvenly">
    <!-- Equal space between and around items -->
</FlexLayout>
```

### **AlignItems Property**

ควบคุมการจัดวาง items ตามแกนรอง (cross axis):

```xml
<FlexLayout Direction="Row" AlignItems="Start">
    <!-- Items aligned to start of cross axis -->
</FlexLayout>

<FlexLayout Direction="Row" AlignItems="Center">
    <!-- Items centered on cross axis -->
</FlexLayout>

<FlexLayout Direction="Row" AlignItems="End">
    <!-- Items aligned to end of cross axis -->
</FlexLayout>

<FlexLayout Direction="Row" AlignItems="Stretch">
    <!-- Items stretched to fill cross axis -->
</FlexLayout>
```

### **การใช้ FlexLayout กับ Wrap Property**

```xml
<FlexLayout Direction="Row" 
           Wrap="Wrap" 
           JustifyContent="SpaceAround">
    
    <!-- These items will wrap to the next line when they don't fit -->
    <Label Text="Item 1" BackgroundColor="LightBlue" Padding="10" />
    <Label Text="Item 2" BackgroundColor="LightGreen" Padding="10" />
    <Label Text="Item 3" BackgroundColor="LightPink" Padding="10" />
    <Label Text="Item 4" BackgroundColor="LightYellow" Padding="10" />
    <Label Text="Item 5" BackgroundColor="LightCoral" Padding="10" />
    <Label Text="Item 6" BackgroundColor="LightCyan" Padding="10" />
    <Label Text="Item 7" BackgroundColor="LightGray" Padding="10" />
    <Label Text="Item 8" BackgroundColor="LightSeaGreen" Padding="10" />
</FlexLayout>
```

### **Individual Item Control (FlexLayout.Order)**

```xml
<FlexLayout Direction="Row">
    <Label Text="C" FlexLayout.Order="3" BackgroundColor="LightBlue" />
    <Label Text="A" FlexLayout.Order="1" BackgroundColor="LightGreen" />
    <Label Text="B" FlexLayout.Order="2" BackgroundColor="LightPink" />
    <!-- Items will display as: A B C (based on Order) -->
</FlexLayout>
```

### **Individual Item Control (FlexLayout.Grow/Shrink)**

```xml
<FlexLayout Direction="Row">
    <Label Text="Fixed Size" BackgroundColor="LightBlue" />
    
    <Label Text="Grow with available space" 
           BackgroundColor="LightGreen" 
           FlexLayout.Grow="1" />
    
    <Label Text="Grow more" 
           BackgroundColor="LightPink" 
           FlexLayout.Grow="2" />
    
    <Label Text="Shrink first" 
           BackgroundColor="LightYellow" 
           FlexLayout.Shrink="2" />
</FlexLayout>
```

### **Individual Item Control (FlexLayout.Basis)**

```xml
<FlexLayout Direction="Row">
    <Label Text="Auto basis" 
           BackgroundColor="LightBlue" />
    
    <Label Text="Specific basis" 
           BackgroundColor="LightGreen" 
           FlexLayout.Basis="100" />
    
    <Label Text="Percentage basis" 
           BackgroundColor="LightPink" 
           FlexLayout.Basis="30%" />
</FlexLayout>
```

### **Individual Item Alignment (FlexLayout.AlignSelf)**

```xml
<FlexLayout Direction="Row" AlignItems="Center" HeightRequest="200">
    <Label Text="Center (default)" BackgroundColor="LightBlue" />
    
    <Label Text="Start" 
           BackgroundColor="LightGreen" 
           FlexLayout.AlignSelf="Start" />
    
    <Label Text="End" 
           BackgroundColor="LightPink" 
           FlexLayout.AlignSelf="End" />
    
    <Label Text="Stretch" 
           BackgroundColor="LightYellow" 
           FlexLayout.AlignSelf="Stretch" />
</FlexLayout>
```

## 🎮 ตัวอย่างการใช้งานจริง

### **Dashboard Layout ด้วย Grid**

```xml
<Grid RowDefinitions="Auto,*,Auto"
      ColumnDefinitions="*,*,*"
      Padding="20"
      RowSpacing="15"
      ColumnSpacing="15">
      
    <!-- Header -->
    <Label Text="Dashboard" 
           Grid.ColumnSpan="3"
           FontSize="24"
           FontAttributes="Bold"
           HorizontalOptions="Center" />
    
    <!-- Stats Panel 1 -->
    <Frame Grid.Row="1" Grid.Column="0"
           BorderColor="LightGray"
           CornerRadius="10"
           HasShadow="True">
        <StackLayout>
            <Label Text="Sales" FontSize="18" />
            <Label Text="$12,345" FontSize="24" TextColor="Green" />
            <Label Text="+12% vs last month" FontSize="12" TextColor="Green" />
        </StackLayout>
    </Frame>
    
    <!-- Stats Panel 2 -->
    <Frame Grid.Row="1" Grid.Column="1"
           BorderColor="LightGray"
           CornerRadius="10"
           HasShadow="True">
        <StackLayout>
            <Label Text="Visitors" FontSize="18" />
            <Label Text="45,678" FontSize="24" TextColor="Blue" />
            <Label Text="+8% vs last month" FontSize="12" TextColor="Blue" />
        </StackLayout>
    </Frame>
    
    <!-- Stats Panel 3 -->
    <Frame Grid.Row="1" Grid.Column="2"
           BorderColor="LightGray"
           CornerRadius="10"
           HasShadow="True">
        <StackLayout>
            <Label Text="Orders" FontSize="18" />
            <Label Text="1,234" FontSize="24" TextColor="Purple" />
            <Label Text="+5% vs last month" FontSize="12" TextColor="Purple" />
        </StackLayout>
    </Frame>
    
    <!-- Footer -->
    <Button Text="Refresh Data"
            Grid.Row="2"
            Grid.ColumnSpan="3" />
</Grid>
```

### **Product List ด้วย FlexLayout**

```xml
<ScrollView>
    <FlexLayout Direction="Row"
               Wrap="Wrap"
               JustifyContent="SpaceAround"
               Padding="10">
        
        <!-- Product Card 1 -->
        <Frame BorderColor="LightGray"
               CornerRadius="10"
               Margin="5"
               Padding="0"
               HeightRequest="220"
               WidthRequest="160"
               HasShadow="True">
            <StackLayout>
                <Image Source="product1.jpg"
                       Aspect="AspectFill"
                       HeightRequest="120" />
                <StackLayout Padding="10">
                    <Label Text="Wireless Earbuds" FontAttributes="Bold" />
                    <Label Text="$59.99" TextColor="Green" />
                    <Button Text="Add to Cart" 
                            HeightRequest="30"
                            FontSize="12"
                            BackgroundColor="DodgerBlue"
                            TextColor="White" />
                </StackLayout>
            </StackLayout>
        </Frame>
        
        <!-- Product Card 2 -->
        <Frame BorderColor="LightGray"
               CornerRadius="10"
               Margin="5"
               Padding="0"
               HeightRequest="220"
               WidthRequest="160"
               HasShadow="True">
            <StackLayout>
                <Image Source="product2.jpg"
                       Aspect="AspectFill"
                       HeightRequest="120" />
                <StackLayout Padding="10">
                    <Label Text="Smartwatch" FontAttributes="Bold" />
                    <Label Text="$129.99" TextColor="Green" />
                    <Button Text="Add to Cart" 
                            HeightRequest="30"
                            FontSize="12"
                            BackgroundColor="DodgerBlue"
                            TextColor="White" />
                </StackLayout>
            </StackLayout>
        </Frame>
        
        <!-- Add more product cards as needed -->
    </FlexLayout>
</ScrollView>
```

### **Form Layout ด้วย Grid**

```xml
<ScrollView>
    <Grid RowDefinitions="Auto,Auto,Auto,Auto,Auto,Auto,Auto"
          ColumnDefinitions="100,*"
          RowSpacing="15"
          Padding="20">
          
        <Label Text="Profile Form" 
               Grid.ColumnSpan="2"
               FontSize="24"
               FontAttributes="Bold"
               HorizontalOptions="Center" />
        
        <!-- First Name -->
        <Label Text="First Name:"
               Grid.Row="1"
               Grid.Column="0"
               VerticalOptions="Center" />
        <Entry Placeholder="Enter first name"
               Grid.Row="1"
               Grid.Column="1" />
        
        <!-- Last Name -->
        <Label Text="Last Name:"
               Grid.Row="2"
               Grid.Column="0"
               VerticalOptions="Center" />
        <Entry Placeholder="Enter last name"
               Grid.Row="2"
               Grid.Column="1" />
        
        <!-- Email -->
        <Label Text="Email:"
               Grid.Row="3"
               Grid.Column="0"
               VerticalOptions="Center" />
        <Entry Placeholder="Enter email"
               Keyboard="Email"
               Grid.Row="3"
               Grid.Column="1" />
        
        <!-- Phone -->
        <Label Text="Phone:"
               Grid.Row="4"
               Grid.Column="0"
               VerticalOptions="Center" />
        <Entry Placeholder="Enter phone number"
               Keyboard="Telephone"
               Grid.Row="4"
               Grid.Column="1" />
        
        <!-- Address -->
        <Label Text="Address:"
               Grid.Row="5"
               Grid.Column="0"
               VerticalOptions="Start" />
        <Editor Placeholder="Enter address"
                HeightRequest="100"
                Grid.Row="5"
                Grid.Column="1" />
        
        <!-- Submit Button -->
        <Button Text="Submit"
                Grid.Row="6"
                Grid.Column="0"
                Grid.ColumnSpan="2"
                BackgroundColor="DodgerBlue"
                TextColor="White" />
    </Grid>
</ScrollView>
```

### **Media Gallery ผสมผสาน Grid และ FlexLayout**

```xml
<Grid RowDefinitions="Auto,*,Auto"
      Padding="10">
      
    <!-- Header with search -->
    <Grid Grid.Row="0"
          ColumnDefinitions="*,Auto"
          Margin="0,0,0,10">
        <Entry Placeholder="Search gallery..." />
        <Button Text="Search"
                Grid.Column="1"
                Margin="10,0,0,0"
                WidthRequest="80" />
    </Grid>
    
    <!-- Gallery with FlexLayout -->
    <ScrollView Grid.Row="1">
        <FlexLayout Direction="Row"
                   Wrap="Wrap"
                   JustifyContent="SpaceBetween">
                   
            <!-- Media Item 1 -->
            <Frame Margin="5"
                   Padding="0"
                   BorderColor="Transparent"
                   WidthRequest="110"
                   HeightRequest="110">
                <Image Source="image1.jpg"
                       Aspect="AspectFill" />
            </Frame>
            
            <!-- Media Item 2 -->
            <Frame Margin="5"
                   Padding="0"
                   BorderColor="Transparent"
                   WidthRequest="110"
                   HeightRequest="110">
                <Image Source="image2.jpg"
                       Aspect="AspectFill" />
            </Frame>
            
            <!-- Media Item 3 -->
            <Frame Margin="5"
                   Padding="0"
                   BorderColor="Transparent"
                   WidthRequest="110"
                   HeightRequest="110">
                <Image Source="image3.jpg"
                       Aspect="AspectFill" />
            </Frame>
            
            <!-- More media items... -->
        </FlexLayout>
    </ScrollView>
    
    <!-- Footer with actions -->
    <Grid Grid.Row="2"
          ColumnDefinitions="*,*,*"
          Margin="0,10,0,0">
        <Button Text="Upload"
                Grid.Column="0"
                Margin="5,0" />
        <Button Text="Select"
                Grid.Column="1"
                Margin="5,0" />
        <Button Text="Share"
                Grid.Column="2"
                Margin="5,0" />
    </Grid>
</Grid>
```

### **ตัวอย่าง Responsive Layout**

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="MauiApp.ResponsivePage">
    
    <ContentPage.Resources>
        <ResourceDictionary>
            <!-- Define styles for different screen sizes -->
            <Style x:Key="TitleLabelStyle" TargetType="Label">
                <Setter Property="FontSize" Value="24" />
                <Setter Property="FontAttributes" Value="Bold" />
                <Setter Property="HorizontalOptions" Value="Center" />
            </Style>
            
            <Style x:Key="CardStyle" TargetType="Frame">
                <Setter Property="BorderColor" Value="LightGray" />
                <Setter Property="CornerRadius" Value="10" />
                <Setter Property="HasShadow" Value="True" />
                <Setter Property="Margin" Value="10" />
            </Style>
        </ResourceDictionary>
    </ContentPage.Resources>
    
    <Grid x:Name="MainGrid"
          RowDefinitions="Auto,*"
          ColumnDefinitions="*">
          
        <!-- Header -->
        <Label Text="Responsive Gallery"
               Style="{StaticResource TitleLabelStyle}"
               Margin="0,20" />
        
        <!-- Content area - will switch between row/column layout -->
        <FlexLayout x:Name="ContentLayout"
                   Grid.Row="1"
                   Direction="Row"
                   Wrap="Wrap"
                   JustifyContent="SpaceAround"
                   AlignItems="Center">
            
            <!-- Item 1 -->
            <Frame Style="{StaticResource CardStyle}"
                   WidthRequest="280"
                   HeightRequest="200">
                <Grid RowDefinitions="*,Auto">
                    <Image Source="image1.jpg"
                           Aspect="AspectFill" />
                    <Label Text="Beautiful Mountains"
                           Grid.Row="1"
                           Margin="5"
                           HorizontalOptions="Center" />
                </Grid>
            </Frame>
            
            <!-- Item 2 -->
            <Frame Style="{StaticResource CardStyle}"
                   WidthRequest="280"
                   HeightRequest="200">
                <Grid RowDefinitions="*,Auto">
                    <Image Source="image2.jpg"
                           Aspect="AspectFill" />
                    <Label Text="Peaceful Lake"
                           Grid.Row="1"
                           Margin="5"
                           HorizontalOptions="Center" />
                </Grid>
            </Frame>
            
            <!-- Item 3 -->
            <Frame Style="{StaticResource CardStyle}"
                   WidthRequest="280"
                   HeightRequest="200">
                <Grid RowDefinitions="*,Auto">
                    <Image Source="image3.jpg"
                           Aspect="AspectFill" />
                    <Label Text="Forest Trail"
                           Grid.Row="1"
                           Margin="5"
                           HorizontalOptions="Center" />
                </Grid>
            </Frame>
            
            <!-- Item 4 -->
            <Frame Style="{StaticResource CardStyle}"
                   WidthRequest="280"
                   HeightRequest="200">
                <Grid RowDefinitions="*,Auto">
                    <Image Source="image4.jpg"
                           Aspect="AspectFill" />
                    <Label Text="City Skyline"
                           Grid.Row="1"
                           Margin="5"
                           HorizontalOptions="Center" />
                </Grid>
            </Frame>
        </FlexLayout>
    </Grid>
</ContentPage>
```

```csharp
// ResponsivePage.xaml.cs
public partial class ResponsivePage : ContentPage
{
    public ResponsivePage()
    {
        InitializeComponent();
        UpdateLayoutForScreenSize();
    }
    
    protected override void OnSizeAllocated(double width, double height)
    {
        base.OnSizeAllocated(width, height);
        UpdateLayoutForScreenSize();
    }
    
    private void UpdateLayoutForScreenSize()
    {
        double width = Width;
        double height = Height;
        
        if (width <= 0 || height <= 0)
            return;
            
        // Desktop/Tablet (landscape) or Phone (landscape)
        if (width > height)
        {
            // Large screen in landscape
            if (width >= 1024)
            {
                // Large screens: 4 columns no wrap
                ContentLayout.Direction = FlexDirection.Row;
                ContentLayout.Wrap = FlexWrap.NoWrap;
            }
            else
            {
                // Medium screens: 2 columns with wrap
                ContentLayout.Direction = FlexDirection.Row;
                ContentLayout.Wrap = FlexWrap.Wrap;
            }
        }
        // Phone (portrait)
        else
        {
            // Small screens: 1 column with wrap
            ContentLayout.Direction = FlexDirection.Column;
            ContentLayout.Wrap = FlexWrap.NoWrap;
        }
    }
}
```

## 🧪 แบบฝึกหัด

### **Exercise 1: สร้าง Grid-based Dashboard**

สร้าง Dashboard หน้าจอที่มี:
1. Header ที่มี logo และ title
2. Sidebar ที่มี menu options
3. Main content area ที่แสดง statistics ในรูปแบบ cards
4. Footer ที่มีข้อมูล copyright และ links

### **Exercise 2: สร้าง FlexLayout Photo Gallery**

สร้าง Photo Gallery ที่:
1. แสดงรูปภาพในรูปแบบ grid แบบยืดหยุ่น
2. ปรับตัวให้แสดงตามขนาดหน้าจอ (1, 2, หรือ 3 คอลัมน์)
3. มีการแสดง captions ใต้แต่ละรูปภาพ
4. มีปุ่มให้คลิกเพื่อดูภาพขนาดใหญ่

### **Exercise 3: สร้าง Responsive Form**

สร้าง Form ที่:
1. มี labels และ input fields แบบ 2 คอลัมน์บนหน้าจอขนาดใหญ่
2. ปรับให้เป็น 1 คอลัมน์บนหน้าจอขนาดเล็ก
3. มี validation และ error messages
4. มีปุ่ม submit ที่ responsive

## 📝 สรุป

### Grid ใน MAUI:
- เหมาะสำหรับ layouts แบบตาราง ที่ต้องการความแม่นยำในการจัดวาง
- กำหนด rows และ columns ได้แบบ exact, auto, หรือ proportional
- ใช้ Grid.Row และ Grid.Column เพื่อวาง elements ในตำแหน่งที่ต้องการ
- สนับสนุน row/column spanning สำหรับ elements ที่ต้องการพื้นที่มากกว่า 1 cell
- สามารถซ้อน (nest) กันได้เพื่อสร้าง complex layouts

### FlexLayout ใน MAUI:
- สร้างแรงบันดาลใจจาก CSS Flexbox ใน web development
- เหมาะสำหรับ one-dimensional layouts ที่ต้องการความยืดหยุ่น
- ควบคุมการจัดวางบน main axis (Direction + JustifyContent)
- ควบคุมการจัดวางบน cross axis (AlignItems + AlignContent)
- สามารถปรับแต่ง individual items ด้วย attached properties
- เหมาะสำหรับ responsive UIs ที่ต้องปรับตัวตามขนาดหน้าจอ

### การเลือกใช้ระหว่าง Grid และ FlexLayout:
- **ใช้ Grid เมื่อ:**
  - ต้องการ precise positioning ในรูปแบบ rows และ columns
  - มี complex layout ที่ต้องการ cell spanning
  - ต้องการจัดวางใน 2 มิติอย่างแม่นยำ

- **ใช้ FlexLayout เมื่อ:**
  - ต้องการ dynamic/flexible layouts ที่ปรับตามเนื้อหา
  - จัดวาง items แบบ linear (แถวหรือคอลัมน์)
  - ต้องการการจัดวางที่ responsive และ adaptive

การผสมผสานทั้งสอง layouts อย่างมีประสิทธิภาพจะช่วยให้คุณสามารถสร้าง UIs ที่ซับซ้อนและ responsive ใน MAUI applications ได้

## 🔍 อ้างอิงเพิ่มเติม

- [MAUI Grid Documentation](https://learn.microsoft.com/dotnet/maui/user-interface/layouts/grid)
- [MAUI FlexLayout Documentation](https://learn.microsoft.com/dotnet/maui/user-interface/layouts/flexlayout)
- [Responsive Layout Patterns](https://learn.microsoft.com/dotnet/maui/user-interface/responsive-layouts)

---

**[⬅️ กลับไปที่ บทที่ 5: Layouts & Controls](./05-layouts-and-controls.md) | [➡️ ไปที่ บทที่ 6: Data Binding & MVVM](./06-data-binding-mvvm.md)**
