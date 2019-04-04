#### 1. 安装webpack需要的依赖包
```javascript
sudo npm install webpack webpack-cli webpack-dev-server -g
```

#### 2.安装成功之后接着创建我们需要使用到的文件夹
```javascript
mkdir config dist src
```
config: 配置文件  
dist: 压缩文件  
src: 入口文件  

#### 3.创建packjson文件
```javascript
npm init -y
```

#### 4.创建入口文件和打包文件
```javascript
touch dist/index.html src/index.js
```

webpack4中打包会默认找src/index.js作为默认入口，所以我们现在先默认打包一下空的js文件，看看是否有用   
输入命令:
```javascript
webpack
```
ok,就能看到文件已经打包完成了。但是我们看到终端报了mode的错误，什么原因呢:   
**mode是webpack中独有的，有两种打包环境，一个是开发环境：development另外一个是生产环境：production
打包的时候输入webpack --mode=development或者webpack --mode=production就不会出现警告提示了** 

看一下你的dist下是不是存在一个mian.js，并且打开main.js，你会发现代码自动压缩了。这个也是webpack4带来的一个很方便的自动化打包功能之一。同时项目文件夹下面多了 一个node_modules文件夹 

#### 5. 创建webpack配置文件
```javascript
touch config/webpack.dev.js
```
接下来我们移除一下dist/main.js src/index.js这两个文件，毕竟我们是自动化打包生成的，总要走点心。
```javascript
rm dist/main.js src/index.js
```
在src目录下，我们创建一个main.js
```javascript
mkdir src/main.js
```

配置**config/webpack.dev.js：**
```javascript
const path = require('path')
module.exports = {
	mode: 'development', // 这里换成production就能压缩js
	// 入口文件的配置
	entry: {
		// 里面的main是可以自定义的
		main: './src/main.js'
	},
	// 出口文件的配置项
	output: {
		// 打包的路径
		path: path.resolve(__dirname, '../dist'),
		// 打包的文件名称
		filename: 'bundle.js'
	},
	// 模块：解读css，转换压缩图片
	module: {},
	// 插件：用于生产模版和各项功能
	plugins: [],
	// 配置webpack开发服务功能
	devServer: {}
}
```
加上注释之后，我们对webpack配置文件的基础目录貌似又更近了一步。现在我们我们来尝试一下我们的配置是否有效，我们在根目录运行
```javascript
webpack --mode="development"
```
发现报错了，因为webpack打包的时候会默认的去找src下的index.js，但是现在被我们删除了，现在只留着main.js，所以我们打开package.json文件，去配置关联我们的webpack.dev.js:
```javascript
"scripts": {
    "build": "webpack --config=config/webpack.dev.js",
    ...
  },
```
ok，关联完成之后，我们就可以跑npm的方式了，运行
```javascript
npm run build
```
运行成功，在dist文件下生成了bundle.js文件。看到这里的同学是不是很熟悉，会不会有一种：哦～～～原来我们的脚手架打包是这么来的。如果有这样的感觉，那我们的初步搭建已经达到目的之一了。这样我们可以愉快的继续深入学习了

#### 6. 多入口文件的配置
如果我们现在有2个入口文件，该怎么办，我们在入口配置里多写一个入口文件试试：
```javascript
/入口文件的配置项
    entry:{
         main:'./src/main.js',
         main2:'./src/main2.js'  // 新增的入口文件
    },
```
打包 npm run build   
我们会发现在4.x低的版本里，会报错错误，而在高版本里，这个错误不会报。归总起来的原因生成了多个相同的文件名bundle.js。所以我们在出口文件动动思路，修改修改。
```javascript
// 出口文件的配置项
	output: {
		// 打包的路径
		path: path.resolve(__dirname, '../dist'),
		// 打包的文件名称
		filename: '[name].js' // 这里的[name]顾名思义，就是入口文件进去的是什么名字，打包出来就是什么名字
	},
```
这时候我们再执行一下打包命令 npm run build，发现在dist下，生成了main.js和main2.js。very good!我们已经实现了多入口文件打包。

#### 7. 配置webpack开发服务功能: webpack-dev-serve
```javascript
// 配置webpack开发服务功能
	devServer: {
		// 设置基本目录结构
		contentBase: path.resolve(__dirname, '../dist'),
		// 服务器的Ip地址，可以使用IP地也可以使用localhost
		host: 'localhost',
		// 服务器压缩是否开启
		compress: true,
		// 配置服务端口号
		port:7771
	}
```

运行webpack-dev-server会报错，因为我们需要在package.json去配置一下webpack-dev-server的启动服务
```javascript
"scripts": {
    "server": "webpak-dev-server --config=config/webpack.dev.js",
    ...
  },
```
这时候，我们运行npm run server就可以跑起来了，我们在浏览器中打开http://localhost:7771/ 就可以看到页面展示出来了。是不是也很熟悉，这个流程？在我们用vue脚手架去开发的时候。   
接下来我们分析一下这个过程   
1. webpack接受到命令之后会压缩文件
2. webpack运行dev-server的命令会启动服务
3. 默认开发的页面为dist/index.html   


ok,接下来我们在index.html页面中去手动引入2个压缩的main.js和main2.js验证是否成功，和我们的流程是否走得通。
```javascript
main.js:
alert('这是main.js')

main2.js:
document.getElementById("study").innerHTML="hello webpack"
```
然后打包，运行，看看效果: ok的，说明我们这个打包流程和服务运行都是完美的。


