# 🚀 Chapter 16: Deployment & Distribution

## การเผยแพร่และจัดจำหน่าย MAUI Applications

### 📋 Chapter Overview

ในบทนี้เราจะเรียนรู้การเตรียม deploy และจัดจำหน่าย .NET MAUI applications ไปยัง app stores ต่างๆ รวมถึงการตั้งค่า CI/CD pipelines สำหรับการ deploy แบบอัตโนมัติ

### 🎯 Learning Objectives

- เตรียม MAUI app สำหรับการ production deployment
- Configure app store metadata และ certificates  
- ตั้งค่า CI/CD pipelines สำหรับ automated deployment
- จัดการ app versioning และ release management
- Deploy ไปยัง Google Play Store, Apple App Store, และ Microsoft Store
- ใช้งาน signing certificates และ security considerations

---

## 1. 📱 Android Deployment

### 1.1 Android App Bundle Configuration

```xml
<!-- Directory.Build.props - Global Android configuration -->
<Project>
  <PropertyGroup Condition="$(TargetFramework.Contains('-android'))">
    <!-- Release Configuration -->
    <AndroidKeyStore Condition="'$(Configuration)' == 'Release'">true</AndroidKeyStore>
    <AndroidSigningKeyStore Condition="'$(Configuration)' == 'Release'">$(MSBuildThisFileDirectory)MyApp.keystore</AndroidSigningKeyStore>
    <AndroidSigningKeyAlias Condition="'$(Configuration)' == 'Release'">myappkey</AndroidSigningKeyAlias>
    <AndroidSigningKeyPass Condition="'$(Configuration)' == 'Release'">$(KeystorePassword)</AndroidSigningKeyPass>
    <AndroidSigningStorePass Condition="'$(Configuration)' == 'Release'">$(KeystorePassword)</AndroidSigningStorePass>
    
    <!-- App Bundle Configuration -->
    <AndroidPackageFormat>aab</AndroidPackageFormat>
    <AndroidUseAapt2>true</AndroidUseAapt2>
    <AndroidCreatePackagePerAbi>false</AndroidCreatePackagePerAbi>
    
    <!-- Optimization Settings -->
    <AndroidDexTool>d8</AndroidDexTool>
    <AndroidLinkTool>r8</AndroidLinkTool>
    <AndroidLinkMode>SdkOnly</AndroidLinkMode>
    <AndroidLinkSkip></AndroidLinkSkip>
    <AndroidLinkResources>true</AndroidLinkResources>
    
    <!-- ProGuard Configuration -->
    <AndroidEnableProguard>true</AndroidEnableProguard>
    <AndroidR8JarPath>$(AndroidSdkDirectory)build-tools\$(AndroidBuildToolsVersion)\lib\r8.jar</AndroidR8JarPath>
  </PropertyGroup>
</Project>
```

### 1.2 Android Manifest Optimization

```xml
<!-- Platforms/Android/AndroidManifest.xml -->
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    
    <!-- App Information -->
    <application 
        android:allowBackup="true"
        android:icon="@mipmap/appicon" 
        android:supportsRtl="true"
        android:theme="@style/Maui.SplashTheme"
        android:usesCleartextTraffic="false"
        android:hardwareAccelerated="true"
        android:largeHeap="false"
        android:extractNativeLibs="false">
        
        <!-- App Startup Configuration -->
        <activity 
            android:name="crc64e1fb321c08285b90.MainActivity" 
            android:exported="true" 
            android:launchMode="singleTop" 
            android:theme="@style/Maui.SplashTheme"
            android:configChanges="orientation|keyboardHidden|keyboard|screenSize|smallestScreenSize|density|screenLayout|uiMode"
            android:resizeableActivity="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        
        <!-- File Provider for Sharing -->
        <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="${applicationId}.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths" />
        </provider>
    </application>
    
    <!-- Required Permissions -->
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.VIBRATE" />
    
    <!-- Optional Permissions with proper targeting -->
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" 
                     android:maxSdkVersion="32" />
    <uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
    
    <!-- Hardware Features -->
    <uses-feature android:name="android.hardware.camera" android:required="false" />
    <uses-feature android:name="android.hardware.camera.autofocus" android:required="false" />
    
    <!-- API Level Support -->
    <uses-sdk android:minSdkVersion="21" android:targetSdkVersion="34" />
</manifest>
```

### 1.3 ProGuard Rules

```bash
# proguard-rules.pro - Android code optimization rules
-verbose

# Keep application classes
-keep class com.yourcompany.yourapp.** { *; }

# MAUI Specific Rules
-keep class Microsoft.Maui.** { *; }
-keep class AndroidX.** { *; }
-keep class Xamarin.** { *; }

# Preserve all native method names and the names of their classes
-keepclasseswithmembernames class * {
    native <methods>;
}

# Keep all classes that have @Keep annotation
-keep @androidx.annotation.Keep class *
-keepclassmembers class * {
    @androidx.annotation.Keep *;
}

# Gson/JSON specific rules
-keepattributes Signature
-keepattributes *Annotation*
-keep class sun.misc.Unsafe { *; }
-keep class com.google.gson.stream.** { *; }

# Network security
-keep class okhttp3.** { *; }
-keep class retrofit2.** { *; }

# Remove logging in release builds
-assumenosideeffects class android.util.Log {
    public static boolean isLoggable(java.lang.String, int);
    public static int v(...);
    public static int i(...);
    public static int w(...);
    public static int d(...);
    public static int e(...);
}

# Optimize and obfuscate
-optimizations !code/simplification/arithmetic,!code/simplification/cast,!field/*,!class/merging/*
-optimizationpasses 5
-allowaccessmodification
-dontpreverify
```

---

## 2. 🍎 iOS Deployment

### 2.1 iOS Build Configuration

```xml
<!-- iOS-specific properties in .csproj -->
<PropertyGroup Condition="$(TargetFramework.Contains('-ios'))">
  <!-- Code Signing -->
  <CodesignKey Condition="'$(Configuration)' == 'Release'">iPhone Distribution</CodesignKey>
  <CodesignProvision Condition="'$(Configuration)' == 'Release'">YourApp Distribution Profile</CodesignProvision>
  <CodesignEntitlements>Platforms/iOS/Entitlements.plist</CodesignEntitlements>
  
  <!-- Build Settings -->
  <MtouchArch>ARM64</MtouchArch>
  <MtouchLink>SdkOnly</MtouchLink>
  <MtouchUseLlvm>true</MtouchUseLlvm>
  <MtouchUseThumb>true</MtouchUseThumb>
  <MtouchEnableBitcode>false</MtouchEnableBitcode>
  
  <!-- App Store Configuration -->
  <IpaPackageName>$(AssemblyName)</IpaPackageName>
  <BuildIpa Condition="'$(Configuration)' == 'Release'">true</BuildIpa>
  
  <!-- Privacy and Security -->
  <MtouchExtraArgs>--registrar:static</MtouchExtraArgs>
  <MtouchFastDev>false</MtouchFastDev>
  <MtouchUseSGen>true</MtouchUseSGen>
</PropertyGroup>
```

