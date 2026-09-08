![[Pasted image 20260424110729.png]]


应用属性文件中配置mybatis的log为Std模式：
![[Pasted image 20260424194411.png]]

JDBC与MyBatis对比：
![[Pasted image 20260424195023.png]]
配置文件解决硬编码；

Mapper解决繁琐，mybatis底层自动帮我们处理结果集并封装到方法返回值。

解决资源浪费：跟线程池类似，Mybatis底层维护了一个数据库连接池，减少资源浪费，连接用完并不会里面关闭，而是返回连接池中等待调用。


## 数据库连接池
![[Pasted image 20260424195959.png]]

### 切换数据库连接池：
1.导入新的数据库连接池依赖
2.属性文件中指定datasourcetype
![[Pasted image 20260424201219.png]]

### 测试Mybatis的Del方法时版本兼容问题：
### 问题及解决过程总结

1. **最初使用 IDEA 运行测试时报 JUnit 相关错误**  
    项目在使用 IDEA 绿三角运行测试方法时，出现了 `NoSuchMethodError`，错误信息指向 `JUnit5TestRunnerUtil` 和 `MethodSelector.getMethodParameterTypes()`。经过分析发现，项目使用的是 Spring Boot 4.x，对应引入了 JUnit 6.x，而当前 IDEA 版本的内置 JUnit 运行器对 JUnit 6 支持不完全，导致测试运行器与项目中的 JUnit 版本不兼容。
2. **将 Spring Boot 版本从 4.x 降级到 3.4.5**  
    为了避免使用过新的 Spring Boot 4.x 依赖，改用更稳定、兼容性更好的 Spring Boot 3.4.5。降级后，项目中的 JUnit 版本也随之回退到 JUnit 5.x，解决了 IDEA 运行测试时 JUnit 版本不兼容的问题。
3. **降级后又出现 Bean 重复注册问题**  
    在 Spring Boot 降级到 3.4.5 后，项目启动测试时又出现 `hikariPoolDataSourceMetadataProvider` Bean 重复定义的问题。错误提示说明 Spring 容器中已经存在同名 Bean，无法再次注册，导致 ApplicationContext 启动失败。
4. **定位到 MyBatis Starter 版本未同步降级**  
    进一步检查依赖后发现，虽然 Spring Boot 已经降级到 3.4.5，但 `mybatis-spring-boot-starter` 仍然使用的是 4.0.1。该版本会引入部分 Spring Boot 4.x 相关自动配置依赖，导致项目中同时存在 Spring Boot 3.x 和 Spring Boot 4.x 的自动配置类，从而引发 Hikari 数据源相关 Bean 的重复注册。
5. **将 MyBatis Starter 同步降级到 3.x**  
    为了保证依赖版本一致，将 `mybatis-spring-boot-starter` 从 4.0.1 降级为 3.0.4，同时删除或同步降级 `mybatis-spring-boot-starter-test`。这样项目中的 Spring Boot、MyBatis Starter、JUnit 等依赖版本保持在同一兼容体系内，避免了 Spring Boot 3.x 与 4.x 混用造成的冲突。
6. **最终处理结果**  
    经过版本调整后，项目依赖从原来的 Spring Boot 4.x + MyBatis Starter 4.x，调整为 Spring Boot 3.4.5 + MyBatis Starter 3.0.4，并统一使用 JDK 17。这样既解决了 IDEA 测试运行器与 JUnit 6 不兼容的问题，也解决了 Spring Boot 自动配置类重复加载导致的 Bean 冲突问题。



### Mybatis删除语句：
![[Pasted image 20260424203548.png]]

`#{}` 占位符，最终会变成 ？，即sql语句会变成预编译sql，最后sql语句和参数一起被携带给MySQL执行操作。


###  # / $区别：
![[Pasted image 20260424203913.png]]

### 插入语句：
传入多个参数时，可以把参数封装在对象中，然后在@Insert注解`#{}`占位时写上对象属性名.
![[Pasted image 20260425101615.png]]
```
@Test  
public void testInsert(){  
    //是定义接口方法的时候规定返回值，这个返回值在执行接口实现类方法时可以拿到。  
    User user = new User(null,"xiaohua","123456","huahua",18);  
    Integer i = userMapper.forInsert(user);  
    System.out.println("受影响行数 "+i);  
}
```

```
@Update("update user set username = #{username},password = #{password},name = #{name},age = #{age} where id = #{id}")  
public void updateByID(User user);
```

```
@Test  
public void testUpdate(){  
    User user = new User(1,"xiaoming","1288772","HUawei",20);  
    userMapper.updateByID(user);  
}
```


### 条件查询：
![[Pasted image 20260425104243.png]]

正常情况下（官方Springboot项目除外），接口方法在编译成字节码文件时不会保留形参名称，只保留形参类型，所以需要为形参指定名称（@Param）；
- 单个形参就直接对应；
- 传入的是对象时写的时对象属性，与形参名无关。

这里犯错了：凡是查询都要在Mapper方法指定返回实体类，以便Mybatis将查询结果封装到对象中。
> 你遇到的错误是 MyBatis 映射查询结果时找不到合适的类型来接收数据，核心原因在于 `selectByCondition` 的**返回值类型**被定义成了 `void`。
> 
> ### 错误关键信息
> 
> text
> 
> No constructor found in void matching [java.lang.Long, java.lang.String, java.lang.String, java.lang.String, java.lang.Integer]
> 
> - `void` 表示 MyBatis 无法确定要把查询结果封装到什么对象里（因为方法声明返回 `void`）。
>     
> - `matching [...]` 表示实际查询到的列类型是 `Long, String, String, String, Integer`。
>     
> - 因为 `void` 没有构造函数，所以直接抛异常。

```
@Select("select * from user where username = #{username} and password = #{password}")  
 public User selectByCondition(@Param("username") String username, @Param("password") String password);
```

- 添加返回值为实体类后测试通过： ![[Pasted image 20260425105601.png]]



### XML配置文件：
Mybatis中SQL语句既可以写在注解中，也可以写在对应的XML文件中，规则如下：
![[Pasted image 20260425114219.png]]


```
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE mapper  
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">  
<!--头部约束，官网寻找-->  
  
<!--根标签-->  
<mapper namespace="com.lancer.Mapper.UserMapper">   
<!--  namespace值是接口的完全引用  -->  
    <!--    其中用SQL关键字的标签,对应接口中定义的方法，来包裹sql语句-->  
    <select id="getAll" resultType="com.lancer.pojo.User">  
# id与方法名对应，表示将sql语句映射到对应接口方法，  
# resultType表示将查询返回的单条记录封装的类型，注意要写完整引用，否则报错！  
            select * from user  
    </select>  

</mapper>
```

#### Mybatis辅助配置：
若没有遵循同包同名规范放置xml文件，需在属性文件中配置指定xml映射文件的位置：
![[Pasted image 20260425114714.png]]