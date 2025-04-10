#import <React/RCTAppSetupUtils.h>
#import <React/RCTAppDependencyProvider.h>
#import "BridgeWrapperBridgeDelegate.h"

@interface BridgeWrapperDependencyProvider : NSObject <RCTAppDependencyProvider>
@end

@implementation BridgeWrapperDependencyProvider

- (id<RCTBridgeDelegate>)bridgeDelegate {
  return [BridgeWrapperBridgeDelegate new];
}

@end

__attribute__((constructor))
static void RegisterBridgeWrapperDependencyProvider() {
  RCTAppSetupSetAppDependencyProvider([BridgeWrapperDependencyProvider new]);
}