### 2.2 Info.plist Configuration

```xml
<!-- Platforms/iOS/Info.plist -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- App Identity -->
    <key>CFBundleDisplayName</key>
    <string>YourApp</string>
    <key>CFBundleIdentifier</key>
    <string>com.yourcompany.yourapp</string>
    <key>CFBundleVersion</key>
    <string>1.0.0</string>
    <key>CFBundleShortVersionString</key>
    <string>1.0</string>
    
    <!-- Supported Device Types -->
    <key>LSRequiresIPhoneOS</key>
    <true/>
    <key>UIDeviceFamily</key>
    <array>
        <integer>1</integer> <!-- iPhone -->
        <integer>2</integer> <!-- iPad -->
    </array>
    
    <!-- Supported Interface Orientations -->
    <key>UISupportedInterfaceOrientations</key>
    <array>
        <string>UIInterfaceOrientationPortrait</string>
        <string>UIInterfaceOrientationLandscapeLeft</string>
        <string>UIInterfaceOrientationLandscapeRight</string>
    </array>
    
    <!-- iPad Specific Orientations -->
    <key>UISupportedInterfaceOrientations~ipad</key>
    <array>
        <string>UIInterfaceOrientationPortrait</string>
        <string>UIInterfaceOrientationPortraitUpsideDown</string>
        <string>UIInterfaceOrientationLandscapeLeft</string>
        <string>UIInterfaceOrientationLandscapeRight</string>
    </array>
    
    <!-- Privacy Usage Descriptions -->
    <key>NSCameraUsageDescription</key>
    <string>This app uses camera to take photos for your profile</string>
    <key>NSPhotoLibraryUsageDescription</key>
    <string>This app needs access to photo library to select images</string>
    <key>NSLocationWhenInUseUsageDescription</key>
    <string>This app uses location services to provide location-based features</string>
    
    <!-- App Transport Security -->
    <key>NSAppTransportSecurity</key>
    <dict>
        <key>NSAllowsArbitraryLoads</key>
        <false/>
        <key>NSAllowsLocalNetworking</key>
        <true/>
    </dict>
    
    <!-- Background Modes -->
    <key>UIBackgroundModes</key>
    <array>
        <string>background-fetch</string>
        <string>background-processing</string>
    </array>
    
    <!-- App Icons and Launch Images -->
    <key>CFBundleIcons</key>
    <dict>
        <key>CFBundlePrimaryIcon</key>
        <dict>
            <key>CFBundleIconFiles</key>
            <array>
                <string>appicon</string>
            </array>
        </dict>
    </dict>
    
    <!-- Minimum iOS Version -->
    <key>MinimumOSVersion</key>
    <string>11.0</string>
    
    <!-- Scene Configuration -->
    <key>UIApplicationSceneManifest</key>
    <dict>
        <key>UIApplicationSupportsMultipleScenes</key>
        <true/>
        <key>UISceneConfigurations</key>
        <dict>
            <key>UIWindowSceneSessionRoleApplication</key>
            <array>
                <dict>
                    <key>UISceneConfigurationName</key>
                    <string>Default Configuration</string>
                    <key>UISceneDelegateClassName</key>
                    <string>SceneDelegate</string>
                </dict>
            </array>
        </dict>
    </dict>
</dict>
</plist>
```

### 2.3 Entitlements Configuration

```xml
<!-- Platforms/iOS/Entitlements.plist -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- App Groups (for sharing data between apps) -->
    <key>com.apple.security.application-groups</key>
    <array>
        <string>group.com.yourcompany.yourapp</string>
    </array>
    
    <!-- Keychain Access Groups -->
    <key>keychain-access-groups</key>
    <array>
        <string>$(AppIdentifierPrefix)com.yourcompany.yourapp</string>
    </array>
    
    <!-- Associated Domains -->
    <key>com.apple.developer.associated-domains</key>
    <array>
        <string>applinks:yourapp.com</string>
    </array>
    
    <!-- Background Modes -->
    <key>com.apple.developer.background-modes</key>
    <array>
        <string>background-fetch</string>
        <string>background-processing</string>
    </array>
    
    <!-- Push Notifications -->
    <key>aps-environment</key>
    <string>production</string>
    
    <!-- In-App Purchase -->
    <key>com.apple.developer.in-app-payments</key>
    <array>
        <string>merchant.com.yourcompany.yourapp</string>
    </array>
</dict>
</plist>
```

---

## 3. 🪟 Windows Deployment

### 3.1 Package.appxmanifest Configuration

