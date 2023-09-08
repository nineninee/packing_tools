## 1. 构建工具简介

## 2. webpack介绍


## 3. 配置文件介绍webpack.config.js
`webpack.config.js`用来配置webpack是怎样去运行的，这个文件是webpack的配置文件，webpack是打包工具，webpack是由node运行，所以他是给node看的，所以写`webpack.config.js`的时候要注意使用CommonJS规范。
（PS：也可以设置成ESM规范）

简记：src里的是前端的内容遵循前端开发规范ESM，src外的遵循后台的开发规范。

`webpack.config.js`向外暴露一个对象，这个对象就相当于webpack的配置对象。
```js
module.exports = {
    mode: "production" // 设置打包的模式，production表示生产模式  development 开发模式
}
```
如果没有设置mode，build的时候会有警告。

webpack是很聪明的，他会根据组件是否被导入或被使用来构建项目。
```js
// index.js
import m1 from "./m1"
import m2 from "./m2"
m1.setH1()

// m1.js
// import $ from "jquery"
export default {
    setH1() {
        document.body.insertAdjacentHTML("beforeend", "<h1>你好！webpack</h1>")
    }
}

// m2.js
export default {
    sayHello() {
        console.log("你好，webpack")
    },
    a: 10,
    b: 20
}

console.log("Hello")
```
在上面的`index.js`中，引入了`m2`但是没有用到，webpack检查`m2`时，就不会引入他暴露的对象，但是`m2`暴露对象之外还执行了方法`console.log("Hello")`，所以他会把这个方法打包到最终的文件中。

老师JQuery演示翻车那里其实也是这个道理。


## 4. entry
我们写react项目的时候可能会发现没有`webpack.config.js`配置文件，这是因为react已经配置好了并且隐藏了。

