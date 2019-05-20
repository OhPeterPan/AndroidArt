概述	
	日常开发只与Handler打交道即可，可以很轻松的将一个任务切换到Handler所在的线程中去执行。
Android的消息机制
	本质：Handler的运行机制
	支撑：MessageQueue 、Looper 

	MessageQueue 消息队列 内部存储了一组消息，以队列的形式对外提供插入与删除工作。其实内部并不是真正的队列，而是使用单链表的数据结构来存储消息

	Looper 轮询器 消息循环。MessageQueue只是存储消息，并不能处理消息，，Looper就是为了去处理消息，Looper会以无限循环的方式去查找是否有新消息，如果有的话就去处理消息，否则一直等待。

	Looper中的ThreadLocal，这并不是线程，作用是在每个线程中存储数据。可以在不同的线程中互不干扰的存储并提供数据，通过ThreadLocal可以很轻松的获取每个线程的Looper。

	线程默认没有Looper，如果需要使用Handler必须为线程创建Looper
	
	主线程(UI线程)，也就是ActivityThread ，创建时就会初始化Looper，没有Looper使用Handler会报异常

ViewRootImpl对UI操作进行验证：
	checkThread方法： checkThread();//只有创建视图的原始线程才能触及此视图

Hnadler工作原理：
	Handler创建时会采用目前县城的Looper来构建内部的消息循环系统，如果当前线程没有Looper，会抛出异常。

ThreadLocal
	线程内部的数据存储类，通过他可以在指定的线程内存储数据，数据存储之后，只有在指定的线程内才可以获取到存储的数据，对于其他线程来说无法获取到数据。
	使用场景
		1、当某些数据是以线程为作用域时并且不同的线程具有不同的数据副本时，考虑使用ThreadLocal
		2、复杂逻辑下的对象传递，比如多层方法调用栈中的监听
	概述：
		不同的线程访问同一个ThreadLocal的get方法，ThreadLocal会从各自的线程中取出一个数组，然后再从数组中根据当前ThreadLocal的索引去查找对应的value值，不同的线程中的数组是不同的，这就是为什么ThreadLocal可以在不同的线程中维护一套数据的副本并且彼此互不干扰

消息队列的工作原理  单链表实现  单链表在插入和删除操作上具有优势
	MessageQueue 两个操作：插入和读取(读取伴随着删除操作)  方法对应 enqueueMessage和next
	enqueueMessage:向消息队列中插入一条消息  源码表示就是做链表的插入操作
	next：从消息队列中取出一条消息并将其从消息队列中删除。是一个无限循环的方法，当队列中没有消息时，会阻塞在这里，有新消息到来时，next会返回这条消息并将其从链表中删除

Looper的工作原理
	扮演消息循环的角色，不停的从MessageQueue中查看是否有新消息，如果有就会立刻处理，否则就一直阻塞在这里。
	通过Looper.prepare()为当前线程创建一个Looper，接着通过Looper.loop()来开启消息循环
	Looper的退出：quit和quitSafely quit直接退出  quitSafely安全的退出，需要把消息队列中所有的消息处理完成之后再退出
	Looper退出后，试用Handler的send方法会返回false。
	
	注：子线程中的Looper在所有事件处理完之后应该调用quit退出，否则这个线程就会一直处于等待状态，退出Looper后，这个线程会立刻停止。

	重要方法：loop 
		死循环，唯一跳出循环的方法是MessageQueue的next方法返回null。但是MessageQueue的next方法没有消息时只是阻塞在哪里，所以Looper.loop()方法也是阻塞在哪里。除非Looper调用quit或者quitSafely来通知MessageQueue退出，这时候MessageQueue的next方法会返回null，此时Looper的loop就结束循环状态。

主线程的消息循环
	Android的主线程就是ActivityThread
	

	