# 云存储服务业务包

涂鸦智能摄像机提供云存储视频服务，通过此业务包可以开通云存储服务。开同云存储服务后，可以通过 [IPC SDK](https://tuyainc.github.io/tuyasmart_home_ios_sdk_doc/zh-hans/resource/Camera.html) 查看和播放云存储视频。

## 接入组件

在工程的 `Podfile` 文件中添加云存储服务业务包组件，并执行 `pod update` 命令

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # 添加云存储服务业务包
  pod 'TuyaSmartCloudServiceBizBundle'
end
```

## 服务协议

云存储服务业务包实现 `TYCameraCloudServiceProtocol` 协议以提供服务，在 `TYModuleServices` 组件中查看 `TYCameraCloudServiceProtocol.h` 协议文件内容为：

```objc
#import <Foundation/Foundation.h>
@class TuyaSmartDeviceModel;
@protocol TYCameraCloudServiceProtocol <NSObject>

/**
* Request cloud service product page
*
* @param deviceModel which device that want to activate cloud services
*/
- (void)requestCloudServicePageWithDevice:(TuyaSmartDeviceModel *)deviceModel completionBlock:(void(^)(__kindof UIViewController *page, NSError *error))callback;

@end

```

### 依赖服务

无

## 使用指南

### 注意事项

1. 使用任何接口之前，务必确认用户已登录
2. 云存储服务与设备一一对应，在获取云存储服务页面时，需要传入对应设备的 `TuyaSmartDeviceModel`
3. 获取云存储服务页面为 `UIViewController`，务必使用 `UINavigationController` 进行 `push` 或 `present` 展示

> 云存储服务与用户信息强相关
> 云存储服务购买页面依赖导航控制器，且会设置导航栏内容，因此需要导航控制器进行包装

### 获取云存储服务购买页面 (UIViewController)

Objc

```objc
id<TYCameraCloudServiceProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYCameraCloudServiceProtocol)];
[impl requestCloudServicePageWithDevice:self.deviceModel completionBlock:^(__kindof UIViewController *page, NSError *error) {
    if (page) {
        [self.navigationController pushViewController:page animated:YES];
    }
}];
```

Swift

```swift
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYCameraCloudServiceProtocol.self)
(impl as? TYCameraCloudServiceProtocol)?.requestCloudServicePage(deviceModel, completionBlock: { (page, error) in
		guard let cloudServiceVc = page {
    		print("\(error!)")
      	return
    }                                                                                                
    yourNaviController.pushViewController(cloudServiceVc, animated: true)
})
```

