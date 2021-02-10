#### (十六)JMM

------

> 1.Volatile是虚拟机提供的**轻量级的同步机制**

​		1.保证可见性

​		2.不保证原子性

​		3.禁止指令重排

> 2.JMM：java的内存模型，不存在的东西是一个概念！

关于JMM的一些同步约定：

​	1.线程解锁前，必须把共享变量立刻刷回主存

​	2.线程加锁前，必须读取主存中的最新值到工作内存中！

​	3.加锁和解锁是同一把锁

线程，工作内存，主内存（8种操作）：

![image-20210110115056026](JUC(2).assets/image-20210110115056026.png)

但此时如果线程B对flag修改了并存储回主存线程A不能及时获取到最新值。

![image-20210110115426009](JUC(2).assets/image-20210110115426009.png)

**内存交互操作有8种，虚拟机实现必须保证每一个操作都是原子的，不可在分的（对于double和long类**
**型的变量来说，load、store、read和write操作在某些平台上允许例外）**

* lock （锁定）：作用于主内存的变量，把一个变量标识为线程独占状态

* unlock （解锁）：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量
  才可以被其他线程锁定

* read （读取）：作用于主内存变量，它把一个变量的值从主内存传输到线程的工作内存中，以便
  随后的load动作使用

* load （载入）：作用于工作内存的变量，它把read操作从主存中变量放入工作内存中

* use （使用）：作用于工作内存中的变量，它把工作内存中的变量传输给执行引擎，每当虚拟机
  遇到一个需要使用到变量的值，就会使用到这个指令

* assign （赋值）：作用于工作内存中的变量，它把一个从执行引擎中接受到的值放入工作内存的变
  量副本中

* store （存储）：作用于主内存中的变量，它把一个从工作内存中一个变量的值传送到主内存中，
  以便后续的write使用

* write （写入）：作用于主内存中的变量，它把store操作从工作内存中得到的变量的值放入主内
  存的变量中

**JMM对这八种指令的使用，制定了如下规则：**

* 不允许read和load、store和write操作之一单独出现。即使用了read必须load，使用了store必须
  write

* 不允许线程丢弃他最近的assign操作，即工作变量的数据改变了之后，必须告知主存

* 不允许一个线程将没有assign的数据从工作内存同步回主内存

* 一个新的变量必须在主内存中诞生，不允许工作内存直接使用一个未被初始化的变量。就是怼变量
  实施use、store操作之前，必须经过assign和load操作

* 一个变量同一时间只有一个线程能对其进行lock。多次lock后，必须执行相同次数的unlock才能解
  锁

* 如果对一个变量进行lock操作，会清空所有工作内存中此变量的值，在执行引擎使用这个变量前，
  必须重新load或assign操作初始化变量的值

* 如果一个变量没有被lock，就不能对其进行unlock操作。也不能unlock一个被其他线程锁住的变量
  对一个变量进行unlock操作之前，必须把此变量同步回主内存



#### (十七)Volatile

------

> 1.保证可见性

![image-20210110141447948](JUC(2).assets/image-20210110141447948.png)

> 2.不保证原子性

原子性：不可分割

线程A字执行任务时候不能被打扰，也不能被分割。要么同时成功，要么同时失败。

若保证原子性，则num结果最终应为20000。加lock和synchronize可保证原子性。

![image-20210110134551831](JUC(2).assets/image-20210110134551831.png)

如果不加lock和synchronized，怎么保证原子性？

![image-20210110123708708](JUC(2).assets/image-20210110123708708.png)

使用原子类解决原子性问题

![image-20210110134117792](JUC(2).assets/image-20210110134117792.png)

![image-20210110134713728](JUC(2).assets/image-20210110134713728.png)

> 3.指令重排

什么是指令重排：你写的程序，计算机并不是按照你写的那样去执行

源代码->编译器优化的重排->指令并行也可能重排->内存系统也可能重排->执行

 **处理器在进行指令重排的时候会考虑数据之间的依赖性。**

![image-20210110140949084](JUC(2).assets/image-20210110140949084.png)

> volatile避免指令重排：内存屏障，cpu指令。

作用：

1.保证特定的操作的执行顺序

2.可以保证某些变量的内存可见性（利用这些特性volatile实现了可见性）

![image-20210110141820807](JUC(2).assets/image-20210110141820807.png)



* volatile可以保证可见性，不能保证原子性，由于内存屏障，可以保证避免指令重排的现象产生！内存屏障在单例模式中使用的最多。

* 需要注意的是：volatile写是在前面和后面**分别插入内存屏障**，而volatile读操作是在**后面插入两个内存屏障**。
* happens-before：

