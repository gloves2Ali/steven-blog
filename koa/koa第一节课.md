### 开头语
Node分发的好处，做中间件的优点：  
1.支持大型项目 多后台的联调，像淘宝 就有php，java等后台的接口  
2.让前后端职能分的更加清楚，在公司越来越多，项目越来越大，人员越来越多的情况下，利用node做中间层转发可以很好的避免 ：java说，这个对象要改成数组，前端：。。。。。f**k。那浏览器上做运算、做分组、以及一系列操作是一定会影响性能的、尤其数据量很大的情况  
3.总而言之、前后台的分离更加明确、前台不在过度依赖后台、后端不再过度等待结合前端、方便解耦、降低沟通成本

### 1.1	koa是什么？有什么用？
Koa是基于Node.js平台的web开发框架它很小，但扩展性很强。
koa 是由 Express 原班人马打造的，致力于成为一个更小、更富有表现力、更健壮的 Web 框架。 使用 koa 编写 web 应用，通过组合不同的 generator，可以免除重复繁琐的回调函数嵌套， 并极大地提升错误处理的效率。koa 不在内核方法中绑定任何中间件， 它仅仅提供了一个轻量优雅的函数库，使得编写 Web 应用变得得心应手。

### 1.2 如何安装koa
首先需要安装node.js，具体安装不介绍了，大家网上可以搜索，下面是node.js的下载地址
https://nodejs.org/zh-cn/
ps：它要求Node.js版本高于V7.6。
查看node版本，过低的话进行升级。
```javascript
node -v
```
Mac更新方法：
```javascript
sudo npm install -g n
sudo n stable
```
搭建koa的环境和项目
创建一个文件夹名为koa2-demo,通过命令行执行
```javascript
cd koa2-demo
```
初始化package.json
```javascript
npm init -y
```
成功生成package.json后，安装koa依赖包
```javascript
npm install koa --save
等同于
npm i koa -S
```
ok,安装成功，接下来我们打印hello koa2  
首先新建一个app.js作为初始化文件  
然后配置启动文件脚本，打开package.json,新增启动脚本
```javascript
"scripts": {
    "start": "node app.js"
},
```
接下来打开app.js，开始写我们的代码
```javascript
const Koa = require('koa')
const app = new Koa()

app.use(async ctx => {
	ctx.body = 'hello koa2'
})

app.listen(2009)
console.log('服务已启动')
```
koa最大的优点就在于use.()中间件的功能，注意：引用中间件的时候一起要按照顺序，后面的教程里会有一个坑。   
接下来做一个demo来深入了解async、await和引用中间件顺序  
```javascript
app.use(async (ctx,next) => {
    ctx.body = 'hello koa2'
    await next()
})

app.use(async (ctx,next) => {
    ctx.body = '顺序为2的use'
    await next()
})

app.use(async (ctx,next) => {
    ctx.body = '顺序为3的use'
    console.log('++++++')
    let c = await middleFunction()
    console.log(c)
    console.log('----')
})

function middleFunction() {
    return new Promise((resolve,reject) => {
        resolve('middleFunction插入')
    })
}
```
打印结果：
```javascript
++++++
middleFunction插入
----
```

### 1.3 认识ctx
我们往页面上去打印一下ctx，不难发现，有关于请求的参数，方法，类型，方式都可以从ctx这个上下文中获取到，这为开发提供极大的便利。  
```javascript
app.use(async (ctx,next) => {
    ctx.body = ctx
})
```
#### 1.3.1 模拟一个GET请求
假设一个商城列表请求，我们该如何获取到请求里的参数呢

```javascript
// 请求的路径为
http://localhost:2009/?id=110089&keyValue=stevenchen&type=0

app.use(async (ctx, next) => {
	let url = ctx.url, // 路径
		method = ctx.method, // 方法
		query = ctx.query // 请求参数:json格式
	querystring = ctx.querystring // 请求参数:字符串格式

	ctx.body = {
		url,
		method,
		query,
		querystring
	}
})
```
就能看到所有打印出来的结果，是不是很方便。

