#设备详情业务包



##功能介绍

- 设备信息编辑（设备头像、设备所在房间、名称）
- 设备信息查询 （ID、信号等）
- 备用网络设置
- 离线提醒功能
- 删除设备

- 常见问题与反馈 （需要接入帮助中心业务包）
- 检查固件升级（需要接入OTA业务包）



##接入组件

### pod集成
在工程的 `Podfile` 文件中添加业务包组件，并执行 `pod update` 命令

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
 # 添加设备详情业务包
  pod 'TuyaSmartDeviceDetailBizBundle', '~> 3.22.0'
end
```
### 权限声明

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

### 配置文件
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


### 服务协议

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

#### 1. OTA功能
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


###6.自定义子功能
####1.配置configList.json
 在configList.json文件的deviceDetail插入自定义type。**注意type值必须以“c_”开头**
 如例：插入"c_test_insert"、"c_test_async_insert"
 ```
 {"deviceDetail": [
     {
            "type":"c_test_insert"
        },
        {
            "type":"c_test_async_insert"
        },
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
  ],}
  ```
  ####2.返回item数据
  #####接口说明：同步返回item数据
   ```
  /// 设置-》同步处理item插入的回调。insertDevMenuItemBlock会在设备详情刷新时候回调
-(void)insertDevMenuItem:(InsertDevMenuItemBlock) insertDevMenuItemBlock;
 ```
  ```
  
//@param type configList.json里自己添加的type
//@param device  设备模型
//@param group   群组模型。 根据group是否为nil，来判断设备还是群组
//@return 遵守TYDeviceDetailCustomMenuModel协议的对象。返回nil，该type的item则不会显示
typedef id<TYDeviceDetailCustomMenuModel> _Nullable (^InsertDevMenuItemBlock)(NSString*  _Nonnull type,
                                                                           TuyaSmartDeviceModel* _Nullable device,
                                                                           TuyaSmartGroupModel* _Nullable group);
  ```
  
  #####接口说明：异步回调item数据
  
   ``` 
  /// 设置-》异步处理item插入的回调，insertDevMenuItemAsyncBlock会在设备详情刷新时候回调
-(void)insertDevMenuItemAsync:(InsertDevMenuItemAsyncBlock) insertDevMenuItemAsyncBlock;
  ```
 ``` 
 //异步处理item插入，当异步操作结束以后，调用complete(id<TYDeviceDetailCustomMenuMode>),回调数据给设备详情，并进行刷新列表。
typedef void (^InsertDevMenuItemAsyncBlock)(NSString* _Nonnull type,
                                            TuyaSmartDeviceModel* _Nullable device,
                                            TuyaSmartGroupModel* _Nullable group,
                                            InsertDevMenuItemComplete _Nonnull complete);
``` 
  
  #####示例代码
  
  **第一步：先新建一个Model类，遵守TYDeviceDetailCustomMenuModel协议**
  
  **oc**
   ```
  //自定义一个类遵守TYDeviceDetailCustomMenuModel协议
 @interface CustomMenuModel : NSObject<TYDeviceDetailCustomMenuModel>
///标题
@property (nonatomic,copy) NSString *title;
///子标题
@property (nonatomic,copy) NSString *detail;
@end

@implementation CustomMenuModel
@end
```
**swift**
```
class CustomMenuModel: NSObject, TYDeviceDetailCustomMenuModel{
    var title : String?
    var detail : String?
}
  ```
**第二步：设置数据回调的block**

 **oc**
   ```
   id<TYDeviceDetailProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYDeviceDetailProtocol)];
        
        [impl insertDevMenuItem:^ id<TYDeviceDetailCustomMenuModel> (NSString * _Nonnull type, TuyaSmartDeviceModel * _Nonnull device, TuyaSmartGroupModel * _Nonnull group) {
            if ([type isEqualToString:@"c_test_insert"]) {
                CustomMenuModel *model = [CustomMenuModel new];
                if (group) { //根据group是否为nil，来判断设备还是群组
                    model.title = type;
                    model.detail = @"group";
                }else{
                    model.title = type;
                    model.detail = @"device";
                }
                return model;
            }
            return nil;
        }];
        
        [impl insertDevMenuItemAsync:^(NSString * _Nonnull type, TuyaSmartDeviceModel * _Nonnull device, TuyaSmartGroupModel * _Nonnull group, InsertDevMenuItemComplete  _Nonnull complete) {
            if ([type isEqualToString:@"c_test_async_insert"]) {
            //耗时操作
            dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
                CustomMenuModel *model = [CustomMenuModel new];
                if (group) { //根据group是否为nil，来判断设备还是群组
                    model.title = type;
                    model.detail = @"group";
                }else{
                    model.title = type;
                    model.detail = @"device";
                }
                complete(model);
            });
            }

        }];
 ```
 **swift**
  ```
   let impl = TuyaSmartBizCore.sharedInstance().service(of: TYDeviceDetailProtocol.self) as? TYDeviceDetailProtocol
   
    impl?.insertDevMenuItem({ (type, deviceModel, groupModel) -> TYDeviceDetailCustomMenuModel? in
            if type == "c_test_insert" {//根据group是否为nil，来判断设备还是群组
                let model = CustomMenuModel.init()
                if groupModel != nil {
                    model.title = type
                    model.detail = "group"
                }else{
                    model.title = type
                    model.detail = "device"
                }
                return model;
            }
            return nil;
        })

impl?.insertDevMenuItemAsync({ (type, deviceModel, groupModel, complete) in
            if type == "c_test_async_insert" {
                //耗时操作
                DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
                    let model = CustomMenuModel.init()
                    if groupModel != nil {
                        model.title = type
                        model.detail = "group"
                    }else{
                        model.title = type
                        model.detail = "device"
                    }
                    complete(model);
                }
            }
        });
  ```
 ####3.插入item点击事件处理
   #####oc示例代码
  ```
      id<TYDeviceDetailProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYDeviceDetailProtocol)];
        [impl clickMenuItem: ^(NSString * _Nonnull type, TuyaSmartDeviceModel * _Nonnull device, TuyaSmartGroupModel * _Nonnull group) {
            NSLog(@"clickItem:  type:%@",type);
        }];
```
 
   #####swift示例代码

```
 let impl = TuyaSmartBizCore.sharedInstance().service(of: TYDeviceDetailProtocol.self) as? TYDeviceDetailProtocol

  impl?.clickMenuItem({ (type, deviceModel, groupModel) in
            print("clickItem:  type:"+type);
    })
 ```