> React v16.8中提供了10个hooks, 日常开发常用的有 `useState`、`useEffect`、`useRef`、`useImperativeHandle`, 其它还有 `useContext`、`useReducer`、`useMemo`、`useCallback`、`useLayoutEffect`等
> 下面一一介绍其定义与使用场景

## useState

> 定义变量，使其具备类组件的 state, 状态更改，视图更新

```js
const [state, setState] = useState(initData)
// initData：设置状态的默认初始值，如果是函数，则函数的返回值作为初始值
```
> 使用时注意事项

1. 更新数据时，无法立刻获取到最新的数据
```js
// 示例
const [name, setName] = useState('test');
function set() {
  console.log(name) // test
  setName('test1')
  console.log(name) // test
}

// 解决方案 创建一个新的变量保存最新的数据
function set() {
  console.log(name) // test
  const newName = "test1"
  setName(newName)
  console.log(name) // test1
}

// 或者:
// 1. 在useEffect中监听获取
// 2. 使用ahooks中的useGetState

```
2. 当存入引用类型的值时， useState存入的值是该值的引用
> 如果一处更改导致了另一处也更改,或者属性值的改变没有触发useEffect, 需要考虑深拷贝后赋值

```js
const text = {name:'test'}
const [state, setState] = useState(text)

function set() {
  setState((oldVal) => {
    oldVal.age = 18
    // 返回一个新的对象,useEffectc才能检测得到
    return {...oldVal} // 使用扩展运算符，如果只有一层,可以看成深拷贝 如果有多层,也就是说对象/数组里面的元素是引用类型,就是浅拷贝；Object.assign(target, obj1, obj2, ...)亦是如此
  })
}
useEffect(() => {
  console.log(state)  // {name: "test", age: 18}
},[state])

```

## useEffect

> 副作用，弥补了函数式组件没有生命周期的缺陷，常用于组件在渲染后执行某些操作
> 默认情况下，effect在第一次渲染和每次更新后都会执行
> 通过控制第二个参数的传入方式来控制useEffect的调用的时机
1. 第二个参数不传，每次的更新都会触发useEffect
2. 第二个参数为一个包含一个或多个参数的数组，数组里面的任意一个发生改变都会触发useEffect函数的重新执行
3. 第二个参数是一个空数组, useEffect只执行一次
> 每次重新渲染都会生成新的effect，替换掉之前的，确保effect中获取的值是最新的
> useEffect调用不会阻塞浏览器dom布局更新（异步），个别情况下（获取布局信息的情况下，使用单独提供的useLayoutEffect，它会在所有的dom变更之后同步调用）
> useEffect清理功能：useEffect里返回的函数会在组件卸载之前执行
```js
function MyComponent() { 
  useEffect(() => { 
    // 这个副作用会在每次渲染之后运行
    return () => { 
      // 这个副作用会在组件卸载之前运行，可以在此清除订阅的外部数据源、绑定的事件、定时器等
    } 
  }) 
}
```

## useRef
> useRef 用于获取当前元素的所有属性
> 有一个高级用法：缓存数据
> 基本用法：
```js
import { useRef } from "react";

const List = () => {
  const ref = useRef(null);

  function onClick() {
    console.log(ref?.current.offsetWidth)
  }

  return (
    <>
      <div ref={ref}>绑定ref的元素</div>
      <button onClick={onClick}>点击</button>
    </>
  );
};

export default List;

```

## useImperativeHandle

> 可以通过`ref`或`forwardRef`, 将自身的属性和方法暴露给父组件。可以在父组件中方便的获取子组件的属性，调用子组件的方法。

