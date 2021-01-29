[TOC]
# Sass/Scss
## 简介
css（cascading style sheets），级联样式表，实际上并不是一种编程语言，而是用来指定网页元素显示样式的工具。通过选择器+样式属性+样式值的方式为选中元素指定样式，并最终聚合为该网页上的样式表，各个元素的样式由表中涉及该元素的属性根据优先级、层叠规则等共同决定。
许多程序员会对css感到迷惑，它本身缺少逻辑性，很少体现出传统程序设计中的解耦思想，更没有变量、控制等语言特性，使得样式的可维护和可扩展编写门槛较高。在此背景下，css预处理器应运而生，它通过真正的编程语言来编写网页样式，并编译为css最终应用于网页，Sass(Syntactically Awesome Style Sheets)就是一种常见的css预处理器。
Sass在css基础上增加了变量、函数、控制语句、混入及导入等功能，使得css程序化设计成为可能。

## 安装与使用
### 独立使用
1. 安装ruby（Windows下需安装，Mac下自带）
2. 安装Sass
```gem install sass```
3. 编写scss/sass文件test.scss
4. 编译为css代码
```sass test.scss```
5. 其它常用命令
```
/* 使用watch选项监视单个文件，修改源文件并保存时会自动编译 */
sass --watch input.scss:output.css
/* 使用watch选项监视整个文件夹 */
sass --watch app/sass:public/stylesheets
/* 查看帮助 */
sass --help
```
### 脚手架项目中使用
安装依赖
```
npm install sass-loader --save-dev
npm install node-sass --save-dev
```

