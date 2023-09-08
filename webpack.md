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