> 基本使用
```js
// useImperativeHandle(ref, createHandle, deps) 
// ref: 父组件传过来的ref
// createHandle处理函数 返回要暴露给父组件的属性和方法
// 依赖项，依赖项如果更改，会形成新的ref对象 同useEffect的依赖项
import { useState, useRef, useImperativeHandle } from "react";

const Child = ({childRef, ...props}) => {

  const [count, setCount] = useState(0)
  const [ name, setName ] = useState('son')

  useImperativeHandle(childRef, () => ({
    name,
    add
  }))

  const add = () => {
    setCount((v) => v + 1)
  }

  return <div>
    <p>点击次数：{count}</p>
    <Button onClick={() => add()}> 子组件的按钮，点击+1</Button>
  </div>
}
export default Child

function Parent() {
  const ref = useRef(null)
  return (
    <>
      <Button
        type="primary"
        onClick={() =>  ref.current.add()}
      >
        父组件点击
      </Button>
      <Child childRef={ref} /> 
      // 注意哦😯 这里传的属性是childRef 而不是ref 如果传ref会报错 需要使用forwardRef转发
      // Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?
    </>
  );
};

export default Parent;
```
> 查阅传ref属性报错的原因了解到，在函数式组件中不允许 ref 作为参数，除了 ref，key 也不允许作为参数，原因是在 React 内部中，ref和key会形成单独的key名
> 需要借助forwardRef，将子组件中的props（其余参数）和 ref 拆分出来，ref 会作为第二个参数进行传递 如：
```js
import { useState, useRef, useImperativeHandle, forwardRef } from "react";

const Child = (props, ref) => {

  const [count, setCount] = useState(0)
  const [ name, setName ] = useState('son')

  useImperativeHandle(childRef, () => ({
    name,
    add
  }))

  const add = () => {
    setCount((v) => v + 1)
  }

  return <div>
    <p>点击次数：{count}</p>
    <Button onClick={() => add()}> 子组件的按钮，点击+1</Button>
  </div>
}
export default forwardRef(Child)

function Parent() {
  const ref = useRef(null)
  return (
    <>
      <Button
        type="primary"
        onClick={() =>  ref.current.add()}
      >
        父组件点击
      </Button>
      <Child ref={ref} /> 
      // 注意哦😯 这里传的属性是childRef 而不是ref 如果传ref会报错
      // Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?
    </>
  );
};

export default Parent;
```
> 上面介绍完了几个项目中常用到的Hooks,下面介绍剩余一些不常用到的

## useContext

> 上下文，与`createContext`配合使用，用户跨层级祖孙、父子组件间的通信
> 基本用法：
```js
// parent.jsx
import { createContext } from 'react'

export const CountContext = createContext({ value: 0, name: 'zs' });

function Parent() {

  return (
    <CountContext.provider value={{ value: 1, age: 18 }}>
      <Child></Child>
    </CountContext.provider>
  )
}

export default Parent

// child.jsx
import { useContext } from 'react'
import { CountContext } from './parent'

function Child() {
  const count = useContext(CountContext);

  return (
    <p>父组件传递的值为：{count.value}</p> // count: {value: 1, age: 18} 当同时设置了默认值和CountContext.provider的value属性时间，以CountContext.provider提供的为准
  )
}

export default Child

```

## useLayoutEffect
> 与useEffect基本一致，区别是useLayoutEffect是同步的
1. useLayoutEffect 是在 DOM 更新之后，浏览器绘制之前执行， 在这里，获取DOM信息、修改DOM，浏览器只会绘制一次
2. useEffect 在渲染时是异步执行,并且要等到浏览器将所有变化渲染到屏幕后才会被执行
3. useLayoutEffect 的执行顺序在 useEffect 之前
4. useLayoutEffect 的callback会阻塞浏览器绘制
5. 一般都使用useEffect, 但在开发时如果连续变更dom属性或变量，页面出现闪烁的情况时，可以考虑使用useLayoutEffect解决

## useMemo
> 每一次的状态更新，都会让组件重新绘制，而重新绘制必然会带来不必要的性能开销，为了防止没有意义的性能开销，React Hooks 提供了 useMemo 函数。
> 它之所以能带来提升，是因为在依赖不变的情况下，会返回相同的引用，避免子组件进行无意义的重复渲染。
```js
// fn：函数，函数的返回值会作为缓存值
// deps：[] 依赖项
const cacheData = useMemo(fn, deps)
```

## useCallback
> useCallback：与 useMemo 极其类似，甚至可以说一模一样，唯一不同的点在于，useMemo 返回的是值，而 useCallback 返回的是函数。

## useReducer
> 功能类似于 redux，与 redux 最大的不同点在于它是单个组件的状态管理，组件通讯还是要通过 props。简单地说，useReducer 相当于是 useState 的升级版，用来处理复杂的 state 变化。
```js
// reducer：函数，可以理解为 redux 中的 reducer，最终返回的值就是新的数据源 state
// initialArg：初始默认值
// init：惰性初始化，可选值
const [state, dispatch] = useReducer(
  (state, action) => {}, 
  initialArg,
  init
)
const [count, dispatch] = useReducer((state: number, action: any) => {
  switch (action?.type) {
    case "add":
      return state + action?.payload;
    case "sub":
      return state - action?.payload;
    default:
      return state;
  }
}, 0)
```