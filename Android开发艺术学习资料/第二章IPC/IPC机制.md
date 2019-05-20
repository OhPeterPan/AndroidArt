IPC进程间通信或者跨进程通信   指两个进程之间进行数据交换的过程


线程：按操作系统中的描述  线程是CPU调度的最小单元，是一种有限的系统资源
进程：一个执行单元，在pc或者或者移动设备上表示一个程序或者一个应用
	一个进程可以包含一个线程，
IPC机制不是android所特有的
	window中的复制粘贴、油槽、管道等进行进程间通信
	linux可以通过命名管道  共享内容  信号量来进行进程间通信
	Android进行进程间通信最有特色的就是Binder  通过Binder可以轻松的实现进程间通信

多进程情况
	1：自身需要多进程模式运行  有的模块需要运行在其他进程中   利用多进程获取更多的内存空间
	2：当前应用需要向其它应用获取数据

android中的多进程模式：
	给4大组件清单文件中设置"android:process"属性  
	还有个方式 使用JNI在native层fork一个新的进程（特殊情况）

	创建新进程实例
		1、
	    <activity
            android:name=".ui.TwoActivity"
            android:configChanges="orientation|keyboardHidden"
            android:process=":remote" />
		2、
       <activity
            android:name=".ui.FirstActivity"
            android:process="com.dalimao.mytaxi.kotlintest.remote"/>
  adb shell ps| grep 包名 查看当前的进程
	
	上述的两种创建进程的方式有区别
			：开头的进程属于当前应用的私有进程，其它应用的组件不可以和他跑在同一个进程中
			进程名不是：开头的属于全局进程  可以通过	ShareUID方式和它跑在同一个进程中
			
			Android系统会为每一个应用分配一个唯一的UID，具有相同uid的应用才能共享数据
			两个应用通过ShareUID跑在同一个进程中是有要求的，需要具有相同的shareUid并且签名也需要相同，在这种情况下，他们可以互相访问私有路径，比如data目录、组件信息....还可以共享内存数据，或者说他们看起来就像是一个应用的两个部分

多进程模式的运行机制

		正常情况下，Android会为每个应用都分配一个虚拟机，不同的虚拟机在内存分配上会有不同的内存地址，这就导致在不同的虚拟机上访问同一个对象会产生多份副本
			
		开启多进程其实就相当于重新开启一个应用
		同一个应用间的多进程：相当于两个不同的应用通过shaerdUID的模式

		使用多进程方式造成的问题:
			1、静态成员跟单例模式完全失效
			2、线程同步机制完全失效
			3、sharedPreferences的可靠性下降
			4、Application会多次创建

IPC基础概念的介绍
	序列化：把对象序列化到硬盘上
	反序列化：从硬盘上把文件反序列化为对象	

	serializable和parcelable接口 以及binder
		
	serializable和parcelable接口可以完成对象的序列化过程

	serializable
			JAVA提供的新的序列化方式 为对象提供标准的序列化以及反序列化的操作
			在实现serializable的实体类中可以定义serialVersionUID		
			
serialVersionUID	
		辅助序列化和反序列化的过程的
	静态成员变量属于类不属于对象，所依不会参与序列化过程	

	工作机制
		序列化的时候会把当前类的serialVersionUID写入到序列化的文件中，当反序列化的时候系统回去检测文件中的serialVersionUID，看他是否与当前类的serialVersionUID一致，如果一致的话证明序列化的类的版本和当前类的版本是相同的，这个时候就成功的反序列化了，否则的话说明当前类和序列化的类相比发生了某些变化（比如成员变量的数量、类型可能发生了变化），这个时候是无法正常反序列化的
	serialVersionUID还是尽量指定的，这样会最大限度的恢复数据，比如增加或者删除了某些变量依然可以反序列化成功
	当然如果类的整体结构已经改变（比如修改了类名），纵然是没有修改serialVersionUID的值，但是依然会反序列化失败
	
		对象序列化
		val user = User("heheh", 34)
        val objectOutputStream = ObjectOutputStream(FileOutputStream(File(filesDir, "out.txt")))
        objectOutputStream.writeObject(user)

		对象反序列化
		val objectInputStream = ObjectInputStream(FileInputStream(File("cache.txt")))
        val user: User? = objectInputStream.readObject() as User?
        objectInputStream.close()
		
		可以修改默认的序列化或者反序列化过程
		    private void writeObject(ObjectOutputStream out) throws IOException {} 
    		private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {} 

	parcelable接口是Android提供的序列化接口


	parcelable、serializable优缺点比较
		parcelable：开销小，比较适合用在Android系统上，主要用于内存序列化上，在硬盘序列化和网络序列化时比较复杂
		serializable：JAVA平台的序列化接口，开销大，但是在硬盘序列化和网络序列化上对比parcelable比较简单，
		内存序列化首选parcelable   硬盘和网络序列化首选serializable

