---
layout: post
title: "WebView loading https content"
date: 2016-07-20 00:37
comments: true
categories: iOS,UIWebView,https,WKWebView 
---

###iOS使用Webview加载https网页

项目新的需求需要在客户端中嵌入网页，以前的项目中也做过类似的功能，感觉不会有什么困难，使用WebView加载指定的URL就行了。但这次我们需要加载https页面，由于没有相关的实战经验，在整个开发过程中踩了不少的坑，现在给大家分享一下。
<!-- more -->
首先我很2的像以前一样直接加载URL，显然是加载不出来内容的。然后开始Google，首先看到可以通过私有API中的setAllowsAnyHTTPSCertificate:forHost方法但这有被拒的风险。还有一种是通过实现自己的NSURLProtocol协议，然后拦截需要拦截的请求，在请求代理中进行证书验证。然后在把请求的结果通过NSURLProtocol的self.client URLProtocol:didLoadData:方法返回给WebView。NSURLProtocol的具体使用需要实现一下几个方法：

```objective-c
+(BOOL)canInitWithRequest:(NSURLRequest *)request{
 	return [request.URL.absoluteString isEqualToString:@"https://"];
}

```

canInitWithRequest方法如果是需要自己实现的请求就return YES 否则 return NO。

```objective-c
+ (NSURLRequest *)canonicalRequestForRequest:(NSURLRequest *)request{
	//返回规范化的请求，直接返回当前的Request就可以。
    NSMutableURLRequest *cdnRequest = [request mutableCopy];
    return cdnRequest;
}

```
```objective-c
- (void)startLoading {
	//具体的加载实现
}
```

```objective-c
- (void)stopLoading {
    //停止
}
```
###下面看看我们具体的实现：

```objective-c
//
//  MyWebviewProtocol.m
//  Cloay
//
//  Created by cloay on 16/6/14.
//  Copyright © 2016年 Cloay. All rights reserved.
//

#import "MyWebviewProtocol.h"
#import "AFSecurityPolicy.h"

@interface MyWebviewProtocol()<NSURLSessionDataDelegate>
@property (nonatomic, strong) NSURLSession *session;

@end

@implementation MyWebviewProtocol

+ (BOOL)canInitWithRequest:(NSURLRequest *)request{
    NSString *urlStr = request.URL.absoluteString;
    CLog(@"----origin request url = %@", urlStr);
    NSRange range = [urlStr rangOfString:@"https://"];
   //这里是我们的拦截逻辑，需要自己实现的请求就return YES
    return range.length > 0;
}

+ (NSURLRequest *)canonicalRequestForRequest:(NSURLRequest *)request{
    NSMutableURLRequest *myRequest = [request mutableCopy];
    return myRequest;
}

- (void)startLoading {
    if (!self.session) {
        NSOperationQueue *queue = [[NSOperationQueue alloc] init];
        self.session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:queue];
    }
    
    NSURLSessionDataTask *task = [self.session dataTaskWithRequest:[TCWebviewProtocol canonicalRequestForRequest:self.request]];
    [task resume];
    
}

- (void)stopLoading {
    [self.session invalidateAndCancel];
    [self.session finishTasksAndInvalidate];
}


#pragma mark NSURLSession Delegate
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task willPerformHTTPRedirection:(NSHTTPURLResponse *)response
        newRequest:(NSURLRequest *)request
    completionHandler:(void (^)(NSURLRequest * __nullable))completionHandler{
    
    [self.client URLProtocol:self wasRedirectedToRequest:request redirectResponse:response];

}

- (void)URLSession:(NSURLSession *)session didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition, NSURLCredential *))completionHandler{
    
    NSURLSessionAuthChallengeDisposition disposition = NSURLSessionAuthChallengePerformDefaultHandling;
    NSURLCredential *credential = nil;
    if([challenge.protectionSpace.authenticationMethod isEqualToString:NSURLAuthenticationMethodServerTrust]){
        AFSecurityPolicy *securityPolicy = [AFSecurityPolicy policyWithPinningMode:AFSSLPinningModeCertificate];
        #if DEBUG
        securityPolicy.allowInvalidCertificates = YES;
        securityPolicy.validatesDomainName = NO;
        #endif
        if ([securityPolicy evaluateServerTrust:challenge.protectionSpace.serverTrust forDomain:challenge.protectionSpace.host]) {
            credential = [NSURLCredential credentialForTrust:challenge.protectionSpace.serverTrust];
            if (credential) {
                disposition = NSURLSessionAuthChallengeUseCredential;
            }
            
        } else {
            disposition = NSURLSessionAuthChallengeCancelAuthenticationChallenge;
        }
        
    }
    completionHandler(disposition, credential);
}

- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveResponse:(NSURLResponse *)response completionHandler:(void (^)(NSURLSessionResponseDisposition))completionHandler{
    [self.client URLProtocol:self didReceiveResponse:response cacheStoragePolicy:NSURLCacheStorageNotAllowed];
    completionHandler(NSURLSessionResponseAllow);
}

- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask
    didReceiveData:(NSData *)data{
    [self.client URLProtocol:self didLoadData:data];
}

- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error{
    if (error) {
         [self.client URLProtocol:self didFailWithError:error];
    } else {
        [self.client URLProtocolDidFinishLoading:self];
    }
}

@end

```

