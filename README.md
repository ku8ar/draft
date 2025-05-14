diff --git a/node_modules/tealium-react-native/.DS_Store b/node_modules/tealium-react-native/.DS_Store
new file mode 100644
index 0000000..7083c2c
Binary files /dev/null and b/node_modules/tealium-react-native/.DS_Store differ
diff --git a/node_modules/tealium-react-native/ios/.DS_Store b/node_modules/tealium-react-native/ios/.DS_Store
new file mode 100644
index 0000000..0187a9b
Binary files /dev/null and b/node_modules/tealium-react-native/ios/.DS_Store differ
diff --git a/node_modules/tealium-react-native/ios/EventEmitter.swift b/node_modules/tealium-react-native/ios/EventEmitter.swift
index 78f4681..3bf83be 100644
--- a/node_modules/tealium-react-native/ios/EventEmitter.swift
+++ b/node_modules/tealium-react-native/ios/EventEmitter.swift
@@ -8,23 +8,30 @@
 
 import Foundation
 
-public class EventEmitter {
+@objc public protocol EventEmitting {
+    func sendEvent(name: String, body: Any?)
+}
+
+public class EventEmitter: NSObject {
 
-    public static var shared = EventEmitter()
-    private static var eventEmitter: TealiumReactNative?
+    public static let shared = EventEmitter()
+    @objc public static func _shared() -> EventEmitter {
+        return shared
+    }
+    private static var eventEmitter: EventEmitting?
 
-    private init() {}
+    private override init() {}
     
-    lazy var allEvents: [String] = {
-        TealiumReactConstants.Events.allCases.map { $0.rawValue }
-    }()
+    @objc public var allEvents: [String] {
+        return TealiumReactConstants.Events.allCases.map { $0.rawValue }
+    }
 
-    func registerEventEmitter(eventEmitter: TealiumReactNative) {
+    @objc public func registerEventEmitter(eventEmitter: EventEmitting) {
         EventEmitter.eventEmitter = eventEmitter
     }
 
-    func dispatch(name: String, body: Any?) {
-        EventEmitter.eventEmitter?.sendEvent(withName: name, body: body)
+    @objc public func dispatch(name: String, body: Any?) {
+        EventEmitter.eventEmitter?.sendEvent(name: name, body: body)
     }
 
 }
diff --git a/node_modules/tealium-react-native/ios/TealiumReactNative.swift b/node_modules/tealium-react-native/ios/TealiumReactNative.swift
index a632cce..c7d3f41 100644
--- a/node_modules/tealium-react-native/ios/TealiumReactNative.swift
+++ b/node_modules/tealium-react-native/ios/TealiumReactNative.swift
@@ -11,7 +11,7 @@ import TealiumSwift
 import tealium_react_native
 
 @objc(TealiumReactNative)
-public class TealiumReactNative: RCTEventEmitter {
+public class TealiumReactNative: NSObject {
     
     static var tealium: Tealium?
     private static var config: TealiumConfig?
@@ -59,11 +59,6 @@ public class TealiumReactNative: RCTEventEmitter {
         }
     }
 
-    override init() {
-        super.init()
-        EventEmitter.shared.registerEventEmitter(eventEmitter: self)
-    }
-    
     public static func registerRemoteCommandFactory(_ factory: RemoteCommandFactory) {
         remoteCommandFactories[factory.name] = factory
     }
@@ -72,16 +67,6 @@ public class TealiumReactNative: RCTEventEmitter {
         optionalModules.append(module)
     }
 
-    @objc
-    public override static func requiresMainQueueSetup() -> Bool {
-        return false
-    }
-    
-    @objc
-    public override func supportedEvents() -> [String] {
-        return EventEmitter.shared.allEvents
-    }
-    
     @objc
     public static func initialize(_ config: [String: Any], _ completion: @escaping (Bool) -> Void) {
         guard let localConfig = tealiumConfig(from: config) else {
diff --git a/node_modules/tealium-react-native/ios/TealiumReactNativeEmitter.m b/node_modules/tealium-react-native/ios/TealiumReactNativeEmitter.m
new file mode 100644
index 0000000..a769595
--- /dev/null
+++ b/node_modules/tealium-react-native/ios/TealiumReactNativeEmitter.m
@@ -0,0 +1,29 @@
+#import "React/RCTEventEmitter.h"
+#import "React/RCTBridgeModule.h"
+#import <Foundation/Foundation.h>
+#import <tealium_react_native-Swift.h>
+
+@interface TealiumReactNativeEmitter : RCTEventEmitter <RCTBridgeModule, EventEmitting>
+@end
+
+@implementation TealiumReactNativeEmitter
+
+RCT_EXPORT_MODULE();
+
+- (instancetype)init {
+  self = [super init];
+  if (self) {
+      [[EventEmitter _shared] registerEventEmitterWithEventEmitter:self];
+  }
+  return self;
+}
+
++ (BOOL)requiresMainQueueSetup {
+  return NO;
+}
+
+- (NSArray<NSString *> *)supportedEvents {
+  return EventEmitter._shared.allEvents;
+}
+
+@end
diff --git a/node_modules/tealium-react-native/ios/TealiumWrapper.swift b/node_modules/tealium-react-native/ios/TealiumWrapper.swift
index 6b0530a..9c478af 100644
--- a/node_modules/tealium-react-native/ios/TealiumWrapper.swift
+++ b/node_modules/tealium-react-native/ios/TealiumWrapper.swift
@@ -18,7 +18,7 @@ class TealiumWrapper: NSObject {
     }
     
     @objc(initialize:callback:)
-    public func initialize(_ config: [String: Any], callback: @escaping RCTResponseSenderBlock) {
+    public func initialize(_ config: [String: Any], callback: @escaping ([Any]) -> Void) {
         TealiumReactNative.initialize(config) { result in
             if result {
                 callback([result])
@@ -47,9 +47,9 @@ class TealiumWrapper: NSObject {
     }
     
     @objc(getFromDataLayer:callback:)
-    public func getFromDataLayer(_ key: String, callback: RCTResponseSenderBlock) {
+    public func getFromDataLayer(_ key: String, callback: @escaping ([Any]) -> Void) {
         guard let item = TealiumReactNative.getFromDataLayer(key: key) else {
-            callback(nil)
+            callback([])
             return
         }
         callback([item])
@@ -61,10 +61,10 @@ class TealiumWrapper: NSObject {
     }
 
     @objc(gatherTrackData:)
-    public func gatherTrackData(callback: @escaping RCTResponseSenderBlock) {
+    public func gatherTrackData(callback: @escaping ([Any]) -> Void) {
         TealiumReactNative.gatherTrackData(completion: { response in
             guard let result = response as Any? else {
-                callback(nil)
+                callback([])
                 return
             }
             callback([result])
@@ -82,7 +82,7 @@ class TealiumWrapper: NSObject {
     }
     
     @objc(getConsentStatus:)
-    public func getConsentStatus(_ callback: RCTResponseSenderBlock) {
+    public func getConsentStatus(_ callback: @escaping ([Any]) -> Void) {
         callback([TealiumReactNative.consentStatus])
     }
     
@@ -92,7 +92,7 @@ class TealiumWrapper: NSObject {
     }
     
     @objc(getConsentCategories:)
-    public func getConsentCategories(_ callback: RCTResponseSenderBlock) {
+    public func getConsentCategories(_ callback: @escaping ([Any]) -> Void) {
         callback([TealiumReactNative.consentCategories])
     }
     
@@ -112,7 +112,7 @@ class TealiumWrapper: NSObject {
     }
 
     @objc(getVisitorId:)
-    public func getVisitorId(_ callback: RCTResponseSenderBlock) {
+    public func getVisitorId(_ callback: @escaping ([Any]) -> Void) {
         guard let visitorId = TealiumReactNative.visitorId else {
             callback([""])
             return
@@ -131,7 +131,7 @@ class TealiumWrapper: NSObject {
     }
     
     @objc(getSessionId:)
-    public func getSessionId(_ callback: RCTResponseSenderBlock) {
+    public func getSessionId(_ callback: @escaping ([Any]) -> Void) {
         guard let sessionId = TealiumReactNative.sessionId else {
             callback([""])
             return
diff --git a/node_modules/tealium-react-native/tealium-react-native.podspec b/node_modules/tealium-react-native/tealium-react-native.podspec
index 01f5471..3d96bad 100644
--- a/node_modules/tealium-react-native/tealium-react-native.podspec
+++ b/node_modules/tealium-react-native/tealium-react-native.podspec
@@ -27,5 +27,7 @@ Pod::Spec.new do |s|
   s.dependency "tealium-swift/RemoteCommands", "~> 2.16"
   s.dependency "tealium-swift/VisitorService", "~> 2.16"
 
+  s.static_framework = true
+
 end
 
