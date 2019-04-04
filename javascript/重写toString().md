####下面代码中 a 在什么情况下会打印 1？
```javascript
var a = ?;
if(a == 1 && a == 2 && a == 3){
 	console.log(1);
}
```
+ 这个问题考引用类型在比较运算符时候,隐式转换会调用本类型toString或valueOf方法
```javascript
var a = {
  i: 1,
  toString() {
    return a.i++;
  }
}
if( a == 1 && a == 2 && a == 3 ) {
  console.log(1);
}
```
**解析**
执行 a == 1时，会调用a这个对象默认值1，并且执行完之后会执行i++，那么a就等于2，以此类推。但是这个判断语句必须按照顺序判断。如果出现a == 1 && a == 4 && a == 3这样的情况是不是成功进判断的。