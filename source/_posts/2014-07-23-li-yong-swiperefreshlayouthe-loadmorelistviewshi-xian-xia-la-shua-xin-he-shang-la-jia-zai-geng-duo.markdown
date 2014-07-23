---
layout: post
title: "利用SwipeRefreshLayout和LoadMoreListView实现下拉刷新和上拉加载更多"
date: 2014-07-23 20:49
comments: true
categories: SwipeRefreshLayout,LoadMoreListView,android,listview 
---
<p>Google自己的下拉刷新组件SwipeRefreshLayout非常nice。和LoadMoreListView结合使用，可以非常容易的实现常用的下拉刷新和上拉加载更多功能。以下是一个简单的Demo。</p>
<p>布局如下:</>
``` xml
	<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    	    xmlns:tools="http://schemas.android.com/tools"
   	    android:id="@+id/container"
    	    android:layout_width="match_parent"
            android:layout_height="match_parent"
    	    tools:context="com.cloay.stunningrefreshloadmoredemo.MainActivity"
    	    tools:ignore="MergeRootFrame" >
   	    <android.support.v4.widget.SwipeRefreshLayout  
    		android:id="@+id/swipe_refresh"  
   		android:layout_width="match_parent"  
   		android:layout_height="match_parent" >  
  
                <com.cloay.stunningrefreshloadmoredemo.widgets.LoadMoreListView  
                     android:id="@+id/listview"  
       		     android:layout_width="match_parent"  
       		     android:layout_height="match_parent" >  
    		</com.cloay.stunningrefreshloadmoredemo.widgets.LoadMoreListView>  
	    </android.support.v4.widget.SwipeRefreshLayout>
	</FrameLayout>
```
<p>主要代码</p>
``` java
public class MainActivity extends ActionBarActivity implements OnRefreshListener, OnLoadMoreListener{

    private SwipeRefreshLayout swipeLayout;  
    private LoadMoreListView listView;  
    private MyAdapter adapter;
    
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

		swipeLayout = (SwipeRefreshLayout) this.findViewById(R.id.swipe_refresh);  
        swipeLayout.setOnRefreshListener(this);  
          
        // set style  
        swipeLayout.setColorScheme(android.R.color.holo_red_light, android.R.color.holo_green_light,  
                android.R.color.holo_blue_bright, android.R.color.holo_orange_light);
        listView = (LoadMoreListView) this.findViewById(R.id.listview);
        listView.setOnLoadMoreListener(this);
        adapter = new MyAdapter();
        listView.setAdapter(adapter);
	}

	@Override
	public void onRefresh() {
		new AsyncTask<Void, Void, Void>() {

			@Override
			protected Void doInBackground(Void... params) {
				try {
					Thread.sleep(3*1000); //sleep 3 seconds
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				return null;
			}

			@Override
			protected void onPostExecute(Void result) {
				adapter.count = 15;
				adapter.notifyDataSetChanged();
				swipeLayout.setRefreshing(false);
				listView.setCanLoadMore(adapter.count < 45);
				super.onPostExecute(result);
			}

		}.execute();
	}

	@Override
	public void onLoadMore() {

		new AsyncTask<Void, Void, Void>() {

			@Override
			protected Void doInBackground(Void... params) {
				try {
					Thread.sleep(3*1000);	//sleep 3 seconds
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				return null;
			}

			@Override
			protected void onPostExecute(Void result) {
				adapter.count += 15;
				adapter.notifyDataSetChanged();
				listView.setCanLoadMore(adapter.count < 45);
				listView.onLoadMoreComplete();
				super.onPostExecute(result);
			}

		}.execute();
	}


	private class MyAdapter extends BaseAdapter{
		public int count = 15;
		@Override
		public int getCount() {
			return count;
		}

		@Override
		public Object getItem(int position) {
			return null;
		}

		@Override
		public long getItemId(int position) {
			return 0;
		}

		@Override
		public View getView(int position, View convertView, ViewGroup parent) {
			ViewHolder holder = null;
			if (convertView == null) {
				holder = new ViewHolder();
				convertView = LayoutInflater.from(MainActivity.this).inflate(android.R.layout.simple_list_item_1, null);
				holder.textV = (TextView) convertView.findViewById(android.R.id.text1);
				convertView.setTag(holder);
			}else{
				holder = (ViewHolder) convertView.getTag();
			}
			holder.textV.setText("This is " + (position + 1) + " line.");
			return convertView;
		}

		private class ViewHolder{
			TextView textV;
		}

	}

}
```
<p>效果还不错，小伙伴可以试试哦！Demo已上传Github: <a href="https://github.com/cloay/StunningPullRefreshAndLoadMore">StunningPullRefreshAndLoadMoreDemo</a></p>