Binder
	Binder是android中的一个类，实现了IBinder接口。
	从IPC的角度来讲，Binder是android中的一种跨进程通信方式，Binder还可以理解为一种虚拟的物理设备，驱动位于/dev/binder。

	从Android Framework的角度来说，Binder是ServiceManager连接各种Manager(ActivityManger、WindowMnager....)和ManagerService的桥梁； 

	从android应用层来说，Binder是客户端与服务端进行通讯的媒介，当bindService的时候，服务端会返回一个包含服务端业务调用的binder对象，通过这个binder对象可以获取服务端提供的服务或者数据，这里的服务包括普通服务以及包含AIDL的服务
   	
	Android开发中，Binder主要用于Service中，包括AIDL和Messenger，普通的service中的Binder不包括进程间通讯
	messenger  底层其实是AIDL，

AIDL学习Binder
		aidl包下新建 Book.java   创建aidl文件夹   Book.aidl   IBookManager.aidl
		java文件中的Book实现Parcelable接口   

	Book.aidl  文件中内容 ： package com.art.demo.one.aidl;  parcelable Book;

	IBookManager.aidl文件内容：package com.art.demo.one.aidl;
							  import com.art.demo.one.aidl.Book;
							  interface IBookManager {
  										List<Book> getBookList();//从服务端获取全部的书籍列表
  										void addBook(in Book book);//添加一个书籍到列表
									}	
	注：尽管Book.aidl已经与IBookManager.aidl同处于同一个包中，但是依然需要导包
	完成这些后系统自动为程序生成Binder类

		在gen目录下的包中有一个接口 IBookManager 以下是代码
		
				package com.art.demo.one.aidl;
			
				public interface IBookManager extends android.os.IInterface {//所有需要在Binder中传输的接口都需要继承IInterface
			    /**
			     * Local-side IPC implementation stub class.  Binder类
					当服务端和客户端在同一个进程的时候，方法调用不会走跨进程的transact过程
					服务端跟客户端跨进程时，transact过程就由内部类Proxy来完成
			     */
			    public static abstract class Stub extends android.os.Binder implements com.art.demo.one.aidl.IBookManager {

					//Binder的唯一标示
			        private static final java.lang.String DESCRIPTOR = "com.art.demo.one.aidl.IBookManager";
			
			        /**
			         * Construct the stub at attach it to the interface.
			         */
			        public Stub() {
			            this.attachInterface(this, DESCRIPTOR);
			        }
			
			        /**
			         * Cast an IBinder object into an com.art.demo.one.aidl.IBookManager interface,
			         * generating a proxy if needed.
			         * 用于将服务端的Binder对象转换成客户端所需的AIDL接口类型的对象，转换分进程
			         * 位于同一个进程时 返回的是服务端的Stub对象本身，不是同一个进程时，返回的是经过系统封装后的Stub.Proxy对象
			         */
			        public static com.art.demo.one.aidl.IBookManager asInterface(android.os.IBinder obj) {
			            if ((obj == null)) {
			                return null;
			            }
			            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
			            if (((iin != null) && (iin instanceof com.art.demo.one.aidl.IBookManager))) {
			                return ((com.art.demo.one.aidl.IBookManager) iin);
			            }
			            return new com.art.demo.one.aidl.IBookManager.Stub.Proxy(obj);
			        }
			
					//返回当前的Binder对象
			        @Override
			        public android.os.IBinder asBinder() {
			            return this;
			        }
						/**
						*此方法运行服务端中的Binder线程池中，当客户端发起跨进程请求的时候，远程请求会通过系统底层封装后交由此方法来处理
						*服务端通过code这个参数来判断请求的是哪个方法，从data中取出方法需要的参数  如果有返回值的话写入reply中返回  
						*返回值为false则表示客户端的请求失败 可以通过
						*/
			        @Override
			        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
			            java.lang.String descriptor = DESCRIPTOR;
			            switch (code) {
			                case INTERFACE_TRANSACTION: {
			                    reply.writeString(descriptor);
			                    return true;
			                }
			                case TRANSACTION_getBookList: {
			                    data.enforceInterface(descriptor);
			                    java.util.List<com.art.demo.one.aidl.Book> _result = this.getBookList();
			                    reply.writeNoException();
			                    reply.writeTypedList(_result);
			                    return true;
			                }
			                case TRANSACTION_addBook: {
			                    data.enforceInterface(descriptor);
			                    com.art.demo.one.aidl.Book _arg0;
			                    if ((0 != data.readInt())) {
			                        _arg0 = com.art.demo.one.aidl.Book.CREATOR.createFromParcel(data);
			                    } else {
			                        _arg0 = null;
			                    }
			                    this.addBook(_arg0);
			                    reply.writeNoException();
			                    return true;
			                }
			                default: {
			                    return super.onTransact(code, data, reply, flags);
			                }
			            }
			        }
			
			        private static class Proxy implements com.art.demo.one.aidl.IBookManager {
			            private android.os.IBinder mRemote;
			
			            Proxy(android.os.IBinder remote) {
			                mRemote = remote;
			            }
			
			            @Override
			            public android.os.IBinder asBinder() {
			                return mRemote;
			            }
			
			            public java.lang.String getInterfaceDescriptor() {
			                return DESCRIPTOR;
			            }

						/**该方法运行在客户端  当客户端远程调用此方法时：首先创建输入型Parcelable _data、输出型Parcelable _reply和返回值对象,然后把该方法的参数信息写入_data（如果有参数的话）；接着调用transact方法发起RPC(远程过程调用)请求，同时当前线程挂起；然后服务端的onTransact方法会被调用，直到RPC过程返回后，当前线程继续执行，并从_reply中取出返回结果
						**/	
		            @Override
			            public java.util.List<com.art.demo.one.aidl.Book> getBookList() throws android.os.RemoteException {
			                android.os.Parcel _data = android.os.Parcel.obtain();
			                android.os.Parcel _reply = android.os.Parcel.obtain();
			                java.util.List<com.art.demo.one.aidl.Book> _result;
			                try {
			                    _data.writeInterfaceToken(DESCRIPTOR);
			                    mRemote.transact(Stub.TRANSACTION_getBookList, _data, _reply, 0);
			                    _reply.readException();
			                    _result = _reply.createTypedArrayList(com.art.demo.one.aidl.Book.CREATOR);
			                } finally {
			                    _reply.recycle();
			                    _data.recycle();
			                }
			                return _result;
			            }
						/**
							基本与getBookList过程一样，就是没有返回值
						*/
			            @Override
			            public void addBook(com.art.demo.one.aidl.Book book) throws android.os.RemoteException {
			                android.os.Parcel _data = android.os.Parcel.obtain();
			                android.os.Parcel _reply = android.os.Parcel.obtain();
			                try {
			                    _data.writeInterfaceToken(DESCRIPTOR);
			                    if ((book != null)) {
			                        _data.writeInt(1);
			                        book.writeToParcel(_data, 0);
			                    } else {
			                        _data.writeInt(0);
			                    }
			                    mRemote.transact(Stub.TRANSACTION_addBook, _data, _reply, 0);
			                    _reply.readException();
			                } finally {
			                    _reply.recycle();
			                    _data.recycle();
			                }
			            }
			        }
			
					//生成两个描述字段，标示不同的方法，用处在于标示在transact过程中客户端请求的到底是哪个方法
			        static final int TRANSACTION_getBookList = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
			        static final int TRANSACTION_addBook = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
			    }
			
			    public java.util.List<com.art.demo.one.aidl.Book> getBookList() throws android.os.RemoteException;
			
			    public void addBook(com.art.demo.one.aidl.Book book) throws android.os.RemoteException;
			}
	
