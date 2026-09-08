Controller和RestController(返回JSON)

RestFul API 请求规范
 {
	 和普通传值区别在于，直接在路径加上参数的值，而参数名称、接收参数处理由后端springMVC处理
 }
 
四种方式：GET/PUT/POST/DELETE


Post:表单数据提交
```java
@PostMapping  
public String handlerOfPost(@RequestBody Map<String,String> map){  
    System.out.println(map);  
  
    return "您已调用POST方法到'/api/test'";  
}
```
![[Pasted image 20260521173420.png]]


PUT:
携带REstFul风格参数+请求体
```java
@PutMapping("/{id}")  
public String handlePUT(@PathVariable Integer id,@RequestBody Map<String,String> map){  
    System.out.println("ID: "+ id.toString());  
    System.out.println(map);  
    return "你调用了PUT方法";  
}
```

![[Pasted image 20260521175327.png]]



### 多环境配置：
![[Pasted image 20260521191446.png]]
通过application.yml去指定使用哪份文件，实现开发、上线隔离；

### 三层架构：
Controller Service Mapper


### 数据库：
NaviCat：图形化
Redis：非关系型（key - value形式）
MySQL：关系型，表形式


### Mybatis-Plus：Mybatis增强版
建sb项目：添加Mybatis-Plus、Lombok、数据库驱动依赖
依旧yml中配置数据库连接、mybatis的SQL映射（resource目录下的xml文件）

Spring Boot 启动类中添加 `@MapperScan` 注解，扫描 Mapper 文件夹：
`@MapperScan("com.baomidou.mybatisplus.samples.quickstart.mapper")`


编写实体类：(表明这个类映射数据库中的哪张表)
```java
@Data

@TableName("`user`")

public class User {

private Long id;

private String name;

private Integer age;

private String email;

}
```


编写Mapper接口类：
```java
public interface UserMapper extends BaseMapper<User> {

}
```
实现了BaseMapper`<T>`类，Mybatis自动根据传入的泛型类为该接口创建实现类，并交由Spring容器管理

注入的方法：
1. Autowired
2. Resource
根据类型/名称来找对象

逻辑：Mybatis-Plus在项目启动时根据MapScan注解去找定义的mapper接口，然后创建其实现类放到Spring容器中.


 java测试类中测试注入：
```java
@SpringBootTest
public class SampleTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testSelect() {
        System.out.println(("----- selectAll method test ------"));
        List<User> userList = userMapper.selectList(null);
        Assert.isTrue(5 == userList.size(), "");
        userList.forEach(System.out::println);
    }

}
```


Service层要调用Mapper代码：
在此层定义接口和对应实现类：

Service下的接口继承IService`<T>`，泛型中传入实体类

impl下的实现类通过@Service实现注入，但是如果要一个个去实现方法太繁琐，我们可以继承通用实现类然后泛型中传入<Mapper ,实体类>，就能让Mybatis帮我们实现，并通过@Service注入实例化。

具体实现的方法见Mybatis-Plus官网，我们可以写测试方法：
测试类里直接注入实现类即可调用方法：
![[Pasted image 20260521203101.png]]

![[Pasted image 20260521203112.png]]



便于前端接收处理数据，可以将返回的类型定为统一类型的对象（数据模型），三个处理模块同级下创建common包，定义统一类型返回对象。
结构：
![[Pasted image 20260521205300.png]]



然后controller层正常注入Service实例对象，定义api方法，！注意，要用注解声明这是Controller类型：`@RestController`实现bean注入，`@RequestMapping`注明请求路径。
```java
@RestController
@RequestMapping("/users")
public class UserController {

    @Autowired
    private UserService userService;

    /**
     * 新增用户
     * @param user
     * @return
     */
    @PostMapping
    public Result add(@RequestBody User user) {
        userService.save(user);
        return Result.success();
    }

    /**
     * 查询单个用户
     * @param id
     * @return
     */
    @GetMapping("/{id}")
    public Result getOne(@PathVariable Long id) {
        return Result.success(userService.getById(id));
    }

    /**
     * 查询所有用户
     * @return
     */
    @GetMapping
    public Result list() {
        return Result.success(userService.list());
    }

    /**
     * 更新用户
     * @param user
     * @return
     */
    @PutMapping
    public Result update(@RequestBody User user) {
        userService.updateById(user);
        return Result.success();
    }

    /**
     * 删除单个用户
     * @param id
     * @return
     */
    @DeleteMapping("/{id}")
    public Result delete(@PathVariable Long id) {
        userService.removeById(id);
        return Result.success();
    }
}
```



### 实现分页：
```java
@GetMapping("/page")  
public Result findPage(@RequestParam(defaultValue = "1") Integer pageNum, @RequestParam(defaultValue = "10") Integer pageSize){  
    return Result.success(  
            userService.page(new Page<>(pageNum,pageSize))  
    );
```
这里使用传统通信方式接收参数，再借助实现类的page方法，传Page对象，用Result包裹统一返回

MybatisPlus需要引入两个依赖才能实现:`mybatis-plus-bom`和`mybatis-plus-jsqlparser`, 并创建一个config包用到下面的两个注解标明此为配置类、将启动类的`mapperScan`注解移到这里。

第三种注入对象方式：
`@configuration`表明配置类
`@Bean`，标记方法，将方法返回的对象注入到Bean中

