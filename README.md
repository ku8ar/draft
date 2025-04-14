📄 OnfidoSdk.h

#import <React/RCTEventEmitter.h>
#import <React/RCTBridgeModule.h>

@interface OnfidoSdk : RCTEventEmitter <RCTBridgeModule>

- (void)start:(NSDictionary *)config
     resolver:(RCTPromiseResolveBlock)resolve
     rejecter:(RCTPromiseRejectBlock)reject;

- (void)withMediaCallbacksEnabled;
- (void)withBiometricTokenCallback;
- (void)provideBiometricToken:(NSString *)biometricToken;

@end

📄 OnfidoSdk.m

#import "OnfidoSdk.h"
#import <Onfido/Onfido.h>
#import "UIViewController+TopMost.h"
#import "NSDictionary+Color.h"

typedef NS_ENUM(NSUInteger, CallbackType) {
    CallbackTypeMedia,
    CallbackTypeEncryptedBiometricToken
};

@interface OnfidoSdk ()
@property (nonatomic, strong) NSMutableArray<NSNumber *> *callbackTypes;
@property (nonatomic, strong) EncryptedBiometricTokenHandlerReceiver *encryptedBiometricTokenHandlerReceiver;
@end

@implementation OnfidoSdk

RCT_EXPORT_MODULE();

- (instancetype)init {
    if (self = [super init]) {
        _callbackTypes = [NSMutableArray new];
        __weak typeof(self) weakSelf = self;
        _encryptedBiometricTokenHandlerReceiver = [[EncryptedBiometricTokenHandlerReceiver alloc]
            initWithTokenRequestedCallback:^(NSDictionary *result) {
                [weakSelf sendEventWithName:@"onTokenRequested" body:result];
            }
            andTokenGeneratedCallback:^(NSDictionary *result) {
                [weakSelf sendEventWithName:@"onTokenGenerated" body:result];
            }];
    }
    return self;
}

- (NSArray<NSString *> *)supportedEvents {
    return @[@"onfidoMediaCallback", @"onTokenRequested", @"onTokenGenerated"];
}

+ (BOOL)requiresMainQueueSetup {
    return NO;
}

RCT_EXPORT_METHOD(withMediaCallbacksEnabled) {
    [self.callbackTypes addObject:@(CallbackTypeMedia)];
}

RCT_EXPORT_METHOD(withBiometricTokenCallback) {
    [self.callbackTypes addObject:@(CallbackTypeEncryptedBiometricToken)];
}

RCT_EXPORT_METHOD(provideBiometricToken:(NSString *)biometricToken) {
    [self.encryptedBiometricTokenHandlerReceiver provideWithEncryptedBiometricToken:biometricToken];
}

RCT_EXPORT_METHOD(start:(NSDictionary *)config
                  resolver:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject) {
    dispatch_async(dispatch_get_main_queue(), ^{
        [self runWithConfig:config resolver:resolve rejecter:reject];
    });
}

- (void)runWithConfig:(NSDictionary *)config
             resolver:(RCTPromiseResolveBlock)resolve
             rejecter:(RCTPromiseRejectBlock)reject {

    NSError *parseError = nil;
    OnfidoPluginConfig *onfidoConfig = [[OnfidoConfigParser new] parse:config error:&parseError];

    if (parseError) {
        reject([@(parseError.code) stringValue], parseError.domain, parseError);
        return;
    }

    NSString *appearanceFilePath = [[NSBundle mainBundle] pathForResource:@"colors" ofType:@"json"];
    NSError *appearanceError = nil;
    OnfidoAppearance *appearance = [AppearanceLoader loadAppearanceFromFile:appearanceFilePath error:&appearanceError];

    if (appearanceError) {
        reject(@"appearance_error", @"Failed to load appearance", appearanceError);
        return;
    }

    if (@available(iOS 12.0, *)) {
        if (onfidoConfig.theme) {
            switch (onfidoConfig.theme) {
                case OnfidoThemeDark:
                    [appearance setUserInterfaceStyle:UIUserInterfaceStyleDark];
                    break;
                case OnfidoThemeLight:
                    [appearance setUserInterfaceStyle:UIUserInterfaceStyleLight];
                    break;
                case OnfidoThemeAutomatic:
                    [appearance setUserInterfaceStyle:UIUserInterfaceStyleUnspecified];
                    break;
            }
        }
    }

    CallbackReceiver *mediaCallback = nil;
    if ([self.callbackTypes containsObject:@(CallbackTypeMedia)]) {
        __weak typeof(self) weakSelf = self;
        mediaCallback = [[CallbackReceiver alloc] initWithCallback:^(NSDictionary *result) {
            [weakSelf sendEventWithName:@"onfidoMediaCallback" body:result];
        }];
    }

    EncryptedBiometricTokenHandler *biometricTokenHandler = [self.callbackTypes containsObject:@(CallbackTypeEncryptedBiometricToken)]
        ? self.encryptedBiometricTokenHandlerReceiver
        : nil;

    NSError *flowError = nil;
    OnfidoFlow *flow = [[[OnfidoFlowBuilder new]
        buildWithConfig:onfidoConfig
        appearance:appearance
        customMediaCallback:mediaCallback
        customEncryptedBiometricTokenHandler:biometricTokenHandler
        error:&flowError] withResponseHandler:^(OnfidoResponse *response) {

            switch (response.status) {
                case OnfidoResponseStatusSuccess:
                    resolve(createResponse(response.results));
                    break;
                case OnfidoResponseStatusError:
                    reject(@"error", @"Encountered an error", response.error);
                    break;
                case OnfidoResponseStatusCancel:
                    reject(@"userExit", @"User canceled", nil);
                    break;
                default:
                    reject(@"unknown", @"Unknown result", nil);
                    break;
            }
        }];

    if (flowError) {
        reject(@"flow_build_error", @"Failed to build Onfido flow", flowError);
        return;
    }

    UIWindow *window = UIApplication.sharedApplication.windows.firstObject;
    UIViewController *topVC = [window.rootViewController findTopMostController];

    NSError *runError = nil;
    [flow runFrom:topVC presentationStyle:UIModalPresentationFullScreen animated:YES error:&runError];
    if (runError) {
        reject(@"run_error", @"Failed to run Onfido flow", runError);
    }
}

@end

📄 UIViewController+TopMost.h

#import <UIKit/UIKit.h>

@interface UIViewController (TopMost)

- (UIViewController *)findTopMostController;

@end

📄 UIViewController+TopMost.m

#import "UIViewController+TopMost.h"

@implementation UIViewController (TopMost)

- (UIViewController *)findTopMostController {
    UIViewController *topController = self;
    while (topController.presentedViewController) {
        topController = topController.presentedViewController;
    }
    return topController;
}


::contentReference[oaicite:5]{index=5}
 






