# Message Center BizBundle

## Introduction

The message center bizBundle provides the business logic of the message center of Tuya APP. The business functions mainly cover the push historical records of various types of messages, mainly including three categories of alarms, homes, and notifications. The alarms include device alarms, scene automation and other execution records.
The setting page of the message center can enable or disable various types of message push. For alarm messages, it can support adding no-disturb periods to the device.



## **Integrate**

Add the `TuyaSmartMessageBizBundle` in the project's `Podfile` file and execute the` pod update` command

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # Add Message BzBndle
  pod 'TuyaSmartMessageBizBundle'
end
```



## Service Protocol

### Service Provided By BizBundle

The message BizBundle implements the `TYMessageCenterProtocol` protocol to provide services，View the`TYMessageCenterProtocol.h` file in the `TYModuleServices` as follows:

```objective-c
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@protocol TYMessageCenterProtocol <NSObject>

/// push消息中心页面 push message center vc
/// @param animated animated
- (void)gotoMessageCenterViewControllerWithAnimated:(BOOL)animated;

@end

NS_ASSUME_NONNULL_END
```



### Dependent Services Required By BizBundle

BizBundle running depends on the protocol method provided by `TYSmartHomeDataProtocol`，Before calling the BizBundle method, the following protocol needs to be implemented

#### TYSmartHomeDataProtocol

Set current home infomation  by implementing the following method.

```objective-c
/**
 Get current home info 
 
 @return TuyaSmartHome
 */
- (TuyaSmartHome *)getCurrentHome;
```



## Operating Guide

### Attention

1. Make sure that the user is logged in before using any interface

2. When the login user changes, be sure to re-judge the mall availability status and re-acquire the mall page

3. Before using bizBundle，must  implement the protocol method `getCurrentHome` in `TYSmartHomeDataProtocol`

Objective-C 

```objective-c
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYSmartHomeDataProtocol.h>


- (void)initCurrentHome {
    // register service
    [[TuyaSmartBizCore sharedInstance] registerService:@protocol(TYSmartHomeDataProtocol) withInstance:self];
}

// implementation
- (TuyaSmartHome *)getCurrentHome {
    TuyaSmartHome *home = [TuyaSmartHome homeWithHomeId:@"current home id"];
    return home;
}
```

Swfit 

```swift
import TuyaSmartDeviceKit

class TYMessageCenterTest: NSObject,TYSmartHomeDataProtocol{

    
    func test() {
        TuyaSmartBizCore.sharedInstance().registerService(TYSmartHomeDataProtocol.self, withInstance: self)
    }
    
    func getCurrentHome() -> TuyaSmartHome! {
        let home = TuyaSmartHome.init(homeId: 111)
        return home
    }
    
}
```




## Go To Message Center Page

Objective-C 

```objective-c
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYMessageCenterProtocol.h>


- (void)gotoDeviceConfig {
    id<TYMessageCenterProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYMessageCenterProtocol)];
    [impl gotoMessageCenterViewControllerWithAnimated:YES];
}
```

Swfit 

``` swift
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYMessageCenterProtocol.self) as? TYMessageCenterProtocol
impl?.gotoMessageCenterViewController(animated: true)
```
Remarks: Due to the open capability of the bizBundle and the dependence of functional components, the click response of some alarm information links in the message center is temporarily not supported.