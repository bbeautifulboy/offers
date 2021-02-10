#### （一）==和equals

==：基本数据类型 ==比较的是值，引用数据类型比较的是内存地址

![image-20210121235022912](java%E5%9F%BA%E7%A1%80.assets/image-20210121235022912.png)



equals：String 类中重写了 equals() 方法用于比较两个字符串的内容是否相等。若没有重写，则Object类的equals方法是比较对象的内存地址。

![image-20210121235440160](java%E5%9F%BA%E7%A1%80.assets/image-20210121235440160.png)

#### （2）hashcode和equals

![image-20210122231118561](java%E5%9F%BA%E7%A1%80.assets/image-20210122231118561.png)

* hashCode（）的作用是获取哈希码，哈希码的作用是确定该对象在哈希表中的索引位置。这个方法在Object上，这就意味着java任何类都有hashCode（）方法。
* 这个方法的作用就是减少equals的次数，提高执行速度。



#### （三）自动装箱与自动拆箱

装箱：将基本类型用他们对应的引用类型包装起来

拆箱：将包装类型转换为基本数据类型

![image-20210123145041464](java%E5%9F%BA%E7%A1%80.assets/image-20210123145041464.png)

==为什么要包装类型？==

1.包装类是对基础类型数据的包装，让基础数据类型“伪装”成类，具有类的特性，可以进行类的方法操作

2.因为java是个面向对象语言，所有操作都是基于对象。Object类是java对象的基础，所有java类都有个共同的 始祖Object类。但java的基础数据类型数据不是类，没有对象操作。所有需要一个中间类将Object类型和基础数据类型建立联系。



对象的相等比较的是内存中存放的值是否相等，而引用相等，比较的是他们指向的内存地址是否相等。



#### （四）static和final

> static

* 静态方法是由static修饰的方法，在类定义的时候已经被装载和分配；只能调用静态成员或方法，非静态方法要实例化后才能调用（因为非静态方法只有new后才会在堆中分配内存，这个方法才会存在，可以被调用到）。

* 静态方法的产生是为了减少资源的浪费，无需实例化即可使用，使用整个类共享的。



> final

* 表示不可改变的，final修饰的类不可被继承，方法不可被重写，变量不可改变。



#### （五）接口和抽象类

![image-20210123162932642](java%E5%9F%BA%E7%A1%80.assets/image-20210123162932642.png)

![image-20210123163028774](java%E5%9F%BA%E7%A1%80.assets/image-20210123163028774.png)

```
//定义接口，含有钻火圈方法
  public interface DrillFireCircle() {
      public abstract void drillFireCircle();
  }
  ​
  //定义抽象类狗类
  public abstract class Dog {
      public abstract void eat();
      public abstract void sleep();
  }
   
  //继承抽象类且实现接口
  class SpecialDog extends Dog implements drillFireCircle {
      public void eat() {
        //....
      }
      public void sleep() {
        //....
      }
      public void drillFireCircle() () {
        //....
      }
  }


```

总结：

继承是一个 "是不是"的关系，而 接口 实现则是 "有没有"的关系。如果一个类继承了某个抽象类，则子类必定是抽象类的种类，而接口实现则是有没有、具备不具备的关系，比如狗是否能钻火圈，能则可以实现这个接口，不能就不实现这个接口。



#### （六）String StringBuilder StringBuffer

* String是不可变的，因为他的底层用的是final修饰的char数组；而另外两个都是继承AbstractStringBuilder，他的源码没有final修饰char数组。所以是可变的。![image-20210123164725609](java%E5%9F%BA%E7%A1%80.assets/image-20210123164725609.png)

![image-20210123164708889](java%E5%9F%BA%E7%A1%80.assets/image-20210123164708889.png)



* String是不可变的所以是线程安全，StringBuilder是加了synchronize锁的所以也是线程安全。但StringBuffer没有加锁所以是线程不安全。

![image-20210123164916042](java%E5%9F%BA%E7%A1%80.assets/image-20210123164916042.png)

![image-20210123164939680](java%E5%9F%BA%E7%A1%80.assets/image-20210123164939680.png)

* 加锁了效率肯定就会低，所以StringBuffer的效率是比StringBuilder低10%-15%。

![image-20210123165203923](java%E5%9F%BA%E7%A1%80.assets/image-20210123165203923.png)



#### （七）NIO BIO AIO

![image-20210204011309808](java%E5%9F%BA%E7%A1%80.assets/image-20210204011309808.png)

> BIO原理：

BIO，就是同步阻塞式IO。服务端创建一个ServerSocket，然后客户端用一个Socket去连接ServerSocket；然后ServerSocket接收到一个Socket的连接请求就创建一个Socket和一个线程去跟那个Socket进行通信。

客户端和服务端的socket进行同步阻塞式的通信，客户端socket发送一个请求，服务端socket进行处理后返回响应，响应必须等处理完后才会返回，在这之前啥也不能干。

缺点就是每次一个客户端接入都要在服务端创建一个线程，所以当有大量客户端的时候，服务端线程也要大量，这就导致了服务器直接负载过高死掉。或者用线程池，但在高并发的情况下也会导致排队延迟。

![image-20210204161012242](java%E5%9F%BA%E7%A1%80.assets/image-20210204161012242.png)



> NIO原理：

NIO，同步非阻塞IO，基于Reactor模型。客户端每发过来一个请求都会在Selector注册一个Channel，客户端通过这个Channel进行连接。

Selector，就是多路复用器，它会不断的轮询注册的Channel；如果有某一个客户端发送请求过来Channel了，Selector就会创建一个线程出来，读请求参数，然后将响应结果发回去，线程发完结果后就没了。

==优化：==

1.通过一个Cached线程池就可以减少线程创建销毁的次数，提升效率。

2.Channel和工作线程中间基于一个Buffer读取数据，通道就可以进行异步的读写。



![image-20210204162838433](java%E5%9F%BA%E7%A1%80.assets/image-20210204162838433.png)

















