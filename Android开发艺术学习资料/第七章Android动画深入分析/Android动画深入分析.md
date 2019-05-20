Android中的动画：
	View动画和属性动画

	
View动画
	View动画  帧动画
	作用对象：View，支持四种动画效果：平移、旋转、缩放、透明度
	四种动画效果对应子类：TranslateAnimation(<translate>标签)、ScaleAnimation(<scale>标签)、RotateAnimation(<rotate>标签)、AlphaAnimation(<alpha>标签)
	
	实现方式：XML和具体类
		示例： anim文件夹下创建xml文件 	既可以是单个动画，也可以是动画集合
			<?xml version="1.0" encoding="utf-8"?>
			<set xmlns:android="http://schemas.android.com/apk/res/android"
			    android:duration="2000"
			    android:fillAfter="true"
			    android:shareInterpolator="true">//表示集合中的动画是否和集合共享同一个插值器
			    <translate
			        android:fromXDelta="100%"//自身的100%   100%p 相当于父元素的百分比  具体数值
			        android:fromYDelta="0"
			        android:toYDelta="0"
			        android:toXDelta="50%" />
			</set>
			

		缩放示例：
				<?xml version="1.0" encoding="utf-8"?>
				<set xmlns:android="http://schemas.android.com/apk/res/android"
				    android:duration="2000"
				    android:fillAfter="true">
				    <scale
				        android:fromXScale="1"
				        android:fromYScale="1"
				        android:pivotX="100%"
				        android:pivotY="100%"
				        android:toXScale="0"
				        android:toYScale="1" />
				</set>

			
		旋转示例：
				<?xml version="1.0" encoding="utf-8"?>
				<set xmlns:android="http://schemas.android.com/apk/res/android"
				    android:duration="2000"
				   >
				    <rotate
				        android:fromDegrees="0"//开始角度
				        android:pivotX="50%"
				        android:pivotY="50%"//旋转的轴点坐标
				        android:toDegrees="360" />//结束角度
				</set>

		透明示例：
				<?xml version="1.0" encoding="utf-8"?>
				<set xmlns:android="http://schemas.android.com/apk/res/android"
				    android:duration="2000">
				    <alpha
				        android:fromAlpha="1"
				        android:toAlpha="0.1" />
				</set>

			使用：
				Animation animation = AnimationUtils.loadAnimation(this, R.anim.xxx);
                View.startAnimation(animation);

	通过对场景不断做图像转换(平移、缩放、旋转、透明度)从而产生动画效果，是一种渐进式动画。
	帧动画是通过一系列的图像产生动画效果
	
自定义View动画
	继承Animation这个抽象类，重写initialize和applyTransformation方法，在initialize方法中做初始化工作，在applyTransformation中做相应的矩阵转换，很多时候使用camera简化矩阵转换的过程

	常见的集合变换:
		1、使用Canvas来做常见的集合变换
		2、使用Matrix来做常见的和不常见的二位转换
		3、使用Camera来做三维变换

帧动画
	播放一组预先定义好的图片，类似于电影播放，对应类AnimationDrawable来使用帧动画		
		示例：drawable文件夹下
			<?xml version="1.0" encoding="utf-8"?>
			<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
			    android:oneshot="false">
			    <item
			        android:drawable="@drawable/app_loading0"
			        android:duration="100" />
			    <item
			        android:drawable="@drawable/app_loading1"
			        android:duration="100" />
			    <item
			        android:drawable="@drawable/app_loading2"
			        android:duration="100" />
			    <item
			        android:drawable="@drawable/app_loading3"
			        android:duration="100" />
			</animation-list>

			使用：AnimationDrawable drawable = (AnimationDrawable) ivAnimation.getBackground();
                 drawable.start();

