## 2019高频笔试面试

### 1.volatile是java虚拟机提供的轻量级同步机制（用于变量的声明）

```
//volitale举例 
private static volatile Singleton instance;

读取速度
硬盘<内存<CPU

线程数量
Thread.activeCount()
线程等待 
Thread.yield();
当前线程名字
Thread.currentThread().getName()
进程睡眠
TimeUnit.SECONDS.sleep(3);
```

#### 			1.1 	 volatile与synchronize比较

```java
1）关键字volatile是线程同步的轻量级实现，所以性能上肯定比synchronized要好。不过随着JDK的更新，synchronized关键字的效率得到了很大的提升，所以一般性的用synchronized关键字还是比较多的，性能也非常好。

2）volatile只能修饰变量，而synchronized关键字可以修饰方法，代码块等，功能更全，当然操作也更麻烦。

3）多线程访问volatile不会发生阻塞，而synchronized会阻塞。

4）volatile能保证数据的可见性，但不能保证原子性。而synchronized关键字既可以保证可见性又能保证原子性，因为它会将私有内存和公共内存中的数据做同步。
    
5）关键字synchronized可以保证在同一时刻，只有一个线程可以执行某一个方法或者某一段代码块。它包含两个特性：互斥性和可见性，同步synchronized不仅可以解决一个线程看到对象处于不一致的状态，它还可以保证进入同步方法或者同步代码块的每个线程，都看到由同一个锁保护之前的所有最新修改效果。
```

#### 			1.2 	volatile三大特性

```java
    三大特性
    1.1保证可见性（A,B两个线程同时操作一个内存，A操作完成之后，B线程可以见到主内存的值）
        class MyDate{
    int number=0;
    public void addTo60(){
        this.number=60;
    }
    public void addPlus(){
        number++;
    }
}
public class Vilatile {
    public static void main(String[] args) {
        MyDate myDate = new MyDate();
        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"\t come in");
            try{
                TimeUnit.SECONDS.sleep(3);
                myDate.addTo60();
                System.out.println(myDate.number);
            }catch (Exception e){
                e.printStackTrace();
            }
        },"aaa").start();
        while(myDate.number==0){//number不为60一直在这里转
        }
        System.out.println("完成");
    }
    	变量如果不加volatile永远不会打印出"完成"，因为没有可见性
    1.2不保证原子性（原子性是指不可分割，也即某个线程在做某个业务时候，中间不可以被加塞或者被分割，需要整体完整，要么同时成功，要么同时失败）
 class MyDate{
    volatile int number=0;
    public void addTo60(){
        System.out.println("我在这里添加了");
        this.number=60;
    }
    public void addPlus(){
        number++;
    }
}
public class Vilatile {
    public static void main(String[] args) {
        getAtomicInteger();
    }
    //保证原子性
    private static void getAtomicInteger() {
        MyDate myDate=new MyDate();
        for (int i = 0; i < 20; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    myDate.atomicInteger.getAndIncrement();
                }
            }, String.valueOf(i)).start();
        }
        while (Thread.activeCount() > 2){
            Thread.yield();
        }
        System.out.println(Thread.currentThread().getName()+" atomicInteger="+myDate.atomicInteger.get());
    }


```

#### 	1.3 	解决原子性

```
      一、加synchronize
      二、用Atomic
      AtomicInteger atomicInteger =new AtomicInteger()
      atomicInteger.addAndGet(int delta);想加几就加几
      atomicInteger.getAndIncrement();自加1
```




    1.3禁止指令重排（指令重排即是类编译为class，在底层用汇编语言进行指令重排，单线程不进行指令重排）
        一、处理器在指令重排时候必须考虑数据依赖性
        1）.对Volatile变量进行写操作时，会在写操作后加入一条store屏障指令，将工作内存中的共享变量的值刷新回主内存
        2）.对Valatile变量进行读操作时，会在读操作前加入一条load屏障指令，从主内存中读取共享变量
        3）.禁止在屏障的前后进行指令重排序优化
####  	1.4 	你在哪些地方用到过volatile？

```汉字
     1.多线程中的单例模式
     2.手写缓存
     3.cas底层源码
```

### 2.jMM(java内存模型)

JMM本身是一种抽象概念，并不真实存在，他描述了一组规则或规范，通过这组规范定义了程序中各个变量的访问方式

#### 2.1 	JMM关于同步的规定

```java
1.线程解锁前，必须把共享变量的值刷新会主内存
2.线程加锁前，必须读取主内存的最新值到自己的工作内存
3.加锁解锁是一把锁
```

#### 2.2 	三大特性

```java
	1可见性（三个线程同是修改主内存，一个修改完，其余接到通知）
	2原子性
	3有序性（禁止指令重排）
```