#### 1.3.2 模拟POST请求
首先要知道 koa 框架里是没有能够获取post请求参数的方法  
只能通过监听数据的原生方法去获取，
```javascript
let postdata = ''
ctx.req.on('data',(data)=>{
    postdata += data
})
ctx.req.on("end",function(){
    console.log(postdata)
})
```
得到的数据是字符串，要分割成数组，在把数组转成键值对，过于繁琐，不过介绍一种es6的数组转键值对的方法可以参考  
demo:
```javascript
将字符串  
let url ='id=12121&name=刘德华&job=haiqu'
转成键值对
```
```javascript
let queryList = url.split('&')
// queryList = ["id=12121", "name=刘德华", "job=haiqu"]
let queryData = {}
for(let [index,query] of queryList.entries()){
let itemList = query.split('=')
    console.log('itemList',itemList)
    queryData[itemList[0]] = itemList[1]
}
// queryData = {id: "12121", name: "刘德华", job: "haiqu"}
```
以上是原生方法的核心代码，如果有兴趣可以去尝试，不过我比较懒，我喜欢用封装好的中间件，接下来我们介绍一个神奇的中间件：koa-bodyparser  
后续我们会引用各种中间件方便开发，所以请小伙伴们做好准备。
```javascript
npm i koa-bodyparser -S
```
成功加入依赖包，然后在app.js里引入并且调用
```javascript
const bodyParser = require('koa-bodyparser')
app.use(bodyParser());
// 推荐一个更完善的调用方法，能解析开发中遇到的几乎所有类型
app.use(
	bodyParser({
		enableTypes: ['json', 'form', 'text']
	})
)
```
接下来通过一个例子来证明一下我们是否能快速的拿到post请求里的参数>。<  
直接上代码，这是一个表单提交
```javascript
app.use(async ctx => {
	if (ctx.url === '/' && ctx.method === 'GET') {
		let html = `
            <h1>个人信息表单提交</h1>
            <form method="POST" action="/detail">
                <spanp>姓名:</spanp>
                <input name="userName" /><br/>
                <spanp>年龄:</spanp>
                <input name="age" /><br/>
                <spanp>职业:</span>
                <input name="job" /><br/>
                <button type="submit">提交</button>
            </form>
        `
		ctx.body = html
	} else if (ctx.url === '/detail' && ctx.method === 'POST') {
	    // 将post 请求的参数反馈出来
		ctx.body = ctx.request.body
	}
})
```
以上就是有关于get和post请求获取参数的方法，是不是很方便。

### 1.4 koa的路由
这节课很关键，我会教大家在实际开发项目中，如何模块化的去构造路由。学好这节课，就可以顺势去学渲染模版了，也是非常简单，不过要一步步来。

#### 1.4.1 koa（node）原生路由
++**这里有一个非常关键的知识点，node路由并不是链接，而且读取html页面内容反馈到js里，通过js渲染出页面**++  
接下来展示一段原生路由的渲染代码，来理解一下。  
首先我们要用到node原生的模块包,用来读取页面
```javascript
const fs = require('fs');
```
接下来我们在根目录下创建一个新的文件夹views，用来存放页面。然后新建三个页面index.html、list.html、error.html。  
index.html:
```javascript
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>index.html</title>
</head>
<body>
    <h1>this is index.html</h1>
</body>
</html>
```
list.html:
```javascript
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>list.html</title>
</head>
<body>
    <h1>this is list.html</h1>
</body>
</html>
```
error.html:
```javascript
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>error.html</title>
</head>
<body>
    <h1>this is error.html</h1>
</body>
</html>
```
接下来在app.js写中间件和渲染方法
```javascript
/**
 * 1.获取访问路径
 * 2.选择页面
 * 3.通过ctx.body渲染在页面上
 */
app.use(async (ctx, next) => {
	let url = ctx.url
	html = await selectRoute(url)
	ctx.body = html
})

/**
 * 选择读取的页面
 * @param {*} url 路径
 */
async function selectRoute(url) {
	let page = 'error.html' // 默认为错误页面
	switch (url) {
        case '/':
        case '/index':
			page = 'index.html'
			break
		case '/list':
			page = 'list.html'
			break
		default:
			break
	}
	// 因为页面读取是异步操作,有可能会报错,所以让他同步读取
	let html = await renderHtml(page)
	return html
}

/**
 * 拿到读取页面的内容
 * @param {*} page 页面名称
 */
function renderHtml(page) {
	return new Promise((resolve, reject) => {
		let pageUrl = `./views/${page}`
		fs.readFile(pageUrl, 'binary', (err, data) => {
			if (err) {
				reject(err)
			} else {
				resolve(data)
			}
		})
	})
}
```
以上就是基础的路由渲染方法，比较简单，也有很多不全的地方。下面就介绍koa-router这个中间件，能带给我们开发多大的方便。

