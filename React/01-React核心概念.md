# React 核心概念

## 文档

[React](https://reactjs.org/)

[create-react-app](https://create-react-app.dev/docs/getting-started/)

[JSX](https://zh-hans.reactjs.org/docs/introducing-jsx.html)



## 项目启动

1. 安装官⽅脚⼿架：`npm install -g create-react-app`

2. 创建项⽬：`create-react-app my-react`  或者不执⾏步骤1，直接执⾏ `npx create-react-app lesson1 `
  
3. 打开项目： `cd my-react`

4. 启动项⽬：`npm start`

5. 暴露配置项：`npm run eject`

如果出现错误类似`/babel-preset-reactapp/node_modules/@babel/runtime/helpers/slicedToArray at`
  `webpackMissingModule`，就 `npm add @babel/runtime`

env.js⽤来处理.env⽂件中配置的环境变量

```js
// config/env.js
// node运⾏环境：development、production、test等
const NODE_ENV = process.env.NODE_ENV;
// 要扫描的⽂件名数组
const dotenvFiles = [
  `${paths.dotenv}.${NODE_ENV}.local`,
  `${paths.dotenv}.${NODE_ENV}`,
  // Don't include `.env.local` for `test` environment
  // since normally you expect tests to produce the same
  // results for everyone
  NODE_ENV !== 'test' && `${paths.dotenv}.local`,
  paths.dotenv,
].filter(Boolean);
// 从.env*⽂件加载环境变量
dotenvFiles.forEach(dotenvFile => {
  if (fs.existsSync(dotenvFile)) {
    require('dotenv-expand')(
      require('dotenv').config({
        path: dotenvFile,
      })
    );
  }
});
```

实践⼀下，修改⼀下默认端⼝号，在根目录创建.env⽂件

```js
PORT=8080
```

webpack.config.js 是webpack配置⽂件，开头的常量声明可以看出cra能够⽀持ts、sass及css模块化

```js
// Check if TypeScript is setup
const useTypeScript = fs.existsSync(paths.appTsConfig);
// style files regexes
const cssRegex = /\.css$/;
const cssModuleRegex = /\.module\.css$/;
const sassRegex = /\.(scss|sass)$/;
const sassModuleRegex = /\.module\.(scss|sass)$/;
```



## React 和 ReactDom

删除src下⾯所有代码，新建index.js

```js
import React from 'react';
import ReactDOM from 'react-dom';
// 这⾥怎么没有出现React字眼？
// JSX => React.createElement(...)
ReactDOM.render(<h1>Hello React</h1>,
document.querySelector('#root'));
```

React负责逻辑控制，数据 -> VDOM

ReactDom渲染实际DOM，VDOM -> DOM

React使⽤JSX来描述UI

React⼊⼝⽂件定义，可以从 `webpack.config.js` 中找到：

```js
entry: [
	// WebpackDevServer客户端，它实现开发时热更新功能
	isEnvDevelopment && require.resolve('react-dev-utils/webpackHotDevClient'),
  // 应⽤程序⼊⼝：src/index
  paths.appIndexJs,
].filter(Boolean),
```



## JSX

### 什么是JSX

JSX是⼀种`JavaScript`的语法扩展，其格式⽐较像模版语⾔，但事实上完全是在`JavaScript`内部实现的。
JSX可以很好地描述UI，能够有效提⾼开发效率。

JSX实质就是`React.createElement`的调⽤，最终的结果是React "元素"（JavaScript对象）。

```js
const jsx = <h2>react study</h2>;
ReactDOM.render(jsx, document.getElementById('root'));
```

### 使⽤JSX

表达式{}的使⽤，index.js

```js
const name = "react study";
const jsx = <h2>{name}</h2>;
```

函数也是合法表达式，index.js

```js
const user = { firstName: "tom", lastName: "jerry" };
function formatName(user) {
  return user.firstName + " " + user.lastName;
}
const jsx = <h2>{formatName(user)}</h2>;
```

jsx是js对象，也是合法表达式，index.js

```js
const greet = <p>hello, Jerry</p>
const jsx = <h2>{greet}</h2>;
```

条件语句可以基于上⾯结论实现，index.js

```js
const showTitle = true;
const title = showTitle ? <h2>{name}</h2> : null;
const jsx = (
 <div>
 	{/* 条件语句 */}
 	{title}
 </div>
);
```

数组会被作为⼀组⼦元素对待，数组中存放⼀组jsx可⽤于显示列表数据

```js
const arr = [1,2,3].map(num => <li key={num}>{num}</li>)
const jsx = (
 <div>
 	{/* 数组 */}
 	<ul>{arr}</ul> 
 </div>
);
```

属性的使⽤

```js
import logo from "./logo.svg";
const jsx = (
 <div>
 	{/* 属性：静态值⽤双引号，动态值⽤花括号；class、for等要特殊处理。 */}
 	<img src={logo} style={{ width: 100 }} className="img" />
 </div>
);
```

css模块化，创建index.module.css，index.js

```js
import style from "./index.module.css";
<img className={style.img} />
```

或者npm install node-sass -D

```js
import style from "./index.module.scss";
<img className={style.img} />
```

更多css modules规则参考 [CSS Modules 用法教程](http://www.ruanyifeng.com/blog/2016/06/css_modules.html)



## 组件

组件是抽象的独⽴功能模块，react应⽤程序由组件构建⽽成。

### 组件分类

组件有两种形式：function组件和class组件。

#### class组件

class组件通常拥有状态和⽣命周期，继承于Component，实现render⽅法，创建pages/ClassComp.js

```js
import React, { Component } from 'react'

export default class ClassComp extends Component {
  render() {
    return (
      <div>
        This is a class component.
      </div>
    )
  }
}
```

创建并指定src/App.js为根组件

```js
import React from 'react';
import ClassComp from './pages/ClassComp.js'

function App() {
  return (
    <div className="App">
      <ClassComp></ClassComp>
    </div>
  );
}

export default App;
```

index.js中使⽤App组件

```js
import App from "./App";
ReactDOM.render(<App />, document.getElementById("root"));
```

#### function组件

函数组件通常⽆状态，仅关注内容展示，返回渲染结果即可。

```js
import React from 'react'

function FuncComp() {
  return (
    <div>函数组件</div>
  )
}

export default FuncComp
```

从React16.8开始引⼊了hooks，函数组件也能够拥有状态。

### 组件状态管理

如果组件中数据会变化，并影响⻚⾯内容，则组件需要拥有状态（state）并维护状态。

#### 类组件中的状态管理

class组件使⽤state和setState维护状态

创建⼀个Clock

```js
import React, { Component } from 'react'

export default class Clock extends Component {
  constructor(props) {
    super(props)
    // 使⽤state属性维护状态，在构造函数中初始化状态
    this.state = {
      currentDate: new Date()
    }
  }
  componentDidMount() {
    this.timerId = setInterval(() => {
      this.setState({
        currentDate: new Date()
      })
    }, 1000)
  }
  componentWillUnmount() {
    clearInterval(this.timerId)
  }
  render() {
    return (
      <div>
        {this.state.currentDate.toLocaleTimeString()}
      </div>
    )
  }
}
```



**setState特性**

⽤`setState`更新状态⽽不能直接修改

```js
this.state.counter += 1; //错误的
```

setState是批量执⾏的，因此对同⼀个状态执⾏多次只起⼀次作⽤，多个状态更新可以放在同⼀个setState中进
⾏：

```js
componentDidMount() {
	// 假如couter初始值为0，执⾏三次以后其结果是多少？
	this.setState({counter: this.state.counter + 1});
	this.setState({counter: this.state.counter + 2});
}
```

setState通常是异步的，因此如果要获取到最新状态值有以下三种⽅式：

传递函数给setState⽅法

```js
this.setState((nextState, props) => ({counter: state.counter + 1}));// 1
this.setState((nextState, props) => ({counter: state.counter + 1}));// 2
this.setState((nextState, props) => ({counter: state.counter + 1}));// 3
```

使⽤定时器：

```js
setTimeout(() => {
	this.changeValue();
 	console.log(this.state.counter);
}, 0);
```

原⽣事件中修改状态

```js
componentDidMount(){
  document.body.addEventListener('click', this.changeValue, false)
}
changeValue = () => {
  this.setState({counter: this.state.counter+1})
  console.log(this.state.counter)
}
```

总结：

`setState`只有在合成事件和⽣命周期函数中是异步的，在原⽣事件如`addEventListener`和`setTimeout`、`setInterval`中都是同步的。

原因：

原⽣事件绑定不会通过合成事件的⽅式处理，⾃然也不会进⼊更新事务的处理流程。`setTimeout`也⼀样，在`setTimeout`回调执⾏时已经完成了原更新组件流程，也不会再进⼊异步更新流程，其结果⾃然就是是同步的了。

#### 函数组件中的状态管理

函数组件通过hooks api维护状态

```js
import React, { useState, useEffect } from 'react'

export default function ClockHook() {
  const [date, setDate] = useState(new Date())
  useEffect(() => {
    const timerId = setInterval(() => {
      setDate(new Date())
    }, 1000)
    return () => clearInterval(timerId)
  })
  return <div>{date.toLocaleTimeString()}</div>
}
```

可以把 `useEffect Hook` 看做`componentDidMount`，`componentDidUpdate` 和 `componentWillUnmount` 这三个函数的组合。


## 事件处理

React中使⽤onXX写法来监听事件。
例如：⽤户输⼊事件，创建 Input.js

```js
import React, { Component } from 'react'

export default class Input extends Component {
  constructor(props) {
    super(props)
    this.state = {
      inputValue: ''
    }
    // this.change = this.change.bind(this)
  }
  btn = () => {
    console.log('btn')
  }
  // 使⽤箭头函数，不需要指定回调函数this，且便于传递参
  change = e => {
    this.setState({
      inputValue: e.target.value
    })
  }
  render() {
    return (
      <div>
        <p>{this.state.inputValue}</p>
        <button onClick={this.btn}>按钮</button>
        <input type="text" onChange={this.change} value={this.state.inputValue} />
      </div>
    )
  }
}
```

事件回调函数注意绑定 `this` 指向，常⻅三种⽅法：
1. 构造函数中绑定并覆盖：`this.change = this.change.bind(this)`
2. ⽅法定义为箭头函数：`change = () => {}`
3. 事件中定义为箭头函数：`onChange = { () => this.change() }`
react⾥遵循单项数据流，没有双向绑定，输⼊框要设置 `value` 和 `onChange`，称为受控组件



## 组件通信

### props属性传递

Props属性传递可⽤于⽗⼦组件相互通信

```js
// index.js
ReactDOM.render(<App title="今天天气真不错" />, document.querySelector('#root'));
// App.js
<h2>{this.props.title}</h2>
```

如果⽗组件传递的是函数，则可以把⼦组件信息传⼊⽗组件，这个常称为状态提升，StateMgt.js

```js
import React, { Component } from 'react'

class Clock extends Component {
  constructor(props) {
    super(props)
    this.state = {
      date: new Date()
    }
  }
  componentDidMount() {
    this.timerId = setInterval(() => {
      this.setState({
        date: new Date()
      }, () => {
        // 每次状态更新就通知⽗组件, 状态提升
        this.props.change(this.state.date)
      })
    }, 1000)
  }
  render() {
    return (
      <div>{this.state.date.toLocaleTimeString()}</div>
    )
  }
}

export default class StateMgt extends Component {
  handleChange(props) {
    // console.log(props)
  }
  render() {
    return (
      <div>
        <Clock change={this.handleChange}></Clock>
      </div>
    )
  }
}
```

hooks写法：

```js
import React, { useState, useEffect, Component } from 'react'

function Clock(props) {
  const [date, setDate] = useState(new Date())
  useEffect(() => {
    const timerId = setInterval(() => {
      setDate(new Date())
      props.change(date)
    }, 1000)
    return () => {
      clearInterval(timerId)
    }
  })
  return <div>{date.toLocaleTimeString()}</div>
}

export default class StateMgt extends Component {
  handleChange(props) {
    // console.log(props)
  }
  render() {
    return (
      <div>
        <Clock change={this.handleChange}></Clock>
      </div>
    )
  }
}
```

### context

跨层级组件之间通信，主要⽤于组件库开发中

### redux

类似vuex，⽆明显关系的组件间通信