### 3.在哪里有使用到Volatile

#### 3.1 	什么是单例模式

```java

单例模式：是一种常用的软件设计模式。在它的核心结构中只包含一个被称为单例的特殊类。
单例模式的要点：一是某个类只能有一个实例；二是它必须自行创建这个实例；三是它必须自行 向整个系统提供这个实例。
单例模式的作用：一是，解决多线程并发访问的问题。二是节约系统内存，提交系统运行的效率，提高系统性能。
```



#### 		3.2 	单例模式，在多线程情况先会出现错误，会new多个对象

```汉字

经典单例模式
	private static SingletonDemo instance=null;

    private SingletonDemo(){
        System.out.println(Thread.currentThread().getName()+"我是构造方法");
    }

    public static SingletonDemo getInstance(){
        if(instance == null){
            instance = new SingletonDemo();
        }
        return instance;
    }
     public static void main(String[] args) {
        System.out.println(SingletonDemo.getInstance()==SingletonDemo.getInstance());
        System.out.println(SingletonDemo.getInstance()==SingletonDemo.getInstance());
    }
解决方法：
	1.在方法上面加上synchronize
	2.DCL(double check Lock 双端检索机制)单例模式，不一定保证线程安全，因为有指令重排序，只能加上volatile禁止指令重排
```

#### 		3.3 	创建单例模式过程

```java
instance=new SingletonDemo();可以分为三步

​memory=allocate();//1.分配对象内存空间
​insatnce(memory)；//2.初始化对象
 instance=memory;//3.设置instance指向刚刚分配的内存地址，此时instance！=null
```

#### 		3.4	 写一个synchronize的静态代码块（但是可能因为指令重排，导致没有取到这个初始化的值，解决办法在需要实例化的方法上面加上violatile,指令重排保证串行语句执行的一致性（单线程），但不管多线程）

```java
public class Singleton {
    private static violatile Singleton instance;

    private Singleton() {
        System.out.println("被初始化了");
    }

    public static  Singleton getInstance() {
        //双重判断机制
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }

    public static void main(String[] args) {

        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                System.out.println(Singleton.getInstance() + "      " + Thread.currentThread().getName());
            }, String.valueOf(i)).start();
        }
    }
}

```

### 4.cas(compare And swap)

#### 	4.1	  Cas(比较并交换)

```java
AtomicInteger atomicInteger=new AtomicInteger();
//多线程地下不要使用i++，使用atomicInteger.getAndIncrement
public void addMyAtomic(){
    atomicInteger.getAndIncrement();
}
public class CasDemo {
    public static void main(String[] args) {
        AtomicInteger atomicInteger=new AtomicInteger(5);
        System.out.println(atomicInteger.compareAndSet(5, 2019)+""+atomicInteger.get());
        System.out.println(atomicInteger.compareAndSet(5, 1024)+""+atomicInteger.get());
    }
}
```

#### 		4.2	 CAS之unsafe.class（Cas是一条并发原语，执行必须连续，不会被打断）

```java
AtomicInteger.getAndIncrement（）//底层调用unsafe类方法
public final int getAndIncrement(){
//解决i++在多线程情况下线程不安全问题，this代表当前对象，valueOfSet,内存偏移量（内存地址）
return unsafe.getAndInt(this,valueoffset,1);//每次自增为1,unsafe是jdk rt包下面的
}
public final int getAndAddInt(Object var1,long var2,int var4){
    int var5;
    do{
        var5=this.getIntVolatile(var1,var2);//1是对象，2是地址偏移量，也就是内存地址,获取地址值
    }while(!this.compareAndSwapInt(var1,var2,var5,var4+var5));
    	return var5;
}
```

#### 		4.3	CAS比较synchronize(为什么使用CAS不用synchronize)

```汉字
1.synchronize(全部加锁，做完一个才可以少一个)对比CAS并发下降
```

#### 		4.4 	CAS缺点

```汉字
1.如果之前有线程一直在修改值，就会一直do while,cpu开销较大
2.只能保证一个共享变量原子操作（对于多个共享变量操作，CAS无法保证变量的原子性，只能加锁）
3.引出ABA问题（首尾一样，中间被改变）
例：A线程操作内存10s，B线程2s，B把内存值改变了，又在一次把内存改回来，A线程觉得内存没有改变，继续操作，造成ABA问题。结果正确，过程不对。
```

4.5 ABA问题