```xml
<!-- Platforms/Windows/Package.appxmanifest -->
<?xml version="1.0" encoding="utf-8"?>
<Package
  xmlns="http://schemas.microsoft.com/appx/manifest/foundation/windows10"
  xmlns:mp="http://schemas.microsoft.com/appx/2014/phone/manifest"
  xmlns:uap="http://schemas.microsoft.com/appx/manifest/uap/windows10"
  xmlns:rescap="http://schemas.microsoft.com/appx/manifest/foundation/windows10/restrictedcapabilities"
  IgnorableNamespaces="uap rescap">

  <Identity
    Name="YourCompany.YourApp"
    Publisher="CN=Your Company Name"
    Version="1.0.0.0" />

  <mp:PhoneIdentity PhoneProductId="your-phone-product-id" PhonePublisherId="00000000-0000-0000-0000-000000000000"/>

  <Properties>
    <DisplayName>Your App Name</DisplayName>
    <PublisherDisplayName>Your Company Name</PublisherDisplayName>
    <Logo>Images\StoreLogo.png</Logo>
    <Description>Your app description for Microsoft Store</Description>
  </Properties>

  <Dependencies>
    <TargetDeviceFamily Name="Windows.Universal" MinVersion="10.0.17763.0" MaxVersionTested="10.0.22000.0" />
    <TargetDeviceFamily Name="Windows.Desktop" MinVersion="10.0.17763.0" MaxVersionTested="10.0.22000.0" />
  </Dependencies>

  <Resources>
    <Resource Language="x-generate"/>
  </Resources>

  <Applications>
    <Application Id="App"
      Executable="$targetnametoken$.exe"
      EntryPoint="$targetentrypoint$">
      <uap:VisualElements
        DisplayName="Your App"
        Description="Your app description"
        BackgroundColor="#1e1e1e"
        Square150x150Logo="Images\MediumTile.png"
        Square44x44Logo="Images\AppList.png">
        <uap:DefaultTile
          Wide310x150Logo="Images\WideTile.png"
          Square71x71Logo="Images\SmallTile.png"
          Square310x310Logo="Images\LargeTile.png"
          ShortName="YourApp">
          <uap:ShowNameOnTiles>
            <uap:ShowOn Tile="square150x150Logo"/>
            <uap:ShowOn Tile="wide310x150Logo"/>
            <uap:ShowOn Tile="square310x310Logo"/>
          </uap:ShowNameOnTiles>
        </uap:DefaultTile>
        <uap:SplashScreen Image="Images\SplashScreen.png" />
      </uap:VisualElements>
      
      <!-- File Associations -->
      <Extensions>
        <uap:Extension Category="windows.fileTypeAssociation">
          <uap:FileTypeAssociation Name="yourappfile">
            <uap:SupportedFileTypes>
              <uap:FileType>.yaf</uap:FileType>
            </uap:SupportedFileTypes>
          </uap:FileTypeAssociation>
        </uap:Extension>
        
        <!-- Protocol Handler -->
        <uap:Extension Category="windows.protocol">
          <uap:Protocol Name="yourapp">
            <uap:DisplayName>Your App Protocol</uap:DisplayName>
          </uap:Protocol>
        </uap:Extension>
      </Extensions>
    </Application>
  </Applications>

  <Capabilities>
    <Capability Name="internetClient" />
    <uap:Capability Name="documentsLibrary" />
    <uap:Capability Name="picturesLibrary" />
    <uap:Capability Name="removableStorage" />
    <rescap:Capability Name="broadFileSystemAccess" />
  </Capabilities>
</Package>
```

### 3.2 Windows-specific Build Configuration

```xml
<!-- Windows-specific properties in .csproj -->
<PropertyGroup Condition="$(TargetFramework.Contains('-windows'))">
  <!-- Package Configuration -->
  <UseWinUI>true</UseWinUI>
  <WindowsAppSDKSelfContained>true</WindowsAppSDKSelfContained>
  <WindowsPackageType>MSIX</WindowsPackageType>
  
  <!-- Certificate Information -->
  <PackageCertificateThumbprint Condition="'$(Configuration)' == 'Release'">Your-Certificate-Thumbprint</PackageCertificateThumbprint>
  <PackageCertificateKeyFile Condition="'$(Configuration)' == 'Release'">YourApp_TemporaryKey.pfx</PackageCertificateKeyFile>
  <PackageCertificatePassword Condition="'$(Configuration)' == 'Release'">$(CertificatePassword)</PackageCertificatePassword>
  
  <!-- Store Association -->
  <GenerateAppInstallerFile>false</GenerateAppInstallerFile>
  <AppxPackageSigningEnabled>true</AppxPackageSigningEnabled>
  <AppxAutoIncrementPackageRevision>true</AppxAutoIncrementPackageRevision>
  <AppxSymbolPackageEnabled>false</AppxSymbolPackageEnabled>
  <GenerateTestArtifacts>true</GenerateTestArtifacts>
  
  <!-- Deployment Settings -->
  <HoursBetweenUpdateChecks>0</HoursBetweenUpdateChecks>
  <AppxBundle>Always</AppxBundle>
  <AppxBundlePlatforms>x86|x64|arm64</AppxBundlePlatforms>
</PropertyGroup>
```

---

## 4. 🔄 CI/CD Pipeline Configuration

### 4.1 GitHub Actions Workflow

