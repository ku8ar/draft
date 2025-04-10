#import <Foundation/Foundation.h>
#import <React/RCTAppDependencyProvider.h>

@interface BridgeProvider : NSObject <RCTAppDependencyProvider>
@end






#import "BridgeProvider.h"
#import "BridgeBundle.h"

@implementation BridgeProvider

- (id<RCTBridgeDelegate>)bridgeDelegate {
  return [BridgeBundle new];
}

@end