```java
AtomicReference<>//原子类泛型
  public static void main(String[] args) {
        AtomicReference<User> atomicReference=new AtomicReference<>();
        User z3=new User("22",11);
        User z4=new User("11",1);

        atomicReference.set(z3);
        boolean b = atomicReference.compareAndSet(z3, z4);
        System.out.println(b+"  == "+atomicReference.toString());
    }
//解决ABA问题 加时间戳
public class ABADemo {
    static AtomicReference<Integer> atomicReference=new AtomicReference<>(100);
    static AtomicStampedReference<Integer> atomicStampedReference=new AtomicStampedReference<>(100,1);//属性值,时间戳
    public static void main(String[] args) {
        new Thread(() -> {
            atomicReference.compareAndSet(100,101);
            atomicReference.compareAndSet(101,100);
        }, "  t1").start();
        new Thread(() -> {
            try {
                System.out.println(Thread.activeCount());
                //暂停一秒
                TimeUnit.SECONDS.sleep(1);
                System.out.println(atomicReference.compareAndSet(100, 2019)+"==="+atomicReference.get());
            }catch (Exception e) {
            e.printStackTrace();
            }
        }, "  t2").start();
        try {
            TimeUnit.SECONDS.sleep(2);
        }catch (Exception e){
            e.printStackTrace();
        }
        System.out.println("ABA问题解决");
        new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();
            System.out.println(stamp);
            try{
                TimeUnit.SECONDS.sleep(1);//保证下面线程获取时间戳
            }catch (Exception e){
                e.printStackTrace();
            }
            System.out.println(atomicStampedReference.compareAndSet(100, 100, 1, 11));//本身值，期望值，原本邮戳，修改完的邮戳
            System.out.println(atomicStampedReference.getStamp());
        }, "  t3").start();
        new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();
            System.out.println(stamp);
            try{
                TimeUnit.SECONDS.sleep(3);//保证上面获取到版本号
            }catch (Exception e){
                e.printStackTrace();
            }
            System.out.println(atomicStampedReference.compareAndSet(100, 1000, 1, 2));
            System.out.println(atomicStampedReference.getStamp());
        }, "  t4").start();
    }
    }



```

### 5.集合类不安全

#### 5.1 	ArrayList线程不安全

ArrayList为什么线程不安全，因为arrayList.add（）没有加锁

```java
1.故障现象
   java.util.ConcurrentModificationException（并发修改异常）是ArrayList在高并发下面常见的异常
    public static void main(String[] args) {
       /* new ArrayList<Integer>();*///new ArrayList创建时候是一个空的list，初始值为10，加入泛型
        List<String> list= new ArrayList<>();
        for (int i = 0; i <30 ; i++) {
            new Thread(() -> {
                list.add(UUID.randomUUID().toString().substring(0,2));
                System.out.println(list);
            }, String.valueOf(i)).start();
        }
        }
2.导致原因
    1.并发争抢修改导致，参考花名册签名情况，一个人正在写，另外一个人过来抢夺，导致数据出错，也就是并发修改异常
    
3.解决方案
    1.用Vector Vector是jdk1.0出现，ArrayList是jdk1.2出现
    2.List<String> list= Collections.synchronizedList(new ArrayList<>());
	3.CopyOnWriteArrayList();
/*写时复制
	CopyOnWrite容器即写时复制的容器。往一个容器添加元素时，不直接往当前容器Object[]添加，而是先将当前容器Object[]进行Copy，复制出一个新的容器Object[] newElements,然后新的容器Object[] newElements里添加元素，添加完元素之后，再将原容器的引用指向新的容器 setArray（newElements）;这样做的好处是可以对CopyOnWrite容器进行并发的读，而不需要加锁。因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器
*/
		public boolean add(E e){
            final ReentrantLock lock=this.lock;
            lock.lock();
            try{
                Object[] elements=getArray();
                int len=elements.length;
                Object[] newElements =Array.copeOf(elements,len+1);
                newElements[len]=e;
                setArray(newElements);
                return true;
            }finally{
                lock.unlock();
            }
        }
4.优化建议
```

#### 5.2 	hashSet线程不安全

```java
与list一致，解决方案
   1.Collections.synchronizedSet(new HashSet<>())；
   2. Set<String> set= new CopyOnWriteArraySet<>();
public CopyOnWriteArraySet() {
        al = new CopyOnWriteArrayList<E>();
    }
CopyOnWriteArraySet<>()底层是 CopyOnWriteArrayList<E>()
    HashSet底层是HashMap
    public HashSet() {
        map = new HashMap<>();
    }
```

#### 5.3 	Map线程不安全

```java
解决方案
    1.Collections.synchronizedMap(new HashMap<>())
    2.Map<String,String> map=new ConcurrentHashMap<>();
```

```java
栈管运行，堆管存储
```

### 6.java锁

#### 6.1 	java锁之公平锁和非公平锁

