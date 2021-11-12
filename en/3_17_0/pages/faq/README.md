# FAQ BizBundle

## Introduction

The FAQ BizBundle provides an iOS container that hosts "questions and feedback", and provides a troubleshooting and feedback channel for your app.

## Integrate

Add the `TuyaSmartHelpCenterBizBundle` in the project's `Podfile` file and execute the `pod update` command

```objective-c
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # Add Faq BizBundle 
  pod 'TuyaSmartHelpCenterBizBundle'
end
```

## Service Protocol

### Service Provided By BizBundle

The FAQ BizBundle implements the  `TYHelpCenteProtocol` protocol to provide services. View the `TYHelpCenteProtocol.h` file in the `TYModuleServices` as follows:

```objective-c
@protocol TYHelpCenteProtocol <NSObject>

@optional

/**
 *  jump to HelpCenter page
 **/
- (void)gotoHelpCenter;

@end
```

### Service Required By BizBundle

None.

## User Guidance

### Precautions

1. Before using any method, be sure to confirm that the user is logged in.

2. When the login user changes, be sure to re-judge the FAQ availability status and re-acquire the FAQ page

   
### Go To FAQ Page

Objective-C

```objective-c
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYHelpCenteProtocol.h>

id<TYHelpCenteProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYHelpCenteProtocol)];

[impl gotoHelpCenter];

```



Swift

```swift
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYHelpCenteProtocol.self) as? TYHelpCenteProtocol

impl?.gotoHelpCenter()

```



 