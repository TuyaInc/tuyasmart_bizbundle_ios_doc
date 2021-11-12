# 场景业务包

## 功能介绍

业务功能包括涂鸦智能场景模块的「添加智能」和「编辑智能」的业务逻辑和UI界面。

智能场景分为「一键执行场景」和「自动化场景」，下文分别简称为「场景」和「自动化」。

场景是用户添加动作，手动触发；自动化是由用户设定条件，当条件触发后自动执行设定的动作。

涂鸦云支持用户根据实际生活场景，通过设置气象或设备条件，当条件满足时，让一个或多个设备执行相应的任务。


## 接入组件

在工程的 `Podfile` 文件中添加场景业务包组件，并执行 `pod update` 命令

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # 添加场景业务包
  pod 'TuyaSmartSceneBizBundle', '~> 3.22.0'
end
```
**注意**

场景业务包中天气条件会用到位置信息，其中会涉及到部分苹果隐私权限的声明。

- 需要在工程的 `info.plist` 中添加如下权限声明：

```
NSLocationWhenInUseUsageDescription
```


## 服务协议

### 提供服务

场景业务包实现 `TYSmartSceneProtocol` 协议和`TYSmartSceneBizProtocol`以提供服务。

在 `TYModuleServices` 组件中查看 `TYSmartSceneProtocol.h` 协议文件内容为：

```objc

@protocol TYSmartSceneProtocol <NSObject>
/**
 *	跳入新增场景页面，新增场景或者自动化
 *
 * @param callback 创建完成后结果回调
 */
- (void)addAutoScene:(void(^)(TuyaSmartSceneModel *secneModel, BOOL addSuccess))callback;
/**
 *	跳入编辑场景页面，编辑指定场景或者自动化。
 *  注意调用此方法前，需要调用TYSmartSceneBizProtocol的getSceneListWithHomeId：方法获取家庭下的场景列表，这样会生成场景缓存，之后才能正常跳入编辑页面。
 *
 * @param model 要进行编辑的场景model对象
 */
- (void)editScene:(TuyaSmartSceneModel *)model;

@end

```

在 `TYModuleServices` 组件中查看 `TYSmartSceneBizProtocol.h` 协议文件内容为：

```objc
/**
 * 获取场景列表，包括自动化和场景
 * 
 * @param  家庭Id 
 */
- (void)getSceneListWithHomeId:(long long)homeId withSuccess:(void(^)(NSArray <TuyaSmartSceneModel *> *scenes))success failure:(void(^)(NSError *error))failure;

```


### 依赖服务

场景业务包正常运行需要依赖  `TYSmartHomeDataProtocol` 这个协议提供的协议方法，调用业务包之前需要实现以下协议：

#### TYSmartHomeDataProtocol

提供场景组件所需的当前家庭信息

```objc
/**
 获取当前的家庭，当前没有家庭的时候，返回nil。
 
 @return TuyaSmartHome
 */
- (TuyaSmartHome *)getCurrentHome;
```
#### TYSmartHouseIndexProtocol
提供场景组件所需的管理员身份信息。如果非管理员也允许编辑场景，返回YES即可。

```objc
/**
 * 是否是当前家庭的管理员。
 *
 * @return YES代表是管理员
 */
- (BOOL)homeAdminValidation;
```


## 使用指南

### 注意事项

1. 任何接口调用之前，务必确认用户已登录。

2. 调用场景业务包逻辑前，要先实现 `TYSmartHomeDataProtocol` 中的协议方法`getCurrentHome`和`TYSmartHouseIndexProtocol `中的协议方法`homeAdminValidation`。

3. swift需要在桥接文件内引用业务包头文件。

   ```objc
   #import <TuyaSmartBizCore/TuyaSmartBizCore.h>
   #import <TYModuleServices/TYModuleServices.h>
   #import <TuyaSmartSceneKit/TuyaSmartSceneModel.h>
   ```

Objective-C 示例

```objc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYSmartHomeDataProtocol.h>
#import <TYModuleServices/TYSmartHouseIndexProtocol.h>
#import <TuyaSmartDeviceKit/TuyaSmartDeviceKit.h>
- (void)registerProtocol {
    // 注册要实现的协议
    [[TuyaSmartBizCore sharedInstance] registerService:@protocol(TYSmartHomeDataProtocol) withInstance:self];
    [[TuyaSmartBizCore sharedInstance] registerService:@protocol(TYSmartHouseIndexProtocol) withInstance:self];
}

