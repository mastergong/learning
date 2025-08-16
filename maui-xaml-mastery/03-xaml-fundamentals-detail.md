# XAML Markup Extensions ใน .NET MAUI

## 🎯 จุดประสงค์การเรียนรู้

เมื่อเสร็จสิ้นบทเรียนนี้ คุณจะสามารถ:
- เข้าใจแนวคิดพื้นฐานของ XAML Markup Extensions
- ใช้งาน Markup Extensions ที่สำคัญทั้งหมดใน MAUI
- สร้าง Custom Markup Extensions ของตัวเอง
- ประยุกต์ใช้ Markup Extensions เพื่อลดการเขียนโค้ดซ้ำและเพิ่มความยืดหยุ่น
- แก้ไขปัญหาที่พบบ่อยเกี่ยวกับ Markup Extensions

## 📋 Markup Extensions คืออะไร?

**Markup Extensions** เป็นคุณสมบัติพิเศษใน XAML ที่ช่วยให้เราสามารถขยายความสามารถนอกเหนือจากการตั้งค่าคุณสมบัติปกติ โดยจะใช้สัญลักษณ์วงเล็บปีกกาครอบคู่ `{` และ `}` เพื่อบอกให้ XAML parser รู้ว่าต้องประมวลผลพิเศษ

### **วัตถุประสงค์ของ Markup Extensions**

1. **ใช้ค่าจากแหล่งอื่น** - แทนที่จะเป็นค่าตายตัว เช่น ข้อมูลจาก resources, bindings
2. **คำนวณค่าแบบไดนามิก** - คำนวณค่าในขณะ runtime
3. **อ้างอิงถึง objects อื่น** - เช่น อ้างถึง elements หรือ resources อื่น
4. **ลดโค้ดซ้ำซ้อน** - ช่วยให้โค้ด XAML สะอาดและเป็นระเบียบ

## 🌟 Markup Extensions ที่สำคัญ

### 1. **{Binding}**

การเชื่อมต่อข้อมูลแบบไดนามิกระหว่าง source (ViewModel) และ target (UI element)

#### รูปแบบพื้นฐาน
```xml
<Label Text="{Binding Name}" />
```

#### รูปแบบการใช้งานแบบละเอียด
```xml
<Label Text="{Binding 
         Path=FullName, 
         Mode=TwoWay, 
         Converter={StaticResource StringToUpperConverter}, 
         ConverterParameter=true, 
         StringFormat='Welcome, {0}!', 
         FallbackValue='Guest'}" />
```

#### คุณสมบัติที่สำคัญของ Binding
- **Path** - คุณสมบัติที่ต้องการ bind (สามารถละได้ ถ้าเป็นคุณสมบัติแรก)
- **Mode** - รูปแบบการ bind (OneTime, OneWay, TwoWay, OneWayToSource, Default)
- **Converter** - ตัวแปลงข้อมูลระหว่าง source และ target
- **ConverterParameter** - พารามิเตอร์เพิ่มเติมสำหรับ converter
- **StringFormat** - รูปแบบการแสดงผลข้อความ
- **FallbackValue** - ค่าที่ใช้เมื่อ binding ล้มเหลว
- **TargetNullValue** - ค่าที่ใช้เมื่อค่าที่ได้เป็น null

#### การใช้งานกับ Relative Source
```xml
<!-- Bind to element ตัวเอง -->
<Entry Text="{Binding Source={RelativeSource Self}, Path=Height}" />

<!-- Bind to parent element -->
<Label Text="{Binding Source={RelativeSource AncestorType={x:Type StackLayout}}, Path=Spacing}" />
```

#### การใช้งานกับ ElementName
```xml
<!-- Bind to element อื่นใน page -->
<Slider x:Name="volumeSlider" Maximum="100" />
<Label Text="{Binding Source={x:Reference volumeSlider}, Path=Value, StringFormat='Volume: {0:F0}%'}" />
```

### 2. **{StaticResource}**

เข้าถึง resource ที่ประกาศไว้ใน ResourceDictionary

