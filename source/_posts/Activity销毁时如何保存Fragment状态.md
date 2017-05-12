---
title: Activity销毁时如何保存Fragment状态
date: 2017-04-25 19:25:23
tags: 
---
转载请标明出处： 
http://blog.csdn.net/a940659387/article/details/50730076

原文：http://emuneee.com/blog/2013/01/07/saving-fragment-states/
在Android 3.0(SDK 11)以后，Android 出现了一个伟大的功能：如何在你的App中保存和还原你的Framgent数据。
<!-- more -->
### 它是什么 ###

	我再我下一个App开发中期的时候参考Android API时,偶然发现  FragmentManager.putFragment(Bundle, String, Fragment) and FragmentManager.getFragment(Bundle, String) 这两个方法。它们的作用？
	
	putFragment
将一个Fragment的引用传递到Bundle中。这个Bundle可以一直保存它的数据，以后通过调用
	[getFragment(Bundle, String)](http://developer.android.com/reference/android/app/FragmentManager.html#getFragment%28android.os.Bundle,%20java.lang.String%29)方法得到一个一样的Fragmet 实例。

	getFragment

取回一个先前通过 [putFragment(Bundle, String, Fragment)](http://developer.android.com/reference/android/app/FragmentManager.html#putFragment%28android.os.Bundle,%20java.lang.String,%20android.app.Fragment%29).方法存储的fragment实例。

	根据我的理解，当你把当前的Fragment放置到bundle中，他会保存一个的指针指向这个fragment的地址;当你调用getFragment()方法时，它将返回你通过putFragment保存的指针对应的Fragment实例。


你也可以通过使用FramgentManager APIs 去保存一个fragment 对应的指针（我感觉叫引用更好） 到bundle中 在以后用到时取出。它也允许你实现这个指针的存储跟取出（还原）。 

		---Diane Hackborn ,Android 架构师。  [Source](https://groups.google.com/forum/?fromgroups=#!topic/android-developers/NBlMJnMaGbo)

### 如何去使用它 ###
一个使用这个API的方法是在我的这个Acticity的创建和销毁的时候。自fragment出现以来，我通常用两个Activity来使用它，一个是我的activity,另一个用来设置Fragment。我再我的activity中管理fragment.

**putFragment**

	在activity的 onSaveInstanceState 像这样使用：

```

@Override
protected void onSaveInstanceState(Bundle outState) {
   FragmentManager manager = getFragmentManager();
   manager.putFragment(outState, MyFragment.TAG, mMyFragment);
}

```
MyFragment 是一个在我应用中创建的Fragment，这个bundle,outState,只是一个简单的bunlde,它将存储你fragment(即MyFragment)对应的引用（指针）。 MyFragment.TAG 是你的fragment指针以后会用到的key值。

**GetFragment**

我有一个自定义方法（instantiateFragments） ，它会做一到两件事情：当我的应用是冷启动时抓取我的fragment实例 或者 获得现在的MyFragment内容。

```
private void instantiateFragments(Bundle inState) {
   FragmentManager manager = getFragmentManager();
   FragmentTransaction transaction = manager.beginTransaction();

   if (inState != null) {
      mMyFragment = (MyFragment) manager.getFragment(inState, MyFragment.TAG);
   } else {
      mMyFragment = new MyFragment();
      transaction.add(R.id.fragment, mMyFragment, MyFragment.TAG);
      transaction.commit();
   }
}
```

我在我的activity中调用instantiateFragments 这个方法，onRestoreInstanceState传入的参数为null时（onCreate 方法通过冷启动触发），当通过热启动触发时，传入的参数为bundle.

```
@Override
protected void onRestoreInstanceState(Bundle inState) {
   instantiateFragments(inState);
}
```

当用户旋转设备时，Android 将销毁Activity,并在销毁前触发onSaveInstanceState，允许开发者来
保存数据。一旦activity被重新启动，这个OS 将触发onRestoreInstanceState 恢复开发者旋转前的保存的状态。

我鼓励你继续更深调查Android框架。因为所有类型都有细微的差别,他们将简化Android应用程序开发工作。



	