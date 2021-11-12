# Device Configuration BizBundle

## Features

The business functions cover all the Wi-Fi devices, ZigBee devices, Bluetooth devices, QR code scanning devices (such as GPRS & NB-IOT devices)  and other types of devices currently equipped with TuyaSmart APP. 

### 1. Wi-Fi Devices

Support Wi-Fi smart devices to connect to the cloud service.  The Wi-Fi device mainly includes EZ mode and AP mode, among which IPC device also supports the QR code scanning mode

| Attributes                   | Description                                                  |
| ---------------------------- | ------------------------------------------------------------ |
| EZ mode                      | Also known as the fast connection mode. The App packs the network data packets into the designated area of the 802.11 data packets and sends them to the surrounding environment. The Wi-Fi module of the smart device is in the promiscuous model, and  captures all the packets in the network, and parses out the  network information packet sent by the App according to the agreed protocol data format. |
| AP mode                      | Also known as hotspot mode. The mobile phone connects the smart device's hotspot, and the two parties establish a Socket connection to exchange data through the agreed port. |
| Camera QR code scanning mode | The camera device obtains the configuration  data information by scanning the QR code on the App. |

### 2. ZigBee Devices

Support ZigBee gateway and sub-device network Configuration

| Attributes     | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| ZigBee         | ZigBee technology is a short-range, low-complexity, low-power, low-speed, low-cost two-way wireless communication technology. It is mainly used for data transmission between various electronic devices with short distances, low power consumption and low transmission rates, as well as typical applications with periodic data, intermittent data and low response time data transmission. |
| ZigBee Gateway | The device that integrates the coordinator and WiFi functions in the ZigBee network is responsible for the establishment of the ZigBee network and the storage of data information. |
| Sub-device     | Routing or terminal equipment in ZigBee network, responsible for data forwarding or terminal control response. |

### 3. Bluetooth Devices

Tuya Bluetooth has three technical lines, including SingleBLE, SigMesh, TuyaMesh, and dual-mode devices

| Attributes       | Description                                                  |
| ---------------- | ------------------------------------------------------------ |
| SingleBLE        | Bluetooth device connected one-to-one with mobile phone via Bluetooth |
| SigMesh          | Adopts the Bluetooth topology communication released by the Bluetooth Technology Alliance |
| TuyaMesh         | Adopting Tuya's self-developed Bluetooth topology communication |
| Dual-mode Device | Support multi-protocol, ie devices with both Wi-Fi and BLE capabilities |

### 4. Scanning Code Network Configuration Devices

This type of device is connected to the Tuya cloud service after being powered on. APP scans the QR code on the device (must be the QR code rule supported by the Tuya cloud service, and supports specific firmware access methods. Consult Tuya Technology-related business and project managers ) and enables  device to activate on the Tuya Cloud

| Attributes    | Description                                                  |
| ------------- | ------------------------------------------------------------ |
| GPRS Device   | Smart devices that use GPRS communication technology to access the network and connect to cloud services |
| NB-IOT Device | Smart device adopting NarrowBand-Internet of Things technology |

### 5. Automatic Network Discovery

Integrated with Tuya universal Configuration network technology to provide users with a set of fast Configuration network functions

## Access

Add the bizbundle in  the `Podfile` file, then execute pod update command

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # add device config bizbundle
  pod 'TuyaSmartActivatorBizBundle'
end
```



**Note**

The Wi-Fi device Activation process needs to obtain the  ssid of the current Wi-Fi , and the project needs to add the following permission statement in info.plist. Then create a CLLocationManager object, and call the requestWhenInUseAuthorization method

```
NSLocationAlwaysAndWhenInUseUsageDescription
NSLocationAlwaysUsageDescription
NSLocationWhenInUseUsageDescription
```



The QR code scanning function requires system camera permissions, and the following permission statement needs to be added to info.plist.

```
NSCameraUsageDescription
```



## Custom Config

### 1. Bluetooth Function

BizBundle supports Wi-Fi, Bluetooth and other types of devices to connect to the network. Set `needBle` false，when not needed

If you need the Bluetooth function, you need to add the Bluetooth permission declaration in the project's info.Plist file, set the `needBle`  property in `ty_custom_config.json`  to true, and then add the following dependencies to the project:

**Permission statement**

```
NSBluetoothAlwaysUsageDescription
NSBluetoothPeripheralUsageDescription
```

**Configuration**

```json
{
    "config": {
        "appId": 123,    
        "tyAppKey": "xxxxxxxxxxxx", 
        "appScheme": "tuyaSmart",
        "hotspotPrefixs": ["SmartLife"], 
        "needBle": true // true ,need bluetooth
    },
   "colors":{
        "themeColor": "#FF5A28", 
    }
}
```

**Dependencies**

```ruby
  pod 'TYBLEInterfaceImpl'
  pod 'TYBLEMeshInterfaceImpl'
  pod 'TuyaSmartBLEKit'
  pod 'TuyaSmartBLEMeshKit'