```java
公平锁：是指多个线程按照申请锁的顺序来获取锁，类似排队打饭，队列
非公平锁：是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁，在高并发的情		 况下，有可能会造成优先级反转或饥饿现象
并发包中ReentrantLock的创建可以指定构造函数的boolean类型来得到公平锁或非公平锁，默认是非公平锁，非公平锁的优点是	吞吐量比公平锁大（比如女生请教阳哥，想要请假，一句话，并不需要很久时间）。对于Synchronized而言，也是一种非公平锁
```

#### 6.2 	java可重入锁（又名递归锁）

```java
public sync void method01(){
    method02();
}
public sync void method02(){
    
}
同步方法在访问同步方法
ReentrantLock/Synchronized就是典型的可重入锁
    
```

```java
package com.java1234.demo.yangge;


import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class Phone implements Runnable{
    public synchronized void sendSMS() throws Exception{
        System.out.println(Thread.currentThread().getId()+"\t invoke sendSMS()");
        sendEmail();
    }
    public synchronized void sendEmail() throws Exception{
        System.out.println(Thread.currentThread().getId()+"\t ####invoke sendEmail()");
    }
    Lock lock=new ReentrantLock();

    @Override
    public void run() {
        get();
    }
    public void get(){
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getId()+"/t invoke get()");
            set();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
    public void set(){
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getId()+"/t invoke set()");
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
}

public class ReentrantLockDemo{
    public static void main(String[] args) {
        Phone phone=new Phone();
        new Thread(() -> {
            try {
                phone.sendSMS();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "  t1").start();
        new Thread(() -> {
            try {
                phone.sendSMS();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "  t2").start();

        while (Thread.activeCount()>2){
            Thread.yield();
        }
        System.out.println();
        System.out.println();
        Thread t3=new Thread(phone);
        Thread t4=new Thread(phone);
        t3.start();
        t4.start();

    }
}

```

#### 6.3 自旋锁

自旋锁（spinlock）是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程上下切换的消耗，缺点是循环会消耗CPU（CAS就是)

```java
package com.java1234.demo.yangge;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicReference;

public class SpinLockDemo {
    AtomicReference<Thread> atomicReference=new AtomicReference();
    public void myLock(){
        Thread thread=Thread.currentThread();
        System.out.println(Thread.currentThread().getName()+"/t come in O(∩_∩)O哈哈~");
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        while (!atomicReference.compareAndSet(null,thread)){
        }
        System.out.println(Thread.currentThread().getName()+"lock");
    }

    public void myUnLock(){
        Thread thread=Thread.currentThread();
        boolean b = atomicReference.compareAndSet(thread, null);
        System.out.println(Thread.currentThread().getName()+"\t invoked myUnLock()");
    }
    public static void main(String[] args) {
        SpinLockDemo spinLockDemo=new SpinLockDemo();
        new Thread(()->{
            spinLockDemo.myLock();
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            spinLockDemo.myUnLock();
        },"a").start();

        try {
            TimeUnit.SECONDS.sleep(1);
        }catch (Exception e){
            e.printStackTrace();
        }

        new Thread(()->{
            spinLockDemo.myLock();
            spinLockDemo.myUnLock();
        },"bb").start();
    }
}

```

#### 6.4 读写锁(写入不可以被插入，读取可以被看见

写操作不被打断，读操作可以被打断 使用ReentrantReadWriteLock

```java
package com.java1234.demo.yangge;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

class MyCache{
    private volatile Map<String,Object> map=new HashMap<>();
    private Lock lock=null;
    private ReentrantReadWriteLock rwLock=new ReentrantReadWriteLock();
    public  void put(String key,Object value){
        rwLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName()+"\t 正在写入"+key);
            try {
                TimeUnit.MILLISECONDS.sleep(300);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            map.put(key, value);
            System.out.println(Thread.currentThread().getName()+"写入完成");
        }finally {
            rwLock.writeLock().unlock();
        }
    }
    public void get(String key){
        rwLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName()+"\t 正在读取");
            try {
                TimeUnit.MILLISECONDS.sleep(300);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            Object ca = map.get(key);
            System.out.println(Thread.currentThread().getName()+"读取完成"+ca);
        }finally {
            rwLock.readLock().unlock();
        }

    }
    public void clearMap(){
        map.clear();
    }
}

public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyCache myCache=new MyCache();
        for (int i = 0; i < 5; i++) {
            final int temp=i;
            new Thread(() -> {
                myCache.put(temp+"",temp+"");
            }, String.valueOf(i)).start();
        }
        for (int i = 0; i < 5; i++) {
            final int temp=i;
            new Thread(() -> {
                myCache.get(temp+"");
            }, String.valueOf(i)).start();
        }
    }
}

```

### 7.CountDownLatch/CyclicBarrier/Semaphore使用过吗？

#### 7.1.CountDownLatch

