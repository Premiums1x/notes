# JavaScript 原型、构造函数与 new

> 这篇笔记解决一个最容易绕的问题：`prototype`、`__proto__`、构造函数、`new` 到底是什么关系？

## 先记住最核心的一句话

**JavaScript 中，函数也是对象。**

更准确地说：

- 普通对象：可以保存属性和方法
- 函数：也是对象，但比普通对象多了一个能力——**可以被调用 `()`**

例如：

```js
function Person() {}

Person.age = 18
console.log(Person.age) // 18
```

函数 `Person` 能像对象一样挂属性，就是因为函数本身也是一种特殊对象。

---

## 1. 对象的 `[[Prototype]]` 是什么？

JavaScript 对象内部都有一个 `[[Prototype]]` 链接。

它的作用可以理解为：

> **如果我自己没有某个属性，就去哪个对象继续找？**

例如：

```js
const obj = {}

obj.toString()
```

我们没有给 `obj` 定义 `toString`，但它依然可以调用，因为查找过程大致是：

```text
obj
 ↓ [[Prototype]]
Object.prototype
 ↓ [[Prototype]]
null
```

`toString` 就存在于 `Object.prototype` 上。

因此 JavaScript 的属性查找可以简单理解成：

```text
当前对象没有
    ↓
去它的原型找
    ↓
还没有就继续向上找
    ↓
一直找到 null 为止
```

这条向上查找的链，就是**原型链**。

---

## 2. `__proto__` 是什么？

`[[Prototype]]` 是 JavaScript 内部概念，我们不能直接写：

```js
obj.[[Prototype]]
```

很多普通对象可以通过：

```js
obj.__proto__
```

访问自己的原型。

例如：

```js
const obj = {}

console.log(obj.__proto__ === Object.prototype)
// true
```

不过更标准的写法是：

```js
Object.getPrototypeOf(obj)
```

所以学习时可以暂时把：

```js
obj.__proto__
```

理解成：

> **obj 自己的原型是谁？**

或者简单记成：

> `__proto__`：**我爸是谁？**

注意：`__proto__` 并不是那个真正的内部槽本身，它只是常见的访问方式。

---

## 3. 函数也是对象，所以函数也有自己的原型

例如：

```js
function Person() {}
```

`Person` 是函数，但它本身也是对象，所以它自己也有 `[[Prototype]]`。

通常：

```js
Person.__proto__ === Function.prototype
// true
```

于是 `Person` 自己的原型链是：

```text
Person
 ↓ __proto__
Function.prototype
 ↓ __proto__
Object.prototype
 ↓
null
```

这也解释了为什么函数可以直接使用：

```js
Person.call()
Person.apply()
Person.bind()
```

因为这些公共方法主要来自：

```js
Function.prototype
```

所以：

```text
Person 本身是函数对象
        ↓
沿自己的原型链
        ↓
Function.prototype
        ↓
找到 call / apply / bind
```

---

## 4. 最容易混的地方：函数还有一个 `.prototype`

看这个函数：

```js
function Person() {}
```

它同时存在两个完全不同的东西：

```js
Person.__proto__
Person.prototype
```

它们千万不要混。

### `Person.__proto__`

表示：

> **Person 这个函数对象自己继承谁？**

通常：

```js
Person.__proto__ === Function.prototype
```

### `Person.prototype`

这是 `Person` 身上的一个普通属性，它指向一个对象。

```js
console.log(typeof Person.prototype)
// "object"
```

这个对象不是给 `Person` 自己用的，而主要是给未来通过：

```js
new Person()
```

创建出来的实例使用。

因此可以记成：

```text
Person.__proto__
= Person 自己认谁当“父级”

Person.prototype
= Person 将来 new 出来的实例认谁当“父级”
```

或者用一句更好记的话：

> `__proto__`：**我爸是谁？**
>
> `prototype`：**我以后 new 出来的孩子认谁当爸？**

---

## 5. `new Person()` 到底做了什么？

例如：

```js
function Person(name) {
  this.name = name
}

const p = new Person("Tom")
```

为了理解，可以把 `new Person("Tom")` 粗略拆成几步。

### 第一步：创建一个新对象

```js
const p = {}
```

### 第二步：让新对象的原型指向 `Person.prototype`

也就是建立：

```js
p.__proto__ === Person.prototype
```

