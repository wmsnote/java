# 注解的定义和反射

## 1. jdk中的三个基本注解

* @override:重写父类的方法,覆盖了父类的方法,只限于用在方法上
* @deprecated:类或方法已过时
* @SuppressWarnings:抑制编译器警告


## 2. 自定义注解(形体)

定义注解的属性, 属性类型   属性名() [default 默认值 ]; 

注解类型只能是基本数据类型,String,Class,注解,枚举以及以上类型的一维数组

### 2.1 定义注解:

```java
//注解用@interface声明,注解也是一个类
public @interface Ann1 {
    //定义注解的属性, 属性类型   属性名() [default 默认值 ];
    String name();//注解属性,没有默认值,使用时必须给定值
    int age() default 18;//注解属性有默认值,使用时可以不用给值
    //Date date();//注解类型只能是基本数据类型,String,Class,注解,枚举以及以上类型的一维数组
}

class Demo{
    @Ann1(name="小芳",age=12)
    public void m1(){
    }
}
```

### 2.2 注解的属性是注解类型,注解套注解:

```java
//定义注解
public @interface Param {
    String name() default "";
    String value() default "";
    String description() default "";
}
public @interface Ann2 {
    String URL() default "";
    Param[] params();//注解的属性是注解类型,也可以是一维数组
}
class Demo2{
    @Ann2(params={@Param(name="encoding",value="UTF-8",description="编码集")})
    public void m1(){
    }
}

```

### 2.3 value属性的特殊性

```java
    public @interface Ann3 {
        String name() default "";
        String value() default "";
    }
    class Demo3{
        //value属性比较特殊,给value赋值可以不用指定属性名,默认value属性
        @Ann3("aaa")
        public void m1(){
        }
        @Ann3(name="333",value="aaa")
        public void m2(){
        }
    }
```

### 2.4 一维数组

```java
public @interface Ann4 {
    String[] value() default "";
}
class Demo4{
    @Ann4("asd")//指定一个值
    public void m1(){
    }
    @Ann4(value={"aaa","bbb"})//指定多个值,value可以省略
    public void m2(){
    }
}
```

### 2.5 枚举类型,确定值的取值范围

```java
public @interface EnumAnnotation {
    //类型是枚举类型的,限定其值的取值范围
    OperationType value();
}

enum OperationType{
    add,delete,update,select;
}
class C1{
    @EnumAnnotation(OperationType.add)//赋值,只能是指定枚举类型的值
    public void addTest(){
    }
}

```

## 3. 注解的反射(灵魂)

java.lang.reflect.AnnotatedElement 
  
该接口的子类: 
  
1. Class：表示一个类型 
2. Method：表示一个方法 
3. Field：表示一个字段 
4. Constructor：表示一个构造方法

方法api | 含义
------|---
T getAnnotation(Class annotationType) | 获取指定类型的注解实例的引用 
Annotation[] getAnnotations() | 获取所有的注解，包含继承下来的。 
Annotation[] getDeclaredAnnotations() | 获取自己直接使用的注解，不包含继承下来的。 
boolean isAnnotationPresent(Class<? extends Annotation> annotationType) | 看看指定的注解在不在。

```java
//模拟@Test的原理JunitTest
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyTest {
}
public class ServiceImplTest {
    @MyTest
    public void save(){
        System.out.println("save");
    }
}
public class MyRunner {
    public static void main(String[] args)  {
        // TODO Auto-generated method stub  
        Method[] methods = ServiceImplTest.class.getMethods();
        for (Method method : methods) {
            boolean flg = method.isAnnotationPresent(MyTest.class);
            if(flg){
                method.invoke(ServiceImplTest.class.newInstance());
            }   
        }
    }
}

```

### 3.1 3个元注解(用在注解上的注解)


* @Retention：作用，改变注解的存活范围,RetentionPolicy：SOURCE CLASS  RUNTIME 
* @Target：作用，标识注解应用的位置,ElementType:  TYPE,FIELD,METHOD,PARAMETER,CONSTRUCTOR,LOCAL_VARIABLE,ANNOTATION_TYPE 
* @Inherited:作用, 子类是否继承该注解


### 3.2 模拟JunitTest的测试效率(反射属性值)

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyTest {
    long timeout() default -1;//-1说明不测试执行效率,单位是纳秒
}
public class ServiceImplTest {

    @MyTest(timeout=10)
    public void save(){
        System.out.println("save");
    }
}
public class MyRunner {
    public static void main(String[] args) throws IllegalAccessException, IllegalArgumentException, InvocationTargetException, InstantiationException {
        // TODO Auto-generated method stub
        Class<ServiceImplTest> clazz = ServiceImplTest.class;
        Method[] methods = clazz.getMethods();
        for (Method method : methods) {
            //得到方法上该注解实例的引用
            MyTest myTest = method.getAnnotation(MyTest.class);
            //如果不为null,说明有指定注解
            if(myTest!=null){
                //获取注解属性值
                //timeout如果大于0,说明要测试效率
                long timeout = myTest.timeout();
                if(timeout<0){
                    method.invoke(clazz.newInstance(), null);
                }else{
                    long startTime = System.nanoTime();
                    method.invoke(clazz.newInstance(), null);
                    long endTime = System.nanoTime();
                    if(endTime-startTime>timeout) throw new RuntimeException(method.getName()+"运行超时");
                }   
            }   
        }
    }
}
```

## 4. 注意：学习注解的意义

开发中，要通过XML配置，指挥程序的运行。

* 缺点：开发不直观，麻烦；
* 优点：避免硬编码。 


注解替代XML作为配置用的。

* 优点：只管，开发简便，快速。
* 缺点：硬编码。
