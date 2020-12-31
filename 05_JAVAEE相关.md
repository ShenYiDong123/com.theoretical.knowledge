### 版本号 ###
version=1.0-20201224-001

### 基本介绍 ###
总结原理


### JAVA EE相关 ###

**  多线程ThreadPoolExecutor **
为什么要使用线程池？
1.可重复使用已有线程，避免对象创建、消亡和过度切换的性能开销。
2.避免创建大量同类线程所导致的资源过度竞争和内存溢出的问题。
3.支持更多功能，比如延迟任务线程池（newScheduledThreadPool）和缓存线程池

`public class ThreadPoolFactoryUtil {
`	/**
`	 * 线程名
`	 */
`	private static ThreadFactory namedThreadFactory = new ThreadFactoryBuilder()
`	        .setNameFormat("thread-name-%d").build();
`	/**
`	 * 自定义核心线程池
`	 * maximumPoolSize：参数定义为服务器核数+1
`	 */
`	public static ExecutorService THREADPOOL = new ThreadPoolExecutor(5, 5,
`	        0L, TimeUnit.MILLISECONDS,
`	        new LinkedBlockingQueue<Runnable>(1024),
`	        namedThreadFactory,
`	        new ThreadPoolExecutor.AbortPolicy());
`}

//开启线程
`ThreadPoolFactoryUtil.THREADPOOL.execute(() -> {
	//业务代码
`});

实际开发中采用LinkedBlockingQueue
原因：
LinkedBlockingQueue是一个阻塞队列，内部由两个ReentrantLock来实现出入队列的线程安全，由各自的Condition对象的await和signal来实现等待和唤醒功能。它和ArrayBlockingQueue的不同点在于：
队列大小有所不同，ArrayBlockingQueue是有界的初始化必须指定大小，而LinkedBlockingQueue可以是有界的也可以是无界的(Integer.MAX_VALUE)，对于后者而言，当添加速度大于移除速度时，在无界的情况下，可能会造成内存溢出等问题。
数据存储容器不同，ArrayBlockingQueue采用的是数组作为数据存储容器，而LinkedBlockingQueue采用的则是以Node节点作为连接对象的链表。
由于ArrayBlockingQueue采用的是数组的存储容器，因此在插入或删除元素时不会产生或销毁任何额外的对象实例，而LinkedBlockingQueue则会生成一个额外的Node对象。这可能在长时间内需要高效并发地处理大批量数据的时，对于GC可能存在较大影响。
两者的实现队列添加或移除的锁不一样，ArrayBlockingQueue实现的队列中的锁是没有分离的，即添加操作和移除操作采用的同一个ReenterLock锁，而LinkedBlockingQueue实现的队列中的锁是分离的，其添加采用的是putLock，移除采用的则是takeLock，这样能大大提高队列的吞吐量，也意味着在高并发的情况下生产者和消费者可以并行地操作队列中的数据，以此来提高整个队列的并发性能



** String为什么是不可变的 **
1、创建字符串时，如果该字符串已经存在于池中，则将返回现有字符串的引用，而不是创建新对象。
   多个String变量引用指向同一个内存地址。如果字符串是可变的，用一个引用更改字符串将导致其他引用的值错误。这是很危险的。
2、字符串的Hashcode在java中经常配合基于散列的集合一起正常运行，这样的散列集合包括HashSet、HashMap以及HashTable。
   不可变的特性保证了hashcode永远是相同的。不用每次使用hashcode就需要计算hashcode。这样更有效率。因为当向集合中插入对象时，
   是通过hashcode判别在集合中是否已经存在该对象了（不是通过equals方法逐个比较，效率低）
3、不可变对象不能被写，所以不可变对象自然是线程安全的，因为不可变对象不能更改，它们可以在多个线程之间自由共享。

由于效率和安全性的原因，字符串被设计为不可变