```xml
<!-- Define resources -->
<ContentPage.Resources>
    <Color x:Key="primaryColor">#512BD4</Color>
    <Style x:Key="labelStyle" TargetType="Label">
        <Setter Property="TextColor" Value="{StaticResource primaryColor}" />
        <Setter Property="FontSize" Value="18" />
    </Style>
</ContentPage.Resources>

<!-- Use resources -->
<StackLayout>
    <Label Text="Hello, MAUI!" Style="{StaticResource labelStyle}" />
    <Button Text="Click Me" BackgroundColor="{StaticResource primaryColor}" />
</StackLayout>
```

### 3. **{DynamicResource}**

คล้ายกับ StaticResource แต่จะอัปเดตเมื่อ resource มีการเปลี่ยนแปลง เหมาะกับการทำ themes ที่เปลี่ยนแปลงขณะ runtime

```xml
<ContentPage.Resources>
    <ResourceDictionary>
        <Color x:Key="dynamicTextColor">Black</Color>
    </ResourceDictionary>
</ContentPage.Resources>

<StackLayout>
    <Label Text="This text changes color" TextColor="{DynamicResource dynamicTextColor}" />
    <Button Text="Toggle Dark Mode" Clicked="OnToggleDarkMode" />
</StackLayout>
```

```csharp
void OnToggleDarkMode(object sender, EventArgs e)
{
    // เปลี่ยนสีแบบไดนามิก
    if (Resources["dynamicTextColor"] as Color == Colors.Black)
        Resources["dynamicTextColor"] = Colors.White;
    else
        Resources["dynamicTextColor"] = Colors.Black;
}
```

### 4. **{x:Static}**

เข้าถึงค่า static fields, properties, enums, หรือ constants

```xml
<!-- Access static field or property -->
<Label Text="{x:Static system:Math.PI}" />

<!-- Access constant -->
<Label Text="{x:Static local:AppConstants.AppName}" />

<!-- Access enum value -->
<Label TextColor="{x:Static Colors.Red}" />
```

### 5. **{TemplateBinding}**

ใช้เฉพาะภายใน ControlTemplate เพื่อเชื่อมต่อคุณสมบัติจาก templated parent

```xml
<ControlTemplate x:Key="CustomButtonTemplate">
    <Frame BorderColor="{TemplateBinding BorderColor}"
           BackgroundColor="{TemplateBinding BackgroundColor}"
           CornerRadius="8"
           HasShadow="True">
        <Label Text="{TemplateBinding Text}"
               TextColor="{TemplateBinding TextColor}"
               HorizontalOptions="Center"
               VerticalOptions="Center" />
    </Frame>
</ControlTemplate>

<Button Text="Custom Button"
        ControlTemplate="{StaticResource CustomButtonTemplate}"
        BackgroundColor="DodgerBlue"
        TextColor="White" />
```

### 6. **{AppThemeBinding}**

เลือกค่าตามธีมของระบบ (Light/Dark)

```xml
<Label Text="Adaptive Text"
       TextColor="{AppThemeBinding Light=Black, Dark=White}" />

<Frame BackgroundColor="{AppThemeBinding Light={StaticResource LightBackgroundColor}, 
                                        Dark={StaticResource DarkBackgroundColor}}">
    <!-- Content -->
</Frame>
```

### 7. **{OnPlatform}**

กำหนดค่าที่แตกต่างกันตามแพลตฟอร์ม

```xml
<Label Text="Platform Specific"
       FontSize="{OnPlatform iOS=20, Android=24, WinUI=22}"
       TextColor="{OnPlatform iOS=Blue, Android=Green, WinUI=Purple}" />

<!-- รูปแบบที่ซับซ้อนมากขึ้น -->
<Label>
    <Label.TextColor>
        <OnPlatform x:TypeArguments="Color">
            <On Platform="iOS" Value="Blue" />
            <On Platform="Android" Value="Green" />
            <On Platform="WinUI" Value="Purple" />
        </OnPlatform>
    </Label.TextColor>
</Label>
```

### 8. **{OnIdiom}**

กำหนดค่าตามประเภทอุปกรณ์ (Phone, Tablet, Desktop, TV, Watch)

