---
layout: post
title: "iOS动态库注入实践-伪造微信定位"
date: 2017-02-27 18:29
comments: true
categories: dynamic library,runtime,inject
---


原理：创建一个动态库利用oc的runtime特性替换定位相关回调方法伪造GPS坐标，然后将动态库注入微信。
##伪造GPS坐标
原理图：

![fake](http://cloay.com/images/fake.jpg)

替换CLLocationManager的setDelegate:方法，将CLLocationManager的delegate替换为我们自定义的WMLocationProxy代理类，WMLocationProxy持有真正的CLLocationManager的delegate。

```objective-c
- (void)wm_setDelegate:(id)delegate {
    WMLocationProxy *proxy = [[WMLocationProxy alloc] init];
    proxy.oLMDelegate = delegate;
    self.proxy = proxy;
    [self wm_setDelegate:proxy];
}

```
<!-- more -->
在WMLocationProxy中转发CLLocationManagerDelegate的方法时先替换位置坐标，然后再转发给CLLocationManagerDelegate。

```objective-c
- (void)forwardInvocation:(NSInvocation *)anInvocation {
    if (self.oLMDelegate && [self.oLMDelegate respondsToSelector:anInvocation.selector]) {
        if (anInvocation.selector == @selector(locationManager:didUpdateToLocation:fromLocation:)) {
            
            CLLocation *originLocation;
            [anInvocation getArgument:&originLocation atIndex:3];
            
            CLLocation *newLocation = [self makeFakeLocationWithOriginLocation:originLocation];
            [anInvocation setArgument:&newLocation atIndex:3];
        }
        
        if (anInvocation.selector == @selector(locationManager:didUpdateLocations:)) {
            __unsafe_unretained NSArray *locations;
            [anInvocation getArgument:&locations atIndex:3];
            if (locations.count > 0) {
                CLLocation *originLocation = locations[0];
                CLLocation *fakeLocation = [self makeFakeLocationWithOriginLocation:originLocation];
                NSArray *fakeLocations = @[fakeLocation];
                self.orginLocations = fakeLocations;
                [anInvocation setArgument:&fakeLocations atIndex:3];
            }
        }
        [anInvocation invokeWithTarget:self.oLMDelegate];
    }
}
```

##动态库（Dynamic library）注入

iOS8发布之后Xcode允许创建动态库，我们可以创建一个动态库和原ipa文件合并重新打包ipa。具体参考[iOSDylibInjectionDemo](https://github.com/depoon/iOSDylibInjectionDemo)

stackoverflow关于动态库的问题可以看看

* [Library? Static? Dynamic? Or Framework? Project inside another project](http://stackoverflow.com/questions/15331056/library-static-dynamic-or-framework-project-inside-another-project)
* [Can you build dynamic libraries for iOS and load them at runtime?](https://stackoverflow.com/questions/4733847/can-you-build-dynamic-libraries-for-ios-and-load-them-at-runtime)

<br>
<br>
<br>
<br>