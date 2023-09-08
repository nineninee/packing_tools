## 1. Vite & Webpack
vite也是前端的构件工具。 

当项目比较大时，webpack第一次构件项目会特别慢，虽然第二次会好一些，但是也没好到哪里去。这样会导致开发效率很低，虽然用dev-server会好一些但是还是存在一些问题。

webpack每次运行项目都需要打包，要打包好了才能运行，所以速度不会可观。

vite采用了不同的运行方式：
- 开发时，并不对代码进行打包，而是直接采用ESM的方式来运行项目。
- 在项目部署时，再对项目进行打包，然后将打包的代码部署到服务器。
	- 虽然部署的时候可以不打包，但是不打包的时候浏览器要发很多请求请求不同的资源。
- 除了速度外，vite使用起来更加方便，开箱即用。我们使用webpack时，需要配置，配置plugin、loader等，而vite直接帮我们配置好了
	- vite在一开始就会将应用中的模块区分为依赖和源码两类，改进服务器启动时间。
		- 依赖：大多为在开发时不会变动的JavaScript。vite将使用esbuild预构建依赖，esbuild使用Go编写，并且比以JavaScript编写的打包器预构建依赖快10-100倍。
		- 源码：通常包含一些并非直接是js的脚本，并不是所有的远吗都需要同时被加载。


## 2. Vite的使用
### 安装
`npm i -D vite`
安装完成以后会自动集成命令行工具。

### 使用

#### 基本使用
**vite**不需要创建一个src目录作为源码目录，他默认将整个根目录作为源码目录。

在根目录下创建`index.html`：
```html
<!DOCTYPE html>
<html lang="zh">
    <head>
        <meta charset="UTF-8" />
        <meta http-equiv="X-UA-Compatible" content="IE=edge" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Document</title>
        <script type="module" src="./src/index.js"></script>
    </head>
    <body></body>
</html>
```

在根目下在创建`index.js`：
```js
document.body.insertAdjacentHTML("beforeend", "<h1>你好Vite</h1>")
```

在`index.html`中手动引入`index.js`，**vite**开发模式下是基于**ESM**模块化规范的，所以我们引入index.js时需要指定`type="module"`，表示这里采取的是**ESM**模块化规范。
```html
<!DOCTYPE html>
<html lang="zh">
    <head>
        // ...
        <script type="module" src="./index.js"></script>
    </head>
</html>
```

执行命令：`npm vite`
会启动一个开发的服务器。
我们会发现vite没有给我们生成打包内容，并且很快就启动了服务器。
访问指定的地址能查看到`index.html`中的内容，会显示：
```
你好，Vite。
```

#### 多模块
创建一个m1.js：
```js
export default {
    setH2() {
        document.body.insertAdjacentHTML("beforeend", "<h2>我是m1添加的h2</h2>")
    }
}
```

在index.js中引入：
```js
import m1 from "./m1"

document.body.insertAdjacentHTML("beforeend", "<h1>你好Vite</h1>")
m1.setH2()
```

当我们保存会后发现vite迅速重新加载了文件，网页的内容也自动刷新了，显示：
```
你好Vite
我是m1添加的h2
```

vite会启动内置的服务器，所有的东西都没有打包。

#### src文件夹
所有的内容都放在根目录，文件体量较大时会出现问题。

我们可以把`index.js`和`m1.js`迁移到根目下的src目录下，然后修改`index.html`中引入`index.js`的路径：
```html
<!DOCTYPE html>
<html lang="zh">
    <head>
        // ...
        <script type="module" src="./src/index.js"></script>
    </head>
</html>
```

vite又会快速地重新加载`index.html`，显示的内容和原来一致。

### 打包
`npm vite build`
和之前的webpack比较，这里打包速度快了非常多。

打包以后会生成一个dist目录。我们打开dist目录下的index.html，会发现无法正常显示内容。

vite在开发的时候以ESM这种形式去运行，打包项目的时候也是用ESM模块的形式对项目进行打包。ESM模块必须要通过url加载，上面我们访问index.html是直接访问静态页面，所以页面加载不出来，我们必须通过服务器去访问。
那我们是否可以通过live server去访问呢？也不行，因为根目录有问题，访问的时候必须把dist作为根目录去访问才行。

