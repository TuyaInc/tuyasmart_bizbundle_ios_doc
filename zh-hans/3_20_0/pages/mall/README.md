# H5 商城业务包

## 介绍

H5 商城业务包提供了承载 「App 商城」的 iOS 容器，让您的 App 具备强大的商城能力，让移动端流量通过商城变现。

> 「App 商城」为涂鸦平台提供的一项增值服务，详情可以在涂鸦智能平台 [增值服务](https://www.tuya.com/vas/) 中搜索 「App 商场」了解

## 接入组件

在工程的 `Podfile` 文件中添加商城业务包组件，并执行 `pod update` 命令

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # TuyaSmart SDK
  pod "TuyaSmartHomeKit"
  # 添加 H5 商城业务包
  pod 'TuyaSmartMallBizBundle', '~> 3.20.0'
end
```

## 服务协议

### 提供服务

H5 商城业务包实现 `TYMallProtocol` 协议以提供服务，在 `TYModuleServices` 组件中查看 `TYMallProtocol.h` 协议文件内容为：

```oc
#import <Foundation/Foundation.h>

typedef NS_ENUM(NSUInteger, TYMallPageType) {
    TYMallPageTypeHome,      // Mall home page
    TYMallPageTypeOrders,    // Mall orders page
};

@protocol TYMallProtocol <NSObject>

/**
 * Check if mall enable for current logged user.
 * You should check this every time after logged user changed.
 */
- (void)checkIfMallEnableForCurrentUser:(void(^)(BOOL enable, NSError *error))callback;

/**
 * Request special mall page with `TYMallPageType`
 * You should replace mall page every time after logged user changed.
 * @param pageType Mall page type
 */
- (void)requestMallPage:(TYMallPageType)pageType completionBlock:(void(^)(__kindof UIViewController *page, NSError *error))callback;

@end
```

### 依赖服务
无

## 使用指南

### 注意事项

1. 使用任何接口之前，务必确认用户已登录
2. 获取商城页面之前，需要首先确认当前用户所注册的服务区是否开通商城服务
3. 登录用户发生变化时，务必重新判断商城可用状态并重新获取商城页面
4. 获取商城页面为 `UIViewController`，务必使用 `UINavigationController` 进行 `push` 或 `present` 展示

> 商城与用户信息强相关
> 商城页面依赖导航控制器，且会设置导航栏内容，因此需要导航控制器进行包装

### 查询可用状态

Objective-C 示例

```oc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYMallProtocol.h>

id<TYMallProtocol> mallImpl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYMallProtocol)];
[mallImpl checkIfMallEnableForCurrentUser:^(BOOL enable, NSError *error) {
  if (error) {
    // 查询失败时返回 error
    NSLog(@"%@",error);
  } else {
    // enable 为 true 则商城可用，否则不可用
    NSLog(@"%@",@(enable));
  }
}];
```

Swift 示例

```swift
let mallImpl = TuyaSmartBizCore.sharedInstance().service(of: TYMallProtocol.self)
(mallImpl as? TYMallProtocol)?.checkIfMallEnable(forCurrentUser: { (enable, error) in
    if let e = error {
        print("\(e)")
        return
    }
    print("\(enable)")
})
```

### 获取商城页面 (UIViewController)

目前提供一下两个页面：
- 商城首页 (TYMallPageTypeHome)
- 商城订单页 (TYMallPageTypeOrders)

接入时可以根据需要获取对应页面来展示

Objective-C 示例

```oc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYMallProtocol.h>

id<TYMallProtocol> mallImpl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYMallProtocol)];
[impl requestMallPage:TYMallPageTypeHome completionBlock:^(__kindof UIViewController *page, NSError *error) {
    UINavigationController *nav = [[UINavigationController alloc]initWithRootViewController:page];
    [self presentViewController:nav animated:YES completion:nil];
}];
```

Swift 示例

```swift
let mallImpl = TuyaSmartBizCore.sharedInstance().service(of: TYMallProtocol.self)
(mallImpl as? TYMallProtocol)?.request(.home, completionBlock: { (mallVc, error) in
    if let e = error {
        print("\(e)")
        return
    }
    // push
    yourNaviController.pushViewController(mallVc!, animated: true)
    // present
    let naviVc = UINavigationController.init(rootViewController: mallVc!)
    yourViewController.present(naviVc, animated: true, completion: nil)
})
```



