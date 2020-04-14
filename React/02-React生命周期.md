## React 生命周期

React V16.3之前的⽣命周期

![image-20200413074353248](file:///Users/zengchen/Desktop/2020/review/React/assets/image-20200413074353248.png?lastModify=1586735504)



V16.4之后的⽣命周期： V17可能会废弃的三个⽣命周期函数⽤ getDerivedStateFromProps替代，⽬前使⽤的话加上 UNSAFE_： componentWillMount componentWillReceiveProps componentWillUpdate 引⼊两个新的⽣命周期函数： static getDerivedStateFromProps getSnapshotBeforeUpdate

![image-20200413074454333](file:///Users/zengchen/Desktop/2020/review/React/assets/image-20200413074454333.png?lastModify=1586735504)

#### 变更缘由

原来（React v16.0前）的⽣命周期在React v16推出的Fiber之后 就不合适了，因为如果要开启async rendering，在render函数之前的所有函数，都有可能被执⾏多次。 

原来（React v16.0前）的⽣命周期有哪些是在render前执⾏的 呢？ componentWillMount componentWillReceiveProps shouldComponentUpdate componentWillUpdate

如果开发者开了async rendering，⽽且⼜在以上这些render前 执⾏的⽣命周期⽅法做AJAX请求的话，那AJAX将被⽆谓地多次 调⽤。明显不是我们期望的结果。

⽽且在 componentWillMount⾥发起AJAX，不管多快得到结果也赶不 上⾸次render，⽽且componentWillMount在服务器端渲染也会 被调⽤到（当然，也许这是预期的结果），这样的IO操作放在 componentDidMount⾥更合适。 

禁⽌不能⽤⽐劝导开发者不要这样⽤的效果更好，所以除了 shouldComponentUpdate，其他在render函数之前的所有函数 （componentWillMount，componentWillReceiveProps， componentWillUpdate）都被getDerivedStateFromProps替 代。 

也就是⽤⼀个静态函数getDerivedStateFromProps来取代被 deprecate的⼏个⽣命周期函数，就是强制开发者在render之前 只做⽆副作⽤的操作，⽽且能做的操作局限在根据props和state 决定新的state React v16.0刚推出的时候，是增加了⼀个componentDidCatch ⽣命周期函数，这只是⼀个增量式修改，完全不影响原有⽣命周 期函数；但是，到了React v16.3，⼤改动来了，引⼊了两个新 的⽣命周期函数。

新引⼊了两个新的⽣命周期函数：getDerivedStateFromProps，getSnaps hotBeforeUpdate getDerivedStateFromProps

```js
static getDerivedStateFromProps(props, state)
getSnapshotBeforeUpdate(prevProps, prevState)
class ScrollingList extends React.Component {
   constructor(props) {
     super(props);
     this.listRef = React.createRef();
   }
   getSnapshotBeforeUpdate(prevProps, prevState) {
     // 我们是否要添加新的 items 到列表?
     // 捕捉滚动位置，以便我们可以稍后调整滚动.
     if (prevProps.list.length < this.props.list.length) {
       const list = this.listRef.current;
       return list.scrollHeight - list.scrollTop;
     }
     return null;
   }
   componentDidUpdate(prevProps, prevState, snapshot) {
     // 如果我们有snapshot值, 我们已经添加了 新的items.
     // 调整滚动以⾄于这些新的items 不会将旧items推出视图。
     // (这边的snapshot是 getSnapshotBeforeUpdate⽅法的返回值)
     if (snapshot !== null) {
       const list = this.listRef.current;
       list.scrollTop = list.scrollHeight - snapshot;
     }
   }
   render() {
     return (
      <div ref={this.listRef}>{/* ...contents...*/}</div>
     );
   }
import React, { Component } from "react";
/*
V17可能会废弃的三个⽣命周期函数⽤
getDerivedStateFromProps替代，⽬前使⽤的话加上
UNSAFE_：
- componentWillMount
- componentWillReceiveProps
- componentWillUpdate
 */
export default class LifeCycle extends Component {
   constructor(props) {
    super(props);
    this.state = {
     	counter: 0,
   	};
 		console.log("constructor", this.state.counter);
 	 }
 	 static getDerivedStateFromProps(props, state) {
   // getDerivedStateFromProps 会在调⽤ render ⽅法之前调⽤，
   // 并且在初始挂载及后续更新时都会被调⽤。
   // 它应返回⼀个对象来更新 state，如果返回 null 则不更新任何内容。
   const { counter } = state;
   console.log("getDerivedStateFromProps", counter);
   return counter < 8 ? null : { counter: 0 };
 }
 getSnapshotBeforeUpdate(prevProps, prevState) {
 const { counter } = prevState;
 console.log("getSnapshotBeforeUpdate",
counter);
 return null;
 }
 /* UNSAFE_componentWillMount() {
 //不推荐，将会被废弃
 console.log("componentWillMount",
this.state.counter);
 } */
 componentDidMount() {
 console.log("componentDidMount",
this.state.counter);
 }
 componentWillUnmount() {
 //组件卸载之前
 console.log("componentWillUnmount",
this.state.counter);
 }
 /* UNSAFE_componentWillUpdate() {
 //不推荐，将会被废弃
 console.log("componentWillUpdate",
this.state.counter);
 } */
 componentDidUpdate() {
 console.log("componentDidUpdate",
this.state.counter);
 }
 shouldComponentUpdate(nextProps, nextState) {
 const { counter } = this.state;
 console.log("shouldComponentUpdate", counter,
nextState.counter);
 return counter !== 5;
 }
 setCounter = () => {
 this.setState({
 counter: this.state.counter + 1,
 });
 };
 render() {
 const { counter } = this.state;
 console.log("render", this.state);
 return (
 <div>
 <h1>我是LifeCycle⻚⾯</h1>
 <p>{counter}</p>
 <button onClick={this.setCounter}>改变
counter</button>
 {/* {!!(counter % 2) && <Foo />} */}
 <Foo counter={counter} />
 </div>
 );
 }
}
class Foo extends Component {
 UNSAFE_componentWillReceiveProps(nextProps) {
 //不推荐，将会被废弃
 // UNSAFE_componentWillReceiveProps() 会在已挂
载的组件接收新的 props 之前被调⽤
 console.log("Foo componentWillReceiveProps");
 }
 componentWillUnmount() {
 //组件卸载之前
 console.log(" Foo componentWillUnmount");
 }
 render() {
 return (
 <div
 style={{ border: "solid 1px black",
margin: "10px", padding: "10px" }}
 >
 我是Foo组件
 <div>Foo counter: {this.props.counter}
</div>
 </div>
 );
 }
}
```

范例：创建Lifecycle.js

#### 验证⽣命周期

在上述示例中，重点是从 getSnapshotBeforeUpdate 读取 scrollHeight 属性，因为 “render” 阶段⽣命周期（如 render）和 “commit” 阶段⽣命周期（如 getSnapshotBeforeUpdate 和 componentDidUpdate）之间 可能存在延迟。

官⽹给的例⼦：



在render之后，在componentDidUpdate之前。 getSnapshotBeforeUpdate() 在最近⼀次渲染输出（提交到 DOM 节点）之前调⽤。它使得组件能在发⽣更改之前从 DOM 中捕获⼀些信息（例如，滚动位置）。此⽣命周期的任何返回值 将作为参数传递给 componentDidUpdate()。 此⽤法并不常⻅，但它可能出现在 UI 处理中，如需要以特殊⽅ 式处理滚动位置的聊天线程等。 应返回 snapshot 的值（或 null）。

getSnapshotBeforeUpdate



static getDerivedStateFromProps(props, state) 在组件创 建时和更新时的render⽅法之前调⽤，它应该返回⼀个对象来更 新状态，或者返回null来不更新任何内容。

React v16.4后的getDerivedStateFromProps



这样的话理解起来有点乱，在React v16.4中改正了这⼀点，让 getDerivedStateFromProps⽆论是Mounting还是Updating， 也⽆论是因为什么引起的Updating，全部都会被调⽤，具体可看 React v16.4 的⽣命周期图。

React v16.3

![image-20200413074649167](file:///Users/zengchen/Desktop/2020/review/React/assets/image-20200413074649167.png?lastModify=1586735504)

React v16.3 的⽣命周期图

getDerivedStateFromProps 会在调⽤ render ⽅法之前调 ⽤，并且在初始挂载及后续更新时都会被调⽤。它应返回⼀个对 象来更新 state，如果返回 null 则不更新任何内容。 请注意，不管原因是什么，都会在每次渲染前触发此⽅法。这与 UNSAFE_componentWillReceiveProps 形成对⽐，后者仅在⽗ 组件重新渲染时触发，⽽不是在内部调⽤ setState 时。