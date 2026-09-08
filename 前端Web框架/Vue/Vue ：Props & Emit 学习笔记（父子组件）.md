## 1. Props：父传子（给子组件“传入参数/属性”）

### 1.1 Props 是什么？

- `props` 用于 **父组件 → 子组件** 传递数据。
    
- 可以理解为：**父组件给子组件传一个“属性/参数”**，子组件接收后就能使用。
    
- 本质上：**子组件把父组件传入的数据当作“输入”**（Input）。
    

### 1.2 父组件如何传？

`<MyButton :message="JS表达式" />`

注意点：

- `:` 表示后面是 **JS 表达式**，一般用双引号：`"someExpr"`
    
- 不加 `:` 时，传的是**字符串字面量**：
    
    `<MyButton message="hello" />`
    
- 如果你想用 `:` 但传字符串，需要写成：
    
    `<MyButton :message="'hello'" />`
    

### 1.3 子组件如何接收？

在 `<script setup>` 中使用 `defineProps()` 宏接收：

`<script setup> const props = defineProps({   message: String }) </script>`

使用时：

`props.message`

### 1.4 Props 的特点：一般只读

- `props` 通常是**只读的**（不建议子组件直接修改 props）。
    
- 如果需要基于 props 做计算/加工，使用 `computed()`：
    

`import { computed } from 'vue'  const upperMessage = computed(() => props.message?.toUpperCase())`

### 1.5 常见坑：不要直接解构 props（会丢响应式）

⚠️ 不推荐：

`const { message } = defineProps(['message'])`

这样 `message` 可能会失去响应式，导致更新不跟随。

✅ 推荐：

- 直接用 `props.message`
    
- 或者需要解构时用 `toRefs(props)`（进阶再用也行）
    

---

## 2. Emit：子传父（子组件向父组件“抛出消息/事件”）

### 2.1 Emit 是什么？

- `emit` 用于 **子组件 → 父组件** 通知事件发生了，必要时还能携带参数。
    
- 可以理解为：**子组件发出一个“消息/事件”，父组件监听并决定怎么处理。**
    

一句话总结（非常重要）：

- **子组件决定什么时候发出（when）**
    
- **父组件决定怎么处理（how）**
    

### 2.2 子组件如何定义要抛出的事件？

在 `<script setup>` 中使用 **`defineEmits()`**（注意是复数）：

`<script setup> const emit = defineEmits(['push']) </script>`

### 2.3 子组件什么时候抛出？

通常在用户交互或逻辑完成时抛出，比如点击按钮、输入变化、请求成功等。

✅ 推荐写法（更清晰）：在脚本里调用 `emit()`，模板里只触发函数

`<script setup> const emit = defineEmits(['push'])  function onClick() {   emit('push', '可以带参数') } </script>  <template>   <button @click="onClick">Push</button> </template>`

（你原来写的模板 `$emit("push")` 思路也对，只是 `<script setup>` 更推荐上面这种方式。）

### 2.4 父组件如何监听子组件的 emit？

父组件用事件监听绑定对应事件名：

`<MyButton @push="handlePush" />`

父组件处理函数接收参数：

`<script setup> function handlePush(payload) {   console.log('子组件传来的参数：', payload) } </script>`

---

## 3. 关系总结：Props vs Emit（最核心理解）

### 3.1 数据流方向

- **Props：父 → 子（数据输入）**
    
- **Emit：子 → 父（事件通知）**
    

### 3.2 更准确的“本质”

- **props = 状态/数据（state/data）**
    
- **emit = 事件/通知（event/notify）**
    

所以 emit 不只是“把数据传回去”，更多是：

> 子组件告诉父组件：某件事发生了，请父组件决定是否、以及如何更新状态。

---

## 4. 补充：Vue 常见模式（了解即可）——v-model 本质也是 props + emit

很多双向绑定组件，背后就是：

- 父传子：`modelValue`（props）
    
- 子通知父更新：`update:modelValue`（emit）
    

父组件：

`<MyInput v-model="text" />`

子组件（核心思想）：

`emit('update:modelValue', newValue)`

---

## 5. 一句话记忆法

- **props：给子组件“喂数据”**
    
- **emit：子组件“喊一声”通知父组件发生了什么**
    
- **子发通知，父做决定；父管数据，子别乱改。**