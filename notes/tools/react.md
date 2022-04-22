# react
## 导入与安装
通过script标签引入react脚本文件
```html
<!-- 开发版本CDN链接 -->
<script src="https://unpkg.com/react@16/umd/react.development.js" crossorigin></script>
<script src="https://unpkg.com/react-dom@16/umd/react-dom.development.js" crossorigin></script>

<!-- 生产版本CDN链接 -->
<script src="https://unpkg.com/react@16/umd/react.production.min.js" crossorigin></script>
<script src="https://unpkg.com/react-dom@16/umd/react-dom.production.min.js" crossorigin></script>
```

通过工具链在项目上安装
- 创建新的单页面应用：[create-react-app](https://github.com/facebook/create-react-app)
```bash
npx create-react-app my-app
cd my-app
npm start
```
- 通过Node.js构建服务端渲染网站：[Next.js](https://nextjs.org/)
- 构建面向内容的静态网站：[Gatsby](https://www.gatsbyjs.com/)
- 开发组件库、集成至现有代码仓：[Neutrino](https://neutrinojs.org/)、[parcel](https://parceljs.org/)、[razzle](https://github.com/jaredpalmer/razzle)
## main concepts
### JSX
### 元素渲染
#### DOM元素 vs React元素
`const element = <h1>hello world</h1>`
#### 将元素渲染为DOM
```js
// 定义React元素
const element = <h1>hello world</h1>;
// 向ReactDOM.createRoot()传入DOM元素
const root = ReactDOM.createRoot(
  document.getElementById('root');
);
// 将React元素传给root.render()
root.render(element);
```
#### 更新渲染元素
更新UI唯一方法：创建新元素并传给`root.render()`。
#### React只更新需更新的部分
React比较当前元素（及子元素）和之前的元素（及子元素），并只应用必须的更新。
### 组件和props
组件：接收props参数并返回React元素。
#### 函数组件和类组件
函数组件
```js
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```
类组件
```ts
class Welcome extend React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```
#### 组件渲染
```js
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
// props对象为{name: 'Jodie'}
const element = <Welcome name="Jodie" />;
const root = ReactDom.createRoot(document.getElementById('root'));
root.render(element);
```
#### 组合组件
组件可引用其它组件。
```js
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

function App(props) {
  return (
    <div>
      <Welcome name="Fa" />
      <Welcome name="Ge" />
      <Welcome name="Jodie" />
    </div>
  );
}
```
#### 拆分组件
#### props的只读性
React组件必须为纯函数，props是受保护的，不可以修改。
### State及生命周期
#### 为类添加本地State
```ts
class Clock extends React.Component {
  // 在constructor中传入props并初始化this.state
  // render函数中即可通过this.state访问状态数据
  constructor(props) {
    super(props);
    this.state = { date: new Date() };
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.data.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```
#### 为类添加生命周期方法
#### 正确使用State
##### 不要直接修改State
```js
// 错误做法
this.state.comment = 'Hello';

// 正确做法
this.setState({comment: 'Hello'});
```
只有在构造函数中才能直接为state赋值
##### state的更新可以是异步的
可以为`setState()`传入一个函数作为参数，该函数参数为前一state和当前props，从而避免依赖this.props和this.state的值来计算下一个state。
```js
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```
##### state将合并更新
React将合并当前state和`setState()`传入的参数对象。
#### 数据流下行
state可以看作是私有的，除了设置并拥有它的组件，其它组件无法访问它。
组件可以把state作为props向下传递给它的子组件。这通常被称为“自顶而下”或“单向”数据流。任何state都被特定组件拥有，由它派生出的数据或UI只能影响它“之下的”组件。
### 事件处理
- 使用驼峰式命名
- 在JSX中传入函数而不是字符串作为事件处理器
需要注意事件处理函数的this绑定
- 在constructor中使用bind绑定
- 使用公共类作用域语法定义
- 使用箭头函数调用
#### 向事件处理函数传参
- 使用箭头函数（事件参数e需要被显式传入）
- 使用bind
### 条件渲染
- if语句条件渲染
- &&条件渲染
- ?:条件渲染
#### 阻止组件渲染
在组件的render函数中返回null。
### 列表和keys