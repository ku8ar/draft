#import <React/RCTEventEmitter.h>
#import <React/RCTBridgeModule.h>
#import "OnfidoSdk-Swift.h"

@interface OnfidoSdk : RCTEventEmitter <RCTBridgeModule>
@end

@implementation OnfidoSdk {
  BOOL hasListeners;
  static OnfidoSdk *_instance;
}

RCT_EXPORT_MODULE()

+ (BOOL)requiresMainQueueSetup {
  return NO;
}

- (instancetype)init {
  if (self = [super init]) {
    _instance = self;
  }
  return self;
}

+ (OnfidoSdk *)instance {
  return _instance;
}

- (NSArray<NSString *> *)supportedEvents {
  return @[@"onfidoMediaCallback", @"onTokenRequested", @"onTokenGenerated"];
}

- (void)startObserving {
  hasListeners = YES;
}

- (void)stopObserving {
  hasListeners = NO;
}

- (void)emit:(NSString *)name body:(id)body {
  if (hasListeners) {
    [self sendEventWithName:name body:body];
  }
}

RCT_EXPORT_METHOD(start:(NSDictionary *)config
                  resolver:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)
{
  [[OnfidoSdk shared] startWith:config resolve:resolve reject:reject];
}

RCT_EXPORT_METHOD(withMediaCallbacksEnabled) {
  [[OnfidoSdk shared] withMediaCallbacksEnabled];
}

RCT_EXPORT_METHOD(withBiometricTokenCallback) {
  [[OnfidoSdk shared] withBiometricTokenCallback];
}

RCT_EXPORT_METHOD(provideBiometricToken:(NSString *)biometricToken) {
  [[OnfidoSdk shared] provideBiometricToken:biometricToken];
}

// 👇 metody wołane ze Swifta
+ (void)sendMedia:(NSDictionary *)payload {
  [[self instance] emit:@"onfidoMediaCallback" body:payload];
}

+ (void)sendTokenRequested:(NSDictionary *)payload {
  [[self instance] emit:@"onTokenRequested" body:payload];
}

+ (void)sendTokenGenerated:(NSDictionary *)payload {
  [[self instance] emit:@"onTokenGenerated" body:payload];
}

@end