```yaml
# .github/workflows/maui-cicd.yml
name: MAUI CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  release:
    types: [ published ]

jobs:
  build-android:
    runs-on: ubuntu-latest
    name: Android Build and Deploy
    
    steps:
    - name: Checkout
      uses: actions/checkout@v4
      
    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '8.0.x'
        
    - name: Setup Java JDK
      uses: actions/setup-java@v4
      with:
        distribution: 'microsoft'
        java-version: '11'
        
    - name: Install MAUI Workload
      run: dotnet workload install maui
      
    - name: Restore Dependencies
      run: dotnet restore YourApp.csproj
      
    - name: Build Android
      run: dotnet build YourApp.csproj -c Release -f net8.0-android --no-restore
      
    - name: Sign Android APK
      if: github.event_name == 'release'
      env:
        KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
        KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
      run: |
        echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 --decode > MyApp.keystore
        dotnet publish YourApp.csproj -c Release -f net8.0-android \
          --no-restore \
          -p:AndroidKeyStore=true \
          -p:AndroidSigningKeyStore=MyApp.keystore \
          -p:AndroidSigningKeyAlias=$KEY_ALIAS \
          -p:AndroidSigningKeyPass=$KEYSTORE_PASSWORD \
          -p:AndroidSigningStorePass=$KEYSTORE_PASSWORD
          
    - name: Upload Android Artifacts
      uses: actions/upload-artifact@v4
      with:
        name: android-artifacts
        path: |
          **/*.aab
          **/*.apk
          
    - name: Deploy to Google Play Store
      if: github.event_name == 'release'
      uses: r0adkll/upload-google-play@v1
      with:
        serviceAccountJsonPlainText: ${{ secrets.GOOGLE_PLAY_SERVICE_ACCOUNT }}
        packageName: com.yourcompany.yourapp
        releaseFiles: '**/*.aab'
        track: production
        status: completed

  build-ios:
    runs-on: macos-latest
    name: iOS Build and Deploy
    
    steps:
    - name: Checkout
      uses: actions/checkout@v4
      
    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '8.0.x'
        
    - name: Install MAUI Workload
      run: dotnet workload install maui
      
    - name: Setup Provisioning Profile
      if: github.event_name == 'release'
      env:
        PROVISIONING_PROFILE_BASE64: ${{ secrets.PROVISIONING_PROFILE_BASE64 }}
        CERTIFICATE_BASE64: ${{ secrets.CERTIFICATE_BASE64 }}
        CERTIFICATE_PASSWORD: ${{ secrets.CERTIFICATE_PASSWORD }}
      run: |
        # Install provisioning profile
        echo $PROVISIONING_PROFILE_BASE64 | base64 --decode > profile.mobileprovision
        mkdir -p ~/Library/MobileDevice/Provisioning\ Profiles
        cp profile.mobileprovision ~/Library/MobileDevice/Provisioning\ Profiles/
        
        # Install certificate
        echo $CERTIFICATE_BASE64 | base64 --decode > certificate.p12
        security create-keychain -p "" build.keychain
        security import certificate.p12 -k build.keychain -P $CERTIFICATE_PASSWORD -T /usr/bin/codesign
        security list-keychains -s build.keychain
        security default-keychain -s build.keychain
        security unlock-keychain -p "" build.keychain
        security set-keychain-settings -t 3600 -u build.keychain
        
    - name: Restore Dependencies
      run: dotnet restore YourApp.csproj
      
    - name: Build iOS
      run: dotnet build YourApp.csproj -c Release -f net8.0-ios --no-restore
      
    - name: Publish iOS
      if: github.event_name == 'release'
      run: |
        dotnet publish YourApp.csproj -c Release -f net8.0-ios --no-restore \
          -p:BuildIpa=true \
          -p:CodesignKey="iPhone Distribution" \
          -p:CodesignProvision="YourApp Distribution Profile"
          
    - name: Upload iOS Artifacts
      uses: actions/upload-artifact@v4
      with:
        name: ios-artifacts
        path: |
          **/*.ipa
          
    - name: Deploy to App Store
      if: github.event_name == 'release'
      env:
        APP_STORE_CONNECT_API_KEY: ${{ secrets.APP_STORE_CONNECT_API_KEY }}
        APP_STORE_CONNECT_ISSUER_ID: ${{ secrets.APP_STORE_CONNECT_ISSUER_ID }}
        APP_STORE_CONNECT_KEY_ID: ${{ secrets.APP_STORE_CONNECT_KEY_ID }}
      run: |
        xcrun altool --upload-app --type ios \
          --file "**/*.ipa" \
          --apiKey $APP_STORE_CONNECT_KEY_ID \
          --apiIssuer $APP_STORE_CONNECT_ISSUER_ID

  build-windows:
    runs-on: windows-latest
    name: Windows Build and Deploy
    
    steps:
    - name: Checkout
      uses: actions/checkout@v4
      
    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '8.0.x'
        
    - name: Install MAUI Workload
      run: dotnet workload install maui
      
    - name: Restore Dependencies
      run: dotnet restore YourApp.csproj
      
    - name: Build Windows
      run: dotnet build YourApp.csproj -c Release -f net8.0-windows10.0.19041.0 --no-restore
      
    - name: Create MSIX Package
      if: github.event_name == 'release'
      env:
        CERTIFICATE_PASSWORD: ${{ secrets.WINDOWS_CERTIFICATE_PASSWORD }}
      run: |
        echo "${{ secrets.WINDOWS_CERTIFICATE_BASE64 }}" | Out-File -FilePath cert.txt
        certutil -decode cert.txt YourApp.pfx
        dotnet publish YourApp.csproj -c Release -f net8.0-windows10.0.19041.0 --no-restore \
          -p:PackageCertificateKeyFile=YourApp.pfx \
          -p:PackageCertificatePassword=$env:CERTIFICATE_PASSWORD
          
    - name: Upload Windows Artifacts
      uses: actions/upload-artifact@v4
      with:
        name: windows-artifacts
        path: |
          **/*.msix
          **/*.appxbundle
          
    - name: Deploy to Microsoft Store
      if: github.event_name == 'release'
      uses: isaacrlevin/windows-store-action@1.0
      with:
        tenant-id: ${{ secrets.AZURE_TENANT_ID }}
        client-id: ${{ secrets.AZURE_CLIENT_ID }}
        client-secret: ${{ secrets.AZURE_CLIENT_SECRET }}
        app-id: ${{ secrets.WINDOWS_STORE_APP_ID }}
        package-path: '**/*.msixbundle'
```

### 4.2 Azure DevOps Pipeline

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
    - main
    - develop
  tags:
    include:
    - v*

pool:
  vmImage: 'macOS-latest'

variables:
  buildConfiguration: 'Release'
  dotnetVersion: '8.0.x'

stages:
- stage: Build
  displayName: 'Build Applications'
  jobs:
  - job: BuildAndroid
    displayName: 'Build Android'
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: UseDotNet@2
      displayName: 'Install .NET'
      inputs:
        version: $(dotnetVersion)
        
    - script: dotnet workload install maui
      displayName: 'Install MAUI Workload'
      
    - task: DotNetCoreCLI@2
      displayName: 'Restore packages'
      inputs:
        command: 'restore'
        projects: '**/*.csproj'
        
    - task: DotNetCoreCLI@2
      displayName: 'Build Android'
      inputs:
        command: 'build'
        projects: '**/*.csproj'
        arguments: '-c $(buildConfiguration) -f net8.0-android'
        
    - task: AndroidSigning@3
      condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
      displayName: 'Sign Android Package'
      inputs:
        apkFiles: '**/*.aab'
        apksignerKeystoreFile: 'MyApp.keystore'
        apksignerKeystorePassword: $(KeystorePassword)
        apksignerKeystoreAlias: $(KeyAlias)
        apksignerKeyPassword: $(KeyPassword)
        
    - task: PublishBuildArtifacts@1
      displayName: 'Publish Android Artifacts'
      inputs:
        pathToPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: 'android'

  - job: BuildiOS
    displayName: 'Build iOS'
    pool:
      vmImage: 'macOS-latest'
    steps:
    - task: UseDotNet@2
      displayName: 'Install .NET'
      inputs:
        version: $(dotnetVersion)
        
    - script: dotnet workload install maui
      displayName: 'Install MAUI Workload'
      
    - task: InstallAppleCertificate@2
      condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
      displayName: 'Install Apple Certificate'
      inputs:
        certSecureFile: 'YourApp_Distribution.p12'
        certPwd: $(CertificatePassword)
        keychain: 'temp'
        
    - task: InstallAppleProvisioningProfile@1
      condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
      displayName: 'Install Provisioning Profile'
      inputs:
        provisioningProfileLocation: 'secureFiles'
        provProfileSecureFile: 'YourApp_Distribution.mobileprovision'
        
    - task: DotNetCoreCLI@2
      displayName: 'Restore packages'
      inputs:
        command: 'restore'
        projects: '**/*.csproj'
        
    - task: DotNetCoreCLI@2
      displayName: 'Build iOS'
      inputs:
        command: 'publish'
        projects: '**/*.csproj'
        arguments: '-c $(buildConfiguration) -f net8.0-ios -p:BuildIpa=true'
        
    - task: PublishBuildArtifacts@1
      displayName: 'Publish iOS Artifacts'
      inputs:
        pathToPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: 'ios'

