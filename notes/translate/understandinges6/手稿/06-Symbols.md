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
每个函数都有一个`Symbol.hasInstance`方法，它决定了一个给定的对象是否为该函数的一个实例。该方法定义在`Function.prototype`上，因此所有的函数都可以继承`instanceof`属性的默认行为，该方法是非可写、非可配置和非可枚举的，以此来确保它不会被错误地重写。
`Symbol.hasInstance`方法接受一个单独的参数：被检查的值。如果值为函数的一个实例那么它返回true。为了理解`Symbol.hasInstance`如何工作，考虑以下代码：
```
obj instanceof Array;
```
该代码等效于：
```
Array[Symbol.hasInstance][obj];
```
ECMAScript6本质上重定义了`instanceof`操作符为这个方法调用的简写语法。现在引入了一个方法调用，你可以改变`instanceof`实际上的工作方式。
例如，假设你想定义一个不声明任何实例对象的函数。你可以通过硬编码`Symbol.hasInstance`的返回值为`false`来实现，例如：
```
function MyObject() {
  // ...
}

Object.defineProperty(MyObject, Symbol.hasInstance, {
  value: function(v) {
    return false;
  }
});

let obj = new MyObject();

console.log(obj instanceof MyObject);   // false
```
你必须使用`Object.defineProperty()`来覆盖一个非可写属性，因此本例中用该方法来用一个新的函数来重写`Symbol.hasInstance`方法。这个新函数总是返回`false`，即使`obj`实际上为`MyObject`类的一个实例，`instanceof`操作符在`Object.defineProperty()`调用后返回`false`。
当然，你也可以检查值并根据任何条件来决定某个值是否应该被认为是一个实例。例如，也许1到100之间的数值可以被认为是某类特殊数字类型的实例。为了实现上述行为，你可能写出下面的代码：
```
function SpecialNumber() {
  // empty
}

Object.defineProperty(SpecialNumber, Symbol.hasInstance, {
  value: function(v) {
    return (v instanceof Number) && (v >= 1 && v <= 100);
  }
});

let two = new Number(2),
    zero = new Number(0);

console.log(two instanceof SpecialNumber);    // true
console.log(zero instanceof SpecialNumber);   // false
```
这段代码定义了一个`Symbol.hasInstance`方法，它在值为`Number`实例且值在1到100之间时返回`true`。因此，`SpecialNumber`将声明`two`为一个实例，即使在`SpecialNumber`函数和`two`变量之间并没有直接的定义关系。注意，`instanceof`的左操作数必须为一个对象来触发`Symbol.hasInstance`调用，因为非对象将导致`instanceof`始终直接返回`false`。
你也可以为所有的内建函数，例如`Date`和`Error`函数来重写默认的`Symbol.hasInstance`属性。然而这并不是被推荐的做法，因为你代码的效果可能出乎意料并且令人迷惑。只在你自己的函数必需时才重写`Symbol.hasInstance`才是明智的做法。
### Symbol.isConcatSpreadable
JavaScript数组有一个`concat()`方法，它被设计用来连接两个数组。下面为它的用法：
```
let colors1 = [ "red", "green" ],
    colors2 = colors1.concat([ "blue", "black" ]);

console.log(colors2.length);   // 4
console.log(colors2);   // ["red","green","blue","black"]
```
该代码连接了一个新的数组到`colors1`末尾并创建了`colors2`，它是一个包含上述两个数组中所有项的单独数组。然而，`concat()`方法也可以接受非数组参数，此时这些参数被简单地加入到数组的末尾。例如：
```
let colors1 = [ "red", "green" ],
    colors2 = colors1.concat([ "blue", "black" ], "brown");

console.log(colors2.length);   // 5
console.log(colors2);   // ["red","green","blue","black","brown"]
```
这里，额外参数`"brown"`被传给`concat()`并成为`colors2`数组的第五项。为什么一个数组参数与一个字符串参数的对待方式不一样？JavaScript标准说数组被自动分割为单独的项，其它的类型不会。在ECMAScript6之前，并没有方法来调节这种行为。
`Symbol.isConcatSpreadable`属性是一个标识函数有一个`length`属性和数值键的布尔值，它的数值键属性值应该独立地添加到`concat()`调用的结果中。与其他众所周知的symbols不同，该symbol属性不会默认出现在任何标准对象上。相反，该symbol是一个增强`concat()`在某些特定类型对象上工作方式的方法，有效缩短了默认行为。你可以定义任何类型，使其工作方式类似于`concat()`中的数组，例如：
```
let collection = {
  0: "Hello",
  1: "world",
  length: 2,
  [Symbol.isConcatSpreadable]: true
};

let messages = [ "Hi" ].concat(collection);

console.log(messages.length);   // 3
console.log(messages);          // [ "Hi", "Hello", "world" ]
```
本例中的`collection`对象被设置为看起来像一个数组：它有一个`length`属性和两个数值键。`Symbol.isConcatSpreadable`属性被设置为`true`来标识属性值应该作为独立项添加到数组中去。当`collection`被传入`concat()`方法时，结果数组在`"Hi"`元素后有独立的`"Hello"`和`"world"`项。
你也可以将数组子类的`Symbol.isConcatSpreadable`设置为`false`来阻止它们被`concat()`调用分割。子类将在第八章中讨论。
### Symbol.match, Symbol.replace, Symbol.search, Symbol.split
字符串和正则表达式在JavaScript中一致有着紧密的联系。特别地，字符串类型有几个接受正则表达式为参数的方法：
* `match(regex)` - 判断给定字符串是否匹配一个正则表达式
* `replace(regex, replacement)` - 使用`replacement`替换正则表达式匹配项
* `search(regex)` - 定位字符串中正则表达式匹配的位置
* `split(regex)` - 在正则表达式匹配处分割字符串为数组
在ECMAScript6之前，这些方法与正则表达式交互的方式是开发者不可见的，因此没有使用开发者自定义对象去模拟正则表达式的方式。ECMAScript6定义了四个与上述方法对应的symbol，有效地将这种本地行为外包给了`RegExp内`置对象。
`Symbol.match`，`Symbol.replace`，`Symbol.search`和`Symbol.split` symbol代表了`match()`，`replace()`，`search()`和`split()`方法在第一个正则表达式参数上调用的方法。这四个symbol属性定义在`RegExp.prototype`上，作为字符串方法将使用的默认实现。
知道这些，你可以创建一个对象，以类似正则表达式的方法与字符串方法一起使用。为了实现它，你可以在代码中使用以下symbol函数：
* `Symbol.match` - 一个接受一个字符串参数并返回匹配项数组或null（无匹配项）的函数。
* `Symbol.replace` - 一个接受一个字符串参数和一个替换字符串并返回字符串的函数。
* `Symbol.search` - 一个接受一个字符串参数并返回匹配项的数字索引或-1（无匹配项）的函数。
* `Symbol.split` - 一个接受一个字符串参数并返回一个包含匹配项处拆分的字符串片段的数组的函数。
在对象上定义这些属性的能力使得你可以创建实现了不用正则表达式的模式匹配的对象并在期待正则表达式的方法中使用它们。下面是一个展示实际使用这些symbol的例子：
```
// 等效于 /^.{10}$/
let hasLengthOf10 = {
  [Symbol.match]: function(value) {
    return value.length === 10 ? [value] : null;
  },
  [Symbol.replace]: function(value, replacement) {
    return value.length === 10 ? replacement : value;
  },
  [Symbol.search]: function(value) {
    return value.length === 10 ? 0 : -1;
  },
  [Symbol.split]: function(value) {
    return value.length === 10 ? ["", ""] : [value];
  }
}

let message1 = "Hello world",      // 11个字符
    message2 = "Hello John";       // 10个字符

let match1 = message1.match(hasLengthOf10),
    match2 = message2.match(hasLengthOf10);

console.log(match1);   // null
console.log(match2);   // ["Hello John"]

let replace1 = message1.replace(hasLengthOf10, "Howdy!"),
    replace2 = message2.replace(hasLengthOf10, "Howdy!");

console.log(replace1);   // "Hello world"
console.log(replace2);   // "Howdy!"

let search1 = message1.search(hasLengthOf10),
    search2 = message2.search(hasLengthOf10);

console.log(search1);   // -1
console.log(search2);   // 0

let split1 = message1.split(hasLengthOf10),
    split2 = message2.split(hasLengthOf10);

console.log(split1);   // ["Hello John"]
console.log(split2);   // ["", ""]
```
`hasLength10`对象意在像一个匹配长度为10的字符串的正则表达式一样工作。`hasLength10`的四个方法都通过合适的symbol来实现，接着两个字符串上对应的方法被调用了。第一个字符串，`message1`，有11个字符，因此并不匹配；第二个字符串，`message2`，有10个字符，因此匹配。虽然不是一个正则表达式，`hasLength10`被传入了每个字符串方法并被正确使用，这正是因为这些附加的方法。
虽然这是一个简单的例子，能够执行比当前正则表达式更复杂的匹配，为自定义模式匹配提供了更多可能。
### Symbol.toPrimitive方法
JavaScript频繁地尝试在应用某些操作符时将对象隐式转换为原始值。例如，当你将一个字符串与对象使用双等号（`==`）操作符进行比较时，对象在比较之前被转换为一个原始值。确切的应使用的原始值之前是一个内部操作，但是ECMAScript6通过`Symbol.toPrimitive`方法暴露了这个值（使其可变）。
`Symbol.toPrimitive`方法定义在每个标准类型的原型上，它规定了当对象转换为一个原始值时将发生什么。当需要进行一个原始转换时，`Symbol.toPrimitive`被调用，它具有一个单独参数，在标准中被称为`hint`。`hint`参数是三个字符串值之一。如果`hint`是`"number"`，那么`Symbol.toPrimitive`应该返回一个数字。如果`hint`是`"string"`，那么应该返回一个字符串，如果是`"default"`，那么操作符没有类型指定偏向。
对于多数标准对象来说，数值模式有以下行为，按优先级排序：
1. 调用`valueOf()`方法，如果结果是原始值，返回它。
2. 否则，调用`toString()`方法，如果结果是原始值，返回它。
3. 否则，抛出一个错误。
类似地，对于多数标准对象，字符串模式有以下优先级：
1. 调用`toString()`方法，如果结果是原始值，返回它。
2. 否则，调用`valueOf()`方法，如果结果是原始值，返回它。
3. 否则，抛出一个错误。
在多数例子中，标准对象以等效于数值模式的方式对待默认模式（除了`Date`，它的默认模式等效于字符串模式）。通过定义一个`Symbol.toPrimitive`方法，你可以重写这些默认强制转换行为。
默认模式只在`==`操作符、`+`操作符和单独传给`Date`构造函数作为参数时使用。多数操作符使用字符串或者数值模式。
为了重写默认转换行为，使用`Symbol.toPrimitive`并给它赋一个函数作为值。例如：
```
function Temperature(degrees) {
  this.degrees = degrees;
}

Temperature.prototype[Symbol.toPrimitive] = function(hint) {
  switch (hint) {
    case "string":
      return this.degrees + "\u00b0";    // degrees symbol

    case "number":
      return this.degrees;

    case "default":
      return this.degrees + " degrees";
  }
};

let freezing = new Temperature(32);

console.log(freezing + "!");    // "32 degrees"
console.log(freezing / 2);      // 16
console.log(String(freezing));  // "32"
```
这段脚本定义了一个`Temperature`构造函数并重写了原型上的`Symbol.toPrimitive`。根据`hint`参数表示字符串、数值或默认模式（`hint`参数由JavaScript引擎填充）返回不同值。在字符串模式下，`Symbol.toPrimitive`方法返回Unicode度数symbol表示的温度。在数值模式下，它简单返回数值，在默认模式下，它在数值后增加单词`"degrees"`。
每个日志语句触发不同的`hint`参数值。`+`操作符通过设置`hint`为`"default"`触发默认模式，`/`操作符通过设置`hint`为`"number"`触发数值模式，`String()`函数通过设置`hint`为`"string"`触发字符串模式。使三种模式返回不同值时可能的，更常见的是将默认模式设为与字符串或数值模式相同。
### Symbol.toStringTag
JavaScript中最有趣的问题之一曾经是多个全局执行环境可用。这在网页浏览器中一个页面包含了一个iframe时出现，因为页面和iframe由各自的执行环境。在多数案例中，这并不是一个问题，因为数据可以在多个环境中来回传递而不需要担心。在对象在不同环境中传递后试图识别正处理的对象类型时会出现问题。
这个问题的典型例子为从一个iframe中传递一个数组给所包含的网页或者反过来。在ECMAScript6定义中，iframe和包含页面各自代表一个不同的*域*，即一个Javascript的执行环境。每个域有自己的全局作用域和它的全局对象拷贝。无论数组是在哪个域中创建的，它都无疑时一个数组。然而当它被传递到不同的域时，`instanceof Array`调用将返回`false`，因为数组是通过不同域中的构造函数创建的，Array代表的是当前域中的构造函数。
#### 一个识别问题的解决方案
#### ECMAScript6解决方案
### Symbol.unscopables
## 总结
