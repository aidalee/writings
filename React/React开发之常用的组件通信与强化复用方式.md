## 常用通信方式
- 父传子 `props`
- 子传父 `callback` 通过父组件传递的 callback 函数，通知父组件

## 祖孙跨层级通信
- `context`方式 类似vue提供的`provide`、`inject`
主要使用方法：
1. 创建 Context：React.createContext()
2. Provider：提供者，外层提供数据的组件
```js
// 提供方式：
<Context.provider {...props}></Context.provider>
```
3. Consumer ：消费者，内层获取数据的组件
```js
// 消费方式：
// static contextType = Contenxt class组件中
// Hooks：useContext // 函数式组件中
```
## 常用的强化组件方式

### HOC 高阶组件
> 如果一个组件接收的参数是一个组件，并且返回也是一个组件，那么这个组件就是高阶组件（HOC）
> hoc进阶：HOC 可以做很多事情，比如强化 props、条件渲染、性能优化、事件赋能、反向继承等具体可参看：
- https://juejin.cn/post/7103345085089054727
- https://juejin.cn/post/71215517017314099343

### 自定义Hooks模式强化组件复用逻辑
> Hooks 是 React v16.8 以后新增的 API，目的是增加代码的可复用性、逻辑性，最主要的目的是解决了函数式组件无状态的问题，这样既保留了函数式的简单，又解决了没有数据管理状态的缺陷。自定义Hooks 辅助组件，让其开发更加丝滑、简洁、维护性更高。

### extends 继承模式（不常用）
```js
    import React from "react";
    import { Button } from "antd";

    class Child extends React.Component<any, any> {
      constructor(props: any) {
        super(props);
        this.state = {
          msg: "test extends",
        };
      }

      talk() {
        console.log("Child中的talk");
      }

      render() {
        return (
          <>
            <div>{this.state.msg}</div>
            <Button type="primary" onClick={() => this.talk()}>
              点击
            </Button>
          </>
        );
      }
    }

    class Index extends Child {
      talk() {
        console.log("extends中的talk");
      }
    }

    export default Index;


```

### 为什么使用函数式组件？class组件的缺陷有哪些
1. super的传递
> 实际上 super 的作用等于执行 Component 函数，如果不使用 super() 就会导致 Component 函数内的 props 找不到，也就是在代码中使用 this.props 打印出 undefined，所以这段代码是必要的。

```js
class Child extends React.Component<any, any> {
  constructor(props: any) {
    super(props) // 如果直接调用(如 super() )的结果又是什么？
    this.state = {
      name: 'test'
    }
  }
}
```
2. 奇怪的`this`指向
> 之所以 Class 组件需要 bind 的根本原因是在 React 的事件机制中，dispatchEvent 调用的 invokeGuardedCallback 是直接使用的 func，并没有指定调用的组件，所以在 Class 组件中的方法都需要手动绑定 this。
> 而箭头函数本身并不会创建自己的 this，它会继承上层的 this，所以不需要进行绑定，this 本身就是指向的组件。

3. 繁琐的生命周期
> 在 Class 组件中有很多关于生命周期的 API，以此用来数据管理，主要的版本分为 v16.0 和 v16.4，如：componentDidMount、getDerivedStateFromProps(prevProps, prevState) 等，大概有 9 个 API，我们要完全掌握生命周期的用法，显然并不是一件容易的事
> 从代码中看到函数式组件并没有 super，更不用关心 this 的指向，相对于类中繁琐的生命周期，都可以使用 useState、useEffect等 Hooks 代替，极大地降低了代码数量、行数，使其更容易上手，代码更加简洁、规整、维护性更高。
