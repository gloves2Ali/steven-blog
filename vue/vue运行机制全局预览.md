#### vue运行机制全局预览
1、初始化：在new Vue()之后，调用init函数进行初始化，通过Object.defineProperty设置setter与getter函数。  

2、挂载：调用$mount挂载组件  

3、编译： 1）使用正则编译template模版中的指令
       2）标记静态节点
       3）将AST转化成render function
       4) 执行render function就可以得到新的VNode节点  

4、触发getter函数进行依赖收集，将观察者Watcher对象加入订阅者Dep中。触发setter函数通知到Watcher值修改了，需要重新修改视图，Watcher开始调用update来更新视图，中心还有一个patch的过程。（getter --> setter --> Watcher -->update）
5、patch：经过diff算法比较新老VNode差异，进行dom更新。（使用innerHTML渲染真实DOM）