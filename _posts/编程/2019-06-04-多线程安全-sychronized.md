---
layout: post
title: "多线程安全-sychronized"
categories: [编程]
tags: [Java,多线程]
published: True


---

## 造成线程数据错乱的三要素 ( 同时也是保持线程安全的要素 )

### 名词解释

- **原子性**（Synchronized, Lock）即一个操作或者多个操作，要么执行 要么就都不执行，在执行过程中不可打断
- **有序性** (Volatile，Synchronized, Lock)  程序默认的执行顺序
- **可见性**  (Volatile，Synchronized,Lock)  当变量被修改时，所修改的值会被立即同步到主存中，其他程序或者进程再读取时获取的值是最新的

## synchronized的三种应用方式

- 修饰实例方法，作用于当前实例加锁，进入同步代码前要获得当前实例的锁

  

  ```java
  public class AccountingSync implements Runnable{
      //共享资源(临界资源)
      static int i=0;
  
      /**
       * synchronized 修饰实例方法
       * 暂时先不添加
       */
      public void increase(){
          i++;
      }
      @Override
      public void run() {
          for(int j=0;j<1000000;j++){
              increase();
          }
      }
      public static void main(String[] args) throws InterruptedException {
          AccountingSync instance=new AccountingSync();
          Thread t1=new Thread(instance);
          Thread t2=new Thread(instance);
          t1.start();
          t2.start();
          t1.join();
          t2.join();
          System.out.println(i);
      }
      /**
       * 输出结果:
       * 1802452
       */
  }
  ```

  

  **i++** 赋值操作并没有 原子性 在读取原来的参数和返回新的参数的这段时间中，如果我们第二个线程也做了同样的操作，两个线程看到的参数是一样的，同样执行了 +1 的操作，这里就造成了线程安全破坏，我们最终输出的结果是 1802452 

  如果我们添加上 synchronized 此时 increase() 方法在一个时间内 只能被一个线程读写，也就避免脏数据的产生。

  

  但是这样也不是安全的，如果我们 new 出两个 AccountingSync 对象去执行操作 结果是怎么样？

  ```java
  public class AccountingSyncBad implements Runnable{
      static int i=0;
      public synchronized void increase(){
          i++;
      }
      @Override
      public void run() {
          for(int j=0;j<1000000;j++){
              increase();
          }
      }
      public static void main(String[] args) throws InterruptedException {
          //new新实例
          Thread t1=new Thread(new AccountingSyncBad());
          //new新实例
          Thread t2=new Thread(new AccountingSyncBad());
          t1.start();
          t2.start();
          //join含义:当前线程A等待thread线程终止之后才能从thread.join()返回
          t1.join();
          t2.join();
          System.out.println(i);
      }
  }
  ```

  结果依旧是产生了脏数据，原因是两个实例对象锁并不同相同，此时如果两个线程操作数据并非共享的，线程安全是有保障的，遗憾的是如果两个线程操作的是共享数据，安全将没有保障，他们本身的锁只能保持本身

- 修饰静态方法，作用于当前类对象加锁，进入同步代码前要获得当前类对象的锁

  ```java
  public class AccountingSyncClass implements Runnable{
      static int i=0;
  
      /**
       * 作用于静态方法,锁是当前class对象,也就是
       * AccountingSyncClass类对应的class对象
       */
      public static synchronized void increase(){
          i++;
      }
  
      /**
       * 非静态,访问时锁不一样不会发生互斥
       */
      public synchronized void increase4Obj(){
          i++;
      }
  
      @Override
      public void run() {
          for(int j=0;j<1000000;j++){
              increase();
          }
      }
      public static void main(String[] args) throws InterruptedException {
          //new新实例
          Thread t1=new Thread(new AccountingSyncClass());
          //new心事了
          Thread t2=new Thread(new AccountingSyncClass());
          //启动线程
          t1.start();t2.start();
  
          t1.join();t2.join();
          System.out.println(i);
      }
  }
  ```

  这个实例当中我们将 synchronized 作用于静态方法了,因为静态方法不属于任何实例对象，它是类成员，所以这把锁也可以理解为加在了 Class 上 ，但是如果我们线程A 调用了 class 内部 static synchronized 方法 线程B 调用了 class 内部 非 static 方法 是可以的，不会发生互斥现象,因为访问静态 synchronized 方法占用的锁是当前类的class对象，而访问非静态 synchronized 方法占用的锁是当前实例对象锁

