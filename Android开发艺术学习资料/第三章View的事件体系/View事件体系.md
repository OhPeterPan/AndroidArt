
View的基础知识
  View是android体系中所有控件的基类

	View的位置参数
		view的位置主要由它的四个顶点来决定，分别对应View的四个属性：Top，Left，Right，bottom，分别对应相对于父容器的上左右下坐标，这些坐标都相当于View的父容器来说的。
		
		left:grtLeft
		Top:getTop
		right:getRight
		bottom:getBottom

		从Android3.0开始：增加几个额外的参数：x，y，translationX和translationY。这几个参数也是相对于父容器的坐标，translationX和translationY的默认值为0

		x和y是代表view的左上角的坐标，
		translationX和translationY代表View左上角坐标相对于父容器的偏移量
		
		以上几个参数的换算关系：
			x=left+translationX
			y=top+translationY
		在View平移的过程中，left和top是代表View原始左上角的坐标，其值并不会发生改变，发生改变的是x、y、translationX、translationY这四个参数

	MotionEvent
		手指触摸屏幕时产生的触摸事件
		典型的触摸事件：
			ACTION_DOWN:手指按下事件
			ACTION_MOVE:手指移动事件
			ACTION_UP:手指离开屏幕事件
		得到手指触摸的x和y坐标(通过MotionEvent对象得到)：
			getX和getY  得到相对于自身左上角的坐标
			getRawX和getRawY  得到相对于手机屏幕左上角的坐标

	TouchSlop对象	
		是系统能识别的被认为是滑动的最小距离  常量由设备自定义  int类型
		获取方法：ViewConfiguration.get(Context context).getScaledTouchSlop();

	VelocityTracker
		速度追踪。用于追踪手指在滑动过程中的速度，包括水平速度和竖直方向上的速度
		使用方法：
				在OnTouchEvent方法中使用
					   VelocityTracker velocityTracker = VelocityTracker.obtain();
       				   velocityTracker.addMovement(event);

				想获取当前速度：
					   velocityTracker.computeCurrentVelocity(100);//100毫秒内手指滑过的像素数
        			   velocityTracker.getXVelocity();
       				   velocityTracker.getYVelocity();
			    公式	   速度=(终点位置-起点位置)/时间段  所以速度可能为负值

		内存重置并回收：   velocityTracker.clear();
        				 velocityTracker.recycle();
				

	GestureDetector
		手势检测，用于辅助检测用户的单机、滑动、长按、双击等行为
		使用方法 ：
		        GestureDetector   gestureDetector = new GestureDetector(this, gestureListener);
		        gestureDetector.setIsLongpressEnabled(false);//解决长按屏幕后无法拖动的现象
				private GestureDetector.OnGestureListener gestureListener = new GestureDetector.OnGestureListener() {
							
							//手指轻轻触摸屏幕的一瞬间 由一个ACTION_DOWN事件触发
					        @Override
					        public boolean onDown(MotionEvent motionEvent) {

					            return false;
					        }
					
							//press  压、按、逼迫、紧抱  
							  手指轻轻的触摸屏幕，尚未松开或者滑动  和onDown相比，强调的是没有松开或者拖动的状态
					        @Override
					        public void onShowPress(MotionEvent motionEvent) {
					
					        }
					
							//手指轻触屏幕后松开，一个单击事件		
					        @Override
					        public boolean onSingleTapUp(MotionEvent motionEvent) {
					            return false;
					        }
					
							//手指按下屏幕并滑动  有一个ACTION_DOWN事件和多个ACTION_MOVE事件触发 拖动行为
					        @Override
					        public boolean onScroll(MotionEvent motionEvent, MotionEvent motionEvent1, float v, float v1) {
					            return false;
					        }
					
							//长按事件
					        @Override
					        public void onLongPress(MotionEvent motionEvent) {
					
					        }
					
							//用户按下触摸屏、快速滑动后松开，有一个ACTION_DOWN、多个ACTION_MOVE、和一个ACTION_UP事件触发  快速滑动行为
					        @Override
					        public boolean onFling(MotionEvent motionEvent, MotionEvent motionEvent1, float v, float v1) {
					            return false;
					        }
					    };

			gestureDetector.setOnDoubleTapListener(new GestureDetector.OnDoubleTapListener() {

								//严格的单击行为  与onSingleTapUp的区别  触发该方法后后面不可能再紧跟着另一个单击行为，只可能是单击，不可能是双击中的一次单击
					            @Override
					            public boolean onSingleTapConfirmed(MotionEvent e) {
					                return false;
					            }
					
								//双击  不可能与onSingleTapConfirmed共存
					            @Override
					            public boolean onDoubleTap(MotionEvent e) {
					                return false;
					            }
					
								//发生了双击行为
					            @Override
					            public boolean onDoubleTapEvent(MotionEvent e) {
					                return false;
					            }
					        });
			然后：
				在onTouchEvent里面
				boolean resume=gestureDetector.onTouchEvent(event);
				return resume；
				

	Scroller对象
			弹性滑动对象，用于实现View的弹性滑动
			解决问题：当我们在使用scrollTo/scrollBy滑动时，滑动过程是瞬间完成的，用户体验效果比较差。这时候可以使用Scroller来实现有过渡效果的滑动，过程不是瞬间完成的，而是在一定的时间间隔内完成的。与View的computeScroll方法配合完成