```定义
定义：让一些线程阻塞直到另外一些线程完成一系列操作时被唤醒）
count（计数）down（向下）latch（开始）（火箭发射倒计时,闭锁）
CountDownLatch主要有两个方法，当一个或多个线程调用await方法时，调用线程会被堵塞。
其他线程调用countDown方法会将计数器减1（调用countDown方法的线程不会阻塞）
当计数器为0时，因调用await方法阻塞的线程会被唤醒，继续执行
```



```java
package com.java1234.demo.yangge;

import java.util.concurrent.CountDownLatch;

public class CountDownLatchDemo {

    public static void main(String[] args) throws Exception{
        CountDownLatch countDownLatch=new CountDownLatch(6);
        for (int i = 1; i < 7; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName()+"国走了");
                countDownLatch.countDown();
            }, CountryEnum.foreach_CountryEnum(i).getRetMessage()).start();
        }
        countDownLatch.await();
        System.out.println(Thread.currentThread().getName()+"秦帝国");
    }


    private static void closeDoor() throws InterruptedException {
        CountDownLatch countDownLatch=new CountDownLatch(6);
        for (int i = 0; i < 6; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName()+"走了");
                countDownLatch.countDown();
            }, String.valueOf(i)).start();
        }
        countDownLatch.await();
        System.out.println(Thread.currentThread().getName()+"关门班长");
    }
}
//枚举使用
public enum CountryEnum
{
    ONE(1,"齐"),TWO(2,"楚"),THREE(3,"燕"),FOUR(4,"韩"),FIVE(5,"赵"),SIX(6,"魏");
    private String retMessage;
    private Integer retCode;
    public Integer getRetCode() {
        return retCode;
    }
    public String getRetMessage() {
        return retMessage;
    }

    CountryEnum(Integer retCode, String retMessage) {
        this.retCode = retCode;
        this.retMessage = retMessage;
    }
    public static CountryEnum foreach_CountryEnum(int index){
        CountryEnum[] myArray = CountryEnum.values();
        for(CountryEnum element : myArray){
            if (index==element.getRetCode()){
                return element;
            }
        }
        return null;
    }
}

```

#### 7.2CyclicBarrier

```java
不停做加法，才会可以执行与CountDownLatch相反
```

```java
package com.java1234.demo.yangge;

import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierDemo {

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier=new CyclicBarrier(7,()->{
            System.out.println("召唤神龙");
        });
        for (int i = 0; i <7 ; i++) {
            final int tempInt = i;
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName()+"收集到"+tempInt+"龙珠");
                try {
                    cyclicBarrier.await();
                }catch (Exception e){
                    e.printStackTrace();
                }
            }, String.valueOf(i)).start();
        }
    }
}

```

### 8.阻塞队列

#### 8.1什么是阻塞队列（都有什么）

```lilun
1.ArrayBlockingQueue:是一个基于数组结构的有界阻塞队列，此队列按FIFO（先进先出）原则对元素进行排序
2.LinkedBlockingQueue:是一个基于链表的阻塞队列，此队列按照FIFO排序元素，吞吐量通常高于ArraBlockingQueue
3.SynchronsQueue:一个不存储的阻塞队列。每个插入必须等到另外一个移除
所谓阻塞，在某些情况下会挂起线程（即阻塞），一旦条件满足，被挂起的线程又会被唤醒
BlockingQueue实现Collection与list同一级别
```

![阻塞队列方法](C:\Users\Administrator\Desktop\学习笔记\picture\1585662676(1).jpg)

```java
package com.java1234.demo.yangge;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.SynchronousQueue;
import java.util.concurrent.TimeUnit;

public class BlockingQueueDeme {

    public static void main(String[] args) throws Exception{
        //List list=new ArrayList();
        BlockingQueue<String> blockingQueue=new ArrayBlockingQueue<>(3);
        //直接抛异常
      /*  System.out.println(blockingQueue.add("a"));
        System.out.println(blockingQueue.add("b"));
        System.out.println(blockingQueue.add("c"));
        System.out.println(blockingQueue.add("c"));
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());*/
        //true or false
        /*System.out.println(blockingQueue.offer("a"));
        System.out.println(blockingQueue.offer("b"));
        System.out.println(blockingQueue.offer("c"));
        System.out.println(blockingQueue.offer("d"));
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());*/
        //不达到目的就一直堵着
        /*blockingQueue.put("a");
        blockingQueue.put("a");
        blockingQueue.put("a");
        blockingQueue.put("a");
        System.out.println("==========================");
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());*/
        //过时不候
        /*System.out.println(blockingQueue.offer("a", 2L, TimeUnit.SECONDS));
        System.out.println(blockingQueue.offer("a", 2L, TimeUnit.SECONDS));
        System.out.println(blockingQueue.offer("a", 2L, TimeUnit.SECONDS));
        System.out.println(blockingQueue.offer("a", 2L, TimeUnit.SECONDS));*/
        BlockingQueue<String> blockingQueue1=new SynchronousQueue<>();
        new Thread(() -> {
            try {
                System.out.println(Thread.currentThread().getName()+"a");
                blockingQueue1.put("a");
                System.out.println(Thread.currentThread().getName()+"a");
                blockingQueue1.put("a");
                System.out.println(Thread.currentThread().getName()+"a");
                blockingQueue1.put("a");
                System.out.println(Thread.currentThread().getName()+"a");
                blockingQueue1.put("a");
                System.out.println(Thread.currentThread().getName()+"a");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "  AAA").start();
        new Thread(() -> {
            try {
                Thread.sleep(5000);
                blockingQueue1.take();
                System.out.println("取出");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "  BBB").start();
    }
}
```

