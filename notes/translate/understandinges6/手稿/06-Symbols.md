# Symbols和Symbol属性
Symbols是ECMAScript6中引入的一个原始类型，加入到已有的字符串、数字、布尔、`null`和`undefined`原始类型。Symbols开始被用作创建私有对象成员的一个方法，这是JavaScript开发人员一直以来都想要的特性。在Symbols之前，任何使用字符串名的属性都很好访问，无论它的名字是否很晦涩，“私有名”特性旨在让开发人员使用非字符串属性名。此时，常规的检测这些私有名的技术无法生效。
私有名提案最终演化为ECMAScript6中的Symbols，本章将教你如何有效地使用Symbols。虽然实现细节维持一致（即为属性名增加了非字符串值），私有的目标被丢弃。取而代之，symbol属性与其它对象属性分开分类。
## 创建Symbols
Symbols在JavaScript原始类型中是独特的，它没有一个特定的字面量形式，如布尔类型的`true`和数字类型的`42`。你可以通过使用全局`Symbol`函数来创建一个symbol，如下例所示：
```
let firstName = Symbol();
let person = {};

person[firstName] = "Nicholas"；
console.log(person[firstName]);  // "Nicholas"
```
此处，创建了一个symbol`firstName`来为`person`对象赋一个新的属性。这个symbol在你每次想要访问同一个属性的时候被用到。合适地为这个symbol变量命名是一个很好的方法，这样你可以轻易地判断这个symbol代表什么。
因为symbol是原始值，调用`new Symbol()`将抛出一个错误。你也可以通过`new Object(yourSymbol)`来创建一个Symbol的实例，但是这种能力什么时候可以使用仍未可知。
`Symbol`函数同时也接受一个描述symbol的可选参数。描述滋生不能被用来访问属性，但是可以用来调试。例如：
```
let firstName = Symbol("first name");
let person = {};

person[firstName] = "Nicholas";
console.log("first name" in person);  // false
console.log(person[firstName]);  // "Nicholas"
console.log(firstName);  // "Symbol(first name)"
```
一个symbol的描述被存在内部属性`[[Description]]`中。这个属性在symbol的`toString()`方法被调用时被读取，无论是直接或间接调用。`firstName`symbol的`toString()`方法被本例的`console.log()`间接调用，因此描述被日志打印出来。无法直接使用代码访问`[[Description]]`。我建议总是在提供一个描述，从而使得读取和调试symbol更方便。
## 识别Symbols
由于symbols为原始值，你可以使用`typeof`操作符来判断一个变量是否包含symbol。ECMAScript6扩展了`typeof`来在用在symbol的时候返回`"symbol"`。例如：
```
let symbol = Symbol("test symbol");
console.log(typeof symbol);   // "symbol"
```
虽然有其它间接地判断一个变量是否为symbol的方法，`typeof`操作符是最准确的且被推荐的方法。
## 使用Symbols
你可以在任何你想使用一个计算属性名的地方使用symbol。你已经在本章中看到使用symbol的方括号表示法，但你也可以使用`Object.defineProperty()`和`Object.defineProperties()`调用来在计算对象字面量属性中使用symbol，如：
```
// 使用一个计算对象字面量属性
let person = {
  [firstName]: "Nicholas"
};

// 使得该属性为只读
Object.defineProperty(person, firstName, { writable: false });

let lastName = Symbol("last name");

Object.defineProperties(person, {
  [lastName]: {
    value: "Zakas",
    writable: false
  }
});

console.log(person[firstName]);  // "Nicholas"
console.log(person[lastName]);   // "Zakas"
```
该例首先使用一个计算对象字面量属性来创建`firstName`symbol属性。下一行将该属性设置为只读。接着，通过`Object.defineProperties()`方法创建了一个只读的`lastName`symbol属性。一个计算对象字面量属性再次被使用，不过这次是作为`Object.defineProperties()`函数调用的第二个参数的一部分。
虽然symbol可以在任何允许计算属性名的地方使用，你也可能需要一个在不同代码段中共享这些symbol来高效使用它们的系统。
## 共享Symbols
你可能会想要在你代码中不同地方使用相同的symbol。例如，假如在你的应用中有两个不同的对象类型，它们可能使用同一个symbol属性来表示一个特殊的标识符。在文件间或者大范围代码内追踪symbol是困难且容易出错的。这也是为什么ECMAScript提供了一个可以在任何时候访问的全局symbol注册。
当你想要创建一个用于共享的symbol时，使用`Symbol.for()`方法而不是调用`Symbol()`方法。`Symbol.for()`方法接受一个单独的参数，它为你想创建的symbol的字符串标识符。该参数也被用作该symbol的描述。例如：
```
let uid = Symbol.for("uid");
let object = {};

object[uid] = "12345";

console.log(object[uid]);  // "12345"
console.log(uid);  // "Symbol(uid)"
```
`Symbol.for()`方法首先搜索全局symbol注册中是否已经存在键为`"uid"`的symbol。如果有，该方法返回已经存在的symbol。如果没有，那么创建一个新的symbol并使用指定键注册全局symbol，并返回新的symbol。这说明使用相同的键多次调用`Symbol.for()`将返回同一个symbol，如下：
```
let uid = Symbol.for("uid");
let object = {
  [uid]: 12345
};

console.log(object[uid]);  // "12345"
console.log(uid);   // "Symbol(uid)"

let uid2 = Symbol.for("uid");

console.log(uid === uid2);   // true
console.log(object[uid2]);   // "12345"
console.log(uid2);           // "Symbol(uid)"
```
在这个例子中，`uid`和`uid2`包含相同的symbol，所以它们可以相互交换。第一次`Symbol.for()`调用创建了symbol，第二次调用从全局symbol注册中获得symbol。
共享symbol的另一个独特之处在于你可以通过调用`Symbol.keyFor()`来获得关联全局symbol注册中symbol的键。例如：
```
let uid = Symbol.for("uid");
console.log(Symbol.keyFor(uid));  // "uid"

let uid2 = Symbol.for("uid");
console.log(Symbol.keyFor(uid2));  // "uid"

let uid3 = Symbol("uid");
console.log(Symbol.keyFor(uid3));  // undefined
```
注意`uid`和`uid2`都返回`"uid"`键。symbol uid3不在全局symbol注册中，因此没有和它相关联的键，`Symbol.keyFor()`返回`undefined`。
全局symbol注册是一个共享环境，正如全局作用域。这表明你无法对什么已经或者并不存在在该环境中做假设。使用symbol键命名空间可以在使用第三方组件时减少命名相同冲突。例如，jQuery代码可能在使用`"jquery."`作为所有键的前缀，如键`"jquery.element"`或类似的。
## Symbol强制转换
类型转换是JavaScript一个独特的部分，该语言在转换一个数据类型为另一个的能力上有很大的灵活性。然而Symbols在强制转换方面并不灵活，因为其它类型缺少对symbol的逻辑相等关系。具体来说，symbol无法被强制转换为字符串或数字，因此，它们不会被意外用作被期许为symbol的属性。
本章中的例子使用`console.log()`来演示symbol的输出，这因`console.log()`调用symbol的`String()`来创建有效输出而得以实现。你可以直接使用`String()`来获得相同的结果。例如：
```
let uid = Symbol.for("uid"),
    desc = String(uid);

console.log(desc);   // "Symbol(uid)"
```
`String()`函数调用`uid.toString()`并返回该symbol的字符串描述。如果你试图将symbol与一个字符串直接相连，那么将抛出一个错误：
```
let uid = Symbol.for("uid"),
    desc = uid + "";    // 错误
```
连接`uid`和一个空字符串要求`uid`首先被强制转换为一个字符串。当该强制转换被检测到时将抛出一个错误来避免这样使用它。
类似地，你无法强制转换symbol为一个数字。所有的算数操作都将在symbol上使用时抛出错误。例如：
```
let uid = Symbol.for("uid"),
    sum = uid / 1;   // 错误
```
该例试图使用uid除1，这将导致一个错误。无论是什么算数操作（逻辑操作不会抛出错误，因为所有symbol都被认为是`true`，正如JavaScript中的其它非空值）都将抛出错误。
## 获取Symbol属性
`Object.keys()`和`Object.getOwnPropertyNames()`方法可以获取一个对象上的所有属性名称。前者返回所有可枚举属性名，后者返回所有属性名而不管它是否可枚举。然而为了维持ECMAScript5功能，它们都不返回symbol属性。取而代之，ECMAScript6中新增了`Object.getOwnPropertySymbols()`方法来允许你从对象上获取属性symbols。
`Object.getOwnPropertySymbols()`的返回值是自身属性symbols的一个数组。例如：
```
let uid = Symbol.for("uid");
let object = {
  [uid]: "12345"
};

let symbols = Object.getOwnPropertySymbols(object);

console.log(symbols.length);      // 1
console.log(symbols[0]);          // "Symbol(uid)"
console.log(object[symbols[0]]);  // "12345"
```
在这段代码中，`object`具备一个名为`uid`的symbol属性。`Object.getOwnPropertySymbols()`返回的数组是一个只含该symbol的数组。
所有对象开始都没有自身的symbol属性，但是对象可以从它们的原型上继承symbol属性。ECMAScript6预定义了几个这样的属性，使用所谓的众所周知的symbol来实现。
## 使用众所周知的Symbols暴露内部操作
ECMAScript5的一个中心主题在于暴露和定义JavaScript中的一些“魔法”部分，即一些开发人员当时无法模仿的部分。ECMAScript6通过暴露更多该语言之前的内部逻辑来延续该传统，主要使用symbol原型属性来定义特定对象的基本行为。
ECMAScript6有一些被称为*众所周知的symbols*的预定义symbols，它们代表了JavaScript中一些之前被认为是仅内部操作的常见行为。每个众所周知的symbol有一个`Symbol`对象上的属性来表示，例如`Symbol.create`。
这些众所周知的symbols为：
* `Symbol.hasInstance` - 一个被`instanceof`用来判断对象继承的方法。
*` Symbol.isConcatSpreadable` - 一个布尔值，标识`Array.prototype.concat()`应该在当一个集合被作为参数传入`Array.prototype.concat()`时打平该集合中的元素。
* `Symbol.iterator` - 一个返回迭代器的方法。（迭代器在第八章中。）
* `Symbol.match` - 一个被`String.prototype.match()`用来比较字符串的方法。
* `Symbol.replace` - 一个被`String.prototype.replace()`用来替换子字符串的方法。
* `Symbol.search` - 一个被`String.prototype.search()`用来定位子字符串的方法。
* `Symbol.species` - 用来创建派生对象的构造函数。（派生对象在第九章中）。
* `Symbol.split` - 一个被`String.prototype.split()`用来拆分字符串的方法。
* `Symbol.toPrimitive` - 一个返回对象的原始值表示的方法。
* `Symbol.toStringTag` - 一个被`Object.prototype.toString()`用来创建对象描述的字符串。
* `Symbol.unscopables` - 一个属性为不应包含在with语句中的对象属性名称的对象。
一些常用的众所周知的symbols在接下来几节中讨论，其它的将在本书的剩余部分穿插讨论，以保证上下文的正确性。
使用众所周知的symbol重写一个定义的方法将改变一个普通对象为奇异对象，因为它改变了一些默认内部行为。它并不会对你的代码造成实际上的影响，它只是改变了标准描述对象的方式。
### Symbol.hasInstance属性
### Symbol.isConcatSpreadable
### Symbol.match, Symbol.replace, Symbol.search, Symbol.split
### Symbol.toPrimitive方法
### Symbol.toStringTag
#### 一个识别问题的解决方案
#### ECMAScript6解决方案
### Symbol.unscopables
## 总结
