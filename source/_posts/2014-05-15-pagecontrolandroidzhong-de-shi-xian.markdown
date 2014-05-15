---
layout: post
title: "pagecontrolandroid中的实现"
date: 2014-05-15 22:46
comments: true
categories: pagecontrol android 
---
<p>Android中没有PageControl控件 自己实现如下：</p>
``` java
public class PageControl extends LinearLayout {
	private int count = 0;
	private int selected = 0;
	private int nomal_icon_id = R.drawable.pagecontro_icon_normal;
	private int foucsed_icon_id = R.drawable.pagecontro_icon_foucsed;
	private Context context;
	public PageControl(Context context) {
		super(context);
		this.context = context;
	}
	
	public PageControl(Context context, AttributeSet attrs){
		super(context, attrs);
		this.context = context;
		generateView();
	}
	
	public PageControl(Context context, int count) {
		super(context);
		this.count = count;
		this.context = context;
		generateView();
	}

	private void generateView(){
		removeAllViews();
		LayoutParams params =  new LayoutParams(context.getResources().getDimensionPixelSize(R.dimen.home_banner_pagecontrol_shape), 
				context.getResources().getDimensionPixelSize(R.dimen.home_banner_pagecontrol_shape));
		params.setMargins(10, 0, 0, 10);
		for(int i = 0; i < count ; i++){
			ImageView indicatorV = new ImageView(context);
			if (i == selected) {
				indicatorV.setImageResource(foucsed_icon_id);
			}else{
				indicatorV.setImageResource(nomal_icon_id);
			}
			addView(indicatorV, params);
		}
	}
	
	public void setCount(int count){
		this.count = count;
		generateView();
	}
	
	public void setSelected(int index){
		this.selected = index;
		generateView();
	}
	
	public void setIconId(int nomalId, int foucsedId){
		this.nomal_icon_id = nomalId;
		this.foucsed_icon_id = foucsedId;
	}
}
```
直接在布局中定义就可以使用了。
