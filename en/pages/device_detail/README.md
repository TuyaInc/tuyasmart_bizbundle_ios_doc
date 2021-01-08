#Device Detail BizBundle

------------
##Features

- Device information editing (device avatar, room, name)
- Device information query (ID, signal, etc.)
- Alternate network Settings
- Offline reminder function
- Remove device
- Frequently asked questions and feedback (need access to help Center business package)

- Check firmware upgrade (OTA business package is required)

------------

##Biz Bundle integration

### 1.pod integration
Add the business package component to the project's 'Podfile' file and execute the 'pod update' command

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
 # Device Detail BizBundle
  pod 'TuyaSmartDeviceDetailBizBundle', '~> 3.22.0'
end
```
### 2.Permission statement

**Device business package device avatar upload, use of photo albums and cameras, which will involve some Apple's privacy permission statement.**

-Need to add the following permission statement in the project's `info.plist`:
```
<!-- Album -->
<key>NSPhotoLibraryUsageDescription</key>
<string>App needs your consent to access the album</string>
<!-- Camera -->
<key>NSCameraUsageDescription</key>
<string>App needs your consent to access the camera</string>
```


------------

### 3. Configuration file
1. Create a new **configList.json** in the main project (the file already exists, no need to create it)

2. Add in configList.json
key: **deviceDetail**
value: the following array
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
3. The content of the configList.json configuration file is roughly as follows
```js
 {
  "deviceDetail": [{"type":"header"} ],
 "Other configuration page key":[xxxxx]
  }
```


4. Customize device details
Note that the order of the items in the deviceDetail array affects the display order of the items on the device detail page. If the item is removed, the device details page sub-function will also be removed accordingly.


deviceDetail type | Function point
---|---
header | View and modify the device icon, name, location
device_info | Device Information
net_setting | Device backup network
off_line_warn | Device offline reminder
section_other | Section header, no actual function
help_and_feedback | FAQ and feedback, need to integrate FAQ and feedback business package
check_firmware_update | Check firmware upgrade, need to integrate firmware upgrade service package
empty | empty view, no actual function
footer | remove device


### 4. Service Agreement

#### Dependent service (need to be implemented)
#####1.TYSmartHomeDataProtocol

```
/**
 To use this API to get the current family, be sure to use the protocol’s ‘updateCurrentHomeId:’ Api when updating the current family id.
 Get the current family, if there is no family, return nil.
 
 @return TuyaSmartHome
 */
-(TuyaSmartHome *)getCurrentHome;
```

Objective-C example

```objective-c
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYSmartHomeDataProtocol.h>


-(void)initCurrentHome {
    // Register the protocol to be implemented
    [[TuyaSmartBizCore sharedInstance] registerService:@protocol(TYSmartHomeDataProtocol) withInstance:self];
}

// Implement the corresponding protocol method
-(TuyaSmartHome *)getCurrentHome {
    TuyaSmartHome *home = [TuyaSmartHome homeWithHomeId:@"current family id"];
    return home;
}
```

Swfit example

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
### 4. Service Protocol

#### Dependent service (need to be implemented)
#####1.TYSmartHomeDataProtocol

```
/**
 To use this API to get the current family, be sure to use the protocol’s ‘updateCurrentHomeId:’ Api when updating the current family id.
 Get the current family, if there is no family, return nil.
 
 @return TuyaSmartHome
 */
-(TuyaSmartHome *)getCurrentHome;
```

Objective-C example

```objective-c
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYSmartHomeDataProtocol.h>


-(void)initCurrentHome {
    // Register the protocol to be implemented
    [[TuyaSmartBizCore sharedInstance] registerService:@protocol(TYSmartHomeDataProtocol) withInstance:self];
}

// Implement the corresponding protocol method
-(TuyaSmartHome *)getCurrentHome {
    TuyaSmartHome *home = [TuyaSmartHome homeWithHomeId:@"current family id"];
    return home;
}
```

Swfit example

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

####Optional function implementation

#### 1. OTA function
 #####**Access required**: [Access OTA Service Package](../ota/README.md "Access OTA Service Package")

#### 2. FAQ and feedback function
 #####**Access required**: [Access Help Center Business Package](../faq/README.md "Access Help Center Service Package")




###5. Provide service (method call)

 Method | Parameters |
 -------- | :-----------:
 Jump to network detection page | device id
Jump to the device details page and push |TuyaSmartDeviceModel/TuyaSmartGroupModel

####oc sample code
```
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYDeviceDetailProtocol.h>

 id<TYDeviceDetailProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYDeviceDetailProtocol)];
 
 //Jump to the device network detection page
 [impl gotoDeviceDetailNetworkViewControllerWithDeviceId:@"device id"];
 
 
  // Jump to the device details page, use push
  
  //If it is a device, new TuyaSmartDevice
 TuyaSmartDevice * device = [TuyaSmartDevice deviceWithDeviceId:@"device id"]
  [impl gotoDeviceDetailDetailViewControllerWithDevice: device.deviceModel group: nil];
  //If it is a group, new TuyaSmartGroup
 TuyaSmartGroup * group = [TuyaSmartGroup groupWithGroupId:@"group id"]
   [impl gotoDeviceDetailDetailViewControllerWithDevice: nil group: group.deviceModel];


```

####swift sample code
```
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYDeviceDetailProtocol.self) as? TYDeviceDetailProtocol
//Jump to the device network detection page
impl?.gotoDeviceDetailNetworkViewController(withDeviceId: "Device id")

  // Jump to the device details page, use push

  //If it is a device, new TuyaSmartDevice
  let device = TuyaSmartDevice.init(deviceId: "vdevo160888044089994")
  impl?.gotoDeviceDetailDetailViewController(withDevice: device?.deviceModel, group: nil)
  //If it is a group, new TuyaSmartGroup
  let group = TuyaSmartGroup.init(groupId: "group id")
  impl?.gotoDeviceDetailDetailViewController(withDevice: nil, group: group?.groupModel)


```