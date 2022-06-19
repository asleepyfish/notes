# 1 流的定义

流是从支持数据处理操作的源生成的元素序列，源可以是数组、文件、集合、函数。流不是集合元素，它不是数据结构并不保存数据，它的主要目的在于计算。

# 2 生成流的方式

## 2.1 通过集合生成

```java
List<Integer> integerList = Arrays.asList(1, 2, 3, 4, 5);
Stream<Integer> stream = integerList.stream();
```

## 2.2 通过数组生成

```java
int[] intArr = new int[]{1, 2, 3, 4, 5};
IntStream stream = Arrays.stream(intArr);
```

通过Arrays.stream方法生成流，并且该方法生成的流是数值流【即IntStream】而不是Stream<Integer>。

使用数值流可以避免计算过程中拆箱装箱，提高性能。

Stream API提供了mapToInt、mapToDouble、mapToLong三种方式将对象流【即Stream】转换成对应的数值流，

同时提供了boxed方法将数值流转换为对象流。

## 2.3 通过值生成

```java
Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5);
```

通过Stream的of方法生成流，通过Stream的empty方法可以生成一个空流

## 2.4 通过文件生成

```java
Stream<String> lines = Files.lines(Paths.get("data.txt"), Charset.defaultCharset());
```

通过Files.line方法得到一个流，并且得到的每个流是给定文件中的一行

## 2.5 通过函数生成 

提供了iterate和generate两个静态方法从函数中生成流

### 2.5.1 iterate

```java
Stream<Integer> stream = Stream.iterate(0, n -> n + 2).limit(5);
```

iterate方法接受两个参数，第一个为初始化值，第二个为进行的函数操作，因为iterator生成的流为无限流，通过limit方法对流进行了截断，只生成5个偶数

### 2.5.2 generate

```java
Stream<Double> stream = Stream.generate(Math::random).limit(5);
```

generate方法接受一个参数，方法参数类型为Supplier，由它为流提供值。generate生成的流也是无限流，因此通过limit对流进行了截断。

# 3 流的操作类型

## 3.1 中间操作

一个流可以后面跟随零个或多个中间操作。其目的主要是打开流，做出某种程度的数据映射/过滤，然后返回一个新的流，交给下一个操作使用。这类操作都是惰性化的，仅仅调用到这类方法，并没有真正开始流的遍历，真正的遍历需等到终端操作时，常见的中间操作有下面即将介绍的filter、map等。

## 3.2  终端操作

一个流有且只能有一个终端操作，当这个操作执行后，流就被关闭了，无法再被操作，因此一个流只能被遍历一次，若想在遍历需要通过源数据在生成流。终端操作的执行，才会真正开始流的遍历。如下面即将介绍的count、collect等。

# 4 流的使用

## 4.1 中间操作

### 4.1.1 filter筛选

```java
List<Integer> integerList = Arrays.asList(1, 1, 2, 3, 4, 5);
Stream<Integer> stream = integerList.stream().filter(i -> i > 3);
```

通过使用filter方法进行条件筛选，filter的方法参数为一个条件

### 4.1.2 distinct去除重复元素

```java
List<Integer> integerList = Arrays.asList(1, 1, 2, 3, 4, 5);
Stream<Integer> stream = integerList.stream().distinct();
```

通过distinct方法快速去除重复的元素

### 4.1.3 limit返回指定流个数

```java
List<Integer> integerList = Arrays.asList(1, 1, 2, 3, 4, 5); 
Stream<Integer> stream = integerList.stream().limit(3);
```

通过limit方法指定返回流的个数，limit的参数值必须>=0，否则将会抛出异常

### 4.1.4 skip跳过流中的元素

```java
List<Integer> integerList = Arrays.asList(1, 1, 2, 3, 4, 5);
Stream<Integer> stream = integerList.stream().skip(2);
```

通过skip方法跳过流中的元素，上述例子跳过前两个元素，所以打印结果为2,3,4,5，skip的参数值必须>=0，否则将会抛出异常

### 4.1.5 map流映射

所谓流映射就是将接受的元素映射成另外一个元素。

```java
List<String> stringList = Arrays.asList("Java 8", "Lambdas",  "In", "Action");
Stream<Integer> stream = stringList.stream().map(String::length);
```

通过map方法可以完成映射，该例子完成中String -> Integer的映射

### 4.1.6 flatMap流转换

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

### 4.1.7 元素匹配

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

## 4.2 终端操作

### 4.2.1 统计流中元素个数

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

### 4.2.2 查找

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

### 4.2.3 reduce将流中的元素组合起来

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

### 4.2.4 获取流中最小最大值

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

### 4.2.5 求和

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

### 4.2.6 通过averagingInt求平均值

```java
double average = menu.stream().collect(averagingInt(Dish::getCalories));
```

如果数据类型为double、long，则通过averagingDouble、averagingLong方法进行求平均。

### 4.2.7 通过summarizingInt同时求总和、平均值、最大值、最小值

```java
IntSummaryStatistics intSummaryStatistics = menu.stream().collect(summarizingInt(Dish::getCalories));
double average = intSummaryStatistics.getAverage();  //获取平均值
int min = intSummaryStatistics.getMin();  //获取最小值
int max = intSummaryStatistics.getMax();  //获取最大值
long sum = intSummaryStatistics.getSum();  //获取总和
```

如果数据类型为double、long，则通过summarizingDouble、summarizingLong方法。

### 4.2.8 通过foreach进行元素遍历

```java
List<Integer> integerList = Arrays.asList(1, 2, 3, 4, 5);
integerList.stream().forEach(System.out::println);
```

### 4.2.9 返回集合

```java
List<String> strings = menu.stream().map(Dish::getName).collect(Collectors.toList());
Set<String> sets = menu.stream().map(Dish::getName).collect(Collectors.toSet());
```

通过遍历和返回集合的使用发现流只是把原来的外部迭代放到了内部进行，这也是流的主要特点之一。内部迭代可以减少很多代码量

### 4.2.10 通过joining拼接流中的元素

```java
String result = menu.stream().map(Dish::getName).collect(Collectors.joining(", "));
```

默认如果不通过map方法进行d映射处理拼接的toString方法返回的字符串，joining的方法参数为元素的分界符，如果不指定生成的字符串将是一串的，可读性不强。

### 4.2.11 进阶通过groupingBy进行分组

```java
Map<Type, List<Dish>> result = dishList.stream().collect(Collectors.groupingBy(Dish::getType));
```

在collect方法中传入groupingBy进行分组，其中groupingBy的方法参数为分类函数。还可以通过嵌套使用groupingBy进行多级分类。

```java
Map<String, Map<Integer, List<Dish>>> result = menu.stream().collect(Collectors.groupingBy(Dish::getName,
                Collectors.groupingBy(Dish::getCalories)));
```

### 4.2.12 进阶通过partitioningBy进行分区

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