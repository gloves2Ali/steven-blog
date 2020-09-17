Memoization 是一种将函数返回值缓存起来的方法，在 Lisp, Ruby, Perl, Python 等语言中使用非常广泛。随着 Ajax 的兴起，客户端对服务器的请求越来越密集（经典如 autocomplete），如果有一个良好的缓存机制，那么客户端 JavaScript 程序的效率的提升是显而易见的。

Memoization 原理非常简单，就是把函数的每次执行结果都放入一个散列表中，在接下来的执行中，在散列表中查找是否已经有相应执行过的值，如果有，直接返回该值，没有才真正执行函数体的求值部分。很明显，找值，尤其是在散列中找值，比执行函数快多了。现代 JavaScript 的开发也已经大量使用这种技术。
```html
 
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd"> 
<html> 
<head> 
<meta http-equiv="content-type" content="text/html; charset=utf-8"/> 
<title>JavaScript Memoization</title> 
<script type="text/javascript"> 
 
/**
 * JavaScript Momoization
 * @param {string} func name of function / method
 * @param {object} [obj] mothed's object or scope correction object
 *
 * MIT / BSD license
 */
 
function Memoize(func, obj){
    var obj = obj || window,
        func = obj[func],
        cache = {};
    return function(){
        var key = Array.prototype.join.call(arguments, '_');
        if (!(key in cache))
            cache[key] = func.apply(obj, arguments);
        return cache[key];
    }
}
 
 
var fib = {
    fib: function(n){
         if (n == 0 || n == 1)
             return 1;
        return this.fib(n-1) + this.fib(n-2);
    },
    fib_memo: function(n){
         if (n == 0 || n == 1)
             return 1;
        return this.fib_memo(n-1) + this.fib_memo(n-2);
    }
}
 
fib.fib_memo = Memoize('fib_memo', fib);
 
function get(id){
    return document.getElementById(id);
}
 
function test(el, func, scope, arg){
    if ('string' == typeof el) el = get(el);
    el.onclick = function(){
        var start = (new Date()).getTime();
        var result = func.call(scope, arg);
        var end = (new Date()).getTime();
        this.nextSibling.innerHTML = '结果为 ' + result + '，耗时 ' + (end-start) + 'ms';
    }
}
 
window.onload = function(){
    test('fib', fib.fib, fib, 30);
    test('fib_memo', fib.fib_memo, fib, 30);
}
</script> 
</head> 
<body> 
<h1>JavaScript Memoization</h1> 
<h2>Snippets</h2> 
<pre class="brush:javascript;gutter:false;toolbar:false"> 
/**
 * JavaScript Momoization
 * @param {string} func name of function / method
 * @param {object} [obj] mothed's object or scope correction object
 *
 * MIT / BSD license
 */
 
function Memoize(func, obj){
    var obj = obj || window,
        func = obj[func],
        cache = {};
    return function(){
        var key = Array.prototype.join.call(arguments, '_');
        if (!(key in cache))
            cache[key] = func.apply(obj, arguments);
        return cache[key];
    }
}
 
var fib = {
    fib: function(n){
         if (n == 0 || n == 1)
             return 1;
        return this.fib(n-1) + this.fib(n-2);
    },
    fib_memo: function(n){
         if (n == 0 || n == 1)
             return 1;
        return this.fib_memo(n-1) + this.fib_memo(n-2);
    }
}
 
fib.fib_memo = Memoize('fib_memo', fib);
 
</pre> 
<p><button id="fib">测试 <code>fib.fib(30)</code></button><b></b></p> 
<p><button id="fib_memo">测试 <code>fib.fib_memo(30)</code></button><b></b></p> 
</body> 
</html> 

```