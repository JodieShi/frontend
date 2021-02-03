[TOC]
understanding es6
# 介绍
Javascript的核心语言特性由ECMA-262标准制定。该标准定义的语言为ECMAScript，我们熟悉的在浏览器和Node环境中使用的JavaScript实际上是ECMAScript的一个超集。虽然浏览器和Node环境通过其它对象和方法为JavaScript添加了更多功能，但是该语言的仍然如ECMAScript中所定义。ECMA-262的不断发展对整个JavaScript的成功来说至关重要，而本书涵盖了该语言的最新重大更新——ECMAScript6中的变更。
## ES6之路
## 关于本书
### 浏览器及弄得兼容性
### 目标读者
### 概览
### 约定
### 帮助与支持
## 致谢

# 块绑定
传统上， 变量声明的工作方式是JavaScript编程中棘手的一部分。在多数基于C的语言，变量（或*绑定*）是在声明时创建的。然而，在JavaScript中并不是这样的。你的变量何时真正地被创建取决于你是如何声明它的，ECMAScript6提供了一些更方便控制作用域的选项。本章阐述了为什么经典的`var`声明是令人迷惑的，介绍了ECMAScript6中的块级绑定，并提供了一些使用它们的最佳实践。
## 变量定义和提升
使用`var`定义的变量被认为是处于函数（如果是在函数外定义的则为全局域）的顶部，而不管实际上声明是何处发生的，这叫做*提升(hoisting)*。为了解释提升做了什么，考虑以下函数定义：
```
function getValue(condition) {
  
  if (condition) {
    var value = "blue";
    
    // 其它代码

    return value;
  } else {
    // value在此处的值是undefined
    return null;
  }

  // value在此处的值是undefined
}
```
如果你不熟悉JavaScript，你可能认为变量value只会在`condition`的值是true的时候被创建。实际上，变量`value`无论如何都会被创建。在背后，JavaScript引擎变换`getValue`函数为如下形式：
```
function getValue(condition) {
  var value;
  
  if (condition) {
    value = "blue";

    // 其它代码
    return value;
  } else {

    return null;
  }
}
```
`value`的声明被提升到了顶部，而初始化值则在原来的地方。这说明变量`value`在`else`分支中实际上也是可以访问的，如果在那里访问，变量就是一个`undefined`值，因为它还没有被初始化。
新的JavaScript开发者经常要花一定时间来习惯变量提升，误解这种独特行为可能会导致bug。为此，ECMAScript6引入令人块级作用域选项来强化对变量生命周期的控制。
## 块级声明
块级声明指的是给定块域外无法访问的变量声明。块作用域，也被称作词法作用域，按如下规则创建：
1. 函数内
2. 块内（由{和}符号定义）
块级作用域是多数基于C的语言的工作方式，在ECMAScript6中引入块级作用域选项意在给JavaScript提供相同的灵活性（和统一性）。
### let声明
`let`声明语法和`var`语法相同。基本上，你可以使用`let`来替代`var`去声明一个变量，但是局限变量作用域于当前代码块中（也有一些其它的微小差别，我们稍后讨论它）。由于`let`声明不会提升到封闭块的顶端，你可能总是想在块中首先声明`let`变量，这样你可以在整个块中使用它们。这里有个例子：
```
function getValue(condition) {
  if (condition) {
    let value = "blue";

    // 其它代码
    return value;
  } else {
    // value在此处不存在
    return null;
  }

  // value在此处不存在
}
```
这个版本的`getValue`函数行为更接近于你在其他基于C的语言中所期望的。因为`value`变量是用`let`而不是`vae`声明的，该声明并未被提升到函数定义的顶端，一旦执行流脱离`if`块，变量`value`也无法再被访问。如果`condition`等于false，那么`value`将永远不会被创建或初始化。
### 无重复声明
如果一个标识符已经在一个作用域中被定义，那么在该作用域中使用`let`来声明这个标识符将会导致抛出一个错误。例如：
```
var count = 30;

// 语法错误
let count = 40;
```
在这个例子中，`count`被定义了两次：一次用`var`，另一次用`let`。应为`let`不允许重复定义一个在相同作用域中已经存在的标识符，因此`let`声明将抛出一个错误。另一方面，如果`let`声明创建了一个与包含域中变量同名的变量，那么不会有错误抛出，如下述代码所示：
```
var count = 30;

// 不会抛出错误
if (condition) {
  let count = 40;
  // 其它代码
}
```
`let`声明不会抛出错误，因为它在`if`语句中创建了一个叫`count`的新变量，而不是在外围块中创建`count`。在`if`块内，新变量覆盖局部的`count`，从而在执行离开块前避免访问它。
### 常量声明
你可以在ECMScript6中使用`const`声明语法来定义变量。使用`const`声明的变量被认为是*常量（constants）*，表明它们的值一旦设定就不会再改变。因此，每个`const`变量必须在声明时被初始化，如下所示：
```
// 有效常量
const maxItems = 30;

// 语法错误：缺少初始化
const name;
```
`maxItems`被初始化了，所以它的`const`声明没有问题，正常工作。但是如果你尝试运行包含该段代码的程序，`name`变量将导致一个语法错误，因为`name`未被初始化。
#### 常量 vs let声明
常量，类似于`let`声明，是块级声明。这表明一旦执行流离开常量被声明的块，就不再可以被访问，并且声明不会被提升，如下面这个例子所阐述：
```
if (condition) {
  const maxItems = 5;
  // 更多代码
}

// maxItems在此处无法被访问
```
在这段代码中，常量`maxItems`在一个`if`语句中被声明。一旦语句结束执行，`maxItems`在这个块外部就不可再被访问。
另外一点类似于`let`的地方是当使用相同域中已经定义过的变量标识符来声明`const`时会抛出一个错误。这与变量是使用`var`声明的（全局或函数作用域）还是`let`声明的（块作用域）。例如，考虑以下代码：
```
var message = "Hello!";
let age = 25;

// 下面的每个语句都将抛出错误
const message = "Goodbye!";
const age = 30;
```
这两个`const`声明独立时是有效的，但是这个例子中之前给出了`var`和`let`声明，那它们都不会按照预想工作。
除了这些相似处，`let`和`const`之间有个很大的区别需要被记住。在严格模式和非严格模式中，尝试赋一个`const`给之前定义的常量都将抛出一个错误：
```
const maxItems = 5;
maxItems = 6;  // 抛出错误
```
非常类似其他语言中的常量，`maxItems`变量无法在之后赋一个新值。不同于其他语言中的常量，如果是一个对象，常量的值是可以被改变的。
#### 使用const声明对象
一个`const`声明阻止绑定的修改，并非值本身。这表明对象的`const`声明不会阻止对这些对象的修改。例如：
```
const person = {
  name: "Nicholas"
};

// 工作
person.name = "Greg";

// 抛出一个错误
person = {
  name: "Greg"
};
```
这里，使用一个初始值为具有一个属性的对象来创建`person`绑定。改变`person.name`而不导致错误是可能的，因为这改变了`person`包含的，而不改变`person`所绑定的那个值。当代码尝试去给`person`赋一个值的时候（因此尝试改变绑定），错误就将被抛出。这个`const`处理对象的细节很容易被误解。请记住：`const`阻止对绑定的修改，而不是对值的修改。
### 暂时性死区
使用`let`或`const`声明的变量只有在声明之后才可以被访问。尝试在声明前访问会导致一个引用错误，即使是使用常规的安全操作，如本例中的`typeof`操作：
```
if (condition) {
  console.log(typeof value);  // 引用错误!
  let value = "blue";
}
```
此处，`value`变量是使用`let`声明和初始化的，但是这个语句并不会被执行，因为之前的代码行中抛出了一个错误。这个问题是由于`value`存在于JavaScript社区称之为*暂时性死区（temporal dead zone, TDZ）*中。TDZ并未在ECMAScript标准中明确命名，但却是常用来描述`let`和`const`声明在声明之前无法被访问的术语。本节涵盖了一些TDZ导致的声明放置的细节，虽然例子中使用的都是`let`，注意它们同样适用于`const`。
当JavaScript引擎浏览接下来的块并发现了一个变量声明，它或者将声明提升到函数的顶端或全局作用域（对于`var`），或者将声明放入TDZ中（对于`let`和`const`）。任何尝试访问TDZ中变量的行为都将导致一个运行时错误。变量只有当执行流到达变量声明，并因此可以安全使用时才被移出TDZ。
这在任何你尝试在定义前使用`let`或`const`声明的变量时都是成立的。正如前一个例子表明的，它甚至适用于常规安全`typeof`操作符。然而你可以在变量声明块外来对变量使用`typeof`，虽然这可能不会给你你所想要的结果。考虑以下代码：
```
console.log(typeof value);  // undefined
if (condition) {
  let value = "blue";
}
```
当`typeof`操作执行时`value`变量并不在TDZ中，因为它出现在`value`的声明块外。这表明当前没有`value`绑定，因此`typeof`直接返回`undefined`。
TDZ只是块绑定的一个独特的地方。另一个独特的方面与在循环中使用它们有关。
## 循环中的块绑定
可能开发人员最希望变量具有块级作用域的领域是在循环中，此时一次性计数变量只能在循环中被使用。例如，下面的代码在JavaScript中并不少见：
```
for (var i = 0; i < 10; i++) {
  process(items[i]);
}

// i在此处依然可以被访问
console.log(i);   // 10
```
在其他语言中，块级作用域是默认的，这个例子应该可以按预想的方式工作。，只有在`for`循环中才可以访问到`i`变量。然而，在JavaScript中，`i`在循环结束后依旧可以被访问，因为`var`声明被提升了。像下面代码中这样使用`let`来代替，则会获得预想的行为：
```
for (let i = 0; i < 10; i++) {
  process(items[i]);
}

// 此处的i不可被访问，抛出一个错误
console.log(i);
```
在这个例子中，变量`i`只在`for`循环中存在。一旦循环完成，变量在其它地方就不再可访问。
### 循环中的函数
`var`的特性一直以来使得在循环中创建函数成为问题，因为循环变量可以在循环作用域外被访问。考虑以下代码：
```
var funcs = [];

for (var i = 0; i < 10; i++) {
  funcs.push(function() { console.log(i); });
}

funcs.forEach(function(func) {
  func();  // 输出"10"10次
});
```
你可能原先希望代码打印数字0到9，但是它在一行中输出了10次10。这是因为`i`在循环的每次迭代中被共享，表明每个在循环中被创建的函数共享同一个变量的引用。变量`i`在循环结束时的值为`10`，因此当`console.log(i)`被调用时，这个值每次都被打印。
为了解决这个问题，开发者在循环内使用立即调用的函数表达式（immediately-invoked function expressions, IIFEs）来强制创建要迭代的变量的新副本，如下例所示：
```
var funcs = [];

for (var i = 0; i < 10; i++) {
  funcs.push((function(value) {
    return function() {
      console.log(value);
    }
  }(i)));
}

funcs.forEach(function(func) {
  func();  // 输出0, 1, 2直到9
});
```
这个版本在循环中使用IIFE。`i`变量被传递给了IIFE，它创建自己的副本并存储为`value`。这是本次迭代中使用的值，因此调用每个函数将按循环次数从0到9返回预想的值。幸运的是，ECMASript中的`let`和`const`块级绑定可以为你简化这个循环。
### 循环中的let声明
`let`声明通过有效模仿前个例子中IIFE所做的来简化循环。在每次迭代中，循环创建一个新的变量并按前一次迭代中的同名变量来赋值。这表明你可以完全忽略IIFE并获得预想的结果，如下：
```
var funcs = [];

for (let i = 0; i < 10; i++) {
  funcs.push(function() {
    console.log(i);
  });
}

funcs.forEach(function(func) {
  func();  // 输出0, 1, 2至9
});
```
该循环与使用`var`和IIFE的工作方式完全一样，但是更为整洁。`let`声明在循环的每一轮创建一个新的变量`i`，所以循环中创建的每个函数都有自己的`i`副本。每个`i`的副本具有自己的值，并在循环每次迭代的开始创建和赋值，`for-in`和`for of`循环中也是一样的，如下所示：
```
var funcs = [],
    object = {
      a: true,
      b: true,
      c: true
    };

for (let key in object) {
  funcs.push(function() {
    console.log(key);
  });
}

funcs.forEach(function(func) {
  func();   // 输出 "a", "b"和"c"
});
```
在这个例子中，`for-in`循环展示了与`for`循环相同的行为。循环的每一轮，创建新的`key`绑定，所以每个函数都有自己的`key`变量副本。结果是每个函数输出一个不同的值。如果使用`var`变量来声明`key`，那么所有函数都将输出"c"。
|> 需要理解`let`声明在循环中的行为在标准中是一个特别定义的行为，并不是与`let`的非提升特性相关联的。实际上，`let`的早起实现并不具备这个行为，它是在后来才被加入的。
### 循环中的const声明
ECMAScript6标准并没有明确不允许在循环中使用`const`声明，然而，基于你使用的循环类型仍有一些区别。对于一个正常的`for`循环，你可以使用`const`初始化，但是如果你尝试改变它的值循环将抛出一个警告：
```
var funcs = [];

// 一次迭代后将抛出错误
for (const i = 0; i < 10; i++) {
  funcs.push(function() {
    console.log(i);
  });
}
```
在这段代码中，`i`变量被定义为常量。循环的第一轮迭代，`i`为0，成功执行。当`i++`执行时将抛出一个错误，因为它试图修改一个常量。因此，你只能在循环初始化中用`const`声明一个变量，如果你不准备修改这个变量。
另一方面，当在`for-in`和`for-of`循环中使用时，`const`变量和`let`变量的行为相同。所以下面的例子不会导致错误：
```
var func = [],
    object = {
      a: true,
      b: true,
      c: true
    };

// 不会导致错误
for (const key in object) {
  funcs.push(function() {
    console.log(key);
  });
}

funcs.forEach(function(func) {
  func();   // 输出"a", "b"和"c"
});
```
这段代码和“循环中的let声明”一节中的第二个例子功能几乎完全相同。唯一的区别在于`key`的值不能在循环中改变。`for-in`和`for-of`循环中`const`可以工作，因为循环初始器在循环的每次迭代创建新的绑定，而不是去尝试改变已有绑定的值（正如前一个例子中使用`for`而不是`for-in`一样）。
## 全局块绑定
另外一个`let`和`const`不同于`var`的地方在于他们在全局作用域中的表现。当`var`在全局作用域中使用时，它创建一个新的全局变量，成为全局对象（浏览器中为`window`）的一个属性。这表明你可能会使用`var`不经意间覆盖一个已有的全局属性，例如：
```
// 在浏览器中
var RegExp = "Hello!";
console.log(window.RegExp);   // "Hello!"

var ncz = "Hi!";
console.log(window.ncz);   // "Hi!"
```
虽然`RegExp`是一个定义在`window`上的全局变量，它因会被`var`声明覆盖而并不安全。该例展示了一个覆盖原有变量的新`RegExp`全局变量。类似地，`ncz`被定义为一个全局变量，并被立刻定义为`window`的一个属性。这是JavaScript一直以来的工作方式。
如果你在全局作用域中使用`let`和`const`，那么全局作用域中创建了一个新的绑定但是不会作为属性添加给全局对象。这也说明你不能使用`let`和`const`来覆盖全局变量，你只能遮蔽它。这里有一个例子：
```
// 在浏览器中
let RegExp = "Hello!";
console.log(RegExp);   // "Hello!"
console.log(window.RegExp === RegExp);   // false

const ncz = "Hi!";
console.log(ncz);   // "Hi!"
console.log("ncz" in window);  // false
```
这里，一个新的`RegExp`的`let`声明创建了一个绑定，遮盖了全局的`RegExp`。这表明`window.RegExp`和`RegExp`不是一样的，因此在全局作用域中并无冲突。同样，`ncz`的`const`声明创建了一个绑定但是没有在全局对象上创建属性。这种能力使得`let`和`const`在全局作用域中使用更为安全，如果你不想在全局对象上创建属性。
|> 如果你有需要从全局对象中获得的代码，那么你或许仍然想在全局作用域中使用`var`。当你想要跨frame或者跨窗口获得代码时，这在浏览器中非常常见。
## 新兴块绑定最佳实践
虽然ECMAScript6仍在开发中，但是人们普遍认为你应该默认使用`let`而不是`var`来声明变量。对于许多JavaScript开发者来说，`let`可以准确按`var`原应表现的方式工作，因此直接替换逻辑上是合理的。在这种情况下，你可以使用`const`来定义需要修改保护的变量。
然而，随着更多开发者迁移至ECMAScript6，一种可选的方法流行开来：默认使用`const`，而只在你知道变量的值需要修改的时候使用`let`。它的原理在于多数变量在初始化后不会改变自身的值，因为预料外的值变更可能是bug来源。当你采用ECMASCript6时，这个想法极具吸引力并值得探索。
## 总结
`let`和`const`块绑定向JavaScript中引入了词法作用域。这些声明不会被提升并且只在他们声明的块中存在。这提供了更像其他语言中的行为并减少了导致意外错误的几率，因为变量现在可以准确在需要它们时声明。副作用是你无法在它们声明前访问它们，即使使用一个类似`typeof`的安全操作。尝试在声明前访问一个块绑定将导致一个错误，因为绑定现在存在于暂时性死区中（TDZ）。
在多数情况下，`let`和`const`和`var`的行为类似，但是在循环中并不相同。对于`let`和`const`来说，`for-in`和`for-of`循环在循环的每次迭代中创建新的绑定。这说明在循环体中创建的函数可以直接访问循环绑定值，因为它们正在当前迭代中，而不是在循环的最后迭代之后（如使用`var`时候的行为）。在`for`循环中使用`let`也是一样的，然而在`for`循环中尝试使用`const`声明可能会导致错误。
当前块绑定的最佳事件是默认使用`const`而在你知道变量的值需要被改变时使用`let`。这确保了代码的基本不变性，从而避免了某些类型的错误。
# Promises和异步编程