#### 1.4.2 koa-router
首先安装koa-router中间件
```javascript
npm i koa-router -S
```
安装成功后，在app.js里引用初始化
```javascript
const Router = require('koa-router')
const router = new Router()
等同于
const router = require('koa-router')()
```
全部完成后，接下来我们用这个router中间件，写一个简单的路由demo
```javascript
// 如果这个方法里不去执行next()，会卡在这个引用方法里，所以开头说的，顺序很重要。
app.use(async (ctx, next) => {
	console.log('卡在这里了')
	await next()
})

router.get('/list', async (ctx, next) => {
	ctx.body = 'list页面'
})
app.use(router.routes(), router.allowedMethods())
```
++小技巧：路由可以连接着写++  
```javascript
router.get('/list', async (ctx, next) => {
	ctx.body = 'list页面'
}).get('/detail',(ctx,next)=>{
    ctx.body = 'detail页面'
})
```

### 1.4.3 模块化构建层级路由
简单的路由可以满足我们玩儿，但是不能满足写项目，会很累   
假设现在有一个商城，有列表、详情、订单、活动等等数个甚至数十个模块，不可能将所有的路由都放在app.js去引用。会导致模块难以维护和代码太过混乱。所以这时候，我们需要去模块化路由。  
接下来我们要做的最重要的一件事就是...在根目录下创建名为router的文件夹，然后在router文件夹里，创建2个js文件，分别为list.js和detail.js，代表商城的列表和详情路由。这样我们就可以把所有有关于列表的请求都写在这个请求里，十分易于维护，并且我们建立层级功能，让路由一目了然。  
看一下list.js该怎么写这个路由  

```javascript
list.js:

const router = require('koa-router')()

router.get('/user',async(ctx,next)=>{
    ctx.body = '<h1>路径为 /list/user </h1>'
})

module.exports = router
```
暴露出list.js里的路由模块后，在app.js里去引用这个模块
```javascript
app.js:

// 引入list路由
const listRoute = require('./router/list')
// 引用list子路由
router.use('/list',listRoute.routes(), listRoute.allowedMethods())
// 引用总路由中间件
app.use(router.routes(), router.allowedMethods())
```
访问 http://localhost:2009/list/user 可以看到页面上出现了变化，详情页、活动页也是相同构造。

### 1.5 pug引擎

Pug原名不叫Pug，是大名鼎鼎的jade，后来由于商标的原因，改为Pug，哈巴狗。其实只是换个名字，语法都与jade一样。丑话说在前面，Pug有它本身的缺点——可移植性差，调试困难，性能并不出色，但使用它可以加快开发效率。     
利用pug只是抛砖引玉，后期可以根据项目开发需求加入各类的引擎模版  
#### 1.5.1 安装并使用pug渲染页面
```javascript
npm i pug -S
```
完成之后，还要安装一个koa-views依赖，用于映射页面
```javascript
npm i koa-views -S
```
在app.js里引用这个模块,并且使用app.use插入中间件
```javascript
const views = require('koa-views')

app.use(
	views(__dirname + '/views', {
		extension: 'pug'
	})
)
```
ok，这个views文件夹是我们上面模拟原生路由创建的。现在我们创建一个user.pug，创建ok之后，我们就只剩下来两个问题。  
1. 用什么方法将数据返到页面？
2. pug模版是怎么渲染页面的？  
通过一个简单的demo来实现，打开刚才使用到的/router/list.js
```javascript
/router/list.js:

router.get('/user',async(ctx,next)=>{
    let title = '你好，刘德华!'
    await ctx.render('user',{
        title
    })
})

user.pug:

// 这两种方法都是可以获取到值的
h1= title
h2 #{title}
```

