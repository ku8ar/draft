
# 🧩 Integrating `@callstack/repack` in a Swift Framework (`Bridge.framework`)

The implementation of [`@callstack/repack`](https://github.com/callstack/repack) inside a Swift framework is nearly identical to how it’s used in a standard React Native app.

Repack still handles Metro configuration, dynamic JS bundle loading, and dev server support — just like in any standalone React Native app.

However, using Repack **inside a Swift framework built with `BUILD_LIBRARY_FOR_DISTRIBUTION = YES`** requires a few extra steps.

---

## ✅ What stays the same?

- ✅ `metro.config.js`, `repack.config.js`
- ✅ Custom Metro scripts (e.g., `build-bridge.js`)
- ✅ Your React Native app entry (`App.tsx`, `index.ts`)
- ✅ Split bundles & dev server support

---

## ⚠️ What needs to change for framework compatibility?

### 1. ✨ Patch `@callstack/repack`

The Swift code inside `callstack-repack` (`CodeSigningUtils.swift`) originally used:

- `SwiftyRSA` and `JWTDecode` — neither of which are compatible with Swift module interfaces
- `@objc` + public APIs that violate module stability

🔧 Solution:

- Replace `SwiftyRSA` and `JWTDecode` with native `Security.framework` and manual JWT decoding
- Avoid exposing any `public` API in Swift
- Ensure all types are `internal` or `@objc` without breaking evolution rules
- Use `SecKeyVerifySignature` for signature validation

✅ After this patch, the Swift portion of Repack becomes compatible with framework distribution.

---

### 2. 🔧 Update your `Podfile`

To prevent Xcode from generating and verifying `.swiftinterface` files (which will fail for Repack), disable it per-target:

```ruby
post_install do |installer|
  installer.pods_project.targets.each do |target|
    if target.name == 'callstack-repack'
      target.build_configurations.each do |config|
        config.build_settings['BUILD_LIBRARY_FOR_DISTRIBUTION'] = 'NO'
        config.build_settings['OTHER_SWIFT_FLAGS'] ||= ''
        config.build_settings['OTHER_SWIFT_FLAGS'] += ' -Xfrontend -no-verify-emitted-module-interface'
      end
    end
  end
end
