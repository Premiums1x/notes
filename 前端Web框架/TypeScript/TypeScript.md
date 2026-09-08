ts包含js，编译时将ts代码转成js代码

一.编译：
1.npm i typescript -g

tsc（“typescript compiler”） 目标ts文件
这种方法不能热更新，了解即可。

2.在安装typescript基础上自动化编译：
tsc --init
生成一个tsconfig.json配置文件，用来配置ts如何转换为js
- target项：遵循的规范
- noEmitOnError：出错时不提交

tsc --watch （目标ts文件），不加就监视全部ts文件

二、类型声明：
对变量：let a: string ,可规定变量存储什么类型数据
对函数能限制其参数：function count(x: number , y: number), 也能限制函数返回值：
在写参数括号的后面：function count(x: number , y: number) : number{}
- 此外对函数参数的数量限制也很严格
- 字面量类型：let a: "hello"，意思是限制a只能存"hello"这个字面量。

三、类型推断
若无进行类型声明，ts会根据代码自动推断：
let a = 999， ts判断变量a的类型是number

四、类型总览：
js的类型：
- string
- number
- boolean
- null
- undefined
- bigint
- symbol
- object（包含Array、Function、Date、Error等）

TS数据类型：
- 上述所有js类型
六个新类型:
- any：1.任意类型，放弃了对变量的类型检查。 
	2.分显式与隐式（什么都不写）。
	3.any类型变量可赋值给任意类型变量。
- unknown：1.未知类型，一个类型安全的any，不确定数据类型时使用。
	2.无法将unknown类型赋值给任意类型变量，除非1.判断，2.断言:x = a as string 或x = <string> a。
	3.unknown类型访问一切属性、方法都报错，与any相反。
	4.断言指定类型来访问属性、方法：(str as string).toUpperCase
- never
- void
- tuple
- enum

两个自定义类型方式：
- type
- interface
注意string、String，一个是（原始）基本数据类型，一个是包装数据类型（构造函数的名称），包装数据类型的范围 > 基本数据类型，如果限制类型为包装数据类型，则不能赋值为对应的同名小写基本数据类型，其余同理。

一般很少用到大写的构造函数（用于创建包装对象），类型声明时一般用小写

自动装箱：js引擎在必要时会自动将原始类型包装成对象，以便调用方法、属性。