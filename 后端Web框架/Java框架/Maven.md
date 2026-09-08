用于管理、构建Java项目的工具
![[Pasted image 20260416190149.png]]
Maven把项目也视作对象，构建项目的各个周期通过插件完成，每个阶段又产生一定临时文件，存放在target文件下。

### Maven的结构：
pom.xml配置文件中包含了POM（描述当前项目信息），Dependency管理项目第三方依赖

jar包查找顺序：
- （不含私服）
![[Pasted image 20260416190847.png]]

- 含私服时影响jar包查找顺序：不过核心都是最终下载到本地仓库然后关联
![[Pasted image 20260416191034.png]]


## Maven作用
1.依赖管理：
![[Pasted image 20260416153402.png]]
无需手动下载jar包并复制到项目的lib文件夹下，
只需在maven配置文件描述要用到的依赖信息，
其自动下载并导入

实例：
用maven构建java项目：
![[Pasted image 20260416153724.png]]
在Dependencise标签下：用Dependency标签写明每一个依赖的jar包信息
实践：
![[Pasted image 20260416155043.png]]
看清别写进properties里了。


2.项目构建标准流程
![[Pasted image 20260416155230.png]]
直接用其提供的指令，快速实现每个项目流程操作,

直接双击生存期中的指令就可完成对应操作，且是跨平台的。
![[Pasted image 20260416155444.png]]

指令的执行又依赖于插件，
执行生命周期的底层就是去调用对应插件完成相应项目构建操作：
![[Pasted image 20260416204246.png]]
*故maven底层更像是一个插件执行框架*


3.统一项目结构：
![[Pasted image 20260416155626.png]]

不同构建工具可能构建的java项目结构不一，
基于maven构建的java项目，结构是统一的

![[Pasted image 20260416155822.png]]
resources：配置文件




### Maven仓库：
![[Pasted image 20260416191337.png]]

#### 安装Maven并配置本地、远程仓库：
![[Pasted image 20260416191552.png]]

### Maven in IDEA
- 配置全局Maven：
![[Pasted image 20260416194315.png]]

以及Maven 的runner：
![[Pasted image 20260416194406.png]]

java编译器选jdk版本一致就行。

- 配置IDEA的空项目
 - 创建空项目
![[Pasted image 20260416194506.png]]

#### Maven坐标（资源的唯一标识，唯一标识Maven项目以及引入依赖）
![[Pasted image 20260416195645.png]]
#### Maven导入
方式一，导入pom.xml配置文件
![[Pasted image 20260416195857.png]]

#### Maven依赖配置
![[Pasted image 20260416200815.png]]
![[Pasted image 20260416200915.png]]


**Maven的依赖传递**：我们导入的依赖也会依赖其他jar包，此时会随着一起导入，而如果不需要传递进来的某个依赖，我们在配置导入的Dependency（单个具体依赖）内用Exclusions（注意是s，因为可能排除多个）标签配置即可：
![[Pasted image 20260416202336.png]]



### Maven生命周期：
![[Pasted image 20260416203135.png]]

![[Pasted image 20260416203217.png]]
核心关注五点：
![[Pasted image 20260416203343.png]]

- 注意是同一套，下面四个属于default，执行时clean不会在前先执行（不是同一套生命周期）
- 且install会将当前maven项目安装到本地仓库，查找文件夹时需根据签名查找对应的jar/war包。

**执行生命周期的方式：
（1）Maven面板
（2）命令行：`mvn 生命周期`
![[Pasted image 20260416203449.png]]


### Maven的依赖范围：
![[Pasted image 20260420210528.png]]
导入的jar包通过scope标签限定其作用范围。

**与生命周期相关：** 点击maven生命周期中的test时，所有符合规范的单元测试都会运行，然后在控制台输出日志，若执行test后续的操作时，想跳过前方的test阶段，点右上角”禁止“图标即可。
![[Pasted image 20260420211055.png]]


