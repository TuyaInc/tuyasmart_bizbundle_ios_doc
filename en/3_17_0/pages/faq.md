# FAQ

## Device Control BizBundle

**1. Why does the navigation bar style of other view controllers change after calling the Push method to jump to the panel?**

The essence of the device control business package is to provide a view controller to load the interface rendered by React-Native, so the navigation bar is hidden inside the view controller, mainly to change the transparency. Therefore, after using the push method to enter the panel controller, when pop back to its own controller, you need to restore the style (the main reason is that the push and pop animations provided by iOS are managed by NavigationController, and the NavigationBar used is the only one; in NavigationController Under the Stack storage structure, whenever the ViewController in the Stack modifies the navigation bar, it will inevitably affect the display effect of other ViewControllers). Two ways to restore styles are suggested below:

Case 1.

```objective-c
// 1. Import header file
#import <TYNavigationController/TYNavigationTopBarProtocol.h>
// 2. Show navigation bar
self.ty_topBarHidden = NO;
self.ty_topBarAlpha = 1.0;
// 3. Restore the navigation bar style (for example: modify the background color)
self.ty_topBarBackgroundImage = <#Image#>;
// Or
self.ty_topBarBackgroundColor = <#Color#>;
```

Case 2.

```objective-c
// 1. Import header file
#import <TYNavigationController/TYNavigationCallbackProtocol.h>
// 2. Implement the proxy method provided by TYNavigationCallbackProtocol
- (void)ty_naviTransitioning:(id<UIViewControllerTransitionCoordinatorContext>)context {
    // 3. Show navigation bar
    [self.navigationController setNavigationBarHidden:false animated:true];
    // 4. Restore the navigation bar style (for example: modify the background color)
    [self.navigationController.navigationBar setBarTintColor:<#Color#>];
  	// Or
    [self.navigationController.navigationBar setBackgroundImage:<#Image#> forBarMetrics:UIBarMetricsDefault];
}
```

**2. When relying on plugins, `pod update` reports an error?**

When the project uses cocoapods to integrate the business package, due to the logic of the pod, when the version of each Pod library is not specified in the Podfile, the latest official version number (x.y.z, where x, y, and z are all numbers) will be pulled by default. If you need to rely on the extended functions of the device control service package, you need to rely on the relevant plug-in. Since the version number of the plug-in is pre-release, you need to specify the version number marked with the latest date of the baseline of the device control service package in the Podfile ( [Click to view the business package version number](../pages/versions.md)).