Binder中两个很重要的方法
		linkToDeath、unlinkToDeath
		背景:当Binder连接死亡时，会导致我们的远程调用失败。
		这两个方法属于配对出现，通过linkToDeath可以给Binder设置一个死亡代理，当Binder死亡时，我们会收到一个通知，这时候我们就可以重新发起连接请求从而恢复连接。
	
		通过isBinderAlive也可以判断Binder是否死亡
	
形形色色的进程间通讯方式：
	1、使用Bundle
	2、使用共享文件
	3、使用messenger(信使 底层还是使用AIDL实现) 一次只处理一个请求，所以不用考虑线程同步问题  使用handler message 来进行进程间通讯，但是由于message的载体android2.2之后也只是只支持系统提供的实现Parcelable接口的类的传输，所以导致适用范围很有限，但是还好Message支持Bundle，Bundle支持大量的数据类型，
	4、使用AIDL
	5、ContentProvider
	6、Socket

AIDL：
		由于使用messenger只能串行的方式处理客户端发送来的消息，如果有大量的消息同时发送，服务端只能一个一个的处理，而且messenger只能用来传递消息，如果想跨进程调用服务端的方法，这种Messenger就无法做到了，这时候只能使用AIDL来实现跨进程的方法调用，AIDL是Messenger的底层实现，因此Messenger也是AIDL,系统封装好方便我们调用。
		
	流程：
		客户端、服务端、AIDL接口
		
	AIDL支持的数据类型：
		1、基本数据类型
		2、String和CharSequence
		3、List：只支持ArrayList，里面的每个元素都必须能被AIDL支持
		4、map：只支持hashMap，里面的每个元素都必须被AIDL支持，包括KEY和VALUE
		5、所有实现了Parcelable接口的对象（不管在不在一个包中，只要使用了就必须使用import导入进来）
		6、所有AIDL本身也可以在AIDL文件中使用
	
	注：AIDL文件中除了基本数据类型，其他类型的参数都必须标上方向：in、out、inout
		in：表示输入型参数，out：输出型参数  inout：输入输出型参数
		定向Tag 所有的非基本参数都需要一个定向Tag来指出数据流通的方式，基本数据类型的定向Tag默认且只能是in
		定向参数Tag只能修饰参数，并不能修饰返回值  
		in 、out代表客户端与服务端的两条单向的数据流向 inout表示两端可双向流通数据
		当tag为in时，表示服务端将会接受到客户端传过来的完整的对象，但是客户端不会因为服务端对实参的改变而同步改变
		当tag为out时，表示服务端将会接受客户端传递的实参的空对象，在服务端对实参的对象进行改变时，客户端会同步改变
		当tag为inout，表示服务端既能接受到完整的参数对象，客户端在服务端将对象改变时会同步改变

	AIDL只支持方法，不支持声明静态常量
	跨进程通信发生在不同的应用程序时，需要把服务端的aidl文件copy到客户端中，而且如果有使用到实现了Parcelable接口的类，客户端也需要创建一个类来接收，包名与服务端一样（这里并没没有写错，就是这样要求的）