#### 8. CSS文件的打包
webpack的loaders可以将我们平常写的sass、less转译成css  
首先创建index.css
```javascript
mkdir src/css
touch src/css/index.css
```
我们在index.css里随意写下几行css代码
```css
body {
  background: #999;
  font-size: 30px;
  font-weight: bold;
  color: #000;
}
```
这个时候我们在入口文件，main.js里把css引入
```javascript
import css from './css/index.css'

alert('这是main.js')
```
然后打包一下，发现报错了
```javascript
ERROR in ./src/css/index.css 1:5
Module parse failed: Unexpected token (1:5)
You may need an appropriate loader to handle this file type.
```
报错的原因的，webpack需要loader的支持才会打包css，所以我们下载一个css的loader配置，需要两个依赖包：**style-loader 、 css-loader**    
安装一下**npm i style-loader css-loader -S**   
ok，安装成功之后，接下来我们需要在webpack.dev.js配置module,webpack 根据正则表达式，来确定应该查找哪些文件，并将其提供给指定的 loader。在这种情况下，以 .css 结尾的全部文件，都将被提供给 style-loader 和 css-loader。
代码如下：
```javascript
// 模块：解读css，转换压缩图片
	module: {
		rules: [
			// css loader
			{
				test: /\.css$/,
				use: [
					{
						loader: 'style-loader'
					},
					{
						loader: 'css-loader'
					}
				]
			}
		]
	},
```
ok,配置完css的规则，我们打包一下，看看这个规则是否生效。   
可以的吧，ok，我们css生效了，这里我们就不展示图片了，css的属性都读到了。


#### 9.加载图片
使用file-loader插件来打包图片，先安装一下
```javascript
npm install --save-dev file-loader
```
然后我们配置一下图片压缩的module配置项：
```javascript
module: {
 rules: [
    ...
   {
    test: /\.(png|svg|jpg|gif)$/,
    use: [
       'file-loader'
     ]
   }
 ]
}
```
src/main.js:
```javascript
import a from './a.jpeg'
```

我们开始打包：接下来看见打包的项目中多了一个3fa8159d735d03a652cca81ba0dba7be.jpeg的文件，这意味着 webpack 在 src 文件夹中找到我们的文件，并成功处理过它！

#### 10.打包HTMl文件
**场景**：我们需要把src下的html文件打包到dist文件下，并且自动引入打包好的js。
**实现**：首先将dist下的index.html文件手动剪切到src下，这样子，我们就形成了src为开发目录，dist为打包目录了，从项目分类来说已经分的很明确了。   
现在我们需要一个插件的帮助：**html-webpack-plugin**   
安装一下
```javascript
npm install --save-dev html-webpack-plugin
```
再回到webpack.dev.js的plugins插件模块里：
const HtmlWebpackPlugin = require('html-webpack-plugin');
```javascript
plugins: [
		new HtmlWebpackPlugin({
			title: 'Output Management',
		})
	],
```
在我们构建之前，你应该了解，虽然在 dist/ 文件夹我们已经有 index.html 这个文件，然而 HtmlWebpackPlugin 还是会默认生成 index.html 文件。这就是说，它会用新生成的 index.html 文件，把我们的原来的替换。让我们看下在执行 npm run build 后会发生什么：

我们遇到了：
Error: Cannot find module 'webpack/lib/node/NodeTemplatePlugin'
这个问题，经过查询之后，是由于全局安装了webpack导致的。
解决办法：  
卸载webpack并重装webpack  
npm remove webpack -g  
npm install --save-dev webpack-cli  
npm install --save-dev webpack-cli webpack-command  
我们接着打包，看到文件已经全部打包到dist下面去了。goodjob。

#### 11.less的打包和分离
下面主要说下Less文件如何打包和分离。Less 是一门 CSS 预处理语言，它扩展了 CSS 语言，增加了变量、Mixin、函数等特性，使 CSS 更易维护和扩展。
首先要安装Less的服务  
首先安装一下less和less的打包loader
```javascript
npm install --save-dev less less-loader
```
然后在去配置文件里去配置一下less-loader
```javascript
{
    test: /\.less$/,
    use: [
        {
           loader: "style-loader"
        }, 
        {
            loader: "css-loader" 
        },
        {
            loader: "less-loader" 
        }
    ]
}

```
然后创建一个theme.less文件：
```javascript
@dt: #000;
#cdt{
  width:100px;
  height:100px;
  background-color:@dt;
}
```javascript
引入main.js里中：
```
import './theme.less'
```
我们build一下，我们打开index.html，调试查看head头部里面的style,我们发现我们写的#cdt这个类被打包进来了。说明我们的打包功能是ok的！

#### 12.babel的打包
将ES6打包，适用在低版本浏览器中
```javascript
npm install --save-dev babel-core babel-loader babel-preset-es2015 babel-preset-react
```
完成之后在loader里加入配置
```javascript
{
	test:/\.(jsx|js)$/,
	use:{
		loader:'babel-loader',
		options:{
			presets:[
				'es2015','react'
			]
		}
	},
	exclude:/node_modules/
}
```
报了个错：
```javascript
Error: Cannot find module '@babel/core'
```
查询了一下发现是babel版本的问题,修改一下版本
```javascript
sudo cnpm i babel-loader@7.1.5 -D
```
我们再build一下，发现成功了，然后运行index.html校验一下发现es6的语法我们一句打包进去了。因为我们测试始终是在谷歌浏览器，所以并不会报什么错误，如果在IE8等低版本的浏览器中，babel的使用就显得异常重要。



