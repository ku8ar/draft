#import <React/RCTBridgeModule.h>
#import <React/RCTEventEmitter.h>

@interface OnfidoSdk : RCTEventEmitter <RCTBridgeModule>
@end

@implementation OnfidoSdk {
  BOOL hasListeners;
}

RCT_EXPORT_MODULE();

+ (BOOL)requiresMainQueueSetup {
  return NO;
}

- (instancetype)init {
  if (self = [super init]) {
    [OnfidoSdk.shared setBridgeEmitter:self]; // <-- najważniejsze
  }
  return self;
}

- (NSArray<NSString *> *)supportedEvents {
  return [OnfidoSdk.shared supportedEvents];
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
  [OnfidoSdk.shared startWith:config resolve:resolve reject:reject];
}

RCT_EXPORT_METHOD(supportedEvents) {
  // niepotrzebne, używane tylko w extern_module
}

RCT_EXPORT_METHOD(withMediaCallbacksEnabled) {
  [OnfidoSdk.shared withMediaCallbacksEnabled];
}

RCT_EXPORT_METHOD(withBiometricTokenCallback) {
  [OnfidoSdk.shared withBiometricTokenCallback];
}

RCT_EXPORT_METHOD(provideBiometricToken:(NSString *)token) {
  [OnfidoSdk.shared provideBiometricToken:token];
}

@end
