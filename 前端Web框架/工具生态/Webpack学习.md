基本概念理解：  
>  **webpack**：一个 npm 包（构建工具本体）
- **webpack.config.js**：配置文件，用来告诉 webpack 怎么打包/处理资源/输出到哪
```
// webpack.config.js
module.exports = {
  entry: './src/main.js',
  output: { /* ... */ },
  module: { rules: [/* loaders */] },
  plugins: [/* plugins */]
}

```

- **entry**：配置项之一，用来指定“从哪个入口文件开始构建依赖图并打包”


 **Webpack = 从入口出发建立依赖图（模块关系）→ 按规则把各种资源转换成模块 → 把它们组合成浏览器可运行的输出，并在生产构建时做一堆优化。**

Webpack 是一个 **模块打包工具（module bundler）**。它会从你的入口文件（比如 `src/index.js`）开始，把项目里用到的各种“模块”都解析出来：

- JS/TS 模块（`import`/`require`）
    
- CSS/SCSS/Less
    
- 图片、字体等静态资源
    
- 甚至 Vue/React 的单文件组件（配合 loader）
    

然后把它们按规则处理、合并、优化，输出到 `dist/` 之类的目录，得到浏览器能直接跑的文件（bundle）。

### Webpack 用来做什么

它主要解决 4 件大事：

#### 1) 模块化依赖管理

你写的代码会拆成很多文件，通过 `import` 互相引用。Webpack 会把依赖图（dependency graph）构建出来，确保最终输出包含所有需要的东西，并且顺序正确。

#### 2) 资源“都当模块处理”

Webpack 的理念是：**一切皆模块**。  
CSS、图片、字体，都可以被 JS 引入，然后由 Webpack 的规则去处理，比如：

- 把 `scss` 编译成 `css`
    
- 把图片转成文件输出或内联成 base64
    
- 把 CSS 注入到页面或抽离成独立 CSS 文件
    

#### 3) 代码转换（兼容性 & 新语法）

通过 loader / babel 等，把你写的现代代码转换成更多浏览器可兼容的版本，比如：

- ES6+ → ES5
    
- TypeScript → JavaScript
    
- Vue SFC → JavaScript + CSS
    

#### 4) 构建优化与开发体验

Webpack 不只是“打包”，还会做很多工程化能力：

- **开发服务器**（webpack-dev-server）：本地启动、自动刷新、热更新 HMR
    
- **生产优化**：压缩 JS/CSS、tree-shaking、代码分割（split chunks）、缓存优化（hash 文件名）
    
- **环境区分**：development / production 用不同配置
    

---

### 核心概念（你学结构时最重要）

- **Entry（入口）**：从哪开始构建依赖图
    
- **Output（输出）**：打包结果放哪、文件名怎么命名
    
- **Loader**：处理“非 JS 文件”或转换 JS（比如 `babel-loader`, `css-loader`）
    
- **Plugin**：更大范围的构建能力（生成 HTML、抽离 CSS、拷贝静态文件、定义环境变量等）
    
- **Mode**：开发/生产模式，默认优化策略不同
    

一句话总结：

- **loader 负责“把某类文件变成 Webpack 能理解的模块”**
    
- **plugin 负责“在整个构建流程里做更复杂的事情”**

## webpack的项目结构：
- `src/`：业务源码
    
- `public/`：不经打包直接拷贝的静态文件（有的项目叫 `static/`）
    
- `dist/`：构建产物（打包输出）
    
- `webpack.config.js` 或 `webpack/`：webpack 配置（大型项目会拆成多个配置文件）
    
- `package.json`：脚本命令（`dev/build`）和依赖（loader/plugin 都在这里）


### entry是Webpack构建的起点，告诉Webpack从哪个文件开始建立依赖图。
entry是什么？项目的起点文件：
entry 通常指向一个“引导/启动”文件，比如：

- 纯 JS 项目：`./src/index.js`
    
- React 项目：`./src/index.jsx`（里面 `ReactDOM.createRoot(...).render(<App />)`）
    
- Vue 项目：`./src/main.js`（里面 `createApp(App).mount('#app')`）
    
- 多页面项目：可能有多个 entry（每个页面一个）
    

这些入口文件里一般会做“初始化”和“挂载”，并导入必要资源，比如：

- 导入你的根组件：`import App from './App.vue'`
    
- 导入全局样式：`import './styles/index.css'`
    
- 初始化路由/状态管理：`import router from './router'`
    
- 注册全局组件/指令、挂载应用
    
- （可选）引入 polyfill、全局配置、埋点等  


# 实践：

1.创建一个空项目文件夹blog
```cmd
 D:\.A-Front Code学习> mkdir blog
```
2.进入新建的项目文件夹

```cmd
PS D:\.A-Front Code学习> cd blog
```

含义：初始化项目，即**在当前文件夹生成一个 `package.json`**（项目依赖清单 + 脚本配置文件）。