View的滑动  
		3种方式：
			1、使用ScrollTo/ScrollBy方法	  这个滑动的是View的内容  2、3滑动的是View本身			
			2、通过动画给View施加平移效果进行滑动
			3、通过改变View的LayoutParams使得View重新布局实现滑动

	scrollTo/scrollBy
			scrollBy相对于当前位置的相对滑动 
			scrollTo参数传入位置的绝对滑动

	使用动画 (nineoldandroids) 3.0以下不适合需要交互的View(补间动画改变的是影像，不是真正的View位置)

	LayoutParams  适合有交互需求的View

弹性滑动
		Scroller  handler#postDelayed  Thread#sleep

View事件分发机制
		三个方法：dispatchTouchEvent(MotionEvent event)
				onInterceptTouchEvent(MotionEvent event)
				onTouchEvent(MotionEvent event)
		伪代码阐述他们之间的关系
			    public boolean dispatchTouchEvent(MotionEvent ev) {
			        boolean consume = false;
			
			        if (onInterceptTouchEvent(ev))
			            consume = onTouchEvent(ev);
			        else
			            consume = child.dispatchTouchEvent(ev);
			        return consume;
			    }

		事件传递顺序：
			Activity->Window->View

			Activity:    
					public boolean dispatchTouchEvent(MotionEvent ev) {
					        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
					            onUserInteraction();
					        }
					        if (getWindow().superDispatchTouchEvent(ev)) {//一个事件序列中返回false会转交给Activity的onTouchEvent方法处理
					            return true;
					        }
					        return onTouchEvent(ev);
					    }

			Window：
					@Override
				    public boolean superDispatchTouchEvent(MotionEvent event) {
				        return mDecor.superDispatchTouchEvent(event);//mDecor为继承自FrameLayout的自定义控件
				    }

			DecorView：
						public boolean superDispatchTouchEvent(MotionEvent event) {
		       			 return super.dispatchTouchEvent(event);
		    			}

			ViewGroup:
						public boolean dispatchTouchEvent(MotionEvent ev) {....}


	1、一个事件序列从手指触摸屏幕开始 到手指离开屏幕的那一刻结束，在这一过程中所产生的一系列事件 down事件开始 数量不定的move事件  up事件结束

	2、正常情况下，一个事件只能被一个View拦截并消耗，因为一个元素一旦拦截了事件，那么同一序列的所有事件都会直接交给这个拦截的元素处理，，因此同一个事件序列中的事件不能分别由两个View同时处理，但是可以通过特殊手段做到一个事件几个View一起处理

	3、某个View一旦拦截事件，那么一个事件序列都会交给这个View处理，它的onInterceptTouchEvent不会被调用

	4、一个View一旦开始处理事件，如果不消耗down事件，那么同一序列的其它事件都不会交给他处理，事件会被父类的onTouchEvent处理

	5、如果View不消耗除down事件以外的其它事件，那么这个点击事件会消失，此时父元素的onTouchEvent并不会被调用，当前View可以接收到后续的事件，最终这些消失的事件会传递给Activity处理

	6、ViewGroup的onInterceptTouchEvent默认返回false  即不拦截事件

	7、View是没有onInterceptTouchEvent方法的  事件一旦传递给了他 它的onTouchEvent方法就会调用

	8、View的onTouchEvent默认返回true  除非不可点击(clickable和longClickable同时为false) View的longClickable默认是false  clickable不同的控件情况不一样 View通过setOnclickListener 会将CLICKABLE设置为true 通过setOnLongClickListener将LONG_CLICKABLE设置为true

	9、View的enable属性不影响onTouchEvent的返回值，哪怕一个View是disable状态的，只要longClickable和clickable有一个是true，onTouchEvent就返回true
	10、View的onClick发生的前提是View可点击，并且会收到down和up事件

	11、事件传递是由外到内 ，事件总是先传递给父元素，然后由父元素分发给子View，通过requestDisallowInterceptTouchEvent方法可以在子View中干预父View的事件分发过程，但是down事件除外

	12、当一个View正在处理事件时被父类拦截，会接收到一个cancel事件
	
	Window类可以控制顶级View的外观和行为策略，唯一实现是PhoneWindow。
					
	事件是否会被子View接收：
		子元素是否正在进行动画和点击事件的坐标是否落在子元素的区域内

滑动冲突
	常见的滑动冲突场景：
		1、外部滑动方向与内部滑动方向不一致		 ViewPager和Fragment配合使用 fragment里面有ListView
		2、外部滑动方向与内部滑动方向一致  
		3、上述两种情况的嵌套  场景 外部有个slideMenu效果 内部有个ViewPager，Viewpager的页面中又有一个ListView

滑动冲突的处理规则：
	场景一：谁想用事件就处理事件
	场景二，场景三：根据实际规则处理冲突

	实际规则：
		1、外部拦截法  onInterceptTouchEvent中拦截  但是请记住不要拦截down方法 因为down事件一旦被拦截 后续事件序列中的所有事件都会被父元素给使用
		
		2、内部拦截法  父元素不拦截任何事件  所有的事件都传递给子元素处理 若子元素需要就消耗此事件  否则交由父元素进行处理 ，这种方法与Android事件分发机制不一致，需要配合requestDisallowInterceptTouchEvent方法才能正常工作  而且需要父类拦截除了down事件以外的其它全部事件
	
				
					