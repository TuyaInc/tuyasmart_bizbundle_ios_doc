# 框架接入



## 业务包内容介绍

| Cocoapods 组件                        | 内容说明                                         | - |
| --------------------------- | ------------------------------------------------ | ---- |
| TuyaSmartBizCore            | 业务基础处理库，用于启动业务包以及自定义一些功能 | 必选 |
| TYModuleServices            | 业务包模块实现的协议                             | 必选 |
| TuyaSmartXXXBizBundle | 各种业务包实现组件                             | 可选 |



`TuyaSmartBizCore` 和 `TYModuleServices` 是使用业务包必须要依赖的基础库。

###  业务包核心库 (TuyaSmartBizCore) 介绍

提供启动业务包以及自定义配置功能

#### 主题色及自定义配置功能

该部分的配置，需要客户按照如下方式生成一份名为 ty_custom_config.json 的文件放入工程目录下

```json
{
    "config": {
        "appId": 123,    
        "tyAppKey": "xxxxxxxxxxxx", 
        "appScheme": "tuyaSmart",
        "hotspotPrefixs": ["SmartLife"],
        "needBle": true
    },
   "colors":{
        "themeColor": "#FF5A28", 
    }
}
```



**参数介绍**

| 参数            | 说明                         | 类型 | 必选 | 默认值 |
| --------------- | ---------------------------- |-| - | -|
| appId           | 应用 Id，在涂鸦开发者平台进入您的应用/SDK管理页面，页面 URL 中的 id 参数即为 appId，例如链接为 https://iot.tuya.com/oem/app?id=888888 则 appId 为 888888 | Number | 是 | 无 |
| tyAppKey        | 涂鸦开发者平台/App工作台/AppSDK 对应 SDK 中的 「AppKey」 | String | 是 | 无 |
| appScheme       | 涂鸦开发者平台/App工作台/AppSDK 对应 SDK 中的「渠道标识符」                   | String | 是 | 无 |
| hotspotPrefixs  | 配网设备热点前缀             | Array | 否 | ["SmartLife"] |
| needBle         | 是否需要支持蓝牙设备配网     | Boolean | 否 | true |
| themeColor      | UI 主题色设置                | String | 否 | #FF5A28 |



#### 接口介绍

业务包基础库，提供业务包调用的入口方法，同时也提供了客户需要实现协议服务的注册方法，具体提供的方法如下图：

```objc
/**
 * Get the instance which implement the special service protocol
 * eg:
 *  id<TYMallProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYMallProtocol)];
 *  [impl xxx]; // your staff...
 *
 * @param serviceProtocol service protocol
 * @return instance
 */
- (id)serviceOfProtocol:(Protocol *)service;

/**
 * Register a instance for service which can not be served by BizBundle itself
 * Each service can only register one instance or class at a time, whichever is the last
 *
 * @param service   service protocol
 * @param instance  instance which conform to the service protocol, strong reference
 *                  unregister if nil
 */
- (void)registerService:(Protocol *)service withInstance:(id)instance;

/**
 * Register a class for service which can not be served by BizBundle itself
 * Each service can only register one instance or class at a time, whichever is the last
 *
 *
 * @param service   service protocol
 * @param cls       class which conform to the service protocol, [cls new] to get instance
 *                  unregister if nil
 */
- (void)registerService:(Protocol *)service withClass:(Class)cls;

/**
 * Register route handler for route which can not be handled by BizBundle itself
 * @param handler   block to handle route
 *                  @param url  the route url
 *                  @param raw  the route raw data
 *                  @return true if route can be handled, otherwise return false
 */
- (void)registerRouteWithHandler:(BOOL(^)(NSString *url, NSDictionary *raw))handler;

/**
* Update config of biz resource
*
*/
- (void)updateConfig;
```

> 账号登录成功后，务必调用`- (void)updateConfig`接口更新必要的缓存数据，否则部分业务包将无法正常使用。

### 服务协议 (TYModuleServices)

提供各个业务包实现的服务协议


## 快速集成

### 使用 CocoaPods 集成

在 `Podfile` 文件中添加以下内容完成业务包核心库添加：

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
    # 添加涂鸦智能全屋 SDK
    pod "TuyaSmartHomeKit"
    # 若需要相关业务包功能，请添加相关业务库
    pod "TuyaSmartXXXBizBundle"
end
```



然后在项目根目录下执行 `pod update` 命令，集成第三方库。

CocoaPods 的使用请参考：[CocoaPods Guide](https://guides.cocoapods.org)



### 自定义配置项

填写业务包初始化需要的配置

```json
{
    "config": {
        "appId": 123,    
        "tyAppKey": "xxxxxxxxxxxx", 
        "appScheme": "tuyaSmart"
    },
    "colors":{
        "themeColor": "#FF5A28", 
    }
}
```

生成名为 `ty_custom_config.json` 的 json 文件，放到工程根目录下。

涂鸦业务包准备工作已经完成，可以开始业务包的使用，具体业务包的集成使用请到业务包导航目录下寻找相应的说明。
















