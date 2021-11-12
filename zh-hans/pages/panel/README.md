# 设备控制业务包

## 功能介绍

设备控制业务包是涂鸦智能设备控制面板的核心容器，在涂鸦智能 iOS Home SDK 的基础上，提供了设备控制面板的加载和控制的接口封装，加速应用开发过程。主要包括以下功能：

- 面板加载（加载多种设备类型，支持：WIFI、Zigbee、Mesh、BLE）
- 设备控制（支持单设备和群组的控制，不支持群组管理）
- 设备定时



## 接入组件

在工程的 `Podfile` 文件中添加设备控制业务包组件，并执行 `pod update` 命令

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # TuyaSmart SDK
  pod "TuyaSmartHomeKit"
  # 添加设备控制业务包
  pod 'TuyaSmartPanelBizBundle'
  # 若需要扫地机功能，请依赖扫地机相关插件
  # pod 'TuyaRNApi/Sweeper'
end
```

**注意**

设备控制业务包中封装了一系列 RN 接口供面板调用，其中会涉及到部分苹果隐私权限的声明。

- 如果接入的设备面板有使用相册相关的（例如：云相册），则需要在工程的 `info.plist` 中添加如下权限声明：

```
NSPhotoLibraryAddUsageDescription
```

- 如果接入的设备面板有使用照相机相关的（例如：云相册），则需要在工程的 `info.plist` 中添加如下权限声明：

```
NSCameraUsageDescription
```

- 如果接入的设备面板有使用位置信息相关的，则需要在工程的 `info.plist` 中添加如下权限声明：

```
NSLocationWhenInUseUsageDescription
```

- 如果接入的设备面板有使用到麦克风（例如：音乐灯面板），则需要在工程的 `info.plist` 中添加如下权限声明：

```
NSMicrophoneUsageDescription
```

- 如果接入的设备面板是蓝牙相关的，则需要在工程的 `info.plist` 中添加如下权限声明：

```
NSBluetoothAlwaysUsageDescription
NSBluetoothPeripheralUsageDescription
```



## 服务协议

### 提供服务

设备控制业务包实现 `TYPanelProtocol` 协议以提供服务，在 `TYModuleServices` 组件中查看 `TYPanelProtocol.h` 协议文件内容为：

```objc
@protocol TYPanelProtocol <NSObject>
NS_ASSUME_NONNULL_BEGIN

// 清除面板缓存
- (void)cleanPanelCache;


/**
 * 获取设备面板控制器
 *
 * @param deviceModel 设备模型
 * @param initialProps  自定义初始化参数，会以 'extraInfo' 为 key 设置进 RN 应用的 initialProps 中
 * @param contextProps  自定义面板上下文，会以 'extraInfo' 为 key 设置进 Panel Context 中
 * @param completionHandler 回调返回视图控制器
 */
- (void)getPanelViewControllerWithDeviceModel:(TuyaSmartDeviceModel *)deviceModel
                                 initialProps:(nullable NSDictionary *)initialProps
                                 contextProps:(nullable NSDictionary *)contextProps
                            completionHandler:(void (^ _Nullable)(__kindof UIViewController * _Nullable panelViewController, NSError * _Nullable error))completionHandler;

/**
 * 获取群组面板控制器
 *
 * @param groupModel 群组模型
 * @param initialProps  自定义初始化参数，会以 'extraInfo' 为 key 设置进 RN 应用的 initialProps 中
 * @param contextProps  自定义面板上下文，会以 'extraInfo' 为 key 设置进 Panel Context 中
 * @param completionHandler 回调返回视图控制器
 */
- (void)getPanelViewControllerWithGroupModel:(TuyaSmartGroupModel *)groupModel
                                initialProps:(nullable NSDictionary *)initialProps
                                contextProps:(nullable NSDictionary *)contextProps
                           completionHandler:(void (^ _Nullable)(__kindof UIViewController * _Nullable panelViewController, NSError * _Nullable error))completionHandler;

// RN版本号
- (NSString *_Nonnull)rnVersionForApp;

