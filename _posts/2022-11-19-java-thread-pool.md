---
layout:       post
title:        "线程池简单实现"
author:       "hang.li"
header-style: text
catalog:      java
tags:
    - thread
    - java
---

# UML
![img.png](/img/in-post/java/threadpooluml.png)
# 核心代码
## 任务队列满时执行的拒绝策略
- 枚举单例 + 策略(DenyPolicyStrategy.java)
```java
package com.example.executor.service.impl;

import com.example.executor.exception.RunnableDenyException;

/**
 * 任务数量达到上限时, 线程池处理的三种策略
 * 1. 直接将任务丢弃(一个空的执行)
 * 2. 将任务丢弃后, 抛出异常
 * 3. 任务在调用者线程中执行(直接调用Runnable 中的run方法)
 */
public enum DenyPolicyStrategy {
    /**
     * 策略1,丢弃
     */
    DISCARD_DENY_POLICY_STRATEGY((runnable, threadPool) -> {}),
    /**
     * 策略2,抛出异常
     */
    ABORT_DENY_POLICY_STRATEGY((runnable, threadPool) -> {
        throw new RunnableDenyException("The runnable " + runnable + " will be abort.");
    }),
    /**
     * 策略3,抛出异常
     */
    RUNNER_DENY_POLICY_STRATEGY((runnable, threadPool) -> {
        if (!threadPool.isShutdown()) {
            // 如果想要使用的线程池是有效的, 调用run方法
            runnable.run();
        }
    });
    /**
     * 策略方法
     * 函数式接口
     */
    public RejectFunction rejectFunction;

    DenyPolicyStrategy(RejectFunction rejectFunction) {
        this.rejectFunction = rejectFunction;
    }
    public void reject(Runnable runnable, ThreadPool threadPool) throws RunnableDenyException {
        rejectFunction.reject(runnable, threadPool);
    }

    interface RejectFunction {
        void reject(Runnable runnable, ThreadPool threadPool) throws RunnableDenyException;
    }
}

```
## 核心线程池
- 使用Builer模式创建线程池
- 使用了一个核心线程维护线程池中的线程数量
- 关闭线程池时, 关闭其中维护的线程,并且将核心线程打断
- 创建线程时,将自己自定义的线程和jvm中的核心线程绑定,保存到栈中
```java
package com.example.executor.service.impl;

import com.example.executor.exception.InitThreadPoolException;
import com.example.executor.exception.RunnableDenyException;
import com.example.executor.service.DenyPolicyStrategy;
import com.example.executor.service.RunnableQueue;
import com.example.executor.service.ThreadFactory;
import com.example.executor.service.ThreadPool;

import java.util.LinkedList;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 实现 ThreadPool
 */
public class BasicThreadPool implements ThreadPool {

    /**
     * 初始化线程数量
     */
    private final int initSize;

    /**
     * 线程池中最大的线程数量
     */
    private final int maxSize;

    /**
     * 线程池中核心线程数量
     */
    private final int coreSize;

    /**
     * 活跃线程数量
     */
    private int activeCount;

    /**
     * 线程工厂
     */
    private final ThreadFactory threadFactory;

    /**
     * 任务队列
     */
    private final RunnableQueue runnableQueue;

    /**
     * 线程池的状态
     */
    private volatile boolean isShutdown;

    /**
     * 线程池中已经创建的线程列表 <br/>
     * 这里使用LinkedList模拟了一个栈模型
     * 栈顶的对象相对于栈底的对象更年轻, 更容易被回收
     */
    private final LinkedList<ThreadTask> threadStack;

    /**
     * 线程的存活时间
     * 每经过此间隔, 线程池都会更新一次内部线程数量
     */
    private final long keepAliveTime;

    /**
     * 线程的存活时间计量单位
     */
    private final TimeUnit timeUnit;

    /**
     * 线程用于动态创建线程池中线程
     */
    private final Thread threadPoolCoreThread;

    /**
     * builder的控制器
     */
    public BasicThreadPool(Builder builder) {
        this.initSize = builder.initSize;
        this.maxSize = builder.maxSize;
        this.coreSize = builder.coreSize;
        this.threadFactory = builder.threadFactory;
        this.runnableQueue = new LinkedRunnableQueue(builder.queueSize, builder.denyPolicyStrategy, this);
        this.keepAliveTime = builder.keepAliveTime;
        this.timeUnit = builder.timeUnit;

        this.threadStack = new LinkedList<>();
        this.threadPoolCoreThread = new Thread(this::coreThreadRun);
        this.isShutdown = false;
        this.init();
    }

    private void init() {
        for (int i = 0; i < initSize; i++) {
            newThread();
        }
        this.threadPoolCoreThread.start();
    }

    /**
     * 从jvm中创建线程并绑定到线程池中
     * 执行队列中的任务
     */
    private void newThread() {
        InternalTask internalTask = new InternalTask(runnableQueue);
        Thread thread = this.threadFactory.createThread(internalTask);
        ThreadTask threadTask = new ThreadTask(thread, internalTask);
        threadStack.offer(threadTask);
        this.activeCount++;
        thread.start();
    }

    /**
     * 从线程池中去掉一个线程关联
     */
    private void removeThread() {
        ThreadTask threadTask = threadStack.removeLast();
        threadTask.internalTask.stop();
        this.activeCount--;
    }

    /**
     * 计算线程池中, 线程数量
     */
    public void coreThreadRun() {
        while (!isShutdown && !this.threadPoolCoreThread.isInterrupted()) {
            try {
                timeUnit.sleep(keepAliveTime);
            } catch (InterruptedException e) {
                isShutdown = true;
                break;
            }

            synchronized (this) {
                if (isShutdown) {
                    break;
                }

                // 当前队列中有任务还没有被处理, 没有达到核心线程数量, 增长到核心线程数
                if (runnableQueue.size() > 0 && activeCount < coreSize) {
                    for (int i = initSize; i < coreSize; i++) {
                        newThread();
                    }
                    continue;
                }

                // 当前队列中有任务还没有被处理, 没有达到核心线程数量, 增长到核心线程数
                if (runnableQueue.size() > 0 && activeCount < maxSize) {
                    for (int i = coreSize; i < maxSize; i++) {
                        newThread();
                    }
                }

                // 已经没有要处理的任务了, 保留coreSize数量
                if (runnableQueue.size() == 0 && activeCount > coreSize) {
                    for (int i = coreSize; i < activeCount; i++) {
                        removeThread();
                    }
                }
            }
        }
    }

    /**
     * 提交任务到线程池
     *
     * @param runnable 要提交的任务
     */
    @Override
    public void execute(Runnable runnable) throws RunnableDenyException {
        checkIsShutdown();
        this.runnableQueue.offer(runnable);
    }

    /**
     * 关闭线程池
     */
    @Override
    public synchronized RunnableQueue shutdown() {
        if (isShutdown) {
            return null;
        }
        isShutdown = true;
        threadStack.forEach(threadTask -> {
            threadTask.internalTask.stop();
            threadTask.thread.interrupt();
        });
        this.threadPoolCoreThread.interrupt();
        return this.runnableQueue;
    }

    /**
     * 获取线程池的初始化大小
     */
    @Override
    public int getInitSize() {
        checkIsShutdown();
        return this.initSize;
    }

    /**
     * 获取线程池中最大的线程数
     */
    @Override
    public int getMaxSize() {
        checkIsShutdown();
        return this.maxSize;
    }

    /**
     * 获取线程池中核心线程的数量
     * 核心线程: 当线程池中没有任务在执行, 要维护的空闲线程的数量
     */
    @Override
    public int getCoreSize() {
        checkIsShutdown();
        return this.coreSize;
    }

    /**
     * 获取线程池中的用于缓存任务队列的大小
     */
    @Override
    public int getQueueSize() {
        checkIsShutdown();
        return this.runnableQueue.size();
    }

    /**
     * 获取线程池中活跃线程的数量
     */
    @Override
    public synchronized int getActiveCount() {
        return this.activeCount;
    }

    /**
     * 查看线程池是否已经被关闭
     */
    @Override
    public boolean isShutdown() {
        return this.isShutdown;
    }

    /**
     * 判断连接池是否已经被销毁了
     */
    private void checkIsShutdown() {
        if (isShutdown) {
            throw new IllegalStateException("连接池已经被销毁");
        }
    }

    /**
     * 真实从jvm中创建的线程(Thread)和线程池中使用的线程池(InternalTask)关联关系
     */
    private static class ThreadTask {
        Thread thread;
        InternalTask internalTask;

        public ThreadTask(Thread thread, InternalTask internalTask) {
            this.thread = thread;
            this.internalTask = internalTask;
        }
    }

    /**
     * 默认的线程工厂
     * 指定默认的线程组, 线程名称
     */
    private static class DefaultThreadFactory implements ThreadFactory {

        private static final AtomicInteger GROUP_COUNTER = new AtomicInteger(1);

        private static final ThreadGroup threadGroup = new ThreadGroup("MyThreadPool-" + GROUP_COUNTER.getAndDecrement());

        private static final AtomicInteger COUNTER = new AtomicInteger(0);

        /**
         * 创建线程
         */
        @Override
        public Thread createThread(Runnable runnable) {
            return new Thread(threadGroup, runnable, "thread-pool-" + COUNTER.getAndDecrement());
        }
    }

    public static class Builder {

        private int initSize = 2;

        private int maxSize = 6;

        private int coreSize = 4;

        private int queueSize = 1000;

        private DenyPolicyStrategy denyPolicyStrategy = DenyPolicyStrategy.DISCARD_DENY_POLICY_STRATEGY;

        private ThreadFactory threadFactory = new DefaultThreadFactory();

        private long keepAliveTime = 12;

        private TimeUnit timeUnit = TimeUnit.SECONDS;

        public BasicThreadPool build() throws InitThreadPoolException {
            if (this.maxSize < this.initSize) {
                throw new InitThreadPoolException("初始化线程数不能大于最大线程数");
            }
            if (this.coreSize < this.initSize) {
                throw new InitThreadPoolException("初始化线程数不能大于核心线程数");
            }
            if (this.maxSize < this.coreSize) {
                throw new InitThreadPoolException("核心线程数不能大于最大线程数");
            }
            return new BasicThreadPool(this);
        }

        public Builder setInitSize(int initSize) {
            this.initSize = initSize;
            return this;
        }

        public Builder setMaxSize(int maxSize) {
            this.maxSize = maxSize;
            return this;
        }

        public Builder setCoreSize(int coreSize) {
            this.coreSize = coreSize;
            return this;
        }

        public Builder setQueueSize(int queueSize) {
            this.queueSize = queueSize;
            return this;
        }

        public Builder setDenyPolicyStrategy(DenyPolicyStrategy denyPolicyStrategy) {
            this.denyPolicyStrategy = denyPolicyStrategy;
            return this;
        }

        public Builder setThreadFactory(ThreadFactory threadFactory) {
            this.threadFactory = threadFactory;
            return this;
        }

        public Builder setKeepAliveTime(long keepAliveTime) {
            this.keepAliveTime = keepAliveTime;
            return this;
        }

        public Builder setTimeUnit(TimeUnit timeUnit) {
            this.timeUnit = timeUnit;
            return this;
        }
    }
}
```
## 线程池中使用的任务模型
- 任务模型工作就是不断拿线程池中的队列中的任务执行, 如果队列为null时挂起
```java
package com.example.executor.service.impl;

import com.example.executor.service.RunnableQueue;

/**
 * 线程池中使用的线程模型 <br/>
 * 对Runnable的实现 <br/>
 * 一个InternalTask负责执行一个runnableQueue中所有的任务 <br/>
 * 当runnableQueue中的任务列表为空时,当前线程会被挂起 <br/>
 */
public class InternalTask implements Runnable {

    private final RunnableQueue runnableQueue;

    private volatile boolean running = true;

    public InternalTask(RunnableQueue runnableQueue) {
        this.runnableQueue = runnableQueue;
    }

    @Override
    public void run() {
        // 当前线程没有被中断, 并且执行状态是true
        while (running && !Thread.currentThread().isInterrupted()) {
            // 从队列中拿到任务, 使用当前线程执行
            Runnable task;
            try {
                task = runnableQueue.take();
            } catch (InterruptedException e) {
                this.running = false;
                break;
            }
            task.run();
        }
    }

    /**
     * 停止当前线程
     * 不再从任务列表中获取任务, 结束run方法
     */
    public void stop() {
        this.running = false;
    }
}
```
## 任务队列
- 任务按顺序执行,这里维护了一个队列(先进先出)
```java
package com.example.executor.service.impl;

import com.example.executor.exception.RunnableDenyException;
import com.example.executor.service.DenyPolicyStrategy;
import com.example.executor.service.RunnableQueue;
import com.example.executor.service.ThreadPool;

import java.util.LinkedList;

/**
 * 队列基于LinkedList的一个实现 <br/>
 * 加上了队列中任务上限的限定 <br/>
 * 任务入队时加上了到达上限的判断, 并能使用初始化的拒绝策略进行拒绝 <br/>
 * 提取任务, 任务列表时null时, 进入线程等待 <br/>
 */
public class LinkedRunnableQueue implements RunnableQueue {

    /**
     * 队列中任务的最大值
     */
    private final int limit;

    /**
     * 如果任务队列满了, 要执行的拒绝策略
     */
    private final DenyPolicyStrategy denyPolicyStrategy;

    /**
     * 任务队列存放的容器
     */
    private final LinkedList<Runnable> runnableLinkedList = new LinkedList();

    private final ThreadPool threadPool;

    public LinkedRunnableQueue(int limit, DenyPolicyStrategy denyPolicyStrategy, ThreadPool threadPool) {
        this.limit = limit;
        this.denyPolicyStrategy = denyPolicyStrategy;
        this.threadPool = threadPool;
    }

    /**
     * 当有新任务时,首先进入offer队列中
     */
    @Override
    public void offer(Runnable runnable) throws RunnableDenyException {
        synchronized (runnableLinkedList) {
            if (runnableLinkedList.size() >= limit) {
                // 达到任务容量的上限
                denyPolicyStrategy.reject(runnable, threadPool);
            } else {
                runnableLinkedList.add(runnable);
                // 唤醒被阻塞的队列
                runnableLinkedList.notifyAll();
            }
        }
    }

    /**
     * 工作线程通过take方式获取Runnable
     */
    @Override
    public Runnable take() throws InterruptedException {
        synchronized (runnableLinkedList) {
            while (runnableLinkedList.isEmpty()) {
                try {
                    runnableLinkedList.wait();
                } catch (InterruptedException e) {
                    throw e;
                }
            }
        }
        return runnableLinkedList.removeFirst();
    }

    /**
     * 获取任务列表中的任务数量
     */
    @Override
    public int size() {
        synchronized (runnableLinkedList) {
            return runnableLinkedList.size();
        }
    }
}
```
# 其他一些接口的定义
1. 一些抛出的异常
```java
package com.example.executor.exception;

/**
 * 线程池初始化失败抛出的异常
 */
public class InitThreadPoolException extends Exception {
    public InitThreadPoolException(String message) {
        super(message);
    }
}
```
```java
package com.example.executor.exception;

import com.example.executor.service.DenyPolicyStrategy;

/**
 * 任务队列满时,抛出的异常
 *
 * @see DenyPolicyStrategy
 */
public class RunnableDenyException extends Exception {
    public RunnableDenyException(String message) {
        super(message);
    }
}
```
2. ThreadPool线程池
```java
package com.example.executor.service;

import com.example.executor.exception.RunnableDenyException;

/**
 * 线程池
 */
public interface ThreadPool {
    /**
     * 提交任务到线程池
     */
    void execute(Runnable runnable) throws RunnableDenyException;

    /**
     * 关闭线程池
     */
    RunnableQueue shutdown();

    /**
     * 获取线程池的初始化大小
     */
    int getInitSize();

    /**
     * 获取线程池中最大的线程数
     */
    int getMaxSize();

    /**
     * 获取线程池中核心线程的数量 <br/>
     * 核心线程: 当线程池中没有任务在执行, 要维护的空闲线程的数量
     */
    int getCoreSize();

    /**
     * 获取线程池中的用于缓存任务队列的大小
     */
    int getQueueSize();

    /**
     * 获取线程池中活跃线程的数量
     */
    int getActiveCount();

    /**
     * 查看线程池是否已经被关闭
     */
    boolean isShutdown();
}
```
3. 线程工厂定义(创建jvm线程, 构造组,线程名)
```java
package com.example.executor.service;

/**
 * 函数式接口 <br/>
 * 线程工厂,用于创建线程 <br/>
 * 拥有创建线程的方法 <br/>
 */
@FunctionalInterface
public interface ThreadFactory {
    /**
     * 创建线程
     */
    Thread createThread(Runnable runnable);
}

```
4. 任务队列接口
```java
package com.example.executor.service;

import com.example.executor.exception.RunnableDenyException;

/**
 * 任务队列<br/>
 * 拥有任务入队, 出队, 获取任务列表方法
 */
public interface RunnableQueue {

    /**
     * 当有新任务时,首先进入offer队列中
     */
    void offer(Runnable runnable) throws RunnableDenyException;

    /**
     * 工作线程通过take方式获取Runnable
     */
    Runnable take() throws InterruptedException;

    /**
     * 获取任务列表中的任务数量
     */
    int size();
}
```
5. 测试Demo
```java
package com.example.executor;

import com.example.executor.exception.InitThreadPoolException;
import com.example.executor.service.ThreadPool;
import com.example.executor.service.impl.BasicThreadPool;

import java.util.concurrent.TimeUnit;

public class Demo {
    public static void main(String[] args) {
        ThreadPool threadPool;
        try {
            threadPool = new BasicThreadPool.Builder()
                    .setInitSize(2)
                    .setCoreSize(4)
                    .setMaxSize(6)
                    .build();
        } catch (InitThreadPoolException e) {
            e.printStackTrace();
            return;
        }
        try {
            for (int i = 0; i < 20; i++) {
                threadPool.execute(() -> {
                    try {
                        TimeUnit.SECONDS.sleep(10);
                        System.out.println(Thread.currentThread().getName() + " is running and done.");
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                });
            }

            for (; ; ) {
                System.out.println("activeCount: " + threadPool.getActiveCount());
                System.out.println("queueSize: " + threadPool.getQueueSize());
                System.out.println("coreSize: " + threadPool.getCoreSize());
                System.out.println("maxSize: " + threadPool.getMaxSize());
                System.out.println("========================");
                TimeUnit.SECONDS.sleep(5);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
# 全部代码
[全部代码](https://download.csdn.net/download/liha12138/12551449)