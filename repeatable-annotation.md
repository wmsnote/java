# java 8新特性--注解的变化及类型推断

Java 8在两个方面对注解机制进行了改进，分别为:

1. 你现在可以定义重复注解
2. 你可以为任何目标添加注解

Java中的注解是一种对程序元素进行配置，提供附加信息的机制。


## 2. 重复注解

之前版本的Java禁止对同样的注解类型声明多次。由于这个原因，下面的第二句代码是无效的。

```java
@interface 
Author {
    String name(); 
}
@Author(name="Raoul") @Author(name="Mario") @Author(name="Alan") 
classBook{ }
```

Java程序员经常通过一些惯用法绕过这一限制。例如可以声明一个新的注解，它包含了你希望重复的注解数组。这种方法的形式如下:

```java

@interface Author {    
    String name(); 
}

@interface Authors {    
    Author[]value(); 
}

@Authors( {@Author(name="Raoul"), @Author(name="Mario") , @Author(name="Alan") } )
class Book{}

```

Book类的嵌套注解相当难看。这就是Java 8想要从根本上移除这一限制的原因，去掉这一限制后，代码的可读性会好很多。

现在，如果你的配置允许重复注解，你可以毫无顾虑地一次声明 多个同一种类型的注解。它目前还不是默认行为，你需要显式地要求进行重复注解。
创建一个重复注解

如果一个注解在设计之初就是可重复的，你可以直接使用它。但是，如果你提供的注解是为用户提供的，那么就需要做一些工作，说明该注解可以重复。下面是你需要执行的两个步骤:

```java
@Repeatable(Authors.class) 
@interface Author {    
    String name();
} 
@interface
Authors {    
    Author[] value(); 
}
```

完成了这样的定义之后，Book类可以通过多个@Author注解进行注释，如下所示:

```java
@Author(name="Raoul") @Author(name="Mario") @Author(name="Alan") 
class Book{ }
```

编译时，Book会被认为使用了

```java
@Authors({@Author(name="Raoul"), @Author(name =”Mario”), @Author(name=”Alan”)})
```

这样的形式进行了注解，所以，你可以把这种新的机 制看成是一种语法糖，它提供了Java程序员之前利用的惯用法类似的功能。为了确保与反射方法 在行为上的一致性，注解会被封装到一个容器中。Java API中的`getAnnotation(Class annotation-Class)`方法会为注解元素返回类型为T的注解。如果实际情况有多个类型为T的注解，该方法的返回到底是哪一个呢?
类Class提供了一个新的`getAnnotationsByType` 方法，它可以帮助我们更好地使用重复注解。比如，你可以像下面这样打印输出Book类的所有Author注解:

```java
public static void main(String[] args) { 
    Author[] authors = Book.class.getAnnotationsByType(Author.class); //java8提供的循环及lambda表达式
    Arrays.asList(authors).forEach(a -> {    System.out.println(a.name()); } ); 
}
```

## 3. 类型注解

从Java 8开始，注解已经能应用于任何目标。这其中包括new操作符、类型转换、instanceof检查、泛型类型参数，以及implements和throws子句。

这里，我们举了一个例子，这个例子中 类型为String的变量name不能为空，所以我们使用了@NonNull对其进行注解:

```java
@NonNull String name = person.getName();
```

类似地，你可以对列表中的元素类型进行注解:

```java
List<@NonNull Car> cars = new ArrayList<>(); 
```

利用好对类型的注解非常有利于我们对程序进行分析。这两个例子中，通过这一工具我们可以确保getName不返回空，cars列表中的元素总是非空值。这会 极大地帮助你减少代码中不期而至的错误。
Java 8并未提供官方的注解或者一种工具能以开箱即用的方式使用它们。它仅仅提供了一种功能，你使用它可以对不同的类型添加注解。

## 3. 泛型类型推断

Java 8对泛型参数的推断进行了增强。相信你对Java 8之前版本中的类型推断已经比较熟了。

比如，Java中的方法emptyList方法定义如下: 

```java
static <T> List<T> emptyList();
```

emptyList方法使用了类型参数T进行参数化。你可以像下面这样为该类型参数提供一个显式的类型进行函数调用:

```java
List<Car> cars = Collections.<Car>emptyList();
```

不过Java也可以推断泛型参数的类型。上面的代码和下面这段代码是等价的: 

```java
List<Car> cars = Collections.emptyList();
```

Java 8出现之前，这种推断机制依赖于程序的上下文(即目标类型)，具有一定的局限性。
Java 8中，目标类型包括向方法传递的参数，因此你不再需要提供显式的泛型参数

