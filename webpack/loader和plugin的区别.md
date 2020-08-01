#### loader和plugin的区别
+ loader是一个扩展器,将A文件进行编译成B文件，比如：将A.less转换为A.css，单纯的文件转换过程；
+ plugin是一个扩展器，它丰富了webpack本身，针对是loader结束后，webpack打包的整个过程，它并不直接操作文件，而是基于事件机制工作，会监听webpack打包过程中的某些节点，执行广泛的任务；