- stage: Deploy
  displayName: 'Deploy to Stores'
  dependsOn: Build
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
            displayName: 'Deploy to Google Play Store'
            inputs:
              serviceConnection: 'Google Play Store'
              applicationId: 'com.yourcompany.yourapp'
              action: 'SingleBundle'
              bundleFile: '$(Pipeline.Workspace)/android/**/*.aab'
              track: 'production'
              
  - deployment: DeployiOS
    displayName: 'Deploy to App Store'
    environment: 'Production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AppStoreRelease@1
            displayName: 'Deploy to Apple App Store'
            inputs:
              serviceEndpoint: 'App Store Connect'
              appIdentifier: 'com.yourcompany.yourapp'
              appType: 'iOS'
              ipaPath: '$(Pipeline.Workspace)/ios/**/*.ipa'
              shouldSkipWaitingForProcessing: true
              shouldSkipSubmission: false
```

---

## 5. 📊 Version Management & Release Notes

### 5.1 Automated Version Bumping

```xml
<!-- MSBuild targets for version management -->
<Project>
  <PropertyGroup>
    <!-- Semantic Versioning -->
    <MajorVersion>1</MajorVersion>
    <MinorVersion>0</MinorVersion>
    <PatchVersion>$([System.DateTime]::Now.ToString('MMdd'))</PatchVersion>
    <BuildNumber>$([System.DateTime]::Now.ToString('HHmm'))</BuildNumber>
    
    <!-- Application Version -->
    <ApplicationVersion>$(MajorVersion).$(MinorVersion).$(PatchVersion)</ApplicationVersion>
    <ApplicationDisplayVersion>$(ApplicationVersion)</ApplicationDisplayVersion>
    
    <!-- Android Specific -->
    <ApplicationVersionCode Condition="$(TargetFramework.Contains('-android'))">$([MSBuild]::Add($([MSBuild]::Multiply($(MajorVersion), 10000)), $([MSBuild]::Add($([MSBuild]::Multiply($(MinorVersion), 100)), $(PatchVersion)))))</ApplicationVersionCode>
    
    <!-- iOS Specific -->
    <CFBundleVersion Condition="$(TargetFramework.Contains('-ios'))">$(ApplicationVersionCode)</CFBundleVersion>
    <CFBundleShortVersionString Condition="$(TargetFramework.Contains('-ios'))">$(ApplicationVersion)</CFBundleShortVersionString>
  </PropertyGroup>
  
  <!-- Git-based versioning -->
  <Target Name="GetGitCommitInfo" BeforeTargets="BeforeBuild">
    <Exec Command="git rev-parse --short HEAD" ConsoleToMSBuild="true" IgnoreExitCode="true">
      <Output TaskParameter="ConsoleOutput" PropertyName="GitCommitHash" />
    </Exec>
    <Exec Command="git rev-list --count HEAD" ConsoleToMSBuild="true" IgnoreExitCode="true">
      <Output TaskParameter="ConsoleOutput" PropertyName="GitCommitCount" />
    </Exec>
    
    <PropertyGroup>
      <BuildNumber Condition="'$(GitCommitCount)' != ''">$(GitCommitCount)</BuildNumber>
      <InformationalVersion>$(ApplicationVersion)+$(GitCommitHash)</InformationalVersion>
    </PropertyGroup>
  </Target>
</Project>
```

### 5.2 Release Notes Generator

```csharp
// Tools/ReleaseNotesGenerator.cs
using System.Text.Json;
using System.Text.RegularExpressions;

public class ReleaseNotesGenerator
{
    public static async Task<string> GenerateReleaseNotes(string version, string gitLogOutput)
    {
        var notes = new StringBuilder();
        notes.AppendLine($"# Release Notes - Version {version}");
        notes.AppendLine($"Release Date: {DateTime.Now:yyyy-MM-dd}");
        notes.AppendLine();
        
        var commits = ParseGitLog(gitLogOutput);
        var categorizedCommits = CategorizeCommits(commits);
        
        foreach (var category in categorizedCommits)
        {
            if (category.Value.Any())
            {
                notes.AppendLine($"## {category.Key}");
                notes.AppendLine();
                
                foreach (var commit in category.Value)
                {
                    notes.AppendLine($"- {commit.Message} ({commit.Hash})");
                }
                notes.AppendLine();
            }
        }
        
        return notes.ToString();
    }
    
    private static List<GitCommit> ParseGitLog(string gitLog)
    {
        var commits = new List<GitCommit>();
        var lines = gitLog.Split('\n', StringSplitOptions.RemoveEmptyEntries);
        
        foreach (var line in lines)
        {
            var parts = line.Split('|');
            if (parts.Length >= 3)
            {
                commits.Add(new GitCommit
                {
                    Hash = parts[0].Trim(),
                    Message = parts[1].Trim(),
                    Author = parts[2].Trim()
                });
            }
        }
        
        return commits;
    }
    
    private static Dictionary<string, List<GitCommit>> CategorizeCommits(List<GitCommit> commits)
    {
        var categories = new Dictionary<string, List<GitCommit>>
        {
            ["🚀 New Features"] = new(),
            ["🐛 Bug Fixes"] = new(),
            ["⚡ Performance"] = new(),
            ["🎨 UI/UX"] = new(),
            ["📚 Documentation"] = new(),
            ["🔧 Other"] = new()
        };
        
        foreach (var commit in commits)
        {
            var message = commit.Message.ToLower();
            
            if (message.Contains("feat:") || message.Contains("feature:"))
                categories["🚀 New Features"].Add(commit);
            else if (message.Contains("fix:") || message.Contains("bug:"))
                categories["🐛 Bug Fixes"].Add(commit);
            else if (message.Contains("perf:") || message.Contains("performance:"))
                categories["⚡ Performance"].Add(commit);
            else if (message.Contains("ui:") || message.Contains("style:"))
                categories["🎨 UI/UX"].Add(commit);
            else if (message.Contains("docs:") || message.Contains("doc:"))
                categories["📚 Documentation"].Add(commit);
            else
                categories["🔧 Other"].Add(commit);
        }
        
        return categories;
    }
}