服务端的回调实现：场景  我们在服务端完成操作后通知客户端操作结果
		Binder会把客户端传递过来的对象重新转化为一个新的对象，所以使用传统的注册解注册时，会导致解注册失败，就是因为客户端的对象与服务端的对象不是同一个，
		如果想要解注册就需要使用RemoteCallbackList
		RemoteCallbackList：系统专门提供的用于删除远程listener的接口，是一个泛型，支持管理任意的AIDL接口，其实是因为虽然服务端会把客户端传递过来的对象重新转化，但是他们底层的Binder对象是同一个。
 		遍历RemoteCallbackList中的对象
		     int m = remoteCallbackList.beginBroadcast();
            for (int i = 0; i < m; i++) {
                IOnNewBookArrivedListener listener = remoteCallbackList.getBroadcastItem(i);
                listener.onNewBookArrived(book);//此方法运行在客户端的binder线程池中
            }
            remoteCallbackList.finishBroadcast();
		注：beginBroadcast()与finishBroadcast()必须成对的出现

添加服务端验证：不能让所有的客户端都能连接我们的服务端（单独列出来两个）：
	1、在onBinder中使用自定义权限 如果验证不通过就返回null  弊端：会因为我们的远程服务端对象为null而中断连接
	2、使用onTransact中返回值来判断  如果验证不通过就返回false，使用自定义权限或者使用UID和PID来验证


