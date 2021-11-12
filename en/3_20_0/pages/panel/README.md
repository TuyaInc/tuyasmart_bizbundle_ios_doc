# Device Control BizBundle

## Features Overview

- Tuya Smart iOS Device Control BizBundle is the core container of Tuya Smart Device Control Panel, based on the Tuya Smart iOS Home SDK, it provides the interface package for loading and controlling the device control panel to speed up the application development process. It mainly includes the following functions:
  - Load Device Panel (Supported hardware device type: WIFI、Zigbee、Mesh、BLE)
  - Device Panel Control (Supported Device and Group Control, Unsupported Group Manager)
  - Device Alarm



## Integrate

Add the `TuyaSmartPanelBizBundle` in the project's `Podfile` file and execute the` pod update` command

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # TuyaSmart SDK
  pod "TuyaSmartHomeKit"
  # Add Device Control BizBundle
  pod 'TuyaSmartPanelBizBundle', '3.20.0.1.4'
  # If you need the sweeper, please rely on the relevant plug-in of the sweeper
  # pod 'TuyaRNApi/Sweeper', '5.29.62-Bizbundle.3.20.0-1'
end
```

**Note**

The Device Control BizBundle encapsulates a series of RN interfaces for the panel to call, which will involve some of Apple's privacy rights statements.

- If the connected device panel is related to the use of photo albums (for example: cloud albums), you need to add the following permission statement in the project's `info.plist`:

```
NSPhotoLibraryAddUsageDescription
```

- If the connected device panel is related to using the camera (for example: cloud album), you need to add the following permission statement in the project's `info.plist`:

```
NSCameraUsageDescription
```

- If the connected device panel is related to the use of location information, you need to add the following permission statement in the project's `info.plist`:

```
NSLocationWhenInUseUsageDescription
```

- If the connected device panel uses a microphone (for example: a music lamp panel), you need to add the following permission statement in the project's `info.plist`:

```
NSMicrophoneUsageDescription
```

- If the connected device panel is Bluetooth related, you need to add the following permission statement in the project's `info.plist`:

```
NSBluetoothAlwaysUsageDescription
NSBluetoothPeripheralUsageDescription
```



## Service Protocol

### Provide Service

BizBundle  provide services of  `TYPanelProtocol.h ` in `TYModuleServices` component as follows:

```objc
@protocol TYPanelProtocol <NSObject>
NS_ASSUME_NONNULL_BEGIN

// clear panel cache
- (void)cleanPanelCache;

/**
 * jump to device control panel，by push
 *
 * @param device        DeviceModel
 * @param group         GroupModel
 * @param initialProps  Custom initialization parameters will be set into the initialProps of the RN application with'extraInfo' as the key
 * @param contextProps  Custom panel context will be set into Panel Context with'extraInfo' as key
 * @param completion Result callback
 */
- (void)gotoPanelViewControllerWithDevice:(TuyaSmartDeviceModel *)device
                                    group:(nullable TuyaSmartGroupModel *)group
                             initialProps:(nullable NSDictionary *)initialProps
                             contextProps:(nullable NSDictionary *)contextProps
                               completion:(void(^ _Nullable)(NSError * _Nullable error))completion;

/**
 * jump to device control panel，by present
 *
 * @param device        DeviceModel
 * @param group         GroupModel
 * @param initialProps  Custom initialization parameters will be set into the initialProps of the RN application with'extraInfo' as the key
 * @param contextProps  Custom panel context will be set into Panel Context with'extraInfo' as key
 * @param completion Result callback
 */
- (void)presentPanelViewControllerWithDevice:(TuyaSmartDeviceModel *)device
                                       group:(nullable TuyaSmartGroupModel *)group
                                initialProps:(nullable NSDictionary *)initialProps
                                contextProps:(nullable NSDictionary *)contextProps
                                  completion:(void (^ _Nullable)(NSError * _Nullable error))completion;

// RN Version For BizBundle
- (NSString *_Nonnull)rnVersionForApp;

NS_ASSUME_NONNULL_END
@end
```



### Dependent Services

The main function of the device control biz bundle is to load the device, and there will be different functions for different devices. To ensure the normal operation of these functions, it will rely on the following protocols： `TYSmartHomeDataProtocol`、 `TYDeviceDetailProtocol`、 `TYSettingsProtocol`、 `TYOTAGeneralProtocol`、 `TYGroupHandleProtocol`、 `TYRNCameraProtocol`、 `TYCameraProtocol`。

#### TYSmartHomeDataProtocol

Provide the current family information required to load the device panel, **must be implemented**

```objc
/**
 Get the current family. If there is no family, return nil.
 
 @return TuyaSmartHome
 */
- (TuyaSmartHome *)getCurrentHome;
```

####  TYDeviceDetailProtocol

Click the jump event in the Panel Toolbar Right Menu of the device panel interface

```objc
/**
 Click the button click event on the right side of the navigation bar to jump to the device details page

 @param device DeviceModel
 @param group GroupModel, maybe nil.
 */
- (void)gotoDeviceDetailDetailViewControllerWithDevice:(TuyaSmartDeviceModel *)device group:(TuyaSmartGroupModel *)group;
```

#### TYSettingsProtocol

When triggering the release of dp control devices in the panel, an optional switch is provided for the app to emit sound effects. Implement the following method to return whether to enable sound effects:

```objc
/**
 * Check if audio is turned on
 */