entry用来指定打包时的主文件
- 打包单个文件：根据约定高于配置原则，我们一般的入口文件都是默认的`./src/index.js`
- 打包多个文件(数组）： `entry: ["./src/a.js", "./src/b.js"]`会把`a.js`和`b.js`的内容打包同一个`dist/main.js`中
- 打包多个文件(对象）： `entry: {"./src/a.js", "./src/b.js"}`会分别把`a.js`和`b.js`的内容打包到`dist/a.js`和`dist/b.js`中
```js
const path = require("path")
module.exports = {
    mode: "production", // 设置打包的模式，production表示生产模式  development 开发模式
    // entry: "./hello/hello.js", // 用来指定打包时的主文件 默认 ./src/index.js
    // entry: ["./src/a.js", "./src/b.js"],
    // entry: {
    //     a: "./src/a.js",
    //     b: "./src/b.js"
    // },
    // entry: "./src/a.js",
}
```

## 5. output
output用来配置代码打包后的地址.

- **filename**：指定打包后的文件名。`filename: "main.js"`会将文件打包到`dist/main.js`；`filename: "bundle.js"`会将文件打包到`dist/bundle.js`
	- 打包的时候如果文件名一样，新的打包文件就会覆盖旧文件
	- 如果文件名不一样，会生成新的打包文件，并且不会清除旧的文件
	- filename打包多文件：如上面entry要打包成多文件，但是filename只制定一个文件就会报错，可以用变量：`filename:"[name]-[id]-[hash].js"`，`name`是entry指定的属性值，`id`和`hash`自动生成。
- **clean**：设置true表示自动清理打包目录
- **path**：指定打包目录，必须要绝对路径。`path: path.resolve(__dirname, "hello")`会将文件打包到`hello/main.js`中
```js
const path = require("path")
module.exports = {
    mode: "production", 
    entry: "./src/index.js"
    output: {
        // path: path.resolve(__dirname, "dist"), // 指定打包的目录，必须要绝对路径
        // filename: "main.js", // 打包后的文件名
        // filename:"[name]-[id]-[hash].js",
        // clean: true // 自动清理打包目录
    }, // 配置代码打包后的地址
```

## 5. loader
之前我们都只是打包一些js文件，但是项目中还会有html和css文件等。

如果我们直接在dist目录下新建`index.html`和`index.css`虽然可以实现使用打包的js文件的功能，但是这样是不合理的。
- dist目录应该是由webpack自动生成的
- webpack如果设置clean: true，那么我们创建的`index.html`和`index.css`在重新创建后都会被清除。

那我们就应该把`index.css`创建在src目录下，那`index.html`该怎么使用`index.css`呢，不可能在`index.html`直接使用相对目录导入，但是直接在`index.js`中引入会报错。
```js
// 直接将css引入到js中
import "./style/index.css"

document.body.insertAdjacentHTML("beforeend", "<h1>今天天气真不错，风才12级！</h1>")
```

webpack的核心思想是：万物皆可打包。
但是现在的webpack和express有点类似，只是一个空盒子，只能打包js文件。

如果想让webpack具有处理其他文件的能力的话我们需要引入loader。以css为例，引入css-loader就能处理css文件，可以处理js文件中的样式，其他类型的文件也是同理。
使用步骤：
- 安装：`npm i css-loader -D`
- 配置：配置loader使用module属性的rules，module属性中
	- test：用于匹配文件，`test: /\.css$/i`这里表示匹配以.css结尾的文件
	- use：表示匹配的文件用什么loader处理，`use: "css-loader"`这里表示用css-loader去处理。
	- 配置完以后在index.js中引入index.css虽然没报错，但是样式没起效果，这是因为一个编程思想"单一原则"，一个loader只干一个事，这里css-loader就负责把css转换为js代码，但是他是否在页面中生效他不管。
	- 所以为了使样式生效，还要在引入style-loader。这里还要注意顺序，css-loader要写在后面，因为是从后往前执行的。
```js
const path = require("path")
module.exports = {
    mode: "production",

    module: {
        rules: [
            {
                test: /\.css$/i,
                use: ["style-loader", "css-loader"]
            },
            {
                test:/\.(jpg|png|gif)$/i,
                type:"asset/resource" // 图片直接资源类型的数据，可以通过指定type来处理
            }
        ]
    }
}
```

图片也是一种资源，引入图片也需要图片的loader。如果不配置直接打包会报错。
图片在webpack中直接就支持了，所以rules不用use，直接用type。也就是引入图片不需要做特殊处理，只用返回一个路径就可以。
```js
// 直接将css引入到js中
import "./style/index.css"
// 引入图片
import An from "./assets/an.jpg"

document.body.insertAdjacentHTML("beforeend", "<h1>今天天气真不错，风才12级！</h1>")

document.body.insertAdjacentHTML("beforeend", `<img src="${An}" />`)
// 如果没配置图片loader，打包会报错
```

## 6. babel
webpack是很聪明的，在打包编译的时候会自动帮我们优化我们的代码：
```js
document.body.insertAdjacentHTML(
    "beforeend",
    "<h1>今天天气真不错，风才12级！</h1>"
)

const a = 10
console.log(a)
// 打包以后会直接被优化成console.log(10)

const fn = () => {
 return "哈哈"
}
console.log(fn())
// 打包以后会直接被优化成console.log("哈哈")

document.body.onclick = () => {
    alert("你点我干嘛！")
}
// 这里就不能被优化了，因为webpack不知道回调函数什么时候执行，所以就给我们保留了
```

这里引出了一个新的问题，箭头函数在低版本的浏览器中不支持，那我们下面的代码放到ie中可能就会出现问题。
```js
document.body.onclick = () => {
    alert("你点我干嘛！")
}
```

- 在编写js代码时，经常需要使用一些js中的新特性 ，而新特性在旧的浏览器中的兼容性不好。此时就导致我们无法使用一些新的特性。
- 但是我们现在希望能够使用新的特性，我们可以采用折中的方案。依然使用新特性编写代码，但是代码编写完成时我们可以通过一些工具将新代码转换为旧代码。
- babel就是这样一个工具，可以将新的js语法转换为旧的，以提高代码的兼容性。
- 我们如果希望在webpack中支持babel，则需要向webpack中引入babel的loader。

使用步骤：
- 安装：`npm i -D babel-loader @babel/core @babel/preset-env`
	- babel-loader：类似于一个中间件，把babel和webpack连接到一起了，这样就可以直接在webpack中直接使用babel
	- @babel/core：是babel的核心，转换代码是由他完成的
	- @babel/preset-env：babel中也有很多插件，转换换不同的内容可能对应不同的插件@babel/preset-env准备好了常用的插件，如果不用这个预设，要用什么都要我们自己
- 配置：
	- 匹配js或mjs的文件，mjs指的是模块化的js文件
	- exclude指的是排除的目录，不对这些目录下的文件进行转换
	- use这里用的是对象，表示要做更加详细的配置
		- loader：使用什么loader
		- options：loader遵循什么配置，这里的配置是默认配置
```js
module: {
	rules: [
		{
			test: /\.m?js$/,
			exclude: /(node_modules|bower_components)/,
			use: {
				loader: "babel-loader",
				options: {
					presets: ["@babel/preset-env"]
				}
			}
		}
	]
},
```
配置完成以后，重新build，箭头函数会被转换为普通函数。
```js
// main.js
document.body.onclick = function (){
    alert("你点我干嘛！")
}
```

- 在package.json中设置兼容列表，也就是代码需要兼容哪些浏览器
browserlist: https://github.com/browsesrlist/browserslist

```json
"broeserslist": [
	"defaults"
]
```

## 6. 插件（Plugin）
**loader**是拓展**webpack**处理文件的功能，我们需要新增某些类型的文件就加入对应的**loader**来处理这个文件。

插件用来为**webpack**扩展功能

在前面的实践中，我们都是手动创建`dist/index.html`，但这个需要我们关掉`output`中的`clean`选项，我们现在需要**webpack**在打包的时候自动帮我们生成这样一个html网页。这个功能不涉及打包，只需要自动帮我们创建文件。

**html-webpack-plugin**
- 这个插件可以在打包代码后，自动在打包目录中生成html页面

使用步骤
- 安装依赖：`npm i -D html-webpack-plugin`
- 配置插件
	- 引入插件： 在`webpack.config.js`中先引入, 然后在plugin配置项中配置
	- 配置：**HTMLPlugin**中还可以传入配置对象来设置html，比如html的标题，还有直接在指定一个模板，这样就可以按照这个模板在dist目录下生成对应的html，并且会自动插入\<script>引入`main.js`
```js
const path = require("path")
// 引入html插件
const HTMLPlugin = require("html-webpack-plugin")

module.exports = {
    plugins: [
        new HTMLPlugin({
            // title: "Hello Webpack",
            template: "./src/index.html"
        })
    ],
}
```

## 9. 开发服务器
```json
{
    "scripts": {
        "build": "webpack",
    }
}
```
现在我们每次打包都要重新`npm build`一下，很麻烦。

可以直接在命令行输入`npm webpack --watch`来监视文件是否发生变化来自动打包，每当我们更改文件内容，就会重新自动打包。

也可以将`npm webpack --watch`直接设置成`package.json`中的脚本命令，这样就可以直接在命令行中输入`npm watch`启动
```json
{
    "scripts": {
        "build": "webpack",
        "watch": "webpack --watch"
    }
}
```

但是这种方式我们也不常用，因为watch他是以静态资源的形式去访问的。在真实开发环境中，我们的项目应该部署到服务器上。为了让我们的项目模拟真实的环境，所以在开发的时候通常都希望有一个临时的服务器，这样测试和上线的时候都是服务器。这就要用到webpack-dev-sever

使用步骤：
- 安装：`npm i -D webpack-dev-server`
- 使用：直接在命令行中输入`npm webpack server`就能编译打包资源，并放到一个临时服务器上。
	- `npm webpack server --open`会在打包后直接打开网页
	- 我们可以将这个命令放到package.json中：`"dev": "webpack server --open"`，这样直接在命令行输入`npm dev`就能执行了
```json
{
    "scripts": {
        "build": "webpack",
        "watch": "webpack --watch",
        "dev": "webpack server --open"
    }
}
```