public class GitCommit
{
    public string Hash { get; set; } = string.Empty;
    public string Message { get; set; } = string.Empty;
    public string Author { get; set; } = string.Empty;
}
```

---

## 6. 🔒 Security & Code Signing

### 6.1 Certificate Management Script

```powershell
# Scripts/ManageCertificates.ps1
param(
    [Parameter(Mandatory=$true)]
    [ValidateSet("Android", "iOS", "Windows")]
    [string]$Platform,
    
    [Parameter(Mandatory=$true)]
    [ValidateSet("Generate", "Install", "Verify")]
    [string]$Action,
    
    [string]$CertPath,
    [string]$Password
)

function Generate-AndroidKeystore {
    param([string]$KeystorePath, [string]$KeystorePassword)
    
    Write-Host "Generating Android keystore..." -ForegroundColor Green
    
    $keytoolPath = "${env:JAVA_HOME}\bin\keytool.exe"
    if (-not (Test-Path $keytoolPath)) {
        throw "Keytool not found. Please ensure JAVA_HOME is set correctly."
    }
    
    & $keytoolPath -genkey -v -keystore $KeystorePath -alias myappkey -keyalg RSA -keysize 2048 -validity 10000 -storepass $KeystorePassword -keypass $KeystorePassword -dname "CN=Your Company, OU=IT Department, O=Your Company, L=Your City, S=Your State, C=US"
    
    Write-Host "Android keystore generated successfully at: $KeystorePath" -ForegroundColor Green
}

function Install-iOSCertificate {
    param([string]$CertPath, [string]$CertPassword)
    
    Write-Host "Installing iOS certificate..." -ForegroundColor Green
    
    # Create temporary keychain
    $keychainName = "maui-build.keychain"
    $keychainPassword = "temp-password"
    
    security create-keychain -p $keychainPassword $keychainName
    security set-keychain-settings -t 3600 -u $keychainName
    security unlock-keychain -p $keychainPassword $keychainName
    security import $CertPath -k $keychainName -P $CertPassword -T /usr/bin/codesign
    security set-key-partition-list -S apple-tool:,apple: -s -k $keychainPassword $keychainName
    
    Write-Host "iOS certificate installed successfully" -ForegroundColor Green
}

function Generate-WindowsCertificate {
    param([string]$CertPath, [string]$CertPassword)
    
    Write-Host "Generating Windows certificate..." -ForegroundColor Green
    
    $cert = New-SelfSignedCertificate -Type Custom -Subject "CN=Your Company" -KeyUsage DigitalSignature -FriendlyName "MAUI App Certificate" -CertStoreLocation "Cert:\CurrentUser\My" -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.3", "2.5.29.19={text}")
    
    $password = ConvertTo-SecureString -String $CertPassword -Force -AsPlainText
    Export-PfxCertificate -cert "Cert:\CurrentUser\My\$($cert.Thumbprint)" -FilePath $CertPath -Password $password
    
    Write-Host "Windows certificate generated successfully at: $CertPath" -ForegroundColor Green
    Write-Host "Certificate thumbprint: $($cert.Thumbprint)" -ForegroundColor Yellow
}

function Verify-Certificate {
    param([string]$Platform, [string]$CertPath)
    
    Write-Host "Verifying $Platform certificate..." -ForegroundColor Green
    
    switch ($Platform) {
        "Android" {
            $keytoolPath = "${env:JAVA_HOME}\bin\keytool.exe"
            & $keytoolPath -list -v -keystore $CertPath
        }
        "iOS" {
            security find-identity -v -p codesigning
        }
        "Windows" {
            Get-PfxCertificate -FilePath $CertPath | Format-List
        }
    }
}

# Main execution
try {
    switch ($Action) {
        "Generate" {
            switch ($Platform) {
                "Android" { 
                    if (-not $CertPath) { $CertPath = "MyApp.keystore" }
                    if (-not $Password) { $Password = Read-Host -Prompt "Enter keystore password" -AsSecureString | ConvertFrom-SecureString }
                    Generate-AndroidKeystore -KeystorePath $CertPath -KeystorePassword $Password 
                }
                "Windows" { 
                    if (-not $CertPath) { $CertPath = "MyApp.pfx" }
                    if (-not $Password) { $Password = Read-Host -Prompt "Enter certificate password" -AsSecureString | ConvertFrom-SecureString }
                    Generate-WindowsCertificate -CertPath $CertPath -CertPassword $Password 
                }
                "iOS" { 
                    Write-Host "iOS certificates must be generated through Apple Developer Portal" -ForegroundColor Yellow 
                }
            }
        }
        "Install" {
            if ($Platform -eq "iOS") {
                Install-iOSCertificate -CertPath $CertPath -CertPassword $Password
            } else {
                Write-Host "Install action is only supported for iOS platform" -ForegroundColor Yellow
            }
        }
        "Verify" {
            Verify-Certificate -Platform $Platform -CertPath $CertPath
        }
    }
} catch {
    Write-Error "Error occurred: $($_.Exception.Message)"
    exit 1
}
```

---

## 7. 📱 Store Optimization

### 7.1 App Store Metadata

```json
// app-store-metadata.json
{
  "common": {
    "appName": "Your App Name",
    "shortDescription": "Brief description of your app",
    "fullDescription": "Detailed description explaining all features and benefits of your app. This should be compelling and highlight key value propositions.",
    "keywords": ["productivity", "task management", "organization", "MAUI", "cross-platform"],
    "category": "Productivity",
    "contentRating": "4+",
    "privacyPolicyUrl": "https://yourapp.com/privacy",
    "supportUrl": "https://yourapp.com/support",
    "websiteUrl": "https://yourapp.com"
  },
  "googlePlay": {
    "title": "Your App - Task Manager",
    "shortDescription": "Organize your tasks efficiently with this cross-platform app",
    "fullDescription": "Transform your productivity with Your App, the ultimate task management solution built with cutting-edge .NET MAUI technology.\n\n🚀 KEY FEATURES:\n• Intuitive task creation and management\n• Cross-platform synchronization\n• Beautiful, responsive design\n• Offline-first architecture\n• Advanced filtering and sorting\n• Custom themes and personalization\n\n✨ WHAT MAKES IT SPECIAL:\n• Lightning-fast performance\n• Native feel on every platform\n• Regular updates and improvements\n• Privacy-focused design\n• No ads or tracking\n\n📱 PERFECT FOR:\n• Students managing assignments\n• Professionals organizing projects\n• Teams collaborating on tasks\n• Anyone wanting to be more productive\n\nDownload now and experience the future of task management!",
    "recentChanges": "• Improved performance and stability\n• New dark theme option\n• Enhanced synchronization\n• Bug fixes and optimizations",
    "categoryId": "PRODUCTIVITY",
    "contentRating": "EVERYONE",
    "targetAudience": ["ADULTS", "TEENAGERS"],
    "inAppProducts": false,
    "containsAds": false
  },
  "appStore": {
    "title": "Your App - Task Manager",
    "subtitle": "Organize your life efficiently",
    "description": "Transform your productivity with Your App, the ultimate task management solution built with cutting-edge technology.\n\nKEY FEATURES:\n• Intuitive task creation and management\n• Cross-platform synchronization\n• Beautiful, responsive design\n• Offline-first architecture\n• Advanced filtering and sorting\n• Custom themes and personalization\n\nWHAT MAKES IT SPECIAL:\n• Lightning-fast performance\n• Native feel on every platform\n• Regular updates and improvements\n• Privacy-focused design\n• No ads or tracking\n\nPERFECT FOR:\n• Students managing assignments\n• Professionals organizing projects\n• Teams collaborating on tasks\n• Anyone wanting to be more productive\n\nDownload now and experience the future of task management!",
    "keywords": "productivity,task,management,organization,planner,todo,gtd,efficiency",
    "primaryCategory": "PRODUCTIVITY",
    "secondaryCategory": "BUSINESS",
    "contentRating": "4+",
    "inAppPurchases": false,
    "gameCenterEnabled": false
  },
  "microsoftStore": {
    "displayName": "Your App - Task Manager",
    "description": "Transform your productivity with Your App, the ultimate task management solution.\n\nKEY FEATURES:\n• Intuitive task creation and management\n• Cross-platform synchronization\n• Beautiful, responsive design\n• Offline-first architecture\n• Advanced filtering and sorting\n• Custom themes and personalization\n\nDownload now and experience the future of task management!",
    "shortTitle": "Your App",
    "category": "Productivity",
    "subcategory": "Personal finance",
    "hardwareRequirements": {
      "minimumSystemRam": "4 GB",
      "minimumSystemProcessor": "x64, ARM64",
      "minimumSystemDirectXVersion": "11",
      "recommendedSystemRam": "8 GB"
    },
    "features": [
      "Cross-platform synchronization",
      "Offline support",
      "Touch optimized",
      "Keyboard shortcuts",
      "High DPI support"
    ]
  }
}
```

### 7.2 Store Assets Generator

```csharp
// Tools/StoreAssetsGenerator.cs
using SkiaSharp;

