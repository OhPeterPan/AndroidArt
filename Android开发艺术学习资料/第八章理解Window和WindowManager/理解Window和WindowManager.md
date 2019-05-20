 window
	表示窗口的概念，是一个抽象类，实现类是phoneWindow
	
	创建
		windowManager创建 WindowManager是外界访问Window的入口，Window的具体实现位于WindowManagerService中，WindowManager和WindowManagerService是一个IPC过程。Android中所有的视图都是通过Window来呈现。
	Dialog、Activity、Toast这些视图都是附加到Window上的，Window是View的直接管理者
	单击事件由Window传递给DecorView，然后由DecorView传递给我们的View
	Activity得setContentView底层都是由Window来实现的

Window和WindowManager(android6.0以后由于权限变得严格，需要先申请权限)
	AndroidManifest 文件中添加权限 
		    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
   			<uses-permission android:name="android.permission.SYSTEM_OVERLAY_WINDOW" /> 
	
	示例：
		    if (Build.VERSION.SDK_INT >= 23) {
            if (!Settings.canDrawOverlays(this)) {//判断是否有浮窗权限
                Intent intent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION, Uri.parse("package:" + getPackageName()));
                if (getPackageManager().resolveActivity(intent, PackageManager.MATCH_DEFAULT_ONLY) != null)
                    startActivityForResult(intent, 10);//去开启浮窗权限的界面
            }
        } else {
            Button button = new Button(this);
            button.setText("我是一个window");
            WindowManager.LayoutParams layoutParams = new WindowManager.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT,
                    ViewGroup.LayoutParams.WRAP_CONTENT, 0, 0, PixelFormat.TRANSLUCENT);

            layoutParams.gravity = Gravity.BOTTOM | Gravity.RIGHT;
            layoutParams.x = 100;
            layoutParams.y = 300;
            layoutParams.flags = WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE |
                    WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL |
                    WindowManager.LayoutParams.FLAG_SHOW_WHEN_LOCKED;
            layoutParams.type = WindowManager.LayoutParams.TYPE_SYSTEM_ERROR;
            mWindowManager.addView(button, layoutParams);
        }
	
	重要设置：flags和type  两者必不可少
		flags表示Window的属性，通过这些选项可以控制Window的显示特性
		常用flags：
			FLAG_NOT_FOCUSABLE 表示Window不需要获取焦点，也不需要接受各种输入事件，此标记会同时启用FLAG_NOT_TOUCH_MODAL，最终事件会直接传递给下层具有焦点的Window
			FLAG_NOT_TOUCH_MODAL此模式下，系统会将当前Window区域以外的单击事件传递给底层的Window，当前Window区域以内的单击事件自己处理，一般都需要开启此标记，否则其它Window将无法接受到事件
            FLAG_DIM_BEHIND 窗口以外的区域变暗
			FLAG_SHOW_WALLPAPER  系统壁纸显示在Window的后面 很怪异的一个标志位

		常用type:
			type表示Window的类型，有三种类型：应用Window、子Window、系统Window
			应用Window：对应着一个Activity
			子Window：不能单独存在，必须有一个父Window，比如Dialog
			系统Window：需要声明权限能创建的Window，不如Toast和系统状态栏这些都是系统的Window
			
		Android8.0新增一个type  TYPE_APPLICATION_OVERLAY
			显示在
				TYPE_PHONE
				TYPE_PRIORITY_PHONE
				TYPE_SYSTEM_ALERT
				TYPE_SYSTEM_OVERLAY
				TYPE_SYSTEM_ERROR
				…
				这些窗口之上。

	Window是分层的：层级大的会覆盖在层级小的Window的上面
		三类Window中：应用Window层级范围 1~99  子Window层级范围1000~1999  系统Window的层级范围2000~2999
		这些层级范围对应WindowManager.LayoutParams的type参数，想要Window位于所有Window的最顶层，采用较大的层级即可

WindowManager extends ViewManager
	常用的只有三个方法：添加View 、更新View、删除View 
	如果想要是Window可以滑动，首先给View设置onTouchListener，然后在onTouch中不断更新View的位置即可
	示例：
		    button.setOnTouchListener(new View.OnTouchListener() {
                    @Override
                    public boolean onTouch(View view, MotionEvent motionEvent) {
                        if (motionEvent.getAction() == MotionEvent.ACTION_MOVE) {
                            layoutParams.x = (int) motionEvent.getRawX();
                            layoutParams.y = (int) motionEvent.getRawY();
                            mWindowManager.updateViewLayout(button, layoutParams);
                        }
                        return false;
                    }
                });

Window的内部机制
  1、Window的添加过程
	抽象概念  对应一个View和一个ViewRootImpl，
	Window和View通过ViewRootImpl来建立联系，因此Window并不是实际存在的，它是以View的形式存在的(从WindowManager的三个主要方法都是针对View的操作可以看出，View才是Window存在的实体)，无法直接访问Window，想要访问只能通过WindowManager
	
	Window的添加过程  WindowManager是一个借口 实现类WindowManagerImpl
		WindowManager的addView决定 在此方法中可以看到 全部交由WindowManagerGlobal来处理

	步骤：
		1、检查参数是否合法，如果是子Window那么还需要调整一些布局参数

		2、创建ViewRootImpl并将View添加到列表中

		3、通过ViewRootImpl来更新界面并完成Window的添加过程
			这步由ViewRootImpl的setView方法来完成，内部调用requestLayout完成异步刷新请求，requestLayout方法内部调用scheduleTraversals来完成View的绘制(scheduleTraversals是View绘制的入口)，接着通过WindowSession最终完成Window的添加过程，紧接着通过WindowSession最终来完成Window的添加过程。window的添加过程是一个IPC调用， mWindowSession.addToDisplay(mWindow, mSeq, mWindowAttributes,
                            getHostVisibility(), mDisplay.getDisplayId(),
                            mAttachInfo.mContentInsets, mInputChannel);最终调用WindowManagerService添加Window
		在WindowManagerService内部会为每一个应用保留单独的Session

  2、Window的删除过程  removeView异步删除  removeViewImmediate同步删除  (immediate 立即、直接) 详情看图
	WindowManagerImpl->WindowManagerGlobal

  3、Window的更新过程
		WindowManagerImpl->WindowManagerGlobal
		将要更新的Layoutparams替换原来的，然后更新ViewRootImpl中的layoutParams，使用root的setLayoutParams来实现，最后通过scheduleTraversals来对View重新布局，包括测量、布局、重绘三个过程，除了对View本身的重绘外，还会调用mWindowSession来更新Window，最后还是会交给WindowManagerService的relayout来具体实现界面的更新，同样是一个IPC过程

Window的创建过程
	View是Android中视图的呈现方式，但是View不能单独存在，必须附着在Window这个抽象的概念上面，有视图的地方就有Window。
	提供视图的地方：Activity、Dialog、Toast。除了这些还有一些依托Window而实现的视图，比如PopUpWindow、菜单。
	
	有视图的地方就有Window！有视图的地方就有Window！！有视图的地方就有视图！！！ 重要的事情说三遍

	Activity的Window的创建过程
		ActivityThread  
	Dialog的Window创建过程
	Toast的Window创建过程

			
			
