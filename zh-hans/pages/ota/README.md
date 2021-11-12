# OTA 业务包

## 功能介绍
设备 OTA 指的是设备通过网络进行固件的下载并更新的过程。固件升级业务包可以帮助你快速集成设备固件升级的能力。


## 接入组件
在工程的 `Podfile` 文件中添加 OTA 业务包组件，并执行 `pod update` 命令

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # 添加ota业务包
  pod 'TuyaSmartOTABizBundle'
end
```

**注意**

如果支持蓝牙设备 OTA 升级，首先需要在项目的 info.Plist 文件中添加蓝牙权限的声明。
```
NSBluetoothAlwaysUsageDescription
NSBluetoothPeripheralUsageDescription
```

## 服务协议
OTA 业务包实现 `TYOTAGeneralProtocol` 协议以提供服务，在 TYModuleServices 组件中查看 TYOTAGeneralProtocol.h 协议文件内容为：

```objc
#import <Foundation/Foundation.h>

@class TuyaSmartDeviceModel;

typedef NS_ENUM(NSUInteger, TYOTAControllerTheme) {
    TYOTAControllerWhiteTheme,
    TYOTAControllerBlackTheme
};

@protocol TYOTAGeneralProtocol <NSObject>


/**
 检查设备是否支持固件升级
 
 @param deviceModel 需要检查固件升级的设备
 YES: 支持
 NO: 不支持
 */
- (BOOL)isSupportUpgrade:(TuyaSmartDeviceModel *)deviceModel;

/**
 检查设备固件更新，如果有更新会显示展示出固件更新提示
 
 @param deviceModel 需要检查固件升级的设备
 @param isManual 是否手动检测升级
  @param theme 主题色
 YES: 手动检测升级，检测时弹出loading框。当有固件新版本时(检测升级、强制升级、提醒升级)，显示OTA VC。
 NO: 自动检测升级, 检测时不弹出loading框。当有强制升级时、提醒升级时，弹出固件升级提示，点确定后显示OTA VC。
 */
- (void)checkFirmwareUpgrade:(TuyaSmartDeviceModel *)deviceModel isManual:(BOOL)isManual theme:(TYOTAControllerTheme)theme;

@end

```

若需要自定义处理强制升级返回逻辑（默认 popToRootViewController），需要实现 `TYOTAGeneralExternalProtocol` 提供的协议方法。

```objc
@protocol TYOTAGeneralExternalProtocol <NSObject>


@optional
/// @brief 取消强制升级事件自定义处理
///
/// 1. 强制升级情况，检测升级中，升级弹框点击“取消” 按钮事件。
/// 2. 强制升级情况，点击 navigationBar 返回按钮（有进度情况会有弹框，点击“确定”按钮）。
///
/// @return YES 自定义处理，NO 默认 `[self.navigationController popToRootViewControllerAnimated:YES]`
- (BOOL)didTapCancelForceUpgrade;

@end
```

## 使用指南

### 注意事项
任何接口调用之前，务必确保用户已登录

### 检测升级，进入 OTA 页面

Objective-C 示例

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

Swift 示例

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

### 自定义强制升级返回逻辑
因为“强制升级” 被取消后，默认 `[self.navigationController popToRootViewControllerAnimated:YES]`，
如果需要自定义行为，需要注册并实现 `TYOTAGeneralExternalProtocol`。

Objective-C 示例

```objc
#import <TYModuleServices/TYModuleServices.h>
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>

@interface TYOTAObjcDemo () <TYOTAGeneralExternalProtocol>

@end
@implementation TYOTAObjcDemo

/// 注册协议
+ (void)registerService {
    [[TuyaSmartBizCore sharedInstance] registerService:@protocol(TYOTAGeneralExternalProtocol) withInstance:self];
}

/// 实现协议
#pragma mark - TYOTAGeneralExternalProtocol -
- (BOOL)didTapCancelForceUpgrade {
    NSLog(@"didTapCancelForceUpgrade");
    return YES;
}

@end

```

在适合地方注册  `[TYOTAObjcDemo registerService];`

Swift 示例
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

在适合地方注册  `OTASwiftDemo.registerService()`