我们的项目兼容iOS7以上系统，所以这里我们就使用了NSURLSession来实现具体的请求。通过委托的URLSession:didReceiveChallenge:completionHandler:方法进行证书的验证。通过这种方式看似很好的解决了我们的问题，也没有什么难度。然而在我们后续的测试和使用WKWebview优化的过程中相关坑陆续的蹦出来了，页面怎么都加载不出来了。把我都快坑炸了。。。

###我们发现的坑
开始调试时通过这种方式在iOS7和iOS9中没有什么问题，所有页面正常显示。后来我们为了性能在iOS8以上系统使用WKWebview来实现我们的功能。WKWebView是iOS8以后出的新的性能更优的WebView，虽然WKWebview的请求不能通过NSURLProtocol这种方式拦截，但可以直接通过WKNavigationDelegate中的webView:didReceiveChallenge:completionHandler:方法完成我们的需求，这样更简单。然后我们就马不停蹄的在iOS8以后的系统中替换成了WKWebView，然而事实证明我们是Too young了。在调试时我们发现iOS8系统死活不调用webView:didReceiveChallenge:completionHandler:方法，后来发现这是iOS8的一个bug（[ Server trust authentication challenges aren’t sent to the navigation delegate](https://bugs.webkit.org/show_bug.cgi?id=135327)），直到iOS9才修复。

接着我们又改为在iOS9使用WKWebView,iOS9以下继续使用UIWebView。悲剧的是在接下来的调试中我们又发现了一个问题。我们页面中需要登录，做法是客户端中用户登录后把token传给页面，让页面做一次自动登录。调试时在iOS7中一切正常，然而在iOS8中一直登录不成功，页面一直重定向到一个登录页面。这个问题纠结了好几天后来发现我们没有保持cookie，我们的NSURLSession的配置使用ephemeralSessionConfiguration，看文档发现ephemeralSessionConfiguration不保持cookie的。好吧，又2了。。。

###总结
客户端嵌入https网页时，在iOS8及一下还是只能使用UIWebView，通过实现NSURLProtocol自己实现网络请求进行证书验证。在iOS9系统中直接使用WKWebView就能很好的满足常见的需求。

参考链接：

* [NSHipster NSURLProtocol
](http://nshipster.com/nsurlprotocol/) 
* [ Server trust authentication challenges aren’t sent to the navigation delegate](https://bugs.webkit.org/show_bug.cgi?id=135327)


<br>
<br>
<br>
