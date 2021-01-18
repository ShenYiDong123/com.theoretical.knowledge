### 版本号 ###
version=1.0-20200118-001

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
	
** JAVA IO流**
	按照流的流向分，可以分为输入流和输出流；
	按照操作单元划分，可以划分为字节流和字符流；
	按照流的⻆⾊划分为节点流和处理流。
	
	InputStream/Reader: 所有的输入流的基类，前者是字节输入流，后者是字符输入流。
	OutputStream/Writer: 所有输出流的基类，前者是字节输出流，后者是字符输出流。
	
	既然有了字节流,为什么还要有字符流?
		字符流是由 Java 虚拟机将字节转换得到的，问题就出在这个过程还算是非常耗时，并且，如果
	我们不知道编码类型就很容易出现乱码问题。所以， I/O 流就干脆提供了一个直接操作字符的接⼝，
	⽅便我们平时对字符进⾏流操作。如果音频文件、图片等媒体文件字节流⽐᫾好，如果涉及到字符的
	话使⽤字符流⽐᫾好。
	
** BIO,NIO,AIO 有什么区别? **
	BIO (Blocking I/O): 同步阻塞 I/O 模式，数据的读取写入必须阻塞在一个线程内等待其完
	成。在活动连接数不是特别高（小于单机 1000）的情况下，这种模型是不错的，可以让每
	一个连接专注于自己的个连接专注于自己的 I/O 并且编程模型简单，也不用过多考虑系统的过载、限流等问题。线
	程池本身就是个天然的漏斗，可以缓冲一些系统处理不了的连接或请求。但是，当然对十万甚
	至百万级连接的时候，传统的 BIO 模型是无能为力的。因此，我们需要一种更高效的 I/O 处理
	模型来应对更高的并发量。

	NIO (Non-blocking/New I/O): NIO 是⼀种同步非阻塞的 I/O 模型，在 Java 1.4 中引入了
	NIO 框架，对应 java.nio 包，提供了 Channel , Selector，Buffer 等抽象。NIO 中的 N 可
	以理解为 Non-blocking，不单纯是 New。它⽀持⾯向缓冲的，基于通道的 I/O 操作方法。 NIO
	提供了与传统 BIO 模型中的 Socket 和 ServerSocket 相对应的 SocketChannel 和
	ServerSocketChannel 两种不同的套接字通道实现,两种通道都支持阻塞和非阻塞两种模
	式。阻塞模式使用就像传统中的支持一样，比较简单，但是性能和可靠性都不好；非阻塞模式正
	好与之相反。对于低负载、低并发的应用程序，可以使用同步阻塞 I/O 来提升开发速率和更好
	的维护性；对于高负载、高并发的（网络）应用，应使使用 NIO 的非阻塞模式来开发

	AIO (Asynchronous I/O): AIO 也就是 NIO 2。在 Java 7 中引用了 NIO 的改进版 NIO 2,它是
	异步非阻塞的 IO 模型。异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返
	回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。AIO 是异
	步 IO 的缩写，虽然 NIO 在网络操作中，提供了非阻塞的做法，但是 NIO 的 IO 行为还是同步
	的。对于 NIO 来说，我们的业务线程是在 IO 操作准备好时，得到通知，接着就由这个线程自己进行
	IO 操作，IO 操作本身是同步的。查阅网上相关资料，我发现就目前来说 AIO 的应用还
	不是很广泛，Netty 之前也尝试使用过 AIO，不过又放弃了。
	
**  Arraylist 与 LinkedList 区别? **
	是否保证线程安全： ArrayList 和 LinkedList 都是不同步的，也就是不保证线程安全；
	底层数据结构： Arraylist 底层使⽤的是 Object 数组； LinkedList 底层使⽤的是双向链表 数据结构（JDK1.6之前为循环链表，JDK1.7取消了循环。注意双向链表和双向循环链表的区别，下⾯有介绍到！）
	插入和删除是否受元素位置的影响： ① ArrayList 采用数组存储，所以插入和删除元素时间复杂度受元素位置的影响。② LinkedList 采⽤链表存储，所以对于 add(​E e) ⽅法的插⼊，删除元素时间复杂度不受元素位置的影响，近似 O（1），如果是要在指定位置 i 插⼊和删除元素的话（ (add(int index, E element) ） 时间复杂度近似为 o(n)) 因为需要先移动到指定位置再插入。
	是否支持快速随机访问： LinkedList不支持高效的随机元素访问，而ArrayList 支持。快速随机访问就是通过元素的序号快速获取元素对象(对应于 get(int index) 用法)。
	内存空间占用： ArrayList的空间浪费主要体现在在list列表的结尾会预留一定的容量空间，而LinkedList的空间花费则体现在它的每个元素都需要消耗比ArrayList更多的空间（因为要存放直接后继和直接前驱以及数据）。
	
**  RandomAccess **
	`public interface RandomAccess {
	`}
	查看源码我们发现实际上 RandomAccess 接口中什么都没有定义。所以，在我看来RandomAccess 接口不过是个标识罢了。标识什么？ 标识实现这个接⼝的类具有随机访问功能。
	在 binarySearch（ ）用法中，它要判断传⼊的list 是否 RamdomAccess 的实例，如果是，调用indexedBinarySearch（）方法，如果不是，那么调用 iteratorBinarySearch（）方法
	`public static <T>
		`int binarySearch(List<? extends Comparable<? super T>> list, T key) {
		`if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
			`return Collections.indexedBinarySearch(list, key);
		`else
			`return Collections.iteratorBinarySearch(list, key);
	`}
	再总结下 list 的遍历方式选择：
	实现了 RandomAccess 接口的list，优先选择普通 for 循环 ，其次 foreach,如ArrayList
	未实现 RandomAccess 接口的list，优先选择iterator遍历（foreach遍历底层也是通过iterator实现的,），大size的数据，千万不要使用普通for循环，如linkedList
	
	
**  HashMap和HashTable的区别 **
	线程是否安全：HashMap 是非线程安全的，HashTable 是线程安全的；HashTable 内部的方法经过 synchronized 修饰。（如果你要保证线程安全的话就使用 ConcurrentHashMap吧！）；
	效率：因为线程安全的问题，HashMap 要比 HashTable 效率高点。另外，HashTable 基本被淘汰，不要在代码中使用它；
	对Null key 和Null value的支持： HashMap 中，null 可以作为键，这样的键只有一个，可以有一个或多个键所对应的值为 null。但是在 HashTable 中 put 进的键值只要有⼀个 null，直接抛出 NullPointerException。
	初始容量大小和每次扩充容量大小的不同 ： 
	①创建时如果不指定容量初始值，Hashtable 默认的初始容量为11，之后每次扩充，容量变为原来的2n+1。HashMap 默认的初始化容量为16。之后每次扩充，容量变为原来的2倍。
	②创建时如果给定了容量初始值，那么 Hashtable 会直接使用你给定的大小，而HashMap 会将其扩充为2的幂次方大小（HashMap 中的 tableSizeFor() 方法保证）。也就是说 HashMap 总是使用2的幂作为哈希表的大小
	底层数据结构： JDK1.8 以后的 HashMap 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间。Hashtable 没有这样的机制。