NS_ASSUME_NONNULL_END
@end
```



### 依赖服务

设备控制业务包主要功能为加载设备，针对不同设备会有不同的一些功能，为保证这些功能正常运行，会依赖如下几个协议： `TYSmartHomeDataProtocol`、 `TYDeviceDetailProtocol`、 `TYSettingsProtocol`、 `TYOTAGeneralProtocol`、 `TYGroupHandleProtocol`、 `TYRNCameraProtocol`、 `TYCameraProtocol`。

#### TYSmartHomeDataProtocol

提供加载设备面板所需的当前家庭信息，**必须实现**

```objc
/**
 获取当前的家庭，当前没有家庭的时候，返回nil。
 
 @return TuyaSmartHome
 */
- (TuyaSmartHome *)getCurrentHome;
```

####  TYDeviceDetailProtocol

设备面板界面右上角点击跳转事件

```objc
/**
 导航栏右边按钮点击事件，跳转到设备详情页

 @param device 设备
 @param group 群组，若有就传
 */
- (void)gotoDeviceDetailDetailViewControllerWithDevice:(TuyaSmartDeviceModel *)device group:(TuyaSmartGroupModel *)group;
```

#### TYSettingsProtocol

面板内触发下发 dp 控制设备时，提供可选开关供 App 发出音效。实现如下方法返回是否开启音效：

```objc
/**
 * 检查是否开启音效
 */
- (BOOL)soundEnabled;
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

#### TYGroupHandleProtocol

BLE Mesh 品类的设备，面板内有需要跳转到 Mesh 群组界面的事件，若需要跳转 mesh 群组，实现如下方法：

```objc
/**
跳转本地 mesh 群组

@available 1.0.0
@param params 
@param success 
@param failure 
*/
- (void)impl_jumpToMeshLocalGroup:(NSDictionary*)params success:(RCTResponseSenderBlock)success failure:(RCTResponseErrorBlock)failure ;
```

#### TYRNCameraProtocol

加载摄像头设备 RN 面板的跳转事件，若同时接入摄像头面板业务包 `TuyaSmartCameraRNPanelBizBundle`，则不需要实现；

```objc
/**
 获取摄像头RN面板
 @param devId 摄像头设备的devId
 */
- (UIViewController *)cameraRNPanelViewControllerWithDeviceId:(NSString *)devId;
```

#### TYCameraProtocol

加载摄像头设备原生面板的跳转事件，若同时接入摄像头面板业务包 `TuyaSmartCameraPanelBizBundle`，则不需要实现；

```objc
/**
 获取摄像头Native面板
 @param devId 摄像头设备的devId
 @param uiName 摄像头设备的uiName，不同的uiName对应不同版本的面板
 */
- (UIViewController *)viewControllerWithDeviceId:(NSString *)devId uiName:(NSString *)uiName;
```



## 使用指南

### 注意事项

1、任何接口调用之前，务必确认用户已登录

2、调用设备控制业务包逻辑前，要先实现 `TYSmartHomeDataProtocol` 中的协议方法`getCurrentHome`

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

### 清除面板缓存

Objective-C 

```objc
id<TYPanelProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYPanelProtocol)];
[impl cleanPanelCache];
```

Swift

```swift
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYPanelProtocol.self) as? TYPanelProtocol
impl?.cleanPanelCache()
```

### 打开面板

Objective-C 

```objc
id<TYPanelProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYPanelProtocol)];
// 获取面板视图控制器，自行跳转
if (deviceModel) {
    [impl getPanelViewControllerWithDeviceModel:deviceModel initialProps:nil contextProps:nil completionHandler:^(__kindof UIViewController * _Nullable panelViewController, NSError * _Nullable error) {
    }];
} else if (groupModel) {
    [impl getPanelViewControllerWithGroupModel:groupModel initialProps:nil contextProps:nil completionHandler:^(__kindof UIViewController * _Nullable panelViewController, NSError * _Nullable error) {
    }];
}
```

Swift

```swift
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYPanelProtocol.self) as? TYPanelProtocol
// 获取面板视图控制器，自行跳转
impl?.getPanelViewController(with: deviceModel, initialProps: nil, contextProps: nil, completionHandler: { (vc, err) in

})
impl?.getPanelViewController(with: groupModel, initialProps: nil, contextProps: nil, completionHandler: { (vc, err) in

})
```