![image-20210201001613096](JUC(2).assets/image-20210201001613096.png)



> 4.MESI(缓存一致性协议)

当CPU写数据时，如果发现操作的变量是共享变量，即在其他CPU中也存在该变量的副本，会发出信号通知其他CPU将该变量的缓存行置为无效状态，因此当其他CPU需要读取这个变量时，发现自己缓存中缓存该变量的缓存行是无效的，那么它就会从内存重新读取。

==那是怎么发现数据是否失效呢？==

* 嗅探：每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。

==嗅探的缺点==

由于Volatile的MESI缓存一致性协议，需要不断的从主内存嗅探和cas不断循环，无效交互会导致总线带宽达到峰值。

所以不要大量使用Volatile，至于什么时候去使用Volatile什么时候使用锁，根据场景区分。



#### (十八)彻底玩转单例模式

------

单例模式一定要构造器私有。保证一个类只有一个实例就是单例模式。

> 饿汉式单例：一上来就把内存的所有对象加进来

![image-20210110181725279](JUC(2).assets/image-20210110181725279.png)

> 懒汉式单例：需要用的时候再加载对象

```
//懒汉单例模式
public class LazyMan {

    private static boolean qinjiang = false;//五.1。那么此时通过一个标志位也可以避免单例被破坏。

    private LazyMan() {
        synchronized (LazyMan.class) {//五.2
            if (qinjiang == false) {
                qinjiang = true;
            } else {
                throw new RuntimeException("不要试图使用反射破坏异常");
            }
        }
        /*synchronized (LazyMan.class){//三。从双重检测升级成了三重检测，可以避免某一类反射
            //类的class只有一个，如果之前创建过了，那么就存在类的class了。再通过反射就不给他创建
            if (lazyMan!=null){
                throw new RuntimeException("不要试图使用反射破坏异常");
            }
        }*/
        System.out.println(Thread.currentThread().getName() + "ok");
    }

    private volatile static LazyMan lazyMan;

    //双重检测锁模式的懒汉式单例 DCL懒汉式
    public static LazyMan getInstance() {
        if (lazyMan == null) {
            synchronized (LazyMan.class) {
                if (lazyMan == null) {
                    lazyMan = new LazyMan();//一。不是一个原子性操作
                    /*1.分配内存空间 
                     *2.执行构造方法，初始化对象
                     * 3.把这个对象指向这个空间
                     *
                     * 123
                     * 132 A，可能会指令重排操作，
                     *     B
                     * 这时如果再进来一个线程B，看到A已经占了空间，就会以为有lazyMan了，就会直接执行return语句。
                     * 但其实线程A发生了指令重排，2还未执行。实际上lazyMan仍然是空的。
                     * 所以要给lazyMan变量加volatile禁止指令重排。
                     * */
                }
            }
        }
        return lazyMan;
    }


    public static void main(String[] args) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException, NoSuchFieldException {
/*//        LazyMan instance = LazyMan.getInstance();

        Constructor<LazyMan> declaredConstructor = LazyMan.class.getDeclaredConstructor(null);
        
        declaredConstructor.setAccessible(true);
        LazyMan instance2 = declaredConstructor.newInstance();//二。通过反射可以破坏单例，就是调用无参构造方法
        LazyMan instance = declaredConstructor.newInstance();//四。但是如果不通过new创造对象，两个对象都是反射创建那么单例模式又破坏了。
        System.out.println(instance);//com.beautifulboy.LazyMan@14ae5a5
        System.out.println(instance2);//com.beautifulboy.LazyMan@7f31245a*/

        //六。仍然可以通过改变类中私有属性破坏单例
        Field qinjiang = LazyMan.class.getDeclaredField("qinjiang");
        qinjiang.setAccessible(true);

        Constructor<LazyMan> declaredConstructor = LazyMan.class.getDeclaredConstructor(null);
        declaredConstructor.setAccessible(true);
        LazyMan instance = declaredConstructor.newInstance();

        qinjiang.set(instance, false);

        LazyMan instance2 = declaredConstructor.newInstance();

        System.out.println(instance);//com.beautifulboy.LazyMan@7f31245a
        System.out.println(instance2);//com.beautifulboy.LazyMan@6d6f6e28


    }

}
```

总结：单例模式是不安全的



> 枚举

反射不能破坏枚举的单例。

