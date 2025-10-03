# Lambda Expression

​       Lambda表达式可以理解为简洁地表示可传递的匿名函数的一种方式，他没有名称，但是有参数列表、函数主体、返回类型，可能还有一个可以抛出的异常列表,使用Lambda表达式，就可以只传递业务逻辑真正需要的代码。

​			**只有在接受函数式接口的地方才可以使用`Lambda`表达式**

## 基本语法

Java8中有效的Lambda表达式如下所示：

```java
(string s)->s.length;
(Apple a)->a.getWeight()>150
(int x,int y)->{
    System.out.println("Result:"+(x+y));//对于多行表达式的Lambda表达式可以使用花括号来囊括其方法体，单行可以不用
}
()->42 //Lambda表达式也可以没有输入参数
```

总结下来，`Lambda`表达式的基本语法为`(parameters)->expressions`.，注意：

- `(Integer i)->return "Alan"+i;`，对于单行语句，不使用花括号囊括的方法体，不应用使用`return`，而是直接返回内容。
- `(String s)->{'IronMan";}`,对于加了花括号的方法体，则必须使用`return`关键字来进行显示的返回。

## 函数式接口

**Lambda表达式可以在函数式接口中运用，函数式接口是仅仅定义了一个抽象方法的接口**。例如`Predicate`：

```java
public interface Predicate<T>{ boolean test(T t);}
```

常用的函数式接口还有：

- `Comparator<T>{ int compare(T o1, T o2);}`
- `Runnable{ void run()}`
- ...

> 函数描述符

函数式接口大的抽象方法的的签名基本上就是`Lambda`表达式的签名，**这种抽象方法就叫做函数描述符**。

- `Predicate`接口的函数描述符为`(T)->boolean`
- `Comparator`接口的函数描述符为：`(T,T)->int`
- `Runnable`接口的函数描述符为：`()->void`.

```java
public class FirstUsing {
    public static void main(String[] args) throws IOException {
        FileReader r=new FileReader(
                "FileForTesting.txt");
        BufferedReader br=new BufferedReader(r);
        processFile(a->
        {
            String s1=a.readLine();
            System.out.println(s1);
        return s1;},br);
        br.close();

    }
    public static void processFile(BufferedReaderProcessor processor,BufferedReader reader) throws IOException {
        processor.processLine(reader);
    }
}
interface BufferedReaderProcessor {
    String processLine(BufferedReader reader) throws IOException;
}
```

即使用函数式接口与此前的行为参数化一样：

1. 新建一个函数式接口
2. 建立一个引用函数式接口的方法
3. 使用Lambda表达式来灌注具体的行为。

## 常用的函数式接口

### Predicate

Predicate为谓词函数式接口：`Predicate<T>{ boolean test(T t);}`，即输入一个类型返回一个`boolean`的值，用于条件筛选。它中间定义的方法为：`boolean test(T t)`.

```
public class FrequentUsingInterface {
    public static void main(String[] args) {
        for (Apple apple : Apple.apples) {
            usingPredicate((Apple a)->a.getColor().equals("yellow"),apple);
        }
    }
    //由于使用的泛型直接制定了对象，因此这里要声明泛型的参数为Apple类。
    public static void usingPredicate(Predicate<Apple> p,Apple a){
        if (p.test(a)){
            System.out.println(a);
        }
    }
}
```

### Function

`Function`接口的函数描述符为`(T )->R`，即输入一个类型的对象，返回另一个类型的对象，它中间定义的方法为：`R apply(T t)`.

### Consumer

`Consumer`是一个消费者方法，输入一个类型的对象，返回为空，其间定义方法为`void accept(T t);`,函数描述符为`T->void`.

`forEach()`就是一个`Consumer`接口的继承者。



## 类型检查，推断与限制



### 类型检查

Lambda的类型检查是从`Lambda`中的上下文推断出来的，具体类型检查过程为：

1. 调用`Lambda`表达式：`filter(invertory,(Apple a)->a.getWeight()>150);`
2. 查看`filter`函数定义：`filter(List<Apple> inventory, Predicate<Apple> p)`
3. 查看接口的具体抽象方法`public interface Predicate<T>{ boolean test(Apple apple)}`
4. 进行类型检查，是否行为模式符合函数签名`(Apple)->boolean`

**由此过程的推理：如果方法引用中使用了某种**特定类型的方法**（比如类 `Apple` 的 `getColor` 方法），而没有明确泛型类型时，编译器无法推断参数的类型。因此，必须明确泛型类型，写明泛型的具体类型**

如：

```java
public static void main(String[] args) {
    for (Apple apple : Apple.apples) {
        usingPredicate((Apple a)->a.getColor().equals("yellow"),apple);
    }
}
public static <T> void usingPredicate (Predicate<T> p,T t){
    if (p.test(t)){
        System.out.println(t);
    }
}
```

中，函数`usingPredicate`没有使用特定类型的方法，里面可以继续使用泛型。

而在`public static void usingPredicate (Predicate<T> p,Apple a)`这里，既不是泛型方法，且已经指明了：

- `Predicate<Apple>` 的泛型已经明确为 `Apple`。
- 因为 Lambda 表达式 `a -> a.getColor().equals("green")` 使用了 `Apple.getColor()`（属于特定类型的方法），因此泛型类型必须明确写为 `Apple`。
- 如果不明确指定类型（比如使用 `Predicate<?>` 或原始 `Predicate`），编译器会因为类型无法推断而报错。

> 显式定义函数式接口

```java
Comparator<Integer> c=(a,b)->a.compareTo(b);
```

