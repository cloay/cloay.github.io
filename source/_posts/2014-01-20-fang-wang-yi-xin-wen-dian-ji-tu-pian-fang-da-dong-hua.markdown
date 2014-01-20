---
layout: post
title: "仿网易新闻点击图片放大动画"
date: 2014-01-20 21:16
comments: true
categories: Android,ScaleAnmation,TranslateAnimation 
---
点击新闻中小图，从图片位置以动画的方式逐渐移动到屏幕中心并放大图片<br/>
具体实现：<br/>
``` java
    public static void browerImage(Activity activity, final ImageView originImageV, final ImageView scaleImageV){
                int x = originImageV.getLeft();
                int y = originImageV.getTop();
``` 
<!-- more -->
``` java
    public static void browerImage(Activity activity, final ImageView originImageV, final ImageView scaleImageV){
                int x = originImageV.getLeft();
                int y = originImageV.getTop();
                DisplayMetrics metrics = new DisplayMetrics();
                activity.getWindowManager().getDefaultDisplay().getMetrics(metrics);
                int width = metrics.widthPixels;
                AnimationSet animationSet = new AnimationSet(true);
                
                TranslateAnimation translateAnimation = new TranslateAnimation(
                Animation.ABSOLUTE, x,
                Animation.ABSOLUTE, 0,
                Animation.ABSOLUTE, y,
                Animation.ABSOLUTE, scaleImageV.getY() - getStatusBarHeight(activity));
        translateAnimation.setDuration(1500);
        animationSet.addAnimation(translateAnimation);

        ScaleAnimation scaleAnimation = new ScaleAnimation(1.0f, (float)width/originImageV.getWidth(), 1.0f,
                        (float)width/originImageV.getWidth(), 
                        Animation.ABSOLUTE, 0,
                Animation.ABSOLUTE, 0);
        scaleAnimation.setDuration(1500);
        animationSet.addAnimation(scaleAnimation);
        
        
        animationSet.setFillEnabled(true);
        animationSet.setAnimationListener(new AnimationListener() {
                        
                        @Override
                        public void onAnimationStart(Animation animation) {
                                
                        }
                        
                        @Override
                        public void onAnimationRepeat(Animation animation) {
                                
                        }
                        
                        @Override
                        public void onAnimationEnd(Animation animation) {
                                originImageV.setVisibility(View.INVISIBLE);
                                scaleImageV.setVisibility(View.VISIBLE);
                                originImageV.clearAnimation();
                        }
                });
        
        originImageV.startAnimation(animationSet);
        }
```
点击大图返回动画<br/>
``` java
public static void closeImage(Activity activity, final ImageView originImageV, final ImageView scaleImageV){
                int originX = originImageV.getLeft();
                int originY = originImageV.getTop();
                int scaleX = scaleImageV.getLeft();
                int scaleY = scaleImageV.getTop();
                System.out.println("scaleY: " + scaleY + " bottom: " + scaleImageV.getBottom());
                
                AnimationSet animationSet = new AnimationSet(true);
                
                TranslateAnimation translateAnimation = new TranslateAnimation(
                Animation.ABSOLUTE, scaleX,
                Animation.ABSOLUTE, originX,
                Animation.ABSOLUTE, scaleY - getStatusBarHeight(activity),
                Animation.ABSOLUTE, originY);
        translateAnimation.setDuration(1500);
        animationSet.addAnimation(translateAnimation);

        ScaleAnimation scaleAnimation = new ScaleAnimation((float)scaleImageV.getWidth()/originImageV.getWidth(), 1.0f, 
                        (float)scaleImageV.getWidth()/originImageV.getWidth(), 1.0f,  
                        Animation.ABSOLUTE, 0,
                Animation.ABSOLUTE, 0);
        scaleAnimation.setDuration(1500);
        animationSet.addAnimation(scaleAnimation);
        
        
        animationSet.setFillEnabled(true);
        animationSet.setAnimationListener(new AnimationListener() {
                        
                        @Override
                        public void onAnimationStart(Animation animation) {
                                scaleImageV.setVisibility(View.INVISIBLE);
                                originImageV.setVisibility(View.VISIBLE);
                        }
                        
                        @Override
                        public void onAnimationRepeat(Animation animation) {
                                
                        }
                        
                        @Override
                        public void onAnimationEnd(Animation animation) {
                                originImageV.clearAnimation();
                        }
                });
        
        originImageV.startAnimation(animationSet);
        }
```
下面是获取状态栏高度<br/>
``` java 
/**
        * @Title: getStatusBarHeight 
        * @Description:  
        * @param activity
        * @return int     
         */
        public static int getStatusBarHeight(Activity activity){
                int result = 0;
                   int resourceId = activity.getResources().getIdentifier("status_bar_height", "dimen", "android");
                   if (resourceId > 0) {
                      result = activity.getResources().getDimensionPixelSize(resourceId);
                   }
                   return result;
        }
```
