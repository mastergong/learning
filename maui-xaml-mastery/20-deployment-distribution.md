# 🚀 Chapter 20: Deployment & Distribution

จากการเรียนรู้ในบทก่อนหน้า เราได้เข้าใจเรื่อง accessibility และ localization แล้ว ในบทนี้เราจะมาเรียนรู้กระบวนการ deployment และ distribution ของ .NET MAUI applications ไปยัง app stores และ enterprise environments ต่าง ๆ

## 🎯 Learning Objectives

- เข้าใจกระบวนการ deployment สำหรับแต่ละ platform
- สร้าง CI/CD pipelines ด้วย Azure DevOps และ GitHub Actions  
- จัดการ app signing, certificates และ provisioning profiles
- เรียนรู้ store submission processes และ requirements
- ใช้งาน enterprise deployment strategies
- เข้าใจ app versioning และ update mechanisms

---

## 1. 📱 Platform-Specific Deployment

### 1.1 Android Deployment

#### Android App Bundle (AAB) Configuration

```xml
<!-- In the .csproj file -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFrameworks>net8.0-android</TargetFrameworks>
    <OutputType>Exe</OutputType>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <ApplicationId>com.yourcompany.yourapp</ApplicationId>
    <ApplicationVersion>1</ApplicationVersion>
    <ApplicationDisplayVersion>1.0.0</ApplicationDisplayVersion>
    
    <!-- Android specific properties -->
    <AndroidPackageFormat>aab</AndroidPackageFormat>
    <AndroidKeyStore>true</AndroidKeyStore>
    <AndroidSigningKeyStore>keystore.keystore</AndroidSigningKeyStore>
    <AndroidSigningKeyAlias>myapp</AndroidSigningKeyAlias>
    <AndroidSigningKeyPass>password</AndroidSigningKeyPass>
    <AndroidSigningStorePass>password</AndroidSigningStorePass>
    
    <!-- Enable R8 code shrinking -->
    <AndroidEnableProguard>true</AndroidEnableProguard>
    <AndroidLinkMode>SdkOnly</AndroidLinkMode>
    
    <!-- Target Android versions -->
    <SupportedOSPlatformVersion Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'android'">21.0</SupportedOSPlatformVersion>
    <TargetFramework Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'android'">net8.0-android34.0</TargetFramework>
  </PropertyGroup>
</Project>
```

#### Android Keystore Management

```bash
# Generate keystore
keytool -genkey -v -keystore myapp.keystore -alias myapp -keyalg RSA -keysize 2048 -validity 10000

# Verify keystore
keytool -list -v -keystore myapp.keystore

# Export certificate for upload to Play Console
keytool -export -rfc -alias myapp -file upload_certificate.pem -keystore myapp.keystore
```

#### Build Configuration for Release

```xml
<!-- Build configurations in .csproj -->
<PropertyGroup Condition="'$(Configuration)' == 'Release'">
  <AndroidPackageFormat>aab</AndroidPackageFormat>
  <AndroidUseAapt2>true</AndroidUseAapt2>
  <AndroidCreatePackagePerAbi>false</AndroidCreatePackagePerAbi>
  <AndroidSupportedAbis>arm64-v8a;armeabi-v7a;x86_64</AndroidSupportedAbis>
  
  <!-- Performance optimizations -->
  <PublishTrimmed>true</PublishTrimmed>
  <TrimMode>link</TrimMode>
  <AndroidLinkTool>r8</AndroidLinkTool>
  
  <!-- Security -->
  <AndroidHttpClientHandlerType>Xamarin.Android.Net.AndroidClientHandler</AndroidHttpClientHandlerType>
  <EmbedAssembliesIntoApk>true</EmbedAssembliesIntoApk>
</PropertyGroup>
```

### 1.2 iOS Deployment

#### iOS Provisioning & Certificates

```xml
<!-- iOS deployment settings in .csproj -->
<PropertyGroup Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'ios'">
  <SupportedOSPlatformVersion>11.0</SupportedOSPlatformVersion>
  <TargetFramework>net8.0-ios17.0</TargetFramework>
  
  <!-- Code signing -->
  <CodesignKey>iPhone Distribution: Your Company Name</CodesignKey>
  <CodesignProvision>YourApp_AppStore</CodesignProvision>
  <CodesignEntitlements>Platforms/iOS/Entitlements.plist</CodesignEntitlements>
  
  <!-- App Store configuration -->
  <MtouchArch>ARM64</MtouchArch>
  <MtouchLink>SdkOnly</MtouchLink>
  <MtouchHttpClientHandler>NSUrlSessionHandler</MtouchHttpClientHandler>
  <MtouchTlsProvider>Default</MtouchTlsProvider>
  
  <!-- Optimizations -->
  <MtouchEnableSGenConc>true</MtouchEnableSGenConc>
  <OptimizePNGs>true</OptimizePNGs>
</PropertyGroup>
```