```xml
<StackLayout Padding="{OnIdiom Phone=10, Tablet=20, Desktop=30}">
    <Label Text="Welcome"
           FontSize="{OnIdiom Phone=18, Tablet=24, Desktop=30}" />
</StackLayout>

<!-- รูปแบบที่ซับซ้อนมากขึ้น -->
<Label Text="Welcome">
    <Label.FontSize>
        <OnIdiom x:TypeArguments="x:Double">
            <OnIdiom.Phone>18</OnIdiom.Phone>
            <OnIdiom.Tablet>24</OnIdiom.Tablet>
            <OnIdiom.Desktop>30</OnIdiom.Desktop>
        </OnIdiom>
    </Label.FontSize>
</Label>
```

### 9. **{Binding} กับ MultiBinding**

รวมค่าจากหลาย binding sources โดยใช้ converter

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:local="clr-namespace:MyMauiApp.Converters"
             x:Class="MyMauiApp.MainPage">
    <ContentPage.Resources>
        <local:FullNameConverter x:Key="FullNameConverter" />
    </ContentPage.Resources>
    
    <Label>
        <Label.Text>
            <MultiBinding Converter="{StaticResource FullNameConverter}">
                <Binding Path="FirstName" />
                <Binding Path="LastName" />
            </MultiBinding>
        </Label.Text>
    </Label>
</ContentPage>
```

```csharp
public class FullNameConverter : IMultiValueConverter
{
    public object Convert(object[] values, Type targetType, object parameter, CultureInfo culture)
    {
        if (values == null || values.Length < 2)
            return string.Empty;
            
        string firstName = values[0]?.ToString() ?? string.Empty;
        string lastName = values[1]?.ToString() ?? string.Empty;
        
        return $"{firstName} {lastName}";
    }
    
    public object[] ConvertBack(object value, Type[] targetTypes, object parameter, CultureInfo culture)
    {
        if (value is string fullName)
        {
            string[] nameParts = fullName.Split(' ');
            if (nameParts.Length >= 2)
                return new object[] { nameParts[0], nameParts[1] };
        }
        
        return new object[] { string.Empty, string.Empty };
    }
}
```

### 10. **{x:Type}**

อ้างอิงถึงประเภทข้อมูล (Type) ไม่ใช่ instance

```xml
<!-- การใช้งานกับ x:Array -->
<x:Array Type="{x:Type sys:String}">
    <x:String>Option 1</x:String>
    <x:String>Option 2</x:String>
    <x:String>Option 3</x:String>
</x:Array>

<!-- การใช้งานกับ ListView -->
<ListView>
    <ListView.ItemTemplate>
        <DataTemplate>
            <ViewCell>
                <Label Text="{Binding}" />
            </ViewCell>
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```

### 11. **{x:Null}**

กำหนดค่า null ให้กับคุณสมบัติ

```xml
<Button Text="Reset Image" Clicked="OnResetImage">
    <Button.ImageSource>
        <x:Null />
    </Button.ImageSource>
</Button>

<Image Source="{Binding ProfileImage, FallbackValue={x:Null}}" />
```

### 12. **{x:Array}**

สร้างและกำหนดค่าให้กับอาร์เรย์

```xml
<ListView>
    <ListView.ItemsSource>
        <x:Array Type="{x:Type x:String}">
            <x:String>Option 1</x:String>
            <x:String>Option 2</x:String>
            <x:String>Option 3</x:String>
        </x:Array>
    </ListView.ItemsSource>
</ListView>
```

## 🔧 การสร้าง Custom Markup Extension

### **ขั้นตอนการสร้าง Custom Markup Extension**

1. สร้างคลาสที่สืบทอดจาก `BindableObject` และ `IMarkupExtension<T>`
2. implement เมธอด `ProvideValue(IServiceProvider serviceProvider)`
3. กำหนดคุณสมบัติที่ต้องการ
4. ใช้งานใน XAML

### **ตัวอย่าง 1: Enum Description Markup Extension**

สร้าง markup extension ที่แสดงค่า description จาก Description attribute ของ enum

```csharp
// 1. สร้าง Description attribute สำหรับ enum
public enum UserStatus
{
    [Description("ผู้ใช้ใหม่")]
    New,
    