所以要访问dist下的页面：
- 把dist放到服务器中去执行
- 执行`npm vite preview`启动服务器，这样就可以正常访问。
	- npm vite启动开发服务器，我们代码修改他就会刷新，并没有对代码进行build打包
	- npm vite preview是预览打包后的代码

我们可以把启动开发服务器，打包代码和预览打包后的代码写成脚本文件：
```json
{
	"scripts": {
		"dev": "vite",
		"build": "vite build",
		"preview": "vite preview",
	}
}
```

### vite命令构建项目
前面都是我们手动创建html、js等文件，但是我们他可以直接通过vite帮助我们生成一个项目。

运行`npm create vite@latest`命令，然后一次设置项目名称、项目框架(Vanilla是原生js)、 选择js还是ts开发，然后vite就会帮我们生成项目：
![[vite创建项目.jpg]]



## 3. Vite配置
### 3.1 vite配置好了很多东西
在webpack中，我们在js文件中引入css还需要配置css-loader，但是在vite中，我们可以直接引入使用，样式是生效的。

webpack给我们暴露的是一些比较底层的东西，很多需要我们去配置；
vite很多都帮我们配置好了，我们可以直接拿来用，但是本质都是打包工具。

### 3.2 vite配置兼容性
```js
// src/index.js
import "./styles/index.css"

document.body.insertAdjacentHTML("beforeend", "<h1>你好Vite</h1>")
document.body.onclick = () => {
    alert("哈哈哈")
}
```

vite打包后在dist/assets目录下会有打包好的index.js文件，查看这个文件发现箭头函数是原样打包的。

那vite如何像babel一样将代码转换为旧浏览器支持的代码能？需要使用插件。

Vite可以使用插件进行拓展，这得益于Rollup优秀的插件接口设计和一部分vite独有的额外选项。rollup是vite打包的时候用的一个工具，他是用这个rollup最终对项目进行打包的。如果想对vite扩展功能，就需要给vite添加插件。

要用一个插件，需要将他添加到项目的DevDependencies并在vvite.config.js配置文件中的plugins数组中引入他。

以为传统浏览器提供支持的@vitejs/plugin-legacy插件为例，它类似于babel：
- 新建vite.config.js
	- webpack中是按照CommonJS规范去暴露的，采用的写法是module.exports={}；而vite他把默认的模块化变成了es6的模块化，写法是export default {}。
	- vite还在暴露对象前写了defineConfig，写了以后配置的时候会有补全提示。`import {defineConfig} from "vite"`   `export default defineConfig({})`
- 安装插件：`npm i -D @vitejs/plugin-legacy`
- 引入使用插件
```js
import { defineConfig } from "vite"
import legacy from "@vitejs/plugin-legacy"

export default defineConfig({
    plugins: [
        legacy({})
    ]
})
```

到这里打包会报错，因为legacy在打包的时候要用到terser插件，他可以压缩代码。所以我们需要装一下`npm i -D terser`

打包以后dist/assets下会有index.js和index-legacy.js，前者是没做兼容性处理的，后者是做兼容性处理的。此时我们采取的默认配置，默认所有浏览器都支持箭头函数了，所以这里两个js文件中的箭头函数都还是原样的。

要配置浏览器，我们需要配置legacy，当我们加上ie11以后，箭头函数就能变成普通函数了。
```js
export default defineConfig({
    plugins: [
        legacy({
            targets: ["defaults", "ie 11"]
        })
    ]
})
```

最后的兼容性处理：
在打包的index.html中，我们的两个js文件都会引入。
type="module"表示浏览器支持module，所以大部分语法没问题，引入未兼容低版本的的文件；
nomodule表示这一段代码只有浏览器不支持模块化的时候才会使用这个。
```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <script type="module" crossorigin src="/assets/index.js"></script>
</head>
<body>
    <script nomodule crossorigin  id="vite-legacy-entry" src="/assets/index-legacy.js"></script>
</body>
</html>
```