使用Lambda表示式子显示进行定义时，最好在泛型里面写明具体类型，以便于编译器进行类型验证。

### 类型推断

Java编译器会从上下文中推断出使用什么函数时表达式接口来配合Lambda表达式，也可以推断出适合`Lambda`的函数签名。

> 没有类型推断

`List<Apple> list=usingPredicate((Apple a)->a.getColor().equals("yellow"),apple)`

Lambda函数中指明了参数具体类型为`Apple`,就没有使用到类型推断

> 具有类型推断

`List<Apple> list=usingPredicate((a)->a.getColor().equals("yellow"),apple)`

使用到了类型推断，省略了函数描述符中对于输入参数的类型的下定义，而通过目标类型来得到。这里是直接在`List<>`泛型中指明。

**当`Lambda`表达式仅有一个类型需要推断时，可以省略参数两边的括号：**

`List<Apple> list=usingPredicate(a->a.getColor().equals("yellow"),apple)`

### 使用局部变量

Lambda表达式支持使用自由变量，被称作捕获Lambda：

```java
int port=32;
Runnable r=()->{
    System.out.println(port);
};
r.run();
```

这里面就使用了外部定义的变量，对其进行了打印，**实际上，`Lambda`表达式可以无限制地捕获实例变量和静态变量，但是对于外部变量的使用具有一定限制：其引用的局部变量是最终`final`修饰的，或者事实上是最终的：**

```
int port=32;
Runnable r=()->{
    System.out.println(port);
};
r.run();
port=332;
```

以上代码就把无法进行编译，因为`port`进行了修改，在事实上已经不是最终的了。局部变量限制原因如下：

1. 实例变量都存储在堆中，而局部变量存储在栈中，在Jvm中每一个线程都有一个方法栈区，直接访问一个局部变量可能会是创建该局部变量的线程收回之后才访问，因此java中访问自由局部变量实际上是访问其副本而不是原始便利，如果局部变量仅仅赋值一次就没有什么区别，因此就有了这个限制。
2. 这一限制**不鼓励你使用改变外部变量的典型命令式函数编程模式**

## 方法引用

方法引用可以使我们更快地使用现有的方法定义，并像`Lambda`一样传递它们。

先前：

```java
Apple.apples.sort((Apple a,Apple b)-> a.getWeight()>b.getWeight()?1:-1);
```

现在：`Apple.apples.sort(Comparator.comparing(Apple::getWeight));`

方法引用的思想是：如果`Lambda`表达式直接代表使用这个方法，那么还是通过名称去调用它，而非去描述如何调用它。如`Apple::getWeight`就是调用了`Apple`类的`getWeight()`方法。

| Lambda                                | 等效的方法引用      |
| ------------------------------------- | ------------------- |
| (Apple a)-> a.getWeight()             | Apple::getWeight    |
| (String str, int i)->str.subString(i) | String::subString   |
| (String s)->System.out.println(s)     | System.out::pringln |

> 方法引用的类型

1. 指向静态方法的方法引用，如`Integer.parstInt()`,引用写为：`Integer::parseInt`
2. Lambda表达式子中的类参数的方法，如`(arg0,arg1)->arg0.method();`可以写为：`(arg0,arg1)->ClassName::method`
3. 上文对象的方法：调用一个已经存在的外部对象中的方法，注意前述所定的局部变量使用限制。

> 构造函数引用

构造函数分为有参构造和无参构造，可以使用不同的`Interface`来对应不同的输入参数的构造函数：

`Supplier<Apple> c1=Apple::new`对应于无参构造

`Function<Integer,Apple> c2=Apple::new`对应于有参构造

`BiFunction<String,Integer,Apple> c3=Apple::new`对应于三参数构造

最后使用即：

```java
c1.get()
c2.apply(1);
c3.apply("green",1);
```

此外对于更多数量的有参构造器可以使用自定义的接口来实现。

## 使用sort方法实战

使用Java 8的api `sort`的实战：`void sort(Comparator<? super E> c)`，即需要一个比较器来进行对象的比较排序。这里也是用行为参数化来传递比较策略。

## Lambda表达式的复合使用

### 比较器复合

```java
Comparator<Apple> comparator=(a,b)->a.getColor().compareTo(b.getColor());
Comparator<Apple> comparator1=Comparator.comparing(Apple::getWeight);
Apple.apples.sort(comparator1);
for (Apple a : Apple.apples) {
    System.out.println(a);
}
```

可以在后面加上：

- `reversed()`表示进行逆序
- thenComparing()表示下一个比较对象

```java
Comparator<Apple> comparator2=Comparator.comparing(Apple::getWeight).thenComparing(Apple::getColor).reversed();
```

### 谓词复合

谓词是返回值为`boolean`类型的函数，因此可以考虑进行与或非：

- negate:`redApple.negate()`
- and:`redApple.and(a->a.getWeight()<100)`
- or:`redApple.or(a->a.getColor.equals("green"))`

### 函数复合

Function接口为了将其表示的`Lambda`表达式可以结合起来，提供了两个默认方法：

- andThen:f2(f1)=14

  ```java
  Function<Integer,Integer> f1=(a)->a+2;
  Function<Integer,Integer> f2=(a)->a*2;
  Function<Integer,Integer> f3=f1.andThen(f2);
  System.out.println(f3.apply(5));
  ```

- Compose:f1(f2)=12

  ```java
  Function<Integer,Integer> f1=(a)->a+2;
  Function<Integer,Integer> f2=(a)->a*2;
  Function<Integer,Integer> f4=f1.compose(f2);
  System.out.println(f4.apply(5));
  ```