概念上可以理解成：

```text
p
 ↓ __proto__
Person.prototype
```

### 第三步：用新对象作为 `this` 执行构造函数

类似于：

```js
Person.call(p, "Tom")
```

于是构造函数里的：

```js
this.name = name
```

相当于：

```js
p.name = "Tom"
```

### 第四步：返回这个对象

最终：

```js
const p = new Person("Tom")
```

得到了实例对象 `p`。

> 上面只是为了理解的简化模型。真实的 `new` 还涉及构造函数显式返回对象等规则。

---

## 6. 为什么要有 `Person.prototype`？

假设：

```js
function Person(name) {
  this.name = name
}

Person.prototype.sayHello = function () {
  console.log("Hello, " + this.name)
}

const p1 = new Person("Tom")
const p2 = new Person("Jack")
```

`name` 是每个实例自己的数据：

```text
p1
└── name: "Tom"

p2
└── name: "Jack"
```

但 `sayHello` 可以共享：

```text
           Person.prototype
           └── sayHello()
              ↑         ↑
              │         │
             p1        p2
```

所以：

```js
p1.sayHello === p2.sayHello
// true
```

两个实例最终找到的是同一个函数对象。

这就是 `prototype` 一个非常重要的作用：

> **存放多个实例可以共享的属性和方法。**

---

## 7. `p.sayHello()` 为什么能找到方法？

执行：

```js
p1.sayHello()
```

JavaScript 会沿原型链查找：

```text
p1
│
│ 自己有没有 sayHello？
│ 没有
↓
Person.prototype
│
│ 有没有 sayHello？
│ 有
↓
找到并执行
```

所以实例本身不一定真的拥有这个方法。

可以验证：

```js
p1.hasOwnProperty("sayHello")
// false
```

但：

```js
p1.sayHello
```

依然可以访问，因为方法来自原型链。

---

## 8. `Person.prototype` 自己又是谁的实例？

`Person.prototype` 本身也是一个普通对象。

因此它也有自己的原型：

```js
Person.prototype.__proto__ === Object.prototype
// true
```

于是实例 `p` 的完整原型链通常是：

```text
p
 ↓ __proto__
Person.prototype
 ↓ __proto__
Object.prototype
 ↓ __proto__
null
```

这也解释了为什么：

```js
p.toString()
```

能够调用。

查找过程是：

```text
p
没有 toString
 ↓
Person.prototype
没有 toString
 ↓
Object.prototype
有 toString
 ↓
找到
```

---

## 9. Person 自己和 p 是两条不同的原型链

这是整个知识点最关键的图。

```js
function Person() {}
const p = new Person()
```

### `p` 的原型链

```text
p
 ↓ __proto__
Person.prototype
 ↓ __proto__
Object.prototype
 ↓
null
```

### `Person` 自己的原型链

```text
Person
 ↓ __proto__
Function.prototype
 ↓ __proto__
Object.prototype
 ↓
null
```

为什么会有两条？

因为 `Person` 有两个身份：

1. `Person` 自己是一个**函数对象**
2. `Person` 又可以作为**构造函数**被 `new`

所以：

```text
Person
│
├── __proto__ ──→ Function.prototype
│                  ↑
│                  负责 Person 自己继承什么
│
└── prototype ──→ Person.prototype
                    ↑
                    给 new 出来的实例使用

const p = new Person()

p.__proto__ ─────→ Person.prototype
```

最核心的等式是：

```js
p.__proto__ === Person.prototype
```

更推荐使用标准 API 表达：

```js
Object.getPrototypeOf(p) === Person.prototype
```

---

## 10. `constructor` 又是什么？

默认情况下：

```js
Person.prototype.constructor === Person
// true
```

关系可以理解成：

```text
Person
  │
  │ prototype
  ▼
Person.prototype
  │
  │ constructor
  └────────────→ Person
```

也就是说：

- `Person.prototype` 是一个对象
- 这个对象默认有一个 `constructor` 属性
- `constructor` 又指回 `Person`

注意不要把这里的：

```js
Person.prototype.constructor
```

和 `class` 中写的：

```js
constructor() {}
```

在概念上完全混为一谈。它们有联系，但这里首先只需要理解为原型对象上的一个属性。

关联笔记：[[Class关键字]]

---

## 11. `class` 和这些东西是什么关系？

