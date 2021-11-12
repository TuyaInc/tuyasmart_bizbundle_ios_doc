# House Management

## Features

The family biz bundle mainly includes business such as family, member, room, etc., which are the basic conditions for the management of the devices after devices are activated. The device can set the room where the device is located in the family after the device configuration, meanwhile, the family members who have different authority under the family correspond to different operation authority, and the family is also the largest unit of scene intelligence execution.

## Integrate

Add the `TuyaSmartFamilyBizBundle` in the project's Podfile file and execute the `pod update` command

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # TuyaSmart SDK
  pod "TuyaSmartHomeKit"
  # House Management
  pod 'TuyaSmartFamilyBizBundle', '~> 3.22.0'
end
```

## Service Protocol

### Service Provided By BizBundle
House Management implements `TYFamilyProtocol`to provide services，View the`TYFamilyProtocol.h` file in the `TYModuleServices` as follows：

```objc
#import <Foundation/Foundation.h>
#import <UIKit/UIKit.h>

typedef void(^DoneActionBlock)(NSString *changedName);

/// Family Management Service
@protocol TYFamilyProtocol <NSObject>

@optional

/// jump to Family Management ViewController
- (void)gotoFamilyManagement;

@end
```

## User Guidance

### Precautions

1. Before using any method, be sure to confirm that the user is logged in.

### goto family management page

Objective-C 

```objc
#import <TYModuleServices/TYModuleServices.h>
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>

id<TYFamilyProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYFamilyProtocol)];
if ([impl respondsToSelector:@selector(gotoFamilyManagement)]) {
    [impl gotoFamilyManagement];
}
```

Swift 

```swift
guard let impl = TuyaSmartBizCore.sharedInstance().service(of: TYFamilyProtocol.self) as? TYFamilyProtocol else {
    return
}
impl.gotoFamilyManagement?()
```

### Invite family members

#### Invite
You can invite family members to join your smart house by `add member`ability in `house setting`page, there are two methods to invite members:
1. invite by Tuya account
2. invite by invitation code

#### accept/refuse invitation
1. You can input the invitation code at page `Home Management`->`Join a home` to accept the invitation

2. You can accept or refuse the invitation at the page `Home Management`, when you enter the page , there will be the home invitation row at the list , click the row to handle the invitation

You may be want implement the delegate of `TuyaSmartHomeManager` to handle invitation in realtime, then pop an invitation alert to handle the invitation.

```objc
- (void)homeManager:(TuyaSmartHomeManager *)manager didAddHome:(TuyaSmartHomeModel *)home;
```

To judge if the home is an invitation or not, you can see the `dealStatus` of `TuyaSmartHomeModel`

```objc
typedef NS_ENUM(NSUInteger, TYHomeStatus) {
    TYHomeStatusPending = 1,      /**< Not deciding whether to join the home */
    TYHomeStatusAccept,           /**< The invitee has agreed to join the home */
    TYHomeStatusReject            /**< The invitee have refused to join the home */
};
```

Demo Objective-C code：
```objc
TuyaSmartHomeManager *manager = [TuyaSmartHomeManager new];
manager.delegate = self;

....
//next is the implementation of delegation

- (void)homeManager:(TuyaSmartHomeManager *)manager didAddHome:(TuyaSmartHomeModel *)homeModel {
    if (homeModel.dealStatus <= TYHomeStatusPending && homeModel.name.length > 0) {
    ///pop the invitation alert
    
    ///accept invitation
    TuyaSmartHome *home = [TuyaSmartHome homeWithHomeId:homeModel.homeId];
    [home joinFamilyWithAccept:YES success:^(BOOL result) {} failure:^(NSError *error) {}];
    
    ///refuse invitation
    [home joinFamilyWithAccept:NO success:^(BOOL result) {} failure:^(NSError *error) {}];
    }
}
```

Demo Swift code：
```Swift
let manager = TuyaSmartHomeManager()
manager.delegate = self

....
//next is the implementation of delegation

func homeManager(_ manager: TuyaSmartHomeManager!, didAddHome homeModel: TuyaSmartHomeModel!) {
    if (homeModel.dealStatus.rawValue <= TYHomeStatus.pending.rawValue) && (homeModel.name.isEmpty == false) {
        ///pop the invitation alert
        
        ///accept invitation
        let home = TuyaSmartHome(homeId: homeModel.homeId)
        home?.joinFamily(withAccept: true, success: { (result) in
            
        }, failure: { (error) in
            
        })
        
        ///refuse invitation
        home?.joinFamily(withAccept: false, success: { (result) in
            
        }, failure: { (error) in
            
        })
        
    }
}

```