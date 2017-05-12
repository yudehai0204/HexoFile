---
title: android 自动播放Banner
date: 2017-04-28 15:28:17
tags: Android
---
### 先看一下效果图 ###
![这里写图片描述](http://img.blog.csdn.net/20160115104236488)

#### 支持本地图片以及网络图片or本地网络混合。 ####
<!-- more -->
使用方式：

```
<com.jalen.autobanner.BannerView
        android:id="@+id/banner"
        android:layout_width="match_parent"
        android:layout_height="230dip">
</com.jalen.autobanner.BannerView>
```

核心的实现代码

```
 int length = mList.size();
        View view = LayoutInflater.from(mContext).inflate(R.layout.banner_view,this,true);
        LinearLayout ll = (LinearLayout) view.findViewById(R.id.ll_points);
        vp= (ViewPager) view.findViewById(R.id.vp);
        ll.removeAllViews();
        LinearLayout.LayoutParams ll_parmas = new LinearLayout.LayoutParams(LayoutParams.WRAP_CONTENT,LayoutParams.WRAP_CONTENT);
        ll_parmas.leftMargin=5;
        ll_parmas.rightMargin=5;
        for(int i=0;i<length;i++){
            ImageView img = new ImageView(mContext);
            img.setLayoutParams(ll_parmas);
            if(i==0){
                img.setImageResource(R.mipmap.dot_focus);
            }else{
                img.setImageResource(R.mipmap.dot_blur);
            }
            ll.addView(img);
            mImgs.add(img);

            final ImageView imgforview = new ImageView(mContext);
            imgforview.setOnClickListener(this);
            imgforview.setScaleType(ImageView.ScaleType.FIT_XY);
            if(mList.get(i).getType()==0){//本地图片
                imgforview.setImageResource(mList.get(i).getDrawableforint());

            }else{//网络
                Glide.with(mContext).load(mList.get(i).getDrawableforurl()).diskCacheStrategy(DiskCacheStrategy.ALL).into(imgforview);
//                Glide.with(mContext).load(mList.get(i).getDrawableforurl()).listener(new RequestListener<String, GlideDrawable>() {
//                    @Override
//                    public boolean onException(Exception e, String model, Target<GlideDrawable> target, boolean isFirstResource) {
//                        Log.d("yu","Faile:"+e.toString());
//                        return false;
//                    }
//
//                    @Override
//                    public boolean onResourceReady(GlideDrawable resource, String model, Target<GlideDrawable> target, boolean isFromMemoryCache, boolean isFirstResource) {
//                            imgforview.setImageDrawable(resource);
//                        return false;
//                    }
//                }).into(imgforview);
//                Log.d("yu","url: "+mList.get(i).getDrawableforurl());
            }
            mViews.add(imgforview);
        }

        vp.setAdapter(new MyAdapter());
        vp.addOnPageChangeListener(onPageChange);
```

自动轮播利用的是handler的postdelay方法。

```
private Runnable task = new Runnable() {
        @Override
        public void run() {
            if(isAuto){
               currentItem = currentItem%(mViews.size());
//                Log.d("yu","runalbe "+currentItem);
                if(currentItem==0){
                    vp.setCurrentItem(currentItem,false);
                }else{
                    vp.setCurrentItem(currentItem);
                }
                currentItem++;
                mHandle.postDelayed(task,delaytime);
            }else{
                mHandle.postDelayed(task,delaytime);
            }
        }
    };
```

利用isAuto判断是否正在自动轮播 如果为false 不自动切换item。isAuto赋值操作位于OnPageChangeListener的onPageScrollStateChanged方法中：

```
 public void onPageScrollStateChanged(int state) {

            switch (state){
                case ViewPager.SCROLL_STATE_IDLE://用户什么都没有操作
                    isAuto=true;
                    currentItem = vp.getCurrentItem();
//                    Log.d("yu","IDLE"+currentItem);
//                    if(vp.getCurrentItem()==mViews.size()){
//                        vp.setCurrentItem(0,false);
//                    }
                    break;
                case ViewPager.SCROLL_STATE_DRAGGING://正在滑动
                    isAuto =false;
                    break;
                case ViewPager.SCROLL_STATE_SETTLING://滑动结束
                    isAuto=true;
                    break;

            }
        }
```

当状态为SCROLL_STATE_DRAGGING时  说明用户正在操作 ，看下源码中的解释：

```
 /**
     * Indicates that the pager is in an idle, settled state. The current page
     * is fully in view and no animation is in progress.
     */
    public static final int SCROLL_STATE_IDLE = 0;

    /**
     * Indicates that the pager is currently being dragged by the user.
     */
    public static final int SCROLL_STATE_DRAGGING = 1;

    /**
     * Indicates that the pager is in the process of settling to a final position.
     */
    public static final int SCROLL_STATE_SETTLING = 2;
```

大致意思呢就是 0代码没有任何操作。1页面正在被用户拖动。2代表成功切换至下一页面。


### 源码地址 ###

[点击跳转](https://github.com/yudehai0204/autoBanner)

有问题随时欢迎Issues;
