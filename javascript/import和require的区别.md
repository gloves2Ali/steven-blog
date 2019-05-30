#### import和require的区别

+ require/exports由CommonJS延伸，而import/export则是新的ECMAScript版本，即ES6包含进来的。事实上，目前我们编写的import/export最终都是编译为require/exports来执行的。而负责编译的就是babel这个神一样的项目。由于babel将还未被宿主环境（浏览器、node等）直接支持的ES6 Module编译为ES5的CommonJS，也就是编译为require/exports这种写法。就是因为用Webpack安装上babel-loader这个loader，才使我们可以随心所欲的使用ES6。

+ 写法：
``` Javascript 
// require/exports用法很简单
// 导出
exports.fs = fs
module.exports = fs
// 引用
const fs = require('fs)


// import/export写法有很多种
// 导出 
export default fs
export const fs
exprot function readFile
export {readFile, read}
export * from 'fs'
// 引用
import fs from 'fs'
import {default as fs} from 'fs'
import * as fs from 'fs'
import {readFile} from 'fs'
import {readFile as read} from 'fs'
import fs, {readFile} from 'fs'
```

+ 区别
require是赋值过程并且是运行时才执行（运行时加载），import是解构过程并且是编译时执行（编译时加载）。require可以理解为一个全局方法，所以就意味着可以在任何地方执行。而import必须写在引用的上面。