#### Entitlements.plist Configuration

```xml
<!-- Platforms/iOS/Entitlements.plist -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <!-- Enable App Groups if needed -->
  <key>com.apple.security.application-groups</key>
  <array>
    <string>group.com.yourcompany.yourapp</string>
  </array>
  
  <!-- Push notifications -->
  <key>aps-environment</key>
  <string>production</string>
  
  <!-- Associated domains for deep linking -->
  <key>com.apple.developer.associated-domains</key>
  <array>
    <string>applinks:yourapp.com</string>
  </array>
  
  <!-- Background modes -->
  <key>UIBackgroundModes</key>
  <array>
    <string>background-fetch</string>
    <string>background-processing</string>
  </array>
</dict>
</plist>
```

### 1.3 Windows Deployment

#### Windows Package Configuration

```xml
<!-- Windows specific settings -->
<PropertyGroup Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'windows'">
  <SupportedOSPlatformVersion>10.0.17763.0</SupportedOSPlatformVersion>
  <TargetFramework>net8.0-windows10.0.19041.0</TargetFramework>
  <UseWinUI>true</UseWinUI>
  
  <!-- Package settings -->
  <WindowsPackageType>MSIX</WindowsPackageType>
  <WindowsAppSDKSelfContained>true</WindowsAppSDKSelfContained>
  
  <!-- Store association -->
  <GenerateAppInstallerFile>false</GenerateAppInstallerFile>
  <AppxAutoIncrementPackageRevision>false</AppxAutoIncrementPackageRevision>
  <AppxSymbolPackageEnabled>false</AppxSymbolPackageEnabled>
  <GenerateTestArtifacts>true</GenerateTestArtifacts>
</PropertyGroup>
```

---

## 2. 🔧 Build Automation & CI/CD

### 2.1 Azure DevOps Pipeline

#### azure-pipelines.yml

```yaml
# Azure DevOps pipeline for .NET MAUI
trigger:
  branches:
    include:
    - main
    - develop
  paths:
    exclude:
    - README.md
    - docs/*

variables:
  BuildConfiguration: 'Release'
  DotNetVersion: '8.0.x'
  
pool:
  vmImage: 'macOS-latest' # Required for iOS builds

stages:
- stage: Build
  displayName: 'Build Applications'
  jobs:
  - job: BuildAndroid
    displayName: 'Build Android'
    steps:
    - task: UseDotNet@2
      displayName: 'Install .NET SDK'
      inputs:
        packageType: 'sdk'
        version: '$(DotNetVersion)'
        
    - task: DotNetCoreCLI@2
      displayName: 'Restore NuGet packages'
      inputs:
        command: 'restore'
        projects: '**/*.csproj'
        
    - task: DotNetCoreCLI@2
      displayName: 'Build Android'
      inputs:
        command: 'build'
        projects: '**/*Android*.csproj'
        arguments: '--configuration $(BuildConfiguration) --framework net8.0-android'
        
    - task: DotNetCoreCLI@2
      displayName: 'Publish Android'
      inputs:
        command: 'publish'
        projects: '**/*Android*.csproj'
        arguments: '--configuration $(BuildConfiguration) --framework net8.0-android'
        publishWebProjects: false
        zipAfterPublish: false
        
    - task: AndroidSigning@3
      displayName: 'Sign Android Package'
      inputs:
        apkFiles: '**/*.aab'
        apksignerKeystoreFile: 'android-keystore.keystore'
        apksignerKeystorePassword: '$(AndroidKeystorePassword)'
        apksignerKeystoreAlias: '$(AndroidKeystoreAlias)'
        apksignerKeyPassword: '$(AndroidKeyPassword)'
        
    - task: PublishBuildArtifacts@1
      displayName: 'Publish Android Artifacts'
      inputs:
        pathToPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: 'Android'

  - job: BuildiOS
    displayName: 'Build iOS'
    steps:
    - task: UseDotNet@2
      displayName: 'Install .NET SDK'
      inputs:
        packageType: 'sdk'
        version: '$(DotNetVersion)'
        
    - task: InstallAppleCertificate@2
      displayName: 'Install Apple Certificate'
      inputs:
        certSecureFile: 'ios-distribution.p12'
        certPwd: '$(iOSCertificatePassword)'
        keychain: 'temp'
        
    - task: InstallAppleProvisioningProfile@1
      displayName: 'Install Provisioning Profile'
      inputs:
        provisioningProfileLocation: 'secureFiles'
        provProfileSecureFile: 'ios-appstore.mobileprovision'
        
    - task: DotNetCoreCLI@2
      displayName: 'Build iOS'
      inputs:
        command: 'build'
        projects: '**/*iOS*.csproj'
        arguments: '--configuration $(BuildConfiguration) --framework net8.0-ios'
        
    - task: DotNetCoreCLI@2
      displayName: 'Publish iOS'
      inputs:
        command: 'publish'
        projects: '**/*iOS*.csproj'
        arguments: '--configuration $(BuildConfiguration) --framework net8.0-ios'
        
    - task: PublishBuildArtifacts@1
      displayName: 'Publish iOS Artifacts'
      inputs:
        pathToPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: 'iOS'

- stage: Test
  displayName: 'Run Tests'
  dependsOn: Build
  jobs:
  - job: UnitTests
    displayName: 'Unit Tests'
    steps:
    - task: DotNetCoreCLI@2
      displayName: 'Run Unit Tests'
      inputs:
        command: 'test'
        projects: '**/*Tests*.csproj'
        arguments: '--configuration $(BuildConfiguration) --collect:"XPlat Code Coverage" --logger trx'
        
    - task: PublishTestResults@2
      displayName: 'Publish Test Results'
      inputs:
        testResultsFormat: 'VSTest'
        testResultsFiles: '**/*.trx'
        
    - task: PublishCodeCoverageResults@1
      displayName: 'Publish Code Coverage'
      inputs:
        codeCoverageTool: 'Cobertura'
        summaryFileLocation: '**/coverage.cobertura.xml'

- stage: Deploy
  displayName: 'Deploy to Stores'
  dependsOn: Test
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployAndroid
    displayName: 'Deploy to Google Play'
    environment: 'Production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: GooglePlayRelease@4
            displayName: 'Release to Google Play'
            inputs:
              serviceConnection: 'GooglePlay'
              applicationId: 'com.yourcompany.yourapp'
              bundleFile: '$(Pipeline.Workspace)/Android/**/*.aab'
              track: 'production'
              
  - deployment: DeployiOS
    displayName: 'Deploy to App Store'
    environment: 'Production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AppStoreRelease@1
            displayName: 'Release to App Store'
            inputs:
              serviceConnection: 'AppStore'
              appIdentifier: '$(iOSAppId)'
              ipaPath: '$(Pipeline.Workspace)/iOS/**/*.ipa'
              releaseTrack: 'TestFlight'
```

