//
//  OnfidoSdk.m
//
//  Copyright © 2016-2025 Onfido. All rights reserved.
//

#import <React/RCTBridgeModule.h>
#import <React/RCTEventEmitter.h>
#import "onfido_react_native_sdk-Swift.h"

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
  return @[@"onfidoMediaCallback", @"onTokenRequested", @"onTokenGenerated"];
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
                  rejecter:(RCTPromiseRejectBlock)reject)
{
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
