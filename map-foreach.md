## Java中遍历Map的几种方法

### **keySet values**

如果只需要map的key或者value，用map的keySet或values方法无疑是最方便的

```java
for (String key : map.keySet()) {
			
			System.out.println(key);
			
}
```

### **keySet get(key)**

如果需要同时获取key和value，可以先获取key,然后再通过map的get(key)获取value

需要说明的是，该方法不是最优选择，一般不推荐使用

```java
for (String key : map.keySet()) {
			
			System.out.println(key);
			System.out.println(map.get(key));
			
		}
```

### **entrySet**

通过对map entrySet的遍历，也可以同时拿到key和value，一般情况下，性能上要优于上一种,这一种也是最常用的遍历方法

```java
for (Map.Entry<String,Object> entry : map.entrySet()) {
			System.out.println(entry.getKey());
			System.out.println(entry.getValue());
		}
```

### **Iterator**

对于上面的几种foreach都可以用Iterator代替，其实foreach在java5中才被支持，foreach的写法看起来更简洁

但Iterator也有其优势：在用foreach遍历map时，如果改变其大小，会报错，但如果只是删除元素，可以使用Iterator的remove方法删除元素

```java
Iterator<Entry<String, Object>> iterator = map.entrySet().iterator();
		
		while (iterator.hasNext()) {
			Map.Entry<String,Object> entry = iterator.next();
			System.out.println(entry.getKey());
			System.out.println(entry.getValue());
			iterator.remove();//删除元素
			
		}
```

### **Lambda**

java8提供了Lambda表达式支持，语法看起来更简洁，可以同时拿到key和value，不过，经测试，性能低于entrySet,所以更推荐用entrySet的方式

```java
map.forEach((key,value)->{
			System.out.println(key);
			System.out.println(value);
		});
```



总结

1. 如果只是获取key，或者value，推荐使用keySet或者values方式
2. 如果同时需要key和value推荐使用entrySet
3. 如果需要在遍历过程中删除元素推荐使用Iterator
4. 如果需要在遍历过程中增加元素，可以新建一个临时map存放新增的元素，等遍历完毕，再把临时map放到原来的map中