### 2.2 GitHub Actions

#### .github/workflows/maui-ci-cd.yml

```yaml
name: .NET MAUI CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  DOTNET_VERSION: '8.0.x'
  PROJECT_PATH: 'src/YourMauiApp/YourMauiApp.csproj'

jobs:
  build-android:
    runs-on: ubuntu-latest
    name: Android Build
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: ${{ env.DOTNET_VERSION }}
        
    - name: Install MAUI Workloads
      run: dotnet workload install maui --ignore-failed-sources
      
    - name: Restore dependencies
      run: dotnet restore ${{ env.PROJECT_PATH }}
      
    - name: Build
      run: dotnet build ${{ env.PROJECT_PATH }} -c Release -f net8.0-android --no-restore
      
    - name: Test
      run: dotnet test --no-build --verbosity normal
      
    - name: Publish
      run: dotnet publish ${{ env.PROJECT_PATH }} -c Release -f net8.0-android --no-restore
      
    - name: Upload Android Artifact
      uses: actions/upload-artifact@v3
      with:
        name: android-app
        path: '**/*.aab'

  build-ios:
    runs-on: macos-latest
    name: iOS Build
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: ${{ env.DOTNET_VERSION }}
        
    - name: Install MAUI Workloads
      run: dotnet workload install maui --ignore-failed-sources
      
    - name: Restore dependencies
      run: dotnet restore ${{ env.PROJECT_PATH }}
      
    - name: Build
      run: dotnet build ${{ env.PROJECT_PATH }} -c Release -f net8.0-ios --no-restore
      
    - name: Upload iOS Artifact
      uses: actions/upload-artifact@v3
      with:
        name: ios-app
        path: '**/*.ipa'

  build-windows:
    runs-on: windows-latest
    name: Windows Build
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: ${{ env.DOTNET_VERSION }}
        
    - name: Install MAUI Workloads
      run: dotnet workload install maui --ignore-failed-sources
      
    - name: Restore dependencies
      run: dotnet restore ${{ env.PROJECT_PATH }}
      
    - name: Build
      run: dotnet build ${{ env.PROJECT_PATH }} -c Release -f net8.0-windows10.0.19041.0 --no-restore
      
    - name: Upload Windows Artifact
      uses: actions/upload-artifact@v3
      with:
        name: windows-app
        path: '**/*.msix'

  deploy:
    needs: [build-android, build-ios, build-windows]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
    - name: Download Artifacts
      uses: actions/download-artifact@v3
      
    - name: Deploy to Google Play
      uses: r0adkll/upload-google-play@v1
      with:
        serviceAccountJsonPlainText: ${{ secrets.GOOGLE_PLAY_SERVICE_ACCOUNT }}
        packageName: com.yourcompany.yourapp
        releaseFiles: android-app/**/*.aab
        track: production
        
    - name: Deploy to App Store Connect
      uses: apple-actions/upload-testflight-build@v1
      with:
        app-path: ios-app/**/*.ipa
        issuer-id: ${{ secrets.APPSTORE_ISSUER_ID }}
        api-key-id: ${{ secrets.APPSTORE_API_KEY_ID }}
        api-private-key: ${{ secrets.APPSTORE_API_PRIVATE_KEY }}
```

