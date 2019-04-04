```javascript
function deepClone(obj) {
  // 初始化obj
	if(!obj || 'object' !== typeof obj) { 
		return obj
  }
  // 判断obj是否存在pop方法，pop是数组独有的方法
  var c = 'function' === typeof obj.pop ? [] : {}
	for(let i in obj) {
		c[i] = typeof obj[i] === 'object' ? deepClone(obj[i]) : obj[i]
	}
	return c
}
```