    [Description("ใช้งานอยู่")]
    Active,
    
    [Description("ถูกระงับชั่วคราว")]
    Suspended,
    
    [Description("ถูกลบ")]
    Deleted
}

// 2. สร้าง markup extension
[ContentProperty("EnumValue")]
public class EnumDescriptionExtension : IMarkupExtension<string>
{
    public object EnumValue { get; set; }

    public string ProvideValue(IServiceProvider serviceProvider)
    {
        if (EnumValue == null)
            return string.Empty;

        if (EnumValue is Enum enumValue)
        {
            var fieldInfo = enumValue.GetType().GetField(enumValue.ToString());
            
            var attribute = fieldInfo?.GetCustomAttribute<DescriptionAttribute>();
            return attribute?.Description ?? enumValue.ToString();
        }
        
        return EnumValue.ToString();
    }

    object IMarkupExtension.ProvideValue(IServiceProvider serviceProvider)
    {
        return ProvideValue(serviceProvider);
    }
}
```

ใช้งานใน XAML:

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:local="clr-namespace:MyMauiApp.Extensions"
             x:Class="MyMauiApp.MainPage">

    <StackLayout>
        <Label Text="{local:EnumDescription EnumValue={x:Static local:UserStatus.Active}}" />
    </StackLayout>
</ContentPage>
```

### **ตัวอย่าง 2: Arithmetic Markup Extension**

สร้าง markup extension ที่คำนวณค่าทางคณิตศาสตร์

```csharp
[ContentProperty("Expression")]
public class CalcExtension : IMarkupExtension<double>
{
    public string Expression { get; set; }
    
    public double ProvideValue(IServiceProvider serviceProvider)
    {
        if (string.IsNullOrWhiteSpace(Expression))
            return 0;
            
        try
        {
            // ใช้ library สำหรับ evaluate expression
            // (ในตัวอย่างนี้เป็นแค่ pseudocode - คุณต้องใช้ library จริง เช่น NCalc)
            return EvaluateExpression(Expression);
        }
        catch
        {
            return 0;
        }
    }
    
    object IMarkupExtension.ProvideValue(IServiceProvider serviceProvider)
    {
        return ProvideValue(serviceProvider);
    }
    
    private double EvaluateExpression(string expression)
    {
        // ในตัวอย่างนี้ให้สมมติว่าเราทำการคำนวณง่ายๆ (10 + 5) * 2
        return 30;
    }
}
```

ใช้งานใน XAML:

```xml
<Label Text="Result:" />
<Label Text="{local:Calc Expression='(10 + 5) * 2'}" />

<!-- นำมาใช้กับ UI properties -->
<BoxView WidthRequest="{local:Calc Expression='100 + 50'}"
         HeightRequest="{local:Calc Expression='50 * 2'}"
         Color="Blue" />
```

### **ตัวอย่าง 3: Resource Lookup with Fallback**

สร้าง markup extension ที่ค้นหา resource และใช้ค่า fallback ถ้าไม่พบ

```csharp
public class ResourceWithFallbackExtension : IMarkupExtension<object>
{
    public string Key { get; set; }
    public object FallbackValue { get; set; }
    
    public object ProvideValue(IServiceProvider serviceProvider)
    {
        if (serviceProvider == null)
            return FallbackValue;
            
        // ดึง IProvideValueTarget ซึ่งมีข้อมูลเกี่ยวกับ target object และ property
        var provideValueTarget = serviceProvider.GetService<IProvideValueTarget>();
        if (provideValueTarget == null)
            return FallbackValue;
            
        // ดึง element ที่ใช้ markup extension นี้
        var targetElement = provideValueTarget.TargetObject as Element;
        if (targetElement == null)
            return FallbackValue;
            
        // ค้นหา resource จาก visual tree
        var foundResource = targetElement.FindResource(Key);
        return foundResource ?? FallbackValue;
    }
    
    object IMarkupExtension.ProvideValue(IServiceProvider serviceProvider)
    {
        return ProvideValue(serviceProvider);
    }
}
```