现代 JavaScript 可以写：

```js
class Person {
  constructor(name) {
    this.name = name
  }

  sayHello() {
    console.log("Hello, " + this.name)
  }
}

const p = new Person("Tom")
```

看起来很像 Java：

```text
class
 ↓
new
 ↓
实例
```

但是 JavaScript 的 `class` 底层仍然建立在**原型机制**之上。

例如：

```js
Object.getPrototypeOf(p) === Person.prototype
// true
```

而 class 中声明的普通实例方法：

```js
sayHello() {}
```

会放在：

```js
Person.prototype
```

上供实例共享。

所以可以粗略理解为：

> **JavaScript 的 `class` 是对“构造函数 + prototype + 原型链”这套机制提供的更现代、更清晰的语法。**

---

## 12. 为什么 `new Date()`、`new Map()` 也这么写？

例如：

```js
const date = new Date()
const map = new Map()
const set = new Set()
```

这里的 `Date`、`Map`、`Set` 都是 JavaScript 提供的构造器。

所以可以统一理解成：

```text
new 某个构造器()
        ↓
创建对应类型的对象
```

例如：

```js
const date = new Date()

Object.getPrototypeOf(date) === Date.prototype
// true
```

因此 `new Date()` 和 Java 中的 `new Date()` 虽然写法很像，但 JavaScript 背后的对象模型是**原型式继承**，并不是 Java 的传统类继承模型。

---

## 13. 不是所有函数都有可用于 `new` 的 `.prototype`

学习时经常说：

> 函数有 `prototype`

这句话是简化说法。

例如普通函数：

```js
function Person() {}

console.log(Person.prototype)
// 一个对象
```

但箭头函数：

```js
const fn = () => {}

console.log(fn.prototype)
// undefined
```

而且箭头函数不能：

```js
new fn()
```

所以更严谨地说：

> **可以作为构造函数使用的普通函数，通常拥有用于实例的 `.prototype`；函数本身作为对象，则仍然拥有自己的 `[[Prototype]]`。**

---

## 14. 一个特殊情况：`Object.create(null)`

通常普通对象最终都会找到：

```js
Object.prototype
```

但可以显式创建一个没有普通原型链的对象：

```js
const obj = Object.create(null)

Object.getPrototypeOf(obj)
// null
```

因此更严谨的说法不是：

> 所有对象的原型最终都是 `Object.prototype`

而是：

> **很多普通对象的原型链最终经过 `Object.prototype`，然后到 `null`。**

---

# 最终总结

如果只复习最核心内容，记下面几条就够了。

## 1. 函数也是对象

```text
函数 = 一种可以被调用的特殊对象
```

所以函数自己也有原型链。

---

## 2. `__proto__` 看的是“自己继承谁”

```js
obj.__proto__
```

可以粗略理解成：

```text
我爸是谁？
```

更标准的方式：

```js
Object.getPrototypeOf(obj)
```

---

## 3. `.prototype` 是构造函数给实例准备的原型对象

```js
function Person() {}
```

```js
Person.prototype
```

可以粗略理解成：

```text
以后 new Person() 出来的对象，认谁当爸？
```

---

## 4. `new` 最重要的关系

```js
const p = new Person()
```

会建立：

```js
Object.getPrototypeOf(p) === Person.prototype
// true
```

也就是：

```text
p
 ↓
Person.prototype
 ↓
Object.prototype
 ↓
null
```

---

## 5. Person 自己还有另一条原型链

因为 `Person` 自己是函数对象：

```text
Person
 ↓
Function.prototype
 ↓
Object.prototype
 ↓
null
```

所以千万不要混淆：

```text
Person.__proto__
```

和：

```text
Person.prototype
```

一句话区分：

> **`Person.__proto__` 管 Person 自己继承谁；`Person.prototype` 管 `new Person()` 创建出来的实例继承谁。**

---

## 一张图记住全部

```text
                    Person
                   /      \
                  /        \
       __proto__ /          \ prototype
                ▼            ▼
     Function.prototype   Person.prototype
                │            ▲
                │            │
                ▼            │ __proto__
        Object.prototype     p
                ▲
                │
                └──── Person.prototype.__proto__

Object.prototype.__proto__ === null
```

理解这张图后，`new`、构造函数、原型链和 JavaScript `class` 基本就串起来了。
