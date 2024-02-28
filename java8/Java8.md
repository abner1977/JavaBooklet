# java8新特性

## 一、变化

1、速度更快。（底层数据结构变化Map红黑树以及jvm改动元）

2、代码更少。Lambda表达式

3、stream API

3、便于并行

4、最大化减少空指针异常。 Optional 

## 二、lambda表达式

1、匿名函数，理解为可以传递的代码。简介灵活、紧凑。

2、左右遇一括号省  左侧推断类型省  能省则 省

3、Lambda需要函数式接口，（接口中只有一个抽象方法，可添加@FunctionalInterface 注解 标识 函数式接口）

4、四大核心接口。

| 消费型接口 | Consumer<T>   | void accept(T t);  |
| ---------- | ------------- | ------------------ |
| 供给型接口 | Supplier<T>   | T get();           |
| 函数式接口 | Function<T,R> | R appley(T t);     |
| 断言型接口 | Predicate<T>  | boolean test(T t); |

5、方法引用

6、构造器使用

7、数组引用





## 三、stream

是一种数据渠道，用于操作数据源（数组、集合等），所生成的元素序列。

```java
// 创建流的方式
        // 1 通过Collection 提供的stream 或者
        List<String> list1=new ArrayList<>();
        Stream<String> stream = list1.stream();
        list1.parallelStream();
        // 2 Arrays stream 方法
        String[] s2=new String[2];
       Stream<String> stream2 =  Arrays.stream(s2);
       // 3 stream 的of方法
        Stream<Integer> stream3 = Stream.of(1,2,3,4);
        //3  无限流
        Stream<Integer> stream4 = Stream.iterate(0,(x)->x+2);
        stream4.limit(4).forEach(System.out::println);
        // 4 生成随机数
        Stream.generate(()->Math.random())
                .forEach(System.out::println);
```





## 四、并行流

java8对ForkJoin 的封装使用   数据量越大越有效。

work- stealing  “工作窃取模式”   自身线程队列执行完毕，偷取其他线程队列的任务执行 ，最大化利用CPU

## 五、Optional



## 六、练习

```java
package com.abner;


import javax.security.auth.Subject;
import java.io.PrintStream;
import java.util.Comparator;
import java.util.function.*;

public class Main {

    public static void main(String[] args) {
        Integer num = test(10, x -> x + 200);
        System.out.println(num);
        /*四大核心接口*/
        // lambda 函数式编程
        //消费型
        consumerTest(10, (x) -> System.out.println(x + x));
        // 供给型
        String str = supplierTest("开头", () -> "结尾");
        System.out.println(str);
        //函数型
        String st = functionTest("ceshiabne", (x) -> x.toUpperCase());
        System.out.println(st);
        // 断言型
        String isok = predicateTest(10, (x) -> x > 11);
        System.out.println(isok);


//         /*方法引用  如lambda体的方法已经实现了，可以使用方法引用*/
        // 对象::实例方法名
        PrintStream ps = System.out;
        Consumer<String> con = ps::print;
        con.accept("打印");
        Test test=new Test();
        test.setAge("12");
        Supplier<String> sup=()->test.getAge();
        System.out.println(sup.get());
        // 类::静态方法名
        Comparator<Integer> com=Integer::compare;
        System.out.println(String.format("比较结果：%s",com.compare(1,2)));
        // 类::实例方法名
        BiPredicate<String,String> bp=String::equals;
        System.out.println(bp.test("1","2"));
        /*Lambda 体中 调用方法的参数列表和返回值类型，要与函数式
        * 接口中抽象方法的函数列表和返回值类型保持一致*/

        /*若Lambda参数列表中的第一参数是实例方法的调用者，第二参数是实例方法
        * 的参数时，可以使用ClassName::method*/

        /*构造器使用
        * className::new
        * 注意： 需要调用的构造器参数列表需要和抽象方法的参数列表一致
        * */
        Supplier<Test> supt=()->new Test();
        Supplier<Test> supN=Test::new;
        Test  test2=supN.get();
        System.out.println(test2);

        /*数组引用
        * Type::new
        * */
        //lambda
        Function<Integer,String[]> fun=(x)->new String[x];
        System.out.println(fun.apply(10).length);
        //数组引用替换
        Function<Integer,String[]> fun2=String[]::new;
        System.out.println(fun2.apply(20).length);

    }


    public static Integer test(Integer num, MyInterface myInterface) {
        return myInterface.calculate(num);
    }

    public static void consumerTest(Integer num, Consumer<Integer> con) {
        con.accept(num);
    }

    public static String supplierTest(String num, Supplier<String> su) {
        String param = su.get();
        return num + param;
    }

    public static String functionTest(String str, Function<String, String> fu) {
        return fu.apply(str);
    }

    public static String predicateTest(Integer a, Predicate<Integer> pr) {
        if (pr.test(a)) {
            return "1";
        } else {
            return "2";
        }
    }


}

```

## 七、特殊操作

reduce：规约，可以将流中元素反复结合起来，得到一个值。

```java
  // reduce：规约，可以将流中元素反复结合起来，得到一个值。
        List<Integer> list = Arrays.asList(1,2,3,4,5);
        Integer sum = list.stream().reduce(0,(x,y)->x+y);
        Optional<Integer> sum2= list.stream().reduce(Integer::sum);
        System.out.println("sum---"+sum);
        System.out.println("sum---"+sum2.get());
```

filter :流中 排除一些元素

limit :截断流，使元素不超过指定数量

skip(n):跳过元素，返回一个扔掉前n个元素的流。不足n个，返回空流。和limit(n)互补。

distinct:去重，通过hashCode和equals去除重复元素。

allmatch:是否匹配所有元素

anymatch:至少匹配一个元素

nonematch:是否没有匹配所有元素

findFirst:返回第一个元素

findAny:返回任意元素

count:总个数

max:流中最大值

min:流中最小值

collect:将流转换为其他形式，接收一个Collector接口的实现。用于流中数据的汇总

## 八、疑问

1、为什么lambda表达式使用的局部变量要是final的？

第一是实例变量都存储在堆中，堆是线程共享的。而**局部变量则保存在栈上**。如果Lambda可以直接访问局部变量，而且Lambda是在一个线程中使用的，则使用**Lambda的线程，可能会在分配该变量的线程将这个变量收回之后，去访问该变量**。因此，Java为避免这个问题，在访问自由局部变量时，**实际上是在访问它的副本**，而不是访问原始变量。为了保证局部变量和lambda中复制品 的数据一致性，就必须要这个限制。

第二，这一限制**不鼓励你使用改变外部变量的典型命令式编程模式**(这种模式会阻碍Java8很容易做到的并行处理)。

**<u>Lambda表达式中访问的实例变量和静态变量是可读可写的。</u>**





















