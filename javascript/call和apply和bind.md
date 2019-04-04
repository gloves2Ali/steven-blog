### call、apply、bind的区别

* 首先说下前两者的区别。
* call和apply都是为了解决改变this的指向。作用都是相同的，只是传参的方式不同。
* 除了第一个参数外（不传入第一个参数，那么默认为window），可以接受一个参数列表，apply只接受一个参数数组。
``` Javascript 
    let a = {
      value: 1
    }
    function getValue(name, age) {
      console.log(name)
      console.log(age)
      console.log(this.value)
    }
    getValue.call(a, 'yck', '24') // -> 'yck' '24' 1
    getValue.apply(a, ['cdt', '39']) // -> 'cdt' '39' 1
```

* 模拟实现call和apply
1. 不传入第一个参数，那么默认为window
2. 改变了this指向，让新的对象可以执行该函数。那么思路是否可以变成给新的对象添加一个函数，然后在执行完以后删除
``` Javascript
    Function.prototype.myCall = function (context) {
      var context = context || window
      // 给context添加一个属性
      // getValue.call(a, 'yck', '25') => a.fn = getValue
      context.fn = this
      // 将context后面的参数取出来
      var args = [...arguments].slice(1)
      // getValue.call(a, 'yck', '24') => a.fn('yck', '24')
      var result = context.fn(...args)
      // 删除fn
      delete context.fn
      return result
    }

    // 执行代码
    var val = {
    value: 1
    }
    function getValue(name, age) {
      console.log(name)
      console.log(age)
      console.log(this.value)
    }
    getValue.myCall(val, 'cdt', '40') // -> cdt 40 1
```