```



### 2. Custom Device Hotspot Name Prefix

Tuya Device hotspot name prefix defaults to `SmartLife`.  Can custom hotspot name prefix as your device's through setting `hotspotPrefixs` in `ty_custom_config.json`

```json
{
    "config": {
        "appId": 123,    
        "tyAppKey": "xxxxxxxxxxxx", 
        "appScheme": "tuyaSmart",
        "hotspotPrefixs": ["SL"], //  device hotspots name prefix 
        "needBle": true
    },
   "colors":{
        "themeColor": "#FF5A28", 
    }
}

```



## Service Protocol

### Provide Service

BizBundle  provide services of  `TYActivatorProtocol.h ` in `TYModuleServices` component as follows:

```objc
#ifndef TYActivatorProtocol_h
#define TYActivatorProtocol_h

@class TuyaSmartHome;

@protocol TYActivatorProtocol <NSObject>

/**
 * Start config
 * Goto device config list view 
 */
- (void)gotoCategoryViewController;

/**
 *  Obtain device information after each device connection
 */
- (void)getActivatedDeviceListWithCompletion:(void (^)(NSArray *_Nullable devIdList))completion;

@end
#endif /* TYActivatorProtocol_h */

```



Custom the return operation form device config list View , you need to implement the protocol method provided by TYActivatorExternalExtensionProtocol

**TYActivatorExternalExtensionProtocol**

```objc
/**
 *  Back action form device config list View
 *  Need to implement when additional operations are needed
 */
- (BOOL)categoryViewControllerCustomBackAction;
```



### Dependent Services

BizBundle running depends on the protocol method provided by TYSmartHomeDataProtocol. Before calling the BizBundle method, the following protocol needs to be implemented

#### TYSmartHomeDataProtocol

Set current home infomation  by implementing the following method.

```objc
/**
 Get current home info 
 
 @return TuyaSmartHome
 */
- (TuyaSmartHome *)getCurrentHome;
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
    TuyaSmartHome *home = [TuyaSmartHome homeWithHomeId:@"current home id"];
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



### Start Config

Objective-C 

```objc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYActivatorProtocol.h>


- (void)gotoDeviceConfig {
    id<TYActivatorProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYActivatorProtocol)];
    [impl gotoCategoryViewController];
  
    // get result
    [impl activatorCompletion:TYActivatorCompletionNodeNormal customJump:NO completionBlock:^(NSArray * _Nullable deviceList) {
        NSLog(@"deviceList: %@",deviceList);
    }];
}
```



Swift 

```swift
let homeImpl = TuyaSmartBizCore.sharedInstance().service(of: TYActivatorProtocol.self) as? TYActivatorProtocol
homeImpl?.goActivatorRootView()

impl?.activatorCompletion(TYActivatorCompletionNodeNormal, customJump: false, completionBlock: { (evIdList:[Any]?) in
            print(devIdList ?? [])
        })
```



### Custom  the Return Operation Form Category View Controller

Objective-C 

```objc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYActivatorExternalExtensionProtocol.h>


- (void)initCurrentHome {
  // register service
    [[TuyaSmartBizCore sharedInstance] registerService:@protocol(TYActivatorExternalExtensionProtocol) withInstance:self];
}

// implementation
- (BOOL)categoryViewControllerCustomBackAction {
    [self.navigationController popToRootViewControllerAnimated:YES];
    return YES;
}
```



Swift 

```swift

class TYActivatorTest: NSObject,TYActivatorExternalExtensionProtocol{

    func test() {
      // register service
 TuyaSmartBizCore.sharedInstance().registerService(TYActivatorExternalExtensionProtocol.self, withInstance: self)
    }
    
  // implementation
    func categoryViewControllerCustomBackAction() -> Bool {
        self.navigationController?.popToRootViewController(animated: true)
        return true;
    }
    
}
```



## Change Log

- 2020.07.02  Add custom return  method form device config list View 