### 2.3 Environment Configuration

#### Build Configuration Manager

```csharp
// Services/EnvironmentService.cs
public class EnvironmentService : IEnvironmentService
{
    public EnvironmentType Environment { get; private set; }
    public string ApiBaseUrl { get; private set; }
    public string AppCenterKey { get; private set; }
    
    public EnvironmentService()
    {
        SetEnvironment();
    }
    
    private void SetEnvironment()
    {
#if DEBUG
        Environment = EnvironmentType.Development;
        ApiBaseUrl = "https://api-dev.yourapp.com";
        AppCenterKey = "your-dev-appcenter-key";
#elif STAGING
        Environment = EnvironmentType.Staging;
        ApiBaseUrl = "https://api-staging.yourapp.com"; 
        AppCenterKey = "your-staging-appcenter-key";
#elif RELEASE
        Environment = EnvironmentType.Production;
        ApiBaseUrl = "https://api.yourapp.com";
        AppCenterKey = "your-prod-appcenter-key";
#endif
    }
    
    public bool IsProduction => Environment == EnvironmentType.Production;
    public bool IsDevelopment => Environment == EnvironmentType.Development;
}

public enum EnvironmentType
{
    Development,
    Staging,
    Production
}

public interface IEnvironmentService
{
    EnvironmentType Environment { get; }
    string ApiBaseUrl { get; }
    string AppCenterKey { get; }
    bool IsProduction { get; }
    bool IsDevelopment { get; }
}
```

---

## 3. 📱 App Store Optimization

### 3.1 App Store Metadata

#### Store Listing Optimization

```csharp
// Models/StoreMetadata.cs
public class StoreMetadata
{
    public string AppName { get; set; } = "Your Amazing App";
    public string ShortDescription { get; set; } = "Transform your daily productivity";
    public string LongDescription { get; set; } = @"
        Experience the future of mobile productivity with our revolutionary app.
        
        Key Features:
        • Intuitive and beautiful interface
        • Cross-platform synchronization
        • Advanced security features
        • Offline functionality
        • Real-time collaboration
        
        Perfect for professionals, students, and anyone who values efficiency.
        Download now and join millions of satisfied users worldwide!
    ";
    
    public List<string> Keywords { get; set; } = new()
    {
        "productivity", "efficiency", "mobile", "sync", "collaboration"
    };
    
    public List<string> Screenshots { get; set; } = new()
    {
        "screenshot1.png", "screenshot2.png", "screenshot3.png"
    };
    
    public string PrivacyPolicyUrl { get; set; } = "https://yourapp.com/privacy";
    public string TermsOfServiceUrl { get; set; } = "https://yourapp.com/terms";
    public string SupportUrl { get; set; } = "https://yourapp.com/support";
}
```

#### App Rating & Review Management

