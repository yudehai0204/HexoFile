---
title: Android适配6.0，全局Dialog终极解决方案
date: 2017-04-28 15:18:52
tags: Android
---
6.0推出之后，部分手机完全限制了全局Toast弹出类型。so 撸主
(http://blog.csdn.net/a940659387/article/details/50152561)  
这个文章也该光荣退休了。
**but 想要看这篇文章的童鞋应该先看撸主的前一篇文章，why?  because 你不看辣个就看不懂这个。**
<!-- more -->
### Application.ActivityLifecycleCallbacks ###

顾明思议，肯定是用在Application中的 。 关于不知道怎么配置Application的童鞋可自行谷歌。

再瞅，带着callback must 是一个接口。  没错  就是一个接口 一个在application中实现用来监听activity

生命周期触发步骤的东东。

#### 没图说个蛋蛋 先上图片 ####
![这里写图片描述](http://img.blog.csdn.net/20160909141533953)

#### 方法 ####

```
  @Override
    public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
        if(activity.getParent()!=null){
            NewContext = activity.getParent();
        }else
            NewContext = activity;
    }

    @Override
    public void onActivityStarted(Activity activity) {
        if(activity.getParent()!=null){
            NewContext = activity.getParent();
        }else
            NewContext = activity;
    }

    @Override
    public void onActivityResumed(Activity activity) {
        if(activity.getParent()!=null){
            NewContext = activity.getParent();
        }else
            NewContext = activity;
    }

    @Override
    public void onActivityPaused(Activity activity) {
            ToastUtils.getInstanse().cancel();
    }

    @Override
    public void onActivityStopped(Activity activity) {

    }

    @Override
    public void onActivitySaveInstanceState(Activity activity, Bundle outState) {
    }

    @Override
    public void onActivityDestroyed(Activity activity) {

    }
```
这个NewContext 就是我们的主要东东 记录当前用户停留在哪一个页面 。简单来说就是记录dialog弹出时

应该依赖那个activity.

soso 为了适配新方法 我们在全局的service中也做了一些小小的改动哦 。不懂怎么写一个全局弹出

dialog的service的童鞋请看这个文章；
[http://blog.csdn.net/a940659387/article/details/50152561]


本撸主还是粘出代码供your 一览吧。


```
public class CommonDialogService extends Service implements CommonDialogListener{
	@Override
	public IBinder onBind(Intent intent) {
		return null;
	}
	/**this is TV*/
	private static TextView tv;
	private static Dialog dialog;
	/**判断是否已经new Dialog*/
	private void showDialog(int iscancle){

		if(dialog==null){
		dialog = new Dialog(CommonApplication.NewContext, R.style.MyDialogStyle);
		View view = LayoutInflater.from(this).inflate(R.layout.progressbar_item,
				null);
		dialog.setContentView(view);
		tv = (TextView) view.findViewById(R.id.mylaodint_text_id);
		ImageView progressImageView = (ImageView) view
				.findViewById(R.id.myloading_image_id);
		AnimationDrawable animationDrawable = (AnimationDrawable) progressImageView
				.getDrawable();
		animationDrawable.start();
//		WindowManager.LayoutParams windows = dialog.getWindow().getAttributes();
//			windows.type=WindowManager.LayoutParams.TYPE_TOAST;
//			dialog.getWindow().setAttributes(windows);
//		dialog.getWindow().setType(WindowManager.LayoutParams.TYPE_TOAST);
			if(iscancle==1){
				dialog.setCanceledOnTouchOutside(true);
			}else{
				dialog.setCanceledOnTouchOutside(false);
			}

			dialog.setOnKeyListener(new DialogInterface.OnKeyListener() {
				@Override
				public boolean onKey(DialogInterface dialog, int keyCode, KeyEvent event) {
					dialog.dismiss();
					dialog=null;
					return false;
				}
			});
			dialog.setOnDismissListener(new DialogInterface.OnDismissListener() {
				@Override
				public void onDismiss(DialogInterface dialog) {
					dialog =null;
				}
			});
		}

		if(dialog!=null&&!dialog.isShowing()){
			dialog.show();
			WindowManager.LayoutParams lp = dialog.getWindow()
					.getAttributes();
			lp.width = ConstantsYiBaiSong.WinWidth/ 3;
			lp.height = LayoutParams.WRAP_CONTENT;
			dialog.getWindow().setAttributes(lp);
			}
		}
	
	public void onCreate() {
		/**this将此service与工具类绑定*/
		ToastUtils.getInstanse().setListener(this);
	}

	
	/**showDialog*/
	@Override
	public void show() {
		showDialog(1);
		dialog.setCancelable(true);
		tv.setText("请稍后...");

	}

	/**cancleDialog*/
	@Override
	public void cancle() {
		if(null!=dialog){
			dialog.dismiss();
			dialog = null;
		}
	}

	/**show have custom Text dialog*/
	@Override
	public void showstr(String str) {
		showDialog(1);
		dialog.setCancelable(true);
		tv.setText(str);

	}

	/**show uncancle's dialog */
	@Override
	public void showunCancle() {
		showDialog(2);
		dialog.setCanceledOnTouchOutside(false);
		dialog.setCancelable(false);
		tv.setText("请稍后...");
	}

	@Override
	public void setStr(String str) {
		if(dialog==null){
			return;
		}
		if(dialog.isShowing()){
//			Log.e("TAG", "str====:" + str);
			tv.setText(str+"");
		}
	}

	@Override
	public void showDismissListener(final ToastUtils.DisMisCallBack callBack) {
		if(CommonApplication.NewContext!=null&&dialog==null){
			dialog = new Dialog(CommonApplication.NewContext, R.style.MyDialogStyle);
			View view = LayoutInflater.from(this).inflate(R.layout.progressbar_item,
					null);
			dialog.setContentView(view);

			tv = (TextView) view.findViewById(R.id.mylaodint_text_id);
			ImageView progressImageView = (ImageView) view
					.findViewById(R.id.myloading_image_id);

			AnimationDrawable animationDrawable = (AnimationDrawable) progressImageView
					.getDrawable();
			animationDrawable.start();
			dialog.setOnDismissListener(new DialogInterface.OnDismissListener() {
				@Override
				public void onDismiss(DialogInterface dialog) {
					callBack.disMisCallBack();
				}});
//		WindowManager.LayoutParams windows = dialog.getWindow().getAttributes();
//			windows.type=WindowManager.LayoutParams.TYPE_TOAST;
//			dialog.getWindow().setAttributes(windows);
//		dialog.getWindow().setType(WindowManager.LayoutParams.TYPE_TOAST);
//			if(iscancle==1){
//				dialog.setCanceledOnTouchOutside(true);
//			}else{
//				dialog.setCanceledOnTouchOutside(false);
//			}
		}
		dialog.setCanceledOnTouchOutside(false);
		if(dialog!=null&&!dialog.isShowing()){
			dialog.show();
			WindowManager.LayoutParams lp = dialog.getWindow()
					.getAttributes();
			lp.width = ConstantsYiBaiSong.WinWidth/ 3;
			lp.height = LayoutParams.WRAP_CONTENT;
			dialog.getWindow().setAttributes(lp);
		}

	}

	;

	@Override
	public void onDestroy() {
		super.onDestroy();
		if(null!=dialog &&dialog.isShowing()){
			dialog.cancel();
		}
	}
}
```
细心的童鞋可能已经发现 有些代码已经被注释了  。  因为么 不注释 6.0就会啪啪啪一直蹦。 you know .


话到这里  撸主已经没有太多的话想说了。so女生压底图一张，然后上demo;

![这里写图片描述](http://img.blog.csdn.net/20160909141659424)


git地址：[全局Dialog适配6.0](https://github.com/yudehai0204/ApplicationDialog)