- (BOOL)soundEnabled;
```

#### TYOTAGeneralProtocol

When entering the device panel, provide an event to check the device firmware update. Implement the following method to check the firmware upgrade:

```objc
/**
 Check the device firmware update, if there is an update, it will display the firmware update prompt
 
 @param deviceModel DeviceModel
 @param isManual Whether to manually detect the upgrade
 @param theme theme color
 */
- (void)checkFirmwareUpgrade:(TuyaSmartDeviceModel *)deviceModel isManual:(BOOL)isManual theme:(TYOTAControllerTheme)theme;
```

#### TYGroupHandleProtocol

For BLE Mesh devices, there are events in the panel that need to jump to the Mesh group interface. If you need to jump to the mesh group, implement the following methods:

```objc
/**
Jump to local mesh group

@available 1.0.0
@param params 
@param success 
@param failure 
*/
- (void)impl_jumpToMeshLocalGroup:(NSDictionary*)params success:(RCTResponseSenderBlock)success failure:(RCTResponseErrorBlock)failure ;
```

#### TYRNCameraProtocol

Load the jump event of the RN panel of the camera device, if you access the camera panel service package `TuyaSmartCameraRNPanelBizBundle` at the same time, it does not need to be implemented;

```objc
/**
 Get camera RN panel
 @param devId The devId of the camera device
 */
- (UIViewController *)cameraRNPanelViewControllerWithDeviceId:(NSString *)devId;
```

#### TYCameraProtocol

Load the jump event of the native panel of the camera device. If the camera panel business package `TuyaSmartCameraPanelBizBundle` is connected at the same time, it does not need to be implemented;

```objc
/**
	Get Camera Native Panel
 @param devId The devId of the camera device
 @param uiName The uiName of the camera device, different uiName corresponds to different versions of the panel
 */
- (UIViewController *)viewControllerWithDeviceId:(NSString *)devId uiName:(NSString *)uiName;
```



## Guidance

### Attention

1、Make sure that the user is logged in before using any interface

2、Before using bizbundle，must  implement the protocol method `getCurrentHome` in `TYSmartHomeDataProtocol`

Objective-C 

```objc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYSmartHomeDataProtocol.h>

- (void)initCurrentHome {
    // register service
    [[TuyaSmartBizCore sharedInstance] registerService:@protocol(TYSmartHomeDataProtocol) withInstance:self];
}

// implementation
- (TuyaSmartHome *)getCurrentHome {
    TuyaSmartHome *home = [TuyaSmartHome homeWithHomeId:@"current_home_id"];
    return home;
}
```

Swift

```swift
import TuyaSmartDeviceKit

class TYActivatorTest: NSObject,TYSmartHomeDataProtocol{
  	
    func test() {
        TuyaSmartBizCore.sharedInstance().registerService(TYSmartHomeDataProtocol.self, withInstance: self)
    }
    
    func getCurrentHome() -> TuyaSmartHome! {
        let home = TuyaSmartHome.init(homeId: 111)
        return home
    }
    
}
```

### Clear Panel Cache

Objective-C 

```objc
id<TYPanelProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYPanelProtocol)];
[impl cleanPanelCache];
```

Swift

```swift
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYPanelProtocol.self) as? TYPanelProtocol
impl?.cleanPanelCache()
```

### Open Panel

Objective-C 

```objc
id<TYPanelProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYPanelProtocol)];
// Method 1: Jump by default Push method
[impl gotoPanelViewControllerWithDevice:deviceModel group:nil initialProps:nil contextProps:nil completion:^(NSError * _Nullable error) {
    if (error) {
        NSLog(@"Load error: %@", error);
    }
}];
// Method 2: Jump using Present
[impl presentPanelViewControllerWithDevice:deviceModel group:nil initialProps:nil contextProps:nil completion:^(NSError * _Nullable error) {
    if (error) {
        NSLog(@"Load error: %@", error);
    }
}];
// Method 3: Get the panel view controller and jump by itself
[impl getPanelViewControllerWithDeviceModel:device groupModel:group initialProps:nil contextProps:nil completionHandler:^(__kindof UIViewController * _Nullable panelViewController, NSError * _Nullable error) {

}];
```

Swift

```swift
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYPanelProtocol.self) as? TYPanelProtocol
// Method 1: Jump by default Push method
impl?.gotoPanelViewController(withDevice: deviceModel!, group: nil, initialProps: nil, contextProps: nil, completion: { (error) in
    if let e = error {
        print("\(e)")
    }
})
// Method 2: Jump using Present
impl?.presentPanelViewController(withDevice: deviceModel!, group: nil, initialProps: nil, contextProps: nil, completion: { (error) in
    if let e = error {
        print("\(e)")
    }
})
// Method 3: Get the panel view controller and jump by itself
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYPanelProtocol.self) as? TYPanelProtocol
impl?.getPanelViewController(with: deviceModel!, groupModel: nil, initialProps: nil, contextProps: nil, completionHandler: { (panelViewController, error) in
		if let e = error {
        print("\(e)")
    }
})
```
