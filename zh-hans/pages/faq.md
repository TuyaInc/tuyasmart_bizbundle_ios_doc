# FAQ

## 设备控制业务包

**1. 为什么调用 Push 方式跳转面板之后，其他视图控制器的导航栏样式变了？**

设备控制业务包本质是提供一个视图控制器来加载 React-Native 渲染的界面，所以该视图控制器内部将导航栏进行了相关的隐藏处理，主要是改变了透明度。因此在使用 push 方式进入面板控制器之后，pop 回自身控制器时，需要将样式还原（主要原因是 iOS 提供的 push 和 pop 动画是使用 NavigationController 来管理的，其中使用的 NavigationBar 是唯一的；在 NavigationController 的 Stack 存储结构下，每当 Stack 中的 ViewController 修改了导航栏，势必会影响其他 ViewController 展示的效果）。以下建议两种样式还原方式：

方式一：

```objective-c
// 1. 导入头文件
#import <TYNavigationController/TYNavigationTopBarProtocol.h>
// 2. 显示导航栏
self.ty_topBarHidden = NO;
self.ty_topBarAlpha = 1.0;
// 3. 还原导航栏样式(例如：修改背景色)
self.ty_topBarBackgroundImage = <#Image#>;
// Or
self.ty_topBarBackgroundColor = <#Color#>;
```

方式二：

```objective-c
// 1. 导入头文件
#import <TYNavigationController/TYNavigationCallbackProtocol.h>
// 2. 实现 TYNavigationCallbackProtocol 提供的代理方法
- (void)ty_naviTransitioning:(id<UIViewControllerTransitionCoordinatorContext>)context {
    // 3. 显示导航栏
    [self.navigationController setNavigationBarHidden:false animated:true];
    // 4. 还原导航栏样式(例如：修改背景色)
    [self.navigationController.navigationBar setBarTintColor:<#Color#>];
  	// Or
    [self.navigationController.navigationBar setBackgroundImage:<#Image#> forBarMetrics:UIBarMetricsDefault];
}
```

**2. 依赖插件的时候，`pod update` 报错？**

工程在使用 cocoapods 集成业务包时，由于 pod 的逻辑，Podfile 中没有指明每个 Pod 库的版本时，默认会拉取最新的正式版本号（x.y.z，其中 x、y、z 均为数字）。若需要依赖设备控制业务包的扩展功能时，即需要依赖相关插件，由于插件的版本号属于 pre-release，因此，需要在 Podfile 中指明设备控制业务包的所在基线的最新日期标明的版本号（[点击查看业务包版本号](../pages/versions.md)）。

**3.进入面板时，弹框提示【当前版本不支持该设备，请升级 App】？**

在调用进入设备面板业务包的接口前，确认已提前获取过设备信息，即成功调用过 Home SDK 中的该方法：

```objective-c
/**
 *  After init home, need to get home details
 *  初始化 home 对象之后需要获取家庭的详情，homeModel,roomList,deviceList,groupList 才有数据
 *
 *  @param success     Success block
 *  @param failure     Failure block
 */
- (void)getHomeDetailWithSuccess:(void (^)(TuyaSmartHomeModel *homeModel))success
                         failure:(TYFailureError)failure;
```

若后续仍然出现该提示，有以下四种可能：

1. 当前 SDK 支持的MQTT 通信协议低于硬件的通信协议，即 `deviceModel.pv > TUYA_CURRENT_GW_PROTOCOL_VERSION`
2. 当前 SDK 支持的局域网通信协议低于硬件的通信协议，即 `deviceModel.lpv > TUYA_CURRENT_LAN_PROTOCOL_VERSION`
3. 面板 UI 包不支持当前的版本（`deviceModel.rnFind` 为 False）
4. 面板未找到合适的视图控制器以展示 (`deviceModel.uiType`)