**yarn init**：
创建/初始化一个新的 Node 项目配置文件（`package.json`）。  
正常情况下它会问你一堆问题（项目名、版本、描述、作者、license 等）。
### `y`

`-y` = `--yes`，意思是：**全部用默认值，不提问，直接生成**。

只有这样初始化后我们才能继续：
- `yarn add ...` 安装依赖
- `yarn dev/build` 运行脚本

3.打开vscode
```cmd
 D:\.A-Front Code学习\blog> code .
```

4.为项目安装webpack和webpack-cli
```bash
yarn add -D webpack webpack-cli
```
这句命令拆开看：
### 1）`yarn`

用 **Yarn 包管理器**来执行命令。

### 2）`add`

表示“安装依赖（包）”。等价于 npm 的 `install`。

### 3）`-D`

`-D` 是 `--dev` 的缩写，意思是：把这些包安装为 **devDependencies（开发依赖）**。

开发依赖的意思是：

- 你在**开发/打包**时需要它（比如 webpack 打包）
    
- 但应用真正上线运行时（尤其是后端 Node 项目）不一定需要它  
    对前端项目来说：webpack 是构建工具，所以放 devDependencies 最合理。
    

### 4）`webpack webpack-cli`

要安装的两个包名：

- `webpack`：打包工具本体
    
- `webpack-cli`：命令行工具，让你可以在终端里运行 `webpack` 这个命令

安装完后：
- 这两个包会安装到node_modules/
- package.json同步更新多出一段：
```json
"devDependencies": {
  "webpack": "^5.xx.x",
  "webpack-cli": "^6.xx.x"
}
```
- 锁文件（`yarn.lock`）会更新，用来锁定精确版本。

## 动手进行配置

 **运行webpack（安装后）：**
```cmd
npx webpack
```
**webpack如果不写webpack.config.js配置文件的话：**
- 默认入口将会是根目录下的index.js
- out输出则是根目录下的dist文件夹->中的main.js文件

**以下对于写webpack.config.js配置文件来说:**
在webpack.config.js文件中需要使用node的模块化语法导出一个空对象：
```js
module.exports = {}
```
然后再在这个空对象里面写相应的配置项。


在导出的配置对象可以配置的内容有：
1.打包时采用开发/生产模式：
*（如果不写这个默认会是production模式）*
```js
mode:"development"//开发模式看代码会更方便
//或是mode:"production"，生产则代码体积方面更小
```

2.webpack对依赖项进行识别的入口文件：
```js
entry:'./src/index.js'
```

3.webpack打包后文件存放的位置,用output:{}配置项进行配置,其中可写：
  - filename键值对，自定义打包后js文件的名字
  - path键值对，自定义打包后文件所存放的路径：
  但使用前需导入nodejs提供的path库，通过该库可获得webpack.config.js所在的目录，然后基于此目录再去找新的目录去存放打包后的文件
```js
const path = require('path')
```

```js
  output:{

    // 现在每次生成的文件名都是dist.js,浏览器会根据文件名进行缓存,我们希望浏览器每次都会刷新到最新的js文件,所以要通过给文件名加上hash(一串随机字符串).

    // filename:'[name].[contenthash].js',其中name可以写死也可以不写,不写就默认是main,contenthash则会根据每次js文件的内容随机计算出一串字符,这样每次修改js文件生成的文件名都是带有一串随机字符串的文件名,避免了浏览器缓存

    filename:'dist.js',

    path:path.resolve(__dirname,'dist')//resolve可传多个参数，表示多级路径，此处表示放到当前目录下的dist文件中

  },
```


4.loader（转换器）,配置在module:{}配置项中
webpack只能识别JavaScript、JSON类型文件，其他类型文件（如css、png、vue文件）需要通过loader转换，才能被webpack识别、打包。
*********
**写在module配置项(写loader对应的详细配置：匹配何种后缀的文件以及要用什么loader)中**：
```js
  module:{

    rules:[
//1.这里是style-loader,css-loader(通过yarn add --dev或-D style-loader,css-loader 命令安装 )
      {

        test:/\.css$/i , //正则表达式，表示对应哪些后缀的文件,'.'这个符号需要用\转义，$表示是结尾，i则是忽略大小写
        use:['style-loader','css-loader'] //使用哪些loader（让webpack可以识别对应文件）,从右到左加载
      },

  
//2.对于图片静态资源
      //对于图片等静态资源，webpack原生支持，无需再安装额外的loader

      {
        test:/\.(png|svg|jpg|jpeg|gif)$/i,
        type:'asset/resource'
      },

//3.babel-loader等：当我们使用一些新的JavaScript特性进行开发时，仍要对旧版本的浏览器作兼容处理，如：可将ES6语法新增的箭头函数转换为ES5（以往）的标准函数

//此处下载的包：yarn add --dev babel-loader @babel/core @babel/preset-env
      {
        //使用对象形式，因为要给babel-loader配置一些属性
        // babel-loader：当我们使用一些新的JavaScript特性时仍要兼顾旧版本的浏览器
        test:/\.js$/,
        exclude:/node_modules/,//这样设置就不会转译/编译 node_modules 里的代码
        use:{
          loader:'babel-loader',
          options:{
            presets:["@babel/preset-env"],
            // 给loader传递一些配置，presets:["@babel/preset-env"]，这样就能自动转义代码了
          }
        }
      }
    ],
  },
```

