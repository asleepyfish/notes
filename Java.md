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

```java
单线程的线程池又什么意义？
1. 复用线程。
2. 单线程的线程池提供了任务队列和拒绝策略（当任务队列满了之后（Integer.MAX_VALUE），新来的任务就会拒绝策略）
```

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

ThreadPoolExecutor继承自AbstractExecutorService，而AbstractExecutorService实现了ExecutorService接口

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```

- corePoolSize ：线程池中核心线程数的最大值
- maximumPoolSize ：线程池中能拥有最多线程数
- workQueue：用于缓存任务的阻塞队列

当调用线程池execute() 方法添加一个任务时，线程池会做如下判断：

1、如果有空闲线程，则直接执行该任务；
2、如果没有空闲线程，且当前运行的线程数少于corePoolSize，则创建新的线程执行该任务；
3、如果没有空闲线程，且当前的线程数等于corePoolSize，同时阻塞队列未满，则将任务入队列，而不添加新的线程；
4、如果没有空闲线程，且阻塞队列已满，同时池中的线程数小于maximumPoolSize ，则创建新的线程执行任务；
5、如果没有空闲线程，且阻塞队列已满，同时池中的线程数等于maximumPoolSize ，则根据构造函数中的 handler 指定的策略来拒绝新的任务。

1）有界队列：

SynchronousQueue ：一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于 阻塞状态，吞吐量通常要高于LinkedBlockingQueue，静态工厂方法 Executors.newCachedThreadPool 使用了这个队列。
ArrayBlockingQueue：一个由数组支持的有界阻塞队列。此队列按 FIFO（先进先出）原则对元素进行排序。一旦创建了这样的缓存区，就不能再增加其容量。试图向已满队列中放入元素会导致操作受阻塞；试图从空队列中提取元素将导致类似阻塞。
2）无界队列：

LinkedBlockingQueue：基于链表结构的无界阻塞队列，它可以指定容量也可以不指定容量（实际上任何无限容量的队列/栈都是有容量的，这个容量就是Integer.MAX_VALUE）
PriorityBlockingQueue：是一个按照优先级进行内部元素排序的无界阻塞队列。队列中的元素必须实现 Comparable 接口，这样才能通过实现compareTo()方法进行排序。优先级最高的元素将始终排在队列的头部；PriorityBlockingQueue 不会保证优先级一样的元素的排序。

# 2. Stream流

## 2.1 流的定义

流是从支持数据处理操作的源生成的元素序列，源可以是数组、文件、集合、函数。流不是集合元素，它不是数据结构并不保存数据，它的主要目的在于计算。

## 2.2 生成流的方式

### 2.2.1 通过集合生成

```java
List<Integer> integerList = Arrays.asList(1, 2, 3, 4, 5);
Stream<Integer> stream = integerList.stream();
```

### 2.2.2 通过数组生成

```java
int[] intArr = new int[]{1, 2, 3, 4, 5};
IntStream stream = Arrays.stream(intArr);
```

通过Arrays.stream方法生成流，并且该方法生成的流是数值流【即IntStream】而不是Stream<Integer>。

使用数值流可以避免计算过程中拆箱装箱，提高性能。

Stream API提供了mapToInt、mapToDouble、mapToLong三种方式将对象流【即Stream】转换成对应的数值流，

同时提供了boxed方法将数值流转换为对象流。

### 2.2.3 通过值生成

```java
Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5);
```

通过Stream的of方法生成流，通过Stream的empty方法可以生成一个空流

### 2.2.4 通过文件生成

```java
Stream<String> lines = Files.lines(Paths.get("data.txt"), Charset.defaultCharset());
```

通过Files.line方法得到一个流，并且得到的每个流是给定文件中的一行

### 2.2.5 通过函数生成 

提供了iterate和generate两个静态方法从函数中生成流

#### 2.2.5.1 iterate

```java
Stream<Integer> stream = Stream.iterate(0, n -> n + 2).limit(5);
```

iterate方法接受两个参数，第一个为初始化值，第二个为进行的函数操作，因为iterator生成的流为无限流，通过limit方法对流进行了截断，只生成5个偶数

#### 2.2.5.2 generate

```java
Stream<Double> stream = Stream.generate(Math::random).limit(5);
```

generate方法接受一个参数，方法参数类型为Supplier，由它为流提供值。generate生成的流也是无限流，因此通过limit对流进行了截断。

## 2.3 流的操作类型

### 2.3.1 中间操作

一个流可以后面跟随零个或多个中间操作。其目的主要是打开流，做出某种程度的数据映射/过滤，然后返回一个新的流，交给下一个操作使用。这类操作都是惰性化的，仅仅调用到这类方法，并没有真正开始流的遍历，真正的遍历需等到终端操作时，常见的中间操作有下面即将介绍的filter、map等。

### 2.3.2  终端操作

一个流有且只能有一个终端操作，当这个操作执行后，流就被关闭了，无法再被操作，因此一个流只能被遍历一次，若想在遍历需要通过源数据在生成流。终端操作的执行，才会真正开始流的遍历。如下面即将介绍的count、collect等。

## 2.4 流的使用

### 2.4.1 中间操作

#### 2.4.1.1 filter筛选

```java
List<Integer> integerList = Arrays.asList(1, 1, 2, 3, 4, 5);
Stream<Integer> stream = integerList.stream().filter(i -> i > 3);
```

通过使用filter方法进行条件筛选，filter的方法参数为一个条件

#### 2.4.1.2 distinct去除重复元素

```java
List<Integer> integerList = Arrays.asList(1, 1, 2, 3, 4, 5);
Stream<Integer> stream = integerList.stream().distinct();
```

通过distinct方法快速去除重复的元素

#### 2.4.1.3 limit返回指定流个数

```java
List<Integer> integerList = Arrays.asList(1, 1, 2, 3, 4, 5); 
Stream<Integer> stream = integerList.stream().limit(3);
```

通过limit方法指定返回流的个数，limit的参数值必须>=0，否则将会抛出异常

#### 2.4.1.4 skip跳过流中的元素

```java
List<Integer> integerList = Arrays.asList(1, 1, 2, 3, 4, 5);
Stream<Integer> stream = integerList.stream().skip(2);
```

通过skip方法跳过流中的元素，上述例子跳过前两个元素，所以打印结果为2,3,4,5，skip的参数值必须>=0，否则将会抛出异常

#### 2.4.1.5 map流映射

所谓流映射就是将接受的元素映射成另外一个元素。

```java
List<String> stringList = Arrays.asList("Java 8", "Lambdas",  "In", "Action");
Stream<Integer> stream = stringList.stream().map(String::length);
```

通过map方法可以完成映射，该例子完成中String -> Integer的映射

#### 2.4.1.6 flatMap流转换

将一个流中的每个值都转换为另一个流。

```java
List<String> wordList = Arrays.asList("Hello", "World");
List<String> strList = wordList.stream()
    .map(w -> w.split(""))
    .flatMap(Arrays::stream)
    .distinct()
    .collect(Collectors.toList());