### 自动填充：
```java
//    自动填充实现：  
    @TableField(value = "create_time",fill = FieldFill.INSERT)  
    private LocalDateTime createTime;
```
`@TableField`表映射表中的哪个字段，fill值是自动填充时机。
然后我们需要创建handler类来实现填充方法。
官网：
```md
自动填充功能通过实现 `com.baomidou.mybatisplus.core.handlers.MetaObjectHandler` 接口来实现。你需要创建一个类来实现这个接口，并在其中定义插入和更新时的填充逻辑。
```
例：创建MyMetaObjectHandler类，并实现插入、更新方法
```java
    @Override  
    public void updateFill(MetaObject metaObject) {  
        log.info("开始更新填充...");  
//        使用示例：填充的字段名，填充的值  
//        this.strictInsertFill(metaObject, "updateUserId", Long.class, 123456L)  
        this.strictUpdateFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());  
    }
```
`@JsonFormat(pattern = "yyyy-mm-dd HH:mm:ss")`
时间类型数据可通过此注解规范格式

**第四种往springboot容器中注入对象方式：** `@Component`.


### 条件构造器
就是实现条件查询的方法：
使用举例：通过LambdaQueryWrpper类
```java
@GetMapping("/page")  
public Result findPage(@RequestParam(defaultValue = "1") Integer pageNum,  
                       @RequestParam(defaultValue = "10") Integer pageSize,  
                       @RequestParam(defaultValue = "") String name  
){  
    LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();  
    if (!"".equals(name)){  
        queryWrapper.like(User::getName,name);  
    }  
    return Result.success(  
            userService.page(new Page<>(pageNum,pageSize).queryWrapper)
```



### 结合Redis：
把redis的操作类对象注入到容器中，
需要先配置相关依赖：`spring-boot-starter-data-redis`,Redis连接池依赖：`commons-pools`

yml中配置Redis连接信息：（spring的下一级data对象）
![[Pasted image 20260522173821.png]]

创建Redis配置类`RedisConfig`，注入Redis连接池对象、重新配置Redis操作对象。
后续可基于此操作对象封装RedisUtil工具类。

### 请求拦截器框架：Sa-Token
用于判断请求路径是否白名单、是否对用户请求放行等等
- pom中配依赖：`sa-token-spring-boot3-starter`
- yml中进行配置：Sa-Token相关配置项:
	 token 名称（同时也是 cookie 名称）  
	token-name: Authorization

### Sa-Token集成Redis：
```
Sa-Token 默认将数据保存在内存中:
1. 重启后数据会丢失。
2. 无法在分布式环境中共享数据。

为此，Sa-Token 提供了扩展接口，你可以轻松将会话数据存储在一些专业的缓存中间件上（比如 Redis）， 做到重启数据不丢失，而且保证分布式环境下多节点的会话一致性。
```

此时并未实现前后端分离，引入Redis实现认证信息持久化：
配置依赖：
1. ```
   <!-- Sa-Token 整合 RedisTemplate -->:
   sa-token-redis-template
   ```
2. ```
   <!-- 提供 Redis 连接池 -->
   commons-pool2
   ```


### 实现跨域：创建CosConfig配置类,进行配置
实现效果：后端返回的JSON格式数据中，鉴权token被设置为名为 Authorization的值
，前端就能拿到这个并利用Axios在每次发请求时在请求头携带上这对KEY-VALUE。



### 全局过滤器：
通过SaTokenConfigure设置过滤器类，然后将其对象`@Bean`注入到容器。

前后端分离结构时需要定制化返回数据，将返回结果转成JSON格式，引入Hutool的JSONUtil工具类：
```md
Hutool是一个小而全的Java工具类库，通过静态方法封装
```

如重写异常处理函数，返回JSON格式数据：
```java
// 异常处理函数：每次认证函数发生异常时执行此函数  
.setError(e -> {  
    // 设置响应头  
    SaHolder.getResponse().setHeader("Content-Type", "application/json;charset=UTF-8");  
    // 使用封装的 JSON 工具类转换数据格式  
    return JSONUtil.toJsonStr( SaResult.error(e.getMessage()) );  
})
```


### Sa-Token实现权限认证：
RBAC模型，给用户标记所拥有的权限存入数据库，然后定义请求路径需要的访问权限，用户访问时从数据库查询用户所拥有的权限是否匹配当前所需权限，匹配：放行，反之不放行。

实现:需要实现 `StpInterface`接口，告诉框架指定账号拥有的权限码集合是哪些

### 注解鉴权：
`注解鉴权 —— 优雅的将鉴权与业务代码分离！`
写在方法上，只有拥有注解中所定义权限的用户才会被放行：
如：` @SaCheckPermission("user:add")`: 权限校验 —— 必须具有指定权限才能进入该方法。`

前提:`为了使用注解鉴权，你必须手动将 Sa-Token 的全局拦截器注册到你项目中。`

然后即可使用注解鉴权


### 全局异常处理：
后端抛出异常返回前端（JSON），创建全局异常处理类注入Spring容器。

全局异常处理类用`@RestControlAdvice`注解，里面编写要对异常进行处理的方法，用`ExceptionHandle`注解标识，注解值写上异常类的反射对象，标明是针对哪种异常的处理：
```java
package com.lancer.mybatisplus.handler;  
  
import cn.dev33.satoken.context.SaHolder;  
import cn.dev33.satoken.exception.NotPermissionException;  
import com.lancer.mybatisplus.common.Result;  
import org.springframework.web.bind.annotation.ExceptionHandler;  
  
/**  
 * @RestControllerAdvice */public class GlobalExceptionHandler {  
    /**  
     * 没有权限拦截方法  
     * @param e  
     * @return  
     */    @ExceptionHandler(NotPermissionException.class)  
    public Result notPermissionException(NotPermissionException e) {  
        SaHolder.getResponse().setStatus(403);  
        return Result.error(e.getMessage());  
    }  
}
```


