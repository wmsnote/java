---
title: bean转化
tags: java
notebook: java
---

## 基于充血模式

```java
import java.util.function.Supplier;

import org.springframework.beans.BeanUtils;

public interface BaseBean {
	public default <T> T convert(Supplier<T> supplier) {
		T t = supplier.get();
		BeanUtils.copyProperties(this, t);
		return t;
	}
}
```

```java
import java.util.function.Function;
import java.util.function.Supplier;

import org.springframework.beans.BeanUtils;

import lombok.AllArgsConstructor;
import lombok.Data;

@Data
@AllArgsConstructor
public class UserVo implements BaseBean {
	
	private String username;
	
	private Integer password;
	
//	public <T> T convert(Supplier<T> supplier) {
//		T t = supplier.get();
//		BeanUtils.copyProperties(this, t);
//		return t;
//		
//	}
	
//	public <T> T convert(Function<UserVo, T> function) {
//		return function.apply(this);
//		
//	}
	
//	public <T> T convert(Convent<T, UserVo> con) {
//		return con.apply(this);
//		
//	}
	
	
//	public <T> T convert(UserVo userVo, Convent<T, UserVo> con) {
//		return con.apply(userVo);
//		
//	}

	


}

```

```java
@Data
public class UserBo {
	
	private String username;
	
	private Integer password;
	
	

}
```

```java
public class DemoApplication {

	public static void main(String[] args) {
		UserVo userVo = new UserVo("wms", 123456);
		
		UserBo userBo = userVo.convert(UserBo::new);
		
//		UserBo userBo = userVo.convert((s)->{
//			UserBo uBo = new UserBo();
//			BeanUtils.copyProperties(s, uBo);
//			return uBo;
//		});
		System.out.println(userBo);
		
	}

}
```

bean 使我们使用最多的模型之一，我将以大篇幅去讲解 bean，希望读者好好体会。

## domain 包名

根据很多 Java 程序员的”经验”来看，一个数据库表则对应着一个 domain 对象，所以很多程序员在写代码时，包名则使用：`com.xxx.domain` ，这样写好像已经成为了行业的一种约束，数据库映射对象就应该是 domain。但是你错了，domain 是一个领域对象，往往我们再做传统 Java 软件 Web 开发中，这些 domain 都是**贫血模型**，是没有行为的，或是没有足够的领域模型的行为的，所以，以这个理论来讲，这些 domain 都应该是一个普通的 entity 对象，并非领域对象，所以请把包名改为:`com.xxx.entity`。

如果你还不理解我说的话，请看一下 Vaughn Vernon 出的一本叫做《IMPLEMENTING DOMAIN-DRIVEN DESIGN》(实现领域驱动设计)这本书，书中讲解了贫血模型与领域模型的区别，相信你会受益匪浅。

## DTO

数据传输我们应该使用 DTO 对象作为传输对象，这是我们所约定的，因为很长时间我一直都在做移动端 API 设计的工作，有很多人告诉我，他们认为只有给手机端传输数据的时候(input or output)，这些对象成为 DTO 对象。请注意！这种理解是错误的，只要是用于网络传输的对象，我们都认为他们可以当做是 DTO 对象，比如电商平台中，用户进行下单，下单后的数据，订单会发到 OMS 或者 ERP 系统，这些对接的返回值以及入参也叫 DTO 对象。

我们约定某对象如果是 DTO 对象，就将名称改为 `XXDTO`，比如订单下发OMS：`OMSOrderInputDTO`。

## DTO 转化

正如我们所知，DTO 为系统与外界交互的模型对象，那么肯定会有一个步骤是将 DTO 对象转化为 BO 对象或者是普通的 entity 对象，让 service 层去处理。

### 场景

比如添加会员操作，由于用于演示，我只考虑用户的一些简单数据，当后台管理员点击添加用户时，只需要传过来用户的姓名和年龄就可以了，后端接受到数据后，将添加创建时间和更新时间和默认密码三个字段，然后保存数据库。

```java
@RequestMapping("/v1/api/user")
@RestController
public class UserApi {

    @Autowired
    private UserService userService;

    @PostMapping
    public User addUser(UserInputDTO userInputDTO){
        User user = new User();
        user.setUsername(userInputDTO.getUsername());
        user.setAge(userInputDTO.getAge());

        return userService.addUser(user);
    }
}

```

我们只关注一下上述代码中的转化代码，其他内容请忽略：

```java
User user = new User();
user.setUsername(userInputDTO.getUsername());
user.setAge(userInputDTO.getAge());
```

### 请使用工具

上边的代码，从逻辑上讲，是没有问题的，只是这种写法让我很厌烦，例子中只有两个字段，如果有 20 个字段，我们要如何做呢？ 一个一个进行 set 数据吗？当然，如果你这么做了，肯定不会有什么问题，但是，这肯定不是一个最优的做法。

