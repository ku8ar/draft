#import <React/RCTBridgeModule.h>
#import <React/RCTEventEmitter.h>

@interface OnfidoSdk : RCTEventEmitter <RCTBridgeModule>
@end

@implementation OnfidoSdk {
  BOOL hasListeners;
}

RCT_EXPORT_MODULE()

+ (BOOL)requiresMainQueueSetup {
  return NO;
}

- (NSArray<NSString *> *)supportedEvents {
  return [[OnfidoSdkBridge shared] supportedEvents];
}

- (void)startObserving {
  hasListeners = YES;
}

- (void)stopObserving {
  hasListeners = NO;
}

- (void)sendEvent:(NSString *)name body:(id)body {
  if (hasListeners) {
    [super sendEventWithName:name body:body];
  }
}

RCT_EXPORT_METHOD(start:(NSDictionary *)config
                  resolver:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject) {
  [[OnfidoSdkBridge shared] startWith:config
                              resolve:^(id result) { resolve(result); }
                              reject:^(NSString *code, NSString *message, NSError *error) {
                                reject(code, message, error);
                              }];
}

RCT_EXPORT_METHOD(withMediaCallbacksEnabled) {
  [[OnfidoSdkBridge shared] withMediaCallbacksEnabled];
}

RCT_EXPORT_METHOD(withBiometricTokenCallback) {
  [[OnfidoSdkBridge shared] withBiometricTokenCallback];
}

RCT_EXPORT_METHOD(provideBiometricToken:(NSString *)biometricToken) {
  [[OnfidoSdkBridge shared] provideBiometricToken:biometricToken];
}

@end