```
//enum枚举，本身也是一个类

public class EnumSingle{
    INSTANCE;
    public EnumSingle getInstance(){
        return INSTANCE;
    }

}

class Test{
    public static void main(String[] args) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
        EnumSingle instance1 = EnumSingle.INSTANCE;
        Constructor<EnumSingle> declaredConstructor=EnumSingle.class.getDeclaredConstructor(String.class,int.class);
        declaredConstructor.setAccessible(true);
        EnumSingle instance2 = declaredConstructor.newInstance();

        //NoSuchMethodException: com.kuang.single.EnumSingle.<init>()，与想象中的错误不一样。原本通过反射创建对象会报错“反射不能破坏枚举”。
        System.out.println(instance1);
        System.out.println(instance2);
    }

}
```

通过探究发现，枚举没有无参构造，只有有参构造。

![image-20210112125726222](JUC(2).assets/image-20210112125726222.png)

枚举类型最终反编译的源码（通过jad反编译），发现只有有参构造，所以上面通过反射无参构造对象报的错是“无此方法”：

![image-20210112125807724](JUC(2).assets/image-20210112125807724.png)

小结：

1.DCL懒汉式双重检测

​		锁类模板+volatile避免指令重排![image-20210128133401011](JUC(2).assets/image-20210128133401011.png)

2.通过反射破坏单例

![image-20210128134143758](JUC(2).assets/image-20210128134143758.png)

3.DCL升级为3重检测

![image-20210128134252564](JUC(2).assets/image-20210128134252564.png)

4.同一种反射破坏3重检测

![image-20210128134528147](JUC(2).assets/image-20210128134528147.png)

5.标志位

![image-20210128134627463](JUC(2).assets/image-20210128134627463.png)

6.反射获取标志位破坏单例

![image-20210128134741438](JUC(2).assets/image-20210128134741438.png)

6.只有枚举才能解决单例不被破坏

![image-20210128135025056](JUC(2).assets/image-20210128135025056.png)



> 静态匿明内部类

![image-20210110182510722](JUC(2).assets/image-20210110182510722.png)

在匿明内部类中创建Holder对象，在外部类中调用HOLDER。



#### (十九)深入理解CAS（乐观锁）

------

> CAS:就是原子类中的compareAndSet方法（比较并交换）

![image-20210110205532427](JUC(2).assets/image-20210110205532427.png)

> Unsafe类：java可以通过这个类操作内存（unsafe里有很多native方法）

![image-20210110210257257](JUC(2).assets/image-20210110210257257.png)

原子类中getAndIncrement方法是怎样实现++操作呢？他的底层就是调用unsafe类的getAndIncrement方法直接操作内存，效率高。

getAndIncrement方法的源码的compareAndSwapInt方法，是CPU更底层的CAS。第一个参数表示当前的对象，第二个参数表示当前对象内存的值。

![image-20210110210932975](JUC(2).assets/image-20210110210932975.png)

这里首先获取var1内存对象的偏移值为var5；然后使用CAS比较并交换，如果期望的var1，var2的值等于取出来的var5的值，那么执行后面内存地址的值+1的操作（var4为1）。

do while不停的旋转，直到值能够成功为止。这个是自旋锁。

![image-20210110212053286](JUC(2).assets/image-20210110212053286.png)

CAS：比较当前工作内存中的值和主存中的值，如果这个值是期望的，那么执行操作！如果不是就一直循环，因为底层是自旋锁。

缺点：

​	1.自旋锁会一直循换耗时

​	2.一次性只能保证一个共享变量的原子性

​	3.ABA问题



> CAS:ABA问题（狸猫换太子）

![image-20210110223613546](JUC(2).assets/image-20210110223613546.png)

```java
public class CASDemo {
    //CAS compareAndSet:比较并交换
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(2020);

        //期望，更新。如果我期望的值达到了就更新，否则不更新。CAS是CPU并发原语！
        // public final boolean compareAndSet(int expect, int update)
        //================捣蛋的线程=================
        System.out.println(atomicInteger.compareAndSet(2020, 2021));
        System.out.println(atomicInteger.get());

        System.out.println(atomicInteger.compareAndSet(2021, 2020));
        System.out.println(atomicInteger.get());
        //================期望的线程=================
        System.out.println(atomicInteger.compareAndSet(2020, 666));
        System.out.println(atomicInteger.get());

    }
}

```



#### (二十)原子引用

------

> 原子引用可以解决ABA问题，对应乐观锁思想！

带版本号的原子操作！

