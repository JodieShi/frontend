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
IEEE 754只能准确表示-2^53 和 2^53之间的数字，而在这个“安全”范围外，二进制表示最终被重用于多个数值。这表明JavaScript只能在问题变得明显之前安全地表示IEEE 754范围内的数字。例如，考虑以下代码：
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
因此，ECMAScript6位`Math`对象增加了几个方法来提升常规算术计算的速度。提升常规计算也提升了执行很多计算的应用的全局速度，如图形程序。新的方法列举如下：
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
ECMAScript6提供了比之前JavaScript版本更好的Unicode支持，它也改变了哪些字符可以被用作标识符。在ECMAScript5中，已经可以使用Unicode转义序列作为标识符。例如：
```js
// 在ECMAScript5和6中有效
var \u0061 = "abc";

console.log(\u0061);       // "abc"

// 等效于
console.log(a);            // "abc"
```
本例中，在`var`语句之后，你可以使用`\u0061`或`a`来访问变量。在ECMAScript6中，你也可以使用Unicode码点转义序列作为标识符，如下：
```js
// 在ECMAScript5和6中有效
var \u{61} = "abc";

console.log(\u{61});       // "abc"

// 等效于
console.log(a);            // "abc"
```
本例只是将`\u0061`替换成它相等的码点。否则，它与前一个例子将完全相同。
此外，ECMAScript6根据[Unicode Standard Annex #31: Unicode Identifier and Pattern Syntax](http://unicode.org/reports/tr31/)指定了有效的标识符，它给出了如下规则：
1. 首字符必须是`$`，`_`或任何一个具有`ID_Start`派生核心属性的Unicode符号
2. 每个随后的字符必须是`$`，`_`，`\u200c`（一个零宽度的非连接符），`\u200d`（一个零宽度的连接符）或任何一个具有`ID_Continue`派生核心属性的Unicode符号。

`ID_Start`和`ID_Continue`派生核心属性在Unicode标识符和模式语法中被定义，作为识别适合作为标识符（如变量和作用域）使用的符号。该标准并非JavaScript专用。
## 形式化`__proto__`属性
即使在ECMAScript5已经结束之前，多个JavaScript引擎已经实现了一个名为`__proto__`的自定义属性，它可以被用作获取和设置`[[Prototype]]`属性。因此`__proto__`是`Object.getPrototypeOf()`和`Object.setPrototypeOf()`方法的早期先导。指望所有JavaScript引擎移除这个属性并不实际（已有广泛使用的JavaScript库使用了`__proto__`），因此ECMAScript6也形式化了`__proto__`行为。但是形式化出现在ECMA-262的附录B中，与以下警告一起：
> 这些特性并不认为是ECMAScript核心语言的一部分。编程人员在编写新ECMAScript代码时不应该使用或假设这些特性或行为的存在。不鼓励ECMAScript具体实现去实现这些特性，除非这些实现是网页浏览器的一部分或者需要运行网页浏览器遇到的相同遗留ECMAScript代码。

ECMAScript标准推荐使用`Object.getPrototypeOf()`和`Object.setPrototypeOf(`)，因为`__proto__`具有以下特性：
1. 你只能在一个对象字面量中一次指定`__proto__`。如果你指定两个`__proto__`属性，那么将抛出一个错误。这是唯一一个具有该限制的对象字面量属性。
2. `["__proto__"]`计算属性按一个常规属性工作，它不会设置或返回当前对象的属性。所有与对象字面量属性相关的规则都应用这个形式，不同于有异常情况非计算形式。

虽然你应该避免使用`__proto__`属性，标准定义它的方法却很有意思。在ECMAScript6引擎中，`Object.prototype.__proto__`被定义为一个访问器属性，它的get方法调用`Object.getPrototypeOf()`而set方法调用`Object.setPrototypeOf(`)方法。这使得使用`__proto__`和`Object.getPrototypeOf()`/`Object.setPrototypeOf(`)没有实际差别，除了`__proto__`允许你直接设置一个对象字面量的原型。下面为它如何工作：
```js
let person = {
  getGreeting() {
    return "Hello";
  }
};

let dog = {
  getGreeting() {
    return "Woof";
  }
};

// 原型为person
let friend = {
  __proto__: person
};
console.log(friend.getGreeting());                       // "Hello"
console.log(Object.getPrototypeOf(friend) === person);   // true
console.log(friend.__proto__ === person);                // true

// 设置原型为dog
friend.__proto__ = dog;
console.log(friend.getGreeting());                       // "Woof"
console.log(Object.getPrototypeOf(friend) === dog);      // true
console.log(friend.__proto__ === dog);                   // true
```
无需调用`Object.create()`来创建`friend`对象，本例创建了一个标准对象字面量，并赋了一个值给`__proto__`属性。而当使用`Object.create()`方法来创建一个对象时，你需要为任何额外的对象属性指定完整的属性描述符。