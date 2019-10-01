##ThreadLocal用法详解和原理
###一、用法

ThreadLocal用于保存某个线程共享变量：对于同一个static ThreadLocal，不同线程只能从中get，set，remove自己的变量，而不会影响其他线程的变量。

1、ThreadLocal.get: 获取ThreadLocal中当前线程共享变量的值。

2、ThreadLocal.set: 设置ThreadLocal中当前线程共享变量的值。

3、ThreadLocal.remove: 移除ThreadLocal中当前线程共享变量的值。

4、ThreadLocal.initialValue: ThreadLocal没有被当前线程赋值时或当前线程刚调用remove方法后调用get方法，返回此方法值。
	
	package com.coshaho.reflect;
	 
	/**
	 * ThreadLocal用法
	 * @author coshaho
	 *
	 */
	public class MyThreadLocal
	{
	    private static final ThreadLocal<Object> threadLocal = new ThreadLocal<Object>(){
	        /**
	         * ThreadLocal没有被当前线程赋值时或当前线程刚调用remove方法后调用get方法，返回此方法值
	         */
	        @Override
	        protected Object initialValue()
	        {
	            System.out.println("调用get方法时，当前线程共享变量没有设置，调用initialValue获取默认值！");
	            return null;
	        }
	    };
     
    public static void main(String[] args)
    {
        new Thread(new MyIntegerTask("IntegerTask1")).start();
        new Thread(new MyStringTask("StringTask1")).start();
        new Thread(new MyIntegerTask("IntegerTask2")).start();
        new Thread(new MyStringTask("StringTask2")).start();
    }
     
    public static class MyIntegerTask implements Runnable
    {
        private String name;
         
        MyIntegerTask(String name)
        {
            this.name = name;
        }
 
        @Override
        public void run()
        {
            for(int i = 0; i < 5; i++)
            {
                // ThreadLocal.get方法获取线程变量
                if(null == MyThreadLocal.threadLocal.get())
                {
                    // ThreadLocal.et方法设置线程变量
                    MyThreadLocal.threadLocal.set(0);
                    System.out.println("线程" + name + ": 0");
                }
                else
                {
                    int num = (Integer)MyThreadLocal.threadLocal.get();
                    MyThreadLocal.threadLocal.set(num + 1);
                    System.out.println("线程" + name + ": " + MyThreadLocal.threadLocal.get());
                    if(i == 3)
                    {
                        MyThreadLocal.threadLocal.remove();
                    }
                }
                try
                {
                    Thread.sleep(1000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
            }  
        }
         
    }
     
    public static class MyStringTask implements Runnable
    {
        private String name;
         
        MyStringTask(String name)
        {
            this.name = name;
        }
 
        @Override
        public void run()
        {
            for(int i = 0; i < 5; i++)
            {
                if(null == MyThreadLocal.threadLocal.get())
                {
                    MyThreadLocal.threadLocal.set("a");
                    System.out.println("线程" + name + ": a");
                }
                else
                {
                    String str = (String)MyThreadLocal.threadLocal.get();
                    MyThreadLocal.threadLocal.set(str + "a");
                    System.out.println("线程" + name + ": " + MyThreadLocal.threadLocal.get());
                }
                try
                {
                    Thread.sleep(800);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
            }
        }
         
    }
<strong>}
</strong>

运行结果如下：   
调用get方法时，当前线程共享变量没有设置，调用initialValue获取默认值！   
线程IntegerTask1: 0   
调用get方法时，当前线程共享变量没有设置，调用initialValue获取默认值！  
线程IntegerTask2: 0   
调用get方法时，当前线程共享变量没有设置，调用initialValue获取默认值！  
调用get方法时，当前线程共享变量没有设置，调用initialValue获取默认值！  
线程StringTask1: a  
线程StringTask2: a  
线程StringTask1: aa  
线程StringTask2: aa  
线程IntegerTask1: 1  
线程IntegerTask2: 1  
线程StringTask1: aaa  
线程StringTask2: aaa  
线程IntegerTask2: 2  
线程IntegerTask1: 2  
线程StringTask2: aaaa  
线程StringTask1: aaaa  
线程IntegerTask2: 3  
线程IntegerTask1: 3  
线程StringTask1: aaaaa  
线程StringTask2: aaaaa  
调用get方法时，当前线程共享变量没有设置，调用initialValue获取默认值！  
线程IntegerTask2: 0  
调用get方法时，当前线程共享变量没有设置，调用initialValue获取默认值！  
线程IntegerTask1: 0  

###二、原理

线程共享变量缓存如下：

Thread.ThreadLocalMap<ThreadLocal, Object>;

1、Thread: 当前线程，可以通过Thread.currentThread()获取。

2、ThreadLocal：我们的static ThreadLocal变量。

3、Object: 当前线程共享变量。

我们调用ThreadLocal.get方法时，实际上是从当前线程中获取ThreadLocalMap<ThreadLocal, Object>，然后根据当前ThreadLocal获取当前线程共享变量Object。

ThreadLocal.set，ThreadLocal.remove实际上是同样的道理。

这种存储结构的好处：

1、线程死去的时候，线程共享变量ThreadLocalMap则销毁。

2、ThreadLocalMap<ThreadLocal,Object>键值对数量为ThreadLocal的数量，一般来说ThreadLocal数量很少，相比在ThreadLocal中用Map<Thread, Object>键值对存储线程共享变量（Thread数量一般来说比ThreadLocal数量多），性能提高很多。


关于ThreadLocalMap<ThreadLocal, Object>弱引用问题：

当线程没有结束，但是ThreadLocal已经被回收，则可能导致线程中存在ThreadLocalMap<null, Object>的键值对，造成内存泄露。（ThreadLocal被回收，ThreadLocal关联的线程共享变量还存在）。

虽然ThreadLocal的get，set方法可以清除ThreadLocalMap中key为null的value，但是get，set方法在内存泄露后并不会必然调用，所以为了防止此类情况的出现，我们有两种手段。

1、使用完线程共享变量后，显示调用ThreadLocalMap.remove方法清除线程共享变量；

2、JDK建议ThreadLocal定义为private static，这样ThreadLocal的弱引用问题则不存在了。

##ThreadLocal内存泄漏原因以及避免方案
ThreadLocal的原理是操作Thread内部的一个ThreadLocalMap，这个Map的Entry继承了WeakReference,设值完成后map中是(WeakReference,value)这样的数据结构。Java中的弱引用在内存不足的时候会被回收掉，回收之后变成(null,value)的形式，key被收回掉了。 

如果这个线程执行完之后销毁，value也会被回收，这样也不会出现内存泄露。但如果是在线程池中，线程 执行完后不被回收，而是返回线程池中。此时Thread有个强引用 指向 ThreadLocalMap，ThreadLocalMap有强引用 指向 Entry，导致Entry中key为null的value无法被回收，一直存在内存中。在执行了ThreadLocal.set()方法之后一定要记得使用ThreadLocal.remove(),将不要的数据移除掉，避免内存泄漏。