```csharp
// Services/AppReviewService.cs
public class AppReviewService : IAppReviewService
{
    private const int LaunchCountThreshold = 10;
    private const int DaysSinceInstallThreshold = 7;
    
    public async Task<bool> ShouldRequestReview()
    {
        var launchCount = Preferences.Default.Get("LaunchCount", 0);
        var installDate = Preferences.Default.Get("InstallDate", DateTime.MinValue);
        var lastReviewRequest = Preferences.Default.Get("LastReviewRequest", DateTime.MinValue);
        var hasRated = Preferences.Default.Get("HasRated", false);
        
        if (hasRated) return false;
        
        var daysSinceInstall = (DateTime.Now - installDate).TotalDays;
        var daysSinceLastRequest = (DateTime.Now - lastReviewRequest).TotalDays;
        
        return launchCount >= LaunchCountThreshold &&
               daysSinceInstall >= DaysSinceInstallThreshold &&
               daysSinceLastRequest >= 30; // Don't ask too frequently
    }
    
    public async Task RequestReview()
    {
        try
        {
#if ANDROID
            await RequestAndroidReview();
#elif IOS
            await RequestiOSReview();
#endif
            
            Preferences.Default.Set("LastReviewRequest", DateTime.Now);
        }
        catch (Exception ex)
        {
            // Log error but don't crash
            System.Diagnostics.Debug.WriteLine($"Review request failed: {ex.Message}");
        }
    }
    
#if ANDROID
    private async Task RequestAndroidReview()
    {
        var reviewManager = ReviewManagerFactory.Create(Platform.CurrentActivity);
        var request = reviewManager.RequestReviewFlow();
        
        if (request.IsSuccessful)
        {
            var reviewInfo = request.Result;
            var flow = reviewManager.LaunchReviewFlow(Platform.CurrentActivity, reviewInfo);
        }
    }
#endif

#if IOS
    private async Task RequestiOSReview()
    {
        if (UIDevice.CurrentDevice.CheckSystemVersion(14, 0))
        {
            await StoreKit.SKStoreReviewController.RequestReview();
        }
    }
#endif

    public void IncrementLaunchCount()
    {
        var count = Preferences.Default.Get("LaunchCount", 0);
        Preferences.Default.Set("LaunchCount", count + 1);
        
        if (count == 0)
        {
            Preferences.Default.Set("InstallDate", DateTime.Now);
        }
    }
}

public interface IAppReviewService
{
    Task<bool> ShouldRequestReview();
    Task RequestReview();
    void IncrementLaunchCount();
}
```

### 3.2 App Analytics & Tracking

#### App Center Integration

```csharp
// App.xaml.cs
public partial class App : Application
{
    public App()
    {
        InitializeComponent();
        
        // Initialize App Center
        var environmentService = ServiceHelper.GetService<IEnvironmentService>();
        AppCenter.Start(environmentService.AppCenterKey, 
            typeof(Analytics), typeof(Crashes), typeof(Distribute));
            
        MainPage = new AppShell();
    }
    
    protected override void OnStart()
    {
        // Track app launch
        var reviewService = ServiceHelper.GetService<IAppReviewService>();
        reviewService.IncrementLaunchCount();
        
        Analytics.TrackEvent("AppLaunched", new Dictionary<string, string>
        {
            { "Version", AppInfo.VersionString },
            { "BuildNumber", AppInfo.BuildString },
            { "Platform", DeviceInfo.Platform.ToString() },
            { "DeviceModel", DeviceInfo.Model },
            { "OSVersion", DeviceInfo.VersionString }
        });
    }
    
    protected override void OnSleep()
    {
        Analytics.TrackEvent("AppBackgrounded");
    }
    
    protected override void OnResume()
    {
        Analytics.TrackEvent("AppResumed");
    }
}
```

#### Custom Analytics Service

```csharp
// Services/AnalyticsService.cs
public class AnalyticsService : IAnalyticsService
{
    public void TrackEvent(string eventName, Dictionary<string, string> properties = null)
    {
        try
        {
            properties ??= new Dictionary<string, string>();
            
            // Add common properties
            properties["Timestamp"] = DateTime.UtcNow.ToString("yyyy-MM-ddTHH:mm:ssZ");
            properties["AppVersion"] = AppInfo.VersionString;
            properties["Platform"] = DeviceInfo.Platform.ToString();
            
            Analytics.TrackEvent(eventName, properties);
            
            // Also log to debug console in development
#if DEBUG
            System.Diagnostics.Debug.WriteLine($"Analytics: {eventName} - {string.Join(", ", properties.Select(p => $"{p.Key}={p.Value}"))}");
#endif
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"Analytics tracking failed: {ex.Message}");
        }
    }
    
    public void TrackPageView(string pageName, string source = null)
    {
        var properties = new Dictionary<string, string>
        {
            { "PageName", pageName }
        };
        
        if (!string.IsNullOrEmpty(source))
            properties["Source"] = source;
            
        TrackEvent("PageView", properties);
    }
    
    public void TrackUserAction(string action, string target, Dictionary<string, string> additionalData = null)
    {
        var properties = new Dictionary<string, string>
        {
            { "Action", action },
            { "Target", target }
        };
        
        if (additionalData != null)
        {
            foreach (var kvp in additionalData)
                properties[kvp.Key] = kvp.Value;
        }
        
        TrackEvent("UserAction", properties);
    }
    
    public void TrackError(Exception exception, Dictionary<string, string> properties = null)
    {
        properties ??= new Dictionary<string, string>();
        properties["ErrorType"] = exception.GetType().Name;
        
        Crashes.TrackError(exception, properties);
    }
}

public interface IAnalyticsService
{
    void TrackEvent(string eventName, Dictionary<string, string> properties = null);
    void TrackPageView(string pageName, string source = null);
    void TrackUserAction(string action, string target, Dictionary<string, string> additionalData = null);
    void TrackError(Exception exception, Dictionary<string, string> properties = null);
}
```

---

