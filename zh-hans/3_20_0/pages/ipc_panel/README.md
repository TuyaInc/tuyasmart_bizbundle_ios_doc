- [摄像机 Native 面板业务包](#native)
- [摄像机 RN 面板业务包](#rn)
- [摄像机设置面板业务包](#cameraSetting)

<span id="native"></span>
# 摄像机 Native 面板业务包

## 功能概述

涂鸦智能 iOS IPC 业务包 （TuyaSmartCameraPanelBizBundle） 是基于 [Tuya Smart Camera SDK](<https://tuyainc.github.io/tuyasmart_camera_ios_sdk_doc/>) 开发的一系列摄像机功能相关的面板 SDK。主要包括以下功能：

- 预览面板，回放面板，云存储面板，消息中心面板，相册面板，设置面板。

## 接入组件

在  ```Podfile``` 文件中加入以下代码：

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # 添加摄像机面板业务包
  pod 'TuyaSmartCameraPanelBizBundle'
end
```

然后在项目根目录下执行 ```pod update``` 命令，集成第三方库。

CocoaPods 的使用请参考：[CocoaPods Guides](https://guides.cocoapods.org/)

**注意**

设备控制业务包中封装了一系列 RN 接口供面板调用，其中会涉及到部分苹果隐私权限的声明。

- 如果接入的设备面板有使用相册相关的（例如：相册），则需要在工程的 `info.plist` 中添加如下权限声明：

```
NSPhotoLibraryAddUsageDescription
```

- 如果接入的设备面板有使用到麦克风（例如：摄像机对讲），则需要在工程的 `info.plist` 中添加如下权限声明：

```
NSMicrophoneUsageDescription
```

## 服务协议

摄像机业务包实现 `TYCameraProtocol` 协议以提供服务，在 `TYModuleServices` 组件中查看 `TYCameraProtocol.h` 协议文件内容为：

```objc
#import <UIKit/UIKit.h>

@class TuyaSmartDeviceModel;

@protocol TYCameraProtocol <NSObject>

/**
 获取摄像头Native面板
 @param devId 摄像头设备的devId
 @param uiName 摄像头设备的uiName，不同的uiName对应不同版本的面板 为deviceModel里的uiName属性
 */
- (UIViewController *)viewControllerWithDeviceId:(NSString *)devId uiName:(NSString *)uiName;

@optional

/**
 跳转摄像头回放面板
 @param deviceModel 摄像头设备
 */
- (void)deviceGotoCameraNewPlayBackPanel:(TuyaSmartDeviceModel *)deviceModel;

/**
 跳转摄像头云存储面板
 @param deviceModel 摄像头设备
 */
- (void)deviceGotoCameraCloudStoragePanel:(TuyaSmartDeviceModel *)deviceModel;

/**
 跳转摄像头消息中心面板
 @param deviceModel 摄像头设备
 */
- (void)deviceGotoCameraMessageCenterPanel:(TuyaSmartDeviceModel *)deviceModel;

/**
 跳转摄像头相册面板
 @param deviceModel 摄像头设备
 */
- (void)deviceGotoPhotoLibrary:(TuyaSmartDeviceModel *)deviceModel;

@end
```

### 依赖服务

设备控制业务包主要功能为加载设备，针对不同设备会有不同的一些功能，为保证这些功能正常运行，会依赖如下几个协议： `TYSmartHomeDataProtocol`、 `TYOTAGeneralProtocol` 。

#### TYSmartHomeDataProtocol

提供加载设备面板所需的当前家庭信息，**必须实现**

```objc
/**
 获取当前的家庭，当前没有家庭的时候，返回nil。
 
 @return TuyaSmartHome
 */
- (TuyaSmartHome *)getCurrentHome;
```

#### TYOTAGeneralProtocol

进入设备面板时，提供检查设备固件更新的事件。实现如下方法用于检查固件升级：

```objc
/**
 检查设备固件更新，如果有更新会显示展示出固件更新提示
 
 @param deviceModel 需要检查固件升级的设备
 @param isManual 是否手动检测升级
  @param theme 主题色
 YES: 手动检测升级，检测时弹出loading框。当有固件新版本时(检测升级、强制升级、提醒升级)，显示OTA VC。
 NO: 自动检测升级, 检测时不弹出loading框。当有强制升级时、提醒升级时，弹出固件升级提示，点确定后显示OTA VC。
 */
- (void)checkFirmwareUpgrade:(TuyaSmartDeviceModel *)deviceModel isManual:(BOOL)isManual theme:(TYOTAControllerTheme)theme;
```

#### 

## 使用指南

### 注意事项

1. 使用任何接口之前，务必确认该设备在当前用户下。
2. 此接口，只适用于摄像机设备调用，即 deviceModel.category 为 “sp” 类型的设备。
3. 调用业务包逻辑前，要先实现 `TYSmartHomeDataProtocol` 中的协议方法`getCurrentHome`。
4. 此业务包，与之前的 `TuyaSmartCameraPanelSDK` 互斥，二者不能共存，迁移之前，请查看 [迁移指南](https://tuyainc.github.io/tuyasmart_camera_panel_ios_sdk_doc/zh-hans/Upgrading.html)。

Objective-C 

```objc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYSmartHomeDataProtocol.h>

- (void)initCurrentHome {
    // 注册要实现的协议
    [[TuyaSmartBizCore sharedInstance] registerService:@protocol(TYSmartHomeDataProtocol) withInstance:self];
}

// 实现对应的协议方法
- (TuyaSmartHome *)getCurrentHome {
    TuyaSmartHome *home = [TuyaSmartHome homeWithHomeId:@"当前家庭id"];
    return home;
}
```

Swift

```swift
import TuyaSmartDeviceKit

class TYActivatorTest: NSObject,TYSmartHomeDataProtocol{
  	
    func test() {
        TuyaSmartBizCore.sharedInstance().registerService(TYSmartHomeDataProtocol.self, withInstance: self)
    }
    
    func getCurrentHome() -> TuyaSmartHome! {
        let home = TuyaSmartHome.init(homeId: 111)
        return home
    }
    
}
```

### 

### 获取预览面板 (UIViewController)

摄像机原生预览面板，包括视频实时预览，清晰度切换，声音开关控制，截图，录制，对讲等功能，移动侦测，PTZ 方向控制，收藏点添加/删除，巡航控制等。

Objc

```objc
id<TYCameraProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYCameraProtocol)];
UIViewController *vc = [impl viewControllerWithDeviceId:self.deviceModel.devId uiName:self.device.uiName];
[self.navigationController pushViewController:vc animated:YES];
```

Swift

```swift
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYCameraProtocol.self) as? TYCameraProtocol
impl?.viewControllerWithDeviceId(withDeviceId: deviceModel.devId!, uiName: deviceModel.uiName) 
```

<span id="rn"></span>
# 摄像机 RN 面板业务包

## 功能概述

涂鸦智能 iOS IPC RN 业务包（ TuyaSmartCameraRNPanelBizBundle ）是基于 [Tuya Smart Camera SDK](<https://tuyainc.github.io/tuyasmart_camera_ios_sdk_doc/>) 开发的一系列摄像机功能相关的面板 SDK。主要包括以下功能：

- 预览面板，回放面板，云存储面板，消息中心面板，相册面板，设置面板。

## 接入组件

在  ```Podfile``` 文件中加入以下代码：

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # 添加面板控制业务包
  pod 'TuyaSmartPanelBizBundle'
  # 添加摄像机面板业务包 接入RN页面包的同时，也接入该业务包，使得原生相册等功能也可以正常使用。
  pod 'TuyaSmartCameraPanelBizBundle'
  # 添加摄像机 RN 面板业务包
  pod 'TuyaSmartCameraRNPanelBizBundle'
end
```

然后在项目根目录下执行 ```pod update``` 命令，集成第三方库。

CocoaPods 的使用请参考：[CocoaPods Guides](https://guides.cocoapods.org/)

**注意**

设备控制业务包中封装了一系列 RN 接口供面板调用，其中会涉及到部分苹果隐私权限的声明。

- 如果接入的设备面板有使用相册相关的（例如：相册），则需要在工程的 `info.plist` 中添加如下权限声明：

```
NSPhotoLibraryAddUsageDescription
```

- 如果接入的设备面板有使用到麦克风（例如：摄像机对讲），则需要在工程的 `info.plist` 中添加如下权限声明：

```
NSMicrophoneUsageDescription
```

## 服务协议

摄像机业务包实现 `TYRNCameraProtocol` 协议以提供服务，在 `TYModuleServices` 组件中查看 `TYRNCameraProtocol.h` 协议文件内容为：

```objc
#import <UIKit/UIKit.h>

@protocol TYRNCameraProtocol <NSObject>

/**
 获取摄像头RN面板
 @param devId 摄像头设备的devId
 */
- (UIViewController *)cameraRNPanelViewControllerWithDeviceId:(NSString *)devId;

@end
```

### 依赖服务

设备控制业务包主要功能为加载设备，针对不同设备会有不同的一些功能，为保证这些功能正常运行，会依赖如下几个协议： `TYSmartHomeDataProtocol`、 `TYOTAGeneralProtocol` 。

#### TYSmartHomeDataProtocol

提供加载设备面板所需的当前家庭信息，**必须实现**

```objc
/**
 获取当前的家庭，当前没有家庭的时候，返回nil。
 
 @return TuyaSmartHome
 */
- (TuyaSmartHome *)getCurrentHome;
```

#### TYOTAGeneralProtocol

进入设备面板时，提供检查设备固件更新的事件。实现如下方法用于检查固件升级：

```objc
/**
 检查设备固件更新，如果有更新会显示展示出固件更新提示
 
 @param deviceModel 需要检查固件升级的设备
 @param isManual 是否手动检测升级
  @param theme 主题色
 YES: 手动检测升级，检测时弹出loading框。当有固件新版本时(检测升级、强制升级、提醒升级)，显示OTA VC。
 NO: 自动检测升级, 检测时不弹出loading框。当有强制升级时、提醒升级时，弹出固件升级提示，点确定后显示OTA VC。
 */
- (void)checkFirmwareUpgrade:(TuyaSmartDeviceModel *)deviceModel isManual:(BOOL)isManual theme:(TYOTAControllerTheme)theme;
```

## 使用指南

### 注意事项

1. 使用任何接口之前，务必确认该设备在当前用户下。
2. 此接口，只适用于摄像机设备调用，即 deviceModel.category 为 “sp” 类型的设备。
3. 调用业务包逻辑前，要先实现 `TYSmartHomeDataProtocol` 中的协议方法`getCurrentHome`。
4. 接入此业务包后，必须同时也接入 `TuyaSmartCameraPanelBizBundle` 业务包，因为有些相关功能代码（例如摄像机相册面板代码等）在该业务包中。

Objective-C 

```objc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYSmartHomeDataProtocol.h>

- (void)initCurrentHome {
    // 注册要实现的协议
    [[TuyaSmartBizCore sharedInstance] registerService:@protocol(TYSmartHomeDataProtocol) withInstance:self];
}

// 实现对应的协议方法
- (TuyaSmartHome *)getCurrentHome {
    TuyaSmartHome *home = [TuyaSmartHome homeWithHomeId:@"当前家庭id"];
    return home;
}
```

Swift

```swift
import TuyaSmartDeviceKit

class TYActivatorTest: NSObject,TYSmartHomeDataProtocol{
  	
    func test() {
        TuyaSmartBizCore.sharedInstance().registerService(TYSmartHomeDataProtocol.self, withInstance: self)
    }
    
    func getCurrentHome() -> TuyaSmartHome! {
        let home = TuyaSmartHome.init(homeId: 111)
        return home
    }
    
}
```

### 注册 TYRNCameraProtocol 协议

**注意**

`TYRNCameraProtocol` 的协议方法，只有在需要实现自定义摄像机RN面板（返回自定义的摄像机面板）的情况下，才需要自己去注册和实现，默认 `TuyaSmartPanelBizBundle` 业务包内部会有对应的实现逻辑。

Objc

```objc
[[TuyaSmartBizCore sharedInstance] registerService:@protocol(TYRNCameraProtocol) withInstance:self];
```

Swift

```swift
TuyaSmartBizCore.sharedInstance().registerService(TYRNCameraProtocol.self, withInstance: self)
```

### 获取预览面板 (UIViewController)

摄像机RN预览面板，包括视频实时预览，清晰度切换，声音开关控制，截图，录制，对讲等功能。

Objc

```objc
id<TYRNCameraProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYRNCameraProtocol)];
UIViewController *vc = [impl cameraRNPanelViewControllerWithDeviceId:self.deviceModel.devId];
[self.navigationController pushViewController:vc animated:YES];
```

Swift

```swift
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYRNCameraProtocol.self) as? TYRNCameraProtocol
impl?.cameraRNPanelViewControllerWithDeviceId(withDeviceId: deviceModel.devId!) 
```

<span id="cameraSetting"></span>
# 摄像机设置面板业务包

## 功能概述

涂鸦智能 iOS IPC  设置业务包 （TuyaSmartCameraSettingBizBundle） 是基于 [Tuya Smart Camera SDK](<https://tuyainc.github.io/tuyasmart_camera_ios_sdk_doc/>) 开发的一系列摄像机常用设置等相关的面板。主要包括以下功能：

- 设备详情，基础设置，存储卡设置，增值服务，其它，重启等。

## 接入组件

在  ```Podfile``` 文件中加入以下代码：

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # 添加摄像机设置面板业务包
  pod 'TuyaSmartCameraSettingBizBundle'
end
```

然后在项目根目录下执行 ```pod update``` 命令，集成第三方库。

CocoaPods 的使用请参考：[CocoaPods Guides](https://guides.cocoapods.org/)

## 服务协议

摄像机设置业务包实现 `TYCameraSettingProtocol` 协议以提供服务，在 `TYModuleServices` 组件中查看 `TYCameraSettingProtocol.h` 协议文件内容为：

```objc
#import <UIKit/UIKit.h>

@protocol TYCameraSettingProtocol <NSObject>

/**
 获取摄像头设置面板
 @param params 摄像头设置页面相关的参数（例如：设置首页为 @{@"devId" : 设备devID, @"cameraSettingPanelIdentifier" : @"cameraSettingIndexIdentifier"} ）
 */
- (UIViewController *)settingViewControllerWithDeviceParams:(NSDictionary *)params;

@end
```

### 注意事项

1. 使用任何接口之前，务必确认该设备在当前用户下。
2. 只适用于摄像机设备调用，即 deviceModel.category 为 “sp” 类型的设备。
3. 接入此业务包后，默认就可以实现摄像机设置相关功能。如果用户想自行实现摄像头设置面板，注册下面对应协议，实现代理方法即可。

### 注册 TYCameraSettingProtocol 协议

**注意**

`TYCameraSettingProtocol` 的协议方法，只有在需要实现自定义摄像机设置面板（返回自定义的摄像机设置面板）的情况下，才需要自己去注册和实现，默认 `TuyaSmartCameraSettingBizBundle` 业务包内部会有对应的实现逻辑。

Objc

```objc
[[TuyaSmartBizCore sharedInstance] registerService:@protocol(TYCameraSettingProtocol) withInstance:self];
```

Swift

```swift
TuyaSmartBizCore.sharedInstance().registerService(TYCameraSettingProtocol.self, withInstance: self)
```