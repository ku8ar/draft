## 🔁 Architecture: Native-Controlled Networking (Host ➝ JS)

+------------------------------------------------------------+
|                 🟢 Host App (iOS / Android)                |
|------------------------------------------------------------|
|                                                            |
|  1. Create secure session:                                 |
|     - iOS: NSURLSession with SSL pinning                   |
|     - Android: OkHttpClient with proxy, certs              |
|                                                            |
|  2. Inject into native module:                             |
|     DBSFetch.shared().configure(with: session)             |
|                                                            |
|  3. DBSFetch.fetch(...) exposed to JS via bridge           |
|     ⇨ Accepts: url, method, headers, body                  |
|     ⇨ Returns: status, headers, body                       |
+------------------------------▼-----------------------------+

+------------------------------------------------------------+
|             🟡 React Native Bridge & Integration           |
|------------------------------------------------------------|
|                                                            |
|  4. MonkeyXMLHttpRequest wraps DBSFetch.fetch              |
|     - Emits standard XHR events                            |
|     - Looks & behaves like real XMLHttpRequest             |
|                                                            |
|  5. global.XMLHttpRequest = MonkeyXMLHttpRequest           |
|     - Intercepts all network usage in JS                   |
+------------------------------▼-----------------------------+

+------------------------------------------------------------+
|                  🔵 JavaScript Runtime                     |
|------------------------------------------------------------|
|                                                            |
|  6. Any library using fetch() / XHR                        |
|     ⇨ axios, graphql-upload, socket.io, etc.               |
|                                                            |
|  7. Requests go through MonkeyXMLHttpRequest               |
|     ⇨ routed via DBSFetch to native session                |
|                                                            |
|  ✅ JavaScript has NO direct internet access               |
+------------------------------------------------------------+