## 4. 🔐 Security & Compliance

### 4.1 Code Obfuscation

#### Obfuscation Configuration

```xml
<!-- Obfuscation settings in .csproj -->
<PropertyGroup Condition="'$(Configuration)' == 'Release'">
  <!-- Enable assembly linking -->
  <PublishTrimmed>true</PublishTrimmed>
  <TrimMode>link</TrimMode>
  
  <!-- Android specific obfuscation -->
  <AndroidEnableProguard Condition="$(TargetFramework.Contains('-android'))">true</AndroidEnableProguard>
  <AndroidLinkMode Condition="$(TargetFramework.Contains('-android'))">Full</AndroidLinkMode>
  
  <!-- iOS specific obfuscation -->
  <MtouchLink Condition="$(TargetFramework.Contains('-ios'))">Full</MtouchLink>
  <MtouchArch Condition="$(TargetFramework.Contains('-ios'))">ARM64</MtouchArch>
</PropertyGroup>

<!-- Preserve important assemblies and types -->
<ItemGroup>
  <TrimmerRootAssembly Include="YourApp.Core" />
  <TrimmerRootDescriptor Include="TrimmerRoots.xml" />
</ItemGroup>
```

#### TrimmerRoots.xml

```xml
<!-- TrimmerRoots.xml - Preserve important types -->
<linker>
  <assembly fullname="YourApp" preserve="all" />
  <assembly fullname="YourApp.Core">
    <type fullname="YourApp.Core.Models.*" preserve="all" />
    <type fullname="YourApp.Core.Services.*" preserve="all" />
  </assembly>
  
  <!-- Preserve reflection-used types -->
  <assembly fullname="System.Text.Json">
    <type fullname="System.Text.Json.JsonSerializer" preserve="all" />
  </assembly>
</linker>
```

### 4.2 Certificate Pinning

#### Network Security Configuration

```csharp
// Services/SecureHttpClientService.cs
public class SecureHttpClientService : IHttpClientService
{
    private readonly HttpClient _httpClient;
    
    public SecureHttpClientService()
    {
        var handler = new HttpClientHandler();
        
#if ANDROID
        // Certificate pinning for Android
        handler.ServerCertificateCustomValidationCallback = ValidateServerCertificate;
#elif IOS
        // Certificate pinning for iOS
        var nsUrlSessionHandler = new NSUrlSessionHandler
        {
            TrustOverrideForUrl = ValidateServerCertificateiOS
        };
        _httpClient = new HttpClient(nsUrlSessionHandler);
        return;
#endif
        
        _httpClient = new HttpClient(handler);
        _httpClient.DefaultRequestHeaders.Add("User-Agent", GetUserAgent());
        _httpClient.Timeout = TimeSpan.FromSeconds(30);
    }
    
    private bool ValidateServerCertificate(HttpRequestMessage request, X509Certificate2 certificate, X509Chain chain, SslPolicyErrors errors)
    {
        // Implement certificate pinning
        var expectedThumbprints = new[]
        {
            "YOUR_CERTIFICATE_THUMBPRINT_HERE",
            "BACKUP_CERTIFICATE_THUMBPRINT_HERE"
        };
        
        var actualThumbprint = certificate.Thumbprint;
        return expectedThumbprints.Contains(actualThumbprint);
    }
    
#if IOS
    private bool ValidateServerCertificateiOS(NSUrlSessionHandler sender, string url, Security.SecTrust trust)
    {
        // iOS certificate validation
        var certificate = trust.GetCertificate(0);
        var certificateData = certificate.GetNormalizedIssuerSequence();
        
        // Compare with pinned certificate
        return ValidateCertificateData(certificateData);
    }
    
    private bool ValidateCertificateData(byte[] certificateData)
    {
        // Implement your certificate validation logic
        return true; // Simplified for example
    }
#endif
    
    private string GetUserAgent()
    {
        return $"YourApp/{AppInfo.VersionString} ({DeviceInfo.Platform} {DeviceInfo.VersionString})";
    }
    
    public async Task<T> GetAsync<T>(string endpoint)
    {
        var response = await _httpClient.GetAsync(endpoint);
        response.EnsureSuccessStatusCode();
        
        var json = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<T>(json);
    }
    
    public void Dispose()
    {
        _httpClient?.Dispose();
    }
}
```

---

## 5. 📊 Version Management & Updates

### 5.1 App Version Management

#### Version Management Service

