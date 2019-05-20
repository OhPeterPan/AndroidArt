View是Android在视觉上的呈现

初识ViewRoot和DecorView
	ViewRoot对应ViewRootImpl类，连接WindowManager和DecorView的纽带，View的三大流程(measure、layout、draw)均是通过ViewRoot来完成的。
	在ActivityThread中，当Activity对象创建成功后，会将DecorView添加到Window中，同时会创建ViewRootImpl对象，并将ViewRootImpl对象和DecorView建立关联

	ViewRootImpl 中的performTraversals 依次调用 performMeasure  performLayout、performDraw 依次完成measure layout draw过程

	measure过程：
		performMeasure会调用measure方法，在measure方法中又调用onMeasure方法，在onMeasure方法中会对所有的子元素进行measure过程。
		measure过程决定了View的宽和高，Measure过后，可以通过getMeasureWidth/getMeasureHeight获取View测量后的宽和高，几乎所有的情况下都属于是最终的宽和高，特殊情况除外
	
	layout过程：
		决定了View四个顶点坐标以及实际的宽高，layout过程之后，可以通过getTop、geiLeft、getRight、getBottom拿到View四个点的坐标，通过getWidth、getHeight拿到View最终的宽和高

	draw过程：
		让View实实在在的显示在Activity里面，只有这个过程之后内容才是真的显示

View的onMeasure
		    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        	setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
              					 getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    								}

			protected int getSuggestedMinimumWidth() {
       			 return (mBackground == null) ? mMinWidth : max(mMinWidth, mBackground.getMinimumWidth());
   				 }

			public int getMinimumWidth() {
					//获取drawable的原始长度  不同的drawable不同而不同  比如shapeDrawable没有原始宽度， bitmapDrawable有原始宽度
			        final int intrinsicWidth = getIntrinsicWidth();
			        return intrinsicWidth > 0 ? intrinsicWidth : 0;
			    }	

			public static int getDefaultSize(int size, int measureSpec) {
			        int result = size;
			        int specMode = MeasureSpec.getMode(measureSpec);
			        int specSize = MeasureSpec.getSize(measureSpec);
			
			        switch (specMode) {
			        case MeasureSpec.UNSPECIFIED:
			            result = size;
			            break;
			        case MeasureSpec.AT_MOST:
			        case MeasureSpec.EXACTLY:
			            result = specSize;
			            break;
			        }
			        return result;
			    }
				
			  public static int resolveSizeAndState(int size, int measureSpec, int childMeasuredState) {
		        final int specMode = MeasureSpec.getMode(measureSpec);
		        final int specSize = MeasureSpec.getSize(measureSpec);
		        final int result;
		        switch (specMode) {
		            case MeasureSpec.AT_MOST:
		                if (specSize < size) {//取min值
		                    result = specSize | MEASURED_STATE_TOO_SMALL;
		                } else {
		                    result = size;
		                }
		                break;
		            case MeasureSpec.EXACTLY:
		                result = specSize;
		                break;
		            case MeasureSpec.UNSPECIFIED:
		            default:
		                result = size;
		        }
		        return result | (childMeasuredState & MEASURED_STATE_MASK);
		    }

获取View的宽高	
	在Activity的onCreate、onStart、onResume里面均无法正确获取View测量后的宽高  因为测量过程与生命周期方法不是同步的
	四个方法获取View的宽高：
		1、Activity/View#onWindowFocusChanged  Activity的窗口得到焦点和失去焦点时均会被调用一次
		2、view.post(Runnable runnable); 通过post将一个runnable投递到队列的尾部，然后等待Looper调用此runnable的时候，View就已经初始化好了
		3、ViewTreeObserver  使用ViewTreeObserver的众多回调状态可以完成这个功能，比如使用onGlobalLayoutListener这个接口，当View树发生改变或者View树内部的View可见性发生改变时，onGlobalLayout方法将会被回调，这是一个获取View宽/高的一个良好的时机，随着View的状态不断改变，onGlobalLayout会回调多次
			典型代码：
				button.getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
		            @Override
		            public void onGlobalLayout() {
		                button.getViewTreeObserver().removeOnGlobalLayoutListener(this);取消回调
		                //获取View的宽/高
							...
		            }
		        });
		4、view measure(int widthMeasureSpace,int heightMeasureSpace)
				手动进行measure得到View的宽高，情况比较复杂，根据View的LayoutParams来分:
					match_parent  无法measure出具体的宽高，构造此种MeasureSpec需要知道parentSize，即父容器的剩余空间，但是这时候无法得知父容器的大小，所以理论上不可能测量出View的大小
					具体值  可以得到
					wrap_content  无法得到具体宽高

Layout过程
		ViweGroup用来确定子元素的位置，当ViewGroup的位置被确定后，在onLayout中会遍历所有的子元素并调用layout方法，找到合适的位置

		View的测量宽高形成于View的measure过程  View的最终宽高形成于View的layout过程 
		以下代码可以看出 layout后最终的宽高与测量的宽高一般情况下是一致的，但是若果重写了View的layout方法 自己定义left/top/right/bottom可能会导致layout后最终的宽高与测量的宽高不一致
	
					   private void setChildFrame(View child, int left, int top, int width, int height) {//这里面的width/height获取的就是getMeasureWidth/getMeasureHeight
					        child.layout(left, top, left + width, top + height);
					    }

					   public final int getHeight() {
					        return mBottom - mTop;
					    }