ใช้งานใน XAML:

```xml
<ContentPage.Resources>
    <ResourceDictionary>
        <Color x:Key="PrimaryColor">#512BD4</Color>
    </ResourceDictionary>
</ContentPage.Resources>

<Label TextColor="{local:ResourceWithFallback Key=PrimaryColor, FallbackValue=Black}" />
<Label TextColor="{local:ResourceWithFallback Key=SecondaryColor, FallbackValue=Gray}" />
```

## 🎨 การใช้ Markup Extensions อย่างมีประสิทธิภาพ

### **Markup Extensions ซ้อน Markup Extensions**

คุณสามารถใช้ markup extension หนึ่งเป็นค่าให้กับอีก markup extension หนึ่งได้

```xml
<Label Text="{Binding Username, FallbackValue={x:Static local:AppConstants.DefaultUsername}}" />

<Frame BackgroundColor="{AppThemeBinding 
                           Light={StaticResource LightBackgroundColor},
                           Dark={DynamicResource DarkBackgroundColor}}">
    <!-- Content -->
</Frame>
```

### **การใช้ Markup Extensions กับ Converters**

```xml
<ContentPage.Resources>
    <local:BoolToColorConverter x:Key="BoolToColorConverter" />
</ContentPage.Resources>

<Label Text="Status:"
       TextColor="{Binding IsActive, 
                   Converter={StaticResource BoolToColorConverter}, 
                   ConverterParameter={x:Static Colors.Green},
                   FallbackValue={x:Static Colors.Gray}}" />
```

### **Binding กับคอลเลคชั่นข้อมูล**

```xml
<CollectionView ItemsSource="{Binding Products}">
    <CollectionView.ItemTemplate>
        <DataTemplate>
            <Grid>
                <Label Text="{Binding Name}" />
                <Label Text="{Binding Price, StringFormat='${0:F2}'}" />
                <Label IsVisible="{Binding OnSale}"
                       Text="{Binding DiscountPrice, 
                              StringFormat='SALE: ${0:F2}',
                              FallbackValue='SALE'}" />
            </Grid>
        </DataTemplate>
    </CollectionView.ItemTemplate>
</CollectionView>
```

### **Resource Sharing ระหว่างไฟล์**

```xml
<!-- App.xaml -->
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary Source="Resources/Colors.xaml" />
        <ResourceDictionary Source="Resources/Styles.xaml" />
    </ResourceDictionary>
</Application.Resources>

<!-- Colors.xaml -->
<ResourceDictionary xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
                    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">
    <Color x:Key="PrimaryColor">#512BD4</Color>
    <Color x:Key="SecondaryColor">#DFD8F7</Color>
</ResourceDictionary>

<!-- ใช้งานใน page ต่างๆ -->
<Label TextColor="{StaticResource PrimaryColor}" />
```

### **Binding กับ Command และ CommandParameter**

```xml
<Button Text="Save"
        Command="{Binding SaveCommand}"
        CommandParameter="{Binding CurrentItem}" />

<Button Text="Delete"
        Command="{Binding DeleteCommand}"
        CommandParameter="{x:Reference itemId}" />
```

### **Reference พารามิเตอร์ของ x:Array**

```xml
<ContentPage.Resources>
    <x:Array x:Key="colorOptions" Type="{x:Type Color}">
        <Color>Red</Color>
        <Color>Green</Color>
        <Color>Blue</Color>
        <Color>Purple</Color>
        <Color>Orange</Color>
    </x:Array>
</ContentPage.Resources>

<CollectionView ItemsSource="{StaticResource colorOptions}">
    <CollectionView.ItemTemplate>
        <DataTemplate>
            <Frame BackgroundColor="{Binding .}"
                   HeightRequest="50"
                   Margin="10" />
        </DataTemplate>
    </CollectionView.ItemTemplate>
</CollectionView>
```

## 🚫 ข้อควรระวังและแนวทางแก้ไขปัญหา

### **ปัญหาที่พบบ่อยและวิธีแก้ไข**

#### **1. Markup Extension ไม่ทำงานหรือไม่แสดงค่า**