```csharp
// Services/VersionService.cs
public class VersionService : IVersionService
{
    private readonly IHttpClientService _httpClient;
    private readonly IAnalyticsService _analytics;
    
    public VersionService(IHttpClientService httpClient, IAnalyticsService analytics)
    {
        _httpClient = httpClient;
        _analytics = analytics;
    }
    
    public Version CurrentVersion => new Version(AppInfo.VersionString);
    public string BuildNumber => AppInfo.BuildString;
    
    public async Task<UpdateInfo> CheckForUpdates()
    {
        try
        {
            var platform = DeviceInfo.Platform.ToString().ToLower();
            var updateInfo = await _httpClient.GetAsync<UpdateInfo>($"/api/updates/{platform}");
            
            if (updateInfo != null)
            {
                updateInfo.IsUpdateAvailable = new Version(updateInfo.LatestVersion) > CurrentVersion;
                
                _analytics.TrackEvent("UpdateCheck", new Dictionary<string, string>
                {
                    { "CurrentVersion", CurrentVersion.ToString() },
                    { "LatestVersion", updateInfo.LatestVersion },
                    { "IsUpdateAvailable", updateInfo.IsUpdateAvailable.ToString() }
                });
            }
            
            return updateInfo;
        }
        catch (Exception ex)
        {
            _analytics.TrackError(ex);
            return null;
        }
    }
    
    public async Task<bool> ShowUpdateDialog(UpdateInfo updateInfo)
    {
        if (!updateInfo.IsUpdateAvailable) return false;
        
        var message = updateInfo.IsForced 
            ? "A required update is available. Please update to continue using the app."
            : $"Version {updateInfo.LatestVersion} is available. Would you like to update now?\n\n{updateInfo.ReleaseNotes}";
            
        var title = updateInfo.IsForced ? "Required Update" : "Update Available";
        var acceptText = updateInfo.IsForced ? "Update Now" : "Update";
        var cancelText = updateInfo.IsForced ? null : "Later";
        
        var result = await Application.Current.MainPage.DisplayAlert(title, message, acceptText, cancelText);
        
        if (result)
        {
            await OpenStoreForUpdate();
            _analytics.TrackEvent("UpdateAccepted", new Dictionary<string, string>
            {
                { "Version", updateInfo.LatestVersion },
                { "IsForced", updateInfo.IsForced.ToString() }
            });
        }
        else if (!updateInfo.IsForced)
        {
            _analytics.TrackEvent("UpdateDeclined", new Dictionary<string, string>
            {
                { "Version", updateInfo.LatestVersion }
            });
        }
        
        return result;
    }
    
    private async Task OpenStoreForUpdate()
    {
        try
        {
#if ANDROID
            var packageName = AppInfo.PackageName;
            var uri = $"market://details?id={packageName}";
            await Launcher.OpenAsync(uri);
#elif IOS
            var appId = "YOUR_IOS_APP_ID";
            var uri = $"itms-apps://apps.apple.com/app/id{appId}";
            await Launcher.OpenAsync(uri);
#elif WINDOWS
            var familyName = "YOUR_WINDOWS_FAMILY_NAME";
            var uri = $"ms-windows-store://pdp/?ProductId={familyName}";
            await Launcher.OpenAsync(uri);
#endif
        }
        catch (Exception ex)
        {
            _analytics.TrackError(ex);
            await Application.Current.MainPage.DisplayAlert("Error", "Unable to open store for update", "OK");
        }
    }
}

public class UpdateInfo
{
    public string LatestVersion { get; set; }
    public string ReleaseNotes { get; set; }
    public bool IsForced { get; set; }
    public DateTime ReleaseDate { get; set; }
    public string DownloadUrl { get; set; }
    public bool IsUpdateAvailable { get; set; }
}

public interface IVersionService
{
    Version CurrentVersion { get; }
    string BuildNumber { get; }
    Task<UpdateInfo> CheckForUpdates();
    Task<bool> ShowUpdateDialog(UpdateInfo updateInfo);
}
```

### 5.2 Feature Flags & A/B Testing

#### Feature Flag Service