网上有很多工具，支持浅拷贝或深拷贝的 Utils。举个例子，我们可以使用 `org.springframework.beans.BeanUtils#copyProperties` 对代码进行重构和优化：

```java
@PostMapping
public User addUser(UserInputDTO userInputDTO){
    User user = new User();
    BeanUtils.copyProperties(userInputDTO,user);

    return userService.addUser(user);
}
```

`BeanUtils.copyProperties` 是一个浅拷贝方法，复制属性时，我们只需要把 DTO 对象和要转化的对象两个的属性值设置为一样的名称，并且保证一样的类型就可以了。如果你在做 DTO 转化的时候一直使用 set 进行属性赋值，那么请尝试这种方式简化代码，让代码更加清晰!

### 转化的语义

上边的转化过程，读者看后肯定觉得优雅很多，但是我们再写 Java 代码时，更多的需要考虑语义的操作，再看上边的代码：

```java
User user = new User();
BeanUtils.copyProperties(userInputDTO,user);
```

虽然这段代码很好的简化和优化了代码，但是他的语义是有问题的，我们需要提现一个转化过程才好，所以代码改成如下：

```java
@PostMapping
 public User addUser(UserInputDTO userInputDTO){
         User user = convertFor(userInputDTO);

         return userService.addUser(user);
 }

 private User convertFor(UserInputDTO userInputDTO){

         User user = new User();
         BeanUtils.copyProperties(userInputDTO,user);
         return user;
 }
```

这是一个更好的语义写法，虽然他麻烦了些，但是可读性大大增加了，在写代码时，我们应该尽量把语义层次差不多的放到一个方法中，比如：

```java
User user = convertFor(userInputDTO);
return userService.addUser(user);
```

这两段代码都没有暴露实现，都是在讲如何在同一个方法中，做一组相同层次的语义操作，而不是暴露具体的实现。

如上所述，是一种重构方式，读者可以参考 Martin Fowler 的《Refactoring Imporving the Design of Existing Code》(重构 改善既有代码的设计) 这本书中的 Extract Method 重构方式。

## 抽象接口定义

当实际工作中，完成了几个 API 的 DTO 转化时，我们会发现，这样的操作有很多很多，那么应该定义好一个接口，让所有这样的操作都有规则的进行。

如果接口被定义以后，那么 convertFor 这个方法的语义将产生变化，它将是一个实现类。

看一下抽象后的接口：

```java
public interface DTOConvert<S,T> {
    T convert(S s);
}
```

虽然这个接口很简单，但是这里告诉我们一个事情，要去使用泛型，**如果你是一个优秀的 Java 程序员，请为你想做的抽象接口，做好泛型吧**。

我们再来看接口实现：

```java
public class UserInputDTOConvert implements DTOConvert {
    @Override
    public User convert(UserInputDTO userInputDTO) {
        User user = new User();
        BeanUtils.copyProperties(userInputDTO,user);
        return user;
    }
}
```

我们这样重构后，我们发现现在的代码是如此的简洁，并且那么的规范：

```java
@RequestMapping("/v1/api/user")
@RestController
public class UserApi {

    @Autowired
    private UserService userService;

    @PostMapping
    public User addUser(UserInputDTO userInputDTO){
        User user = new UserInputDTOConvert().convert(userInputDTO);

        return userService.addUser(user);
    }
}
```

### review code

如果你是一个优秀的 Java 程序员，我相信你应该和我一样，已经数次重复 review 过自己的代码很多次了。

我们再看这个保存用户的例子，你将发现，API 中返回值是有些问题的，问题就在于不应该直接返回 User 实体，因为如果这样的话，就暴露了太多实体相关的信息，这样的返回值是不安全的，所以我们更应该返回一个 DTO 对象，我们可称它为 UserOutputDTO：

```java
@PostMapping
public UserOutputDTO addUser(UserInputDTO userInputDTO){
        User user = new UserInputDTOConvert().convert(userInputDTO);
        User saveUserResult = userService.addUser(user);
        UserOutputDTO result = new UserOutDTOConvert().convertToUser(saveUserResult);
        return result;
}
```

这样你的 API 才更健全。

不知道在看完这段代码之后，读者有是否发现还有其他问题的存在，作为一个优秀的 Java 程序员，请看一下这段我们刚刚抽象完的代码:

```java
User user = new UserInputDTOConvert().convert(userInputDTO);
```

你会发现，new 这样一个 DTO 转化对象是没有必要的，而且每一个转化对象都是由在遇到 DTO 转化的时候才会出现，那我们应该考虑一下，是否可以将这个类和 DTO 进行聚合呢，看一下我的聚合结果:

