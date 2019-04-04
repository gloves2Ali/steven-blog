**在用 JavaScript 工作时，我们经常和条件语句打交道，这里有5条让你写出更好/干净的条件语句的建议。**

1. 多重判断时使用 Array.includes 
```JavaScript
// bad
function test(fruit) { 
  if (fruit == 'apple' || fruit == 'strawberry') {
    console.log('red') 
  } 
} ​
```
```JavaScript
// good
function test(fruit) {
  const redFruits = ['apple', 'strawberry', 'cherry', 'cranberries']
  if(redFruits.includes(fruit)) { 
    console.log('red')
  }
}
```

2. 更少的嵌套，尽早 return
```javascript
// bad
function test(fruit, quantity) {
    const redFruits = ['apple', 'strawberry', 'cherry', 'cranberries'] 
    // 条件 1: fruit 必须有值  
    if (fruit) {  
      // 条件 2: 必须是red的  
      if (redFruits.includes(fruit)) {  
        console.log('red') 
        // 条件 3: quantity大于10  
        if (quantity > 10) {  
          console.log('big quantity')  
        } 
      }  
    } else {  
      throw new Error('No fruit!')  
    } 
	} 
```
```javascript
// good
function test(fruit, quantity) {
        const redFruits = ['apple', 'strawberry', 'cherry', 'cranberries']
        // 1: 尽早抛出错误 
        if(!fruit) throw new Error('No fruit!') 
        // 条件 2: 当水果不是红色时停止继续执行 
        if (!redFruits.includes(fruit)) return console.log('red') 
        // 条件 3: 必须是大质量的
        if (quantity > 10) { 
          console.log('big quantity') 
        } 
}
```
+ 通过倒置判断条件2，我们避免了嵌套代码。这个技巧在我们需要很长的逻辑判断时是非常有用的，特别是我们希望在条件不满足时候能够停止下来进行判断。但是也会存在问题，没有嵌套是不是比之前两层条件嵌套更好，可读性更高？ 在两层嵌套的情况下：代码比较短而且直接，包含if嵌套的更加清晰 ，倒置判断条件可能家中思考负担 因此，应该尽力减少嵌套和尽早return，但不要过度。  

3. 使用默认参数和解构
+ 使用默认参数和结构 在JavaScript中我们总是需要检查 null / undefined 的值和指定默认值
```javascript
// bad
function test(fruit, quantity) {
  if (!fruit) return 3 
  // 如果 quantity 参数没有传入，设置默认值为1 
  const q = quantity || 1 
  console.log(We have ${q} ${fruit}!) 
}
//test 
test('banana') // We have 1 banana! 11 
test('apple', 2) // We have 2 apple! 
```
如果你熟悉es6的话，通过声明 默认函数参数 来消除变量 q
```javascript
// good
function test(fruit, quantity = 1) {
  if (!fruit) return 3 
  // 如果 quantity 参数没有传入，设置默认值为1 
  const q = quantity
  console.log(We have ${q} ${fruit}!) 
}
//test 
test('banana') // We have 1 banana! 11 
test('apple', 2) // We have 2 apple! 
```

+ 如果fruit是一个object的情况
```javascript
// bad
function test(fruit) {
        // 当值存在时打印 fruit 的值 
  if (fruit && fruit.name) { 
     console.log (fruit.name) 
  } else { 
    console.log('unknown') 
  } 
}
test(undefined) // unknown 
test({ }) // unknown 
test({ name: 'apple', color: 'red' }) // apple
```
我们想知道fruit对象中可能存在name属性。否则我们将打印unknown。我们可以通过默认参数以及结构从而避免判断条件 fruit && fruit.name
```javascript
// good
// 解构 - 仅仅获取 name 属性
// 为其赋默认值为空对象
function test({ name } = {}) {
   console.log(name || 'unknown')
}
test(undefined) // unknown 
test({ }) // unknown 
test({ name: 'apple', color: 'red' }) // apple
```
由于我们只需要声明name属性，我们可以用 {name} 解构参数，然后我们就能使用 name 代替fruit.name 我们也需要 声明空对象 {} 作为默认值。如果我们不这么做，当执行 test(undefined)时，你将得到一个无法对undefined或null解构的错误。因为undefined中没有name属性。

4. 倾向于遍历对象而不是 Switch 语句
```javascript
// bad
function test(color) {
    // 使用条件语句来寻找对应颜色的水果 
    switch (color) {
    case 'red':
        return ['apple', 'strawberry']
    case 'yellow':
        return ['banana', 'pineapple']
    case 'purple':
        return ['grape', 'plum']
    default:
        return []
    }
}
test(null) // [] 
test('yellow') // ['banana', 'pineapple']
```
上面的代码看起来是没有问题的，但是显得累赘，而且不够清晰。用对象遍历实现相同的结果，语法看起来更简洁：
```javascript
// good
const fruitColor = {
    red: ['apple', 'strawberry'],
   	yellow: ['banana', 'pineapple'],
    purple: ['grape', 'plum']
}

function test(color) {
    return fruitColor[color] || []
}
```
+ 我们是不是应该禁止使用 switch 呢？答案是否定的，我们尽可能使用对象遍历，并不需要严格遵守它，而是使用对当前的场景更有意义的方式

5. 对 所有/部分 判断使用 Array.every & Array.some
+ 利用JavaScript Array 的内置方法减少代码行数，下面例子中，我们要想要检查是否所有水果都是红色：
```JavaScript
// bad
const fruits = [{
    name: 'apple',
    color: 'red'
}, {
    name: 'banana',
    color: 'yellow'
}, {
    name: 'grape',
    color: 'purple'
}]

function test() {
  let isAllRed = true;
  // 条件：所有水果都是红色 
  for (let f of fruits) {
      if (!isAllRed) break;
      isAllRed = (f.color == 'red');
  }
  console.log(isAllRed); // false
}
```
我们可以使用 Array.every 减少代码行数
```JavaScript
// good
const fruits = [{
    name: 'apple',
    color: 'red'
}, {
    name: 'banana',
    color: 'yellow'
}, {
    name: 'grape',
    color: 'purple'
}]

function test() {
    const isAllRed = fruits.every(f => f.color == 'red');
    console.log(isAllRed); // false  
}
```
现在是更加简洁了，相同的方式，我们想测试是否存在红色的水果，我们可以使用 Array.some 一行代码实现
```JavaScript
// better
const fruits = [{
    name: 'apple',
    color: 'red'
}, {
    name: 'banana',
    color: 'yellow'
}, {
    name: 'grape',
    color: 'purple'
}]

function test() {
    // 条件：任何一个水果是红色 9 
    const isAnyRed = fruits.some(f => f.color == 'red');
    console.log(isAnyRed) // true 
}
```