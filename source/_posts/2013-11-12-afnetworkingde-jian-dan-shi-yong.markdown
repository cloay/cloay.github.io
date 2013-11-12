---
layout: post
title: "AFNetworking的简单使用"
date: 2013-11-12 15:01
comments: true
categories: afnetworking,asi,ios,iphone,apple 
---
<p>简单登录使用</p><br>
<!-- lang: cpp -->
    MBProgressHUD *hud = [MBProgressHUD showHUDAddedTo:self.view animated:NO];
    hud.labelText = @"正在登录...";
    //init params
    NSDictionary *parameters = [NSDictionary dictionaryWithObjects:[NSArray arrayWithObjects:@"login", nameTextField.text, passwordTextField.text, @"anonymous", nil] forKeys:[NSArray arrayWithObjects:@"cmd", @"login_name", @"password", @"access_token", nil]];
    
    AFHTTPRequestOperationManager *manager = [AFHTTPRequestOperationManager manager];
    manager.responseSerializer = [AFJSONResponseSerializer serializer];
    
    manager.responseSerializer.acceptableContentTypes = [NSSet setWithObject:@"text/html"];
    DLog(@"parmeters:%@", parameters);
    AFJSONRequestSerializer *jsonSerializer = [AFJSONRequestSerializer serializerWithWritingOptions:NSJSONWritingPrettyPrinted];
    manager.requestSerializer = jsonSerializer;
    
    [manager POST:SERVER parameters:parameters success:^(AFHTTPRequestOperation *operation, id responseObject) {
        DLog(@"JSON: %@", responseObject);
        [MBProgressHUD hideAllHUDsForView:self.view animated:YES];
        if ([responseObject objectForKey:@"error"]) {
            int errorCode = [[responseObject objectForKey:@"error_code"] integerValue];
            [Utils showErrorMsgWithErrorCode:errorCode];
            return;
        }
        NSDictionary *accountDic = [responseObject objectForKey:@"account"];
        if (accountDic) {
            appAccount = [Account jsonToAccount:responseObject];

            MainVC *mainVc = [[MainVC alloc] init];
            [self.navigationController pushViewController:mainVc animated:YES];
            
            [Utils HUDShowMsg:[UIApplication sharedApplication].keyWindow msg:@"登录成功！" withDelay:2];
        }else{
            [Utils HUDShowMsg:[UIApplication sharedApplication].keyWindow msg:@"登录失败，请稍候重试！" withDelay:2];
        }
        
    } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
        DLog(@"Error: %@", error);
        [MBProgressHUD hideAllHUDsForView:self.view animated:YES];
        [Utils showErrorMsgWithErrorCode:-1];
    }];