```java
public class UserInputDTO {
private String username;
private int age;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }


    public User convertToUser(){
        UserInputDTOConvert userInputDTOConvert = new UserInputDTOConvert();
        User convert = userInputDTOConvert.convert(this);
        return convert;
    }

    private static class UserInputDTOConvert implements DTOConvert<UserInputDTO,User> {
        @Override
        public User convert(UserInputDTO userInputDTO) {
            User user = new User();
            BeanUtils.copyProperties(userInputDTO,user);
            return user;
        }
    }

}
```

然后 API 中的转化则由：

```java
User user = new UserInputDTOConvert().convert(userInputDTO);
User saveUserResult = userService.addUser(user);
```

变成了：

```java
User user = userInputDTO.convertToUser();
User saveUserResult = userService.addUser(user);
```

我们再 DTO 对象中添加了转化的行为，我相信这样的操作可以让代码的可读性变得更强，并且是符合语义的。

### 再查工具类

再来看 DTO 内部转化的代码，它实现了我们自己定义的 DTOConvert 接口，但是这样真的就没有问题，不需要再思考了吗？

我觉得并不是，对于 Convert 这种转化语义来讲，很多工具类中都有这样的定义，这中 Convert 并不是业务级别上的接口定义，它只是用于普通 bean 之间转化属性值的普通意义上的接口定义，所以我们应该更多的去读其他含有 Convert 转化语义的代码。

我仔细阅读了一下 GUAVA 的源码，发现了 `com.google.common.base.Convert` 这样的定义：

```java
public abstract class Converter<A, B> implements Function<A, B> {
    protected abstract B doForward(A a);
    protected abstract A doBackward(B b);
    //其他略
}
```

从源码可以了解到，GUAVA 中的 Convert 可以完成正向转化和逆向转化，继续修改我们 DTO 中转化的这段代码：

```java
private static class UserInputDTOConvert implements DTOConvert<UserInputDTO,User> {
        @Override
        public User convert(UserInputDTO userInputDTO) {
                User user = new User();
                BeanUtils.copyProperties(userInputDTO,user);
                return user;
        }
}

```

修改后：

```java
private static class UserInputDTOConvert extends Converter<UserInputDTO, User> {
         @Override
         protected User doForward(UserInputDTO userInputDTO) {
                 User user = new User();
                 BeanUtils.copyProperties(userInputDTO,user);
                 return user;
         }

         @Override
         protected UserInputDTO doBackward(User user) {
                 UserInputDTO userInputDTO = new UserInputDTO();
                 BeanUtils.copyProperties(user,userInputDTO);
                 return userInputDTO;
         }
 }
```

看了这部分代码以后，你可能会问，那逆向转化会有什么用呢？其实我们有很多小的业务需求中，入参和出参是一样的，那么我们变可以轻松的进行转化，我将上边所提到的 UserInputDTO 和 UserOutputDTO 都转成 UserDTO 展示给大家。

DTO：

```java
public class UserDTO {
    private String username;
    private int age;

    public String getUsername() {
            return username;
    }

    public void setUsername(String username) {
            this.username = username;
    }

    public int getAge() {
            return age;
    }

    public void setAge(int age) {
            this.age = age;
    }


    public User convertToUser(){
            UserDTOConvert userDTOConvert = new UserDTOConvert();
            User convert = userDTOConvert.convert(this);
            return convert;
    }

    public UserDTO convertFor(User user){
            UserDTOConvert userDTOConvert = new UserDTOConvert();
            UserDTO convert = userDTOConvert.reverse().convert(user);
            return convert;
    }

    private static class UserDTOConvert extends Converter<UserDTO, User> {
            @Override
            protected User doForward(UserDTO userDTO) {
                    User user = new User();
                    BeanUtils.copyProperties(userDTO,user);
                    return user;
            }

            @Override
            protected UserDTO doBackward(User user) {
                    UserDTO userDTO = new UserDTO();
                    BeanUtils.copyProperties(user,userDTO);
                    return userDTO;
            }
    }

}
```

API：

```java
@PostMapping
 public UserDTO addUser(UserDTO userDTO){
         User user =  userDTO.convertToUser();
         User saveResultUser = userService.addUser(user);
         UserDTO result = userDTO.convertFor(saveResultUser);
         return result;
 }

```

当然，上述只是表明了转化方向的正向或逆向，很多业务需求的出参和入参的 DTO 对象是不同的，那么你需要更明显的告诉程序：逆向是无法调用的：

```java
private static class UserDTOConvert extends Converter<UserDTO, User> {
         @Override
         protected User doForward(UserDTO userDTO) {
                 User user = new User();
                 BeanUtils.copyProperties(userDTO,user);
                 return user;
         }

         @Override
         protected UserDTO doBackward(User user) {
                 throw new AssertionError("不支持逆向转化方法!");
         }
 }

```

看一下 doBackward 方法，直接抛出了一个断言异常，而不是业务异常，这段代码告诉代码的调用者，这个方法不是准你调用的，如果你调用，我就”断言”你调用错误了。
