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
#### 循环渲染多个组件
使用`map()`函数
#### keys
未明确指定时React默认使用索引号为keys。
keys在就近数组上下文（兄弟节点）中生效。
keys在兄弟节点中必须是唯一的，但无需是全局唯一的。
keys不会作为属性传递。
### 表单
#### 受控组件
#### textarea标签
在React中，文本值为标签的value属性值。
#### select标签
在React中，选中值为根select标签的value属性值。
设置multiple值为true并传入数组为value值，则select可实现多选。
#### 文件输入标签
在React中，它是一个非受控组件。
#### 处理多个输入
为每个输入元素添加`name`属性，并根据`event.target.name`来进行相应操作。
#### null值受控输入
将`value`设为`undefined`或`null`将导致受控组件可编辑。
#### 受控组件替代品
[非受控组件](https://reactjs.org/docs/uncontrolled-components.html)
#### 成熟解决方案
[Formik](https://formik.org/)
#### 状态提升
将共享状态提升至最近的共同祖先组件。
```ts
const scaleNames = {
  c: 'Celsius',
  f: 'Fahrenheit'
};

class TemperatureInput extends React.Component {
  // temperature, scale接受自父组件
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange(e) {
    this.props.onTemperatureChange(e.target.value);
  }

  render() {
    const temperature = this.props.temperature;
    const scale = this.props.scale;
    return (
      <fieldset>
        <legend>Enter temperature in {scaleNames[scale]}: </legend>
        <input value={temperature}
               onChange={this.handleChange} />
      </fieldset>
    )
  }
}

class BoilingVerdict extends React.Component {
  constructor(props) {
    super(props);
  }

  render() {
    if (this.props.celsius >= 100) {
      return <p>The water would boil.</p>
    }
      return <p>The water would not boil.</p>
  }
}

const toCelsius = (fahrenheit) => {
  return (fahrenheit - 32) * 5 / 9;
};
const toFahrenheit = (celsius) => {
  return (celsius * 9 / 5) + 32;
};
const tryConvert = (temperature, convert) => {
  const input = parseFloat(temperature);
  if (Number.isNaN(input)) {
    return '';
  }
  const output = convert(input);
  const rounded = Math.round(output * 1000) / 1000;
  return rounded.toString();
};

class Calculator extends React.Component {
  // 将temperature提升到父组件中，并通过props向下传给子组件
  constructor(props) {
    super(props);
    this.handleCelsiusChange = this.handleCelsiusChange.bind(this);
    this.handleFahrenheitChange = this.handleFahrenheitChange.bind(this);
    this.state = {temperature: '', scale: 'c'};
  }

  handleCelsiusChange(temperature) {
    this.setState({scale: 'c', temperature})
  }
  handleFahrenheitChange(temperature) {
    this.setState({scale: 'f', temperature})
  }

  render() {
    const scale = this.state.scale;
    const temperature = this.state.temperature;
    const celsius = scale === 'f' ? tryConvert(temperature, toCelsius) : temperature;
    const fahrenheit = scale === 'c' ? tryConvert(temperature, toFahrenheit) : temperature;
    return (
      <div>
        <TemperatureInput
          scale="c"
          temperature={celsius}
          onTemperatureChange={this.handleCelsiusChange} />
        <TemperatureInput
          scale="f"
          temperature={fahrenheit}
          onTemperatureChange={this.handleFahrenheitChange} />
        <BoilingVerdict celsius={parseFloat(celsius)} />
      </div>
    );
  }
}
```
### 组合与继承
#### 包含关系
内部包含块：通过特殊属性`props.children`传递。
多个块：自定义属性传递。
#### 特例关系
属性传递
### React哲学
1. 组件层级拆分
2. 静态版本构建
3. 明确最小UI State集
4. 明确State位置
5. 添加反向数据流
## 高级
### 无障碍
#### 标准
[wcag](https://www.w3.org/WAI/intro/wcag)
[wai-aria](https://www.w3.org/WAI/intro/aria)
#### 语义化html
#### 无障碍表单
**标记**
**错误提醒**
#### 焦点控制
**键盘焦点，焦点轮廓** 在只有键盘的环境下可以正常工作，也不要去除焦点轮廓
**内容跳过** 跳转链接，跳过导航段落，地标元素
**程序控制焦点** 使用DOM元素的Refs
#### 鼠标和指针事件
#### 复杂部件
#### 其它因素
**设置语言**
**设置标题**
**色彩对比度**
### 代码分割
#### 打包
#### 代码分割
**import()动态引入**
**React.lazy** 只支持默认导出
#### 基于路由的代码分割
React Router
#### 命名导出
### Context
组件间共享值，“全局”数据，如当前用户、主题、语言等。
#### 何时使用
在应用中全局需要。
示例：
```ts
// 为主题创建一个context，默认值为'light'
const ThemeContext = React.createContext('light');

// 使用Provider传递
class App extends React.component {
  constructor(props) {
    super(props);
  }

  render() {
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext>
    )
  }
}

// 子组件无需从props中接收该属性
function Toolbar() {
  return (
    <div>
      <ThemeButton></ThemeButton>
    </div>
  )
}

// 后代组件可访问该Context数据
class ThemeButton extends React.component {
  constructor(props) {
    super(props);
  }

  static contextType = ThemeContext;
  render() {
    return <Button theme={this.context} />;
  }
}
```
#### API
```ts
// React.createContext：创建Context对象，参数：默认值
const MyContext = React.createContext(defaultValue);

// Context.Provider：value值提供，value变化时，消费组件都会重新渲染
<MyContext.Provider value={/* value */}></MyContext.Provider>

// Class.contextType：赋值为一个Context对象后可通过this.context来消费最近的Context值
// 可在组件各生命周期中访问
class MyClass extends React.Component {
  constructor(props) {
    super(props);
  }

  componentDidMount() {
    let value = this.context;
    // ...
  }
  componentDidUpdate() {
    let value = this.context;
    // ...
  }
  componentWillUnmount() {
    let value = this.context;
    // ...
  }
  render() {
    let value = this.context;
    // ...
  }
}
MyClass.contextType = MyContext;

// Context.Consumer：消费组件，订阅context变更
<Context.Consumer>
  {value => /* 渲染内容 */}
</Context.Consumer>

// Context.displayName：React DevTools中显示的Context名
MyContext.displayName = 'MyDisplayName';
<MyContext.Provider>   // DevTools中："MyDisplayName.Provider"
<MyContext.Consumer>   // DevTools中："MyDisplayName.Consumer"
```
#### 示例
##### 动态Context
##### 嵌套组件中更新Context
##### 消费多个Context
嵌套，每个消费组件都须是独立节点。
```ts
// Provider
<ThemeContext.Provider value={theme}>
  <UserContext.Provider value={user}>
    <Layout />
  </UserContext.Provider>
</ThemeContext.Provider>


// Consumer
<ThemeContext.Consumer >
  {(theme) => (
    <UserContext.Consumer>
      {(user) => (
        {/*  */}
      )}
    </UserContext.Consumer>
  )}
</ThemeContext.Consumer>
```
### 错误边界
捕获并打印子组件中JS错误并渲染备用UI的React组件。
作用范围：渲染期间、生命周期方法、组件树构造函数。
无法捕获的场景：
- 事件处理
- 异步回调
- ssr
- 自身抛出的错误
实现：在组件class中定义`static getDerivedStateFromError()`或`componentDidCatch()`。前者用来渲染备用UI，后者用来打印错误信息。
#### 未捕获错误新行为
React16开始，未被错误边界捕获的错误将导致整个React组件树被卸载。
#### vs try/catch
### Refs转发
#### 转发refs到DOM组件
```ts
// 通过ref参数传递
const FancyButton = React.forwardRef((props, ref) => (
  // 通过ref接收
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));


// 创建ref并赋值给ref变量
const ref = React.createRef();
// 通过ref传给FancyButton
<FancyButton ref={ref}>click me.</FancyButton>
```
挂载完成后，ref.current指向`<button>`这个DOM节点。
#### 高阶组件中转发refs
使用`React.forwardRef`将refs转发到内部组件。
#### 在DevTools中显示自定义名称
- 命名渲染函数
- 设置函数displayName
### Fragments
返回多个元素，而无需添加新的DOM节点，示例：
```ts
class Columns extends React.Component {
  constructor(props) {
    super(props);
  }

  render() {
    return (
      <React.Fragment>
        <td>1</td>
        <td>2</td>
      </React.Fragment>
    );
  }
}

class Table extends React.Component {
  constructor(props) {
    super(props);
  }

  render() {
    return (
      <table>
        <tr>
          <Columns />
        </tr>
      </table>
    );
  }
}
```
简写语法：`<></>`
唯一可传递给Fragment的属性：key。
### 高阶组件（HOC，Higher Order Components）
高阶组件为参数是组件，返回值为新组件的函数。
```ts
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```
#### 约定：将不相关的props传递给被包裹元素
```js
render() {
  // extraProps：无需透传的属性
  // passThroughProps：剩余属性，将透传
  const { extraProps, ...passThroughProps } = this.props;

  const injectedProp = someStateOrInstanceMethod;

  return (
    <WrappedComponent injectedProp={injectedProp} {...passThroughProps} />
  )
}
```
### 第三方协同
### 深入JSX
JSX可以看作React.createElement(component, props, ...children)语法糖。
```ts
<MyButton color='blue' shadowSize={2}>
  Click Me.
</MyButton>

// 等同于
React.createElement(
  MyButton,
  { color: 'blue', shadowSize: 2 },
  'Click Me'
);
```
### 性能优化
- **使用生产版本**
- **使用构建工具**
- **使用开发者工具**
- **长列表虚拟滚动**
- **渲染优化** 覆写`shouldComponentUpdate`生命周期方法
### Portals
一种将子节点渲染到父组件以外DOM节点的方案。
```js
ReactDOM.createPortal(child, container);
```
#### 通过Portal进行事件冒泡
从portal内部触发的事件将沿React树向上冒泡，即使它们不是其DOM树上的祖先。
### Profiler API
```js
// 一个Profiler组件接收两个prop：id，onRender回调
render(
  <App>
    <Profiler id="Navigation" onRender={callback}>
      <Navigation {...props}></Navigation>
    </Profiler>
  </App>
)
```
#### onRender回调
```js
function onRenderCallback(
  id,                // 发生提交的Profiler树的id
  phase,             // 刚加载：mount；重渲染：update
  actualDuration,    // 本次更新committed花费的渲染时长
  baseDuration,      // 估计不使用memoization的情况下渲染整棵子树需要的时间
  startTime,         // 本次更新中React开始渲染的时间
  commitTime,        // 本次更新中React committed的时间
  interactions       // 本次更新的interactions的集合
) {
  // 具体操作
}
```
### 不使用ES6
#### React类创建
引入create-react-class模块
```js
var createReactClass = require('create-react-class');
var Greeting = createReactClass({
  render: function() {
    return <h1>hello, {this.props.name}</h1>
  }
})
```
#### 声明默认props
通过`getDefaultProps`函数声明
```js
var Greeting = createReactClass({
  getDefaultProps: function() {
    return {
      name: 'Fa'
    };
  }
});
```
#### 设置初始State
通过`getInitialState`函数设置
```js
var Counter = createReactClass({
  getInitialState: function() {
    return {count: this.props.initialCount};
  }
})
```
#### 自动绑定
`createReactClass()`将自动绑定所有方法：
```js
var sayHello = createReactClass({
  getInitialState: function() {
    return {message: 'hello'};
  },

  handleClick: function() {
    alert(this.state.message);
  },

  render: function(params) {
    return (
      <button onClick={this.handleClick}>say hello</button>
    )
  }
})
```
#### Mixins
### 不使用JSX
使用原始`React.createElement()`语法
### Reconciliation
#### 设计动机
关于diff的启发式算法：
- 两个不同类型的元素会产生不同的树
- 可以通过`key`属性来暗示哪些子元素在不同的渲染下保持稳定
#### Diffing算法
- 对比不同类型的元素
- 对比相同类型的DOM元素
- 对比相同类型的组件元素

## Hook
### Hook规则
### React内置Hook
**useState** 返回state及更新state的函数。
```ts
// 基本用法
const [state, setState] = useState(initialState);
setState(newState);

// 函数式更新
setState(prevState => getNewStateFunc(prevState));

// 对象合并
const [state, setState] = useState({});
setState(prevState => {
  return {...prevState, ...updatedValues};
});
```
**useEffect** 接收一个包含命令式，且可能有副作用代码的函数。该函数返回值为清除函数。
```ts
// 基本形式
useEffect(() => {
  effect
  return () => {
    cleanup
  }
}, [input])

// 实例
useEffect(() => {
  // 订阅
  const subscription = props.source.subscribe();
  return () => {
    // 清除订阅
    subscription.unsubscribe()
  };
});

// 条件执行
useEffect(() => {
  () => {
    const subscription = props.source.subscribe();
    return () => {
      // 清除订阅
      subscription.unsubscribe();
    };
  },
  [props.source],
);
```
执行时机：浏览器完成布局与绘制后延迟调用useEffect函数。
依赖数组应包含effect中使用的所有外部作用域中会发生变化的变量。
**useContext** 接收一个context对象并返回该context的当前值。
```ts
const themes = {
  light: {
    foreground: "#000000",
    background: "#eeeeee"
  },
  dark: {
    foreground: "#ffffff",
    background: "#222222"
  }
};

const ThemeContext = React.createContext(themes.light);

function App() {
  return (
    <ThemeContext.Provider value={themes.dark}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  )
}

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return (
    <button style={{ background: theme.background, color: theme.foreground }}>
      styled by theme context
    </button>
  )
}
```
**useReducer** useState替代方案，接收形如(state, action) => newState的reducer函数，返回当前state及dispatch方法。
```ts
// 基本用法
const [state, dispatch] = useReducer(reducer, initialArg, init);

// 实例
const initialState = { count: 0 };
function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count - 1};
    case 'decrement':
      return {count: state.count + 1};
    default:
      throw new Error()
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState)
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
    </>
  );
}

// 惰性初始化：传入init函数
function init(initialCount) {
  return {count: initialCount};
}
function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count - 1};
    case 'decrement':
      return {count: state.count + 1};
    case 'reset':
      return {count: action.payload};
    default:
      throw new Error();
  }
}

function Counter(initialCount) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'reset', payload: initialCount})}>Reset</button>
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
    </>
  );
}
```
**useCallback** 返回一个memoized回调函数。
```ts
// useCallback(fn, deps)

const memoizedCallback = useCallback(
  () => {
    doSomething()
  },
  [a, b],
);
```
**useMemo** 返回一个memoized值。传入“创建”函数和依赖项，仅在依赖项改变时才重新计算memoized值。
```ts
const memoizedValue = useMemo(
  () => computeExpensiveValue(a, b),
  [a, b]
);
```
传入useMemo的函数将在渲染期间执行。
**useRef** 返回一个可变ref对象，其.current属性初始化为传入的参数，返回的ref对象在组件的整个生命周期内存在。
```ts
function TextInputWithFocusButton() {
  const inputEl = useRef(null);

  const onButtonClick = () => {
    inputEl.current.focus();
  }

  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```
它适用于所有变量而不仅仅是组件的ref属性值。
**useImperativeHandle** 可以让你在使用ref时自定义暴露给父组件的实例值，需结合forwardRef一起使用。
```ts
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeHandle(ref, () => (
    {
      focus: () => {
        inputRef.current.focus();
      }
    }));
  return <input ref={inputRef} ... />;
}

FancyInput = forwardRef(FancyInput);
// 父组件<FancyInput ref={inputRef} />可以调用inputRef.current.focus()
```
**useLayoutEffect** 函数签名与useEffect相同，但是执行时机为所有DOM变更后同步调用effect。
**useDebugValue** 用于在React开发者工具中显示自定义hook标签。
```ts
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  // ...

  // 在开发者工具中的这个 Hook 旁边显示标签
  // e.g. "FriendStatus: Online"
  useDebugValue(isOnline ? 'Online' : 'Offline');

  return isOnline;
}
```
