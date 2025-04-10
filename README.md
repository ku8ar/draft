## Project Goal

> ⚠️ **Disclaimer**
> 
> This project serves as a **proof of concept** and may not be stable enough for production environments. 
> Compatibility has only been verified with **React Native versions 0.75 to 0.78**.


This document outlines a clean and modular approach to integrating React Native (RN) code into a native iOS application. Instead of embedding RN directly within the iOS project, we build it as a standalone `XCFramework`. This decouples both codebases, simplifies dependency management, and speeds up native app compilation, all while preserving the flexibility and autonomy of the RN project.

## Architecture Overview

```ascii
[ React Native Code ]
        ↓
[ Target: Bridge ]
  ├── React Native Runtime
  ├── Turbo Native Modules
  ├── Hermes Engine
  └── Resources
        ↓
[ XCFramework ]
        ↓
[ iOS Host App ]
```

## Overview

### ✅ Advantages
- The React Native app remains fully self-contained, independent of the host iOS implementation and its dependencies.
- Completely avoids dependency conflicts between RN and the iOS app.
- Significantly reduces native app build times.
- Encourages proper separation of responsibilities across teams and repositories.

### ⚠️ Drawbacks

| Challenge | Description |
|----------|-------------|
| Library integration | Some RN libraries may require additional handling in the `build-bridge.sh` script.<br><br>🔹 *Example:* Tracking libraries that compile native modules often require manual framework integration. |
| Compatibility issues | Not all RN libraries work flawlessly in a precompiled setup.<br><br>🔹 *Example:* `react-native-reanimated` needs patching, although the workaround is straightforward. |
| Bundle size | Adding multiple RN-based frameworks to the same iOS host can dramatically increase the final app size.<br><br>🔹 *Solution:* Extracting the Hermes binary (a major contributor to `.ipa` size) can help, but then all RN modules must share the same RN version. |

---

## Xcode Project Setup

1. Create a new **Framework** target and name it `Bridge`.  
   *(This name is referenced in both `Bridge.podspec` and `build-bridge.sh`, so update those if you pick a different name.)*

2. Navigate to `Build Phases` → select the `Bridge` target → remove the phase **[CP] Copy Pods Resources**.  
   *(Frameworks cannot modify `Target Support Files` due to SIP restrictions.)*

3. Add the following files to the `Bridge` target:
   - `Bridge.h`
   - `BridgeWrapperViewController.h`
   - `BridgeWrapperViewController.m`
   - `RNBridgeViewController.swift`

   For each header file, set `Header Visibility` to **Public**.

---

## Source Code Configuration

### CocoaPods Setup in the React Native Project

To properly link React Native dependencies into the `Bridge` framework, you need to modify the `Podfile` located in your React Native project's `ios/` directory. Add a dedicated target section like the one below:

```ruby
target 'Bridge' do
  use_react_native!(
    :path => config[:reactNativePath],
    :app_path => "#{Pod::Config.instance.installation_root}/.."
  )
end
```

1. Copy the template implementation into `BridgeWrapperViewController.m`, and update `moduleName` to match your RN app.  
   *(You’ll find it in the RN project’s `AppDelegate.swift`.)*

2. Add these template files to your project:
   - `ios/Bridge/BridgeWrapperViewController.h`
   - `ios/Bridge/Bridge.h`
   - `ios/Bridge/RNBridgeViewController.swift`
   - `ios/build-bridge.sh`  
     *(Adjust the `WORKSPACE` variable to match your RN project name, found in `app.json`.)*

3. Add the `Bridge.podspec` template to `ios/`.  
   *(Be sure to personalize `version`, `homepage`, `author`, etc.)*

---

## Building the Framework

```bash
# From the project root:
npm install
cd ios
pod install
chmod +x build-bridge.sh
./build-bridge.sh
```

---

## What `build-bridge.sh` Does

- Archives the RN framework separately for device and simulator targets
- Combines both into a universal `XCFramework`

---

## Host App Integration

1. In your host app's `Podfile`, add:

```ruby
pod 'Bridge', :path => '../rn-project/ios' # Adjust the path
```

2. Install the pod:

```bash
pod install
```

3. In `AppDelegate.swift`, declare the following property:

```swift
@objc var window: UIWindow?
```

4. Import the `Bridge` module wherever you need it:

```swift
import Bridge
```

5. Use the RN-powered view controller:

```swift
let vc = RNBridgeViewController()
...
```

---

## Troubleshooting

### Crash: `unrecognized selector sent to instance caused by Fabric` 

If you encounter a crash like:

```
unrecognized selector sent to instance
```

when running the host app on iOS, and you're using React Native 0.70+ or Re.Pack, it's likely that **Fabric (the New Architecture)** was unintentionally enabled.

This can happen even if you're not explicitly using Fabric, as some RN versions and libraries enable it by default under certain conditions.

In particular, if you're using `react-native-screens`, it may attempt to invoke Fabric-related functionality during initialization — even when your app isn't Fabric-enabled.

#### ✅ Solution: Disable Fabric explicitly

Open your React Native project's `ios/Podfile` and add the following line **before** `require_relative '../node_modules/react-native/scripts/react_native_pods'`:

```ruby
ENV['RCT_NEW_ARCH_ENABLED'] = '0'
```

Then reinstall your CocoaPods:

```bash
cd ios
pod install
```

This ensures the project is built using the **legacy architecture**, which avoids linking issues with TurboModules and Fabric-enabled symbols that may not exist in the host app.

> ℹ️ Fabric is officially supported in precompiled RN frameworks starting from **React Native 0.79**. If you're targeting earlier versions, it is recommended to keep Fabric disabled.

---

## TODO
- Add support for passing input arguments to the RN module
- Implement output callbacks or communication back to the host app

---

## What's Next
- Code signing and notarization of the framework
- CI/CD integration to automate builds and releases

---

## Glossary

- **Bridge** — The framework target that packages the React Native app into an `XCFramework`.
- **Host** — The native iOS application that integrates one or more RN-powered modules.

