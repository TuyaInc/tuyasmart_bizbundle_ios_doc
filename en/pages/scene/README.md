# Scene BizBundle

## Features Overview

Smart divides into scene or automation actions.

Scene is a condition that users add actions and it is triggered manually; Automation action is a action set by users, and the set action is automatically executed when the condition is triggered.

The Tuya Cloud allows users to set meteorological or device conditions based on actual scenes in life, and if conditions are met, one or multiple devices will carry out corresponding tasks.


## Integrate

Add the `TuyaSmartSceneBizBundle ` in the project's `Podfile` file and execute the` pod update` command

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  pod 'TuyaSmartSceneBizBundle', '~> 3.22.0'
end
```
**Note**

The weather information in the scene will use location information, which will involve part of Apple's privacy statement.

- Add the following permission statement in the project's `info.plist`:

```
NSLocationWhenInUseUsageDescription
```


## Service Protocol

### Provide Service

BizBundle  provide services of  `TYSmartSceneProtocol.h` and `TYSmartSceneBizProtocol.h` in `TYModuleServices` component as follows:

The contents of the `TYSmartSceneProtocol.h` file in the `TYModuleServices` component are:

```objc

@protocol TYSmartSceneProtocol <NSObject>
/**
 *	Jump to the add new scene page, add new scene or automation
 *
 * @param callback Result callback after creation
 */
- (void)addAutoScene:(void(^)(TuyaSmartSceneModel *secneModel, BOOL addSuccess))callback;
/**
 *	 Jump to edit scene page, edit specified scene or automation.
 *  Note that before calling this method, you need to call the getSceneListWithHomeId: method of TYSmartSceneBizProtocol to get the list of scenes in the family, so that the scene cache will be generated, and then you can jump into the editing page normally.
 *
 * @param model The scene model object to be edited
 */
- (void)editScene:(TuyaSmartSceneModel *)model;

@end

```

The contents of the `TYSmartSceneBizProtocol.h` file in the `TYModuleServices` component are:

```objc
/**
 * Get a list of scenes, including automation and scenes
 * 
 * @param  homeId 
 */
- (void)getSceneListWithHomeId:(long long)homeId withSuccess:(void(^)(NSArray <TuyaSmartSceneModel *> *scenes))success failure:(void(^)(NSError *error))failure;

```


### Dependent Services

The scene biz bundle rely on the protocol method provided by the `TYSmartHomeDataProtocol` protocol. Before calling method in 'TYSmartSceneBizProtocol' and `TYSmartSceneProtocol ` , the following protocol needs to be implemented:

#### TYSmartHomeDataProtocol

Provide the current family information required to load the device panel,

 **must be implemented**

```objc
/**
 Get the current family. If there is no family, return nil.
 
 @return TuyaSmartHome
 */
- (TuyaSmartHome *)getCurrentHome;
```
#### TYSmartHouseIndexProtocol
Provide the administrator identity information required by the scene biz bundle. You can just return YES to allow all user to edit scene.

```objc
/**
 * Whether it is the administrator of the current family.
 *
 * @return YES means user's role in current home is administrator.
 */
- (BOOL)homeAdminValidation;
```


## Guidance

### Attention

1. Make sure that the user is logged in before using any interface.

2. Before using bizbundle, you need to implement the protocol method `getCurrentHome` in `TYSmartHomeDataProtocol` and the protocol method `homeAdminValidation` in `TYSmartHouseIndexProtocol` first.

3. To use swift, you need to add the bizbundle header file in the bridge file.

   ```
   #import <TuyaSmartBizCore/TuyaSmartBizCore.h>
   #import <TYModuleServices/TYModuleServices.h>
   #import <TuyaSmartSceneKit/TuyaSmartSceneModel.h>
   ```

Objective-C 

```objc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYSmartHomeDataProtocol.h>
#import <TuyaSmartDeviceKit/TuyaSmartDeviceKit.h>

- (void)registerProtocol {
    // Register the protocol to be implemented
    [[TuyaSmartBizCore sharedInstance] registerService:@protocol(TYSmartHomeDataProtocol) withInstance:self];
    [[TuyaSmartBizCore sharedInstance] registerService:@protocol(TYSmartHouseIndexProtocol) withInstance:self];
}

// Implement protocol method
- (TuyaSmartHome *)getCurrentHome {
    TuyaSmartHome *home = [TuyaSmartHome homeWithHomeId:@"current homeId"];
    return home;
}
- (BOOL)homeAdminValidation {
    //Return value according to the actual user role, or directly return YES to allow all users to edit.
    return YES;
}
```



Swfit 


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



### Add Scene

Objective-C 

```objc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYSmartSceneProtocol.h>


- (void)gotoAddScene {
    id<TYSmartSceneProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYSmartSceneProtocol)];
    [impl1 addAutoScene:^(TuyaSmartSceneModel *secneModel, BOOL addSuccess) {
            
    }];
}
```



Swift 

```swift
func addScene() {
    let impl = TuyaSmartBizCore.sharedInstance().service(of: TYSmartSceneProtocol.self) as? TYSmartSceneProtocol
    impl?.addAutoScene({ (sceneModel, result) in
            
    })
}

```

### Edit Scene

Objective-C 

```objc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYSmartSceneProtocol.h>

- (void)gotoEditScene {
    id<TYSmartSceneProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYSmartSceneProtocol)];
    [impl editScene:(your sceneModel)];
}
```



Swift 

```swift
func editScene() {
    let impl = TuyaSmartBizCore.sharedInstance().service(of: TYSmartSceneProtocol.self) as? TYSmartSceneProtocol
    impl?.editScene(your scenemodel)
        
    }
}

```

### Get the list of scenes

Objective-C 

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



Swift 

```swift
func getSceneList() {
	let impl = TuyaSmartBizCore.sharedInstance().service(of: TYSmartSceneBizProtocol.self) as? TYSmartSceneBizProtocol
   	impl?.getSceneList(withHomeId: 111, withSuccess: { (sceneArr) in
            
   	}, failure: { (error) in
            
  	})
}

```

### NotificationCenter

**kNotificationSmartSceneListUpdate**

Timing of notification:

1. Added scene successfully
2. Edit scene successfully
3. Delete scene successfully ( Bizbundle default scene types are TuyaSmartSceneRecommendTypeNone and TuyaSmartSceneCollectionTypeNone )

**kNotificationSmartSceneSaved**

Timing of notification:

1. Added scene successfully
2. Edit scene successfully

**kNotificationSmartSceneRecomDeleted**

Timing of notification:

1. Delete scene successfully ( Recommend scene or Collection type )