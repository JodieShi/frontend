# Symbols和Symbol属性
Symbols是ECMAScript6中引入的一个原始类型，加入到已有的字符串、数字、布尔、`null`和`undefined`原始类型。Symbols开始被用作创建私有对象成员的一个方法，这是JavaScript开发人员一直以来都想要的特性。在Symbols之前，任何使用字符串名的属性都很好访问，无论它的名字是否很晦涩，“私有名”特性旨在让开发人员使用非字符串属性名。此时，常规的检测这些私有名的技术无法生效。
私有名提案最终演化为ECMAScript6中的Symbols，本章将教你如何有效地使用Symbols。虽然实现细节维持一致（即为属性名增加了非字符串值），私有的目标被丢弃。取而代之，symbol属性与其它对象属性分开分类。
## 创建Symbols
## 使用Symbols
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