public class StoreAssetsGenerator
{
    public static void GenerateAllAssets(string sourceIconPath, string outputDirectory)
    {
        var androidSizes = new Dictionary<string, SKSizeI>
        {
            ["mdpi"] = new(48, 48),
            ["hdpi"] = new(72, 72),
            ["xhdpi"] = new(96, 96),
            ["xxhdpi"] = new(144, 144),
            ["xxxhdpi"] = new(192, 192),
            ["playstore"] = new(512, 512)
        };
        
        var iosSizes = new Dictionary<string, SKSizeI>
        {
            ["29x29"] = new(29, 29),
            ["29x29@2x"] = new(58, 58),
            ["29x29@3x"] = new(87, 87),
            ["40x40@2x"] = new(80, 80),
            ["40x40@3x"] = new(120, 120),
            ["60x60@2x"] = new(120, 120),
            ["60x60@3x"] = new(180, 180),
            ["1024x1024"] = new(1024, 1024)
        };
        
        var windowsSizes = new Dictionary<string, SKSizeI>
        {
            ["Square44x44Logo"] = new(44, 44),
            ["Square71x71Logo"] = new(71, 71),
            ["Square150x150Logo"] = new(150, 150),
            ["Square310x310Logo"] = new(310, 310),
            ["Wide310x150Logo"] = new(310, 150),
            ["StoreLogo"] = new(50, 50)
        };
        
        // Generate Android icons
        var androidDir = Path.Combine(outputDirectory, "Android");
        Directory.CreateDirectory(androidDir);
        foreach (var size in androidSizes)
        {
            var outputPath = Path.Combine(androidDir, $"ic_launcher_{size.Key}.png");
            ResizeImage(sourceIconPath, outputPath, size.Value);
        }
        
        // Generate iOS icons
        var iosDir = Path.Combine(outputDirectory, "iOS");
        Directory.CreateDirectory(iosDir);
        foreach (var size in iosSizes)
        {
            var outputPath = Path.Combine(iosDir, $"AppIcon_{size.Key}.png");
            ResizeImage(sourceIconPath, outputPath, size.Value);
        }
        
        // Generate Windows icons
        var windowsDir = Path.Combine(outputDirectory, "Windows");
        Directory.CreateDirectory(windowsDir);
        foreach (var size in windowsSizes)
        {
            var outputPath = Path.Combine(windowsDir, $"{size.Key}.png");
            ResizeImage(sourceIconPath, outputPath, size.Value);
        }
        
        // Generate store screenshots templates
        GenerateScreenshotTemplates(outputDirectory);
    }
    
    private static void ResizeImage(string inputPath, string outputPath, SKSizeI targetSize)
    {
        using var inputStream = File.OpenRead(inputPath);
        using var inputBitmap = SKBitmap.Decode(inputStream);
        using var resizedBitmap = inputBitmap.Resize(targetSize, SKFilterQuality.High);
        using var outputStream = File.OpenWrite(outputPath);
        resizedBitmap.Encode(outputStream, SKEncodedImageFormat.Png, 100);
    }
    
    private static void GenerateScreenshotTemplates(string outputDirectory)
    {
        var screenshotSizes = new Dictionary<string, SKSizeI>
        {
            ["iPhone6.5"] = new(1242, 2688),
            ["iPhone5.5"] = new(1242, 2208),
            ["iPad12.9"] = new(2048, 2732),
            ["Android_Phone"] = new(1080, 1920),
            ["Android_Tablet"] = new(1600, 2560),
            ["Windows_Desktop"] = new(1920, 1080)
        };
        
        var templatesDir = Path.Combine(outputDirectory, "Screenshots");
        Directory.CreateDirectory(templatesDir);
        
        foreach (var size in screenshotSizes)
        {
            var templatePath = Path.Combine(templatesDir, $"Template_{size.Key}.png");
            CreateScreenshotTemplate(templatePath, size.Value);
        }
    }
    