JavaScript最强大的一部分在于它如何轻易地解决异步编程的问题。作为一种为Web而创建的语言，Javascript从一开始就需要可以响应异步用户交互，例如点击和按键。通过使用回调作为事件的替代方案，Node.js进一步普及了JavaScript中的异步编程。随着越来越多的程序开始使用异步编程，回调和事件已经不足以支撑开发人员想实现的一切。Promises则是该问题的解决方案。
Promises是异步编程的另一种选择，它的工作形式类似于其它语言中的futures和deferreds（*注：futures和deferreds应该指的是其它编程语言中的异步执行方案*）。一个Promise指定了稍后将要执行的代码段（类似事件、回调），并且明确了代码是否成功完成了任务。根据执行结果成功或失败，可以链式串接多个promise，从而使得代码更易于理解和调试。
为了更好地理解promises是如何工作的，有必要先理解一些promises基于的基本概念。
## 异步编程背景
JavasSript引擎是建立于一个单线程事件循环的概念上的，单线程表明同一时间只有一段代码是正在执行的。这一点不同于如Java、C++的其它语言，它们的线程机制可以允许同一时间执行多段代码。当多段代码可以访问并更改状态时，状态维护和状态保护成为一个难题，并且是基于线程的软件中常见的bug源。
JavaScript引擎同一时间只能执行一段代码，因此需要跟踪要运行的代码。代码被保存在*任务队列（job queue）*中。当一段代码进入准备执行的状态时，它将被加入任务队列中。当JavaS引擎结束执行当前代码段时，事件循环开始执行队列中的下一个任务。*事件循环（event loop）*就是JavaScript引擎中控制代码执行和管理任务队列的进程。请记住，作为队列，任务的执行是从队列中第一个任务到最后一个任务逐次执行的。
### 事件模型
当用户点击一个按钮或者按下键盘中的某个按键时，一个如`onclick`的*事件（event）*将被触发，该事件可能通过在任务队列尾添加新任务来响应交互，这就是JavaScript中最基本的异步编程模式。事件处理代码直到事件被触发才执行，并且在执行时有合适的上下文。例如：
```
let button = document.getElementById("my-btn");
button.onclick = function(event) {
  console.log("Clicked");
};
```
在上述代码中，`console.log("Clicked")`直到`button`被点击时才会执行。当`button`被点击时，分配给`onclick`的函数被添加在任务队列的末尾，并将在其之前的所有其它任务执行完时执行。
事件机制适用于简单交互，但是将多个独立的异步嗲用链接在一起时会变得更复杂，因为必须追踪每个事件的事件目标（如前述例子中的`button`）。此外，还必须确保在事件第一次发生前就已经添加好了所有合适的事件处理程序。例如，如果`button`在`onclick`事件绑定之前就被点击了，那么并不会有任何响应。因此虽然事件在响应用户交互和类似的不常见功能时很有用，在面对更复杂的需求时，它并不够灵活。
### 回调机制
Node.js创建之时就通过广泛使用编程中的回调机制来改进了异步编程模型。与事件模型相似，回调机制中的异步代码直到后续的某一时间点才会执行，而不同之处在于将调用的函数是通过参数来传递的，如：
```
readFile("example.txt", function(err, contents) {
  if (err) {
    throw err;
  }
  console.log(contents);
});
console.log("Hi!");
```
该例中使用了Node.js中经典的*错误优先（error-first）*回调风格。`readFile`函数意在读磁盘上的某一个文件（通过第一个参数指定），并在读操作结束之后执行回调（第二个参数）。如果有错误发生，回调函数会通过`err`参数接受一个错误对象；否则回调函数的`contents`参数将接受一个包含文件内容的字符串。
使用回调机制，`readFile()`函数会立刻执行，并在其开始读磁盘时暂停，也就是说`console.log("Hi!")`会在`readFile()`函数调用之后立刻执行输出，此时`console.log(contents)`尚未执行打印。当`readFile()`调用结束时，它在任务队列尾新增回调函数及其参数，当回调函数之前的所有其它任务执行完成时回调被执行。
相比于事件，回调机制更为灵活，因为它在链接多个函数调用时更为简单，例如：
```
readFile("example.txt", function(err, contents) {
  if (err) {
    throw err;
  }
  
  writeFile("example.txt", function(err) {
    if (err) {
      throw err;
    }
    console.log("File is written!");
  });
});
```
在上述代码中，`readFile()`的成功调用将导致另一个异步调用，即`writeFile()`函数，注意这两个异步调用中存在着相同的`err`检查基本模式。当`readFile()`调用结束时，它在任务队列尾新增将导致`writeFile()`调用的任务（假设没有任何错误发生）。接着，`writeFile()`也在结束时向任务队列尾新增一个回调任务。
该机制的效果非常不错，但是你很快会发现自己陷入了一个*回调地狱（callback hell）*。回调地狱在你嵌套多个回调时出现，例如：
```
method1(function(err, result) {
  if (err) {
    throw err;
  }
  
  method2(function(err, result) {
    if (err) {
      throw err;
    }
    
    method3(function(err, result) {
      if (err) {
        throw err;
      }
      
      method4(function(err, result) {
          if (err) {
            throw err;
          }
          
          method5(result);
      });
    });
  });
});
```
像该例中一样嵌套多层方法调用会创建一个难以理解和调试的纠结的代码网。当你想实现更为复杂的功能时，回调也会出现问题。假如你想要同步运行两个异步操作，并在它们都执行结束时通知你该如何做？假如你想同时开始两个异步操作但是只想获得首先结束的结果又该如何做？
在这些场景中，你需要跟踪多个回调和清除操作，promises则可以大大改善这种情况。
## Promise基础
一个Promise是一个异步操作的占位符。代替订阅一个事件或传递一个回调函数，函数可以返回一个promise，如：
```
// readFile承诺在未来某一时间点完成
let promise = readFile("example.txt");
```
在这段代码中，`readFile()`实际上并不立刻开始读文件，这将稍后发生。而是该函数会返回一个表示异步读操作的promise对象，以便你可以在将来使用它。确切地说，你何时可以使用该结果完全取决于promise生命周期如何发挥作用。
### Promise生命周期
每个promise都经历一个短暂的以*挂起（pending）*状态开始的生命周期，它表示异步操作还没结束。一个pending状态的promise被认为是*未解决的（unsettled）*。上个例子中的promise在`readFile()`返回它后就进入pending状态。一旦异步操作完成，promise就被认为是*解决了的（settled）*，并进入以下两个可能的状态：
1. *完成（fulfilled）*：promise的异步操作已经成功完成。
2. *拒绝（rejected）*：promise的异步操作由于错误或其他原因未能成功完成。
一个内部的`[[PromiseState]]`属性会被设为`"pending"`,`"fulfilled"`或`"rejected"`来反映promise的状态。该属性暴露在promise对象上，所以你无法通过编程来决定promise处于什么状态。但你可以通过使用`then()`方法来在promise状态发生变化时执行一个指定的动作。
`then()`方法暴露在所有promise上，它接收两个参数。第一个参数是当promise进入fulfilled状态时调用的函数，第二个参数是promise进入rejected状态时调用的函数。类似于fulfillment函数，rejection函数也被传递了一些有关rejection的附加数据。
|> 任何以上述形式实现了`then()`方法的对象都可以称为一个*thenable*。所有的promise都是thenable，但并不是所有的thenable都是promise。
`then()`方法的两个参数都是可选的，所以你可以监听fulfillment（完成）和rejection（拒绝）的任何组合。例如，考虑以下`then()`调用集合：
```
let promise = readFile("example.txt");

promise.then(function(contents) {
  // fulfillment
  console.log(contents);
}, function(err) {
  // rejection
  console.error(err.message);
});

promise.then(function(contents) {
  // fulfillment
  console.log(contents);
});

promises.then(null, function(err) {
  // rejection
  console.error(err.message); 
});
```
上述三个`then()`调用基于同一个promise。第一个调用同时监听fulfillment和rejection；第二个只监听fulfillment，错误不会被上报；第三个只监听rejection，不上报成功。
Promise也有一个`catch()`方法，当只传递一个rejection处理函数时，它与`then()`方法行为相同。例如，下述`catch()`和`then()`调用功能上是相同的：
```
promise.catch(function(err) {
  // rejection
  console.error(err.message);
});

// is the same as:
promise.then(null, function(err) {
  // rejection
  console.error(err.message);
});
```
`then()`和`catch()`背后的目的是为了让你结合使用它们来正确地处理异步操作结果。这种系统比事件和回调更好，因为它使得操作是成功还是失败完全清楚（当错误存在时事件通常不会被触发，而在回调中你必须总是记得检查错误参数）。只需要知道如果你不附加一个rejection处理函数，所有失败将静默发生。总是附加一个rejection处理函数，即便它只是记录错误。
即使是在promise处于settled态之后被加入任务队列，fulfillment或rejection处理函数也将被执行。这样，你可以在任何时候添加新的fulfillment和rejection处理函数并且保证它们会被调用。例如：
```
let promise = readFile("example.txt");

// 初始fulfillment处理函数
promise.then(function(contents) {
  console.log(contents);

  // 新增一个
  promise.then(function(contents) {
    console.log(contents);
  });
});
```
在上述代码段中，fulfillment处理函数中为同一个promise增加了一个新的fulfillment处理函数。此时promise已经处于完成态，所以新的fulfillmen处理函数被加入任务队列，并在就绪时被调用。rejection处理函数的工作方式相同。
|> 任何一个`then()`或`catch()`调用都将创建一个新的任务，并在promise进入resolved态时被执行。但是这些任务最终会在一个单独的、严格为promises保留的任务队列中结束。只要你理解任务队列的一般工作原理，第二任务队列的确切细节对理解如何使用promises来说并不重要。
### 创建unsettled Promises
新的promises由`Promise`构造函数创建。构造函数接收一个参数：一个被称为*执行器(executor)*的函数，它包含了初始化promise的代码。执行器被传递了两个名为`resolve()`和`reject()`的函数作为参数。`resolve()`函数在执行器成功完成并发出promise准备好了被解决的信号时被调用，而`reject()`函数则表明执行器已经失败。
这里有一个使用Node.js中的promise来实现本章中早先提到的`readFile()`函数的例子：
```
// Node.js例子
let fs = require("fs");

function readFile(filename) {
  return new Promise(function(resolve, reject) {
    // 触发异步操作
    fs.readFile(filename, { encoding: "utf-8" }, function(err, contents) {
      // 检查错误
      if (err) {
        reject(err);
        return;
      }

      // 成功读取
      resolve(contents);
    });
  });
}

let promise = readFile("example.txt");

// 监听fulfilmment和rejection
promise.then(function(contents) {
  // fulfillment
  console.log(contents);
}, function(err) {
  // rejection
  console.error(err.message);
})
```
在该例中，原生Node.js`fs.readFile()`异步调用被一个promise包裹。执行器或者传递一个错误对象给`reject()`函数，或者传递文件内容给`resolve()`函数。
请记住，执行器当`readFile()`函数被调用时立刻执行。当执行器中的`resolve()`或`reject()`函数被调用时，一个新的任务被添加到任务队列来解决这个promise。这被称为*任务调度(job scheduling)*，如果你曾经使用过`setTimeout()`或`setInterval()`函数，那么你应该已经对它很熟悉。在任务调度中，你增加一个新任务到任务队列中，说：“不要现在执行它，在稍后去执行它”。例如，`setTimeout()`函数允许你指定将任务添加到队列前的时延：
```
// 在500ms过去后将方法加入到任务队列中
setTimeout(function() {
  console.log("Timeout");
}, 500);

console.log("Hi!");
```
上述代码调度一个任务，在500ms后将它加入到任务队列中。两个`console.log()`调用的输出结果如下：
```
Hi!
Timeout
```
由于500ms的延迟，传递给`setTimeout()`的函数中的打印在`console.log("Hi!")`调用之后输出。
Promise工作方式与之类似。Promise执行器在源码中任何出现在其之后的代码前执行。例如：
```
let promise = new Promise(function(resolve, reject) {
  console.log("Promise");
  resolve();
});
console.log("Hi!");
```
代码段的输出为：
```
Promise
Hi!
```
调用`resolve()`触发一个异步操作。传递给`then()`和`catch()`的函数是异步执行的，因为它们也是被添加到任务队列中的。这里有一个例子：
```
let promise = new Promise(function(resolve, reject) {
  console.log("Promise");
  resolve();
});

promise.then(function() {
  console.log("Resolved.");
});

console.log("Hi");
```
该例的输出为：
```
Promise
Hi!
Resolved.
```
注意虽然`then()`调用出现在`console.log("Hi!")`代码行前，但它直到稍后才执行（与执行器不同）。这是因为fulfillment和rejection处理函数总是在执行器完成后被添加到任务队列的尾端。
### 创建settled Promises
由于Promise执行器的动态特性，`Promise`构造函数是创建unsettled promise的最佳方式。但是如果你只想要一个表示已知值的promise，那么调度一个仅仅传递值给`resolve()`函数的任务是没有意义的。取而代之，有两种创建给定指定值的settled promise的方法。
#### 使用Promise.resolve()
`Promise.resolve()`方法接收一个单一参数，并返回一个在fulfilled态的promise。这表明没有任务调度发生，你需要给这个promise增加一个或多个fulfillment处理函数来获取值。例如：
```
let promise = Promise.resolve(42);

promise.then(function(value) {
  console.log(value); // 42
});
```
该代码段创建一个fulfilled态的promise，所以fulfillment处理函数收到42作为参数`value`。如果给该promise增加一个rejection处理函数，那么rejection处理函数永远不会被调用，因为该promise永远都不会进入rejected态。
#### 使用Promise.reject()
你也可以用`Promise.reject()`函数来创建一个rejected态的promise。这与`Promise.resolve()`工作方式类似，除了创建的promise是在rejected态，如下：
```
let promise = Promise.reject(42);

promise.catch(function(value) {
  console.log(value); // 42
});
```
除了fulfillment处理函数，任何添加给这个promise的rejection处理函数都将被调用。
|> 如果你传给`Promise.resolve()`或`Promise.reject()`方法一个promise，那么promise将被原样返回。
#### Non-Promise Thenables
`Promise.resolve()`和`Promise.reject()`也接收非promise thenables作为参数。当传递一个非promise thenable时，这些方法创建一个新的promise，它在`then()`方法之后被调用。
当一个对象具备一个接收`resolve`和`reject`作为参数的`then()`方法时，一个非promise thenable就被创建了，如：
```
let thenable = {
  then: function(resolve, reject) {
    resolve(42);
  }
};
```
该例中的`thenable`对象除了`then()`方法外不具备任何与promise相关的特性。你可以通过调用`Promise.resolve()`来将这个`thenable`转换为一个fulfilled态的promise：
```
let thenable = {
  then: function(resolve, reject) {
    resolve(42);
  }
};

let p1 = Promise.resolve(thenable);
p1.then(function(value) {
  console.log(value);
});
```
在这个例子中，`Promise.resolve()`调用`thenable.then()`，因此可以确定promise状态。该promise的状态是fulfilled，因为在`then()`方法中调用了`resolve(42)`。一个叫做`p1`的fulfilled态promise被创建了，并通过`thenable`传递了一个值（即42），因此`p1`的fulfillment处理函数接收到42作为value。
将相同的流程与`Promise.resolve()`一起使用，则可以通过thenable来创建一个rejected态的promise：
```
let thenable = {
  then: function(resolve, rejcet) {
    reject(42);
  }
};

let p1 = Promise.resolve(thenable);
p1.catch(function(value) {
  console.log(value);  // 42
});
```
这个例子和上一个类似，除了`thenable`是rejected态的。当`thenable.then()`执行时，一个值为42的rejected态promise被创建了，该值接下来被传递给`p1`的rejection处理函数。
`Promise.resolve`和`Promise.reject()`的这种工作方式使得你可以方便地处理非promise thenables。许多库优先于在ECMAScript 6中介绍的promises已使用了thenables，因此为了后向兼容先前存在的库，转换thenable为正式promises的能力非常重要。当你不确定一个对象是否是一个promise的时候，通过`Promise.resolve()`或`Promise.reject()`来传递该对象（取决于你的预期结果）是最佳的得知方法，因为promises将被无修改地传递。
### 错误执行处理器
当一个错误在执行器中被抛出时，该promise的rejection处理函数将被调用。例如：
```
let promise = new Promise(function(resolve, reject) {
  throw new Error("Explosion!");
});

promise.catch(function(error) {
  console.log(error.message);
});
```
在上述代码中，执行器试图抛出一个错误。这是每个执行器内部的隐式`try-catch`，这样可以捕获错误，并传递给rejection处理函数。上述例子等价于下面的代码：
```
let promise = new Promise(function(resolve, reject) {
  try {
    throw new Error("Explosion!");
  } catch(ex) {
    reject(ex);
  }
});

promise.catch(function(error) {
  console.log(error.message);
});
```
为简化此类常见用例，执行器可以捕获所有抛出的错误，但是只有当一个rejection处理函数存在时，执行器中抛出的错误才会被上报。
此外，错误将被抑制。这在开发人员使用promises的前期就成为一个问题，JavaScript环境通过提供捕获rejected态promises的钩子函数来解决它。
## 全局Promise Rejection处理
promises的一个最具争议的问题在于promise在无rejection处理函数而进入rejected态时静默失败。一些人认为这规范中的最大缺陷，因为它是JavaScript语言中唯一不显式表现出错误的部分。
由于promises的天然特性，无法直观地确定一个promise的rejection是否被处理了。例如，考虑以下例子：
```
let rejected = Promise.reject(42);

// 此时，rejected未被处理

// 一段时间后
rejected.catch(function(value) {
  // 现在rejected被处理了
  console.log(value)；
});
```
你可以在任何时间调用`then()`和`catch()`函数，并且不管promise是否已经settled，它们都将正确地执行，这使得很难明确一个promise何时被处理。在该例子中，promise立刻进入rejected态，但是直到后来才被处理。
虽然下个版本的ECMAScript可能会解决这个问题，浏览器和Node.js环境都实现了一些改变来解决开发者的这个痛点。它们不属于ECMAScript6标准，但却是使用promises的非常有用的工具。
### Node.js Rejection处理
在Node.js中，`process`对象上有两个与promise rejection处理相关的事件：
- `unhandledRejection`：当一个promise进入rejected态，但在该轮事件循环中没有rejection处理函数被调用时触发
- `rejectionHandled`：当一个promise进入rejected态，且在该轮事件循环后有rejection处理函数被调用时触发
这些事件旨在通过共同协作来帮助识别出进入rejected态但未被处理的promise。
`unhandledRejection`事件处理函数接收两个参数：rejection原因（通常是一个错误对象）和进入rejected态的promise。下述代码演示了一个活动中的`unhandledRejection`：
```
let rejected;

process.on("unhandledRejection", function(reason,promise) {
  console.log(reason.message);  // "Explosion!"
  console.log(rejected === promise);  // true
});

rejected = Promise.reject(new Error("Explosion!"));
```
该例创建了一个带错误对象的rejected态promise，并监听`unhandledRejection`事件。事件处理器接收错误对象作为第一个参数，接收该promise作为第二个。
`rejectionHandled`事件只有一个参数，即进入rejected态的promise。例如：
```
let rejected;

process.on("rejectionHandled", function(promise) {
  console.log(rejected === promise);  // true
});

rejected = Promise.reject(new Error("Explosion!"));

// 等待增加rejection处理函数
setTimeout(function() {
  rejected.catch(function(value) {
    console.log(value.message);  // "Explosion!"
  });
}, 1000);
```
这里`rejectionHandled`事件在rejection处理函数最终调用的时候被触发。如果rejection处理函数是直接在`rejected`创建之后附加到`rejected`上的，那么这个事件将不会被触发。相反地，rejection处理函数将在创建`rejected`的事件循环的同一轮被调用，这没有什么用。
为了正确地追踪可能未被处理的rejected态promise，可以使用`rejectionHandled`和`unhandledRejection`事件来维护一个可能未被处理的rejected态promise列表。然后等待一段时间来检查这个列表。例如：
```
let possiblyUnhandledRejections = new Map();

// 当一个rejection是未处理的，将它加入map中
process.on("unhandledRejection", function(reason, promise) {
  possiblyUnhandledRejections.set(promise, reason);
});

process.on("rejectionHandled", function(promise) {
  possiblyUnhandledRejections.delete(promise);
});

setInterval(function() {
  possiblyUnhandledRejections.forEach(function(reason, promise) {
    console.log(reason.message ? reason.message : reason);

    // 处理这些rejections
    handleRejection(promise, reason);
  });

  possiblyUnhandledRejection.clear();
}, 60000);
```
这是一个简单的未处理rejection追踪器。它使用一个map来存储promise和它的rejection原因。每个promise是一个键，该promise的原因是对应的键值。每次`unhandledRejection`被触发时，promise和它的rejection原因被添加到map中。每次`rejectionHandled`被触发时，被处理的promise从map中被移除。这样，`possiblyUnhandledRejections`随着事件的调用而增缩。`setInterval()`函数调用周期性检查可能未处理的rejections列表，并在控制台中输出相关信息（实际上，除了记录日志，你可能想做另一些事，或者处理该rejection）。该例中使用了map而不是weak map，因为你需要周期性检查这个map来看其中当前有哪些promise，在weak map中这是不可能的。
虽然这个例子是针对Node.js的，浏览器也实现了一个类似的机制来通知开发者未处理的rejections。
### 浏览器Rejection处理
浏览器也会触发两类事件来帮助识别未处理的rejections。这些事件是window对象触发的，效果上和Node.js是相同的：
- `unhandledrejection`：当一个promise进入rejected态并且该轮事件循环中没有rejection处理函数调用时触发
- `rejectionhandled`：当一个promise进入rejected态并且在该轮事件循环后有rejection处理函数被调用时触发
与Node.js实现中传递独立的参数给事件处理函数不同，这些浏览器事件处理函数接收一个拥有以下属性的事件对象：
- `type`：事件名称（"unhandledrejection"或"rejectionhandled"）
- `promise`：rejected态promise
- `reason`：promise的rejection值
浏览器实现的另一个区别是rejction值(`reason`)在这两个事件中都可以获得。例如：
```
let rejected;

window.onhandledrejection = function(event) {
  console.log(event.type);  // "unhandledrejection"
  console.log(event.reason.message);  // "Explosion!"
  console.log(rejected === event.promise); // true
};

window.onrejectionhandled = function(event) {
  console.log(event.type);  // "rejectionhandled"
  console.log(event.reason.message);  // "Explosion!"
  console.log(rejected === event.promise); // true
};

rejected = Promise.reject(new Error("Explosion!"));
```
该段代码使用DOM 0级表示`onunhandledrejection`和`onrejectionhandled`来注册事件处理函数（如果你喜欢，也可以通过`addEventListener("unhandledrejection")`和`addEventListener("rejectionhandled")`）。每个事件处理函数接收一个包含rejected态promise信息的事件对象。在两个事件处理函数中，`type`，`promise`和`reason`属性都是可获得的。
在浏览器中追踪未处理rejections的代码和在Node.js中的也非常相似：
```
let possiblyUnhandledRejections = new Map();

// 当一个rejection是未处理的时候，将它加入map
window.onunhandledrejection = function(event) {
  possiblyUnhandledRejections.set(event.promise, event.reason);
};

window.onrejectionhandled = function(event) {
  possiblyUnhandledRejections.delete(promise);
};

setInterval(function() {
  possiblyUnhandledRejections.forEach(function(reason, promise) {
    console.log(reason.message ? reason.message : reason);

    // 一些处理rejections的方法
    handleRejection(promise, reason);
  });
}, 60000);
```
上述实现几乎完全等同于Node.js中的实现。它使用相同的方法来在map中存储promise和它的rejection值，接着稍后检查他们。其中的唯一真实差别就在于从事件处理函数中获取的信息。
处理promise rejections可能是棘手的，但是你已经开始看到promise到底可以多么强大。现在是时候采取下一步行动，链接多个promise。
## 链式调用Promises
到目前为止，promise看起来是只是增量改进了组合使用回调和`setTimeout()`函数，但是promise远不止这些。更具体地来说，有多个方法来链接promise，从而完成更加复杂的异步行为。
每次`then()`和`catch()`调用实际上创建并返回另一个promise。第二个promise只有当第一个已经进入fulfilled态或rejected态时才被解决（resolved）。考虑以下例子：
```
let p1 = new Promise(function(resolve, reject) {
  resolve(42);
});

p1.then(function(value) {
  console.log(value);
}).then(function() {
  console.log("Finished");
});
```
代码输出为：
```
42
Finished
```
`p1.then()`调用返回第二个promise，并在其上调用`then()`。第二个`then()`fulfillment处理函数只有当第一个promise已经被解决了才调用。如果你拆解上述例子中的链接，它应该如下：
```
let p1 = new Promise(function(resolve, reject) {
  resolve(42);
});

let p2 = p1.then(function(value) {
  console.log(value);
});

p2.then(function() {
  console.log("Finished);
});
```
在这个未链接的代码版本中，`p1.then()`的结果存储在`p2`中，接着`p2.then()`被调用了，来加上最后的fulfillment处理函数。如你所猜测的那样，`p2.then()`也返回一个promise。本例中只是没有使用这个promise。
### 错误捕获
promise链允许你捕获在前一个promise的fulfillment或rejection处理函数中可能出现的错误。例如：
```
let p1 = new Promise(function(resolve, reject) {
  resolve(42);
});

p1.then(function(value) {
  throw new Error("Boom!");
}).catch(function(error) {
  console.log(error.message);  // "Boom!"
});
```
在上述代码中，p1的fulfillment处理函数中抛出了一个错误。在第二个promise上，对`catch()`的链式调用可以通过它的rejection处理函数来接收这个错误。当rejection处理函数抛出一个错误时也是一样的：
```
let p1 = new Promise(function(resolve, reject) {
  throw new Error("Explosion!");
});

p1.catch(function(error) {
  console.log(error.message);   // "Explosion!"
  throw new Error("Boom!");
}).catch(function(error) {
  console.log(error.message);   // "Boom!"
});
```
这里，执行器抛出了一个错误并触发了`p1`promise的rejection处理函数。处理函数中抛出了另一个错误，并被第二个promise的rejection处理函数捕获。链式promise调用可以感知链上其它promise中的错误。
|> 总是在promise链的最后有一个确保你可以正确处理任何可能出现的错误的rejection处理函数。
### Promise链的返回值
promise链的另一个重要方面是从一个promise向下一个传递数据的能力。你可能已经见过在执行器中传递给`resolve()`处理函数的值被传递给了该promise的fulfillment处理函数。你可以通过从fulfillment处理函数中明确一个返回值来在沿该链继续传递数据。例如：
```
let p1 = new Promise(function(resolve, reject) {
  resolve(42);
});

p1.then(function(value) {
  console.log(value);    // "42"
  return value + 1;
}).then(function(value) {
  console.log(value);    // "43"
})；
```
`p1`的fulfillment处理函数在执行时返回了`value + 1`。由于`value`是42（从执行器中获得），fulfillment处理函数返回43。该值接着被传递给第二个promise的fulfillment处理函数，并在控制台中输出该值。
你可以在rejection处理函数中做同样的事。当rejection处理函数被调用时，它可以返回一个值。如果它有返回值，那么它将被传到下一个promise的fulfillment里，如：
```
let p1 = new Promise(function(resolve, reject) {
  reject(42);
});

p1.catch(function(value) {
  // 第一个rejection处理函数
  console.log(value);   // "42"
  return value + 1;
}).then(function(value) {
  console.log(value);   // "43"
});
```
此处，执行器调用`reject()`，携带的值为42。该值被传递给promise的rejection处理函数，并返回val+ 1。尽管返回值是由rejection处理函数返回的，它依旧被传递给链上的下一个promise的fulfillment处理函数。如有必要，某个promise的失败可以在整个链上获得恢复。
### 在Promise链中返回Promises
从fulfillment和rejection处理函数中返回初始类型值可以在promise间传递数据，但是如果你想返回一个对象怎么办？如果对象是一个promise，那么需要执行一个额外的步骤来明确如何去处理。考虑如下例子：
```
let p1 = new Promise(function(resolve, reject) {
  resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
  resolve(43);
});

p1.then(function(value) {
  // 第一个fulfillment处理函数
  console.log(value);    // 42
  return p2;
}).then(function(value) {
  // 第二个fulfillment处理函数
  console.log(value);    // 43
});
```
在这段代码中，`p1`调度了一个任务，resolve(42)。p1的fulfillment处理函数返回了`p2`，它是一个已经进入resolved态的promise。第二个fulfillment处理函数被调用了，因为`p2`已经fulfilled了。如果`p2`已经进入了rejected态，那么rejection处理函数（如果有的话）将取代第二个fulfillment处理函数被调用。
该模式下需要重点认识的是，第二个fulfillment处理函数不是加给`p2`的，而是第三个promise。因此第二个fulfillment处理函数是被附加给这第三个promise的，使得之前的例子等价于：
```
let p1 = new Promise(function(resolve, reject) {
  resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
  resolve(43);
});

let p3 = p1.then(function(value) {
  // 第一个fulfillment处理函数
  console.log(value);   // 42
  return p2;
});

p3.then(function(value) {
  // 第二个fulfillment处理函数
  console.log(value);
});
```
此处可以清楚地看到第二个fulfillment处理函数是附加在`p3`上的，而不是`p2`。这是一个微小但重要的区别，因为如果`p2`进入rejected态，第二个fulfillment处理函数不会被调用。例如：
```
let p1 = new Promise(function(resolve, reject) {
  resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
  reject(43);
});

p1.then(function(value) {
  // 第一个fulfillment处理函数
  console.log(value);  // 42
  return p2;
}).then(function(value) {
  // 第二个fulfillment处理函数
  console.log(value);
});
```
在这个例子中，第二个fulfillment处理函数永远都不会被调用，因为`p2`是rejected态的。然而你可以附加一个rejection处理函数来替代：
```
let p1 = new Promise(function(resolve, reject) {
  resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
  reject(43);
});

p1.then(function(value) {
  // 第一个fulfillment处理函数
  console.log(value);    // 42
  reutrn p2;
}).catch(function(value) {
  // rejection处理韩叔叔
  console.log(value);   // 43
});
```
在这个例子中，rejection处理函数被调用了，因为`p2`进入rejected态。`p2`的rejected值43被传递给了这个rejection处理函数。
从fulfillment或rejection处理函数中返回的thenables不会在promise执行器执行时改变。第一个定义的promise将会首先执行它的执行器，接着第二个promise执行器将会执行，如此以往。返回thenable只是简单地允许你定义一个附加的响应给promise的结果。你可以通过在一个fulfillment处理函数中创建一个新的promise来延迟fulfillment函数的执行。例如：
```
let p1 = new Promise(function(resolve, reject) {
  resolve(42);
});

p1.then(function(value) {
  console.log(value);   // 42
  
  // 创建一个新的promise
  let p2 = new Promise(function(resolve, reject) {
    resolve(43);
  });

  return p2;
}).then(function(value) {
  console.log(value);   // 43
});
```
在这个例子中，在`p1`的fulfillment处理函数中创建了一个新的promise。这表明第二个fulfillment处理函数直到`p2`进入fulfilled态才会被执行。这个模式在你想要等待前一个promise被解决后再触发下一个promise的时候非常有用。
## 多Promises响应
到此为止，本章中的每个例子处理的都是同时响应一个promise。然而，你可能会想要监控多个promise的进程来决定下一个动作。ECMAScript6提供两种监控多个promise的方法：`Promise.all()`和`Promise.race()`。
### Promise.all()方法
`Promise.all()`方法接收一个单独参数，一个监控的promises的可迭代类型（类似于数组），并返回一个promise，只有当可迭代参数中的所有promise都解决了它才被解决。当可迭代参数中的所有promise都是fulfilled态时返回的promise才是fulfilled态的，如下例所示：
```
let p1 = new Promise(function(resolve, reject) {
  resolve(42);
});
let p2 = new Promise(function(resolve, reject) {
  resolve(43);
});
let p3 = new Promise(function(resolve, reject) {
  resolve(44);
});

let p4 = Promise.all([p1, p2, p3]);

p4.then(function(value) {
  console.log(Array.isArray(value));    // true
  console.log(value[0]);  // 42
  console.log(value[1]);  // 43
  console.log(value[2]);  // 44
});
```
这里的每个promise携带一个数字被解决。`Promise.all()`调用创建promise `p4`，并在`p1`，`p2`和`p3`进入fulfilled态时最终进入fulfilled态。传递给`p4`的fulfillment处理函数的结果是一个数组，它包含每个解决值：42，43和44。值按照promise传递给`Promise.all()`的顺序存储，所以你可以匹配promise结果和对应的promise。
如果任一个传递给`Promise.all()`的promise进入rejected态，那么返回的promise立刻进入rejected态，而不等其他promise完成。
```
let p1 = new Promise(function(resolve, reject) {
  resolve(42);
});
let p2 = new Promise(function(resolve, reject) {
  reject(43);
});
let p3 = new Promise(function(resolve, reject) {
  resolve(44);
});

let p4 = Promise.all([p1, p2, p3]);

p4.catch(function(value) {
  console.log(Array.isArray(value));    // false
  console.log(value[0]);  // 43
});
```
在这个例子中，`p2`携带值43进入rejected态。`p4`的rejection处理函数立刻被调用，而不等`p1`和`p3`结束执行（它们仍旧将执行，`p4`只是不会等待）。
rejection处理函数总是接收一个单独的值而不是一个数组，该值是进入rejected态的那个promise携带的rejection值。在这个例子中，rejection处理函数被传递了43来映射`p2`的rejection。
### Promise.race()方法
`Promise.race()`方法提供一个稍微不同的监控多个promise的方法。该方法也接收一个需要监控的promise的可迭代类型参数并返回一个promise，但是返回的promise在第一个promise被解决后就进入被解决的状态。不同于`Promise.all()`方法中等待所有promise被解决，`Promise.race()`方法在数组中任何一个promise进入fulfilled态时返回一个适当的promise。例如：
```
let p1 = Promise.resolve(42);

let p2 = new Promise(function(resolve, reject) {
  resolve(43);
});


let p3 = new Promise(function(resolve, reject) {
  resolve(44);
});

let p4 = Promise.race([p1, p2, p3]);
p4.then(function(value) {
  console.log(value);
});   // 42
```
在上述代码中，`p1`被创建为一个fulfilled态的promise，而其他的是调度任务。`p4`的fulfillment处理函数被和值42一起调用，并忽略其他的promise。传递给`Promise.race()`的promise是真在竞争来看谁被首先解决。如果第一个被解决的promise是fulfilled态，则返回的promise是fulfilled态的；如果第一个被解决的promise是rejected态的，那么返回的promise是rejected态的。这里有一个rejection的例子：
```
let p1 = new Promise(function(resolve, reject) {
  setTimeout(function() {
    resolve(42);
  }, 100);
});

let p2 = new Promise(function(resolve, reject) {
  reject(43);
});

let p3 = new Promise(function(resolve, reject) {
  setTimeout(function() {
    resolve(44);
  }, 50);
});

let p4 = Promise.race([p1, p2, p3]);

p4.catch(function(value) {
  console.log(value);  // 43
})
```
这里，`p1`和`p3`都使用了`setTimeout()`（在Node.js和网页浏览器中都可得）来延迟promise完成。结果是`p4`进入rejected因为`p2`在`p1`和`p3`被解决前进入了rejected态。虽然`p1`和`p3`最终进入fulfilled态，这些结果被忽略了，因为它们在`p2`进入rejected态之后出现。
## Promises继承
与其它内置类型类似，你可以使用promise作为一个派生类的基类。这使得你可以定义你自己的promise变体来扩展内置promise的功能。例如假设你想要创建一个promise，除了`then()`和`catch()`方法以外，还可以调用它名为`success()`和`failure()`的方法。那么你可以创建以下的promise类型：
```
class MyPromise extends Promise {
  // 使用默认构造函数
  success(resolve, reject) {
    return this.then(resolve, reject);
  }

  failure(reject) {
    return this.catch(reject);
  }
}

let promise = new MyPromise(function(resolve, reject) {
  resolve(42);
});

promise.success(function(value) {
  console.log(value);   // 42
}).failure(function(value) {
  console.log(value);
});
```
在上述例子中，`MyPromise`是从`Promise`中派生的，并且有两个附加的方法。`success()`方法模仿`then()`方法，`failure()`方法模仿`catch()`方法。
每个新增的方法使用`this`来调用它所模拟的方法。派生promise和内置promise的功能相同，除了现在可以按你所想调用`success()`和`failure()`方法。
由于静态方法是继承的，因此在派生promise上仍然有`MyPromise.resolve()`，`MyPromise.reject()`，`MyPromise.race()`和`MyPromise.all()`方法。后面的两个方法和内置方法行为一致，但是前两个方法有些许差别。
`MyPromise.resolve()`和`MyPromise.reject()`方法都将返回MyPromise实例，而忽略传递的值。因为这些方法使用`Symbol.species`属性（如第9章中所述）来决定返回的promise类型。如果向一个方法中传递了一个内置promise类型数据，这个promise将进入resolve态或者rejected态，方法将返回一个新的`MyPromise`，这样你可以附加fulfillment或rejection处理函数。例如：
```
let p1 = new Promise(function(resolve, reject) {
  resolve(42);
});

let p2 = MyPromise.resolve(p1);
p2.success(function(value) {
  console.log(value);    // 42
});

console.log(p2 instanceof MyPromise);    // true
```
这里，p1是一个内置的promise类型，并被传递给了`MyPromise.resolve()`方法。该方法的结果，p2，是`MyPromise`的一个实例，p1的resolved值被传递给它的fulfillment处理函数。
如果`MyPromise`的一个实例被传递给`MyPromise.resolve()`或`MyPromise.reject()`方法，那么它将被直接返回而不进入resolved。在其他所有方面，这两个方法和`Promise.resolve()`和`Promise.reject()`表现一致。
### 异步任务执行
在第8章中，我曾介绍了生成器并展示给你如何使用它们来实现异步任务执行，如下：
```
let fs = require("fs");

function run(taskDef) {
  // 创建迭代器，使得它在其它地方可用
  let task = taskDef();

  // 开始任务
  let result = task.next();

  // 递归调用方法来持续调用next()
  function step() {
    // 如果有其他要做的
    if (!result.done) {
      if (typeof result.value === "function") {
        result.value(function(err, data) {
          if (err) {
            result = task.throw(err);
            return;
          }

          result = task.next(data);
          step();
        });
      } else {
        result = task.next(result.value);
        step();
      }
    }
  }

  // 开始程序
  step();
}

// 定义一个使用task执行器的函数
function readFile(filename) {
  return function(callback) {
    fs.readFile(filename, callback);
  };
}

// 执行任务
run(function*() {
  let contents = yield readFile("config.json");
  doSomethingWith(contents);
  console.log("done");
});
```
这种实现有一些痛点。首先，将每个函数包裹在一个返回函数的函数中有一点令人迷惑（甚至这句话也是令人迷惑的）。其次，无法区分用作任务执行器回调的函数返回值和非回调的返回值。
使用promise，你可以通过确认每个异步操作返回一个promise来大大地简化和泛化这个程序。该通用接口表明你可以大大地简化异步代码。这里有一个你可以简化该任务执行器的方法：
```
let fs = require("fs");

function run(taskDef) {

  // 创建迭代器
  let task = taskDef();
  

  // 开始任务
  let result = task.next();

  // 递归调用方法来遍历
  (function step() {
    // 如果有其他要做的
    if (!result.done) {
      // 构造一个resolved态的promise来简化
      let promise = Promise.resolve(result.value);
      promise.then(function(value) {
        result.task.next(value);
        step();
      }).catch(function(error) {
        result = task.throw(error);
        step();
      });
    }
  }());
}

// 定义一个使用任务执行器的函数
function readFile(filename) {
  return new Promise(function(resolve, reject) {
    fs.readFile(filename, function(err, contents) {
      if (err) {
        reject(err);
      } else {
        resolve(contents);
      }
    });
  });
}

// 执行任务
run(function*() {
  let contents = yield readFile("config.json");
  doSomethingWith(contents);
  console.log("Done");
});
```
在这个版本的代码中，一个泛化的`run()`函数执行一个生成器来创建一个迭代器。它调用`task.next()`来开始任务并迭代调用`step()`直到迭代器完成。
在`step()`函数中，如果有其他的要做，那么`result.done`是`false`的。此时，`result.value`应该是一个promise，但是`Promise.resolve()`调用只是为了防止有问题的函数没有返回一个promise（记住，Promise.resolve()只传递任何传入的promise或包裹任何promise中的非promise）。接着，增加了一个fulfillment处理函数来获取promise的值并回传给迭代器。在这之后，`result`在`step()`函数调用自身之前被分配给下一个yield结果。
rejection处理函数存储错误对象中的任何rejection结果。`task.throw()`方法将错误对象回传给迭代器，如果一个错误在任务中被捕获，`result`被分配给下一个yield结果。最后，`step()`在`catch()`中被调用来继续。
这个`run()`函数可以执行任何使用`yield`来实现异步代码的生成器，而不必向开发者暴露promise（或回调）。实际上，由于函数调用返回值总是被转换为一个promise，这个函数甚至可以返回除了promise的其他东西。这表明同步和异步方法在使用 `yield`调用时都将正确工作，并且你永远都不需要检查返回值是否为promise。
唯一需要关系的是确保异步函数，如`readFile`返回一个正确标识自己状态的promise。对于Node.js内置方法，这表明你需要转换这些方法来返回promise而不是使用回调。
### 未来的异步任务执行
在笔者写书同时，有正在进行的工作来向JavaScript中带入一个更简单的异步任务执行语法。`await`语法的工作正在进行，该语法可以密切反映前一节中基于promise的例子。基本想法是使用一个标记为`async`的函数而不是一个生成器，并使用`await`来在调用时替代`yield`，例如：
```
(async function() {
  let contents = await readFile("config.json"); 
  doSomethindWith(contents);
  console.log("Done");
})();
```
`function`前的`async`关键词表明函数将在异步状态下执行。`await`关键词表明`readFile("config.json")`函数调用应该返回一个promise，如果没有，返回值应该被一个promise包裹。正如上一节中`润()`的实现，`await`将会抛出一个错误，如果promise是rejected态的，或者从promise中返回一个值。最终的结果是你可以向写同步代码一样写异步代码，没有处理基于迭代器的状态机的负荷。
`await`语法预计在ECMAScript 2017（ECMAScript 8）中落地。
## 总结
Promise旨在通过提供比事件和回调更多的对异步操作的控制组合使用，来改进JavaScript中异步编程。Promise调度任务来加入JavaScript引擎任务队列中以便稍后执行，而另一个任务队列记录promise的fulfillment和rejection处理函数来确保正确的执行。
Promise有三种状态：pending，fulfilled和rejected。一个promise从pending态开始并在成功执行时进入fulfilled态而在失败时进入rejected态。在任一情况下，处理函数可以在promise被处理的时候添加。`then()`方法允许你添加一个fulfillment处理函数和一个rejection处理函数，`catch()`方法只允许你添加一个rejection处理函数。
你可以通过许多方式来链接多个promise并在其间传递信息。每次`then()`调用都创建并返回一个新的promise，它由前一个resolved而来。这种链可以用来触发对一系列异步事件的响应。你也可以使用`Promise.race()`或`Promise.all()`来监控多个promise处理状态并做出相应响应。
异步任务调用在你组合使用生成器和promise的时候更简单，因为promise给出了一个异步操作可以返回的通用接口。这样你可以使用生成器和yield操作来等待异步响应并适当响应。
多数新的网页API正基于promise被创建，你可以期待未来有更多的接口效仿。
