---
title: RecyclerView
date: 2021-04-29 09:23:54
tags: Android
---
待写
<!--more-->
# 缓存机制（四层）
- Scrap：屏幕内缓存数据，数据可直接复用
- Cache：刚刚移出屏幕的缓存数据，默认大小2个，数据可直接复用
- ViewCacheExtension：留给开发者自定义的缓存，慎用
- RecycledViewPool：当Cache缓存满了以后会根据FIFO（先进先出）的规则把Cache先缓存进去的ViewHolder移出并缓存到RecycledViewPool中，RecycledViewPool默认的缓存数量是5个。数据会被重置

## rcyclerView滑动时，view进来和出去时的流程
- 假设页面可以容纳7条数据，7条数据会依次调用onCreateViewHolder和onBindViewHolder
- 往下滑一条（position=7），那么会把position=0的数据放到mCacheViews中。此时mCacheViews缓存区数量为1，mRecyclerPool数量为0。然后新出现的position=7的数据通过postion在mCacheViews中找不到对应的ViewHolder，通过itemtype也在mRecyclerPool中找不到对应的数据，所以会调用onCreateViewHolder和onBindViewHolder方法
- 再往下滑一条数据（position=8），如上
- 再往下滑一条数据（position=9），position=2的数据会放到mCacheViews中，但是由于mCacheViews缓存区默认容量为2，所以position=0的数据会被清空数据然后放到mRecyclerPool缓存池中。而新出现的position=9数据由于在mRecyclerPool中还是找不到相应type的ViewHolder，所以还是会走onCreateViewHolder和onBindViewHolder方法。所以此时mCacheViews缓存区数量为2，mRecyclerPool数量为1
- 再往下滑一条数据（position=10），这时候由于可以在mRecyclerPool中找到相同viewtype的ViewHolder了。所以就直接复用了，并调用onBindViewHolder方法绑定数据


# 滑动冲突解决
- 自定义父recyclerView并重写onInterceptTouchEvent()方法，不进行拦截
- 子布局重写dispatchTouchEvent()方法，通知通知父层ViewGroup不要拦截点击事件，通过requestDisallowInterceptTouchEvent方法
- 通过事件分发规则我们知道，OnTouchListener优先级很高，可以通过这个来告诉父布局，不要拦截我的事件
```
   holder.recyclerView.setOnTouchListener { v, event ->
            when(event.action){
                //当用户按下的时候，我们告诉父组件，不要拦截我的事件（这个时候子组件是可以正常响应事件的），拿起之后就会告诉父组件可以阻止。
                MotionEvent.ACTION_DOWN,MotionEvent.ACTION_MOVE -> v.parent.requestDisallowInterceptTouchEvent(true)
                MotionEvent.ACTION_UP -> v.parent.requestDisallowInterceptTouchEvent(false)
            }
            false}
```
