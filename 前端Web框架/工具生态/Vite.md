vite是什么？：一个前端的构建工具。和webpack本质上是一样的，都是打包工具。*二编纠正：Vite = **开发服务器 + 构建工具**，生产构建默认用 **Rollup***
- 开发模式下基于ES模块规范，用模块化的时候是以模块形式引入js文件，所以要在`<script type="module" src="./index.js"></script>`标签中加type属性来表明我们要使用ES的模块化规范

相较于webpack，vite采用不同运行方式：
	- webpack是先打包再运行；vite则是在开发时，并不对代码打包，而是采用ES模块化方式运行项目。*二编纠正：对业务源码是“按需 ESM 提供”，但 Vite 通常会对**依赖**做一次预构建/预打包（这也是它冷启动快、依赖处理稳定的原因之一）*
	- 项目部署时，再对项目进行打包，然后部署到服务器中。

	>能否不打包呢？
	>生产环境通常要打包（减少请求、做压缩/拆包/Tree-shaking 等），但不是“理论上完全不能”。：因为不打包的话就算浏览器支持ES模块化，在页面中也需要动态地加载脚本，脚本一多了，浏览器发送请求就会变多，性能就变差。

	-除速度外，vite使用也更方便（开箱即用，不用像webpack一样去配置loader、plugins...）
	- webpack安装后要创建src文件夹，而vite安装(yarn add -D vite)后直接写在根目录即可，vite的默认目录就是根目录。

> #测试vite
` yarn vite`这句命令跟`yarn webpack`异曲同工之妙，作用是开启一个开发服务器，原理也是写了个script.
>>此时不会有任何打包操作。

> #体验vite热刷新速度：
>新建m1.js: 
> ``` export default{
  setH2(){
    document.body.insertAdjacentHTML("beforeend","<h2>由m1.js添加</h2>")
  }
}```
并在index.js中导入：`import m1 from "./m1"`,使用：`m1.setH2()`
可以发现服务器内容立即刷新了


> #vite能自动找入口文件 ，如果把index.js搬家，之前在wp里还要修改配置文件
>>而在vite中，因为我们在index.html中引入了index.js入口文件，只要这里引入正确，vite能自动寻找，不用自行配置。

> #vite打包 :`yarn vite build` 
> 注意点：开发时js文件是ES模块化，打包后的文件也是ES模块化的，而ES模块需要通过url访问（浏览器访问），也就是说需要使用到服务器
> >打包后的代码想要运行，两种方式：
> >1.把打包后的代码文件放到服务器上
> >2.`yarn vite preview`
> vite 和 vite preview的区别：
> >1.`yarn vite`是启动开发服务器，随着我们代码的更新而同步刷新，并没有执行build打包
> >2.`yarn vite preview` 实质上执行的是我们打包后的文件(dist文件夹目录下)
> 

可以在package.json配置文件中写script配置项，来配置命令：
```js
"scripts":{
"dev":"vite",
"build":"vite build"
"preview":"vite preview"
}
```

> #使用命令行快捷创建vite项目 ：
> 1.使用NPM`npm create vite@latest `
> 2.使用yarn`yarn create vite`
> 3.使用PNPM`pnpm create vite`

创建vite项目后，想写自己的东西的话只需要保留node_modules文件夹和依赖管理文件、lock文件即可，其余的可以自己新建src文件夹等等进行操作。

>vite中无需像wp一样要配置loader才能识别css，其自动帮我们配置了，但是如果要用less这种预编译语言，还是需要先下载、加载一个编译器，然后就可以直接用了。

#vite的配置文件: vite.config.js
*与wp不同，wp暴露的对象中是用node默认的require引入、module.exports暴露，从而实现模块化的*
**在vite的配置文件中使用的是ES6模块化**：import引入，export default暴露对象。
>写法1：`import xxx from xxx` `export default({})`
>写法2（区别就是一个这样写会提示）: `import {defineConfig} from 'vite'` `export default defineConfig({})`

Vite中使用插件：
*vite中插件的使用得益于Rollup的良好插件接口设计，最终对项目进行打包的也是Rollup*
>1.像webpack配置bable插件（实现对旧语法、老浏览器的兼容）一样的效果，vite的插件：
>> -先下载：`npm add -D  @vitejs/plugin-legacy`
>>- 导入：`import legacy from  "@vitejs/plugin-legacy"`
>>- 在vite.config.js的暴露对象里写plugins配置项：`plugins:[legacy()]`,跟wp区别就是wp要在前面加个new。
>>-直接运行`npm run build`，可以看见index.js有两个版本，一个是正常的js文件，另一个是做了兼容的带有legacy的js文件。
>>>可以对legacy这一构造函数进行配置兼容：
```js
plugins:[legacy({
	target:['default','ie 11']//配置兼容的目标
})]
```
>>回到；index.js有两个版本，一个是正常的js文件，另一个是做了兼容的带有legacy的js文件上，这点是跟wp不一样的，因为wp的bable配置好后，进行打包只有一个兼容文件
>>这里vite的处理：
>例：支持模块化的话执行如下代码
>> `<script type="module" crossorigin src="/assets/index-9MGdGYj0.js"></script>`
> 而如果不支持模块化则执行：
> `  <script nomodule>!function(){var e=document,t=e.createElement("script");if(!("noModule"in t)&&"onbeforeload"in t){var n=!1;e.addEventListener("beforeload",(function(e){if(e.target===t)n=!0;else if(!e.target.hasAttribute("nomodule")||!n)return;e.preventDefault()}),!0),t.type="module",t.src=".",e.head.appendChild(t),t.remove()}}();</script>`
>