draw过程
	将View绘制到屏幕上面，View的绘制过程遵循如下几步：
		1、绘制背景（drawBakground(Canvas canvas)  此方法不能重写）
		2、绘制自己(onDraw(Canvas canvas))
		3、绘制children (dispatchDraw(Canvas canvas))
		4、绘制装饰  onDrawForeground(canvas);
		5、绘制前景(Android6.0之后加入)		  onDrawForeground(canvas);

	注意：
	   1、出于效率的考虑，ViewGroup 默认会绕过 draw() 方法，换而直接执行 dispatchDraw()，以此来简化绘制流程。所以如果你自定义了某个 ViewGroup 的子类（比如 LinearLayout）并且需要在它的除 dispatchDraw() 以外的任何一个绘制方法内绘制内容，你可能会需要调用 View.setWillNotDraw(false) 这行代码来切换到完整的绘制流程（是「可能」而不是「必须」的原因是，有些 ViewGroup 是已经调用过 setWillNotDraw(false) 了的，例如 ScrollView）。

	    2、有的时候，一段绘制代码写在不同的绘制方法中效果是一样的，这时你可以选一个自己喜欢或者习惯的绘制方法来重写。但有一个例外：如果绘制代码既可以写在 onDraw() 里，也可以写在其他绘制方法里，那么优先写在 onDraw() ，因为 Android 有相关的优化，可以在不需要重绘的时候自动跳过 onDraw() 的重复执行，以提升开发效率。享受这种优化的只有 onDraw() 一个方法。

自定义View
	本书作者分为4类：
		1、继承自View重写onDraw方法
		2、继承自ViewGroup
		3、继承官方出品的View(如TextView)
		4、继承官方出品的ViewGroup
	
	自定义View须知：
		1、直接在继承自View的元素中使用wrap_content可能达不到预期的效果
		2、如果有必要，让你的View支持padding
		3、尽量不要在View中使用Handler  ，因为View有post方法，完全可以替代Handler
		4、View中如果有动画或者线程，需要及时停止  
			在包含View的Activity退出或者当前View被remove时，View的onDetachedFromWindow方法将会被调用。当包含此View的Activity启动时，View的onAttachedToWindow方法会被调用。利用这些方法及时停止线程或者动画，如果不及时处理，可能会导致内存泄漏
		5、View带有滑动嵌套情形时，需要处理好滑动冲突

自定义View的自定义属性步骤：
	第一步：values目录下创建自定义属性的XML，比如attrs.xml，文件名没有什么限制，可以随便取名字
		文件代码示例:
				<?xml version="1.0" encoding="utf-8"?>
				<resources>
				    <declare-styleable name="CircleView">
				        <attr name="circle_color" format="color" />
				    </declare-styleable>
				</resources>
	format:color(颜色值) reference(资源id) dimension(尺寸) String、integer和boolean指的是基本数据类型 fraction(百分数) float(浮点值) 
		   enum(枚举值)
					   （1）属性定义：
					            <declare-styleable name="名称">
					                   <attr name="orientation">
					                          <enum name="horizontal" value="0" />
					                          <enum name="vertical" value="1" />
					                   </attr>            
					
					            </declare-styleable>
					
					    （2）属性使用：
					
					            <LinearLayout
					
					                    xmlns:android = "http://schemas.android.com/apk/res/android"
					                    android:orientation = "vertical"
					                    android:layout_width = "fill_parent"
					                    android:layout_height = "fill_parent"
					                    >
					            </LinearLayout>
			flag(位或运算)
							  （1）属性定义：
					             <declare-styleable name="名称">
					                    <attr name="windowSoftInputMode">
					                            <flag name = "stateUnspecified" value = "0" />
					                            <flag name = "stateUnchanged" value = "1" />
					                            <flag name = "stateHidden" value = "2" />
					                            <flag name = "stateAlwaysHidden" value = "3" />
					                            <flag name = "stateVisible" value = "4" />
					                            <flag name = "stateAlwaysVisible" value = "5" />
					                            <flag name = "adjustUnspecified" value = "0x00" />
					                            <flag name = "adjustResize" value = "0x10" />
					                            <flag name = "adjustPan" value = "0x20" />
					                            <flag name = "adjustNothing" value = "0x30" />
					                     </attr>         
					
					             </declare-styleable>
					
					     （2）属性使用：
					            <activity
					                   android:name = ".StyleAndThemeActivity"
					                   android:label = "@string/app_name"
					                   android:windowSoftInputMode = "stateUnspecified | stateUnchanged　|　stateHidden">
					                   <intent-filter>
					                          <action android:name = "android.intent.action.MAIN" />
					                          <category android:name = "android.intent.category.LAUNCHER" />
					                   </intent-filter>
					             </activity>
			属性定义时可以指定多种类型值
	
	第二步：在View的构造方法里解析自定义属性的值并做相应的处理
			TypedArray ta = context.obtainStyledAttributes(AttributeSet attr,R.styleable.CircleView);
			int color=ta.getColor(R.styleable.CircleView_circle_color,defaultValus);
			ta.recycler();//实现资源
	第三步：在自定义控件中使用自定义属性
			布局文件中添加schemas声明

获取子控件Margin的方法
	重写generateLayoutParams（）函数    generate 形成，造成，引起
		@Override
		protected LayoutParams generateLayoutParams(LayoutParams p) {
		    return new MarginLayoutParams(p);
		}
		 
		@Override
		public LayoutParams generateLayoutParams(AttributeSet attrs) {
		    return new MarginLayoutParams(getContext(), attrs);
		}
		 
		@Override
		protected LayoutParams generateDefaultLayoutParams() {
		    return new MarginLayoutParams(LayoutParams.MATCH_PARENT,
		            LayoutParams.MATCH_PARENT);
		}


		
