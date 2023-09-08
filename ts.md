# Typescript 是 什么

> 它是 JavaScript 的一个严格超集，并添加了可选的静态类型和使用看起来像基于类的面向对象编程语法操作 Prototype。
> js 是动态类型语言（运行时才能发现错误）；
> 静态类型语言，数据类型的检查发生在编译阶段。

# 为什么要使用 ts

> 代码中加入各种各样的类型定义，看上去很浪费时间；但随着类型定义越来越多，效率也随之提高，减少了一些傻瓜型错误的产生。

- 使程序更容易理解，函数或者方法输入输出的参数类型明确 效率更高
- 动态语言约束：需要手动调试等过程
- 在不同的代码块和定义中跳转
- 代码自动补全
- 丰富的接口提示 更少的错误
- 编译期间就能够发现大部分的错误
- 杜绝一些比较常见错误（变量名称写错，传入参数类型错误等）非常好的包容性
- 完全兼容 js
- 第三方库也可以单独编写类型文件

# ts 中的类型

> 基础类型（原始数据类型）：number、string、undefined、null(undefined、null 是所有类型的子类型)
> any（表示允许赋值为任意类型）
> 联合类型：let numberOrString: number | string = 234
> 数组：类型[]——number[]、string[]...
> 类数组：函数中的 arguments:IArguments,htmlCollection:HTMLCollection(NodeList)
> 元组：限定类型的数组，方便把不同类型的数据放入，let user:[string,number]=['viking',20]
> Interface 接口：对对象的形状（shape）进行描述；对类（class）进行抽象；Duck Typing(鸭子类型)

```
interface Person {
    readonly id: number,
    name:string;
    age:number;
    gender?:'男'
}
// 用Person约束viking的形状
let viking:Person={
    id:123,
    name:'viking',
    age:12
}
```

> 函数

```
function add(x:number,y:number):number{return x+y}
const add = function(x:number,y:number):number{ // 此时add也有了类型
    // 函数体内容
}
const add2:(x:number,y:number)=>number=add // 可正确赋值，说明add2和add是相同的类型
```

> 类型推断（论）：ts 会在没有给类型是推断出一个类型

```
let str = 'str'
str=123 // 语法报错，上面已经将str推断为string类型
```

> 枚举 enums

```
enum Direction {
    Up,
    Down,
    Left,
    Right
}
console.log(Direction.Up)
console.log(Direction[0])
enum Direction {
    Up=10,
    Down, // 11
    Left, // 12
    Right // 13
}
enum Direction {
    Up='UP',
    Down='DOWN', // 11
    Left='LEFT', // 12
    Right='RIGHT' // 13
}
```

> 泛型，定义的时候不知道类型，使用的时候再确定类型

```
// 函数在调用的时候传入的是什么类型，T就是什么类型，T是占位符，取值任意
function echo<T>(arg:T):T{
    return arg
}
const result = echo('str')
// 多个参数的泛型,以元组为例
function swap<T,U>(tuple:[T,U]):[U,T]{
    return [tuple[1],tuple[0]]
}
const result = swap(['string',123])
```

> 约束泛型,给泛型约束一个范围

```
function echoWithArr<T>(arg:T):T{
    console.log(arg.length) // 会报错，不能用length
}
// 改：（不报错，但并不是完美解决方案）
function echoWithArr<T>(arg:T[]):T[]{
    console.log(arg.length)
}
// 使用约束泛型（体现出鸭子类型特性）
interface IWithLength {
    length:number
}
function echoWithLength<T extends IWithLength>(arg:T):T{
    console.log(arg.length)
    return arg
}
const str = echoWithLength('str')
const obj = echoWithLength({length:10,with:10})
const arr2 = echoWithLength([1,2,3])
```

> 类中的泛型使用

