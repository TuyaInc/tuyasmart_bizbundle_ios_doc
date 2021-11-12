# Cloud Service BizBundle

Tuya smart cameras provide cloud storage video services, and cloud storage services can be activated through this bizBundle. After active cloud storage service, you can view and play cloud storage videos through [IPC SDK](https://tuyainc.github.io/tuyasmart_home_ios_sdk_doc/en/resource/Camera.html).

## Integrate

Add the `TuyaSmartCloudServiceBizBundle` in the project's `Podfile` file and execute the` pod update` command.

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # Add cloud service bizBundle
  pod 'TuyaSmartCloudServiceBizBundle'
end
```

## Service protocol

The cloud service bizBundle implements the `TYCameraCloudServiceProtocol` protocol to provide services. View the` TYCameraCloudServiceProtocol.h` file in the `TYModuleServices` as follows:

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

### Service Required By BizBundle

None.

## User guidance

### Precautions

1. Before using any method, be sure to confirm that the user is logged in.
2. The cloud storage service corresponds to the device one by one. When request the cloud storage service page, you need to pass in the `TuyaSmartDeviceModel` of the corresponding device.
3. Get the cloud service page as `UIViewController`, be sure to use` UINavigationController` to `push` or` present` it.

> Cloud storage services are strongly related to user information.
> The cloud storage service purchase page depends on the navigation controller and sets the navigation bar content, so the navigation controller needs to be packaged.

### Get cloud service page (UIViewController)

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

