#### webpack打包优化

+ 如何减少打包事件

1、优化Loader：
    ，babel会将代码转成字符串并生成AST，然后继续转化成新的代码，转换的代码越多，效率越低。
    1）优化搜索范围、2）缓存编译过的文件 - cacheDirectory
    
2、HappyPack
    webpack打包也是单线程，使用HappyPack可以将loader的同步执行转为并行，从而减少loader的编译时间
    
3、DllPlugin
    该插件可以将特定的类库提前打包然后引入，这种方式可以极大的减少类库的打包次数，只有当类库有更新版本时才会重新打包
    
4、启用gzip压缩

+ 如何减少打包大小

1、按需加载、首页加载文件越小越好
2、Tree shaking，删除项目中未引用的代码（webpack4默认开启）