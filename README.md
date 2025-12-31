# React Native Core Concepts - Learning Guide

This repository is dedicated to learning React Native core concepts while building applications. The focus is on **native features, permissions, build processes, and over-the-air updates** - not on UI or routing (which you already know from React).

## Table of Contents

1. [Introduction](#introduction)
2. [Permissions System](#permissions-system)
3. [Build Process](#build-process)
4. [Over-the-Air (OTA) Updates](#over-the-air-ota-updates)
5. [Storage and Data Persistence](#storage-and-data-persistence)
6. [Native Modules and Bridges](#native-modules-and-bridges)
7. [Performance Optimization](#performance-optimization)
8. [Debugging and Development Tools](#debugging-and-development-tools)

---

## Introduction

React Native allows you to build native mobile applications using JavaScript and React. Unlike web apps, mobile apps require special handling for:
- **Native permissions** (camera, location, notifications, etc.)
- **Native compilation** (APK for Android, IPA for iOS)
- **App updates** without going through app stores
- **Native device features** (sensors, storage, biometrics, etc.)

This guide focuses on these core React Native concepts.

---

## Permissions System

Mobile apps need explicit user permission to access sensitive features. React Native provides ways to request and handle permissions.

### Using `react-native-permissions` Library

The most comprehensive library for handling permissions is `react-native-permissions`.

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

### Sensor Access

React Native provides access to various device sensors:

#### Accelerometer & Gyroscope

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

---

## Build Process

Understanding how your JavaScript code becomes a native app is crucial.

### Android - Building APK/AAB

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

#### What You Need for Android:

- **JDK** (Java Development Kit) 11 or higher
- **Android Studio** or Android SDK
- **Gradle** (comes with React Native)
- **Signing keystore** for release builds
- **Google Play Developer Account** ($25 one-time fee) for publishing

### iOS - Building IPA

#### Prerequisites

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

#### What You Need for iOS:

- **macOS** computer
- **Xcode** (free)
- **CocoaPods** for dependencies
- **Apple Developer Account** ($99/year)
- **Provisioning profiles** and **certificates**
- **App Store Connect** account for publishing

### Build Comparison

| Platform | Debug Command | Release Output | Required For Publishing |
|----------|--------------|----------------|------------------------|
| Android | `cd android && ./gradlew assembleDebug` | APK/AAB | Google Play Console |
| iOS | `npx react-native run-ios` | IPA | App Store Connect |

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

---

## Next Steps

This guide covers the core concepts. To practice:

1. Clone this repository
2. Create a new React Native app: `npx react-native init LearnRN`
3. Implement each concept one by one
4. Build example apps for each feature
5. Deploy to real devices and test

Happy learning! 🚀