```java
public class CASDemo2 {
    public static void main(String[] args) {
        //注意，如果泛型是一个包装类，注意对象的引用问题
        AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(1, 1);

        //CAS compareAndSet：比较并交换
        new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();//获得版本号
            System.out.println("a1=>"+ stamp);

            try{
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            //version++
            atomicStampedReference.compareAndSet(1,2,
                    atomicStampedReference.getStamp(),atomicStampedReference.getStamp()+1);

            System.out.println("a2=>"+atomicStampedReference.getStamp());

            System.out.println(atomicStampedReference.compareAndSet(2, 1,
                    atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1));

            System.out.println("a3=>"+atomicStampedReference.getStamp());

        }, "a").start();


        //乐观锁的原理相同
        new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();//获得版本号
            System.out.println("b1=>" + atomicStampedReference.getStamp());

            try{
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println(atomicStampedReference.compareAndSet(1, 6,
                    stamp, stamp + 1));

            System.out.println("b2=>"+atomicStampedReference.getStamp());

        }, "b").start();
    }
}

```

![image-20210111004306720](JUC(2).assets/image-20210111004306720.png)

每增加一次CAS就将stamp值++，线程a，b同时获得锁此时stamp值为1；a线程休眠时间短所以先执行，a线程执行完后stmp为3，释放锁；b再执行，此时stamp为3，与一开始的stamp值不同，所以此次CAS操作为false。

**注意：
Integer 使用了对象缓存机制，默认范围是 -128 ~ 127 ，推荐使用静态工厂方法 valueOf 获取对象实例，而不是 new，因为 valueOf 使用缓存，而 new 一定会创建新的对象分配新的内存空间；**

![image-20210111004711250](JUC(2).assets/image-20210111004711250.png)



#### （二十一）各种锁的理解

**1.公平锁，非公平锁**

公平锁：非常公平，不能插队，先来后到。

非公平锁：可以插队（默认都是非公平）

```
public ReentrantLock() {
    sync = new NonfairSync();
}
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

**2.可重入锁**

也叫递归锁，拿到外面的锁之后，就可以拿到里面的锁，自动获得。里面的锁释放后，外面的锁才释放，其他线程才可以进来。

![image-20210111135312002](JUC(2).assets/image-20210111135312002.png)

> Synchronized 版

```
public class Demo01 {
    public static void main(String[] args) {
        Phone phone=new Phone();

        new Thread(()->{
            phone.sms();
        },"A").start();//拿到外面的锁后也就拿到了里面call的锁，
        // 只有A线程执行完后才会把里面和外面的锁释放，B线程才能进来。

        new Thread(()->{
            phone.sms();
        },"B").start();

    }
}
class Phone{
    public synchronized void sms(){
        System.out.println(Thread.currentThread().getName()+"sms");
        call();//拿到外面的锁后也就拿到了里面call的锁
    }
    public synchronized void call(){
        System.out.println(Thread.currentThread().getName()+"call");
    }
}
```

![image-20210111140036864](JUC(2).assets/image-20210111140036864.png)

> lock版

``` 

public class Demo01 {
    public static void main(String[] args) {
        Phone phone = new Phone();

        new Thread(() -> {
            phone.sms();
        }, "A").start();//拿到外面的锁后也就拿到了里面call的锁，
        // 只有A线程执行完后才会把里面和外面的锁释放，B线程才能进来。

        new Thread(() -> {
            phone.sms();
        }, "B").start();

    }
}
class Phone {

    Lock lock = new ReentrantLock();

    public void sms() {
        lock.lock();//细节问题：锁和解锁必须配对，否则会死锁在里面
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "sms");
            call();//这里也有一个锁
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
            lock.unlock();
        }
    }

    public synchronized void call() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "call");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

![image-20210111140904116](JUC(2).assets/image-20210111140904116.png)

小结：两种版本的锁都是可重入锁，一个线程A获得外面的锁后里面的锁也会自动获得，只有当A执行完后才能将锁释放，此时B线程才能进来获得锁执行线程。



**3.自旋锁**

CAS判断，如果不等于期望就一直while自旋循环，直到成为期望再交换。

自定义一个自旋锁：

```
public class SpinlockDemo1 {
    //Thread null
    AtomicReference<Thread> atomicReference=new AtomicReference<>();

    //加锁
    public void myLock(){
        Thread thread=Thread.currentThread();
        System.out.println(Thread.currentThread().getName()+"==>mylock");

        //自旋锁,只要他不成立就一直循环。
        while (!atomicReference.compareAndSet(null,thread)){

        }
    }

    //解锁
    public void myUnLock(){
        Thread thread=Thread.currentThread();
        System.out.println(Thread.currentThread().getName()+"==>myUnlock");
        atomicReference.compareAndSet(thread,null);
    }
}

```

