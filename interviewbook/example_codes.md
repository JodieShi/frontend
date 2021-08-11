# some example codes for common interview questions
## 用apply，call实现bind
```js
// 简单实现
Function.prototype.bind = function() {
  let ctx = this

  let bindingThis = arguments[0]
  let args = [].slice.call(arguments, 1)

  return function() {
    args = [].concat.call(args, arguments)
    ctx.apply(bindingThis, args)
  }
}
```
```js
// 一个复杂的版本
Function.prototype.bind = function(oThis) {
  if (typeof this !== 'function') {
    throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable')
  }

  var aArgs = Array.prototype.slice.call(arguments, 1),
      fToBind = this,
      fNOP = function() {},
      fBound = function() {
        return fToBind.apply(this instanceof fBound ? this : oThis,
                             aArgs.concat(Array.prototype.slice.call(arguments)));
      };

  if (this.prototype) {
    fNOP.prototype = this.prototype; 
  }

  fBound.prototype = new fNOP();
  return fBound;
};
```
## 用reduce实现map
```js
Array.prototype.myMap = function(fn, mapThis) {
  return this.reduce((prev, cur, curIndex, arr) => {
    prev.push(fn.call(mapThis, cur, curIndex, arr))
    return prev
  }, [])
}
```
## 数组flat
```js
// reduce实现
Array.prototype.flat = function() {
  const flatFn = arr => {
    return arr.reduce((prev, cur) => {
      return prev.concat(Array.isArray(cur) ? flatFn(cur) : [cur])
    }, [])
  }

  return flatFn.call(null, this)
}
```
```js
function* flat(arr, num) {
  for (const item of arr) {
    if (Array.isArray(item) && num > 0) {   // num > 0
      yield* flat(item, num - 1);
    } else {
      yield item;
    }
  }
}
```
## 节流
```js
function throttle(fn, delay=200) {
  let timer = null
  return function() {
    if (timer) {
      return
    }

    timer = setTimeout(() => {
      fn.apply(this, arguments)
      timer = null
    }, delay)
  }
}
```
## 防抖
```js
function debounce(fn, delay=200) {
  let timer = null
  return function() {
    if (timer) {
      clearTimeout(timer)
    }

    timer = setTimeout(() => {
      fn.apply(this, arguments)
      timer = null
    }, delay);
  }
}
```
