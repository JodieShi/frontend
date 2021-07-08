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
## Symbol强制转换
## 获取Symbol属性
## 暴露已知Symbols的内部操作
### Symbol.hasInstance属性
### Symbol.isConcatSpreadable
### Symbol.match, Symbol.replace, Symbol.search, Symbol.split
### Symbol.toPrimitive方法
### Symbol.toStringTag
#### 一个识别问题的解决方案
#### ECMAScript6解决方案
### Symbol.unscopables
## 总结
