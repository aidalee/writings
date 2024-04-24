
## 理解 jsx 的编译过程，方便操纵 children、控制 React 渲染，有利于便捷使用 React 插槽组件

> 一段jsx代码会被babel编译成如下

```js
React.createElement(
  type,
  [props],
  [...children]
)
```
> 依次被转换为：

```jsx
// 1:
<div>
   <TextComponent />
   <div>hello,world</div>
   let us learn React!
</div>
// 2:
React.createElement("div", null,
  React.createElement(TextComponent, null),
  React.createElement("div", null, "hello,world"),
  "let us learn React!"
)
// 3:
{
  $$typeof: Symbol(react.element),
  key: null,
  ref: null,
  props: {
    children: [子元素1, 子元素2, ...]
  },
  style: { ... }
}

```
> 最终，在调和阶段，上述 React element 对象的每一个子节点都会形成一个与之对应的 fiber 对象，然后通过 sibling、return、child 将每一个 fiber 对象联系起来。
> 所以，我们有必要先来看一下 React 常用的 fiber 类型，以及 element 对象和 fiber 类型的对应关系。

### 不同种类的 fiber Tag
> React 针对不同 React element 对象会产生不同 tag (种类) 的fiber 对象。tag 与 element 的对应关系：

```jsx
export const FunctionComponent = 0;       // 函数组件
export const ClassComponent = 1;          // 类组件
export const IndeterminateComponent = 2;  // 初始化的时候不知道是函数组件还是类组件 
export const HostRoot = 3;                // Root Fiber 可以理解为根元素 ， 通过reactDom.render()产生的根元素
export const HostPortal = 4;              // 对应  ReactDOM.createPortal 产生的 Portal 
export const HostComponent = 5;           // dom 元素 比如 <div>
export const HostText = 6;                // 文本节点
export const Fragment = 7;                // 对应 <React.Fragment> 
export const Mode = 8;                    // 对应 <React.StrictMode>   
export const ContextConsumer = 9;         // 对应 <Context.Consumer>
export const ContextProvider = 10;        // 对应 <Context.Provider>
export const ForwardRef = 11;             // 对应 React.ForwardRef
export const Profiler = 12;               // 对应 <Profiler/ >
export const SuspenseComponent = 13;      // 对应 <Suspense>
export const MemoComponent = 14;          // 对应 React.memo 返回的组件

```
> 在 React 调和渲染 fiber 节点的时候，如果发现 fiber tag 是 ClassComponent = 1，则按照类组件逻辑处理，如果是 FunctionComponent = 0 则按照函数组件逻辑处理。当然 React 也提供了一些内置的组件，比如说 Suspense 、Profiler 等。

### 把 render 过程变成可控的
```jsx
class Index extends React.Component{
    status = false /* 状态 */
    renderFoot=()=> <div> i am foot</div>
    /* 控制渲染 */
    controlRender=()=>{
        const reactElement = (
            <div style={{ marginTop:'100px' }} className="container"  >   
                 { /* element 元素类型 */ }
                <div>hello,world</div>  
                { /* fragment 类型 */ }
                <React.Fragment>      
                    <div> 👽👽 </div>
                </React.Fragment>
                { /* text 文本类型 */ }
                my name is alien       
                { /* 数组节点类型 */ }
                { toLearn.map(item=> <div key={item} >let us learn { item } </div> ) } 
                { /* 组件类型 */ }
                <TextComponent/>  
                { /* 三元运算 */  }
                { this.status ? <TextComponent /> :  <div>三元运算</div> }  
                { /* 函数执行 */ } 
                { this.renderFoot() }  
                <button onClick={ ()=> console.log( this.render() ) } >打印render后的内容</button>
            </div>
        )
        console.log(reactElement)
        const { children } = reactElement.props
        /* 第1步 ： 扁平化 children  */
        const flatChildren = React.Children.toArray(children)
        console.log(flatChildren)
        /* 第2步 ： 除去文本节点 */
        const newChildren :any= []
        React.Children.forEach(flatChildren,(item)=>{
            if(React.isValidElement(item)) newChildren.push(item)
        })
        /* 第3步，插入新的节点 */
        const lastChildren = React.createElement(`div`,{ className :'last' } ,`say goodbye`)
        newChildren.push(lastChildren)
        
        /* 第4步：修改容器节点 */
        const newReactElement =  React.cloneElement(reactElement,{} ,...newChildren )
        return newReactElement
    }
    render(){
        return this.controlRender()
    }
}

```
### React.createElement 和 React.cloneElement 区别:

> 一个是用来创建 element 。另一个是用来修改 element，并返回一个新的 React.element 对象。


### Babel 解析 JSX 流程
1. @babel/plugin-syntax-jsx 和 @babel/plugin-transform-react-jsx
> 使用@babel/plugin-syntax-jsx插件，能够让 Babel 有效的解析 JSX 语法；@babel/plugin-transform-react-jsx，可以把 React JSX 转化成 JS 能够识别的 createElement 格式

> 其他看小册

### 如果没有在 constructor 的 super 函数中传递 props，那么接下来 constructor 执行上下文中就获取不到 props ，这是为什么呢？
```jsx
/* 假设我们在 constructor 中这么写 */
constructor(){
    super() // 执行 super 等于执行 Component 函数，此时 props 没有作为第一个参数传给 super() ，在 Component 中就会找不到 props 参数，从而变成 undefined 
    console.log(this.props) // 打印 undefined 为什么?
}
```
剖析： 
1. react底层这样定义类组件
```jsx
function Component(props, context, updater) {
  this.props = props;      //绑定props
  this.context = context;  //绑定context
  this.refs = emptyObject; //绑定ref
  this.updater = updater || ReactNoopUpdateQueue; //上面所属的updater 对象
}
/* 绑定setState 方法 */
Component.prototype.setState = function(partialState, callback) {
  this.updater.enqueueSetState(this, partialState, callback, 'setState');
}
/* 绑定forceupdate 方法 */
Component.prototype.forceUpdate = function(callback) {
  this.updater.enqueueForceUpdate(this, callback, 'forceUpdate');
}

```
