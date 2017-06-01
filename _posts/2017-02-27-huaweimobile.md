---
layout: post
title: 华为手机屏幕适配
---

{{ page.title }}
================

# 一、概述
在项目中,测试发现在一些华为手机的屏幕适配上出现了问题,主要是因为华为Mate等一些系列的手机有一个虚拟按键的设计.当这些虚拟按键由用户手势滑出,或默认显示的话,就会遮挡我们本身的应用布局.比如欢迎界面过后是四个Fragment,那么底部的四个tab就会被虚拟的导航栏遮住,非常难看.

![](http://img.blog.csdn.net/20170227154857554?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3poc2V1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

当然,欢迎页的图片适配也同样会出现问题.
Google后得出第一个问题的解决方案.第二个图片的问题则用自己摸索的方式解决,当然也非常简单.

#二、布局由于虚拟按键导致导航栏顶上去的解决方法
在我们的项目中加载Fragment的MainActivity,以及其他一般的Activity继承的BaseActivity中的onCreate方法中添加如下代码:

    if (AndroidWorkaround.checkDeviceHasNavigationBar(this)) {
       AndroidWorkaround.assistActivity(findViewById(android.R.id.content));
    }

其中AndroidWorkaround使我们为了解决该问题而封装的类,也可以看作是一个特定的工具类:

  	/**
	* 解决底部屏幕按键适配
 	* Created by Mercury on 2016/10/25.
	*/
	public class AndroidWorkaround {

    	public static void assistActivity(View content) {
	        new AndroidWorkaround(content);
	    }
	
	    private View mChildOfContent;
	    private int usableHeightPrevious;
	    private ViewGroup.LayoutParams frameLayoutParams;
	
	    private AndroidWorkaround(View content) {
	        mChildOfContent = content;
	        mChildOfContent.getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
	            public void onGlobalLayout() {
	                possiblyResizeChildOfContent();
	            }
	        });
	        frameLayoutParams = mChildOfContent.getLayoutParams();
	    }
	
	    private void possiblyResizeChildOfContent() {
	        int usableHeightNow = computeUsableHeight();
	        if (usableHeightNow != usableHeightPrevious) {
	
	            frameLayoutParams.height = usableHeightNow;
	            mChildOfContent.requestLayout();
	            usableHeightPrevious = usableHeightNow;
	        }
	    }
	
	    private int computeUsableHeight() {
	        Rect r = new Rect();
	        mChildOfContent.getWindowVisibleDisplayFrame(r);
	        return (r.bottom);
	    }
	
	    public static boolean checkDeviceHasNavigationBar(Context context) {
	        boolean hasNavigationBar = false;
	        Resources rs = context.getResources();
	        int id = rs.getIdentifier("config_showNavigationBar", "bool", "android");
	        if (id > 0) {
	            hasNavigationBar = rs.getBoolean(id);
	        }
	        try {
	            Class systemPropertiesClass = Class.forName("android.os.SystemProperties");
	            Method m = systemPropertiesClass.getMethod("get", String.class);
	            String navBarOverride = (String) m.invoke(systemPropertiesClass, "qemu.hw.mainkeys");
	            if ("1".equals(navBarOverride)) {
	                hasNavigationBar = false;
	            } else if ("0".equals(navBarOverride)) {
	                hasNavigationBar = true;
	            }
	        } catch (Exception e) {
	
	        }
	        return hasNavigationBar;
	
	    }

	}

重新测试，发现无论是否弹出虚拟按键，都不会再次遮挡tab按钮。

#三、原理
上面的代码需要在setContentView后面执行。其最初的解决方案是stackoverflow上有人为了适配软键盘在全屏下的布局问题。
开始先判断该设备上是否存在导航栏。为什么用findViewById(android.R.id.content)呢？因为android.R.id.content这个id代表的就是所在页面的根布局，而并不需要特别指定一个id给该布局。可以通过调用系统API返回的结果，也可以通过判断该手机是否为华为手机，操作系统属于哪种类型来来判断。
一旦确定该设备存在导航栏，将对该布局进行重新测量。首先mChildOfContent得到其视图树，对全局高度实现监听。
###### OnGlobalLayoutListener 是ViewTreeObserver的内部类，当一个视图树的布局发生改变时，可以被ViewTreeObserver监听到，这是一个注册监听视图树的观察者(observer)，在视图树的全局事件改变时得到通知。ViewTreeObserver不能直接实例化，而是通过getViewTreeObserver()获得。
接着得到视图目前可用的总高度，将其赋值给mChildOfContent的布局高度。调用requestLayout，让mChildOfContent要求自己的parent view对自己重新设置位置。

#四、全屏图片的适配
解决了布局的问题，再来看欢迎页启动时候全屏图片的适配问题。发现该方法对于图片不适用。如下图，当虚拟按键弹出时，图片照样被遮挡了底部的一小部分。
![](http://img.blog.csdn.net/20170227154932937?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3poc2V1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
如果隐藏虚拟按键，图片大小恢复正常
![](http://img.blog.csdn.net/20170227154949734?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3poc2V1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
仔细想想，对于一个ImageView直接占据一个layout的情况，是没有必要再去写一些代码进行适配的。到布局里一看，发现ImageView的属性 android:scaleType="centerCrop"
将其改为 android:scaleType="fitXY"就可以解决了。这样图片可能高度会随着虚拟键的弹出而压缩，但是很好的适配了布局高度的变化而不会被遮挡。

关于scaleType的详细介绍，留待其他文章里再探讨。