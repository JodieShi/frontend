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
### 浏览器Rejection处理
## 链式调用Promises
### 错误捕获
### 在Promise链中返回值
### 在Promise链中返回Promises
## 多Promises响应
### Promise.all()方法
### Promise.race()方法
## Promises继承
### 异步任务执行
## 总结