```csharp
// Services/FeatureFlagService.cs
public class FeatureFlagService : IFeatureFlagService
{
    private readonly Dictionary<string, bool> _localFlags;
    private readonly IHttpClientService _httpClient;
    
    public FeatureFlagService(IHttpClientService httpClient)
    {
        _httpClient = httpClient;
        _localFlags = new Dictionary<string, bool>
        {
            { "NewUserInterface", false },
            { "AdvancedAnalytics", true },
            { "PremiumFeatures", false },
            { "ExperimentalMode", false }
        };
    }
    
    public async Task InitializeAsync()
    {
        try
        {
            var remoteFlags = await _httpClient.GetAsync<Dictionary<string, bool>>("/api/feature-flags");
            
            if (remoteFlags != null)
            {
                foreach (var flag in remoteFlags)
                {
                    _localFlags[flag.Key] = flag.Value;
                }
            }
        }
        catch
        {
            // Use local defaults if remote fetch fails
        }
    }
    
    public bool IsEnabled(string featureName)
    {
        return _localFlags.TryGetValue(featureName, out bool value) && value;
    }
    
    public T GetVariant<T>(string experimentName, T defaultValue)
    {
        // A/B testing implementation
        var userId = GetUserId();
        var hash = HashFunction(userId + experimentName);
        var bucket = hash % 100;
        
        return experimentName switch
        {
            "ButtonColorExperiment" => bucket < 50 ? (T)(object)"Blue" : (T)(object)"Green",
            "OnboardingFlow" => bucket < 33 ? (T)(object)"Standard" : 
                              bucket < 66 ? (T)(object)"Enhanced" : (T)(object)"Minimal",
            _ => defaultValue
        };
    }
    
    private string GetUserId()
    {
        return Preferences.Default.Get("UserId", Guid.NewGuid().ToString());
    }
    
    private int HashFunction(string input)
    {
        return Math.Abs(input.GetHashCode());
    }
}

public interface IFeatureFlagService
{
    Task InitializeAsync();
    bool IsEnabled(string featureName);
    T GetVariant<T>(string experimentName, T defaultValue);
}
```

---

## 6. 🧩 Practical Exercise

### 🎯 Complete Deployment Pipeline Setup

ในแบบฝึกหัดนี้ คุณจะสร้าง complete deployment pipeline สำหรับ .NET MAUI application:

1. **Setup Project Configuration:**
   - กำหนด Android signing configuration
   - สร้าง iOS provisioning profiles
   - ตั้งค่า Windows packaging

2. **Create CI/CD Pipeline:**
   - สร้าง Azure DevOps หรือ GitHub Actions pipeline
   - ตั้งค่า automated testing
   - กำหนด deployment stages

3. **Implement App Store Features:**
   - เพิ่ม app rating request system
   - สร้าง analytics tracking
   - ใช้งาน feature flags

4. **Security Implementation:**
   - เพิ่ม certificate pinning
   - ใช้งาน code obfuscation
   - สร้าง secure storage

5. **Version Management:**
   - สร้าง automatic update checking
   - ใช้งาน A/B testing framework
   - จัดการ app versioning

### Model Solution Highlights:

```csharp
// MauiProgram.cs - Complete service registration
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
            })
            .RegisterServices();

        return builder.Build();
    }
    
    private static MauiAppBuilder RegisterServices(this MauiAppBuilder builder)
    {
        // Core services
        builder.Services.AddSingleton<IEnvironmentService, EnvironmentService>();
        builder.Services.AddSingleton<IAnalyticsService, AnalyticsService>();
        builder.Services.AddSingleton<IHttpClientService, SecureHttpClientService>();
        
        // Deployment related services
        builder.Services.AddSingleton<IVersionService, VersionService>();
        builder.Services.AddSingleton<IFeatureFlagService, FeatureFlagService>();
        builder.Services.AddSingleton<IAppReviewService, AppReviewService>();
        
        return builder;
    }
}
```

---

## 📚 Summary

ในบทนี้เราได้เรียนรู้:

### 🔑 Key Concepts
- **Platform-Specific Deployment**: การ deploy ไปยัง Android, iOS และ Windows แต่ละแพลตฟอร์มมีความต้องการที่แตกต่างกัน
- **CI/CD Automation**: การใช้ Azure DevOps และ GitHub Actions สำหรับ automated build และ deployment
- **App Store Optimization**: การเพิ่มประสิทธิภาพสำหรับ app stores และการจัดการ reviews
- **Security Best Practices**: Certificate pinning, code obfuscation และ secure communication

### 🛠️ Tools & Services
- **Build Tools**: .NET CLI, MSBuild, platform-specific SDKs
- **CI/CD Platforms**: Azure DevOps, GitHub Actions
- **Analytics**: App Center Analytics, custom analytics services
- **Security**: Certificate management, obfuscation tools

### 📱 Store Management
- **Google Play**: AAB packaging, automated releases
- **App Store**: iOS provisioning, TestFlight distribution
- **Microsoft Store**: MSIX packaging, store submission

### 🔧 Advanced Features
- **Feature Flags**: A/B testing และ gradual rollout capabilities
- **Version Management**: Automatic update checking และ forced updates
- **Analytics Integration**: User behavior tracking และ crash reporting

การเข้าใจและใช้งาน deployment และ distribution อย่างมีประสิทธิภาพจะช่วยให้คุณสามารถส่งมอบแอปพลิเคชันคุณภาพสูงให้กับผู้ใช้ได้อย่างสม่ำเสมอและปลอดภัย

**Next Chapter Preview:** ในบทต่อไป เราจะเจาะลึกไปที่ **Advanced Patterns & Architectural Strategies** ที่จะทำให้โค้ดของคุณมีคุณภาพและ maintainable มากขึ้น! 🏗️
