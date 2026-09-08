# IOC：
要想把类交由IOC容器管理，由其控制对象的创建与管理，须在类上加上注解：
![[Pasted image 20260423163746.png]]

图示：
![[Pasted image 20260423155818.png]]
可以为bean添加名称：
![[Pasted image 20260423164325.png]]
**Bean 本质上是由 Spring 创建、保存、装配、管理生命周期的 Java 对象。**

Bean若想生效，还需被扫描：
![[Pasted image 20260423164717.png]]

注意点：
![[Pasted image 20260423164841.png]]


# DI详解
常见三种注入：
![[Pasted image 20260423170131.png]]

常用方式一，
方式二、三：
![[Pasted image 20260423165717.png]]

当程序运行时，容器会自动构建UserController对象，并传入所需的bean对象。


### 存在多个bean时，如不处理会报错：
![[Pasted image 20260423171724.png]]

两种方案，一种对于IOC，另一种对于注入时的操作；

方案一，交由IOC时，在将实现类交给IOC容器时注明优先加载：@Primary

方案二：注入时，
@Autowired配合@Qualifier注解 || 直接单用@Resource
![[Pasted image 20260423172007.png]]

