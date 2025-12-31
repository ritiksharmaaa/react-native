# React Native Core Concepts - Learning Guide

This repository is dedicated to learning React Native core concepts while building applications. The focus is on **native features, permissions, build processes, and over-the-air updates** - not on UI or routing (which you already know from React).

## Table of Contents

1. [Introduction](#introduction)
2. [Expo vs Bare React Native](#expo-vs-bare-react-native)
3. [Permissions System](#permissions-system)
4. [Build Process](#build-process)
5. [Over-the-Air (OTA) Updates](#over-the-air-ota-updates)
6. [Storage and Data Persistence](#storage-and-data-persistence)
7. [Native Modules and Bridges](#native-modules-and-bridges)
8. [Performance Optimization](#performance-optimization)
9. [Debugging and Development Tools](#debugging-and-development-tools)

---

## Introduction

React Native allows you to build native mobile applications using JavaScript and React. Unlike web apps, mobile apps require special handling for:
- **Native permissions** (camera, location, notifications, etc.)
- **Native compilation** (APK for Android, IPA for iOS)
- **App updates** without going through app stores
- **Native device features** (sensors, storage, biometrics, etc.)

This guide focuses on these core React Native concepts.

---

## Expo vs Bare React Native

Before diving into the details, it's important to understand the two main approaches to React Native development:

### Bare React Native (React Native CLI)

**What it is:**
- The "standard" React Native setup using `npx react-native init`
- Full access to native Android and iOS code
- Requires native build tools (Android Studio/Xcode)
- More control and flexibility

**Pros:**
- ✅ Complete control over native code
- ✅ Can use any native library
- ✅ No limitations on native features
- ✅ Better for learning the full stack

**Cons:**
- ❌ Requires Android Studio (~4-6GB) or Android SDK
- ❌ Requires Xcode (~15GB on macOS)
- ❌ More complex setup
- ❌ Steeper learning curve

### Expo (Managed Workflow)

**What it is:**
- A framework built on top of React Native
- Pre-configured build pipeline (EAS Build)
- Can build apps in the cloud without local native tools
- Simplified development experience

**Pros:**
- ✅ No need for Android Studio or Xcode
- ✅ Cloud builds with EAS Build (works on 8GB RAM machines)
- ✅ Faster to get started
- ✅ Built-in libraries for common features
- ✅ Perfect for learning without heavy tooling

**Cons:**
- ❌ Some limitations on native modules (though ejecting is possible)
- ❌ Slightly larger app size
- ❌ Less control over native build process

### Which Should You Choose?

**For learning React Native concepts (like you):**
- Start with **Expo** if you have limited RAM (8GB)
- Use **EAS Build** for cloud building (no Android Studio needed)
- Learn the concepts first, then understand how they work in bare React Native
- This guide covers **both approaches** so you understand the full flow

**This guide's approach:**
- Shows **standard/bare React Native** methods first (for learning the full flow)
- Includes **Expo alternatives** and **lightweight methods** for low-RAM machines
- Explains **cloud build options** that don't require bulky SDKs
- Look for 💡 **Low-RAM Alternative** sections throughout

---

## Permissions System

Mobile apps need explicit user permission to access sensitive features. React Native provides ways to request and handle permissions.

### Bare React Native: Using `react-native-permissions` Library

The most comprehensive library for handling permissions in bare React Native is `react-native-permissions`.

#### Installation

```bash
npm install react-native-permissions
# or
yarn add react-native-permissions
```

#### iOS Setup

Add permissions to `ios/YourApp/Info.plist`:

```xml
<key>NSCameraUsageDescription</key>
<string>We need camera access to take photos</string>

<key>NSPhotoLibraryUsageDescription</key>
<string>We need photo library access to select photos</string>

<key>NSMicrophoneUsageDescription</key>
<string>We need microphone access to record audio</string>

<key>NSLocationWhenInUseUsageDescription</key>
<string>We need location access to show nearby places</string>

<key>NSLocationAlwaysUsageDescription</key>
<string>We need location access to track your activity</string>

<key>NSContactsUsageDescription</key>
<string>We need contacts access to find friends</string>

<key>NSCalendarsUsageDescription</key>
<string>We need calendar access to schedule events</string>

<key>NSRemindersUsageDescription</key>
<string>We need reminders access</string>

<key>NSMotionUsageDescription</key>
<string>We need motion sensor access to track activity</string>

<key>NSBluetoothAlwaysUsageDescription</key>
<string>We need Bluetooth access to connect devices</string>

<key>NSFaceIDUsageDescription</key>
<string>We need Face ID access for authentication</string>
```

Then run:
```bash
cd ios && pod install && cd ..
```

#### Android Setup

Add permissions to `android/app/src/main/AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.READ_CONTACTS" />
<uses-permission android:name="android.permission.WRITE_CONTACTS" />
<uses-permission android:name="android.permission.READ_CALENDAR" />
<uses-permission android:name="android.permission.WRITE_CALENDAR" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.CALL_PHONE" />
<uses-permission android:name="android.permission.READ_CALL_LOG" />
<uses-permission android:name="android.permission.WRITE_CALL_LOG" />
<uses-permission android:name="android.permission.SEND_SMS" />
<uses-permission android:name="android.permission.READ_SMS" />
<uses-permission android:name="android.permission.RECEIVE_SMS" />
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
<uses-permission android:name="android.permission.ACCESS_NOTIFICATION_POLICY" />
<!-- POST_NOTIFICATIONS: Only required for Android 13+ (API 33+) -->
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
```

#### Code Example: Requesting Permissions

```javascript
import { check, request, PERMISSIONS, RESULTS } from 'react-native-permissions';
import { Platform } from 'react-native';

// Camera Permission
const requestCameraPermission = async () => {
  const permission = Platform.select({
    ios: PERMISSIONS.IOS.CAMERA,
    android: PERMISSIONS.ANDROID.CAMERA,
  });

  const result = await check(permission);
  
  switch (result) {
    case RESULTS.UNAVAILABLE:
      console.log('This feature is not available on this device');
      break;
    case RESULTS.DENIED:
      console.log('Permission not requested / is denied but requestable');
      const requestResult = await request(permission);
      return requestResult === RESULTS.GRANTED;
    case RESULTS.LIMITED:
      console.log('Permission is limited: some actions are possible');
      return true;
    case RESULTS.GRANTED:
      console.log('Permission is granted');
      return true;
    case RESULTS.BLOCKED:
      console.log('Permission is denied and not requestable anymore');
      return false;
  }
};

// Location Permission
const requestLocationPermission = async () => {
  const permission = Platform.select({
    ios: PERMISSIONS.IOS.LOCATION_WHEN_IN_USE,
    android: PERMISSIONS.ANDROID.ACCESS_FINE_LOCATION,
  });

  const result = await request(permission);
  return result === RESULTS.GRANTED;
};

// Notification Permission
const requestNotificationPermission = async () => {
  if (Platform.OS === 'ios') {
    const result = await request(PERMISSIONS.IOS.NOTIFICATIONS);
    return result === RESULTS.GRANTED;
  } else {
    // Android 13+ (API 33+) requires POST_NOTIFICATIONS permission
    // For older versions, notifications work without explicit permission
    const result = await request(PERMISSIONS.ANDROID.POST_NOTIFICATIONS);
    return result === RESULTS.GRANTED;
  }
};

// Microphone Permission
const requestMicrophonePermission = async () => {
  const permission = Platform.select({
    ios: PERMISSIONS.IOS.MICROPHONE,
    android: PERMISSIONS.ANDROID.RECORD_AUDIO,
  });

  const result = await request(permission);
  return result === RESULTS.GRANTED;
};

// Contacts Permission
const requestContactsPermission = async () => {
  const permission = Platform.select({
    ios: PERMISSIONS.IOS.CONTACTS,
    android: PERMISSIONS.ANDROID.READ_CONTACTS,
  });

  const result = await request(permission);
  return result === RESULTS.GRANTED;
};

// Phone State Permission (Android only)
const requestPhoneStatePermission = async () => {
  if (Platform.OS === 'android') {
    const result = await request(PERMISSIONS.ANDROID.READ_PHONE_STATE);
    return result === RESULTS.GRANTED;
  }
  return false;
};

// Storage Permission
const requestStoragePermission = async () => {
  if (Platform.OS === 'android') {
    const result = await request(PERMISSIONS.ANDROID.READ_EXTERNAL_STORAGE);
    return result === RESULTS.GRANTED;
  }
  return true; // iOS handles this differently
};
```

#### Multiple Permissions at Once

```javascript
import { requestMultiple, PERMISSIONS } from 'react-native-permissions';

const requestMultiplePermissions = async () => {
  const permissions = Platform.select({
    ios: [
      PERMISSIONS.IOS.CAMERA,
      PERMISSIONS.IOS.PHOTO_LIBRARY,
      PERMISSIONS.IOS.MICROPHONE,
    ],
    android: [
      PERMISSIONS.ANDROID.CAMERA,
      PERMISSIONS.ANDROID.READ_EXTERNAL_STORAGE,
      PERMISSIONS.ANDROID.RECORD_AUDIO,
    ],
  });

  const results = await requestMultiple(permissions);
  console.log(results);
};
```

---

### 💡 Expo Alternative: Built-in Permission APIs

If you're using **Expo**, many permissions are built-in with simpler APIs:

#### Installation

```bash
# Camera
npx expo install expo-camera

# Location
npx expo install expo-location

# Media Library
npx expo install expo-media-library

# Contacts
npx expo install expo-contacts

# Notifications
npx expo install expo-notifications
```

#### Code Examples

**Camera Permission (Expo):**

```javascript
import { Camera } from 'expo-camera';

const requestCameraPermission = async () => {
  const { status } = await Camera.requestCameraPermissionsAsync();
  return status === 'granted';
};

// Check without requesting
const { status } = await Camera.getCameraPermissionsAsync();
```

**Location Permission (Expo):**

```javascript
import * as Location from 'expo-location';

const requestLocationPermission = async () => {
  const { status } = await Location.requestForegroundPermissionsAsync();
  return status === 'granted';
};

// Get location
const location = await Location.getCurrentPositionAsync({});
```

**Notification Permission (Expo):**

```javascript
import * as Notifications from 'expo-notifications';

const requestNotificationPermission = async () => {
  const { status } = await Notifications.requestPermissionsAsync();
  return status === 'granted';
};
```

**Media Library Permission (Expo):**

```javascript
import * as MediaLibrary from 'expo-media-library';

const requestMediaLibraryPermission = async () => {
  const { status } = await MediaLibrary.requestPermissionsAsync();
  return status === 'granted';
};
```

**Contacts Permission (Expo):**

```javascript
import * as Contacts from 'expo-contacts';

const requestContactsPermission = async () => {
  const { status } = await Contacts.requestPermissionsAsync();
  return status === 'granted';
};
```

**Configuration in `app.json` (Expo):**

```json
{
  "expo": {
    "name": "Your App",
    "plugins": [
      [
        "expo-camera",
        {
          "cameraPermission": "Allow $(PRODUCT_NAME) to access your camera"
        }
      ],
      [
        "expo-location",
        {
          "locationAlwaysAndWhenInUsePermission": "Allow $(PRODUCT_NAME) to use your location."
        }
      ],
      [
        "expo-notifications",
        {
          "icon": "./assets/notification-icon.png",
          "color": "#ffffff"
        }
      ]
    ],
    "ios": {
      "infoPlist": {
        "NSCameraUsageDescription": "This app uses the camera to take photos.",
        "NSPhotoLibraryUsageDescription": "This app accesses your photos.",
        "NSLocationWhenInUseUsageDescription": "This app uses your location."
      }
    },
    "android": {
      "permissions": [
        "CAMERA",
        "ACCESS_FINE_LOCATION",
        "READ_EXTERNAL_STORAGE",
        "WRITE_EXTERNAL_STORAGE"
      ]
    }
  }
}
```

**Benefits of Expo Permissions:**
- ✅ Simpler API (one-line permission requests)
- ✅ Automatic platform handling
- ✅ Config-based permission setup
- ✅ No need to edit native files manually
- ✅ Works with EAS Build (no Android Studio needed)

**When to Use Each:**
- **Expo APIs**: If using Expo and want simplicity
- **react-native-permissions**: If using bare React Native or need more control

---

### Sensor Access

React Native provides access to various device sensors:

#### Accelerometer & Gyroscope (Bare React Native)

```bash
npm install react-native-sensors
```

```javascript
import { accelerometer, gyroscope } from 'react-native-sensors';

// Accelerometer
const subscription = accelerometer.subscribe(({ x, y, z }) => {
  console.log('Accelerometer:', x, y, z);
});

// Gyroscope
const gyroSubscription = gyroscope.subscribe(({ x, y, z }) => {
  console.log('Gyroscope:', x, y, z);
});

// Don't forget to unsubscribe
subscription.unsubscribe();
gyroSubscription.unsubscribe();
```

#### 💡 Expo Alternative: Device Motion & Sensors

```bash
npx expo install expo-sensors
```

```javascript
import { Accelerometer, Gyroscope, Magnetometer } from 'expo-sensors';

// Accelerometer
Accelerometer.setUpdateInterval(100);
const subscription = Accelerometer.addListener(({ x, y, z }) => {
  console.log('Accelerometer:', x, y, z);
});

// Gyroscope
const gyroSubscription = Gyroscope.addListener(({ x, y, z }) => {
  console.log('Gyroscope:', x, y, z);
});

// Clean up
subscription.remove();
gyroSubscription.remove();
```

---

## Build Process

Understanding how your JavaScript code becomes a native app is crucial.

### Approach Overview

This guide covers **three approaches** to building React Native apps:

1. **Standard Approach** (with Android Studio/Xcode) - Best for learning the full flow
2. **Lightweight Approach** (Command-line tools only) - For low-RAM machines (8GB)
3. **Cloud Build Approach** (EAS Build for Expo) - No local SDKs needed

Choose based on your machine specs and learning goals.

---

### Android - Building APK/AAB

#### Standard Approach (with Android Studio)

**What You Need:**
- **JDK** (Java Development Kit) 11 or higher
- **Android Studio** (~4-6GB download, needs 8GB+ RAM)
- **Gradle** (comes with React Native)
- **Signing keystore** for release builds

#### Debug Build (for testing)

```bash
# Navigate to android folder
cd android

# Build debug APK
./gradlew assembleDebug

# Output: android/app/build/outputs/apk/debug/app-debug.apk
```

#### Release Build (for production)

1. **Generate Signing Key**

```bash
cd android/app
keytool -genkeypair -v -storetype PKCS12 -keystore my-release-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000
```

2. **Configure Gradle** - Edit `android/gradle.properties`:

```properties
MYAPP_RELEASE_STORE_FILE=my-release-key.keystore
MYAPP_RELEASE_KEY_ALIAS=my-key-alias
MYAPP_RELEASE_STORE_PASSWORD=*****
MYAPP_RELEASE_KEY_PASSWORD=*****
```

3. **Update `android/app/build.gradle`**:

```gradle
android {
    ...
    signingConfigs {
        release {
            if (project.hasProperty('MYAPP_RELEASE_STORE_FILE')) {
                storeFile file(MYAPP_RELEASE_STORE_FILE)
                storePassword MYAPP_RELEASE_STORE_PASSWORD
                keyAlias MYAPP_RELEASE_KEY_ALIAS
                keyPassword MYAPP_RELEASE_KEY_PASSWORD
            }
        }
    }
    buildTypes {
        release {
            ...
            signingConfig signingConfigs.release
        }
    }
}
```

4. **Build Release APK/AAB**

```bash
cd android

# Build APK
./gradlew assembleRelease
# Output: android/app/build/outputs/apk/release/app-release.apk

# Build AAB (for Play Store)
./gradlew bundleRelease
# Output: android/app/build/outputs/bundle/release/app-release.aab
```

---

#### 💡 Low-RAM Alternative: Command-Line Tools Only (Without Android Studio)

If you have **8GB RAM or less** and can't install Android Studio, you can use Android SDK command-line tools:

**Step 1: Install Android SDK Command-Line Tools**

```bash
# Download SDK command-line tools
# Visit: https://developer.android.com/studio#command-tools
# Download "Command line tools only" (~100MB vs 4-6GB for Android Studio)

# Extract to a location, e.g., ~/Android/cmdline-tools

# Set environment variables in ~/.bashrc or ~/.zshrc
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

**Step 2: Install Required SDK Packages**

```bash
# Accept licenses
sdkmanager --licenses

# Install minimal required packages (much smaller than Android Studio)
sdkmanager "platform-tools" "platforms;android-33" "build-tools;33.0.0"

# Install system image for emulator (optional, only if you want to test)
sdkmanager "system-images;android-33;google_apis;x86_64"
```

**Step 3: Build APK**

```bash
# Same commands as standard approach
cd android
./gradlew assembleDebug
# or
./gradlew assembleRelease
```

**Benefits:**
- ✅ ~500MB download vs ~6GB for Android Studio
- ✅ Uses less RAM during builds
- ✅ Same build output as Android Studio
- ✅ Learn the actual build process without IDE

**Limitations:**
- ❌ No visual IDE (use VS Code instead)
- ❌ No emulator GUI (can use physical device)
- ❌ More command-line work

---

#### 💡 Cloud Build Alternative: EAS Build (Expo)

For **Expo projects**, you can build in the cloud without any Android SDK:

**Step 1: Install EAS CLI**

```bash
npm install -g eas-cli
```

**Step 2: Configure EAS**

```bash
# Login to Expo account (free tier available)
eas login

# Configure project
eas build:configure
```

**Step 3: Build in Cloud**

```bash
# Build APK for Android
eas build --platform android --profile preview

# Build AAB for Play Store
eas build --platform android --profile production

# Builds happen on Expo's servers
# Downloads APK/AAB when complete (~10-15 minutes)
```

**Benefits:**
- ✅ No Android SDK installation needed
- ✅ Works on any machine (even 4GB RAM)
- ✅ Builds on Expo's cloud servers
- ✅ Free tier available (limited builds/month)
- ✅ Perfect for learning without heavy tools

**Costs:**
- Free tier: ~30 builds/month
- Paid plans: Unlimited builds (~$29/month for hobbyists)

**Use Case:**
- Perfect for 8GB RAM machines
- Great for learning and testing
- Can upgrade to bare workflow later

---

#### Requirements Summary

**Standard Approach:**
- **JDK** (Java Development Kit) 11 or higher
- **Android Studio** (~6GB) or Android SDK (~500MB command-line tools)
- **Gradle** (comes with React Native)
- **Signing keystore** for release builds
- **8GB+ RAM recommended** for Android Studio
- **Google Play Developer Account** ($25 one-time fee) for publishing

**Low-RAM Alternative (Command-Line Tools):**
- **JDK** 11 or higher
- **Android SDK command-line tools** (~500MB)
- **4-8GB RAM** sufficient
- **Signing keystore** for release builds

**Cloud Build Alternative (EAS/Expo):**
- **Node.js** and **EAS CLI**
- **Expo account** (free tier available)
- **Any machine** (even 4GB RAM works)
- **Internet connection** for cloud builds

---

### iOS - Building IPA

#### Standard Approach (with Xcode)

- **macOS** (iOS builds only work on Mac)
- **Xcode** (free from Mac App Store)
- **CocoaPods** (`sudo gem install cocoapods`)
- **Apple Developer Account** ($99/year for publishing)

#### Debug Build

```bash
cd ios
pod install
cd ..

# Run on simulator
npx react-native run-ios

# Or open in Xcode
open ios/YourApp.xcworkspace
```

#### Release Build

1. **Open Xcode**

```bash
open ios/YourApp.xcworkspace
```

2. **Configure Signing**
   - Select your project in Xcode
   - Go to "Signing & Capabilities"
   - Select your Apple Developer Team
   - Enable "Automatically manage signing"

3. **Archive the App**
   - Product → Scheme → Edit Scheme
   - Set Run scheme to "Release"
   - Product → Archive
   - Wait for archive to complete

4. **Export IPA**
   - Window → Organizer
   - Select your archive
   - Click "Distribute App"
   - Choose distribution method (App Store, Ad Hoc, Enterprise)
   - Follow the wizard to export IPA

---

#### 💡 Cloud Build Alternative: EAS Build (Expo)

**For users WITHOUT macOS** or with limited RAM:

```bash
# Install EAS CLI
npm install -g eas-cli

# Login
eas login

# Configure project
eas build:configure

# Build iOS in cloud (no Mac needed!)
eas build --platform ios --profile preview

# Submit to App Store (optional)
eas submit --platform ios
```

**Benefits:**
- ✅ Build iOS apps **without a Mac**
- ✅ No Xcode installation needed (~15GB)
- ✅ Builds on Expo's macOS servers
- ✅ Works on Windows/Linux/Low-spec machines

**Requirements:**
- Still need **Apple Developer Account** ($99/year) for distribution
- EAS free tier: ~30 builds/month
- Build time: ~15-20 minutes per build

---

#### Requirements Summary

**Standard Approach (with Xcode):**
- **macOS** computer (required)
- **Xcode** (~15GB)
- **CocoaPods** for dependencies
- **8GB+ RAM recommended**
- **Apple Developer Account** ($99/year)
- **Provisioning profiles** and **certificates**
- **App Store Connect** account for publishing

**Cloud Build Alternative (EAS/Expo):**
- **Any OS** (Windows/Linux/macOS)
- **No Xcode** needed
- **EAS CLI** and **Expo account**
- **Apple Developer Account** ($99/year) still required
- **Any machine** (even 4GB RAM works)
- **Internet connection** for cloud builds

---

### Build Comparison

| Approach | Platform | RAM Required | SDK Size | Machine | Build Time | Best For |
|----------|----------|--------------|----------|---------|------------|----------|
| **Standard (Android Studio)** | Android | 8GB+ | ~6GB | Any OS | 2-5 min | Full learning |
| **CLI Tools Only** | Android | 4-8GB | ~500MB | Any OS | 2-5 min | Low RAM |
| **EAS Build** | Android | Any | 0 (cloud) | Any OS | 10-15 min | Easiest |
| **Standard (Xcode)** | iOS | 8GB+ | ~15GB | macOS only | 5-10 min | Full learning |
| **EAS Build** | iOS | Any | 0 (cloud) | Any OS | 15-20 min | No Mac needed |

**Quick Commands:**

```bash
# Bare React Native (Standard)
npx react-native run-android          # Android debug
cd android && ./gradlew assembleRelease  # Android release
npx react-native run-ios              # iOS debug (macOS only)

# Bare React Native (CLI Tools Only - same as standard)
cd android && ./gradlew assembleDebug    # Uses command-line SDK

# Expo (Cloud Build)
eas build --platform android          # Android cloud build
eas build --platform ios              # iOS cloud build (no Mac!)

# Expo (Local Development)
npx expo start                        # Start dev server
npx expo run:android                  # Run on Android
npx expo run:ios                      # Run on iOS (macOS only)
```

---

## Over-the-Air (OTA) Updates

OTA updates allow you to push JavaScript changes to users without going through app stores.

### Popular OTA Solutions

#### 1. **CodePush** (by Microsoft)

```bash
npm install --save react-native-code-push
```

**Setup:**

```javascript
// App.js
import codePush from 'react-native-code-push';

const App = () => {
  // Your app code
};

// Wrap your app with codePush
const codePushOptions = {
  checkFrequency: codePush.CheckFrequency.ON_APP_RESUME,
  installMode: codePush.InstallMode.IMMEDIATE,
};

export default codePush(codePushOptions)(App);
```

**CLI Commands:**

```bash
# Install CodePush CLI
npm install -g appcenter-cli

# Login
appcenter login

# Register your app
appcenter apps create -d MyApp-iOS -o iOS -p React-Native
appcenter apps create -d MyApp-Android -o Android -p React-Native

# Get deployment keys
appcenter codepush deployment list -a <ownerName>/<appName>

# Release update
appcenter codepush release-react -a <ownerName>/<appName> -d Production
```

**Configuration:**

iOS - `ios/YourApp/Info.plist`:
```xml
<key>CodePushDeploymentKey</key>
<string>YOUR_IOS_DEPLOYMENT_KEY</string>
```

Android - `android/app/build.gradle`:
```gradle
apply from: "../../node_modules/react-native-code-push/android/codepush.gradle"
```

Android - `android/app/src/main/res/values/strings.xml`:
```xml
<string name="CodePushDeploymentKey">YOUR_ANDROID_DEPLOYMENT_KEY</string>
```

#### 2. **Expo Updates** (if using Expo)

```bash
npx expo install expo-updates
```

```javascript
import * as Updates from 'expo-updates';

async function checkForUpdates() {
  try {
    const update = await Updates.checkForUpdateAsync();
    if (update.isAvailable) {
      await Updates.fetchUpdateAsync();
      await Updates.reloadAsync();
    }
  } catch (error) {
    console.log('Error checking for updates:', error);
  }
}
```

#### 3. **Custom OTA Solution**

You can build your own:

```javascript
// 1. Check for updates from your server
const checkForUpdate = async () => {
  const response = await fetch('https://your-api.com/app-version');
  const { latestVersion, bundleUrl } = await response.json();
  
  const currentVersion = '1.0.0'; // From app config
  
  if (latestVersion > currentVersion) {
    return bundleUrl;
  }
  return null;
};

// 2. Download and apply bundle
const downloadAndApplyUpdate = async (bundleUrl) => {
  const bundlePath = `${RNFS.DocumentDirectoryPath}/main.jsbundle`;
  await RNFS.downloadFile({
    fromUrl: bundleUrl,
    toFile: bundlePath,
  }).promise;
  
  // Reload with new bundle
  // Note: This requires native code to load bundle from custom location
};
```

### OTA Limitations

**What CAN be updated:**
- ✅ JavaScript code
- ✅ Images and assets
- ✅ Business logic changes
- ✅ UI changes
- ✅ Bug fixes in JS

**What CANNOT be updated:**
- ❌ Native code changes (Java/Kotlin/Objective-C/Swift)
- ❌ New native dependencies
- ❌ Permission changes in manifest files
- ❌ Native module linking
- ❌ App icon or splash screen (native assets)

For these, you must submit a new version to app stores.

---

## Storage and Data Persistence

React Native provides multiple ways to store data locally.

### 1. AsyncStorage (Key-Value Storage)

```bash
npm install @react-native-async-storage/async-storage
```

```javascript
import AsyncStorage from '@react-native-async-storage/async-storage';

// Store data
const storeData = async (key, value) => {
  try {
    await AsyncStorage.setItem(key, JSON.stringify(value));
  } catch (error) {
    console.error('Error storing data:', error);
  }
};

// Retrieve data
const getData = async (key) => {
  try {
    const value = await AsyncStorage.getItem(key);
    return value != null ? JSON.parse(value) : null;
  } catch (error) {
    console.error('Error retrieving data:', error);
  }
};

// Remove data
const removeData = async (key) => {
  try {
    await AsyncStorage.removeItem(key);
  } catch (error) {
    console.error('Error removing data:', error);
  }
};

// Clear all data
const clearAll = async () => {
  try {
    await AsyncStorage.clear();
  } catch (error) {
    console.error('Error clearing data:', error);
  }
};
```

### 2. SQLite Database

```bash
npm install react-native-sqlite-storage
```

```javascript
import SQLite from 'react-native-sqlite-storage';

const db = SQLite.openDatabase(
  { name: 'mydb.db', location: 'default' },
  () => console.log('Database opened'),
  error => console.log('Error: ', error)
);

// Create table
db.transaction(tx => {
  tx.executeSql(
    'CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT, email TEXT)',
    [],
    () => console.log('Table created'),
    error => console.log('Error creating table:', error)
  );
});

// Insert data
const insertUser = (name, email) => {
  db.transaction(tx => {
    tx.executeSql(
      'INSERT INTO users (name, email) VALUES (?, ?)',
      [name, email],
      (tx, results) => console.log('User inserted'),
      error => console.log('Error inserting user:', error)
    );
  });
};

// Query data
const getUsers = () => {
  db.transaction(tx => {
    tx.executeSql(
      'SELECT * FROM users',
      [],
      (tx, results) => {
        const users = [];
        for (let i = 0; i < results.rows.length; i++) {
          users.push(results.rows.item(i));
        }
        console.log('Users:', users);
      },
      error => console.log('Error querying users:', error)
    );
  });
};
```

### 3. Realm Database

```bash
npm install realm
```

```javascript
import Realm from 'realm';

// Define schema
const UserSchema = {
  name: 'User',
  properties: {
    _id: 'objectId',
    name: 'string',
    email: 'string',
    age: 'int?',
  },
  primaryKey: '_id',
};

// Open realm
const realm = await Realm.open({
  schema: [UserSchema],
  schemaVersion: 1,
});

// Write data
realm.write(() => {
  realm.create('User', {
    _id: new Realm.BSON.ObjectId(),
    name: 'John Doe',
    email: 'john@example.com',
    age: 30,
  });
});

// Query data
const users = realm.objects('User');
console.log('All users:', users);

// Filter data
const filteredUsers = realm.objects('User').filtered('age >= 25');

// Update data
realm.write(() => {
  const user = realm.objects('User').filtered('name = "John Doe"')[0];
  user.age = 31;
});

// Delete data
realm.write(() => {
  const user = realm.objects('User').filtered('name = "John Doe"')[0];
  realm.delete(user);
});

// Close realm
realm.close();
```

### 4. MMKV (Fast Key-Value Storage)

```bash
npm install react-native-mmkv
```

```javascript
import { MMKV } from 'react-native-mmkv';

const storage = new MMKV();

// Store
storage.set('user.name', 'John Doe');
storage.set('user.age', 30);
storage.set('is_logged_in', true);

// Retrieve
const username = storage.getString('user.name');
const age = storage.getNumber('user.age');
const isLoggedIn = storage.getBoolean('is_logged_in');

// Delete
storage.delete('user.name');

// Clear all
storage.clearAll();
```

---

## Native Modules and Bridges

Sometimes you need to access native functionality not available in JavaScript.

### Creating a Native Module (Android)

**1. Create Java/Kotlin Module**

`android/app/src/main/java/com/yourapp/MyNativeModule.java`:

```java
package com.yourapp;

import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
import com.facebook.react.bridge.Promise;

public class MyNativeModule extends ReactContextBaseJavaModule {
    MyNativeModule(ReactApplicationContext context) {
        super(context);
    }

    @Override
    public String getName() {
        return "MyNativeModule";
    }

    @ReactMethod
    public void getDeviceName(Promise promise) {
        try {
            String deviceName = android.os.Build.MODEL;
            promise.resolve(deviceName);
        } catch (Exception e) {
            promise.reject("ERROR", e);
        }
    }
}
```

**2. Register Module**

`android/app/src/main/java/com/yourapp/MyNativePackage.java`:

```java
package com.yourapp;

import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.uimanager.ViewManager;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class MyNativePackage implements ReactPackage {
    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
        List<NativeModule> modules = new ArrayList<>();
        modules.add(new MyNativeModule(reactContext));
        return modules;
    }

    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
        return Collections.emptyList();
    }
}
```

**3. Add to MainApplication**

`android/app/src/main/java/com/yourapp/MainApplication.java`:

```java
@Override
protected List<ReactPackage> getPackages() {
    return Arrays.<ReactPackage>asList(
        new MainReactPackage(),
        new MyNativePackage() // Add this
    );
}
```

**4. Use in JavaScript**

```javascript
import { NativeModules } from 'react-native';

const { MyNativeModule } = NativeModules;

const deviceName = await MyNativeModule.getDeviceName();
console.log('Device name:', deviceName);
```

### Creating a Native Module (iOS)

**1. Create Objective-C/Swift Module**

`ios/YourApp/MyNativeModule.m`:

```objective-c
#import <React/RCTBridgeModule.h>

@interface RCT_EXTERN_MODULE(MyNativeModule, NSObject)

RCT_EXTERN_METHOD(getDeviceName:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)

@end
```

`ios/YourApp/MyNativeModule.swift`:

```swift
import Foundation

@objc(MyNativeModule)
class MyNativeModule: NSObject {
  
  @objc
  func getDeviceName(_ resolve: RCTPromiseResolveBlock, rejecter reject: RCTPromiseRejectBlock) -> Void {
    let deviceName = UIDevice.current.name
    resolve(deviceName)
  }
  
  @objc
  static func requiresMainQueueSetup() -> Bool {
    return true
  }
}
```

**2. Use in JavaScript** (same as Android)

```javascript
import { NativeModules } from 'react-native';

const { MyNativeModule } = NativeModules;

const deviceName = await MyNativeModule.getDeviceName();
console.log('Device name:', deviceName);
```

### Turbo Modules (New Architecture)

React Native's new architecture uses Turbo Modules for better performance:

```javascript
// Create spec file
// NativeMyModule.ts
import { TurboModule, TurboModuleRegistry } from 'react-native';

export interface Spec extends TurboModule {
  getDeviceName(): Promise<string>;
}

export default TurboModuleRegistry.getEnforcing<Spec>('MyNativeModule');
```

---

## Performance Optimization

### 1. **Hermes JavaScript Engine**

Hermes is a JavaScript engine optimized for React Native.

**Enable Hermes:**

Android - `android/app/build.gradle`:
```gradle
project.ext.react = [
    enableHermes: true
]
```

iOS - `ios/Podfile`:
```ruby
use_react_native!(
  :hermes_enabled => true
)
```

Then run:
```bash
cd ios && pod install && cd ..
```

### 2. **Image Optimization**

```javascript
import FastImage from 'react-native-fast-image';

<FastImage
  style={{ width: 200, height: 200 }}
  source={{
    uri: 'https://example.com/image.jpg',
    priority: FastImage.priority.high,
    cache: FastImage.cacheControl.immutable,
  }}
  resizeMode={FastImage.resizeMode.contain}
/>
```

### 3. **List Performance**

```javascript
import { FlatList } from 'react-native';

<FlatList
  data={items}
  renderItem={({ item }) => <ItemComponent item={item} />}
  keyExtractor={item => item.id}
  // Performance props
  initialNumToRender={10}
  maxToRenderPerBatch={10}
  windowSize={5}
  removeClippedSubviews={true}
  getItemLayout={(data, index) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index,
  })}
/>
```

### 4. **RAM Bundles**

Reduce startup time by loading only required code:

`android/app/build.gradle`:
```gradle
project.ext.react = [
    bundleCommand: "ram-bundle",
]
```

### 5. **ProGuard/R8 (Android)**

Minify and obfuscate code:

`android/app/build.gradle`:
```gradle
buildTypes {
    release {
        minifyEnabled true
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
}
```

---

## Debugging and Development Tools

### 1. **React Native Debugger**

Download standalone app: https://github.com/jhen0409/react-native-debugger

- Redux DevTools
- React DevTools
- Network Inspector

### 2. **Flipper**

Official debugging tool from Facebook.

```bash
# Install Flipper desktop app
# https://fbflipper.com/

# Already integrated in React Native 0.62+
```

Features:
- Layout Inspector
- Network Inspector
- Database Viewer
- Shared Preferences Viewer
- Crash Reporter

### 3. **Reactotron**

```bash
npm install --save-dev reactotron-react-native
```

```javascript
import Reactotron from 'reactotron-react-native';

Reactotron
  .configure()
  .useReactNative()
  .connect();

// Use in code
Reactotron.log('Hello world!');
```

### 4. **Performance Monitor**

Access the Performance Monitor in development mode:
- Shake the device (or press Cmd+D on iOS simulator / Cmd+M on Android emulator)
- Select "Show Perf Monitor" from the dev menu
- Shows FPS, memory usage, and JS/UI thread performance

### 5. **Logcat & Console**

```bash
# Android logs
adb logcat

# iOS logs
xcrun simctl spawn booted log stream --predicate 'processImagePath endswith "YourApp"'

# React Native logs
npx react-native log-android
npx react-native log-ios
```

---

## Learning Path Recommendations

1. **Start with Permissions** - Build a simple app that requests camera, location, and storage permissions
2. **Explore Storage** - Implement AsyncStorage for user preferences, then try SQLite for structured data
3. **Build Process** - Create debug and release builds for both platforms
4. **Native Modules** - Write a simple native module that accesses device info
5. **OTA Updates** - Implement CodePush to push updates without app store submission
6. **Performance** - Profile your app, enable Hermes, optimize images and lists
7. **Production** - Deploy to TestFlight (iOS) and Google Play Internal Testing (Android)

---

## Useful Resources

- **Official Docs:** https://reactnative.dev/
- **CodePush:** https://github.com/microsoft/react-native-code-push
- **Permissions:** https://github.com/zoontek/react-native-permissions
- **Flipper:** https://fbflipper.com/
- **Hermes:** https://hermesengine.dev/
- **Native Modules:** https://reactnative.dev/docs/native-modules-intro

---

## Common Commands Cheat Sheet

### Bare React Native

```bash
# Create new React Native project
npx react-native init MyApp

# Run on Android
npx react-native run-android

# Run on iOS
npx react-native run-ios

# Build Android APK
cd android && ./gradlew assembleRelease

# Build Android AAB
cd android && ./gradlew bundleRelease

# Clean Android build
cd android && ./gradlew clean

# Clean iOS build
cd ios && xcodebuild clean

# Install iOS dependencies
cd ios && pod install

# Start Metro bundler
npx react-native start

# Clear cache
npx react-native start --reset-cache

# Check React Native info
npx react-native info
```

### Expo Commands

```bash
# Create new Expo project
npx create-expo-app MyApp

# Start development server
npx expo start

# Run on Android (requires Android device/emulator)
npx expo run:android

# Run on iOS (requires macOS and iOS simulator)
npx expo run:ios

# Build for Android (cloud build - no SDK needed)
eas build --platform android

# Build for iOS (cloud build - no Mac needed)
eas build --platform ios

# Submit to stores
eas submit --platform android
eas submit --platform ios

# Check Expo info
npx expo-doctor
```

---

## Practical Guide for 8GB RAM / Low-Spec Machines

**Your Situation:** You're using Expo, have 8GB RAM, can't install Android Studio, but want to learn the entire flow (not just Expo-specific).

### Recommended Learning Path

#### Phase 1: Start with Expo (Easiest)

```bash
# 1. Create Expo project
npx create-expo-app LearnRN
cd LearnRN

# 2. Start development
npx expo start

# 3. Test on physical device (scan QR code with Expo Go app)
# Download "Expo Go" from Play Store/App Store

# 4. Practice permissions
npx expo install expo-camera expo-location expo-notifications
# Implement permission requests as shown in "Expo Alternative" sections

# 5. Build APK using cloud (no Android Studio needed!)
npm install -g eas-cli
eas login
eas build:configure
eas build --platform android --profile preview
```

**Why this works:**
- ✅ No Android Studio needed (uses EAS cloud builds)
- ✅ 8GB RAM is plenty for Expo development
- ✅ Fast iteration with Expo Go app
- ✅ Learn core concepts quickly

#### Phase 2: Understand Bare React Native (Learning)

```bash
# 1. Create bare React Native project (just to explore)
npx react-native init LearnRNBare
cd LearnRNBare

# 2. Install Android SDK command-line tools (lightweight)
# Download from: https://developer.android.com/studio#command-tools
# Extract to ~/Android/cmdline-tools (~100MB vs 6GB)

# 3. Set environment variables
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin

# 4. Install minimal SDK
sdkmanager --licenses
sdkmanager "platform-tools" "platforms;android-33" "build-tools;33.0.0"

# 5. Build APK (works without Android Studio)
cd android && ./gradlew assembleDebug

# 6. Compare with Expo approach
# - See how android/ and ios/ folders differ
# - Understand native configuration files
# - Learn about AndroidManifest.xml and Info.plist
```

**Why this works:**
- ✅ Command-line tools only ~500MB (vs 6GB for Android Studio)
- ✅ Same build output as Android Studio
- ✅ Understand the "standard" way
- ✅ 8GB RAM sufficient for command-line builds

#### Phase 3: Explore Both Workflows

**Use Expo for:**
- Daily development (faster iteration)
- Testing features quickly
- Building in the cloud
- Deploying to real users

**Use Bare React Native for:**
- Learning native configuration
- Understanding AndroidManifest.xml / Info.plist
- Practicing Gradle builds
- Exploring native modules

### Your Ideal Setup (8GB RAM)

```
Development Machine (8GB RAM):
├── VS Code (lightweight IDE)
├── Node.js & npm
├── Expo CLI (for daily work)
├── EAS CLI (for cloud builds)
├── Android SDK command-line tools (~500MB, optional for learning)
├── Git
└── Physical Android device (for testing)

NO NEED FOR:
❌ Android Studio (~6GB)
❌ Android Emulator (RAM-heavy)
❌ Xcode (macOS only, ~15GB)
```

### Build Workflow for 8GB RAM

**Option 1: EAS Build (Recommended)**
```bash
# No Android Studio, builds in cloud
eas build --platform android
# Wait 10-15 minutes, download APK
# Install on physical device
```

**Option 2: Command-Line Tools (For Learning)**
```bash
# Uses command-line SDK (~500MB)
cd android && ./gradlew assembleDebug
# Takes 2-5 minutes locally
# More RAM-efficient than Android Studio
```

### Learning Both Approaches

**Expo App Structure:**
```
my-expo-app/
├── app.json          # Expo configuration
├── App.js            # Main app file
├── package.json
└── assets/
```

**Bare React Native Structure:**
```
my-bare-app/
├── android/          # Native Android code
│   └── app/
│       └── src/main/
│           └── AndroidManifest.xml
├── ios/              # Native iOS code
│   └── YourApp/
│       └── Info.plist
├── App.js
└── package.json
```

**Key Learning:**
- Expo hides `android/` and `ios/` folders
- Bare React Native exposes them for full control
- EAS Build can build Expo apps without local SDKs
- Both use same JavaScript/React code
- Native features work in both (just different APIs)

### Next Steps for You

1. **Week 1**: Use Expo, practice permissions, storage
2. **Week 2**: Build APK with EAS (cloud build)
3. **Week 3**: Create bare React Native project, explore native folders
4. **Week 4**: Install command-line SDK, build locally (lightweight)
5. **Week 5+**: Compare both approaches, understand trade-offs

**Resources Specific to Your Setup:**
- Expo Docs: https://docs.expo.dev/
- EAS Build: https://docs.expo.dev/build/introduction/
- Android SDK CLI: https://developer.android.com/studio/command-line
- Command-line tools guide: https://developer.android.com/studio/intro/update#sdk-manager

---

## Next Steps

This guide covers the core concepts. To practice:

1. Clone this repository
2. Create a new React Native app: `npx react-native init LearnRN`
3. Implement each concept one by one
4. Build example apps for each feature
5. Deploy to real devices and test

Happy learning! 🚀