- 修饰代码块，指定加锁对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁。

  ```java
  public class AccountingSync implements Runnable{
      static AccountingSync instance=new AccountingSync();
      static int i=0;
      @Override
      public void run() {
          //省略其他耗时操作....
          //使用同步代码块对变量i进行同步操作,锁对象为instance
          synchronized(instance){
              for(int j=0;j<1000000;j++){
                      i++;
                }
          }
      }
      public static void main(String[] args) throws InterruptedException {
          Thread t1=new Thread(instance);
          Thread t2=new Thread(instance);
          t1.start();t2.start();
          t1.join();t2.join();
          System.out.println(i);
      }
  }
  ```
## synchronized 是如何实现的同步
### 同步代码块
内部使用 monitorenter 和 monitorexit 指令实现，monitorenter 指令插入到同步代码块的开始位置，monitorexit 指令插入到同步代码块的结束位置，jvm需要保证每一个monitorenter都有一个 monitorexit 与之对应。任何一个对象都有一个 monitor 与之相关联，当它的monitor被持有之后，它将处于锁定状态。线程执行到 monitorenter 指令前，将会尝试获取对象所对应的 monitor 所有权，即尝试获取对象的锁；将线程执行到 monitorexit 时就会释放锁。

(人话:)每个对象都会与一个monitor相关联，当某个monitor被拥有之后就会被锁住，当线程执行到monitorenter指令时，就会去尝试获得对应的monitor。步骤如下：

1. 每个monitor维护着一个记录着拥有次数的计数器。未被拥有的monitor的计数器为0，当一个线程获得monitor（执行monitorenter）后，该计数器自增变为 1 。当同一个线程再次获得该monitor的时候，计数器再次自增；当不同线程想要获得该monitor的时候，就会被阻塞。

 2. 当同一个线程释放monitor（执行monitorexit指令）的时候，计数器再自减。当计数器为0的时候。monitor将被释放，其他线程便可以获得monitor。
![image](https://upload-images.jianshu.io/upload_images/1209392-d59f422666cee8c4?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###同步方法
不过相对于普通方法，其常量池中多了ACC_SYNCHRONIZED标示符。JVM就是根据该标示符来实现方法的同步的：当方法调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先获取monitor，获取成功之后才能执行方法体，方法执行完后再释放monitor。

在jvm字节码层面并没有任何特别的指令来实现synchronized修饰的方法，而是在class文件中将该方法的access_flags字段中的acc_synchronized标志位设置为1，表示该方法为synchronized方法。

在java设计中，每一个对象自打娘胎里出来就带了一把看不见的锁，即monitor锁。monitor是线程私有的数据结构，每一个线程都有一个monitor record列表，同时还有一个全局可用列表。每一个被锁住对象都会和一个monitor关联。monitor中有一个owner字段存放拥有该对象的线程的唯一标识，表示该锁被这个线程占有。owner：初始时为null，表示当前没有任何线程拥有该monitor，当线程成功拥有该锁后，owner保存线程唯一标识，当锁被释放时，owner又变为null。
![image](https://upload-images.jianshu.io/upload_images/1209392-9f235bc66e910217?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
总结：
同步方法和同步代码块底层都是通过monitor来实现同步的。两者的区别：同步方式是通过方法中的access_flags中设置ACC_SYNCHRONIZED标志来实现；同步代码块是通过monitorenter和monitorexit来实现我们知道了每个对象都与一个monitor相关联。而monitor可以被线程拥有或释放。