打开 http://localhost:2009/list/user 这个地址，发现页面上将title渲染出来了，是不是很酷？



#### 1.5.2 模块化pug
学习新知识的时候，一定要去思考，如何将这个知识点加入到实际开发项目当中去。在vuejs的项目里，变动的只是id=app的div里内容，pug也能做到代码公共化。  
在views下创建一个底层文件layout.pug
```javascript
layout.pug:

doctype html
html
  head
    title= title
    block stylesheet
    link(rel='stylesheet', href='/stylesheets/style.css')
  body
    block content
    p 这里是layout的内容
```
在user.pug里引用layout.pug
```javascript
user.pug:

extends layout

block content
    h1= title
    h2 #{title}
```
刷新页面，可以看到user页面加载了头文件layout.pug，这样我们就可以在开发的时候把需要的模块提出来，达到了模块化的目的。

#### 1.5.3 引用静态资源
开发中一定会用到一些静态资源引用，那如何在koa中去引用这些资源呢？还是需要依赖包的中间件来实现，koa-static就能做到这个事  
安装依赖包
```javascript
npm i koa-static -S
```
在根目录下创建public文件夹用来存放静态资源  
完成之后引用：
```javascript
app.use(require('koa-static')(__dirname + '/public'))
```
public下创建一个stylesheets文件夹，再创建style.css作为根样式表
```javascript
style.css:

h1 {
    color: blue
}
h2 {
    color: red;
}
```
接下来查看一下user.pug里的h1和h2字体颜色是否有变化

#### 1.5.4 对象数组的渲染和mixin方法的实现
这里讲2点知识点  
第一点：在实际开发中，绝大多数我们会用到数组和对象的列表渲染，那么在koa中该如何做。  
第二点：mixin是模块方法，可以公共使用，极大的方便了开发，mixin方法该如何使用。  
下面展示的代码可以解决上面所有的疑问  
首先在router - list.js里模拟一串数据
```javascript
list.js:

router.get('/user',async(ctx,next)=>{
    let title = '你好，刘德华!'
    sortitem = {
		errorcode: '201',
		data: [
			{
				title: '酒店标题1',
				subtitle: '酒店副标题1',
				mark: ['酒店标签1', '酒店标签2'],
				price: 2108,
				location: ['100', '100'],
				introduce: 'no1.景区',
				notice: '这是第一条数据结尾'
			},
			{
				title: '酒店标题2',
				subtitle: '酒店副标题2',
				mark: ['酒店标签a', '酒店标签b'],
				price: 2108 - 2,
				location: ['100', '100'],
				introduce: '第二个景区',
				notice: '这是第二条数据'
			}
		]
	}
	await ctx.render('user', {
		title,
		sortitem
	})
})
```
```javascript
user.pug:

extends layout

block content
    h1= title
    h2 #{title}
    p 接下来是渲染列表
        +itemGroup(sortitem.data)

mixin itemGroup(item)
    ul
        each val,index in item
            each items in val.mark
                li #{items}
            li #{val.title}
            li #{val.subtitle}
            li #{val.price}
            li #{val.introduce}
            li #{val.notice}
```
重启服务，刷新页面，可以看到页面中加载了对象和数组的渲染，并且我们是引用了mixin的方法去实现了，如果你的页面中有模块重用，可以试试mixin。