> 测试

``` 
public class Test2 {
    public static void main(String[] args) throws InterruptedException {
        SpinlockDemo1 lock=new SpinlockDemo1();

        new Thread(()->{
            lock.myLock();
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }finally {
                lock.myUnLock();
            }
        },"T1").start();


        TimeUnit.SECONDS.sleep(1);

        new Thread(()->{
            lock.myLock();
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }finally {
                lock.myUnLock();
            }
        },"T2").start();
    }
}
```

T1拿到锁，T2在等待T1解锁进行自旋，T1解锁后，T2拿到锁停止自旋。

T2必须等T1解锁完，才能进入锁，才能开始执行休眠代码。

**4.死锁**

什么是死锁：线程1拿着A想要拿B，线程2拿着B想要拿A。都想拿对方的资源。

```
public class DeadLockDemo {
    public static void main(String[] args) {

        String lockA="lockA";
        String lockB="lockB";

        new Thread(new MyThread(lockA,lockB),"T1").start();
        new Thread(new MyThread(lockB,lockA),"T2").start();
    }
}
class MyThread implements Runnable{

    private String lockA;
    private String lockB;

    public MyThread(String lockA, String lockB) {
        this.lockA = lockA;
        this.lockB = lockB;
    }

    @Override
    public void run() {
        synchronized (lockA){
            System.out.println(Thread.currentThread().getName()+"lock:"+lockA+"=>"+lockB);

            try{
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            synchronized (lockB){


            }
        }
    }
}
```

> 解决问题

1.使用`jps -l`定位进程号

![	](JUC(2).assets/image-20210111150128149.png)

2.使用`jstack 进程号`找到死锁

![image-20210111150412350](JUC(2).assets/image-20210111150412350.png)





#### （二十二）AQS

* ReentrantLock lock=new ReentrantLock；lock.lock();   它的加锁底层就是通过AQS实现的；它是非公平锁，可以插队，也可以new的时候传入参数true让它变成公平锁。

* AQS（Abstract Queue Synchronized），抽象队列同步器，里面有一个state变量和等待队列![image-20210131233329981](JUC(2).assets/image-20210131233329981.png)

* AQS原理：

  1.如果有两个线程同时要执行lock操作，那么会同时向AQS执行CAS操作更新state=1,；

  2.只有一个线程能CAS成功，成功的执行代码，在AQS有一个加锁线程；失败的会进入等待队列。

  3.当加锁线程执行完后会将state变为0，加锁线程置null；然后唤醒等待队列队头的线程。

* 我们熟知的ReentrantLock，ReentrantReadWriteLock，CountDownLatch，Semaphore和CyclicBarrier就是基于AQS实现的。



#### （二十三）ThreadLocal

* 通常情况下，我们创建的变量是可以被任何一个线程访问并修改的，如果想实现每个线程都有自己的专属本地变量，那么就要使用ThreadLocal。这个类主要解决让每个线程绑定自己的值。
* 使用：

``` 
public class TheadLocalTest {
    private static ThreadLocal<Student> threadLocalStudent = new ThreadLocal<>();
    public static void main(String[] args) {
        // 简单写一个测试线程隔离的例子
        // 原料: 1个ThreadLocal类型的变量 2个线程
        // 期望结果:线程一set的变量 线程二get不到！
        new Thread(()->{
            Student student = new Student();
            System.out.println("线程一保存的对象:"+student.toString());
            threadLocalStudent.set(student);
        }).start();
        new Thread(()->{
            try {
                // 细节！！！ 先睡一会再get避免误差。
                // 可见这是一个严谨性很高的测试Demo
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("期望结果:线程二获取的结果是null,说明线程二拿不到线程一存入的值");
            System.out.println("线程二获取结果:"+threadLocalStudent.get());
        }).start();
    }
}
```

结果：

![image-20210201195740294](JUC(2).assets/image-20210201195740294.png)

线程1存入的值别的线程获取不到，只有存入值的线程才能获取到值。

* 原理：

每个线程都有一个threadLocalMap变量，里面有entry数组存放key-value。

![image-20210201202635764](JUC(2).assets/image-20210201202635764.png)

set方法：获取当前的线程t，然后getMap(t)获取到当前线程的ThreadLocalMap，有则直接set(threadLocal，value)，无则createMap（t，value）。![image-20210201204638372](JUC(2).assets/image-20210201204638372.png)



get方法：![image-20210201204836949](JUC(2).assets/image-20210201204836949.png)

* 内存泄露问题：![image-20210201205533209](JUC(2).assets/image-20210201205533209.png)





















