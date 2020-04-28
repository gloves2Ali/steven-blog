+ 基于js单线程的局限性，如果执行一个很耗时间的函数，那么主线程将会被长时间占用，因此导致事件循环暂停，使得浏览器无法及时渲染和响应，那么将会造成页面崩溃，用户体验下降，所以html5支持了webworker
+ webwork简单理解就是可以让特定的js代码在其他线程中执行，等执行结束后返回结果给主线程接收即可
+ 比如在js中需要实现一个识别图片的算法，而且此算法需要很长的计算时间，如果让js主线程来执行将会导致上述发生的事情，那么正好可以使用webwork技术来实现。
+ 创建一个webworker文件，其中写入算法代码，在最后调用postMessage(result)方法返回结果给主线程，js主代码中通过w=new Worker(文件路径)来创建一个渲染进程的webworker子线程实例，通过w.onmessage=function(e){console.log(e.data)}给其添加一个事件监听器，当webworker中传递消息给js主线程时会在此回调函数中执行，通过调用w.terminate()终止webworker线程
+ webworker线程与js主线程最大的区别就在于webworker线程无法操作window与document对象

``` javascript
// test.html(主线程)
const w = new Worker('postMessage.js')
w.onmessage = function(e) {
  console.log(e.data)
}
w.postMessage('b') // b is cat
w.terminate() // 手动关闭子线程

----------------------------------
// postMessage.js(worker线程)
this.addEventListener('message', (e) => {
  switch (e.data) {
    case 'a': this.postMessage(e.data+' is tom')
      break;
    case 'b': this.postMessage(e.data + ' is cat')
      break;
    default:  this.postMessage(e.data + " i don't know")
    this.close() // 自身关闭
      break;
  }
})
```