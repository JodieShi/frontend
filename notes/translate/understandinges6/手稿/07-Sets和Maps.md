# 07 - Sets和Maps
JavaScript在其大部分历史中只有一种表示集合的类型，即`Array`类型（尽管有的人可能争论所有非数组对象都是键-值对的集合，它们的使用意图与数组有很大区别）。JavaScript中的数组与其它语言中的数组用起来一样，但是缺少其它集合选项表明数组也经常被用作队列和栈。由于数组只使用数字索引，因此开发者在遇到需要非数字索引时都使用非数组对象。这种技术导致了使用非数组对象自定义实现的集合（sets）和映射(maps)。
一个*集合（set）*是不包含重复项的一个值列表。你通常不想像访问数组项一样访问集合中的单个元素，更常见的是检查一个集合中是否存在某个值。一个*映射（Map）*是一个键对应特定值的集合。因此，map中的每项存储两个部分的数据，并通过指定读取的键来获取值。map经常被用作缓存，来存储后续快速获取的数据。虽然ECMCScript 5没有正式的sets和maps，开发者们也使用非数组对象来在这种限制下工作。
ECMAScript 6向JavaScript中新增了sets和maps，本章讨论所有你需要知道的关于这两种集合类型的知识。
首先，我们将讨论开发者们在ECMAScript 6之前用来实现sets和maps的解决方案，以及为什么这些实现是有问题的。在关键背景信息之后，我将介绍ECMAScript 6中sets和maps是如何工作的。
## ECMAScript 5中的sets和maps
在ECMAScript 5中，开发人员使用对象属性来模拟sets和maps，如下：
```
let set = Object.create(null);

set.foo = true;

// 检查是否存在
if (set.foo) {
  // 做某些事
}
```
这个例子中的`set`变量是一个基于`null`原型的对象，保证该对象上无继承的属性。在ECMAScript 5中，使用对象属性作为要检查的唯一值是常见的做法。当一个属性加到`set`对象上，它就被设为`true`，因此条件语句（如本例中的`if`语句）可以简单地检测值是否存在。
用作set和用作map的对象的唯一真实区别在于被存储的值。例如，以下例子中使用对象作为一个map：
```
let map = Object.create(null);

map.foo = "bar";

// 获得一个值
let value = map.foo;

console.log(value);   // bar
```
这段代码在键`foo`中存储了一个`bar`值。不同于sets，maps经常被用作获取信息，而不只是检查键是否存在。
## 解决方案的问题
虽然使用对象作为sets和maps在简单情况下可以正常工作，一旦你触及对象属性的限制，该方法就变得复杂地多。例如，由于所有对象属性都需要是字符串，你必须保证没有两个键等同于同一个字符串。考虑以下代码：
```
let map = Object.create(null);

map[5] = "foo";
console.log(map["5"]);  // "foo"
```
该例分配给数值键`5`一个字符串值`"foo"`。在内部，这个数值被转换为字符串，所以`map["5"]`和`map[5]`实际上指同一个属性。这种内部转换在你想要同时使用数字和字符串作为键地时候可能会导致问题。另外一个问题在你想要使用对象作为键的时候发生，如下：
```
let map = Object.create(null),
    key1 = {},
    key2 = {};

map[key1] = "foo";

console.log(map[key2]);  // "foo"
```
这里，`map[key1]`和`map[key2]`指向同一个值。`key1`和`key2`对象被转换为字符串，因为对象属性必须是字符串。由于`"[Object Object]"`是对象的默认字符串表示，所以`key1`和`key2`都被转换为这个字符串。这可能会导致不那么明显的错误，因为逻辑上假设不同的键对象实际上不同是合理的。
会转换为默认字符串表示导致使用对象作为键是困难的。（在尝试使用对象作为set时这个问题也存在。）
有键值为假的map自身也有特定的问题。当需要一个boolean类型的值时，假值将自动转换为false，例如在`if`语句的条件中。这种转换本身不是个问题，只要你谨慎地使用值。例如，观察以下代码：
```
let map = Object.create(null);

map.count = 1;

// 检查count的存在还是非零值？
if (map.count) {
  // ...
}
```
这个例子在如何使用`map.count`上有些模糊。`if`语句是为了检查`map.count`的存在还是值非零？`if`语句中的代码将继续执行，因为值1是真值。然而，如果`map.count`是0，或者说`map.count`不存在，那么if语句中的值将不会被执行。
当它们出现在一个大型应用中是这些问题将很难识别和调试，这也是ECMAScript 6向语言中加入sets和maps的主要原因。
|> JavaScript中有`in`操作符，它在对象中存在属性时返回`true`而不去读取对象的这个值。然而，`in`操作符也会搜索对象的原型，这使得它只有在对象为`null`原型时才是安全的。即使如此，许多开发者仍然错误地使用上述例子中的代码而不是使用`in`。
## ECMAScript 6中的Sets
ECMAScript 6新增了一个`Set`类型，为无重复值的排序列表。Sets允许快速访问包含的元素，使得追踪离散值更为高效。
### 创建Sets并添加项
Sets使用`new Sets()`创建，通过调用`add()`方法向一个set中添加项。你可以通过`size`属性来检查一个set中有多少项：
```
let set = new Set();

set.add(5);
set.add("5");

console.log(set.size);   // 2
```
Sets不会强转值来确定它们是否相同。这表明一个set可以同时包含数字`5`和字符串`"5"`作为两个独立的项。（唯一的例外是-0和+0被认为是相同的。）你可以向set中添加多个对象，它们也讲保持唯一：
```
let set = new Set(),
    key1 = {},
    key2 = {};

set.add(key1);
set.add(key2);

console.log(set.size);  // 2
```
由于`key1`和`key2`并未被转为字符串，它们在set中计为两个独立的项。（记住，如果它们呗转换为字符串，则都等于`"[Object Object]"`。）
如果`add()`方法不止一次地在同一个值上调用，第一次调用后的所有调用都将被视为无效：
```
let set = new Set();
set.add(5);
set.add("5");
set.add(5);  // 重复调用，将被忽略

console.log(set.size);  // 2
```
你可以通过一个数组来初始化一个set，`Set`构造函数将保证值是唯一的。例如：
```
let set = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
console.log(set.size);   // 5
```
在则个例子中，一个有重复值的数组被用来初始化set。虽然在数组中出现了四次，数字`5`只在set中出现一次。这种功能使得转换已存在的代码或JSON结构为使用sets更为容易。
|> `Set`构造函数实际上接收任何可迭代对象作为参数。数组可以工作是因为它默认是可迭代的，sets和maps也是。`Set`构造函数使用一个迭代器来从参数中获取值。（可迭代和迭代器在第8章中被详细讨论。）
你可以使用`has()`方法来检测哪些值是在set中的，如下：
```
let set = new Set();
set.add(5);
set.add("5");

console.log(set.has(5));  // true
console.log(set.has(6));  // false
```
这里，`set.has(6)`将返回false，因为set中不包含这个值。
### 移除值
也可以从set中移除值。你可以通过`delete()`方法来移除单个值，或者你可以调用`clear()`方法来移除set中的所有值。下面的代码展示了这两种动作：
```
let set = new Set();
set.add(5);
set.add("5");

console.log(set.has(5));   // true

set.delete(5);
console.log(set.has(5));  // false
console.log(set.size);  // 1

set.clear();

console.log(set.has("5"));  // false
console.log(set.size);  // 0
```
在调用`delete()`后，只移除了`5`；在`clear()`方法执行后，`set`变空。
上述所有构成了一个追踪单独有序值的简单机制。然而，如果你想向set中添加多项并在每一项上执行某些操作该怎么办？这就是`forEach()`方法应该出现的地方。
### sets的forEach()方法
如果你已经习惯于使用数组，那么你可能早就熟悉了`forEach()`方法。ECMAScript 5向数组增加了`forEach()`来使不使用`for`循环而处理数组中的每个元素更为简单。这个方法在开发者中很受欢迎，因此在sets中也有相同的方法按相同的方式工作。
forEach()方法被传递了一个接收以下三个参数的回调函数：
1. set中下一个位置的值
2. 与第一个参数相同的值
3. 值所在的set
set版本的`forEach()`和数组版本的有一个奇怪的差别：回调函数的第一个和第二个参数是相同的。虽然这看起来像是个错误，但却是有足够理由的。
其它有`forEach()`方法的对象（数组和maps）传递三个参数给它们的回调函数。数组和maps的前两个参数是键值和键（数组为数字索引）。
然而sets不具备键。ECMAScript 6标准背后的人们可能曾使用过接收两个参数的set版本`forEach()`，但是这使得它与其它两类中的不一致。因此，它们找到了一种保持回调函数相同并接收三个参数的方法：set中的每个值被同时认为是键和值。这样，sets的`forEach()`中第一个和第二个参数总是一样的，使得它与数组和maps中的`forEach()`方法功能一致。
除了参数上的差别，在set上使用forEach()基本和数组上使用相同。这里有一些展示该方法如何工作的代码：
```
let set = new Set([1, 2]);

set.forEach(function(value, key, ownerSet) {
  console.log(key + ' ' + value);
  console.log(ownerSet === set);
});
```
上述代码迭代访问set中的每一项，并输出传递给`forEach()`回调函数的值。每次回调函数执行时，`key`和`value`都是相同的，`ownerSet`和`set`也是相等的。上述代码输出：
```
1 1
true
2 2
true
```
与数组相同，你可以传递`this`值给`forEach()`的第二个参数，如果你需要在你的回调函数中使用`this`：
```
let set = new Set([1, 2]);

let processor = {
  output(value) {
    console.log(value);
  },
  process(dataSet) {
    dataSet.forEach(function(value) {
      this.output(value);
    }, this)
  }
};

processor.process(set);
```
在该例中，`processor.process()`方法在set上调用`forEach()`并传递`this`作为回调的`this`值。这对于`this.output()`可以正确解析为`processor.output()`方法是必需的。`forEach()`回调函数只会用到第一个参数，`value`，所以其他的将被忽略。那你也可以使用一个箭头函数来起到相同的效果而不需要传递第二个参数，如下：
```
let set = new Set([1, 2]);

let processor = {
  output(value) {
    console.log(value);
  },
  process(dataSet) {
    dataSet.forEach((value) => this.output(value));
  }
};

processor.process(set);
```
这个例子中的箭头函数从包含的`process()`函数中获得`this`值，所以它可以正确解析`this.output()`为`processor.output()`调用。
记住虽然sets可以很好地追踪值，并且`forEach()`使你可以按需操作每个值，你无法直接像数组中一样直接通过索引访问值。如果你需要做这件事，那么最好的选项是将set转换为一个数组。
### 将set转换为数组
转换一个数组为set很简单，因为你可以向`Set`构造函数传递该数组。使用扩展运算符将set转换回数组也很简单。第三章中介绍了扩展运算符（`...`）作为一种拆分数组中的项为独立的函数参数的方式。你也可以使用扩展运算符来在可迭代对象上工作，如sets，来转换它们为数组。例如：
```
let set = new Set([1, 2, 3, 3, 3, 4, 5]),
    arrray= [...set];

console.log(set);   // [1, 2, 3, 4, 5]
```
这里，set初始由一个包含重复项的数组载入。set移除了重复项，接着使用扩展运算符将非重复项放入了一个新的数组中。set自身依旧包含了它创建时接收的相同的项（`1`, `2`,` 3`, `4`和`5`）。它们只是被拷贝到一个新的数组中。
这个方法在你已经有一个数组并想创建一个无重项的数组时非常有用。例如：
```
let eliminateDuplicates(items) {
  return [...new Set(items)];
}

let numbers = [1, 2, 3, 3, 3, 4, 5],
    noDuplicates = eliminateDuplicates(numbers);

console.log(noDuplicates);   // [1, 2, 3, 4, 5]
```
在`eliminateDuplicates()`函数中，set只是个在创建无重复项数组前用来过滤重复值的临时中间变量。
### Weak Sets
`Set`类型也可以被称为强set，因为它存储的是对象引用。在`Set`实例中存储的对象与在一个变量中存储这个对象效果相同。只要该`Set`实例的引用存在，那么对象就不会被当作垃圾回收来释放内存。例如：
```
let set = new Set(),
    key = {};

set.add(key);
console.log(set.size);  // 1

// 消除原引用
key = null;
console.log(set.size);  // 1

// 获得原引用
key = [...set][0];
```
在这个例子中，将`key`置为`null`清除了`key`对象的一个引用，但是`set`中的依然存在。你依然可以通过用扩展运算符转换set为数组并访问第一个项来获取`key`。对于多数程序，这种结果是没有问题的，但是有些时候，当其他所有引用清除后set中的引用最好也被清除。例如，如果你的JavaScript代码在一个网页中运行，并想要追踪可能被其他脚本移除的DOM元素，那么你并不想你的代码保留DOM元素的最后一个引用。（这种情况被称作*内存泄漏(memory leak)*。）
为了减少这类问题，ECMAScript 6中也包含了*weak sets*，它只存储对象的弱引用而不存储原始值。一个对象的*弱引用（weak reference）*在它为唯一留存的引用时并不阻止垃圾回收。
#### 创建weak set
weak sets通过`WeakSet`构造函数来创建，并且有一个`add()`方法，一个`has()`方法和一个`delete()`方法。这里有一个使用这三个方法的例子：
```
let set = new WeakSet(),
    key = {};

// 向set中添加对象
set.add(key);
console.log(set.has(key));  // true

set.delete(key);
console.log(set.has(key));  // false
```
使用weak set和使用一个常规set很相似。你可以增加、删除并检查weak set中的引用。你也可以i传递一个可迭代对象给构造函数来初始化weak set的值：
```
let key1 = {},
    key2 = {},
    set = new WeakSet([key1, key2]);

console.log(set.has(key1));  // true
console.log(set.has(key2));  // true
```
在这个例子中，一个数组被传递给`WeakSet`构造函数。由于数组包含两个对象，这些对象将被加入weak set中。记住如果数组中包含非对象值，那么将会抛出一个错误，因为`WeakSet`不接收原始值。
#### set类型间的关键区别
weak set和常规set的最大区别在于对象值的弱引用。这里有一个解释这种区别的例子：
```
let set = new WeakSet(),
    key = {};

// 向set中添加对象
set.add(key);

console.log(set.has(key));  // true

// 移除key的最后一个强引用，也从weak set中移除了该项
key = null;
```
在这段代码执行后，weak set中的`key`引用也不复存在。这种移除是无法校验的，因为你需要一个该对象的引用来传递给`has()`方法。这可能会使得weak set的测试有一点令人迷惑，但是你可以相信JavaScript已经合理地移除了这个引用。
这些例子说明了weak sets和常规sets共享了一些特性，但是存在一些关键差别。即：
1. 在`WeakSet`实例中，`add()`方法会在你试图传递一个非对象的时候抛出一个错误。（`has()`和`delete()`方法对非对象参数总是返回`false`）。
2. weak set是不可迭代的，因此不可以在`for-in`循环中使用。
3. weak set未暴露任何迭代器（例如`keys()`和`values()`方法），所以没有办法通过编程确定一个weak set的内容。
4. weak set没有`forEach()`方法。
5. weak set没有`size`属性。
为了合理处理内存，这些看起来是weak set的功能限制是必须的。一般来说，如果你只需要追踪对象引用，那么你应该使用weak set而不是常规set。
sets给了你一个处理值列表的新方法，但是它们在你需要为值关联一些附加信息时并不好用。这就是为什么ECMAScript 6 也新增了maps。
## ECMAScript 6中的maps
ECMAScript 6的`Map`类型是一个有序的键-值对列表，其中键和值都可以是任何类型的。键的相同性由`Set`对象中相同的方式确定，所以你可以同时有`5`和`"5"`作为键，因为它们是不同的类型。这个使用对象属性作为键有很大不同，因为对象属性总是会被强制转换为字符串类型值。
你可以使用`set()`方法来向map中新增项，并传递一个键和对应关联该键的值。你可以稍后通过向`get()`方法中传递键来获取对应值。例如：
```
let map = new Map();
map.set("title", "Understanding ES6");
map.set("year", 2016);

console.log(map.get("title"));  // "Understanding ES6"
console.log(map.get("year"));   // 2016
```
在这个例子中，保存了两个键-值对。`"title"`键保存了一个字符串，而`"year"`键保存了一个数字。`get()`方法稍后被调用来获取键对应的值。如果哪个键并不在map中存在，那么`get()`将会返回一个特殊值`"undefined"`而不是具体值。
你也可以使用对象作为键，这在老的通过对象属性来创建map的解决方案中是不可能的。下面有一个例子：
```
let map = new Map(),
    key1 = {},
    key2 = {};

map.set(key1, 5);
map.set(key2, 42);

console.log(map.get(key1));  // 5
console.log(map.get(key2));  // 42
```
这段代码使用对象`key1`和`key2`作为map中的键来存储两个不同的值，因为这些键不会被强制转换为其他形式，每个对象都被认为是独特的。这使得你可以给对象关联附加数据而不需要修改对象自身。
### Map方法
maps和sets共享数个方法。这是有意的，并允许你通过类似方式在maps和sets之间交互。以下三个方法在maps和sets中都可以使用：
- `has(key)` - 判定给定值在map中是否存在
- `delete(key)` - 从map中移除键和对应值
- `clear()` - 从map中移除所有值
maps也有一个`size`属性，它表明map中含有多少个键-值对。下面的代码使用了这三个方法以及`size`属性：
```
let map = new Map();
map.set("name", "Nicholas");
map.set("age", 25);

console.log(map.size);  // 2

console.log(map.has("name"));  // true
console.log(map.get("name"));  // "Nocholas"

console.log(map.has("age"));   // true
console.log(map.get("age"));  // 25

map.delete("name");
console.log(map.has("name"));  // false
console.log(map.get("name"));  // undefined
console.log(map.size);  // 1

map.clear();
console.log(map.has("name"));  // false
console.log(map.get("name"));  // undefined
console.log(map.has("age"));  // false
console.log(map.get("age"));  // undefined
console.log(map.size);  // 0
```
与在set中想同，`size`属性总是包含map中的键-值对数量。本例中的`Map`实例初始有`"name"`和`"age"`两个键，所以`has()`在传入这两个键时返回`true`。当`"name"`键通过`delete()`方法删除后，`has()`在传入`"name"`时返回`false`，`size`属性同样指示数量的减少。`clear()`方法移除剩余所有键后，两个键对应的`has()`都返回`false`，`size`也变为0。
`clear()`方法是从map中移除大量数据的快捷方法，但是也有个向map中一次添加多个数据的方法。
### Map初始化
类似于set，你可以通过向`Map`构造函数传递一个数组来初始化map中数据。数组中的每一项也必须是一个数组，它的第一项是键，而第二项是键对应的值。因此整个map是一个两项数组的数组，例如：
```
let map = new Map([["name", "Nicholas"], ["age", 25]]);
console.log(map.has("name"));  // true
console.log(map.get("name"));  // "Nicholas"
console.log(map.has("age"));  // true
console.log(map.get("age"));  // 25
console.log(map.size);  // 2
```
键`"name"`和`"age"`是通过构造函数初始化来加入map中的。虽然数组的数组看起来有一点奇怪，准确地表示键是必须的，因为键可能是任何数据类型。将键存在数组中是唯一确保它们不会在存入map前被强制转换为其他数据类型的方法。
### Map的forEach方法
maps的`forEach()`方法与sets和数组中的`forEach()`方法类似，它接受一个接收三个参数的回调函数：
1. map中下个位置的值
2. 值对应的键
3. 获取值的map
这些回调函数的参数更近地匹配数组中`forEach()`的行为，第一个参数是值而第二个参数是键（对应数组中的数字索引）。下面是一个例子：
```
let map = new Map([["name", "Nicholas"], ["age", 25]]);

map.forEach(function(value, key, ownerMpa) {
  console.log(key + " " + value);
  console.log(ownerMap === map);
});
```
该`forEach()`的回调函数输出传给它的信息。`value`和`key`被直接输出，`ownerMap`与`map`进行比较来表示它们的值是相等的。这将输出：
```
name Nicholas
true
age 25
true
```
`forEach()`回调按键-值对插入map中的顺序接收每个键-值对。这与数组中调用`forEach()`有些不同，数组中的`forEach()`回调是按数字索引顺序接收每项的。
|> 你也可以给`forEach()`提供第二个参数来明确回调函数中的`this`值。这种调用与set版本中的`forEach()`方法行为一致。
### Weak Maps
weak map相较于map与weak set相较于set相同：它们是一种保存弱对象引用的方法。在*弱映射（weak map）*中，每个键都必须是一个对象（如果你试图使用一个非对象键，那么将抛出一个错误），这些对象引用是以弱方式保存的，所以它们不会影响到垃圾回收。当weak map外没有weak map中键的引用时，该键-值对将被从weak map中移除。
最常见的使用weak map的地方为在一个网页中创建一个关联特定DOM元素的对象。例如，一些网页JavaScript库为每个库中引用的DOM元素维护一个自定义对象，这种映射被保存在一个内部对象缓存中。
这种方法的困难之处在于确定一个DOM元素何时不再存在在网页中，这样库可以删除对应的对象。否则，库会维护这个DOM元素的引用而不是该引用是否可用，并造成内存泄露。使用weak map来追踪DOM元素依旧允许库为每个DOM元素关联自定义对象，而且在该对象的DOM元素不再存在时可以自动销毁map中的这个对象。
|> 需要注意只有weak map的键，而不是键值，是弱引用。如果其他引用都被移除，weak map中存为值的对象将阻止垃圾回收。
#### 使用weak map
ECMAScript 6的`WeakMap`类型是一个未排序的键-值对列表，其中键必须是非`null`的对象，而值可以是任意类型。`WeakMap`的接口与`Map`非常相似，`set()`和`get()`分别被用来添加数据和获取数据：
```
let map = new WeakMap(),
    element = document.querySelector(".element");

map.set(element, "Original");
let value = map.get(element);
console.log(value);  // "Original"

// 移除该元素
element.parentNode.removeChild(element);
element = null;

// 此时weak map为空
```
在这个例子中，保存了一个键-值对。`element`键是一个DOM元素，被用来存储一个对应的字符串值。这个值接着通过传递给`get()`方法该DOM元素来获取。当DOM元素后来被从文档中移除，变量置为`null`时，数据被从weak map中移除。
与weak sets相似，也没有办法验证weak map为空，因为它并无`size`属性。由于没有遗留的键引用，你也无法通过调用`get()`方法来获取值。weak map切断了对这个键的访问，当垃圾回收器运行时，该值的内存占用将被释放。
#### weak map初始化
为了初始化一个weak map，向`WeakMap`构造函数传递一个数组的数组。类似于初始化一个常规map，数组中的每个数组应该有两项，第一个为非null对象键而第二个为值（任意数据类型）。例如：
```
let key1 = {},
    key2 = {},
    map = new WeakMap([[key1, "Hello"], [key2, 42]]);

console.log(map.has(key1));  // true
console.log(map.get(key1));  // "Hello"
console.log(map.has(key2));  // true
console.log(map.get(key2));  // 42
```
对象`key1`和`key2`在weak map中被用作键，`get()`和`has()`方法可以访问它们。如果`WeakMap`构造函数在任意键-值对中收到一个非对象键，那么将抛出一个错误。
#### weak map方法
weak map只有两个可用的附加方法来与键-值交互。一个为确定给定键是否在map中存在的`has()`方法和一个移除特定键-值对的`delete()`方法。并没有`clear()`方法，因为这需要枚举键，与在weak set一样，这在weak map中是不可能的。下面的例子使用了`has()`和`delete()`方法：
```
let map = new WeakMap(),
    element = document.querySelector(".element");

map.set(element, "Original");

console.log(map.has(element));  // true
console.log(map.get(element));  // "Original"

map.delete(element);
console.log(map.has(element));  // false
console.log(map.get(element));  // undefined
```
这里，DOM元素再一次在一个weak map中被用作键。`has()`方法是检查一个引用当前是否被用作weak map中键的有效方法。记住它只在你有一个非`null`的键引用时才能工作。这个键通过·方法被强制从weak map中移除，此时`has()`返回`false`，`get()`返回`undefined`。
#### 私有对象数据
虽然多数开发者认为weak map的主要使用案例为关联DOM元素与数据，也存在着其他的可能使用场景（毋庸置疑，有些至今还没被发现）。weak map的一个特殊用法为保存对象实例的私有数据。在ECMAScript 6中，对象的所有属性都是公开的，你需要使用一些创造性手段来使数据只有对象可访问，而不是可以任意访问。考虑以下例子：
function Person(name) {
  this._name = name;
}

