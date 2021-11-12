# OTA BizBundle

## Features
Device OTA refers to the process of downloading and updating the firmware of the device through the network. The OTA biz bundle can help you quickly integrate the device firmware upgrade capabilities.

## Add OTA BizBundle
Add OTA bizbundle in `Podfile` , then run `pod update`

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # add ota bizbundle
  pod 'TuyaSmartOTABizBundle'
end
```

**Warnning**

If you need to support Bluetooth devices OTA upgrade, add the following keys to your app’s Information Property List file.

```
NSBluetoothAlwaysUsageDescription
NSBluetoothPeripheralUsageDescription
```

## Service Protocol
OTA bizbundle implements `TYOTAGeneralProtocol` protocol to provide functionality, the content of TYOTAGeneralProtocol.h in TYModuleServices is as following:

```objc
#import <Foundation/Foundation.h>

@class TuyaSmartDeviceModel;

typedef NS_ENUM(NSUInteger, TYOTAControllerTheme) {
    TYOTAControllerWhiteTheme,
    TYOTAControllerBlackTheme
};

@protocol TYOTAGeneralProtocol <NSObject>


/**
 Check if the device supports firmware upgrade
 
 @param deviceModel the device needs to be checked
 
 @return YES supported, NO unsupported
 */
- (BOOL)isSupportUpgrade:(TuyaSmartDeviceModel *)deviceModel;

/**
 Check device firmware upgrade, showing appropriate OTA ViewController according to upgrade status
 
 @param deviceModel the device needs to be checked
 @param isManual manually check for updates
 @param theme theme color
 
 @note
 isManual YES, showing loading UI, when there is a new version of firmware (check upgrade, mandatory upgrade, remind to upgrade), display the OTA ViewController
 
 isManual NO, not showing loading UI, when there is a mandatory upgrade or remind to upgrade, pop the upgrade alert, tap confirm to display the OTA ViewController
 */
- (void)checkFirmwareUpgrade:(TuyaSmartDeviceModel *)deviceModel isManual:(BOOL)isManual theme:(TYOTAControllerTheme)theme;

@end

```

when need to custom the mandatory upgrade cancel behaviour (default is popToRootViewController), you need to implement the `TYOTAGeneralExternalProtocol` protocol

```objc
@protocol TYOTAGeneralExternalProtocol <NSObject>


@optional
/// @brief Custom the mandatory upgrade cancel behaviour
///
/// 1. mandatory upgrade. check upgrade，tap alert view cancel button
/// 2. mandatory upgrade. tap navigationBar back button（if upgrading is on progress, tap confrim button on alert view）。
///
/// @return YES use this method and early return，NO default behaviour `[self.navigationController popToRootViewControllerAnimated:YES]`
- (BOOL)didTapCancelForceUpgrade;

@end
```

## Guidance

### Attention
Make sure that the user is logged in before using any interface

### Check upgrade, enter OTA viewController

Objective-C

```objc
#import <TYModuleServices/TYModuleServices.h>
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>

+ (void)test:(TuyaSmartDeviceModel *)device {
    id<TYOTAGeneralProtocol> otaImp = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYOTAGeneralProtocol)];
    
    if ([otaImp isSupportUpgrade:device]) {
        [otaImp checkFirmwareUpgrade:device isManual:YES theme:TYOTAControllerWhiteTheme];
    } else {
        NSLog(@"unsupported");
    }
}
```

Swift

```swift
func test(_ deviceModel: TuyaSmartDeviceModel) {
	guard let otaImp = TuyaSmartBizCore.sharedInstance().service(of: TYOTAGeneralProtocol.self) as? TYOTAGeneralProtocol else { return }
	
	if otaImp.isSupportUpgrade(deviceModel) {
		otaImp.checkFirmwareUpgrade(deviceModel, isManual: true, theme: .whiteTheme)
	} else {
		print("unsupported")
	}
}
```

### Custom the mandatory upgrade cancel behaviour
when mandatory upgrad is canceled, default is `[self.navigationController popToRootViewControllerAnimated:YES]`，

if you need custom mandatory upgrade cancel behaviour, register and implements `TYOTAGeneralExternalProtocol`.

Objective-C

```objc
#import <TYModuleServices/TYModuleServices.h>
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>

@interface TYOTAObjcDemo () <TYOTAGeneralExternalProtocol>

@end
@implementation TYOTAObjcDemo

/// register protocol
+ (void)registerService {
    [[TuyaSmartBizCore sharedInstance] registerService:@protocol(TYOTAGeneralExternalProtocol) withInstance:self];
}

/// implement protocol
#pragma mark - TYOTAGeneralExternalProtocol -
- (BOOL)didTapCancelForceUpgrade {
    NSLog(@"didTapCancelForceUpgrade");
    return YES;
}

@end

```

call  `[TYOTAObjcDemo registerService];`

Swift
```swift
import Foundation

class OTASwiftDemo: NSObject, TYOTAGeneralExternalProtocol {
    func registerService() {
        TuyaSmartBizCore.sharedInstance().registerService(TYOTAGeneralExternalProtocol.self, withInstance: self)
    }
    
    // MARK: - TYOTAGeneralExternalProtocol -
    func didTapCancelForceUpgrade() -> Bool {
        print("didTapCancelForceUpgrade")
        return true
    }
}
```

call  `OTASwiftDemo.registerService()`
