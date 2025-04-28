#import <React/RCTBridgeModule.h>

@interface RCT_EXTERN_MODULE(DBSFetchModule, NSObject)

RCT_EXTERN_METHOD(fetch:(NSString *)urlString
                  resolver:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)

@end