5.插件配置项plugins:[]，用来扩展webpack构建过程的机制，接入 Webpack 的整个生命周期，在合适的时机执行一些自定义操作，如打包优化、资源注入、代码分割、产物分析等。
*****
	1.使用html-webpack-plugin插件:可以自动生成html文件，不用我们手写
	同样需要我们先执行控制台命令进行安装：```yarn add --dev html-webpack-plugin```
	  导入：```const HtmlWebpackPlugin = require('html-webpack-plugin')```,导入的是一个构造函数。在plugins这一配置项的键值对中，值是数组。我们直接在数组中实例化该构造函数，此外，还能在此构造函数中传递参数，个性化生成的html文件：
```js
  plugins:[new HtmlWebpackPlugin({
    // 可以传参自定义生成的html文件的一些属性
    title:'博客列表',

  }),
]
```
	
	2.webpack的一个可视化分析打包情况的插件:webpack-bundle-analyzer:
	先控制台安装yarn add --dev webpack-bundle-analyzer
	然后导入：
```js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer')
//这里导入的是一个对象,我们需要的是其中同名的构造函数,可以这样写:const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin(),这样写就直接导入了构造函数
```
	在plugins配置项中进行配置：
```js
plugins:[ // 导入的BundleAnalyzerPlugin是一个对象,需要访问其内部同名构造函数
  new BundleAnalyzerPlugin.BundleAnalyzerPlugin()]
```
	
	3.文件压缩插件：terser-webpack-plugin
	 先控制台安装：yarn add --dev terser-webpack-plugin
导入：
```js
const TerserPlugin = require('terser-webpack-plugin')
```
	变量中存的也是个构造函数，需要在导出的大对象中新增optimization选项:
```js
 optimization:{
    minimize:true,//是否压缩
    minimizer:[new TerserPlugin()]//用什么工具压缩，直接在数组中实例化构造函数

  }
```

6.webpack提供的开发服务器：webpack-dev-server。启动开发服务器后，当我们对js代码进行修改,其就会自动帮我们重新打包,并刷新页面
依旧先控制台安装：
```bush
yarn add --dev webpack-dev-server
```

为了便于启动开发服务器，在package.json文件中写一个script配置项（脚本）：

```script
  "scripts": {
    "start": "webpack serve --open"
  }
```

在导出的整个大对象中对其进行详细配置，配置项为devServer:{}
如下：
```js
  //指定dev-server从哪里加载代码
  devServer:{
    static:'./dist'
  }
```

7.开发工具配置项，方便看打包后的源代码：
```js
devtool:'inline-source-map'
```

 8.路径别名配置项resolve:{}
 配置路径别名(可以防止因为js文件嵌套过深而需要写很多层相对路径,用别名代替可以简化)
 ```js
   resolve:{

    alias:{

      utils : path.resolve(__dirname,'src/utils')//键值对,键:自己设置的别名,值:真实路径

    }

  }
 ```
 好处：
 ```js
 // import { dateToStr } from "../../utils/date";
//原来的写法:相当于先从里往外艰难跑出来,跑出来之后还要再往里走，绕了一大圈来找一个函数

// 现在设置了别名就直接从外往内找,直接定位,而不用"先翻出去了"这么繁琐了
import { dateToStr } from "utils/date";
 ```

**补充**：我们开发时一般是启动开发服务器进行编码，编码完成后需要打包上线时，需要build（在script里写了一条脚本："build" : "webpack"）才能将打包文件“dist”输出，并确保输出的是我们在服务器中完成的最新代码。因为用开发服务器时，代码和文件是在服务器处进行存储的，并非在本地。

9.source-map 开发工具（只能在development模式使用）：将源码和打包后的代码进行一个映射。（运行的是打包后的，调试的时候对源码调试）
	应用于**当我们使用的是打包后的代码，但我们又想要进行调试时。**
	*注意*:打包代码是不可调试的，只能对源码进行调试，所以需要借助这个工具来给打包后的代码和源码进行映射。
配置项如下：
```js
devtool:"inline-source-map"
```
配置后，对在控制台源代码进行调试时，能直接对源码进行调试：
![[Pasted image 20260116144826.png]]
原本是没有demo_01的，只有main.js这一打包后的代码，我们原来是不能对其进行修改的，但此时demo_01映射源码后，对其修改就是对源码作修改。