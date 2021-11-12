# Integration



## BizBundle List

| Cocoapods                        |         Introduction                               | - |
| --------------------------- | ------------------------------------------------ | ---- |
| TuyaSmartBizCore            | Core library, used to manager BizBundles and customize some functions | Required |
| TYModuleServices            | Services Protocol                            | Required |
| TuyaSmartXXXBizBundle | BizBudnles                             | Optional |



`TuyaSmartBizCore` and `TYModuleServices` is core libraries that must be used.

### TuyaSmartBizCore

Used to manager BizBundles and customize some functions

#### Theme color and custom configuration function

The configuration of this part requires the customer to generate a file named `ty_custom_config.json` and place it in the project root directory.

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



**Parameter introduction**

| Parameter            | Introduction                         | Type | Required | Default |
| --------------- | ---------------------------- |-| - | -|
| appId           | App id, enter your App/SDK management page in Tuya Developer Platform, the id parameter in the page URL is appId, for example, the link is https://iot.tuya.com/oem/app?id=888888 then appId 888888                  | Number | true | none |
| tyAppKey        | Tuya Developer Platform/App Service/App SDK, corresponds to "AppKey" in your SDK | String | true | none |
| appScheme       | Tuya Developer Platform/App Service/App SDK, corresponds to "Channel ID" in your SDK | String | true | none |
| themeColor      | Main theme color                | String | false | #FF5A28 |



#### Methods In BizCore

The BizCore provides the method to get instance which conformed BizBundle service protocol, and also provides the registration method for that developer provided class.

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

> After the account is successfully logged in,  call `-(void) updateConfig` to update the necessary cached data, otherwise some bizBundles will not be used normally.

### TYModuleServices

Services protocol provider


## Integrate to project

### Use CocoaPods

Add the following content to the `Podfile` file:

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
    # TuyaSmart SDK
    pod "TuyaSmartHomeKit"
    # BizBudnle that you want
    pod "TuyaSmartXXXBizBundle"
end
```

Then run `pod update`

Refer to Cocoapods: [CocoaPods Guide](https://guides.cocoapods.org)


### Custom configuration

Fill in the configuration required for BizBundle initialization

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
Generate a json file named `ty_custom_config.json` and place it in the root directory of the project.

The preparation work of Tuya iOS BizBundle has been completed, you can start using the BizBundle. 
For the integrated use of specific BizBundle, please go to the article of the BizBundle to find the corresponding instructions.
















