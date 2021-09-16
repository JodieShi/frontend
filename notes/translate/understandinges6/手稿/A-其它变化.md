# 附录A：小变化
除了本书中已经涵盖的其它主要变化，ECMAScript6也做出了一些比较小但仍有助于改进JavaScript的变化。这些变化包括使得整数更易于使用，增加新的计算模块，Unicode标识符调整以及形式化`_proto_`属性。我将在本附录中描述它们。
## 使用整数
JavaScript使用IEEE 754编码系统来表示整数和浮点数，这在过去的许多年里造成了很多困惑。该语言努力去确保开发人员无需关心数字编码的细节，但是问题仍然时不时发生。ECMAScript6试图通过使得整数更易于识别和使用来解决该问题。
### 识别整数
首先，ECMAScript6增加了`Number.isInteger()`方法，它决定JavaScript中的一个值是否表示一个整数。虽然JavaScript使用IEEE 754来表示两种数字类型，浮点数和整数按不同的方式被存储。`Number.isInteger()`方法利用了这一点，当对某个值调用该方法时，JavaScript引擎查看该值的底层表示来决定它是否是一个整数。这表明看起来像浮点数的数字实际上也可能被作为整数存储并导致`Number.isInteger()`返回`true`。例如：
```js
console.log(Number.isInteger(25));    // true
console.log(Number.isInteger(25.0));  // true
console.log(Number.isInteger(25.1));  // false
```
在本代码中，`Number.isInteger`为`25`和`25.0`都返回`true`，虽然后者看起来像是一个浮点数。在JavaScript中，简单地向数字添加小数点不会自动使其成为浮点数。由于`25.0`实际上就是`25`，它被作为一个整数存储。然而数字`25.1`被作为浮点数存储，因为有一个分数值。
### 安全整数
IEEE 754只能准确表示-2^53和2^53之间的数字，而在这个“安全”范围外，二进制表示最终被重用于多个数值。这表明JavaScript只能在问题变得明显之前安全地表示IEEE 754范围内的数字。例如，考虑以下代码：
```js
console.log(Math.pow(2, 53));           // 9007199254740992
console.log(Math.pow(2, 53));           // 9007199254740992
```
本例并无拼写错误，但是两个不同的数字被同一个JavaScript整数表示。该影响在值越超出安全范围时就越明显。
ECMAScript6引入了`Number.isSafeInteger()`方法来更好地标识该语言可以准确表示的整数。它也增加了`Number.MAX_SAFE_INTEGER`和`Number.MIN_SAFE_INTEGER`属性来表示整数范围的上下边界。`Number.isSafeInteger()`方法确保了值是一个整数并落在整数值的安全范围内，如下例中所示：
```js
var inside = Number.MAX_SAFE_INTEGER,
    outside = inside + 1;

console.log(Number.isInteger(inside));          // true
console.log(Number.isSafeInteger(inside));      // true

console.log(Number.isInteger(outside));         // true
console.log(Number.isSafeInteger(outside));     // false
```
数字`inside`是最大的安全整数，所以它对`Number.isInteger()`和`Number.isSafeInteger()`方法都返回`true`。`outside`是第一个有问题的整数值，虽然它是个整数但并不被认为是安全的。
多数时候，在JavaScript中进行整数算术或比较时，你只想处理安全整数，因此，使用`Number.isSafeInteger()`作为输入校验的一部分将是一个好主意。
## 新Math方法
对于游戏和图形的重视使得ECMAScript6在JavaScript中包含了类型化数组，也实现了JavaScript引擎更高效地处理数学计算。但是像asm.js这样的优化策略工作在一个JavaScript子集上来提升性能，需要更多的信息来以尽可能快的方式执行计算。例如，知道数字是否应该被认为是32位整数或64位浮点数对于基于硬件的操作来说很重要，它比基于软件的操作更快。
因此，ECMAScript6位`Math对`象增加了几个方法来提升常规算术计算的速度。提升常规计算也提升了执行很多计算的应用的全局速度，如图形程序。新的方法列举如下：
* `Math.acosh(x)` 返回`x`的反双曲余弦值。
* `Math.asinh(x)` 返回`x`的反双曲正弦值。
* `Math.atanh(x)` 返回`x`的反双曲正切值。
* `Math.cbrt(x)` 返回`x`的立方根。
* `Math.clz32(x)` 返回`x`的32位整数表示中前导零位的数量。
* `Math.cosh(x)` 返回`x`的双曲余弦值。
* `Math.expm1(x)` 返回`x`的指数函数-1的结果。
* `Math.fround(x)` 返回最接近`x`的单精度浮点数。
* `Math.hypot(...values)` 返回每个参数的平方和的平方根。
* `Math.imul(x, y)` 返回两个参数执行真32位乘法后的结果。
* `Math.log1p(x)` 返回`1+x`的自然对数。
* `Math.log10(x)` 返回`x`的以10为底的对数。
* `Math.log2(x)` 返回`x`的以2为底的对数。
* `Math.sign(x)` 如果`x`为负数则返回-1，如果`x`是+0和-0则返回0，如果`x`为正数则返回1。
* `Math.sinh(x)` 返回`x`的双曲正弦值。
* `Math.tanh(x)` 返回`x`的双曲正切值。
* `Math.trunc(x)` 从一个浮点数中移除小数部分并返回一个整数。

解释每个新方法及它们的工作细节超出了本书的范围。但是如果你的应用需要做常见的计算，请确认在自己实现前先检查新的`Math`方法。
## Unicode标识符
## 形式化`__proto__`属性