    private static void CreateScreenshotTemplate(string outputPath, SKSizeI size)
    {
        using var surface = SKSurface.Create(new SKImageInfo(size.Width, size.Height));
        var canvas = surface.Canvas;
        
        // Background gradient
        using var paint = new SKPaint
        {
            Shader = SKShader.CreateLinearGradient(
                new SKPoint(0, 0),
                new SKPoint(0, size.Height),
                new[] { SKColors.Blue, SKColors.Purple },
                SKShaderTileMode.Clamp)
        };
        canvas.DrawRect(0, 0, size.Width, size.Height, paint);
        
        // Add text
        using var textPaint = new SKPaint
        {
            Color = SKColors.White,
            TextSize = size.Height * 0.05f,
            IsAntialias = true,
            Typeface = SKTypeface.FromFamilyName("Arial", SKFontStyle.Bold)
        };
        
        var text = "Your App Screenshot";
        var textWidth = textPaint.MeasureText(text);
        var x = (size.Width - textWidth) / 2;
        var y = size.Height / 2;
        
        canvas.DrawText(text, x, y, textPaint);
        
        // Save
        using var image = surface.Snapshot();
        using var data = image.Encode(SKEncodedImageFormat.Png, 100);
        using var stream = File.OpenWrite(outputPath);
        data.SaveTo(stream);
    }
}
```

---

## 8. 🧪 Practical Exercise

### Exercise: Complete Deployment Pipeline

สร้าง complete deployment pipeline สำหรับ MAUI application ของคุณ:

```xml
<!-- DeploymentExercise.targets -->
<Project>
  <Target Name="PrepareForDeployment" BeforeTargets="Build">
    <Message Text="Preparing for deployment..." Importance="high" />
    
    <!-- Validate required properties -->
    <Error Condition="'$(Configuration)' != 'Release'" Text="Deployment must use Release configuration" />
    <Error Condition="'$(ApplicationVersion)' == ''" Text="ApplicationVersion must be specified" />
    
    <!-- Platform-specific validation -->
    <Error Condition="$(TargetFramework.Contains('-android')) AND '$(AndroidSigningKeyStore)' == ''" 
           Text="Android signing keystore must be specified for deployment" />
    <Error Condition="$(TargetFramework.Contains('-ios')) AND '$(CodesignKey)' == ''" 
           Text="iOS code signing key must be specified for deployment" />
    <Error Condition="$(TargetFramework.Contains('-windows')) AND '$(PackageCertificateKeyFile)' == ''" 
           Text="Windows certificate must be specified for deployment" />
  </Target>
  
  <Target Name="GenerateReleaseNotes" AfterTargets="Build">
    <Exec Command="git log --oneline --since=&quot;1 week ago&quot; --pretty=format:&quot;%h|%s|%an&quot;" 
          ConsoleToMSBuild="true">
      <Output TaskParameter="ConsoleOutput" PropertyName="GitLog" />
    </Exec>
    
    <WriteLinesToFile File="$(OutputPath)ReleaseNotes.txt" 
                      Lines="Release Notes for $(ApplicationVersion);$(GitLog)" 
                      Overwrite="true" />
  </Target>
  
  <Target Name="CreateDeploymentPackage" AfterTargets="Publish">
    <ItemGroup>
      <DeploymentFiles Include="$(PublishDir)**\*.*" />
      <DeploymentFiles Include="$(OutputPath)ReleaseNotes.txt" />
    </ItemGroup>
    
    <MakeDir Directories="$(OutputPath)Deployment" />
    <Copy SourceFiles="@(DeploymentFiles)" 
          DestinationFolder="$(OutputPath)Deployment\%(RecursiveDir)" />
    
    <Message Text="Deployment package created at: $(OutputPath)Deployment" Importance="high" />
  </Target>
</Project>
```

### Configuration Checklist

**Pre-Deployment Checklist:**
- [ ] Application version updated
- [ ] Release notes generated
- [ ] Certificates and signing keys configured
- [ ] App store metadata updated
- [ ] Screenshots and assets prepared
- [ ] Privacy policy and support URLs verified
- [ ] Testing completed on all target platforms
- [ ] Performance benchmarks met
- [ ] Accessibility requirements verified
- [ ] Localization completed

**Store-Specific Checklist:**

**Google Play Store:**
- [ ] App Bundle (AAB) format configured
- [ ] Target API level meets requirements
- [ ] Permissions justified and minimal
- [ ] Google Play Console app created
- [ ] Release track configured (internal/alpha/beta/production)

**Apple App Store:**
- [ ] App Store Connect app created
- [ ] iOS Distribution certificate installed
- [ ] Provisioning profile configured
- [ ] App Store guidelines compliance verified
- [ ] TestFlight beta testing completed

**Microsoft Store:**
- [ ] Partner Center app created
- [ ] MSIX package format configured
- [ ] Windows App Certification Kit passed
- [ ] Store listing completed
- [ ] Age rating obtained

---

## 9. 📚 Summary

ในบทนี้เราได้เรียนรู้:

### 🔑 Key Concepts
- **Multi-platform Deployment** strategies และ best practices
- **Store Submission** processes สำหรับ Google Play, App Store, และ Microsoft Store
- **CI/CD Pipeline** configuration สำหรับ automated deployment
- **Code Signing** และ certificate management
- **Version Management** และ release note generation
- **Store Optimization** และ metadata management

### 🛠️ Technical Skills
- การ configure build settings สำหรับแต่ละ platform
- การตั้งค่า automated pipelines ด้วย GitHub Actions และ Azure DevOps
- การจัดการ certificates และ signing keys อย่างปลอดภัย
- การ optimize apps สำหรับ store submission
- การสร้าง deployment packages และ release artifacts

### 📱 Platform-Specific Knowledge
- **Android:** App Bundle generation, ProGuard optimization, Google Play Console
- **iOS:** IPA creation, code signing, App Store Connect, TestFlight
- **Windows:** MSIX packaging, certificate management, Microsoft Store

### 🔄 DevOps Integration
- Automated version bumping และ semantic versioning
- Git-based release note generation
- Multi-stage deployment pipelines
- Artifact management และ distribution
- Security best practices สำหรับ CI/CD

### 📊 Store Success Factors
- Compelling app store descriptions และ metadata
- Professional screenshot และ asset generation
- Privacy policy และ support documentation
- User feedback management และ app store optimization
- Release cadence และ update strategies

การ deploy MAUI applications เป็น complex process ที่ต้องการความเข้าใจในหลายด้าน จาก technical configuration ไปจนถึง store policies และ user experience considerations การมี automated pipeline ที่ดีจะช่วยให้การ deploy เป็นไปอย่างราบรื่นและลดข้อผิดพลาด

**Next Chapter Preview:** ในบท Advanced Phase ต่อไป เราจะเจาะลึกไปที่ Advanced Architecture Patterns และ Enterprise Development กับ .NET MAUI! 🏆