View动画的特殊使用场景
	1、LayoutAnimation：
		在ViewGroup中可以控制子View的出场效果 比如ListView的item的出场效果
		示例：anim文件夹下定义<layoutAnimation>标签  anim/layout_animation
			<?xml version="1.0" encoding="utf-8"?>
			<layoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
			    android:animation="@anim/layout_translate"//真正做动画的anim文件
			    android:animationOrder="normal"//表示子元素动画的顺序 normal顺序动画 reverse倒序动画  random随机播放动画 
			    android:delay="0.5">//表示子元素开始动画的延迟，跟layout_translate的duration有关，0.5代表延迟150ms
			
			</layoutAnimation>
			
		anim/layout_translate示例
			<?xml version="1.0" encoding="utf-8"?>
			<set xmlns:android="http://schemas.android.com/apk/res/android"
			    android:duration="300">
			    <translate
			        android:fromXDelta="100%p"
			        android:toXDelta="0" />
			</set>
		
	layoutAnimation使用:
		  android:layoutAnimation="@anim/layout_animation"
	除了XML指定LayoutAnimation外，还可以通过LayoutAnimationController来实现
		示例：
				LinearLayout root = findViewById(R.id.root);
		        Animation animation = AnimationUtils.loadAnimation(this, R.anim.layout_translate);
		        LayoutAnimationController controller = new LayoutAnimationController(animation);
		        controller.setDelay(0.5f);
		        controller.setOrder(LayoutAnimationController.ORDER_NORMAL);
		        root.setLayoutAnimation(controller);
		

属性动画
	通过改变对象的属性从而达到动画效果，谨记，改变的是属性的值！！！，所以能力很大
	ValueAnimator、ObjectAnimator、AnimatorSet 等概念

	属性动画可以对任意对象的属性进行动画，而不仅仅是View，动画默认时间间隔300ms，默认帧率10ms/帧。
	
	效果：在一个时间间隔内从一个属性值到另一个属性值的改变，因此属性对象几乎是无所不能的，只要对象有这个属性，都能实现动画效果
	兼容性：使用nineoldandroids

	使用方式：XML和代码  推荐使用代码实现
	XML使用方式：
			 Animator animator = AnimatorInflater.loadAnimator(this, AnimatorInflater.loadAnimator(this, R.animator.test_one).test_one);
	         animator.setTarget(实现动画的对象);
	         animator.start();

插值器和转换器（详情查看：https://blog.csdn.net/harvic880925/article/details/50995268）
	插值器:根据时间流逝的百分比计算出当前属性值改变的百分比
	转换器：根据当前属性改变的百分比计算改变后的属性值

属性动画的监听器：
	AnimatorUpdateListener、AnimatorListenerAdapter(AnimatorUpdateListener的适配器类，可以选择性的实现其中的方法)、AnimatorListener
	
对任意属性做动画
	满足条件：
		1、object必须要提供setxxx方法，如果动画的时候没有传递初始值，还要提供getxxx方法，因为系统会通过getxxx取出初始值(如果不满足条件，直接crash)
		2、object的setxxx对属性所做的改变必须能够通过某种方法反映出来，比如会带来UI的改变....不满足动画无效但不会crash

属性动画的工作原理：
	要求动画作用的对象提供该属性的set方法，属性动画根据你传递的该属性的初始值和最终值，以动画的效果多次去调用set方法。如果没有传递初始值，还必须提供该属性的get方法，因为系统要去获取属性的初始值。

	属性动画需要运行在有Looper的线程中

使用动画注意事项
	1、OOM问题  主要出在帧动画上面，因为用的图片很多，如果图片太大的话会导致OOM
	2、内存泄漏  属性动画中有一类是无限循环的动画，需要及时停止，View动画似乎没有这样的问题
	3、兼容性问题  属性动画Android3.0以上才出现
	4、View动画并没有真正改变View的状态，有时候出现动画后View的setVisibility失效问题，这时候需要view.clearAnimation清除动画即可解决
	5、不要使用px，尽量使用dp
	6、动画元素的交互 3.0以前不管是View动画还是属性动画都没有真正改变View的位置
	7、硬件加速  使用动画的时候建议开启硬件加速，可以提高动画的流畅性

	