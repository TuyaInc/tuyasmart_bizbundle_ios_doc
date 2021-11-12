# 配网业务包

## 功能介绍

业务功能涵盖了目前涂鸦智能的 Wi-Fi 设备、ZigBee 设备、蓝牙设备、支持二维码扫码的设备（例如 GPRS & NB-IoT 设备）等不同类型的设备配网前置操作引导和具体入网激活实现

### 1、Wi-Fi 设备配网

支持 Wi-Fi 智能设备入网连接云服务，Wi-Fi 设备配网主要有 EZ 模式和 AP 模式两种，其中 IPC 设备支持扫二维码方式配网

| 名词             | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| EZ 模式          | 又称快连模式，App 把配网数据包打包到802.11数据包的指定区域中，发送到周围环境；智能设备的 WiFi 模块处于混杂模式下，监听捕获网络中的所有报文，按照约定的协议数据格式解析出 App 发出配网信息包。 |
| AP 模式          | 又称热点模式，手机作为 STA 连接智能设备的热点，双方建立一个 Socket 连接通过约定端口交互数据。 |
| IPC 设备扫码配网 | 摄像头设备通过扫描 App 上的二维码获取配网数据信息            |

### 2、ZigBee 设备配网

支持 ZigBee 网关和子设备配网

| 名词        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| ZigBee 网关 | 融合 ZigBee 网络中协调器和 Wi-Fi 功能的设备，负责 ZigBee 网络的组建及数据信息存储。 |
| 子设备      | ZigBee 网络中的路由或者终端设备，负责数据转发或者终端控制响应。 |

### 3、蓝牙设备配网

涂鸦蓝牙有三条技术线路，主要包括 SingleBLE、SigMesh、TuyaMesh 以及双模设备

| 名词      | 说明                                                     |
| --------- | -------------------------------------------------------- |
| SingleBLE | 通过蓝牙与手机一对一相连的蓝牙单点设备                   |
| SigMesh   | 采用蓝牙技术联盟发布的蓝牙拓扑通信                       |
| TuyaMesh  | 采用涂鸦自研的蓝牙拓扑通信                               |
| 双模设备  | 支持多协议的设备，即同时具备 Wi-Fi 能力和 BLE 能力的设备 |

### 4、扫码配网设备

该类设备上电后即连接了涂鸦云服务，APP 通过扫描设备上的二维码（必须是涂鸦云服务支持的二维码规则，支持固件具体接入方式可咨询涂鸦科技相关商务及项目经理）使能设备去涂鸦云激活绑定

| 名词        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| GPRS 设备   | 采用 GPRS 通信技术接入网络连接云服务的智能设备               |
| NB-IOT 设备 | 采用窄带物联网( NarrowBand-Internet of Things )技术的智能设备 |

### 5、自动发现配网

融合涂鸦智能通用配网技术实现，为用户提供一套快捷配网的功能



## 接入组件

在工程的 `Podfile` 文件中添加配网业务包组件，并执行 `pod update` 命令

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # 添加配网业务包
  pod 'TuyaSmartActivatorBizBundle'
end
```



**注意**

Wi-Fi  设备配网过程需要获取手机当前连接 Wi-Fi  名称，需要项目开启地位权限来获取 Wi-Fi  的名称，在 info.plist 中添加如下权限声明，创建 `CLLocationManager` 示例，并调用 `requestWhenInUseAuthorization` 方法。

```
NSLocationAlwaysAndWhenInUseUsageDescription
NSLocationAlwaysUsageDescription
NSLocationWhenInUseUsageDescription
```

二维码扫码功能需要系统相机权限，需要在 info.plist 中添加以下声明

```
NSCameraUsageDescription
```



## 自定义配置

### 1、蓝牙配网功能

配网业务包支持 Wi-Fi  、蓝牙等类型的设备配网，其中蓝牙配网为可选项，如当前 App 不需要蓝牙配网功能 ，只需要将自定义 `ty_custom_config.json` 中的 `needBle` 属性设置为 false 即可

如果需要蓝牙配网功能，首先需要在项目的 info.Plist 文件中添加蓝牙权限的声明，设置 `ty_custom_config.json` 中的 `needBle` 属性设置为 true，然后需要在项目中添加以下依赖：

**系统权限**

```
NSBluetoothAlwaysUsageDescription
NSBluetoothPeripheralUsageDescription
```

**配置项**

```json
{
    "config": {
        "appId": 123,    
        "tyAppKey": "xxxxxxxxxxxx", 
        "appScheme": "tuyaSmart",
        "hotspotPrefixs": ["SmartLife"], 
        "needBle": true //设置为 true 则表示支持蓝牙设备的配网
    },
   "colors":{
        "themeColor": "#FF5A28", 
    }
}
```

**依赖**

```ruby
  pod 'TYBLEInterfaceImpl'
  pod 'TYBLEMeshInterfaceImpl'
  pod 'TuyaSmartBLEKit'
  pod 'TuyaSmartBLEMeshKit'