## 语法
### 语法格式
**sass** 以.sass为文件后缀，采用缩进格式进行编码；
**scss** 以.scss为文件后缀，采用CSS3语法格式（花括号）。
使用命令sass-convert可对两种语法格式进行转换：
```
sass-convert style.sass style.scss
sass-convert style.scss style.sass
```
### 变量
使用以"$"符号开头的标识符来标识变量：
```
$highlight-color: #F90; // 高亮颜色变量声明
```
作用域：所定义的规则块({})内。
变量引用:
```
p {
  color: $highlight-color;
}
编译结果为
p {
  color: #F90;
}
// 拼接在字符串中时需要以#{}包裹起来
$side: left;
.rounded {
  border-#{$side}-radius: 5px;
}
编译结果为
.rounded {
  border-left-radius: 5px;
]
```
### 嵌套规则
基本规则：通过规则块进行多层嵌套，外层为父选择器，内层为子选择器，通过空格连接，体现为后代选择器效果
```
#content {
  article {
    h1 { color: #333; }
  }
  aside { background-color: #EEE; }
}
编译结果为
#content article h1 {
  color: #333;
}
#content aside {
  background-color: #333;
}
```
特殊标识符&，表示父选择器，内层出现&符号时由父选择器替换，不进行空格连接，常用于伪类等
```
article a {
  color: blue;
  &:hover: { color: red; }
}
编译结果为
article a { color: blue; }
article a:hover { color: red; }
```
群组选择器：逗号","表示，此时各自展开
```
// 作用于子选择器
.container {
  h1, h2, h3 { margin-bottom: .8em; }
}
编译结果为
.container h1, .container h2, .container h3 { margin-bottom: .8em; }
// 作用域父选择器
nav, aside {
  a { color: blue; }
}
编译结果为
nav a, aside a { color: blue; }
```
组合选择器：与css中规则相同
`>`：直接子元素
`+`：同层相邻（后一个）
`~`：同层所有
属性嵌套：中划线前部分:{中划线后部分: 值;}
```
nav {
  border: 1px solid #ccc {
  left: 0px;
  right: 0px;
  }
}
编译结果为
nav {
  border: 1px solid #ccc;
  border-left: 0px;
  border-right: 0px;
}
```
### 导入规则
1. 以下划线_开头的文件不会被编译为css文件，称为局部文件，导入时可忽略下划线
```
// 导入theme/_night-sky.scss
@import "theme/night-sky"
```
2. !default标签设置默认变量值，即该变量未被声明赋值时的默认取值
```
$fancybox-width: 400px !default;
.fancybox {
width: $fancybox-width;
}
编译结果为
.fancybox { width: 400px; }

$fancybox-width: 200px;
$fancybox-width: 400px !default;
.fancybox {
width: $fancybox-width;
}
编译结果为
.fancybox { width: 200px; }
```
3. 嵌套导入：在导入处插入，在导入的规则块内生效
```
// _blue-theme.scss文件内容
aside {
  background-color: blue;
  color: white;
}
// 导入该局部文件
.blue theme {@import "blue-theme"}
// 编译结果为
.blue-theme {
  aside {
    background-color: blue;
    color: white;
  }
}
```
4. css原生导入
当@import路径为以下三种形式时，会生成原生的CSS@import
```
@import "aaa.css"
@import url("aaa.css")
@import url(aaa.css)
```
若要直接导入css文件，而不是通过CSS@import，可以通过修改.css后缀为.scss后缀，然后通过@import引入。
### 注释
sass支持两种形式的注释：
```
// 行内注释，不会出现在编译结果中
/* 块注释，会出现在编译结果中 */
body {
 color /* 出现在原生css不允许的地方, 不会被编译 */: #333;
 padding: 1; /* 出现在原生css不允许的地方, 不会被编译 */
}

```
### 混入@mixin
使用@mixin标识符来标识一个混合器，即一段可重用的样式代码，在使用的地方通过@include来提取混入器内容
```
// 定义mixin
@mixin rounded-corners {
  -moz-border-radius: 5px;
  -webkit-border-radius: 5px;
  border-radius: 5px;
}
// 使用mixin
.notice {
  background-color: green;
  border: 2px solid #00aa00;
  @include rounded-corners;
}
编译结果为
.notice {
  background-color: green;
  border: 2px solid #00aa00;
  -moz-border-radius: 5px;
  -webkit-border-radius: 5px;
  border-radius: 5px;
}
```
使用场景：可重用的共享样式，多用于描述样式的应用效果，如rounded-corners。
混入的为css规则时，遵循嵌套规则。
```
@mixin no-bullets {
  list-style: none;
  li {
    list-style-image: none;
    list-syle-type: none;
    margin-left: 0px;
  }
}
ul.plain {
  color: #444;
  @include no-bullets;
}
编译结果为
ul.plain {
  color: #444;
  list-style: none;
}
ul.plain li {
  list-style-image: none;
  list-style-type: none;
  margin-left: 0px;
}
```
传参
```
// 定义
@mixin m1($name) { ... }
// 传参
@include m1(value);  // 按序传参
@include m1($name: value);  // 具名传参
```
### 继承@extend
使用@extend标识符可以实现样式继承。
```
.error {
  border: 1px solid red;
  background-color: #fdd;
}
.seriousError {
  @extend .error;
  border-width: 3px;
}
```
同时继承的还有与.error相关的组合选择器样式，如
```
.error a { ... }
h1.error { ... }
```
使用场景：继承来源于面向对象的编程思想，适用于类与基于类的延申样式定义，建立在语义化的关系之上，如.seriousError与.error之间的关系，是.seriousError继承于.error，.error是基类。
常见用法
```
// 继承一个html元素
.disabled {
  color: grey;
  @extend a;  // 继承超链接元素<a>
}
```
继承的选择器需要完全匹配，如
```
.seriousError @extend .important.error // .seriousError继承.important.error、h1.important.error等样式，但不继承.important .error单独样式
#main .seriousError @extend .error // 只有#main .seriousError选中元素的样式继承.error样式，class="seriousError"的非#main元素不继承
```
## 实践
### 文件组织
### 样式复用



# Less
Less也是一种CSS预处理语言，具有变量、函数、Mixin等特性，与Sass不同，Less是基于JavaScript实现的，可以运行在Node或浏览器端（客户端），而Sass是基于Ruby实现的，安装和使用需要基于Ruby环境（服务端）。
## 安装与使用
Less可以通过npm在命令行引入，也可以在浏览器中通过文件引入。
### 安装
```
npm install -g less
```
### 命令行调用
```
lessc styles.less  // 编译并输出到标准输出
lessc styles.less styles.css  // 编译并重定向到指定文件
lessc --clean-css styles.less styles.min.css  // 编译为最小化形式CSS（需安装clean-css插件）
```
### 在代码中使用
```
var less = require('less');
less.render('.class {width: (1 + 1)}', function(e, output) {
  console.log(output.css);
});
```
这将触发node进行编译，并输出
```
.class {
  width: 2;
}
```
也可向rende函数中添加一些配置选项，如；
```
var less = require('less');

less.render('.class { width: (1 + 1) }',
  {
    paths: ['.', './lib'],  // 指定@import指令的搜索路径
    filename: 'style.less', // 指定一个文件名
    compress: true          // 压缩结果
  },
  function (e, output) {
    console.log(output.css);
  }
);
```
### 浏览器中使用
通过link引入，并通过script引入less.js：
```
<link rel="stylesheets/less" type="text/css" href="styles.less" />
<script src="less.js" type="text/javascript"></script>
```