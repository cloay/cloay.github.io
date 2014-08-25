---
layout: post
title: "仿QQ5.0Android版 在锁屏桌面显示消息框"
date: 2014-08-25 21:34
comments: true
categories: android, qq, receiver
---
<p>QQ5.0 for Android发布一段时间了，UI交互大调整，感受了一下很不习惯。不过新增了一个很赞的功能，当手机处于锁屏状态是有新消息提示时会在桌面显示一个消息框，并且可以直接回复信息，真的很方便。于是试着自己实现该功能。仔细想了想，大致思路如下：</p>
<!-- more -->
<p>后台service监听消息，当消息到达时，判断当前手机是否处于锁屏状态。如果是则在桌面显示消息框。关键点是如何在锁屏桌面显示消息框。我们仔细观察不难发现，QQ并没有直接在桌面显示消息框，而是极有可能替换了锁屏界面。试着尝试了一下基本实现了该功能。新建一个Theme为Wallpaper的Activity，并将布局定义成对话框样式。在onCreate(Bundle savedInstanceState)方法中加入如下代码：</p>
```java
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		Window window = getWindow();
		window.addFlags(WindowManager.LayoutParams.FLAG_SHOW_WHEN_LOCKED);
		window.addFlags(WindowManager.LayoutParams.FLAG_TURN_SCREEN_ON);
		window.addFlags(WindowManager.LayoutParams.FLAG_DISMISS_KEYGUARD);
		setContentView(R.layout.alert_layout);
		
		titleTextV = (TextView) findViewById(R.id.msg_title_textV);
		titleTextV.setText("New message: ");
		msgTextV = (TextView) findViewById(R.id.msg_content_textV);
		msgTextV.setText("Hi, I am cloay!");
		
		cancleBtn = (Button) findViewById(R.id.alert_cancle_btn);
		cancleBtn.setOnClickListener(this);
		okBtn = (Button) findViewById(R.id.alert_ok_btn);
		okBtn.setOnClickListener(this);
	}
```
<p>当消息到达时，弹出对话框。对话框有取消和确定两个按钮，取消按钮点击返回锁屏界面，点击确定按钮进入应用。锁屏需要获取相应权限，新建一个MyAdminReceiver并在清单文件中加入如下代码：</p>
```xml
	<receiver android:name="com.cloay.alertnotifidemo.MyAdminReceiver" >
            <meta-data android:name="android.app.device_admin"
                android:resource="@xml/my_admin_receiver"/>
            <intent-filter >
                <action android:name="android.app.action.DEVICE_ADMIN_ENABLED"/>
            </intent-filter>
        </receiver>
```
<p>在MainActivity获取权限：</p>
```java
	DevicePolicyManager policyManager = (DevicePolicyManager) getSystemService(DEVICE_POLICY_SERVICE);
		ComponentName componentName = new ComponentName(this, MyAdminReceiver.class);
		
		boolean isAdminActive = policyManager.isAdminActive(componentName);
		if(!isAdminActive){
			Intent intent = new Intent(DevicePolicyManager.ACTION_ADD_DEVICE_ADMIN); 
			intent.putExtra(DevicePolicyManager.EXTRA_DEVICE_ADMIN, componentName); 
			startActivity(intent);
		}
```
<p>当点击了取消按钮后,就可以锁屏了，锁屏之后再让对话框消失</p>
```java
	DevicePolicyManager policyManager = (DevicePolicyManager) getSystemService(DEVICE_POLICY_SERVICE);
			policyManager.lockNow();
			new Handler().postDelayed(new Runnable() {
				
				@Override
				public void run() {
					finish();
				}
			}, 1000);
```
<p>大致就是这些，实现在锁屏桌面显示消息框。效果如下：</p>
<img src="https://raw.githubusercontent.com/cloay/AlertNotifiDemo/master/pic.jpg"/><br>
<a href="https://github.com/cloay/AlertNotifiDemo">完整代码已上传github</a><br>
<p>项目中使用Service模仿每隔10秒收到新消息。</p>