```

### 2、设备热点名称设置

涂鸦设备热点前缀默认为 `SmartLife` , 若当前设备的热点前缀名称已修改，则需要在  `ty_custom_config.json` 文件中设置 `hotspotPrefixs` 属性，设置当前设备的热点前缀

```json
{
    "config": {
        "appId": 123,    
        "tyAppKey": "xxxxxxxxxxxx", 
        "appScheme": "tuyaSmart",
        "hotspotPrefixs": ["SL"], // 修改支持的设备热点前缀为 SL
        "needBle": true
    },
   "colors":{
        "themeColor": "#FF5A28", 
    }
}
```



## 服务协议

### 提供服务

配网业务包实现 `TYActivatorProtocol` 协议以提供服务，在 `TYModuleServices` 组件中查看 `TYActivatorProtocol.h` 协议文件内容为：

```objc

#ifndef TYActivatorProtocol_h
#define TYActivatorProtocol_h

typedef NS_ENUM(NSUInteger, TYActivatorCompletionNode) {
    TYActivatorCompletionNodeNormal
};

@class TuyaSmartHome;

@protocol TYActivatorProtocol <NSObject>

/**
 * Start config
 * Goto device config list view 
 */
- (void)gotoCategoryViewController;


/**
 *  Obtain device information after each device connection
 *  @param node completion node, default TYActivatorCompletionNodeNormal
 *  @param custionJump default false, set true for process not need to jump to de device panel
 */
- (void)activatorCompletion:(TYActivatorCompletionNode)node customJump:(BOOL)customJump completionBlock:(void (^)(NSArray * _Nullable deviceList))callback;


@end
#endif /* TYActivatorProtocol_h */

```

若需要自定义配网品类/产品列表返回，需要实现 `TYActivatorExternalExtensionProtocol` 提供的协议方法

**TYActivatorExternalExtensionProtocol**

```objc
/**
 *  Back action form category View Controller
 *  Need to implement when additional operations are needed
 */
- (BOOL)categoryViewControllerCustomBackAction;
```



### 依赖服务

配网业务包正常运行需要依赖  `TYSmartHomeDataProtocol` 这个协议提供的协议方法，调用业务包之前需要实现以下协议

#### TYSmartHomeDataProtocol

提供配网所需当前家庭信息

```objc
/**
 获取当前的家庭，当前没有家庭的时候，返回nil。
 
 @return TuyaSmartHome
 */
- (TuyaSmartHome *)getCurrentHome;
```



## 使用指南

### 注意事项

1、任何接口调用之前，务必确认用户已登录

2、调用配网业务包逻辑前，要先实现 `TYSmartHomeDataProtocol` 中的协议方法`getCurrentHome`

Objective-C 示例

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



Swift 示例

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



### 进入配网

Objective-C 示例

```objc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYActivatorProtocol.h>


- (void)gotoDeviceConfig {
    id<TYActivatorProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYActivatorProtocol)];
    [impl gotoCategoryViewController];
  
    // 获取配网结果
    [impl activatorCompletion:TYActivatorCompletionNodeNormal customJump:NO completionBlock:^(NSArray * _Nullable deviceList) {
        NSLog(@"deviceList: %@",deviceList);
    }];
}
```



Swift 示例

```swift
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYActivatorProtocol.self) as? TYActivatorProtocol
impl?.gotoCategoryViewController()

impl?.activatorCompletion(TYActivatorCompletionNodeNormal, customJump: false, completionBlock: { (evIdList:[Any]?) in
            print(devIdList ?? [])
        })
```



### 自定义配网品类列表返回

Objective-C 示例

```objc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYActivatorExternalExtensionProtocol.h>


- (void)initCurrentHome {
    // 注册要实现的协议
    [[TuyaSmartBizCore sharedInstance] registerService:@protocol(TYActivatorExternalExtensionProtocol) withInstance:self];
}

// 实现对应的协议方法
- (BOOL)categoryViewControllerCustomBackAction {
    [self.navigationController popToRootViewControllerAnimated:YES];
    return YES;
}
```



Swift 示例  

```swift

class TYActivatorTest: NSObject,TYActivatorExternalExtensionProtocol{

    
    func test() {
 TuyaSmartBizCore.sharedInstance().registerService(TYActivatorExternalExtensionProtocol.self, withInstance: self)
    }
    
    func categoryViewControllerCustomBackAction() -> Bool {
        self.navigationController?.popToRootViewController(animated: true)
        return true;
    }
    
}
```





## 更新记录

- 2020.7.2 添加自定义配网品类/产品列表返回方法
