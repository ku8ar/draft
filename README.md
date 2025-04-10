#import <React/RCTSurfacePresenterBridgeAdapter.h>
#import <React/RCTSurfacePresenter.h>
#import <React/RCTRuntimeExecutorFromBridge.h>
#import <React/RCTBridge+Private.h>

- (void)viewDidLoad {
  [super viewDidLoad];

  NSURL *jsCodeLocation = [[NSBundle mainBundle] URLForResource:@"main" withExtension:@"jsbundle"];
  RCTBridge *bridge = [[RCTBridge alloc] initWithBundleURL:jsCodeLocation
                                              moduleProvider:nil
                                               launchOptions:nil];

  // 🧠 Ręczne uruchomienie SurfacePresenter (zastępuje AppDelegate)
  RCTRuntimeExecutor runtimeExecutor = RCTRuntimeExecutorFromBridge(bridge);
  RCTSurfacePresenterBridgeAdapter *adapter = [[RCTSurfacePresenterBridgeAdapter alloc] initWithBridge:bridge
                                                                                      runtimeExecutor:runtimeExecutor];
  bridge.surfacePresenter = adapter.surfacePresenter;

  RCTRootView *rootView = [[RCTRootView alloc] initWithBridge:bridge
                                                   moduleName:@"rnbridge"
                                            initialProperties:nil];
  rootView.frame = self.view.bounds;
  [self.view addSubview:rootView];
}
