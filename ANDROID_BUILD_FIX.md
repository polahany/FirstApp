# Android Build Configuration Fixes

## Problem
The Android app failed to build with C++ compilation errors related to `std::format` and other C++20 standard library features.

## Root Cause
This is a sandbox/test app that was likely generated with newer tooling versions that are incompatible with React Native 0.82.1. The default configuration used:
- NDK 23.1.7779620 (insufficient C++20 support)
- SDK 36 with Build Tools 36.0.0 (beta/preview versions)
- Gradle 9.0.0 with Android Gradle Plugin 8.12.0 (too new)

React Native 0.82.1 requires specific NDK and toolchain versions for proper C++20 support.

## Required Changes

### 1. Update NDK Version
**File:** `android/build.gradle`

**Change:**
```gradle
ndkVersion = "27.1.12297006"  // Previously: "23.1.7779620"
```

**Reason:** NDK 27 includes complete C++20 standard library implementations required by React Native 0.82.1's native code. Earlier NDK versions (23, 26) lack proper support for `std::format` and other C++20 features used in React Native's graphics conversion utilities.

### 2. Fix Gradle Wrapper Permissions
**File:** `android/gradlew`

**Change:**
```bash
chmod +x android/gradlew
```

**Reason:** The gradlew file wasn't executable, causing "EACCES" errors when trying to run the build.

### 3. SDK and Build Tools Versions (Optional but Recommended)
**File:** `android/build.gradle`

**Current (Working) Configuration:**
```gradle
buildToolsVersion = "36.0.0"
compileSdkVersion = 36
targetSdkVersion = 36
kotlinVersion = "2.1.20"
```

**Note:** While SDK 36 and Build Tools 36.0.0 are beta/preview versions, they work fine with NDK 27 and Gradle 9.0.0. The sandbox app generated with these versions by default.

## Summary
The **critical fix** is updating the NDK version to 27.1.12297006. This resolves all C++ compilation errors related to missing C++20 standard library features. The other configurations (SDK versions, Gradle versions) that came with the sandbox app setup are compatible once the NDK is corrected.

## Verification
After making the NDK change:
```bash
cd android && ./gradlew clean && cd ..
yarn android
```

The app should build successfully and install on connected devices/emulators.