使用ContentProvider完成IPC
		是Android提供的专门用于不同的应用之间进行数据共享的方式  底层实现是Binder

		自定义ContentProvider：实现抽象方法
		onCreate()//由系统回调运行在主线程中
		//这几个方法运行在Binder线程池中
		query(...)、getType(..)、insert(...)、delete(...)、update(...)

	ContentProvider以表格的形式来组织数据，还可以使用文件数据

	ContentProvider通过Uri来区分要访问的数据集合，为了知道我们要访问那个表，需要为表单独的定义Uri和Uri_Code,并将Uri和Uri_Code关联到一起

	在manifest文件中注册ContentProvider
		    <provider
			//android:authorities ContentProvider的唯一标识，通过他外部应用就可以访问我们的Provider，所以必须是唯一的
            android:name=".content_provider.BookContentProvider"
            android:authorities="com.art.demo.one.provider"
            android:process=":provider" />

	使用UriMatcher将Uri和Uri_Code关联到一起，当外界请求访问Provider时，根据请求的Uri得到Uri_Code(sUriMatcher.match(Uri uri))，有了Uri_Code就知道外界想要访问那个表了
	public static final String AUTHORITY = "com.art.demo.one.content_provider.BookContentProvider";
    public static final Uri BOOK_CONTENT_URI = Uri.parse("content://" + AUTHORITY + "/book");
    public static final Uri USER_CONTENT_URI = Uri.parse("content://" + AUTHORITY + "/user");

    public static final int BOOK_CODE = 0;
    public static final int USER_CODE = 1;
    private static final UriMatcher sUriMatcher = new UriMatcher(UriMatcher.NO_MATCH);

    static {
        sUriMatcher.addURI(AUTHORITY, "book", BOOK_CODE);
        sUriMatcher.addURI(AUTHORITY, "user", USER_CODE);
    }

	ContentProvider 不仅支持数据库的增删改查 还支持自定义调用  通过ContentResolve的call方法和ContentProvider的Call方法

IPC操作之一：socket(暂缓....)
	套接字，网络通信中的概念  流式套接字(TCP协议)和数据报(UDP协议)套接字
	TCP是面向连接的协议，提供稳定的双向通信功能 连接的建立需经过三次握手才能完成，为了提高连接的稳定性，自身提供超时重传机制
	UDP是无连接的，提供不稳定的单向通信功能   其实也可以实现双向通信功能的
	效率比较：
		UDP有更好的效率，但是不保证数据的正确传输，尤其是网络拥堵的情况下

	demo：一个跨进程的聊天程序  通过socket实现信息的传输  本身支持传输任意的字节流

Binder连接池：解决一个项目使用AIDL过多的情况
		作用：就是将每个业务模块的Binder请求统一转发到一个远程Service中去执行，从而避免重复创建service的过程(实际操作看代码)

小知识:
	CountDownLatch:
			构造方法:CountDownLatch(int i)  参数表示在整数倒数到0之前，主线程都需要等待,这个所谓的倒数过程都是由各个线程驱动的，每个线程执行完任务就倒数一次   
			
			作用：等待其他的线程都执行完任务，必要时可以对各个任务的执行结果进行汇总，然后主线程才能继续向下执行
				
			主要方法：countDown和await  countDown()用于将计数器减一,await()方法则是使调用该方法的线程处于等待状态

	
		



