# 反射-提高扩展性

反射的对象: **Class** **Field** **Constractor** 

## 1. 创建字节码文件(.class)对象的3种方法

### 1. 任何类型都有静态字段class,返回该类型的字节码文件对象

```java
Class clazz = Person.class;

```

### 2. 任何类都有getClass()方法,定义在Object类中,通过对象调用getClass()方法,返回该对象的字节码文件对象

```java
Person person = new Person();
Class clazz = person.getClass();

```

### 3. 类Class静态方法forName()返回类名字节码文件对象

```java
Class clazz = Class.forName("com.commom.Person");

```

注意: 字符串必须为包名+类名,否则 classNotFindException;

## 2. 创建实例

### 1. 调用空参的构造方法获取实例

```java
Object object = clazz.newInstance();

```

### 2. 调用带参的构造方法获取实例

```java
Constractor constractor = clazz.getDeclaredConstractor(String.class,int.class);//注意: 通过指定类型,获取对应类型的构造器对象
Object object = constractor.newInstance("wms",30);//构造器对象传入实参,创建对象

```

### 3. 获取字段

```java
Field name = clazz. getDeclaredField ( "name") ;//通过字段名,获取指定字段
Field [] fields = clazz .getFields () ;//获取所有字段

```

```log
AccessiableObject
     |--Method
     |--Field
     |--Constractor

```

**注意:**

> clazz.getDeclaredXXX : 获取全部成员,忽略权限修饰,暴力获取clazz.getXXX获取公共(public)成员
> 成员对象.setAccessiable(true) : 忽略权限修饰,暴力访问

### 4. 获取方法

```java
Method method = clazz.getDeclaredMethod(String.class,int.class);//获取指定方法
Method [] declaredMethods = clazz . getDeclaredMethods() ;//获取所有方法
method.invoke(object,"wms",30);//调用一般方法,返回的是该方法的返回值,若为静态,object为null

```