// 实现对应的协议方法
- (TuyaSmartHome *)getCurrentHome {
    TuyaSmartHome *home = [TuyaSmartHome homeWithHomeId:@"当前家庭id"];
    return home;
}
- (BOOL)homeAdminValidation {
    //可根据实际用户身份返回，也可直接返回YES允许所有用户编辑
    return YES;
}
```



Swfit 示例


```swift

class TYSceneTest: NSObject,TYSmartHomeDataProtocol,TYSmartHouseIndexProtocol {
    func test() {
        TuyaSmartBizCore.sharedInstance().registerService(TYSmartHomeDataProtocol.self, withInstance: self);
        TuyaSmartBizCore.sharedInstance().registerService(TYSmartHouseIndexProtocol.self, withInstance: self);
    }
    
    func getCurrentHome() -> TuyaSmartHome! {
        let home = TuyaSmartHome.init(homeId: 111)
        return home
    }
    
    func homeAdminValidation() -> Bool {
        return true
    }
}
```



### 新增场景

Objective-C 示例

```objc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYSmartSceneProtocol.h>


- (void)gotoAddScene {
    id<TYSmartSceneProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYSmartSceneProtocol)];
    [impl addAutoScene:^(TuyaSmartSceneModel *secneModel, BOOL addSuccess) {
            
    }];
}
```



Swift 示例

```swift
func addScene() {
    let impl = TuyaSmartBizCore.sharedInstance().service(of: TYSmartSceneProtocol.self) as? TYSmartSceneProtocol
    impl?.addAutoScene({ (sceneModel, result) in
            
    })
}

```

### 编辑场景

Objective-C 示例

```objc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYSmartSceneProtocol.h>

- (void)gotoEditScene {
    id<TYSmartSceneProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYSmartSceneProtocol)];
    [impl editScene:(your sceneModel)];
}
```



Swift 示例

```swift
func editScene() {
    let impl = TuyaSmartBizCore.sharedInstance().service(of: TYSmartSceneProtocol.self) as? TYSmartSceneProtocol
    impl?.editScene(your scenemodel)
        
    }
}
```

### 获取场景列表

Objective-C 示例

```objc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYSmartSceneBizProtocol.h>

- (void)getSceneList {
    id<TYSmartSceneBizProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYSmartSceneBizProtocol)];
    [impl getSceneListWithHomeId:'your homeId' withSuccess:^(NSArray<TuyaSmartSceneModel *> * _Nonnull scenes) {
            
    } failure:^(NSError * _Nonnull error) {
            
    }];
}
```



Swift 示例

```swift
func getSceneList() {
	let impl = TuyaSmartBizCore.sharedInstance().service(of: TYSmartSceneBizProtocol.self) as? TYSmartSceneBizProtocol
   	impl?.getSceneList(withHomeId: 111, withSuccess: { (sceneArr) in
            
   	}, failure: { (error) in
            
  	})
}

```

### 通知中心

**kNotificationSmartSceneListUpdate**

通知发送时机：

1. 添加场景成功
2. 编辑场景成功
3. 删除场景成功（业务包默认场景类型为TuyaSmartSceneRecommendTypeNone和TuyaSmartSceneCollectionTypeNone）

**kNotificationSmartSceneSaved**

通知发送时机：

1. 添加场景成功
2. 编辑场景成功

**kNotificationSmartSceneRecomDeleted**

通知发送时机：

1. 删除场景成功（推荐场景或者收藏场景）
