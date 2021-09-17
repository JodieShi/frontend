# 附录B：理解ECMAScript 7 (2016)
ECMAScript6的发展使用了大概四年，在这之后，TC-39认为如此长的发展进程是不可持续的。因此，他们改为每年发布一次来确保新的语言特性可以快速进入开发阶段。
更频繁的发布表明每个新ECMAScript版本都会比ECMAScript6有更少的新特性。为了表明这种改变，标准的新版本不再突出版本号，而是引用该标准发布的年份。因此，ECMAScript6也被称为ECMAScript 2015，ECMAScript7的正式名称为ECMAScript 2016。TC-39希望为未来所有的ECMAScript版本使用基于年份的命名。
## 指数操作符
ECMAScript 2016中唯一引入的JavaScript语法变化是*指数操作符*，它是一个将指数应用到基数上的数学操作。JavaScript已有`Math.pow()`方法来执行指数操作，但是JavaScript也是唯一需要方法而不是正式操作符的语言之一。（一些开发人员认为操作符更易读和推理。）
指数操作符是二个星号（`**`），左操作数是基数，而右操作数是指数。例如：
```js
let result = 5 ** 2;

console.log(result);                         // 25
console.log(result === Math.pow(5, 2));      // true
```
该例计算5^2 ，等于25。你也可以使用Math.pow()来获得相同的结果。
### 操作符顺序
指数操作符在JavaScript的所有的二元操作符中有最高的优先级（一元操作符比`**`有更高的优先级）。这表明它在任何组合操作中将被首先应用，如下例所示：
```js
let result = 2 * 5 ** 2;
console.log(result);       // 50
```
先计算5^2 ，结果接着被乘2，最后结果是50。
### 操作数限制
指数操作符有一些其它操作符上没有的特定限制。指数操作符左边的操作数不能是一个除`++`或`--`外的一元表达式。例如，下面为无效语法：
```js
// 语法错误
let result = -5 ** 2;
```
本例中的`-5`是一个语法错误，因为操作顺序不清楚。`-`是只用到`5`上还是`5 ** 2`表达式的结果上？不允许在指数操作符左侧使用一元表达式消除了这种模糊性。为了明确意图，你需要将`-5`或`5 ** 2`用括号括起来，如下：
```js
// ok
let result = -(5 ** 2);     // 等于-25

// also ok
let result = (-5) ** 2;     // 等于25
```
如果你将括号括在表达式上，`-`将被用在整个结果上。而当括号括住`-5`时，很明显你想将`-5`升为二次幂。
你无需在指数操作符左侧使用`++`或`--`时使用括号，因为这些操作符对它们的操作符上有完全明确定义的行为。前缀`++`或`--`咋其它操作符执行前改变操作数，后缀版本直到整个表达式被计算完才应用改变。这些用例在此操作符的左侧都是安全的，如以下代码所示：
```js
let num1 = 2,
    num2 = 2;

console.log(++num1 ** 2);        // 9
console.log(num1);               // 3

console.log(num2-- * 2);         // 4
console.log(num2);               // 1
```
在本例中，`num1`在指数操作符计算前自增1，因此`num1`变为3，操作的结果为9。对于`num2`，值在指数操作时依旧为2，接着减为1。
## Array.prototype.includes()方法
你可能想起ECMAScript6加入了`String.prototype.includes()`来检查给定字符串上是否存在某子字符串。开始，ECMAScript6也想引入一个`Array.prototype.includes()`方法来继续将字符串和数组相似对待的趋势。但是`Array.prototype.includes()`的标准直到ECMAScript6结束时都不完整，因此`Array.prototype.includes()`最终定在ECMAScript 2016中。
### 如何使用Array.prototype.includes()
`Array.prototype.includes()`方法接收两个参数：要搜索的值和一个可选的开始搜索的索引。当第二个参数被提供时，`includes()`从该索引开始匹配。（默认的开始索引是`0`。）当值在数组中被找到时返回值为`true`，否则返回`false`。例如：
```js
let values = [1, 2, 3];

console.log(values.includes(1));           // true
console.log(values.includes(0));           // false

// 从索引2开始搜索
console.log(values.includes(1, 2));        // false
```
此处，调用`values.includes()`对`1`返回`true`，而对`0`返回`false`，因为`0`不在数组中。当使用第二个参数来从索引2开始搜索时（该位置为值`3`），`values.includes()`方法返回`false`，因为数字`1`从索引位置2到数组结尾都不存在。
### 值比较
`includes()`方法执行的值比较使用`===`操作符，唯一例外是`NaN`被认为与`NaN`相等，虽然`NaN === NaN`等于`false`。这与`indexOf()`方法的行为不同，`indexOf()`的比较严格使用`===`。考虑以下代码来观察这种区别：
```js
let values = [1, NaN, 2];

console.log(values.indexOf(NaN));            // -1
console.log(values.includes(NaN));           // true
```
`values.indexOf()`方法对`NaN`返回`-1`，虽然`NaN`包含在`values`数组中。另一方面，`values.includes()`对`NaN`返回`true`，因为它的比较操作符使用一个不同值。
当你只是想检查一个值是否存在于数组中而无需知道索引时，由于`NaN`被`includes()`和`indexOf()`方法处理的差异，我建议使用`includes()`。如果你需要知道值在数组中的位置，那么你需要使用`indexOf()`方法。
该实现的另一个怪异之处在于`+0`和`-0`被认为是相等的。此时，`indexOf()`和`includes()`是相同的：
```js
let values = [1, +0, 2];

console.log(values.indexOf(-0));         // 1
console.log(values.includes(-0));        // true
```
这里，`indexOf()`和`includes()`在`-0`被传入时找到了`+0`，因为两个值被认为相等。注意到这和`Object.is()`方法的行为不同，它认为`+0`和`-0`是不同的值。
## 函数作用域严格模式变化
当ECMAScript5中引入严格模式时，该语言比在ECMAScript6中要简单地多。除此之外，ECMAScript6依旧允许使用`"use strict"`指令来在全局作用域（这将使得所有代码运行在严格模式下）或函数作用域中来指定严格模式。后者最终成为ECMAScript6中的一个问题，因为参数可被定义的方式更加复杂，尤其是与解构和默认参数值一起使用时。为了理解这个问题，考虑下面的代码：
```js
function doSomething(first = this) {
  "use strict";

  return first;
}
```
此处，命名参数`first`被赋了一个默认值，`this`。你认为`first`的值应该是什么？ECMAScript6标准指导JavaScript引擎在本例中将参数作为正在严格模式下运行，因此`this`应该等于`undefined`。然而，当`"use strict"`出现在函数内部时，实现参数在严格模式下运行变得非常不一样，因为参数默认值也可能是函数。这种困难导致多数JavaScript引擎并没实现这种特性（因此`this`应该与全局对象相等）。
由于实现的困难，ECMAScript 2016使得在参数被解构或有默认值的函数内部使用`"use strict"`违法。只有*简单参数列表*，即那些不包含解构或默认值的，被允许在`"use strict"`出现在函数内部时被使用。下面有一些例子：
```js
// ok - 使用一个简单参数列表
function okay(first, second) {
  "use strict";

  return first;
}

// 语法错误
function notOkay1(first, second=first) {
  "use strict";

  return first;
}

// 语法错误
function notOkay2({first, second}) {
  "use strict";

  return first;
}
```
你可以在使用简单参数列表时继续使用`"use strict"`，这是`okay()`按照你预想的方式工作的原因（与它在ECMAScript5中一样）。`notOkay1()`函数将抛出一个语法错误，因为无法在使用默认参数值的函数中使用`"use strict"`。类似地，`notOkay2()`函数也将抛出一个语法错误，因为无法在使用解构参数的函数中使用`"use strict"`。
总之，该变化移除了JavaScript开发者的一个迷惑点和JavaScript引擎的一个实现问题。
