# 1.线程池的创建

线程池的创建⽅法总共有 7 种，但总体来说可分为 2 类：

    1. 通过 ThreadPoolExecutor 创建的线程池；
    2. 通过 Executors 创建的线程池。

线程池的创建⽅式总共包含以下 7 种（其中 6 种是通过 Executors 创建的，1 种是通过ThreadPoolExecutor 创建的）：

```
1. Executors.newFixedThreadPool：创建⼀个固定⼤⼩的线程池，可控制并发的线程数，超出的线程会在队列中等待；
2. Executors.newCachedThreadPool：创建⼀个可缓存的线程池，若线程数超过处理所需，缓存⼀段时间后会回收，若线程数不够，则新建线程；
3. Executors.newSingleThreadExecutor：创建单个线程数的线程池，它可以保证先进先出的执⾏顺序；
4. Executors.newScheduledThreadPool：创建⼀个可以执⾏延迟任务的线程池；
5. Executors.newSingleThreadScheduledExecutor：创建⼀个单线程的可以执⾏延迟任务的线程池；
6. Executors.newWorkStealingPool：创建⼀个抢占式执⾏的线程池（任务执⾏顺序不确定）【JDK1.8 添加】。
7. ThreadPoolExecutor：最原始的创建线程池的⽅式，它包含了 7 个参数可供设置，后⾯会详细讲。
```
## 1.1固定数量的线程池(newFixedThreadPool)

```java
public static class FixedThreadPool {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService es = Executors.newFixedThreadPool(2);
        // 方式1
        Future<String> submit = es.submit(() -> Thread.currentThread().getName());
        System.out.println(submit.get());
        // 方式2
        es.execute(() -> System.out.println(Thread.currentThread().getName()));
    }
}
输出：
pool-1-thread-1
pool-1-thread-2
```

### 1.1.1execute和submit的区别

- execute和submit都属于线程池的方法，execute只能提交Runnable类型的任务
- submit既能提交Runnable类型任务也能提交Callable类型任务。
- execute()没有返回值
- submit有返回值，所以需要返回值的时候必须使用submit

```java
使用submit可以执行有返回值的任务或者是无返回值的任务；而execute只能执行不带返回值的任务。 
```

### 1.1.2⾃定义线程池名称或优先级

```java
/**
     * 定义线程池名称或优先级
     */
public static class FixedThreadPoll {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 创建线程工厂
        ThreadFactory threadFactory = r -> {
            // ！！！！！！！一定要注意：要把任务Runnable设置给新创建的线程
            Thread thread = new Thread(r);
            // 设置线程的命名规则
            thread.setName("我的线程：" + r.hashCode());
            // 设置线程的优先级
            thread.setPriority(Thread.MAX_PRIORITY);
            return thread;
        };
        ExecutorService es = Executors.newFixedThreadPool(2, threadFactory);
        // 参数是接口类，所以lambda的()->{}在括号里面，上面的r->{}是直接一个类，并不是作为入参
        Future<Integer> result = es.submit(() -> {
            int num = new Random().nextInt(10);
            System.out.println("线程优先级" + Thread.currentThread().getPriority() + ", 随机数：" + num);
            return num;
        });
        // 打印线程池返回结果
        System.out.println("返回结果：" + result.get());
    }
}
输出：
线程优先级10, 随机数：0
返回结果：0
```

```java
提供的功能：

        1. 设置（线程池中）线程的命名规则。

        2. 设置线程的优先级。

        3. 设置线程分组。

        4. 设置线程类型（用户线程、守护线程）。
```

## 1.2带缓存的线程池

```java
public static class CachedThreadPool1 {
    public static void main(String[] args) {
        // 创建线程池
        ExecutorService es = Executors.newCachedThreadPool();
        for (int i = 0; i < 3; i++) {
            int finalI = i;
            es.submit(() -> System.out.println("i : " + finalI + "|线程名称：" + Thread.currentThread().getName()));
        }
    }
}
输出：
i : 1|线程名称：pool-1-thread-2
i : 2|线程名称：pool-1-thread-3
i : 0|线程名称：pool-1-thread-1    
```

```java
优点：线程池会根据任务数量创建线程池，并且在一定时间内可以重复使用这些线程，产生相应的线程池。

缺点：适用于短时间有大量任务的场景，它的缺点是可能会占用很多的资源。
```

## 1.3