#### 8.2线程通信之生产者消费者传统版

![1.0与2.0生产者与消费者](C:\Users\Administrator\Desktop\学习笔记\picture\2CW3{`P3S5FVW5[XH]68AWY.png)

```笔记
wait()与notify()是java.lang.Object类下面的方法，不可以使用if判断，应使用循环
```



```java
package com.java1234.demo.yangge;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @ProjectName: springBootJava1234
 * @Package: com.java1234.demo.yangge
 * @ClassName: ProdConsumer_TraditionDemo
 * @Author: Administrator
 * @Date: 2020/4/1 0001 下午 9:11
 * 题目一个初始值为0的变量，两个线程交替操作，一个加1，一个减1，五轮
 * 1.线程         操作           资源类
 * 2.判断         干活            通知
 * 3.防止虚假线程唤醒
 */
public class ProdConsumer_TraditionDemo {

    public static void main(String[] args) {
        ShareDate shareDate = new ShareDate();
        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                try {
                    shareDate.increment();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }, " AA ").start();
        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                try {
                    shareDate.decrement();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }, " BB ").start();
    }


}

class ShareDate{
    private int num=0;
    private Lock lock= new ReentrantLock();
    private Condition condition=lock.newCondition();
    public void increment()throws Exception{
        lock.lock();
        try {
            //1.判断
            while (num!=0){
                //等待
                condition.await();
            }
            num++;
            System.out.println(Thread.currentThread().getName()+num);
            //3.通知
            condition.signalAll();
        } catch (Exception e){
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    public void decrement()throws Exception{
        lock.lock();
        try {
            //1.判断
            while (num==0){
                //等待
                condition.await();
            }
            num--;
            System.out.println(Thread.currentThread().getName()+num);
            //3.通知
            condition.signalAll();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
}
```

#### 8.3synchronize与Lock区别

```理解
1.原始构成
	synchorize是java关键字，属于JVM层面
	Lock是类，是java1.5以后出来的java.util.current.locks.Lock是api层面的
2.使用方法
	synchronize不需要手动释放锁，执行完自动释放
	Lock需要手动释放，若没有释放锁会引起死锁等现象
3.加锁是否公平
	synchronize不公平锁
	ReentrantLock两者都可以，默认是不公平，可以加一个boolean值。里面写true就是一个公平锁
4.是否可以中断
	synchronize不可终端，必须等代码执行完毕，或异常退出
	Lock可以中断1.设置超时方法tryLock（Long timeout，TimeUnit unit）
			  2.lockInterruptibly（）放代码块中，调用interrupt方法可中断
5.锁绑定多个条件condition
	synchronize没有
	ReentrantLock用来实现分组唤醒需要唤醒的线程们，可以精准唤醒，而不是像synchronize要么随机唤醒一个，要么全部唤醒
二、使用Lock有什么好处
```

#### 8.4练习

```题目
多线程之间按顺序调用，实现A->B->C三个线程启动，要求如下
AA打印五次，BB打印十次，CC打印十五次
AA打印五次，BB打印十次......
循环十次
```

```java
package com.java1234.demo.yangge;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @ProjectName: springBootJava1234
 * @Package: com.java1234.demo.yangge
 * @ClassName: SyncAndRenntrantLockDemo
 * @Author: Administrator
 * @Date: 2020/4/1 0001 下午 9:47
 */
public class SyncAndRenntrantLockDemo {

    public static void main(String[] args) {
        Lock lock=new ReentrantLock();
        ShareResource shareResource = new ShareResource();
         new Thread(() -> {
             for (int i = 0; i < 10; i++) {
                 shareResource.print5();
             }
                 }, "AA").start();
         new Thread(() -> {
              for (int i = 0; i < 10; i++) {
                  shareResource.print10();
              }
                  }, " BB ").start();
         new Thread(() -> {
               for (int i = 0; i < 10; i++) {
                   shareResource.print15();
               }
                   }, " CC ").start();
    }
}
class ShareResource{
    private int num=1;
    private Lock lock=new ReentrantLock();
    private Condition c1=lock.newCondition();
    private Condition c2=lock.newCondition();
    private Condition c3=lock.newCondition();
    public void print5(){
        lock.lock();
        try{
            //1.判断
            while (num!=1){
                c1.await();
            }
            for (int i = 0; i < 5; i++) {
                System.out.println("A线程打印"+i);
            }
            num = num + 1;
            //通知某个线程进行工作
            c2.signal();
        }catch (Exception e){
                e.printStackTrace();
        }finally{
            lock.unlock();
        }
    }
    public void print10(){
        lock.lock();
        try{
            //1.判断
            while (num!=2){
                c2.await();
            }
            for (int i = 0; i < 10; i++) {
                System.out.println("B线程打印"+i);
            }
            num = num + 1;
            //通知某个线程进行工作
            c3.signal();
        }catch (Exception e){
            e.printStackTrace();
        }finally{
            lock.unlock();
        }
    }
    public void print15(){
        lock.lock();
        try{
            //1.判断
            while (num!=3){
                c3.await();
            }
            for (int i = 0; i < 15; i++) {
                System.out.println("C线程打印"+i);
            }
            num = 1;
            //通知某个线程进行工作
            c1.signal();
        }catch (Exception e){
            e.printStackTrace();
        }finally{
            lock.unlock();
        }
    }
}
```

#### 8.5线程通信之生产者消费者阻塞队列版

```题目
1.生产消费，循环，并结束
```

```java
package com.java1234.demo.yangge;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

class MyResouce{
    private volatile boolean Falg = true;//默认开启进行生产消费
    private AtomicInteger atomicInteger = new AtomicInteger();
    BlockingQueue<String> blockingQueue = null;

    public MyResouce(BlockingQueue<String> blockingQueue) {
        this.blockingQueue = blockingQueue;
        System.out.println(blockingQueue.getClass().getName());
    }
    public void myProd()throws Exception{
        String data = null;
        boolean retValue;
        while (Falg){
            data = atomicInteger.getAndIncrement()+"";
            retValue = blockingQueue.offer(data, 2L, TimeUnit.SECONDS);
            if (retValue){
                System.out.println(Thread.currentThread().getName() + "\t 插入队列" + data + "成功");
            }else {
                System.out.println(Thread.currentThread().getName() + "\t 插入队列" + data + "失败");
            }
            TimeUnit.SECONDS.sleep(1);
        }
        System.out.println(Thread.currentThread().getName() + "/t 大老板叫停");
    }
    public void myCunsumer()throws Exception {
        String result = null;
        while (Falg){
            result = blockingQueue.poll(2L,TimeUnit.SECONDS);
            if (null == result || result.equalsIgnoreCase("")){
                Falg = false;
                System.out.println(Thread.currentThread().getName() + "\t 超过2秒钟 没有取到值，消费退出");
                System.out.println();
                System.out.println();
                return;
            }
            System.out.println(Thread.currentThread().getName() + "\t 消费成功" + result);
        }
    }
    public void stop()throws Exception{
        this.Falg = false;
    }
}
public class ProdConsumer_BlockQueueDemo {
    public static void main(String[] args) throws Exception {
        MyResouce myResouce = new MyResouce(new ArrayBlockingQueue<>(10));
         new Thread(() -> {
             System.out.println("进入生产线程"+Thread.currentThread().getName());
             try {
                 myResouce.myProd();
             } catch (Exception e) {
                 e.printStackTrace();
             }
         }, " Prod ").start();
          new Thread(() -> {
              System.out.println("进入消费线程"+Thread.currentThread().getName());
              try {
                  myResouce.myCunsumer();
              } catch (Exception e) {
                  e.printStackTrace();
              }
                  }, " BB ").start();
            try {
                    TimeUnit.SECONDS.sleep(5);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            System.out.println("5秒时间到，活动结束");
           myResouce.stop();
    }
}
```

### 9.创建实现callable线程

#### 9.1 Callable与Runnable区别

```注释
class MyThread implements Runnable{
    @Override
    public void run() {

    }
}
class MyThread2 implements Callable<Integer>{
    @Override
    public Integer call() throws Exception {
        return null;
    }
}
区别：1.Runnable没有返回值，Callable有返回值
	 2.Runnable不会抛异常，Callable会抛出异常	
	 3.接口实现方法不一样，一个run（）一个call（）
```

```java
package com.java1234.demo.yangge;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;
import java.util.concurrent.TimeUnit;

class MyThread implements Callable<Integer>{
    @Override
    public Integer call() throws Exception {
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("*************come in callable");
        return 1024;
    }
}
public class CallableDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //用了构造方法
        FutureTask<Integer> futureTask = new FutureTask<>(new MyThread());
        Thread t1 = new Thread(futureTask,"AAA");
        t1.start();
        int result = futureTask.get();
        System.out.println("**********");
        //如无必要，放在最后
        System.out.println(result);
    }
}

```

#### 9.2线程池的使用以及优势

```注释
1,。为什么要用线程池
```

![](C:\Users\Administrator\Desktop\学习笔记\picture\线程池优势.png)

```java
2.使用线程池创建线程
```

```java
	    ExecutorService threadPool = Executors.newFixedThreadPool(5);//一池五个处理五线程
        ExecutorService threadPool = Executors.newSingleThreadExecutor();//一池一个处理一个线程
        ExecutorService threadPool = Executors.newCachedThreadPool();//一池处理N线程,随机扩容


package com.java1234.demo.yangge;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class MyThreadPoolDemo {
    public static void main(String[] args) {
        //显示电脑cpu
        //System.out.println(Runtime.getRuntime().availableProcessors());
        //ExecutorService threadPool = Executors.newFixedThreadPool(5);//一池五个处理五线程
        //ExecutorService threadPool = Executors.newSingleThreadExecutor();
        ExecutorService threadPool = Executors.newCachedThreadPool();

        //模拟十个用户办理，每个用户就是外部的请求线程
        try {
            for (int i = 0; i < 20; i++) {
                threadPool.execute(() -> { //执行线程池
                    System.out.println(Thread.currentThread().getName()+ "\t 办理业务");
                });
                try {
                    TimeUnit.MILLISECONDS.sleep(5);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            threadPool.shutdown(); //释放线程池
        }
    }
}
```

```java
3.线程池的底层
    底层使用LinkBlockQueue阻塞队列
```

![image-20200505204235887](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200505204235887.png)

```java
线程池五大参数
底层实际七个参数
    1.corePoolSize: 今日银行上班窗口
    2.maximumPoolSize: 一共可以容下多少人，池子最大的容量
    3.当人员全部走了，加班的窗口就可以撤退
    4.时间单位，存活时间单位
    5.workQueue:在候客区等待的
    6.默认的制服，工牌
    7.先去今日上班窗口，再去候客区，再去最多容下的加班区，都满了，则劝退
```

![image-20200505205355586](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200505205355586.png)

![image-20200505214544357](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200505214544357.png)

![image-20200505205004171](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200505205004171.png)

#### 9.3线程池的拒绝策略

```java
线程池的拒绝策略，是什么？
    1.等待队列满了，再也塞不下任务了同时，线程池中的max线程也达到了，无法为新任务提供服务，这时候我们需要拒绝策略机制合理的处理这个问题（RejectedExecuteHandler）
    一共四种
    一、AbortPlicy（默认）： 直接抛出RejectedExecutionException异常阻止系统正常运行
    二、CallerRunsPolicy：“调用者运行”一种调节机制，该策略既不会抛弃任务，也不会抛出异常，而是将某些任务回退到调用者，从而降低任务流量
    三、DisCardOldestPolicy: 抛弃队列中等待最久的任务，然后把当前任务加入队列中尝试再次提交当前任务
    四、DisCardPolicy:直接丢弃任务，不予任何处理也不抛出异常。如果允许任务丢失，这是最好的一种方案
```

#### 9.4手写线程池

```java
 public static void main(String[] args) {
        ExecutorService threadPool = new ThreadPoolExecutor(
                2,//核心线程
                5,//最大线程
                1L,//等待时间
                TimeUnit.SECONDS,//等待单位
                new LinkedBlockingDeque<Runnable>(3),//阻塞队列，或者银行候客区
                Executors.defaultThreadFactory(),//默认的工厂
            	new ThreadPoolExecutor.DiscardOldestPolicy());//拒绝策略
        try {
            for (int i = 1; i <= 100; i++) {
                final int temp = i;
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName()+"\t 办理业务" + temp);
                    try {
                        TimeUnit.SECONDS.sleep(3);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                });
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            threadPool.shutdown();
        }
```

#### 9.5合理配置线程池，你是如何考虑的

```java
1.首先运行Runtime.getRuntime().avaliableProcessors();//查看系统cpu核心数
CPU密集型
    一般公式：CPU核数+1个线程的线程池
IO密集型
    由于IO密集型任务线程并不是一直执行任务，则应配置尽可能多的线程，如CPU核数*2
    大部分线程都在阻塞，故需要配置多的线程数
    CPU核数/1-阻塞系数  阻塞系数在0.8~0.9之间
    比如8核CPU： 8/（1-0.9）=80个线程数
```

