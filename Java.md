# 1. 线程池的创建

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
## 1.1 固定数量的线程池(newFixedThreadPool)

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

### 1.1.1 execute和submit的区别

- execute和submit都属于线程池的方法，execute只能提交Runnable类型的任务
- submit既能提交Runnable类型任务也能提交Callable类型任务。
- execute()没有返回值
- submit有返回值，所以需要返回值的时候必须使用submit

```java
使用submit可以执行有返回值的任务或者是无返回值的任务；而execute只能执行不带返回值的任务。 
```

### 1.1.2 ⾃定义线程池名称或优先级

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

## 1.2 带缓存的线程池(newCachedThreadPool)

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

## 1.3 单线程线程池(newSingleThreadExecutor)

```java
public static class SingleThreadExecutor1 {
    public static void main(String[] args) {
        ExecutorService es = Executors.newSingleThreadExecutor();
        for (int i = 0; i < 5; i++) {
            es.submit(() -> System.out.println("线程名：" + Thread.currentThread().getName()));
        }
    }
}
输出：
线程名：pool-1-thread-1
线程名：pool-1-thread-1
线程名：pool-1-thread-1
线程名：pool-1-thread-1
线程名：pool-1-thread-1    
```

```jav
单线程的线程池又什么意义？
1. 复用线程。
2. 单线程的线程池提供了任务队列和拒绝策略（当任务队列满了之后（Integer.MAX_VALUE），新来的任务就会拒绝策略）
```

## 

## 1.4 执行定时任务的线程池(newScheduledThreadPool)

### 1.4.1 延迟执行（一次）

```java
public static class ScheduledThreadPool1 {
    public static void main(String[] args) {
        ScheduledExecutorService ses = Executors.newScheduledThreadPool(3);
        System.out.println("添加任务的时间：" + LocalDateTime.now());
        ses.schedule(() -> System.out.println("执行子任务：" + LocalDateTime.now()), 3, TimeUnit.SECONDS);
    }
}
输出：
添加任务的时间：2022-06-13T18:57:19.742726900
执行子任务：2022-06-13T18:57:22.758147900    
```

### 1.4.2 固定频率执行

```java
public static void main(String[] args) {
    ScheduledExecutorService ses = Executors.newScheduledThreadPool(3);
    System.out.println("添加任务的时间：" + LocalDateTime.now());
    ses.scheduleAtFixedRate(() -> System.out.println(Thread.currentThread().getName() + "执行任务：" + LocalDateTime.now()), 2, 4, TimeUnit.SECONDS);
    // 2是第一次多久之后执行，4是定时任务每隔多久执行一次
}
输出：
添加任务的时间：2022-06-13T19:08:21.725930300
pool-1-thread-1执行任务：2022-06-13T19:08:23.741930800
pool-1-thread-1执行任务：2022-06-13T19:08:27.736881200
pool-1-thread-2执行任务：2022-06-13T19:08:31.742574400
pool-1-thread-2执行任务：2022-06-13T19:08:35.743332900    
```

```java
如果在定义定时任务中设置了内部睡眠时间，比如
    ses.scheduleAtFixedRate((Runnable) () -> {
        System.out.println("执行任务： " + LocalDateTime.now());
        try {
            Thread.sleep(5 * 1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    },2,4, TimeUnit.SECONDS);   
5 > 4 ,哪个值大以哪个值作为定时任务的执行周期
```

### 1.4.3 scheduleAtFixedRate VS scheduleWithFixedDelay

```java
public static class ScheduledThreadPool3 {
    public static void main(String[] args) {
        // 创建线程池
        ScheduledExecutorService ses = Executors.newScheduledThreadPool(3);
        System.out.println("添加任务时间：" + LocalDateTime.now());
        // 2s之后开始执行定时任务，定时任务每隔4s执行一次
        ses.scheduleWithFixedDelay(() -> {
            System.out.println(Thread.currentThread().getName() + "执行任务：" + LocalDateTime.now());
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, 2, 4, TimeUnit.SECONDS);
    }
}
输出：
添加任务时间：2022-06-13T19:20:34.311246
pool-1-thread-1执行任务：2022-06-13T19:20:36.317013100
pool-1-thread-1执行任务：2022-06-13T19:20:41.340114100
pool-1-thread-2执行任务：2022-06-13T19:20:46.362309400
pool-1-thread-2执行任务：2022-06-13T19:20:51.369756400    
```

```java
scheduleAtFixedRate 是以上⼀次任务的开始时间，作为下次定时任务的参考时间的
scheduleWithFixedDelay 是以上⼀次任务的结束时间，作为下次定时任务的参考时间的。
```

## 1.5 定时任务单线程(newSingleThreadScheduledExecutor)

```java
public static class SingleThreadScheduledExecutor1 {
    public static void main(String[] args) {
        ScheduledExecutorService ses = Executors.newSingleThreadScheduledExecutor();
        System.out.println("添加任务时间：" + LocalDateTime.now());
        ses.scheduleAtFixedRate(() -> System.out.println(Thread.currentThread().getName() + "执行时间：" + LocalDateTime.now()), 2, 4, TimeUnit.SECONDS);
    }
}
输出：
添加任务时间：2022-06-13T19:27:06.143427900
pool-1-thread-1执行时间：2022-06-13T19:27:08.146768100
pool-1-thread-1执行时间：2022-06-13T19:27:12.156625
pool-1-thread-1执行时间：2022-06-13T19:27:16.158957500
pool-1-thread-1执行时间：2022-06-13T19:27:20.156330800    
```

## 1.6 根据当前CPU⽣成线程池(newWorkStealingPool)

```java
public static class WorkStealingPool1 {
    public static void main(String[] args) {
        ExecutorService service = Executors.newWorkStealingPool();
        for (int i = 0; i < 5; i++) {
            service.submit(() -> {
                System.out.println("线程名：" + Thread.currentThread().getName());
            });
            while (!service.isTerminated()) {
            }
        }
    }
}
输出：
线程名：ForkJoinPool-1-worker-1    
```

## 1.7 ThreadPoolExecutor