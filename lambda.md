# lambda表达式

## 1. 常见的函数式接口

java.util.function提供了大量的函数式接口

接口名 | 接口签名
----|-----
Predicate | 接受参数T,返回一个Boolean类型的参数
Consumer | 接口参数T,没有返回值
Function | 接受参数T,返回R对象
Supplier | 不接口任何参数,直接通过get()返回指定类型的对象
UnaryOperator | 接受参数T,执行业务逻辑后,返回更新后的T对象
BinaryOperator | 接口接受两个T对象,执行业务逻辑以后,返回一个T对象


## 2. 方法引用

引用类型 | 语法 | 示例
-----|----|---
1. 静态方法引用 | 类型名称::方法名称 | System.out::println
2. 构造方法应用 | classname::new | Person::new
3. 实例方法引用 | 对象引用::实例方法 | this::get










