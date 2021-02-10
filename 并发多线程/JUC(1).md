#### （一） synchronized（悲观锁）和lock

* ==synchronized例子==：

![image-20210113163510603](JUC(1).assets/image-20210113163510603.png)

* ==Lock锁例子==：

​	1.new ReentrantLock(); //新建一个可重入锁

​	2.lock.lock(); //加锁

​	3.lock.unlock();//解锁

![image-20210113163624093](JUC(1).assets/image-20210113163624093.png)

![image-20210113163646532](JUC(1).assets/image-20210113163646532.png)

* ==两者区别==：

  1.synchronized内置的java关键字，Lock是一个java类

   2.synchronized无法判断获取锁的状态，Lock可以判断是否获取到了锁

  3.synchronized会自动释放锁，lock必须手动释放锁！否则死锁

  4.synchronized线程1（获得锁，阻塞），线程2（等待）；Lock锁就不一定会等待下

  5.synchronized可重入锁，不可中断，非公平的；Lock，可重入锁，可以判断锁，非公平（可以自己设置）

  6.synchronized适合少量的代码同步问题；Lock适合锁大量同步代码





>  ==synchronized==

![image-20210131162248257](JUC(1).assets/image-20210131162248257.png)

* 用在同步代码块：可以指定任意的锁；

  ![image-20210131220626591](JUC(1).assets/image-20210131220626591.png)

  用于方法上：锁的是当前实例对象this；

  ![image-20210131220648845](JUC(1).assets/image-20210131220648845.png)

  用在静态方法：锁的是它的class对象

* 在JVM中，对象在内存中分为三块区域：

  * 对象头：
    * Mark Word（标记字段）：默认存储对象的HashCode，分代年龄和锁标志位信息，这些信息会随着锁标志位的变化而变化。
    * Klass Point（类型指针）：虚拟机提供这个指针确定对象是哪个类的视实例。

  * 实例数据：存放类的数据信息

  * 对齐填充：对象起始地址填充为8的整数倍。

    ![image-20210131182802861](JUC(1).assets/image-20210131182802861.png)

* 锁升级碰胀过程：无锁->偏向锁->轻量级锁->（自旋）重量级锁

![4491294-e3bcefb2bacea224](JUC(1).assets/4491294-e3bcefb2bacea224.png)

1.锁消除：JIT编辑器对synchronized锁做优化，在编译期间JIT通过逃逸分析技术分析synchronized锁对象，如果只有一个线程使用编译就不加mointorenter和monitorexit指令。提升代码效率。

2.锁粗化：JIT发现有代码连续加锁，就会将其合并为一个，避免频繁加锁释放锁。



3.偏向锁：可能只有一个线程竞争锁，但也可能会有其他线程竞争，只不过概率很小。

所以就采用偏向锁：mointorenter和mointorexit是要使用cas操作加锁和释放锁，开销大；

但因为大概率只有一个线程竞争锁，所以就给整个锁设置一个偏好（Bias），后面加锁和释放锁基于Bias执行，不需要经过cas，效率提高很多。

如果有其他线程竞争整个锁，此时就会收回之前那个线程分配的Bias偏好。



4.轻量级锁：如果偏向锁没能实现成功，就是因为不同线程竞争太频繁，此时偏向锁升级就会变成轻量级锁。

就是将对象头的Mark Word里的一个轻量级锁指针，尝试指向持有锁的线程，然后判断是否是自己加的锁，如果是自己加的锁那就指行代码；

如果不是自己加的锁那就加锁失败，说明有其他人加了锁，这个时候就升级为重量级锁。（自旋无果后升级为重量级锁）



5.重量级锁：![image-20210131222302173](JUC(1).assets/image-20210131222302173.png)





#### （二）生产者消费者

判断等待，业务，通知。资源类中的方法要用while让其一直等待，否则当有4个下线程开启时会产生虚假唤醒，即用if这一次判断之后，可能别的进程进去了，一样造成num--。 

synchronize版：

![image-20210113162934781](JUC(1).assets/image-20210113162934781.png)

lock版（等待唤醒方法变为await和signalAll）：

![image-20210113163053356](JUC(1).assets/image-20210113163053356.png)

![image-20210113163129297](JUC(1).assets/image-20210113163129297.png)

lock锁可以通过condition实现精准通知唤醒：

![image-20210113163215453](JUC(1).assets/image-20210113163215453.png)

