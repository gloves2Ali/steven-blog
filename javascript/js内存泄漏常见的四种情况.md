#### js内存泄漏常见的四种情况

+ 意外的全局变量  

js中如果不用var声明变量,该变量将被视为window对象(全局对象)的属性,也就是全局变量.
```javascript
function foo(arg) {
    bar = "this is a hidden global variable";
}

// 上面的函数等价于
function foo(arg) {
    window.bar = "this is an explicit global variable";
}
```
所以,你调用完了函数以后,变量仍然存在,导致泄漏.
如果不注意this的话,还可能会这么漏:
```javascript
function foo() {
    this.variable = "potential accidental global";
}
// 没有对象调用foo, 也没有给它绑定this, 所以this是window
foo();
```
你可以通过加上'use strict'启用严格模式来避免这类问题, 严格模式会组织你创建意外的全局变量.

+ 被遗忘的定时器或者回调
```javascript
var someResource = getData();
setInterval(function() {
    var node = document.getElementById('Node');
    if(node) {
        node.innerHTML = JSON.stringify(someResource));
    }
}, 1000);
```
这样的代码很常见, 如果id为Node的元素从DOM中移除, 该定时器仍会存在, 同时, 因为回调函数中包含对someResource的引用, 定时器外面的someResource也不会被释放.

+ 没有清理的DOM元素引用
```javascript
var elements = {
    button: document.getElementById('button'),
    image: document.getElementById('image'),
    text: document.getElementById('text')
};

function doStuff() {
    image.src = 'http://some.url/image';
    button.click();
    console.log(text.innerHTML);
}

function removeButton() {
    document.body.removeChild(document.getElementById('button'));

    // 虽然我们用removeChild移除了button, 但是还在elements对象里保存着#button的引用
    // 换言之, DOM元素还在内存里面.
}
```

+ 闭包
先看这样一段代码:
```javascript
var theThing = null;
var replaceThing = function () {
  var someMessage = '123'
  theThing = {
    someMethod: function () {
      console.log(someMessage);
    }
  };
};
```
调用replaceThing之后, 调用theThing.someMethod, 会输出123, 基本的闭包, 我想到这里应该不难理解.

解释一下的话, theThing包含一个someMethod方法, 该方法引用了函数中的someMessage变量, 所以函数中的someMessage变量不会被回收, 调用someMethod可以拿到它正确的console.log出来.

接下来我这么改一下:
```javascript
var theThing = null;
var replaceThing = function () {
  var originalThing = theThing;
  var someMessage = '123'
  theThing = {
    longStr: new Array(1000000).join('*'),        // 大概占用1MB内存
    someMethod: function () {
      console.log(someMessage);
    }
  };
};
```
我们先做一个假设, 如果函数中所有的私有变量, 不管someMethod用不用, 都被放进闭包的话, 那么会发生什么呢.

第一次调用replaceThing, 闭包中包含originalThing = null和someMessage = '123', 我们设函数结束时, theThing的值为theThing_1.

第二次调用replaceThing, 如果我们的假设成立, originalThing = theThing_1和someMessage = '123'.我们设第二次调用函数结束时, theThing的值为theThing_2.注意, 此时的originalThing保存着theThing_1, theThing_1包含着和theThing_2截然不同的someMethod, theThing_1的someMethod中包含一个someMessage, 同样如果我们的假设成立, 第一次的originalThing = null应该也在.

所以, 如果我们的假设成立, 第二次调用以后, 内存中有theThing_1和theThing_2, 因为他们都是靠longStr把占用内存撑起来, 所以第二次调用以后, 内存消耗比第一次多1MB.

如果你亲自试了(使用Chrome的Profiles查看每次调用后的内存快照), 会发现我们的假设是不成立的, 浏览器很聪明, 它只会把someMethod用到的变量保存下来, 用不到的就不保存了, 这为我们节省了内存.
但如果我们这么写:
```javascript
var theThing = null;
var replaceThing = function () {
  var originalThing = theThing;
  var unused = function () {
    if (originalThing)
      console.log("hi");
  };
  var someMessage = '123'
  theThing = {
    longStr: new Array(1000000).join('*'),
    someMethod: function () {
      console.log(someMessage);
    }
  };
};
```
unused这个函数我们没有用到, 但是它用了originalThing变量, 接下来, 如果你一次次调用replaceThing, 你会看到内存1MB 1MB的涨.

也就是说, 虽然我们没有使用unused, 但是因为它使用了originalThing, 使得它也被放进闭包了, 内存漏了.

强烈建议读者亲自试试在这几种情况下产生的内存变化.

这种情况产生的原因, 通俗讲, 是因为无论someMethod还是unused, 他们其中所需要用到的在replaceThing中定义的变量是保存在一起的, 所以就漏了.