สาเหตุที่เป็นไปได้:
- Namespace ไม่ได้ประกาศ
- ชื่อ extension ผิด
- พารามิเตอร์ไม่ถูกต้อง

```xml
<!-- ❌ ผิด - ไม่มีการประกาศ namespace -->
<Label Text="{local:MyExtension Value=123}" />

<!-- ✅ ถูก - ประกาศ namespace -->
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:local="clr-namespace:MyMauiApp.Extensions"
             x:Class="MyMauiApp.MainPage">
    <Label Text="{local:MyExtension Value=123}" />
</ContentPage>
```

#### **2. Binding ไม่ทำงาน**

สาเหตุที่เป็นไปได้:
- BindingContext ไม่ได้ตั้งค่า
- Path ผิด
- ไม่มีการ implement INotifyPropertyChanged

วิธีแก้ไข:
```csharp
// ตรวจสอบ Output Window ใน Visual Studio สำหรับ binding errors
// เพิ่มเพื่อ debug binding
Debug.WriteLine($"BindingContext: {myControl.BindingContext}");
```

#### **3. StaticResource ไม่พบ**

สาเหตุที่เป็นไปได้:
- Resource ไม่ได้ประกาศใน scope ที่เหมาะสม
- Key ผิด

```xml
<!-- ❌ ผิด - resource กับการใช้งานอยู่คนละ scope -->
<Frame>
    <Frame.Resources>
        <Color x:Key="TextColor">Blue</Color>
    </Frame.Resources>
</Frame>
<Label TextColor="{StaticResource TextColor}" /> <!-- จะไม่เจอ resource -->

<!-- ✅ ถูก - resource อยู่ใน scope ที่สามารถเข้าถึงได้ -->
<ContentPage.Resources>
    <Color x:Key="TextColor">Blue</Color>
</ContentPage.Resources>
<Label TextColor="{StaticResource TextColor}" />
```

#### **4. Markup Extension ซ้อนกันไม่ทำงาน**

```xml
<!-- ❌ ผิด - Markup extension ซ้อนกันโดยไม่มี quotes -->
<Label Text={Binding Name, FallbackValue={x:Static local:Constants.DefaultName}} />

<!-- ✅ ถูก - ใช้ property element syntax -->
<Label>
    <Label.Text>
        <Binding Path="Name">
            <Binding.FallbackValue>
                <x:Static Member="local:Constants.DefaultName" />
            </Binding.FallbackValue>
        </Binding>
    </Label.Text>
</Label>
```

#### **5. Converter ไม่ทำงาน**

```xml
<!-- ❌ ผิด - Converter ไม่ได้ประกาศเป็น resource -->
<Label Text="{Binding IsActive, Converter={local:BoolToColorConverter}}" />

<!-- ✅ ถูก - ประกาศ converter เป็น resource และใช้ StaticResource -->
<ContentPage.Resources>
    <local:BoolToColorConverter x:Key="boolToColor" />
</ContentPage.Resources>
<Label Text="{Binding IsActive, Converter={StaticResource boolToColor}}" />
```

### **Debugging Markup Extensions**

#### **การใช้ Visual Studio Debugger**

1. ใส่ breakpoint ใน `ProvideValue` ของ markup extension
2. รันแอปพลิเคชันในโหมด debug
3. ตรวจสอบค่าที่ส่งเข้ามาและค่าที่ส่งกลับ

#### **การใช้ Debug Output**

```csharp
public object ProvideValue(IServiceProvider serviceProvider)
{
    // เพิ่ม debug output
    System.Diagnostics.Debug.WriteLine($"MyExtension: {nameof(ProvideValue)} called");
    System.Diagnostics.Debug.WriteLine($"Parameter: {MyParameter}");
    
    var result = ComputeValue();
    
    System.Diagnostics.Debug.WriteLine($"Result: {result}");
    return result;
}
```

#### **การตรวจสอบด้วย Visual Tree Debugging**

