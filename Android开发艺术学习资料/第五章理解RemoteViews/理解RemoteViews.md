RemoteViews
	远程View，表示的是一个View结构，可以在其它进程中显示，由于它在其他进程中显示，为了能够更新他的界面，RemoteViews提供了一个基础操作用于跨进程更新它的界面。为了跨进程更新界面，RemoteViews提供了一系列的set方法，这些方法只是View全部方法的子集，另外RmoteViews并不是支持所有的View。

	通知和桌面小程序的界面都运行在其他的进程中(确切的说是在system的进程中)

	使用场景：通知栏和桌面小部件


RemoteView的应用
	RemoteViews在通知栏中的使用
	RemoteViews用于桌面小组件(用到时来看)
		
PendingIntent
	延迟意图  在某一时刻触发  
	适应send或者cancel发送或者取消特定的intent

	支持三种待定意图：
		1、Activity
		2、Broadcast
		3、Service

	PendingIntent匹配规则：
		如果他们内部的Intent和requestCode都相同，则证明两个PendingIntent是相同的

	Intent匹配规则：
		ComponentName和intent-filter相同，那么两个Intent是相同的，而Intent的Extras是不参与匹配过程的，就是Extras不一致，只要componentName和intent-filter是相同的，那么两个Intent就是相同的

	PendingIntent的flags：
		1、FLAG_ONE_SHOT：
				当前描述的PendingIntend只能被使用一次，然后就会自动的cancel，如果后续还有相同的PendingIntent，那么他们的send方法就会调用失败，。对于通知栏消息来说，若采用这个标记位，那么同类的通知只会使用一次，后续的通知点击后将无法打开
		2、FLAG_NO_CREATE：
				当前描述的PendingIntent不会去主动创建，如果当前PendingIntent之前不存在，那么getActivity、getBroadcast、getService方法会直接返回null，即获取PendingIntent就会失败，不常用
		3、FLAG_CANCEL_CURRENT:
				当前描述的PendingIntent如果已经存在，那么他们都会被cancel，然后系统会创建一个新的PendingIntent，对于通知栏来说，那些被cancel的消息点击后无法打开
		4、FLAG_UPDATE_CURRENT：
				当前描述的PendingIntent如果已经存在，那么它们都会被更新，即他们的Intent中的Extra会被替换成新的
	PendingIntent标志位理解：
		如果通知中的id不同时，而且PendingIntent不匹配时，这些PendingIntent互不干扰
		当PendingIntent匹配时，标记位就会发出作用。
			FLAG_ONE_SHOT  后续通知中的PendingIntent都会与第一条通知保持一致，点击其中一个通知后，其他的通知就不再响应点击事件
			FLAG_CANCEL_CURRENT  只有最新的通知可以打开，其他的均打不开
			FLAG_UPDATE_CURRENT 之前弹出的所有通知中的PendingIntent都会被更新，最终于最新的一条通知保持一致，包括其中的Extras，并且这些通知都是可以打开的

RemoteView的内部机制
	在其它进程中显示并更新View界面
	
	支持View类型如下：
		Layout ： Framelayout、LinearLayout 、 RelativeLayout 、 GridLayout
		View   ： AnalogClock、Button、Chronometer、ImageButton、ImageView、ProgressBar、TextView、ViewFlipper、		      ListView、GridVew、StackView、AdapterViewFlipper、ViewStub
	不支持他们的子类以及其他View类型，不支持自定义View
		
	RemoteViews主要用于通知和桌面小部件中，通过NotificationManager和AppWidgetManager来管理，而NotificationManager和AppWidgetManager通过Binder分别和SystemServer进程中的NotificationManagerService以及AppWidgetManagerService进行通讯。由此可见，通知栏和桌面小部件中的布局文件实际上是在NotificationManagerService以及AppWidgetService中被加载的，而他们运行在系统的SystemServer中，这就和我们的进程构成了跨进程通信的场景

	RemoteViews会通过Binder传递到SystemServer进程，RemoteViews实现了parceable接口，因此可以实现跨进程传输，系统会通过RemoteViews中的包名去加载该应用的资源，然后会通过LayoutInflater去加载RemoteViews的布局文件。在SystemServer进程加载后的布局文件就是一个普通的View，只不过相对于我们的进程他是一个RemoteViews，接着系统会对View执行一系列的界面更新任务，这些任务就是之前我们通过set方法来提交的。set方法对View的操作不是立即更新的，RemoteViews会记录这些操作，具体执行时机是在RemoteViews被加载之后才能执行，这样RemoteViews就在SystemServer进程中显示了。需要更新时，我们需要调用一系列的set方法并通过NotificationManager和AppWidgetManager来提交更新任务，具体操作方法也是在SystemServer进程中执行的。
	
	系统并没有直接通过Binder支持View的跨进程访问，而是提供了一个Action的概念，Action代表一个View的操作，Action实现Parceable接口。系统会将View的操作封装到Action对象，并将这些对象跨进程传输到远程进程，远程进程执行Action对象的具体操作。每调用一个set方法，RemoteViews就会添加一个Action对象，当我们通过NotificationManager和AppWidgetManager提交我们的更新时，这些Action对象就会被传输到远程进程并在远程进程中执行更新。远程进程通过RemoteViews的apply方法进行View的更新操作，apply方法内部会遍历所有的Action对象并调用她们的apply方法，具体的View更新操作是由Action对象的apply方法来完成的。
	这样做的好处就是不必定义大量的Binder接口，其次通过在远程进程中批量执行RemoteViews的修改操作从而避免大量的IPC操作，这就提高了程序的性能。
	
	set方法很多是使用反射机制来完成的，但是不是全部的都是使用反射。
RemoteViews的工作过程：

	RemoteViews  apply()方法 遍历所有的Action对象并调用Action的apply()方法完成界面的创建  遍历调用Action对象的reApply方法完成界面的刷新


RemoteViews的意义以及使用RemoteViews实现跨进程更新界面
	如果跨进程通讯中需要频繁的更新界面，可以使用RemoteViews来解决这个问题。
	RemoteViews通过apply方法加载并更新界面;
			View view=RemoteViews.apply(Context context,ViewGroup parent); 
	同一个应用的多进程可以将remoteViews放到intent的Extras中传递过来，但是面对不同应用的多进程通讯就需要双方约定，然后需要更新布局的app在自己的应用中查找布局文件，
		  int layoutId=  getResources().getIdentifier(layoutName,"layout",getPackageName())
		  View view=LayoutInflater.from(this).inflate(layoutId,parentView,false);
		  RemoteViews.reapply(Contenxt context,ViewGroup parentView);
		  
		   
			