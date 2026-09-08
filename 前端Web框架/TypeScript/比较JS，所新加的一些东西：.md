### JS的数据类型外，六种新的类型；

### 索引签名：
```ts
// 索引签名：

let a:{name:string,

  age:number,

  [key:string]:any

// key是string类型，值是any

}

// 在原基础上可以新增键值对
```


### 函数类型声明：
```ts
// 函数类型声明：

// 1.直接定义函数时对参数和返回值进行限定

function myGEM(name:string):string{

  return `${name}`

}

// 2.定义一个变量，该变量存的是函数类型，从而对变量进行限制：

let b :(c:number,d:number) => number
```

### 数组类型声明：
```ts
let words:string[]

let nums:Array<number>//泛型
```
#### 元组：
```ts
// tuple:元组，特殊的数组，元素数量固定，类型已知且可不同：

let tuple : [number,string?,...boolean[]]
```

### 枚举类型：
```ts
// enum枚举类型：

// 1.数字枚举

enum Direction{

  Up,

  Down

}

// 值默认是自动递增的数字，反省映射，可以通过值来获取枚举成员名称

console.log(Direction.Up);

console.log(Direction[0]);

  

// 2.字符串枚举，给值，但丧失反向映射：

enum worlds{

  Asia = 'Asia'

}

  

// 3.常量枚举：内联，减少生成的js代码量

const enum ups{

  Bili = "xiaosa"

}
```

### Type类型别名：
```ts
// type:给类型起别名：

type shuzi = number

  

// 联合类型：(或)

type union = number | string

  

// 交叉类型：(且)

type Squezz = {

  NO:string

}

  

type Address = {

  Cell:string

}

  

type Home =Squezz & Address

  

let l :Home

l={

  NO:"hh",

  Cell:"ye"

}

  

// 特殊情况：

// 若定义函数时严格定义返回类型为void，就只能接收undefined（不写return函数执行默认结果、或手动返回undefined）

function getArea():void{

  return undefined

}

  
  

// 若先做函数类型定义，给变量规定了此类型后再去赋值函数，则ts不严格规定返回类型

type onlyFunc = ()=>void

//不同于js，在ts中做类型定义时=>表定义函数的返回值，而非箭头函数标志

let aFunc:onlyFunc

aFunc = ()=>999//发现居然能返回undefined以外的类型

aFunc = ()=>""

// 用数组方法作类比：forEach仅遍历故回调无需return，map和find需要将某种操作执行结果返回，故要return

// ts为了保证这种情况下，箭头函数能保持简写形式（此时()=>xxx,xxx就作为函数返回值了），故不严格限制函数返回值。

  

// 既然规定了void，同样是遵循即使返回有东西，也不能将其用作后续操作的限制
```


### 类：
 ```ts
 class Person {

    name: string

    age: number

    constructor(name: string, age: number) {

        this.name = name

        this.age = age

    }

    speak() {

        console.log(`我叫: ${this.name}, 今年${this.age}岁`)

    }

}
 
 ```

#### 类定义的简写形式：
![[Pasted image 20260508143825.png]]


#### 类的继承、方法重写：
```ts
// 继承：

class Student extends Person{

  grade:string;

  constructor(name: string, age: number,grade:string){

    super(name,age);

    this.grade = grade;

  }

  

  study(){

    console.log("studying");

  }

  

  override speak(): void {

    console.log("重写");

  }

  

}
```

#### 访问修饰符：
![[Pasted image 20260508143909.png]]

#### 抽象类
无法被实例化，意义在于可被继承，可有普通方法，也可有抽象方法
*通过场景理解：*
![[Pasted image 20260508152535.png]]

*何时使用抽象类？*：
![[Pasted image 20260508152555.png]]


### 接口：
`interface`：一种定义结构的方式，为类/对象/函数规定一种契约，但只能定义格式不能有实现。

用接口的场景：
![[Pasted image 20260508201838.png]]

1. 定义接口来限制类的格式：
```ts
interface IPerson{

  name:string

  age:number

  

  out():void

}

  

class Person implements IPerson{

  constructor(public name:string,public age:number){}

  

  out(): void {

    console.log(`name:${this.name},age:${this.age}`);

  }

}
```   

2. 定义接口规范对象格式：
```ts
interface UserInterface {

    name: string

    readonly gender: string // 只读属性

    age?: number // 可选属性

    run: (n: number) => void

}

  

// ts中接口可作类型

const user: UserInterface = {

    name: "张三",

    gender: '男',

    run(n) {

        console.log(`奔跑了${n}米`)

    }

};
```

3. 定义接口规范函数：
```ts
interface CounterInterface{

  (n:number):number

  // 描述了一个“函数对象”。任何实现这个接口的变量，必须本身就是一个函数，接收一个 number，返回一个 number。


  // 用:而非=>:

  // 这是是类型签名而非属性定义，表示这个对象本身就是某种东西，即CounterInterface这个类型，本身就是一个函数

}
```

4. 接口间继承：
```ts
// 接口间继承：

interface DB{

  name:string

}

  

interface DBMS extends DB{

  age:number

}

// 用子接口作类型时“必须都有”:

  

const DDBB:DBMS = {

  // 注意分清“使用”和“定义”

  // age:number

  name : "LIQING",

  age:1

}
```

5. 接口重复定义时的自动合并（类似于两个接口间继承）：
```ts
interface First{

  name:string

}

  

interface First{

  age:number

}

  

const testFirst:First = {

  name:"xx",

  age:1

}
```

### 混淆概念的区别：
1. Interface和type:
![[Pasted image 20260508202500.png]]

但接口的追加（合并）和继承都可以通过，type的交叉类型来实现：用&去“并”
![[Pasted image 20260508202641.png]]
用的很少


2. interface和抽象类：
![[Pasted image 20260508202902.png]]


### 泛型：
![[Pasted image 20260508204938.png]]

1. 泛型函数：
```ts
function getNumAndStr<T,U>(date:T,time:U):object{

    return {date,time}

}
```
只要定义时用到就要在函数名后写<>，用大写字母声明一个泛型，在使用时明确类型

2. 泛型接口：
```ts
interface Info<T>{

  name:string

  extraInfo:T

}

  

const p:Info<string> = {

  name:"ha",

  extraInfo:"xp"

}
```

3. 泛型类：对象实例化时指定类型
```ts
class Person<T> { 
	constructor( public name: string, public age: number, public extraInfo: T ) { } 

	speak() { console.log(`我叫${this.name}今年${this.age}岁了`) console.log(this.extraInfo)
	 }
 }

 // 测试代码1 
 const p1 = new Person<number>("tom", 30, 250);
```


### 类型声明文件：
![[Pasted image 20260508205416.png]]
一般不用自己写。