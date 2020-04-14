# React 组件化

## 文档

[Context](https://reactjs.org/docs/context.html)
[HOC](https://reactjs.org/docs/higher-order-components.html)
[Hooks](https://zh-hans.reactjs.org/docs/hooks-intro.html)
[antD](https://ant.design/docs/react/practical-projects-cn)
[umi](https://umijs.org/zh-CN/docs)

[AntD Pro](https://pro.ant.design/index-cn)



## 什么是组件化

组件化优点：
1. 增强代码重⽤性，提⾼开发效率
2. 简化调试步骤，提升整个项⽬的可维护性
3. 便于协同开发
4. 注意点：降低耦合性



## 组件跨层级通信 - Context

React中使⽤Context实现祖代组件向后代组件跨层级传值。Vue中的
provide & inject来源于Context
在Context模式下有两个⻆⾊：
Provider：外层提供数据的组件
Consumer ：内层获取数据的组件

### 使⽤Context

创建Context => 获取Provider和Consumer => Provider提供值 =>
Consumer消费值
范例：模拟redux存放全局状态，在组件间共享











### 高阶组件 - HOC





### 组件复合 - Composition



### Hooks

#### 状态钩子 State Hook



#### 副作用钩子 Effect Hook



#### useReducer



#### useContext



#### Hooks相关拓展