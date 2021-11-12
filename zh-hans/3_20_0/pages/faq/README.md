# 常见问题与反馈业务包

## 介绍

常见问题与反馈业务包提供了承载 「问题与反馈」的 iOS容器，为您的App提供了问题排查与反馈渠道。

## 接入组件

在工程的 `Podfile` 文件中添加常见问题与反馈业务包组件，并执行 `pod update` 命令

```objective-c
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # 添加常见问题与反馈业务包
  pod 'TuyaSmartHelpCenterBizBundle', '~> 3.17.0'
end
```

## 服务协议

### 提供服务

常见问题与反馈业务包实现 `TYHelpCenterProtocol` 协议以提供服务，在 `TYModuleServices` 组件中查看 `TYHelpCenterProtocol.h` 协议文件内容为：

```objective-c

@protocol TYHelpCenterProtocol <NSObject>

@optional

/**
 * 跳转到常见问题与反馈页面
 **/
- (void)gotoHelpCenter;

@end
```

### 依赖服务

无

## 使用指南

### 注意事项

1.使用任何接口之前，务必确认用户已登录

2.登录用户发生状态变化时，务必重新判断常见问题与反馈业务包可用状态并重新获取常见问题与反馈页面。

### 跳转帮助中心页面

Objective-C 示例

```objective-c
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYHelpCenterProtocol.h>

id<TYHelpCenterProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYHelpCenterProtocol)];

[impl gotoHelpCenter];

```



Swift 示例

```swift
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYHelpCenterProtocol.self) as? TYHelpCenterProtocol

impl?.gotoHelpCenter()

```

