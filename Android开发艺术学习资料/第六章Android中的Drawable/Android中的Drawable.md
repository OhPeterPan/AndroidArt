简述：
	Drawable是一种可以在Canvas上进行绘制的抽象的概念，种类繁多，常见的图片或者颜色都可以是一个Drawable。使用简单，比自定义View的开发成本低，非图片类型的Drawable占用空间较小，减少App体积很常用。
	表示一种图像的概念，并不全是图片，使用各种颜色也可以构建出很多种图像效果.
	一般都是通过xml定义，也可以使用代码进行定义，代码会很复杂
	Drawable是一个抽象类，所有Drawable实现类的基类，每个具体的Drawable都是它的子类
Drawable的宽高：
	通过getIntrinsicWidth和getIntrinsicHeight可以拿到宽高。但是并不是所有的Drawable都会有宽高，比如颜色生成的Drawable就没有宽高的概念。Drawable的宽高不等同于它的大小，一般来说，Drawable没有大小概念，当做View的背景图时，会被拉伸至View的同等大小。

Drawable的分类：拿几个有代表性的。
	BitmapDrawable
		表示一张图片
			<?xml version="1.0" encoding="utf-8"?>
			<bitmap xmlns:android="http://schemas.android.com/apk/res/android"
			    android:antialias="true"//开启抗锯齿
			    android:dither="true"//开启抖动效果 图片的像素配置和手机屏幕的像素配置不一致的时候，开启这个可以使得高质量的图片在低质量的设备上保持很好的显示效果
			    android:filter="true"//是否开启过滤，图片尺寸被压缩和拉伸时，开启保持较好的显示效果
			
				//图片在容器中的位置显示
			    android:gravity="center|center_vertical
				|right|center_horizontal|end|left|bottom
				|clip_horizontal|clip_vertical|fill|fill_horizontal
				|fill_vertical|start|top"
			    android:src="@drawable/kenan"//图片资源id
				
				//平铺模式  默认不开启   有三种  1、clamp图片四周的像素扩散到周围  2、repeat 无限平铺效果  3、mirror镜子模式
			    android:tileMode="disabled" />

		NinePatchDrawable
		<nine-patch xmlns:android="http://schemas.android.com/apk/res/android"
		    android:dither="true"
		    android:src="@drawable/kenan" />

		shapeDrawable（自己稍微了解过一点）
			通过颜色来构造图形，既可以是纯色的图形，也可以是具有渐变效果的图形
			
			标签含义：
				android:shape   rectangle(矩形)  oval(椭圆)  line(横线)  ring(圆环)
		
		LayerDrawable
			层次化的drawable，通过将不同的Drawable放置在不同的层上面从而达到一种叠加的效果。一个layer_list可以包含多个item，每个item表示一个Drawable。默认情况下，内部所有的Drawable都会被缩放至View的大小，对于bitmap来说，需要使用gravity属性来控制图片的显示效果

			示例：
				<?xml version="1.0" encoding="utf-8"?>
				<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
				    <item>
				        <bitmap android:src="@drawable/kenan" />
				    </item>
				    <item
				        android:bottom="1dp"
				        android:left="10dp"
				        android:right="10dp"
				        android:top="1dp">//Drawable相对于View的上下左右的偏移量
				        <shape>
				            <corners android:radius="12dp" />
				            <solid android:color="@android:color/white" />
				        </shape>
				    </item>
				</layer-list>

		stateListDrawable
			对应select标签，也是一个表示Drawable的集合，每个Drawable都对应一个View的状态，根据View的状态来选择适合的Drawable
			示例：
				<?xml version="1.0" encoding="utf-8"?>
				<selector xmlns:android="http://schemas.android.com/apk/res/android"
				    android:dither="true|false"//防抖动  高质量图片在低质量设备上表现更好
				    android:constantSize="true|false"//Drawable的固有大小是否随着其状态的改变而改变
				    android:variablePadding="true"//Drawable的padding是否随着状态的改变而改变
				    >
				
				</selector>

		LevelListDrawable
			对应<level-list>标签 ，同样表示Drawable集合，集合中的Drawable都有一个等级的概念，根据不同的等级切换对应的Drawable
				<?xml version="1.0" encoding="utf-8"?>
					<level-list xmlns:android="http://schemas.android.com/apk/res/android">
					    <item
					        android:drawable="@drawable/kenan"
					        android:maxLevel="1" />//范围0~10000
					    <item
					        android:drawable="@drawable/ic_meinv"
					        android:maxLevel="100" />
					</level-list>

		TransitionDrawable
			对应<transition>标签 ，用来实现两个Drawable之间淡入淡出的效果
				实例
					<?xml version="1.0" encoding="utf-8"?>
					<transition xmlns:android="http://schemas.android.com/apk/res/android">
					    <item android:drawable="@drawable/kenan" />
					    <item android:drawable="@drawable/ic_meinv" />
					</transition>
					以button为例：
						    TransitionDrawable drawable = (TransitionDrawable) button.getBackground();
       						drawable.startTransition(1000);//动画持续时间
							drawable.reverseTransition(1000);//淡入淡出效果的逆过程

		InsetDrawable
			对应<inset>标签 可以将其他Drawable内嵌到自己当中，并且在四周留出一定的间距。当一个View希望自己的背景比自己实际显示区域小的时候可以使用这个
				示例：
					<?xml version="1.0" encoding="utf-8"?>
						<inset xmlns:android="http://schemas.android.com/apk/res/android"
						    android:insetLeft="15dp"
						    android:insetTop="15dp"
						    android:insetRight="15dp"
						    android:insetBottom="15dp">
						    <shape>
						        <solid android:color="@android:color/black" />
						    </shape>
						</inset> 

		ScaleDrawable
			对应<scale>标签可以根据自己的等级将指定的Drawable缩放到一定的比例  必须设置等级，等级范围0~10000，等级越大，缩放的越小，图片显示的就越大  
					示例：
						<?xml version="1.0" encoding="utf-8"?>
						<scale xmlns:android="http://schemas.android.com/apk/res/android"
						    android:drawable="@drawable/kenan"
						    android:scaleWidth="50%"
						    android:scaleHeight="50%"
						    android:scaleGravity="center" />
						以button为例：
						    ScaleDrawable drawable = (ScaleDrawable) button.getBackground();
						    drawable.setLevel(int )

		ClipDrawable：
			对应<clip>标签，根据自己当前的等级来裁剪另一个Drawable ，裁剪方向可以通过android:clipOrientation和android:gravity这两个属性来共同控制
					示例：
						<?xml version="1.0" encoding="utf-8"?>
						<clip xmlns:android="http://schemas.android.com/apk/res/android"
						    android:clipOrientation="vertical"
						    android:drawable="@drawable/kenan"
						    android:gravity="top" />
						以imageView为例
							     ClipDrawable clipDrawable = (ClipDrawable) imageView.getBackground();
       							 clipDrawable.setLevel(9000);//0~10000  数字越大裁剪的越少

自定义Drawable
	Drawable使用单一：
		1、作为ImageView中的图片显示
		2、作为View的背景展示
	通过View的工作原理可知，系统会调用Drawable的draw方法来绘制View的背景，通过重写Drawable的draw方法来自定义Drawable(省略)

插播知识(图片的数据格式占用字节数)
	
    ALPHA_8 -- (1B)
    RGB_565 -- (2B)
    ARGB_4444 -- (2B)
    ARGB_8888 -- (4B)  这个是Android默认的图片数据格式
    RGBA_F16 -- (8B)

	
