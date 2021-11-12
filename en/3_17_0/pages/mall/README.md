# H5 Mall

## Introduction

H5 Mall provides an container that hosts the "App mall", so that your app has powerful mall capabilities and allows mobile traffic to be realized through the mall.

> "App mall" is a value-added service provided by Tuya platform. For details, you can serach "App mall" in Tuya Smart Platform [Value-added Service](https://www.tuya.com/vas/)

## Integrate

Add the `TuyaSmartMallBizBundle` in the project's `Podfile` file and execute the` pod update` command

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # TuyaSmart SDK
  pod "TuyaSmartHomeKit"
  # Add H5 Mall BizBundle
  pod 'TuyaSmartMallBizBundle'
end
```

## Service Protocol

### Service Provided By BizBundle

The H5 mall BizBundle implements the `TYMallProtocol` protocol to provide services. View the` TYMallProtocol.h` file in the `TYModuleServices` as follows:

```oc
#import <Foundation/Foundation.h>

typedef NS_ENUM(NSUInteger, TYMallPageType) {
    TYMallPageTypeHome,      // Mall home page
    TYMallPageTypeOrders,    // Mall orders page
};

@protocol TYMallProtocol <NSObject>

/**
 * Check if mall enable for current logged user.
 * You should check this every time after logged user changed.
 */
- (void)checkIfMallEnableForCurrentUser:(void(^)(BOOL enable, NSError *error))callback;

/**
 * Request special mall page with `TYMallPageType`
 * You should replace mall page every time after logged user changed.
 * @param pageType Mall page type
 */
- (void)requestMallPage:(TYMallPageType)pageType completionBlock:(void(^)(__kindof UIViewController *page, NSError *error))callback;

@end
```

### Service Required By BizBundle
None.

## User Guidance

### Precautions

1. Before using any method, be sure to confirm that the user is logged in.
2. Before obtaining any mall page, you need to first confirm whether the service area registered by the current user is open for the mall.
3. When the login user changes, be sure to re-judge the mall availability status and re-acquire the mall page
4. Get the mall page as `UIViewController`, be sure to use` UINavigationController` to `push` or` present` it.

> The mall page depends on the navigation controller and sets the navigation bar content.

### Check mall availability

Objective-C

```oc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYMallProtocol.h>

id<TYMallProtocol> mallImpl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYMallProtocol)];
[mallImpl checkIfMallEnableForCurrentUser:^(BOOL enable, NSError *error) {
  if (error) {
    // Return error when query fails
    NSLog(@"%@",error);
  } else {
    // enable is true, the mall is available, otherwise it is not available
    NSLog(@"%@",@(enable));
  }
}];
```

Swift

```swift
let mallImpl = TuyaSmartBizCore.sharedInstance().service(of: TYMallProtocol.self)
(mallImpl as? TYMallProtocol)?.checkIfMallEnable(forCurrentUser: { (enable, error) in
    if let e = error {
        print("\(e)")
        return
    }
    print("\(enable)")
})
```

### Get mall page (UIViewController)

Two pages are currently provided:
- Mall index page (TYMallPageTypeHome)
- Mall orders page (TYMallPageTypeOrders)

you can obtain the corresponding page to display according to your needs

Objective-C

```oc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYMallProtocol.h>

id<TYMallProtocol> mallImpl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYMallProtocol)];
[impl requestMallPage:TYMallPageTypeHome completionBlock:^(__kindof UIViewController *page, NSError *error) {
    UINavigationController *nav = [[UINavigationController alloc]initWithRootViewController:page];
    [self presentViewController:nav animated:YES completion:nil];
}];
```

Swift

```swift
let mallImpl = TuyaSmartBizCore.sharedInstance().service(of: TYMallProtocol.self)
(mallImpl as? TYMallProtocol)?.request(.home, completionBlock: { (mallVc, error) in
    if let e = error {
        print("\(e)")
        return
    }
    // push
    yourNaviController.pushViewController(mallVc!, animated: true)
    // present
    let naviVc = UINavigationController.init(rootViewController: mallVc!)
    yourViewController.present(naviVc, animated: true, completion: nil)
})
```



