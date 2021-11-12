# 消息中心业务包

## 功能介绍

消息中心业务包提供涂鸦APP消息中心业务逻辑。业务功能主要涵盖各类消息的推送历史记录，主要包括告警、家庭、通知三个消息大类。其中告警包含设备告警，场景自动化等执行记录。
消息中心设置页可以启用或关闭各类消息推送，对于告警类消息可以支持对设备添加免打扰时段。



## 接入组件

在工程的 `Podfile` 文件中添加消息中心业务包组件，并执行 `pod update` 命令

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # 添加消息中心业务包
  pod 'TuyaSmartMessageBizBundle'
end
```



## 服务协议

### 提供服务

消息中心业务包实现 `TYMessageCenterProtocol` 协议以提供对外服务，在 `TYModuleServices` 组件中查看 `TYMessageCenterProtocol.h` 协议文件内容为：

```objective-c
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@protocol TYMessageCenterProtocol <NSObject>

/// push消息中心页面 push message center vc
/// @param 动画 animated
- (void)gotoMessageCenterViewControllerWithAnimated:(BOOL)animated;

@end

NS_ASSUME_NONNULL_END
```



### 依赖服务

消息中心业务包正常运行需要依赖  `TYSmartHomeDataProtocol` 这个协议提供的协议方法，调用业务包之前需要实现以下协议

#### TYSmartHomeDataProtocol

提供消息中心所需当前家庭信息

```objective-c
/**
 获取当前的家庭，当前没有家庭的时候，返回nil。
 
 @return TuyaSmartHome
 */
- (TuyaSmartHome *)getCurrentHome;
```



## 使用指南

### 注意事项

1. 在使用任何接口之前，务必确认用户已登录

2. 登录用户发生变化时，务必重新判断消息中心可用状态并重新获取消息中心页面

3. 调用业务包逻辑前，要先实现 `TYSmartHomeDataProtocol` 中的协议方法`getCurrentHome`

Objective-C 示例

```objective-c
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

Swfit 示例

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




## 进入消息中心

Objective-C 示例

```objective-c
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYMessageCenterProtocol.h>


- (void)gotoDeviceConfig {
    id<TYMessageCenterProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYMessageCenterProtocol)];
    [impl gotoMessageCenterViewControllerWithAnimated:YES];
}
```

Swfit 示例

``` swift
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYMessageCenterProtocol.self) as? TYMessageCenterProtocol
impl?.gotoMessageCenterViewController(animated: true)
```
备注：因业务包开放能力及功能组件依赖原因，消息中心部分告警信息链接点击响应暂时不支持。