```csharp
private void InspectVisualTree(Element element, int depth = 0)
{
    string indent = new string(' ', depth * 2);
    Debug.WriteLine($"{indent}{element.GetType().Name}");
    
    if (element is Layout layout)
    {
        foreach (var child in layout.Children)
        {
            InspectVisualTree(child, depth + 1);
        }
    }
}

// เรียกใช้ใน page loaded
InspectVisualTree(this);
```

## 🧪 แบบฝึกหัด

### **แบบฝึกหัดที่ 1: StaticResource และ DynamicResource**

สร้างหน้า XAML ที่มี:
1. ResourceDictionary ที่มี colors, styles, และ templates
2. ใช้ StaticResource เพื่ออ้างอิงถึง resources เหล่านี้
3. สร้างปุ่มที่เมื่อคลิกแล้วเปลี่ยน resource ที่ใช้ DynamicResource
4. สังเกตว่า elements ที่ใช้ DynamicResource จะอัปเดตโดยอัตโนมัติ

### **แบบฝึกหัดที่ 2: Binding และ Value Converters**

สร้าง:
1. ViewModel ที่มี properties หลายประเภท (string, int, bool, DateTime)
2. Value Converters สำหรับแต่ละประเภท (เช่น BoolToColorConverter, DateTimeToStringConverter)
3. UI ที่ใช้ Binding กับ properties เหล่านี้ผ่าน converters
4. ทดลองใช้ StringFormat, FallbackValue, และ TargetNullValue

### **แบบฝึกหัดที่ 3: Custom Markup Extension**

สร้าง Custom Markup Extension ที่:
1. รับค่า key เป็น string
2. ค้นหาค่าจาก AppSettings หรือ preferences
3. รองรับค่า default ถ้าไม่พบ key
4. ใช้ใน XAML เพื่อแสดงการตั้งค่าของแอปพลิเคชัน

### **แบบฝึกหัดที่ 4: Markup Extensions แบบ Advanced**

1. สร้าง CollectionView ที่แสดงรายการสินค้า
2. ใช้ DataTemplate ที่มีการใช้ Binding, StaticResource, และ OnPlatform
3. เพิ่ม MultiBinding เพื่อรวมข้อมูลหลายอย่าง (เช่น ชื่อ + ราคา)
4. ทำให้ UI responsive โดยใช้ OnIdiom สำหรับขนาดหน้าจอต่างๆ

## 📝 สรุป

XAML Markup Extensions เป็นเครื่องมือที่ทรงพลังใน MAUI ที่ช่วยให้เราสร้าง UI ที่ยืดหยุ่น, แสดงผลแบบไดนามิก, และลดการเขียนโค้ดซ้ำซ้อน โดยสรุป:

- **{Binding}** - เชื่อมต่อข้อมูลแบบไดนามิกระหว่าง ViewModel และ UI
- **{StaticResource}/{DynamicResource}** - เข้าถึงทรัพยากรที่กำหนดไว้
- **{x:Static}** - เข้าถึงค่า static
- **{AppThemeBinding}** - ปรับแต่ง UI ตามธีม (Light/Dark)
- **{OnPlatform}/{OnIdiom}** - ปรับแต่ง UI ตามแพลตฟอร์มและอุปกรณ์
- **Custom Markup Extensions** - ขยายความสามารถของ XAML ตามต้องการ

การใช้ Markup Extensions อย่างมีประสิทธิภาพช่วยให้โค้ด XAML สะอาด, เป็นระเบียบ, และง่ายต่อการดูแลรักษา ทำให้การพัฒนา MAUI apps สะดวกและมีประสิทธิภาพมากขึ้น

## 🔍 อ้างอิงเพิ่มเติม

- [Microsoft Docs: XAML Markup Extensions](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/xaml/markup-extensions)
- [Creating Custom Markup Extensions](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/xaml/markup-extensions/creating)
- [Binding Markup Extension](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/app-fundamentals/data-binding/binding-markup-extensions)
- [Resource Dictionaries](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/xaml/resource-dictionaries)

---

**[⬅️ กลับไปที่บทที่ 3: XAML Fundamentals](./03-xaml-fundamentals.md) | [➡️ ไปที่บทที่ 4: First MAUI Application](./04-first-maui-application.md)**
