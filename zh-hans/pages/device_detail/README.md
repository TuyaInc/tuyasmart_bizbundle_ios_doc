#设备详情业务包

------------
##功能介绍

- 设备信息编辑（设备头像、设备所在房间、名称）
- 设备信息查询 （ID、信号等）
- 备用网络设置
- 离线提醒功能
- 删除设备

- 常见问题与反馈 （需要接入帮助中心业务包）
- 检查固件升级（需要接入OTA业务包）

------------

##接入组件

### 1.pod集成
在工程的 `Podfile` 文件中添加业务包组件，并执行 `pod update` 命令

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
 # 添加设备详情业务包
  pod 'TuyaSmartDeviceDetailBizBundle', '~> 3.22.0'
end
```
### 2.权限声明

设备业务包设备头像上传，使用相册和相机，其中会涉及到部分苹果隐私权限的声明。

- 需要在工程的 `info.plist` 中添加如下权限声明：
```
<!-- 相册 -->   
<key>NSPhotoLibraryUsageDescription</key>   
<string>App需要您的同意,才能访问相册</string>   
<!-- 相机 -->   
<key>NSCameraUsageDescription</key>   
<string>App需要您的同意,才能访问相机</string>   
```

### 3.配置文件
1.主工程中新建**configList.json**（已经存在该文件，就不用再新建）

2.在configList.json中添加
key：**deviceDetail** 
value: 下面的数组
```js
    [
        {
            "type":"header"
        },
        {
            "type": "device_info"
        },
        {
            "type": "net_setting"
        },
        {
            "type":"section_off_line_warn"
        },
        {
            "type":"off_line_warn"
        },
        {
            "type":"section_other"
        },
        {
            "type":"help_and_feedback"
        },
        {
          "type":"check_device_network"
         },
        {
            "type":"check_firmware_update"
        },
        {
            "type":"empty",
            "height":16
        },
        {
            "type":"footer"
        }
    ]
```
3.configList.json配置文件内容大致如下
```js
 {
  "deviceDetail": [{"type":"header"  } ],
  "其他配置页面key":[ ]
  }
```

4.自定义设备详情
注意deviceDetail数组item的顺序影响设备详情页item展示的顺序。如过移除item，则相应也会移除设备详情页子功能。


deviceDetail type | 功能点
---|---
header | 查看修改设备图标，名称，位置
device_info | 设备信息
net_setting | 设备备用网络
off_line_warn | 设备离线提醒
section_other | 分区头，无实际功能
help_and_feedback | 常见问题与反馈，需要集成常见问题与反馈业务包
check_firmware_update | 检查固件升级，需要集成固件升级业务包
empty | 空view，无实际功能
footer | 移除设备

------------


### 4.服务协议

#### 依赖服务（需要实现）
#####1.TYSmartHomeDataProtocol

```
/**
 要使用该API获取当前家庭，请务必在更新当前家庭id的时候使用该协议的’updateCurrentHomeId:‘Api。
 获取当前的家庭，当前没有家庭的时候，返回nil。
 
 @return TuyaSmartHome
 */
- (TuyaSmartHome *)getCurrentHome;
```

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

####可选功能实现

#### 1.  OTA功能
 #####**需要接入**：[接入OTA业务包](../ota/README.md  "接入OTA业务包")

#### 2.  常见问题与反馈功能
 #####**需要接入**：[接入帮助中心业务包](../faq/README.md  "接入帮助中心业务包")




###5.提供服务（方法调用）

 方法      |  参数  | 
 -------- | :-----------: 
 跳转到网络检测页     | 设备 id    
跳转到设备详情页，以 push 方式     |TuyaSmartDeviceModel/TuyaSmartGroupModel  

####oc示例代码
```
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYDeviceDetailProtocol.h>

 id<TYDeviceDetailProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYDeviceDetailProtocol)];
 
 //跳转到设备网络检测页
 [impl gotoDeviceDetailNetworkViewControllerWithDeviceId:@"设备 id"];
 
 
  // 跳转到设备详情页，以 push 方式
  
  //如果是设备，new TuyaSmartDevice
 TuyaSmartDevice * device = [TuyaSmartDevice deviceWithDeviceId:@"设备 id"]
  [impl gotoDeviceDetailDetailViewControllerWithDevice: device.deviceModel group: nil];
  //如果是群组，new TuyaSmartGroup
   TuyaSmartGroup * group = [TuyaSmartGroup groupWithGroupId:@"群组 id"]
   [impl gotoDeviceDetailDetailViewControllerWithDevice: nil group: group.deviceModel];


```

####swift示例代码
```
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYDeviceDetailProtocol.self) as? TYDeviceDetailProtocol
//跳转到设备网络检测页
impl?.gotoDeviceDetailNetworkViewController(withDeviceId: "设备 id")

  // 跳转到设备详情页，以 push 方式

  //如果是设备，new TuyaSmartDevice
  let device = TuyaSmartDevice.init(deviceId: "vdevo160888044089994")
  impl?.gotoDeviceDetailDetailViewController(withDevice: device?.deviceModel, group: nil)
  //如果是群组，new TuyaSmartGroup
  let group = TuyaSmartGroup.init(groupId: "群组 id")
  impl?.gotoDeviceDetailDetailViewController(withDevice: nil, group: group?.groupModel)


```