```js
class Queue {
    private data=[];
    push(item){
        return this.data.push(item)
    }
    pop(){
        return this.data.shift()
    }
}
const queue = new Queue()
queue.push(1)
queue.push('str')
console.log(queue.pop().toFixed())
console.log(queue.pop().toFixed()) // 弹出string类型数据，使用了number类型才有的方法
//改：(虽然可行，但是当想使用字符串类型队列，又不得不修改Queue这个类)
class Queue {
    private data=[];
    push(item:number){
        return this.data.push(item)
    }
    pop():number{
        return this.data.shift()
    }
}
// 使用泛型的方式
class Queue<T> {
    private data=[];
    push(item:T){
        return this.data.push(item)
    }
    pop():T{
        return this.data.shift()
    }
}
const queue = new Queue<number>()
queue.push(1)
const queue2 = new Queue<string>()
queue2.push('str')
```

> 接口中的泛型使用

```
// 定义接口时不确定接口中的字段都是什么类型，使用时传入的是什么类型就是什么类型
interface KeyPair<T,U> {
    key:T;
    value:U;
}
let kp1:KeyPair<number,string>={key:123,value:"str"}
let kp2:KeyPair<string,number>={key:'str',value:123}

```

> 还可以用泛型定义数组

```
// 定义数值数组的方式一：
let arr:number[]=[1,2,3]
// 泛型的方式
let arrTwo:Array<number>=[1,2,3]
```

# ts 中的类 Class

# 用 interface 约束类的实现，抽象类的属性和方法

```
interface Radio {
    switchRadio():void;
}
interface Battery {
    checkBatteryStatus()
}
// 用接口interface约束类必须实现Radio接口中定义的功能
class Car implements Radio{
    switchRadio(){

    }
}
class Cellphone implements Radio,Battery{
    switchRadio(){

    }
    checkBatteryStatus(){}
}
或：接口继承
interface RadioWithBattery extends Radio {
    checkBatteryStatus()
}
class Cellphone implements RadioWithBattery{
    switchRadio(){

    }
    checkBatteryStatus(){}
}
```

# 用接口定义(描述)一个函数的类型

```
interface IPlus{
    (a:number,b:number):number // 这里不是在描述组件属性不用箭头
}
function plus(a:number,b:number):number{
    return a+b
}
const a:IPlus=plus
// 用泛型修改，使函数类型更加灵活
interface IPlus<T> {
    (a:T,b:T):T
}
function plus(a:number,b:number):number {
    return a+b
}
function connect(a:string,b:string):string{
    return a+b
}
const a:IPlus<number> = plus
const b:IPlus<string> = connect
```

# 类型别名和类型断言

> 类型别名

```
function sum(x:number,y:number):number{
    return x+y
}
const sum2=sum // 此时sum2由于类型推论，已经有了sum的类型
// 当然也可以麻烦的如下显示的定义sum2的类型
const sum2:(x:number,y:number)=>number=sum
//或者使用类型别名来定义
type PlusType = (x:number,y:number)=>number
function sum(x:number,y:number):number{
    return x+y
}
const sum2:PlusType = sum
```

- 类型别名的常用场景是使用联合类型时

```
// 假如函数接收的参数可能是string类型也可能是个函数类型，如果是string返回这个参数，如果是函数类型返回函数执行的结果
type NameResolve = ()=>string
type NameOrResolver = string | NameResolve
function getName(n:NameOrResolver):string{
    if(typeof n==='string'){
        return n
    } else {
        return n()
    }
}

```

> 类型断言：（使用联合类型时，ts 不确定到底用的是那个类型，只能使用此联合类型里所有类型共用的属性和方法，而有时确实要在不确定类型时访问这个类型下的属性与方法）

```js
function getLength(input:string|number):number{
    input.length// 此时input下只有value和toString方法，访问length会报错
}
// 使用类型断言
function getLength(input:string|number):number{
    const str = input as String
    if(str.length) {
        return str.length
    } else {
        const number = input as Number
        return number.toString().length
    }
    // 或者另一种快捷写法
    if((<string>input).length){
        return (<string>input).length
    }else {
        return input.toString().length
    }
}
```


# 常见问题
