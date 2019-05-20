概述
	高效加载Bitmap、缓存(LruCache和DiskLruCache)、优化列表的卡顿现象(ListView和GridView加载大量的子视图，快速滑动时就容易出现卡顿现象)
	
	LruCache核心思想：当缓存快满时，会淘汰近期最少使用的缓存目标。

Bitmap的高效加载
	Bitmap的加载
		BitmapFactory的四种方法：decodeFile、decodeResource、decodeStream、decodeByteArray，分别用于文件、资源、输入流以及字节数组中加载一个Bitmap对象，最终都是使用本地方法将图片加载为Bitmap对象
	
	Bitmap压缩
		采样率压缩，通过BitmapFactory.Options  inJustDecodeBounds 设为true可以拿到图片的宽高信息并不会加载到内存

		设置inSampleSize 设置为n 压缩为n的2次方  高度和宽度都会压缩 

# Android中的缓存策略 #
	常用LruCache和DiskLruCache

LruCache
	常用于内存缓存，是一个泛型类，内部采用一个LinkedHashMap以强引用的方式存储外界的缓存对象，其提供get和put方法完成缓存的获取和添加操作。
	强引用：直接的对象引用
	软引用：系统内存不足时此对象会被回收
	弱引用：此对象随时会被gc回收
	LruCache是线程安全的
	
	获取虚拟机内存的几个方法
	 (int) (Runtime.getRuntime().maxMemory() / 1024);//这个方法返回的是java虚拟机（这个进程）能够从操作系统那里挖到的最大的内存，以字节为单位，

    (int) (Runtime.getRuntime().totalMemory() / 1024);//java虚拟机现在已经从操作系统那里挖过来的内存大小，

    (int) (Runtime.getRuntime().freeMemory() / 1024 );//内存总是慢慢的从操作系统那里挖的，基本上是用多少挖多少，但是java虚拟机100％的情况下是会稍微多挖一点的，这些挖过来而又没有用上的内存

	    int cacheSize = maxSize / 8;
	   private LruCache lruCache = new LruCache<String, Bitmap>(cacheSize) {//LruCache就这样
	        @Override
	        protected int sizeOf(@NonNull String key, @NonNull Bitmap value) {
	            // 重写此方法来衡量每张图片的大小，默认返回图片数量。
	            //  return super.sizeOf(key, value);
	            return value.getByteCount() / 1024;
	        }

			 @Override//当移除旧缓存时就会调用该方法，可以用来完成一些资源回收工作
        	protected void entryRemoved(boolean evicted, String key, Bitmap oldValue, Bitmap newValue) {
            super.entryRemoved(evicted, key, oldValue, newValue);

        	}
	    };


DiskLruCache
	用于实现存储设备缓存，即磁盘缓存，通过将缓存对象写入文件系统从而实现缓存的效果。

	DiskLruCache的创建
		不能通过构造方法创建，提供open方法用于创建自身
		示例：
		DiskLruCache mDiskLruCache = DiskLruCache.open(file, getVersion(), 1, 10 * 1024 * 1024);	
		 参数1：缓存保存的位置   参数2：版本号：一般为1  参数3：单个节点对应的数据的个数，一般为1   参数4：缓存总大小

	DiskLruCache的缓存添加操作通过Editor完成的，Editor表示的是一个缓存对象的编辑对象。
	    DiskLruCache.Editor editor = mDiskLruCache.edit(key);//获取编辑对象
                    OutputStream outputStream = editor.newOutputStream(0);//由于一个节点对应一个数据 ，参数为0
                    if (downImageUrl(imageUrl, outputStream)) {
                        editor.commit();//提交操作
                    } else {
                        editor.abort();//放弃此次操作
                    }
                    mDiskLruCache.flush();//这个方法用于将内存中的操作记录同步到日志文件（也就是journal文件）当中

	读取缓存
		 DiskLruCache.Snapshot snapshot = mDiskLruCache.get(String key);
		 InputStream is = snapShot.getInputStream(0);
		 Bitmap bitmap = BitmapFactory.decodeStream(is);
	
	移除缓存
	  mDiskLruCache.remove(String key);不常用，因为DiskLruCache会自动移除

	DiskLruCache其它API：
      1、size() 获取缓存的大小 字节数量
	  2、close()  关闭DiskLruCache
	  3、delete() 将所有的缓存数据删除