![image-20210113163258350](JUC(1).assets/image-20210113163258350.png)

![image-20210113163343493](JUC(1).assets/image-20210113163343493.png)

#### （三）锁

1.synchronized锁的是对象（方法的调用者），有两个对象则有两把锁。获得锁的线程必须释放后其他线程才能获得。普通方法不是同步方法不受影响。

![image-20210113162614560](JUC(1).assets/image-20210113162614560.png)

![image-20210113162654318](JUC(1).assets/image-20210113162654318.png)



2.当synchronized前加上static修饰时，此时是两个静态同步方法，类一加载就有静态方法了， 锁的是Class模板了。而不是对象，Class模板只有一个。说白了就是无static锁的是phone对象，有static后锁的是Phone3的Class模板了。但没加static则两个对象锁的是两个不同的对象，互不相干。

![image-20210113162304699](JUC(1).assets/image-20210113162304699.png)

![image-20210113162331362](JUC(1).assets/image-20210113162331362.png)

此时对象锁的是class模板，尽管有两个对象，但不锁它。对象调用发短信方法先获得锁（class模板），此方法执行完后才执行后面的。所有先打印发短信。



3.如果既有静态同步方法和普通同步方法，有两个对象，则多线程中等待时间少的方法先输出。因为这有两个锁，一个锁的是调用者（对象），一个锁的是Class类模板，两者互不相干，此时等待时间少的方法先执行。一个方法阻塞，cpu会调度另一个方法，而这两个方法又不是锁同一个东西，所以等待时间少的先执行。

![image-20210113162418675](JUC(1).assets/image-20210113162418675.png)

![image-20210113162434843](JUC(1).assets/image-20210113162434843.png)

> 小结：new 对象->具体一个手机；static Class 唯一的一个模板



#### （四）不安全集合

* 1.有逼格的异常：ConcurrentModificationException并发修改异常，StackOverflow栈溢出，OM内存溢出

* 2.并发下ArrayList是不安全的，产生并发修改异常。

![Image](JUC(1).assets/Image-1610527119912.png)

解决方案：

1.List<String> list=new Vector<>(); 源码下是synchronize修饰所以线程安全，但是效率低

2.List<String> list=Collentions.synchronizedList(new ArrayList<>());collection类下的一个工具类

3.List<String> list=new CopyOnWriteArrayList<>(); JUC包下的类，它的源码没有用synchronize，而是采用读写分离的形式（写入时复制），既保证了效率又保证了安全。

![Image [2]](JUC(1).assets/Image%20%5B2%5D.png)

* 3.不安全的set集合：同list一样，多线程也会产生并发修改异常

![Image [3]](JUC(1).assets/Image%20%5B3%5D.png)

解决：

1.Set<String> set= Collections.synchronizedSet(new HashSet<>()); 采用工具类将它变为synchronizedSet保证安全。

2.Set<String> set=new CopyOnWriteArraySet<>(); 写入时复制保证效率和性能问题(COW，计算机程序设计领域的一种优化策略)



* 4.HashSet的底层是什么？就是HashMap

![Image [4]](JUC(1).assets/Image%20%5B4%5D.png)

add的本质就是map中的key，所以set是无序的，不重复的。



* 5.不安全的map集合：map集合也是一样，多线程也会产生并发修改异常

解决：

1.Map<String,String> map=Collection.synchronizedMap(new HashMap()); 工具类

2.Map<String,String> map=new ConcurrentHashMap<>(); 和list和set一样，读写分离。它的原理从jdk文档查看。



#### （五）Callable

第三种创建线程的方法（可以有返回值；可以抛出异常；方法不同，call())  。细节：有缓存，结果可能需要等待，会阻塞！

![Image [5]](JUC(1).assets/Image%20%5B5%5D.png)

#### （六）常用的辅助类

* 1.CountDownLatch：减法计数器，countDown()；数量减1。await（）；等待计数器归零，然后再向下执行。

![Image [6]](JUC(1).assets/Image%20%5B6%5D.png)

* 2.CyclicBarrier：加法计数器

  ![Image [7]](JUC(1).assets/Image%20%5B7%5D.png)

* 3.Semaphore：信号量。acquire得到资源，release释放资源。

![Image [8]](JUC(1).assets/Image%20%5B8%5D-1610527563572.png)

#### （七）读写锁（独占锁和共享锁）

