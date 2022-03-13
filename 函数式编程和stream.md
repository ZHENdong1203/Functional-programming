## 什么是函数式接口：
函数式接口(Functional Interface)就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口。  
函数式接口可以被隐式转换为 lambda 表达式。  
lambda体现了函数式接口的一个实现， ->左边是参数，右边是函数体，同样的使用匿名内部类也可以实现。（Lambda表达式的本质只是一个"语法糖"）  

例如Consumer这个函数式接口：  
**匿名内部类实现：**  
```java
//匿名内部类
        Consumer testFunction1 = new Consumer() {
            @Override
            public void accept(Object o) {
                System.out.println("hello" + o);
            }
        };
```  
**Lambda表达式:**  
```java
//Lambda
        Consumer testFunction = message -> System.out.println("hello" + message);
```  

JDK提供的一些核心函数接口  
![image](https://user-images.githubusercontent.com/36890438/158048426-440ea1ac-d678-4708-95e7-979d732ae87a.png)

简单介绍：  
以Function接口为例：  
Funtion接口输入一个T，返回R。  
![image](https://user-images.githubusercontent.com/36890438/158048552-92f8f048-899a-4bb8-9e0c-1f84f6780dc8.png)

接口源码：  
![image](https://user-images.githubusercontent.com/36890438/158048674-174e7ef6-ac1e-474c-9c6a-b78d53b66e75.png)

### 1.@FunctionalInterface注解  
Java 8为函数式接口引入了一个新注解@FunctionalInterface，主要用于编译级错误检查，加上该注解，当你写的接口不符合函数式接口定义的时候，编译器会报错。  
加不加 @FunctionalInterface 对于接口是不是函数式接口没有影响，该注解只是提醒编译器去检查该接口是否仅包含一个抽象方法  
### 2.接口中的非抽象方法  
在以下几种情况下，接口不会把其当作是抽象方法，从而符合函数式接口的定义。  
- 所接口中所定义的方法式默认方法，使用default修饰，有其默认实现。
- 方法是静态方法，因为静态方法不能是抽象方法，而是一个已经实现了的方法。
- 方法是继承来自 Object 类的public方法，因为任何一个接口或类都继承了 Object 的方法  

例如compose这个方法  
他是一个默认方法，因为默认方法不是抽象方法，其有一个默认实现，所以是符合函数式接口的定义的。  

### 非抽象方法解析
#### 1.compose  
compose方法是一个默认方法，这个方法接收一个function作为参数，将参数function执行的结果作为参数给调用的function，以此来实现两个function组合的功能。  
#### 2.andThen  
andThen方法也是接收一个function作为参数，与compse不同的是，先执行本身的apply方法，将执行的结果作为参数给参数中的function。
#### 3.identity  
输入什么，输出就是什么，返回本身

```java
public class TestService {
    public static void main(String[] args) {
        System.out.println("compose方法:");
        System.out.println(TestService.compute1(2, value -> value * 3, value -> value * value) );
        System.out.println("andThen方法:");
        System.out.println(TestService.compute(2, value -> value * 3, value -> value * value) );
    }

    public static int compute1(int a, Function<Integer, Integer> function1, Function<Integer, Integer> function2) {
        return function1.compose(function2).apply(a);
    }

    public static int compute(int a, Function<Integer, Integer> function1, Function<Integer, Integer> function2) {
        return function1.andThen(function2).apply(a);
    }
}
```
![image](https://user-images.githubusercontent.com/36890438/158049531-6c18a01b-0421-4af3-9153-73e01d0552fd.png)

```java
public class TestService {
    public static void main(String[] args) {
        List<StudentVO> list = getStudentList();
        Map<Long, StudentVO> map = list.stream()
                .collect(Collectors.toMap(StudentVO::getId, Function.identity()));

        System.out.println(map);
    }

    private static List<StudentVO> getStudentList()
    {
        StudentVO studentVO1 = new StudentVO();
        studentVO1.setId(1L);
        studentVO1.setName("张三");
        StudentVO studentVO2 = new StudentVO();
        studentVO2.setId(2L);
        studentVO2.setName("李四");
        List<StudentVO> list = Lists.newArrayList();
        list.add(studentVO1);
        list.add(studentVO2);
        return list;
    }
}
```
![image](https://user-images.githubusercontent.com/36890438/158049834-671ba666-e732-45b6-980b-251b8a201e55.png)


## stream  
### 惰性求值和及早求值  
判断一个操作是惰性求值还是及早求值很简单：只需看它的返回值。如果返回值是 Stream， 那么是惰性求值；如果返回值是另一个值或为空，那么就是及早求值。  
stream中的惰性求值方法：  

stream中的及早求值方法：  
![image](https://user-images.githubusercontent.com/36890438/158050865-edd408c5-dc27-4bc4-89d9-994cecd354bb.png)

### 介绍几个常用的惰性求值和及早求值方法：  
#### 1.filter  
用于对Stream中的元素进行过滤，返回一个过滤后的Stream  
filter操作用到的函数接口正是前面介绍过的 Predicate（输入T，返回true或flase）  
![image](https://user-images.githubusercontent.com/36890438/158051011-8de0bbe9-61f3-4d26-88a3-df80b9752899.png)

例如：  
过滤出等于张三的学生：  
```java
public class TestService {
    public static void main(String[] args) {
        List<StudentVO> testList = getStudentList().stream().filter(vo -> vo.getName().equals("张三"))
                .collect(Collectors.toList());

        System.out.println(testList);
    }

    private static List<StudentVO> getStudentList()
    {
        StudentVO studentVO1 = new StudentVO();
        studentVO1.setId(1L);
        studentVO1.setName("张三");
        StudentVO studentVO2 = new StudentVO();
        studentVO2.setId(2L);
        studentVO2.setName("李四");
        List<StudentVO> list = Lists.newArrayList();
        list.add(studentVO1);
        list.add(studentVO2);
        return list;
    }
}
```


#### map
元素一对一转换。  
![image](https://user-images.githubusercontent.com/36890438/158051224-18b80837-6bb5-4e1e-aedc-08401c85e544.png)

map用到的函数接口是Function（输入T，返回R）  
![image](https://user-images.githubusercontent.com/36890438/158051241-cac7f998-6954-4491-aa3f-a824ef78e11b.png)

例如：  
将字母变成大写  
```java
public static void main(String[] args) {
        List<String> testList = Stream.of("a", "b", "c")
                .map(s -> s.toUpperCase(Locale.ROOT))
                        .collect(Collectors.toList());

        System.out.println(testList);
    }
```

#### max和min  
找到stream中的最大值和最小值.  
例如：  
找到ID较小的学生：  
```java
public static void main(String[] args) {
        List<StudentVO> studentVos = Arrays.asList(new StudentVO("张三", 1L),
                new StudentVO("李四", 2L));

        StudentVO a = studentVos.stream().min(Comparator.comparing(vo -> vo.getId())).get();

        System.out.println(a);
    }
```

comparing接收一个Funciton，返回一个比较器  
```java
    /**
     * Returns a lexicographic-order comparator with another comparator.
     * If this {@code Comparator} considers two elements equal, i.e.
     * {@code compare(a, b) == 0}, {@code other} is used to determine the order.
     *
     * <p>The returned comparator is serializable if the specified comparator
     * is also serializable.
     *
     * @apiNote
     * For example, to sort a collection of {@code String} based on the length
     * and then case-insensitive natural ordering, the comparator can be
     * composed using following code,
     *
     * <pre>{@code
     *     Comparator<String> cmp = Comparator.comparingInt(String::length)
     *             .thenComparing(String.CASE_INSENSITIVE_ORDER);
     * }</pre>
     *
     * @param  other the other comparator to be used when this comparator
     *         compares two objects that are equal.
     * @return a lexicographic-order comparator composed of this and then the
     *         other comparator
     * @throws NullPointerException if the argument is null.
     * @since 1.8
     */
    default Comparator<T> thenComparing(Comparator<? super T> other) {
        Objects.requireNonNull(other);
        return (Comparator<T> & Serializable) (c1, c2) -> {
            int res = compare(c1, c2);
            return (res != 0) ? res : other.compare(c1, c2);
        };
    }
```
min或max方法通过比较器去处理流  
```java
    /**
     * Returns the minimum element of this stream according to the provided
     * {@code Comparator}.  This is a special case of a
     * <a href="package-summary.html#Reduction">reduction</a>.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">terminal operation</a>.
     *
     * @param comparator a <a href="package-summary.html#NonInterference">non-interfering</a>,
     *                   <a href="package-summary.html#Statelessness">stateless</a>
     *                   {@code Comparator} to compare elements of this stream
     * @return an {@code Optional} describing the minimum element of this stream,
     * or an empty {@code Optional} if the stream is empty
     * @throws NullPointerException if the minimum element is null
     */
    Optional<T> min(Comparator<? super T> comparator);
```  
Optional是啥？  
一个容器对象，它可能包含也可能不包含非空值。如果存在一个值，isPresent()将返回true, get()将返回值。  

#### collect收集器
转换成其他集合  toList， toMap， toSet
分块  partitioningBy  
它接受一个流，并将其分成两部分。它使用 Predicate 对象判断一个元素应该属于哪个部分，并根据布尔值返回一 个 Map。  
![image](https://user-images.githubusercontent.com/36890438/158053240-d555ec84-a4dd-4cc9-b11e-ed97b509763f.png)

分组  groupingBy  
groupingBy 收集器接受一个分类函数，用来对数据分组，根据分组条件返回Map。  
![image](https://user-images.githubusercontent.com/36890438/158053361-bcb00f90-0f3d-467d-a6a4-f45b0375e7b9.png)

例如：  
根据性别进行分块和分组：  
```java
public static void main(String[] args) {
        Map<String, List<StudentVO>> groupingMap = getStudentList().stream().collect(Collectors.groupingBy(vo -> vo.getSex()));

        Map<Boolean, List<StudentVO>> partitioningMap = getStudentList().stream().collect(Collectors.partitioningBy(vo -> "男".equals(vo.getSex())));

        System.out.println("分块:");
        System.out.println(partitioningMap);
        System.out.println("分组：");
        System.out.println(groupingMap);

    }
```

字符串连接  joining  
joining收集器将流中的值拼接成字符串，提供分隔符，前缀，后缀三个参数。  

例如：  
输出学生名字，逗号拼接，[]作为前缀和后缀  
```java
public static void main(String[] args) {
        String name = getStudentList().stream()
                        .map(vo->vo.getName())
                                .collect(Collectors.joining(",", "[", "]"));

        System.out.println(name);

    }
```
![image](https://user-images.githubusercontent.com/36890438/158053555-cc089922-4fde-4311-928d-0e14186fd42f.png)

还有很多收集器  









