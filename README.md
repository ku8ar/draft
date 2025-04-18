```
+----------------------------------------------------+
|    Bridge (React Native Framework)                 |
|    - Independent module loaded as a framework      |
|    - Performs requests via fetch(), axios, etc.    |
|    - Does not implement its own SSL pinning        |
+--------------------------+-------------------------+
                           |
                           v
+----------------------------------------------------+
|    CustomURLProtocol (registered by host app)      |
|    - iOS feature that intercepts all requests      |
|      made through NSURLSession                     |
|    - Implemented solely in the iOS host application|
+--------------------------+-------------------------+
                           |
                           v
+----------------------------------------------------+
|    SSL Pinning / Security Layer                    |
|    - Host app verifies the server certificate      |
|    - Possible strategies:                          |
|        • SHA256 public key hash comparison         |
|        • DER certificate binary match             |
|    - If validation fails → request is killed     |
+--------------------------+-------------------------+
```

## 🧐 Architecture Overview

### 🔒 Separation of Concerns

- The **React Native Bridge** (inside the framework) is responsible only for rendering UI and handling JS-native communication.
- The **iOS host application** takes full ownership of the network security layer, including **SSL pinning**, request interception, and certificate validation.

This separation ensures that the React Native code does not need to (and cannot) interfere with the security logic.

### 🛡️ Why This Architecture Is Secure

- All requests from the framework are transparently intercepted by `CustomURLProtocol` using native iOS networking (`NSURLSession`).
- SSL certificate validation and pinning are enforced exclusively by native Swift/Obj-C code in the host app.
- This prevents attackers from bypassing pinning logic by tampering with JavaScript code in the framework.
- The framework is reusable and abstracted, but security policy remains under the host app's control.

### 💡 Additional Benefits

- Centralized control allows the host to inject authorization headers, enable network logging, enforce offline modes, or simulate failures.
- The modular design means the framework can be updated or replaced without touching the security-critical code.

---

This architecture offers a robust and clean separation between application logic and security enforcement, making it ideal for enterprise-grade React Native integration.

