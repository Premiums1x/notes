---
title: "React全局状态管理：createContext"
date: 2026-07-27
section: "React"
source: http://120.77.152.123:8088/posts/ContextAPI
tags:
  - "React"
  - "博客迁移"
---
全局状态管理：createContext()，避免组件间需要通过prop层层透传，类似于Vue的Vuex/Pinia。

`const ThemeContext = createContext();`，声明context；

创建一个组件来封装逻辑，`return`创建的context的Provider API的JSX标签，如：`ThemeContext.Provider`的JSX标签。并通过其value的 prop绑定要发送出去的值，如定义一个数据对象·`value=useMemo(()=>{},[监听的数据])`，就可以绑定：`<ThemeContext.Provider value = {value}> <ThemeContext.Provider/>`。

那么以后子组件要使用发送出来的共享数据时，通过`seContext(ThemeContext)`就能拿到。

注意这里可以用上`useMemo`的Hook来优化组件渲染。因为如果直接写共享的数据对象`value={}`,那么组件每次创建都会在内存创建一份地址不同的数据对象，而React的所有用到这个数据对象的组件，一旦拿到这个新地址的数据对象时，会视作有更新，强制触发每个组件重新渲染。而使用`useMemo`的话只有`value`对象真正改变才会创建新的副本，否则会缓存，一直使用旧内存地址的`value`对象。

**注意**: React Context API 有一条铁律：只有被放在 \<ThemeContext.Provider> ... \</ThemeContext.Provider> 内部的子孙组件，才具备调取这个全局状态的资格。

代码示例：

```
const ThemeContext = createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const [primaryColor, setPrimaryColor] = useState('#1890ff');

  // 使用useMemo优化，避免不必要的重新渲染
  const value = useMemo(() => ({
    theme,
    primaryColor,
    toggleTheme: () => setTheme(prev => prev === 'light' ? 'dark' : 'light'),
    setPrimaryColor
  }), [theme, primaryColor]);

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}
```
