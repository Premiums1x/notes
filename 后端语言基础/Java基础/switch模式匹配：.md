`case` 后面的变量是模式变量。`switch` 会先判断传入对象是否匹配该模式；如果匹配成功，就把对象本身或对象解构出来的组成部分绑定到这些变量上，这些变量随后可以在该 `case` 的代码中直接使用。

`case String str -> "String";`  
`case Integer i -> "Integer";`
这两句的意思是：
- `obj` 如果是 `String` 类型，就匹配成功
    
- 匹配成功后，把 `obj` 转成 `String`，并绑定到变量 `str`
    
- 然后你就可以在右边使用 `str`


### `record` 解构这种

case RRecord(int x, int y) -> "RRecord" + x + " " + y;

这不是普通的“类型 + 变量”了，而是 **record pattern（记录模式）**。

它的意思是：
- 先判断 `obj` 是否是 `RRecord`
- 如果是，再把这个 `RRecord` 里的组件拆出来
- 第一个组件绑定给 `x`
- 第二个组件绑定给 `y`

假设有：
record RRecord(int x, int y) {}

那么：
obj = new RRecord(10, 20)

匹配到这句后，就相当于自动做了：
RRecord r = (RRecord) obj;  
int x = r.x();  
int y = r.y();

所以你可以直接写：
"RRecord" + x + " " + y

这就是“解构”。

---

### 和普通写法的区别

普通写法：

case RRecord r -> "RRecord" + r.x() + " " + r.y();

这里你拿到的是整个对象 `r`。

解构写法：

case RRecord(int x, int y) -> "RRecord" + x + " " + y;

这里你拿到的是对象内部的组件值 `x` 和 `y`，不需要再写 `r.x()`、`r.y()`。