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
### Promise生命周期
### 创建unsettled Promises
### 创建settled Promises
#### 使用Promise.resolve()
#### 使用Promise.reject()
#### Non-Promise Thenables
### 错误执行处理器
## 全局Promise Rejection处理
### Node.js Rejection处理
### 浏览器Rejection处理
## 链接Promises
### 错误捕获
### 在Promise链中返回值
### 在Promise链中返回Promises
## 多Promises响应
### Promise.all()方法
### Promise.race()方法
## Promises继承
### 异步任务执行
## 总结