Person.prototype.getName = function() {
  return this._name;
};
这段代码使用前置下划线的通用约定来表明属性是私有的，不应该在对象实例外被修改。它意在使用`getName()`来获取`this._name`并不允许`_name`值改动。然而，并没有什么可以阻止其他人写`_name`属性，所以它可能被有意或无意间覆盖。
在ECMAScript 5中，通过创建一个使用如下模式的对象来近似拥有真正的私有数据是可能的：
```
var Person = (function() {
  var privateData = {},
      privateId = 0;
  
  function Person(name) {
    Obejct.defineProperty(this, "_id", { value: privateId++ });

    privateData[this._id] = {
      name: name
    };
  }

  Person.prototype.getName = function() {
    return privateData[this._id].name;
  };

  return Person;
}());
```
这个例子将`Person`的定义包裹在一个IIFE中，它含两个私有变量`privateData`和`privateId`。`privateData`对象保存每个实例的私有信息，而`privateId`用来产生每个实例的唯一ID。当`Person`构造函数被调用时，一个不可枚举、不可配置且不可写的`_id`属性被添加。
接着，`privateData`对象中创建了一个对应于对象实例ID的条目，保存着`name`。然后，在`getName()`函数中，姓名可以通过`this._id`作为`privateData`的键来获取。因为`privateData`在IIFE外是不可访问的，因此数据实际上是安全的，即使`this_.id`公开暴露。
这种方法的最大问题在于`privateData`中的数据永远不会消失，因为你无法知道对象实例何时会被销毁；`privateData`将永远包含额外数据。这个问题可以通过使用weak map替代来解决，如下：
```
let Person = (function() {
  let privateData = new WeakMap();

  function Person(name) {
    privateData.set(this, { name: name });
  }

  Person.prototype.getName = function() {
    return privateData.get(this).name;
  };

  return Person;
}());
```
这个版本的`Person`例子用weak map而不是对象来保存私有数据。因为`Person`对象实例自身可以被用作键，因此不需要额外追踪单独的ID。当`Person`构造函数被调用，weak map中创建了一个key为`this`，值为包含私有信息的对象的条目。在这个例子中，该值为只包含`name`的对象。`getName()`函数通过传递`this`给`privateData.get()`函数来获取私有信息，它获取到值对象并访问`name`属性。这种技术保留了私有信息的私有性，并当关联的对象实例销毁时销毁该信息。
#### weak map使用和限制
当决定是使用一个weak map还是常规map时，首先要考虑的是你是否只是想使用对象键。在任何你只使用对象键的时候，最佳选择为使用weak map。这将使你优化内存使用并通过确保多余数据在无法访问后不会被保存来避免内存泄漏。
记住weak map几乎使你无法看见其中的内容，所以你无法使用`forEach()`方法，`size`属性或`clear()`方法来处理其中的项。如果你需要一些监视能力，那么常规map将会是更好的选择。只需记住注意内存使用。
当然，如果你只想使用非对象键，那么常规map是你的唯一选择。
## 总结
ECMAScript 6中正式向JavaScript中引入了sets和maps。在这之前，开发者通常使用对象来模拟sets和maps，经常由于对象属性的相关限制而导致问题。
sets是一些唯一值的有序列表。值不会被强制转换来判断相同性。sets自动移除重复值，所以你可以使用set来过滤一个数组的重复项并返回结果。ses并不是数组的子类，所以你不能随机访问一个set的值。然而，你可以使用`has()`方法来判断set中是否存在某个值，并使用`size`属性来监控set中值的数量。Set类型也有一个处理set中每个值的`forEach()`方法。
Weak sets是特殊的sets，它只包含对象。对象以弱引用方式存储，表明weak set中的项不会在它为对象的唯一剩余引用时阻塞垃圾回收。Weak set中的内容无法被监控，因为复杂的内存管理，所以最好只在需要跟踪成组的对象时使用weak sets。
Maps是有序的键-值对，键可以是任何的数据类型。类似于sets，键不会被强制转换来判断相同性，表明你可以有`5`和`"5"`作为独立的键。任意类型的值可以通过`set()`方法关联于一个键，并随后使用`get()`方法来获取。Maps也有一个`size`属性和一个`forEach()`方法来更便利地访问其中的项。
Weak maps是一类特殊的map，它只能有对象键。和weak sets类似，对象键是弱引用并不会在它为唯一剩余对象引用时阻塞垃圾回收。当一个键被回收后，与该键相关联的值会被从weak map中移除。这种内存处理机制使得weak map特别适合为只在访问它们的代码之外进行生命周期管理的对象关联信息。