```

map(w -> w.split(""))的返回值为Stream<String[]>，我们想获取Stream<String>，可以通过flatMap方法完成Stream ->Stream的转换

#### 2.4.1.7 元素匹配

1、allMatch匹配所有

```java
List<Integer> integerList = Arrays.asList(1, 2, 3, 4, 5);
if (integerList.stream().allMatch(i -> i > 3)) {
    System.out.println("值都大于3");
}
```

2、anyMatch匹配其中一个

```java
List<Integer> integerList = Arrays.asList(1, 2, 3, 4, 5);
if (integerList.stream().anyMatch(i -> i > 3)) {
    System.out.println("存在大于3的值");
}
```

3、noneMatch全部不匹配

```java
List<Integer> integerList = Arrays.asList(1, 2, 3, 4, 5);
if (integerList.stream().noneMatch(i -> i > 3)) {
    System.out.println("值都小于3");
}
```

### 2.4.2 终端操作

#### 2.4.2.1 统计流中元素个数

1、通过count 

```java
List<Integer> integerList = Arrays.asList(1, 2, 3, 4, 5);
Long result = integerList.stream().count();
```

2、通过counting

```java
List<Integer> integerList = Arrays.asList(1, 2, 3, 4, 5);
Long result = integerList.stream().collect(counting());
```

与collect联合使用的时候特别有用

#### 2.4.2.2 查找

1、findFirst查找第一个

```java
List<Integer> integerList = Arrays.asList(1, 2, 3, 4, 5);
Optional<Integer> result = integerList.stream().filter(i -> i > 3).findFirst();
```

通过findFirst方法查找到第一个大于三的元素并打印。

2、findAny随机查找一个

```java
List<Integer> integerList = Arrays.asList(1, 2, 3, 4, 5);
Optional<Integer> result = integerList.stream().filter(i -> i > 3).findAny();
```

通过findAny方法查找到其中一个大于三的元素并打印，因为内部进行优化的原因，当找到第一个满足大于三的元素时就结束，该方法结果和findFirst方法结果一样。提供findAny方法是为了更好的利用并行流，findFirst方法在并行上限制更多

#### 2.4.2.3 reduce将流中的元素组合起来

假设我们对一个集合中的值进行求和

jdk8之前

```java
int sum = 0;
for (int i : integerList) {
	sum += i;
}
```

jdk8之后通过reduce进行处理

```java
int sum = integerList.stream().reduce(0, (a, b) -> (a + b));
```

一行就可以完成，还可以使用方法引用简写成：

```java
int sum = integerList.stream().reduce(0, Integer::sum);
```

reduce接受两个参数，一个初始值这里是0，一个BinaryOperator<T> accumulator来将两个元素结合起来产生一个新值，

另外reduce方法还有一个没有初始化值的重载方法。

#### 2.4.2.4 获取流中最小最大值

1、通过min/max获取最小最大值

```java
Optional<Integer> min = menu.stream().map(Dish::getCalories).min(Integer::compareTo);
Optional<Integer> max = menu.stream().map(Dish::getCalories).max(Integer::compareTo);
```

也可以写成

```java
OptionalInt min = menu.stream().mapToInt(Dish::getCalories).min();
OptionalInt max = menu.stream().mapToInt(Dish::getCalories).max();
```

min获取流中最小值，max获取流中最大值，方法参数为Comparator<? super T> comparator。

2、通过minBy/maxBy获取最小最大值

```java
Optional<Integer> min = menu.stream().map(Dish::getCalories).collect(minBy(Integer::compareTo));
Optional<Integer> max = menu.stream().map(Dish::getCalories).collect(maxBy(Integer::compareTo));
```

minBy获取流中最小值，maxBy获取流中最大值，方法参数为Comparator<? super T> comparator。

3、通过reduce获取最小最大值

```java
Optional<Integer> min = menu.stream().map(Dish::getCalories).reduce(Integer::min);
Optional<Integer> max = menu.stream().map(Dish::getCalories).reduce(Integer::max);
```

#### 2.4.2.5 求和

1、通过summingInt

```java
int sum = menu.stream().collect(summingInt(Dish::getCalories));
```

如果数据类型为double、long，则通过summingDouble、summingLong方法进行求和。

2、通过reduce

```java
int sum = menu.stream().map(Dish::getCalories).reduce(0, Integer::sum);
```

3、通过sum

```java
int sum = menu.stream().mapToInt(Dish::getCalories).sum();
```

在上面求和、求最大值、最小值的时候，对于相同操作有不同的方法可以选择执行。可以选择collect、reduce、min/max/sum方法，推荐使用min、max、sum方法。因为它最简洁易读，同时通过mapToInt将对象流转换为数值流，避免了装箱和拆箱操作。

#### 2.4.2.6 通过averagingInt求平均值

```java
double average = menu.stream().collect(averagingInt(Dish::getCalories));
```

如果数据类型为double、long，则通过averagingDouble、averagingLong方法进行求平均。

#### 2.4.2.7 通过summarizingInt同时求总和、平均值、最大值、最小值

```java
IntSummaryStatistics intSummaryStatistics = menu.stream().collect(summarizingInt(Dish::getCalories));
double average = intSummaryStatistics.getAverage();  //获取平均值
int min = intSummaryStatistics.getMin();  //获取最小值
int max = intSummaryStatistics.getMax();  //获取最大值
long sum = intSummaryStatistics.getSum();  //获取总和
```

如果数据类型为double、long，则通过summarizingDouble、summarizingLong方法。

#### 2.4.2.8 通过foreach进行元素遍历

```java
List<Integer> integerList = Arrays.asList(1, 2, 3, 4, 5);
integerList.stream().forEach(System.out::println);
```

#### 2.4.2.9 返回集合

```java
List<String> strings = menu.stream().map(Dish::getName).collect(Collectors.toList());
Set<String> sets = menu.stream().map(Dish::getName).collect(Collectors.toSet());
```

通过遍历和返回集合的使用发现流只是把原来的外部迭代放到了内部进行，这也是流的主要特点之一。内部迭代可以减少很多代码量

#### 2.4.2.10 通过joining拼接流中的元素

```java
String result = menu.stream().map(Dish::getName).collect(Collectors.joining(", "));
```

默认如果不通过map方法进行映射处理拼接的toString方法返回的字符串，joining的方法参数为元素的分界符，如果不指定生成的字符串将是一串的，可读性不强。

#### 2.4.2.11 进阶通过groupingBy进行分组

```java
Map<Type, List<Dish>> result = dishList.stream().collect(Collectors.groupingBy(Dish::getType));
```

在collect方法中传入groupingBy进行分组，其中groupingBy的方法参数为分类函数。还可以通过嵌套使用groupingBy进行多级分类。

```java
Map<String, Map<Integer, List<Dish>>> result = menu.stream().collect(Collectors.groupingBy(Dish::getName,
                Collectors.groupingBy(Dish::getCalories)));
```

#### 2.4.2.12 进阶通过partitioningBy进行分区

分区是特殊的分组，它分类依据是true和false，所以返回的结果最多可以分为两组

```java
Map<Boolean, List<Dish>> result = menu.stream().collect(Collectors.partitioningBy(Dish::isVegetarian));
```

等同于

```java
Map<Boolean, List<Dish>> result = menu.stream().collect(Collectors.groupingBy(Dish::isVegetarian));
```

这个例子可能并不能看出分区和分类的区别，甚至觉得分区根本没有必要，换个明显一点的例子：

```java
List<Integer> integerList = Arrays.asList(1, 2, 3, 4, 5);
Map<Boolean, List<Integer>> result = integerList.stream().collect(Collectors.partitioningBy(i -> i < 3));
```

返回值的键仍然是布尔类型，但是它的分类是根据范围进行分类的，分区比较适合处理根据范围进行分类。