ReadWriteLock和lock一样，只不过更加细粒度的控制（写只可以有一个线程写，读可以有多个线程读），否则写入可能会被别的线程插队，造成并发修改异常。

![Image [9]](JUC(1).assets/Image%20%5B9%5D.png)

![Image [10]](JUC(1).assets/Image%20%5B10%5D.png)

#### （八）阻塞队列

他不是新的东西，跟list和set类是平级的。，常用在多线程并发处理，线程池。

![Image [11]](JUC(1).assets/Image%20%5B11%5D.png)

四组API：

![Image [12]](JUC(1).assets/Image%20%5B12%5D.png)

![Image [13]](JUC(1).assets/Image%20%5B13%5D.png)

#### （九）同步队列

和其他blockingqueue不一样，SynchronousQueue不存储元素，put了一个元素，必须从里面take出来，否则不能在put值进去。

![Image [14]](JUC(1).assets/Image%20%5B14%5D.png)

#### （十）线程池

线程复用，可以控制最大并发数，管理线程。



* 1.三种方法创建线程池（阿里巴巴开发手册不推荐，推荐通过ThreadPoolExecutor创建，否则可能会创建大量的线程，从而导致OOM）：

![Image [15]](JUC(1).assets/Image%20%5B15%5D.png)

* 2.七大参数：从源码得知三种创建线程池的方法本质都是调用ThreadPoolExecutor。

![Image [16]](JUC(1).assets/Image%20%5B16%5D.png)

打开源码发现ThreadPoolExecutor方法里面有七大参数：

![Image [17]](JUC(1).assets/Image%20%5B17%5D.png)

* 3.手动创建线程池

![Image [18]](JUC(1).assets/Image%20%5B18%5D.png)

* 4.四种拒绝策略

![Image [19]](JUC(1).assets/Image%20%5B19%5D.png)

1.new ThreadPoolExecutor.AbortPolicy() //银行满了，还有人进来，不处理这个人的，抛出异常

2.new ThreadPoolExecutor.CallerRunsPolicy() //哪来的去哪里！

3.new ThreadPoolExecutor.DiscardPolicy() //队列满了，丢掉任务，不会抛出异常！

4.new ThreadPoolExecutor.DiscardOldestPolicy() //队列满了，尝试去和最早的竞争，也不会抛出异常！

* 5.线程池的最大大小如何去设置：

CPU密集型：电脑是几核就是几，保持CPU效率最高。代码获取电脑核数：

![Image [20]](JUC(1).assets/Image%20%5B20%5D.png)

IO密集型：判断程序中十分耗IO的线程，一般设为2倍，有15个大型任务，给他30个线程。

#### （十一）四大函数式接口

只有一个抽象方法的接口。用来简化编程模型。（@FunctionalInterface）

* 1.Function函数式接口(输出输入值)：

![Image [21]](JUC(1).assets/Image%20%5B21%5D.png)

* 2.Predicate断定型接口：一个输入值，返回布尔值

  ![Image [22]](JUC(1).assets/Image%20%5B22%5D.png)

* 3.Consumer消费型接口：只有输入，没有返回值

  ![Image [23]](JUC(1).assets/Image%20%5B23%5D.png)

* 4.Supplier供给型接口：没有参数，只返回

  ![Image [24]](JUC(1).assets/Image%20%5B24%5D.png)



#### （十二）Stream流式计算

![Image](JUC(1).assets/Image-1610528150897.png)



#### （十三）ForkJoin

在大数据量时使用，将大任务拆分为小任务。特点：工作窃取，自己的子任务执行完后窃取别人的子任务执行（都是双端队列）， 提高效率。

使用：

1.通过forkjoinPool来执行 

2.计算任务 forkjoinPool.execute(ForkJoinTask task)

3.计算类要继承ForkJoinTask

* 计算类：

![Image [2]](JUC(1).assets/Image%20%5B2%5D-1610528223821.png)

* 测试类：

![Image [3]](JUC(1).assets/Image%20%5B3%5D-1610528244766.png)

* 采用并行流计算会更快：

![Image [4]](JUC(1).assets/Image%20%5B4%5D-1610528262468.png)

#### （十四）异步回调

获取将来的结果。类似于ajax。

无返回值的runAsync异步回调：

![Image [5]](JUC(1).assets/Image%20%5B5%5D-1610528295955.png)

有返回值的supplyAsync异步回调：

![Image [6]](JUC(1).assets/Image%20%5B6%5D-1610528321269.png)





