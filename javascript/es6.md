### es6的语法
* class
class的写法只是让对象原型的写法更加清晰、更像面向对象编程的语法。
```javascript
  class Point {
    constructor(x, y) {
        this.x = x
        this.y = y
    }

    toString() {
        return `${this.x}, ${this.y}`
    }
  }

  typeof Point // "function"
  // 类的数据类型就是函数，类本身就指向构造函数

  var p = new Point(1, 2)
  p.toString() // 1, 2

  // 继承
  class ColorPoint extends Point{
    constructor(x,y,color) {
        super(x,y)
        this.color = color
    }

    toString(){
        return this.color + '' + super.toString()
    }
  }
  let cp = new ColorPoint(1,3,'res')
  cp.toString() // res 1 3
```

* map
```javascript
  const map = new Map()
  const obj = {p: 'Hello World'}

  map.set(obj, 'OK')
  map.get(obj)  // 'OK'

  map.has(obj) // true
  map.delete(obj) // ture
  map.has(obj) // false
  map.size // 1
  map.clear()

  for(let key of map.keys()) {
    console.log(key) // {p: 'Hello World'}
  }

  for(let value of map.values()) {
    console.log(value) // OK
  }

  for(let item of map.entries()) {
    console.log(item[0], item[1]) 
    // {p: 'Hello World'} OK
  }
```

* Set
ES6提供了新的数据结构Set。它类似于数组，但是成员的值都是唯一的，没有重复的值。
```javascript
var s = new Set()
[2, 3, 5, 4, 5, 2, 2].map(x => s.add(x))

for(let i of s) {
  console.log(i) 
  // 2 3 5 4
}


var set = new Set([1, 2, 3, 4, 4]);
set // Set(4) {1, 2, 3, 4}
set.size // 4
[...set] // [1, 2, 3, 4]
```

* Generator
```javascript
  function* helloWorldGenerator() {
    yield 'hello';
    yield 'world';
    return 'ending';
  }

  var hw = helloWorldGenerator();

  hw.next()
  // { value: 'hello', done: false }

  hw.next()
  // { value: 'world', done: false }

  hw.next()
  // { value: 'ending', done: true }

  hw.next()
  // { value: undefined, done: true }

  // 总结一下，调用Generator函数，返回一个遍历器对象，代表Generator函数的内部指针。以后，每次调用遍历器对象的next方法，就会返回一个有着value和done两个属性的对象。value属性表示当前的内部状态的值，是yield语句后面那个表达式的值；done属性是一个布尔值，表示是否遍历结束。
```