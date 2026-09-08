完全等同对于store中state的计算值，通过 `defineStore()` 中的 `getters` 属性来定义它们，**推荐**使用箭头函数，并且它将接收 `state` 作为第一个参数。
```
getters: { doubleCount: (state) => state.count * 2, }
```

### getter除state属性外还可依赖其他getter
通过 `this` 访问到**整个 store 实例**，再去获取所需getter
*注：使用ts时，需定义返回类型*

### 访问store上的getter：
通过 `this`，你可以访问到其他任何 getter，
此时，**需要为这个 getter 指定一个返回值的类型**。
```
doubleCountPlusOne(): number { return this.doubleCount + 1 }
//用到了doubleCount这个getter
```
### 向getter传参？
不可以向它们传递任何参数。
不过，可以从 _getter_ 返回一个函数，该函数可以接受任意参数

此时，getter **不会缓存结果**——每次调用 `getUserById(xxx)` 都会重新执行查找，确保拿到最新数据。
而普通 getter 是*惰性缓存*的，数据变了才重新算。

## 对其他store中getter的访问：
直接在定义当前store时的 getter中 使用
```
getters: { 
	otherGetter(state){ 
		const otherStore = useOtherStore() 
		return state.localData + otherStore.data 
		}
}		
		//import了外部的otherStore
```



## 三种写法访问：getter
  1. setup函数式的用法：
- 直接访问任何 getter(与 访问state 属性一样)

2. 选项式api中：
- **利用`setup()` 返回 store 实例**,然后在computed计算属性选项中，通过this去访问暴露的store，实现对state属性变化的响应（一种回归getter本质，computed，的写法）

3. **不用 `setup()`，用 `mapState` 拍平**
**把 `store.某个getter` 映射成组件自己的 `this.xxx计算属性`**，省掉每处都写 `store.` 前缀。