在模块化的导入导出语法中用的有 require 、module.exports、exports、 import、export、export default。。。前端常用的也就是import、export、export default。但是这么多的关键字总是很容易混淆，所以梳理了一下这块的知识点。

从模块化说起吧，对模块化前世今生不太了解的戳这里[模块化七日谈](http://huangxuan.me/js-module-7day/#/)

### 模块化相关规范

传统的开发模式存在命名冲突，文件依赖等问题，于是JavaScript社区尝试并提出了`AMD`、`CMD`、`CommonJS`等 **模块化规范**，模块化就是把单独的一个功能封装到一个模块（文件）中，模块之间相互隔离，但是可以通过特定的接口公开内部成员，也可以依赖别的模块。模块化开发的好处是方便代码的重用，从而提升开发效率，并且方便后期的维护。但是，这些社区提出的模块化标准还是存在一定的差异性与局限性、**并不是浏览器与服务器通用的模块化标准**，如 **适用于浏览器端的模块化规范AMD(require.js)、CMD(sea.js)**，**适用于服务端的模块化规范CommonJS**。下面来一步步了解`AMD`、`CMD`、`CommonJS`和大一统的`ES6 Module`吧。

### AMD和require.js

`AMD`规范采用异步方式加载模块，模块的加载不影响它后面语句的运行。所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行。这里介绍用`require.js`实现`AMD`规范的模块化：用`require.config()`指定引用路径等，用`define()`定义模块，用`require()`加载模块。

首先我们需要引入`require.js`文件和一个入口文件`main.js`。`main.js`中配置`require.config()`并引入项目中用到的基础模块。

```js
/** 网页中引入require.js及main.js **/
<script src="js/require.js" data-main="js/main"></script>

/** main.js 入口文件/主模块 **/
// 首先用config()指定各模块路径和引用名
require.config({
  baseUrl: "js/lib",
  paths: {
    "jquery": "jquery.min",  //实际路径为js/lib/jquery.min.js
    "underscore": "underscore.min",
  }
});
// 执行基本操作
require(["jquery","underscore"],function($,_){
  // some code here
});

```

引用模块的时候，我们将模块名放在`[]`中作为`reqiure()`的第一个参数；如果我们定义的模块本身也依赖其他模块，那就需要将它们放在`[]`中作为`define()`的第一个参数。

```js
// 定义math.js模块
define(function () {
    var basicNum = 0;
    var add = function (x, y) {
        return x + y;
    };
    return {
        add: add,
        basicNum :basicNum
    };
});
// 定义一个依赖underscore.js的模块
define(['underscore'],function(_){
  var classify = function(list){
    _.countBy(list,function(num){
      return num > 30 ? 'old' : 'young';
    })
  };
  return {
    classify :classify
  };
})

// 引用模块，将模块放在[]内
require(['jquery', 'math'],function($, math){
  var sum = math.add(10,20);
  $("#sum").html(sum);
});

```

### CMD和sea.js

首先CMD不是CommonJS哦，不要记混了。`require.js`在声明依赖的模块时会在第一时间加载并执行模块内的代码，`CMD`是另一种`js`模块化方案，它与`AMD`很相似，不同点在于：`AMD`推崇依赖前置、提前执行，`CMD`推崇依赖就近、延迟执行。

```js
/** AMD写法 **/
define(["a", "b", "c", "d", "e", "f"], function(a, b, c, d, e, f) { 
     // 等于在最前面声明并初始化了要用到的所有模块
    a.doSomething();
    if (false) {
        // 即便没用到某个模块 b，但 b 还是提前执行了
        b.doSomething()
    } 
});

/** CMD写法 **/
define(function(require, exports, module) {
    var a = require('./a'); //在需要时申明
    a.doSomething();
    if (false) {
        var b = require('./b');
        b.doSomething();
    }
});

/** sea.js **/
// 定义模块 math.js
define(function(require, exports, module) {
    var $ = require('jquery.js');
    var add = function(a,b){
        return a+b;
    }
    exports.add = add;
});
// 加载模块
seajs.use(['math.js'], function(math){
    var sum = math.add(1+2);
});

```

### node与CommonJS

Node.js是commonJS规范的主要实践者，它有四个重要的环境变量为模块化的实现提供支持：`module`、`exports`、`require`、`global`。

CommonJS语法规范中：

- 用`module.exports`定义当前模块对外输出的接口（不推荐直接用`exports`）
- 用`require`加载模块。

那么`exports` 和 `module.exports`之间是什么关系呢？

解惑时刻！

CommonJS语法规范中node在执行一个文件时，会给这个文件内生成一个 `exports`和`module`对象，而`module`又有一个`exports`属性。他们之间的关系如图，都指向同一块内存区域。

![88FA4BF1-9F4D-4D3D-BDD8-FB6C0323D420](/Users/please/Desktop/88FA4BF1-9F4D-4D3D-BDD8-FB6C0323D420.png)

看下如下代码：

```js
//utils.js
let a = 100;
 
console.log(module.exports); //能打印出结果为：{}
console.log(exports); //能打印出结果为：{}
 
exports.a = 200; //这里辛苦劳作帮 module.exports 的内容给改成 {a : 200}
 
exports = '指向其他内存区'; //这里把exports的指向指走
 
//test.js
var a = require('/utils');
console.log(a) // 打印为 {a : 200}
```

从上面可以看出，其实`require`导出的内容是`module.exports`指向的内存块内容，并不是`exports`的。简而言之，他们之间的区别就是 `exports` 只是 `module.exports`的一个引用，辅助后者添加属性用的。这样用内存块的概念去理解，就清楚多了。为了避免迷糊，使用时，尽量都用 `module.exports` 导出，然后用`require`导入。这下搞清楚node中使用`CommonJS`规范时`module.exports`和`exports` 的关系以及怎么使用他们了吧。

再来看看 **ES6 Module** 中的`export`和`export default`以及`import`...

### ES6 Module

ES6语法规范中，在语言层面上定义了ES6模块化规范，是 **浏览器端与服务器端通用的模块化开发规范**。so，在开发中我们就可以统一使用ES6的模块化规范的语法来导入导出模块了；避免`module.exports`、`exports`、`export`、 `export default`、`import`、 `require`语法都有，又搞不清楚了。

ES6模块化规范中定义：

- 每个js文件都是一个独立的模块
- 导入模块成员使用import关键字
- 暴露模块成员使用export关键字

来，一起复习下ES6模块化的基本语法

1. 默认导出与默认导入

   - 默认导出语法：export default 默认导出的成员

   - 默认导入语法：import 接收名称 from '模块标识符'

     ```js
     // m1.js
     // 定义私有成员
     let a = 10;
     let c = 20;
     let d = 30;
     function show() {}
     // 将本模块中的私有成员暴露出去，供其它模块使用， 没有暴露出去的d外界访问不到。
     export default {
       a,
       c,
       show
     }
     ```
     
     ```js
     // 导入模块成员
     import m1 from './m1.js'
     console.log(m1) // {a:10, c:20, show:[Function: show]}
     ```

   ⚠️：每个模块中，只允许使用唯一的一次export default，否则会报错。而如果没有导出任何东西的时候，默认会导出一个空对象。

2. 按需导出与按需导入

   - 按需导出语法： export let s1 = 10
   - 按需导入语法：import { s1 } from '模块标识符'

   ```js
   // m1.js文件
   // 定义私有成员
   let a = 10;
   let c = 20;
   let d = 30;
   function show() {}
   // 将本模块中的私有成员暴露出去，供其它模块使用， 没有暴露出去的d外界访问不到。
   export default {
     a,
     c,
     show
   }
   
   //向外按需导出 s1、s2、say
   export let s1 = '111'
   export let s2 = 'ccc'
   export function say = function () {}
   ```

   ```js
   // 导入模块成员(默认导出&按需导出同时存在)
   import m1, { s1, s2 as ss2, say } from './m1.js' // 用 as 定义别名
   console.log(m1)
   console.log(s1)
   console.log(ss2)
   console.log(say)
   ```

   

3. 有时候，我们只想单纯执行某个模块中的代码，并不需要得到模块中向外暴露的成员，此时，可以直接导入并执行模块代码

   ```js
   // m2.js
   // 执行一个for循环操作
   for (let i=0; i<3; i++) {
   	console.log(i)
   }
   ```

   ```js
   // 直接导入并执行模块代码
   import './m2.js'
   ```


4. 复杂项目模块较多，想要方便的统一导出&导入？

   ```js
   // a.js
   export function add () {
   	// doSomething
   }
   export function append () {
   	// doSomething
   }
   // b.js
   export function set () {
   	// doSomething
   }
   // c.js
   export function update () {
   	// doSomething
   }
   // index.js
   export * from './a';
   export * from './b';
   export * from './c';
   
   // 最后引入时只需要:
   import { add, append, set, update } from './index'
   // 而不用繁琐滴酱紫：
   import { add, append } from './a'
   import { set } from './b'
   import { update } from './c'
   ```

   

   最后，再多说四句，总结一下：
   - `export`与`export default`均可用于导出常量、函数、文件、模块等；

   - 在一个文件或模块中，`export`、`import`可以有多个，`export default`仅有一个；

   - 通过`export`方式导出，在导入时要加`{}`，`export default`则不需要；

   - export能直接导出变量表达式，`export default`不行。

   可以通过以下代码去验证一下：

```js
// es6ModuleTest.js
'use strict'
//导出变量
export const a = '100';  
 
 //导出方法
export const dogSay = function(){ 
    console.log('wang wang');
}
 
 //导出方法第二种
function catSay(){
   console.log('miao miao'); 
}
export { catSay };
 
//export default导出
const m = 100;
export default m; 
//export defult const m = 100;// 这里不能写这种格式.
```

```js
// index.js
'use strict'
 
import { dogSay, catSay } from './es6ModuleTest'; //导出了 export 方法 
import m from './es6ModuleTest';  //导出了 export default 
 
import * as testModule from './es6ModuleTest';//as 集合成对象导出

dogSay();
catSay();

console.log(m); // 100

testModule.dogSay();
console.log(testModule.m); // undefined , 因为  as 导出是 把 零散的 export 聚集在一起作为一个对象，而export default 是导出为 default属性。

console.log(testModule.default); // 100
```



### Node.js中通过babel使用ES6模块化规范

node支持CommonJs的模块化规范；但是对ES6的模块化规范支持不是很好，需要通过结合babel来实现在node中使用ES6模块化规范。

babel的使用和配置基本如下，实际运用时根据项目和环境有些不同可以在查阅文档...

1. 首先是安装一些必要的依赖，如：`npm install --save-dev @babel/core @babel/cli @babel/preset-env @babel/node`

2. `npm install --save @babel/polyfill`

3. 项目根目录创建babel配置文件`.babelrc`

   ```json
   {
       "presets": ["@babel/env"]
   }
   ```

   ```js
   // nodetest.js
   export const sayHello = function(){    
       console.log('hello')
   }
   ```

   ```js
   // index.js
   import { sayHello } from './nodetest'
   console.log('sayHello', sayHello) // sayHello [Function: sayHello]
   ```

   

4. 通过`npx babel-node index.js`执行代码

### ES6 模块与 CommonJS 模块的差异

1. CommonJS 模块输出的是一个值的浅拷贝，ES6 模块输出的是值的引用。

   - CommonJS 模块输出的是值的浅拷贝，也就是说，一旦输出一个值，模块内部的变化就影响不到这个值。（来，再复习一下深拷贝和浅拷贝。深拷贝是将一个对象从内存中完整的拷贝一份出来,从堆内存中 **开辟一个新的区域存放新对象**,且 **修改新对象不会影响原对象**。浅拷贝是创建一个新对象，这个对象有着原始对象属性值的一份精确拷贝。如果属性是基本类型，拷贝的就是基本类型的值，如果属性是引用类型，拷贝的就是内存地址 ，所以**如果其中一个对象改变了这个地址，就会影响到另一个对象**。）

   - ES6 模块的运行机制与 CommonJS 不一样。JS 引擎对脚本静态分析的时候，遇到模块加载命令`import`，就会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用，到被加载的那个模块里面去取值。原始值变了，`import`加载的值也会跟着变。因此，ES6 模块是动态引用，并且不会缓存值，模块里面的变量绑定其所在的模块。

     上代码领会一下：

     ```js
     // nodeTest.js
     // 先定义属性a和obj 一个是基本类型 一个是引用类型
     let a = 100;
     let obj = {b: 'bbb'}
     
     // exports添加属性
     exports.a = a; 
     exports.obj = obj;
     
     // 1秒后修改a和obj
     setTimeout(()=>{
         a = 400;
         obj.b='changed'
     }, 1000)
     
     // index.js
     var result = require('./nodeTest');
     console.log(result) 
     setTimeout(()=>{
         console.log('jest', result)
       },7000)
     // 打印结果
     // 立即打印： { a: 100, obj: { b: 'bbb' } }
     // 7秒后打印： jest { a: 100, obj: { b: 'changed' } }
     // 可以看到7秒后打印出的result对象中引用类型obj中的属性值被改掉了，简单类型a的值没有变
     ```

     OK，再来看ES6 Module代码：

     ```js
     // es6Test.js
     export let a = 4
     export let obj = {b: 'bbb'}
     
     setTimeout(()=>{
       a=5;
       obj.b='changed';
     },1000)
     
     // index.js
     import { a, obj } from './es6Test.js'
     console.log(a, obj)
     setTimeout(() => {
       console.log(a, obj)
     }, 7000);
     // 打印结果
     // 立即打印：4 {b: 'bbb'}
     // 7秒后打印：5 {b: 'changed'}
     ```

2. CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。
   - 运行时加载: CommonJS 模块就是对象；即在输入时是先加载整个模块，生成一个对象，然后再从这个对象上面读取方法，这种加载称为“运行时加载”。
   - 编译时加载: ES6 模块不是对象，而是通过 `export` 命令显式指定输出的代码，`import`时采用静态命令的形式。即在`import`时可以指定加载某个输出值，而不是加载整个模块，这种加载称为“编译时加载”。

CommonJS 加载的是一个对象（即`module.exports`），该对象只有在脚本运行完才会生成。而 ES6 模块不是对象，它的对外接口只是一种静态定义，在代码静态解析阶段就会生成。

看完这些你应该可以搞清楚`export`、`export default`、import、`module.exports`、`exports`、`require`的区别了吧。`module.exports`、`exports`、`require`是CommonJS规范中的定义；`export`、`export default`、`import`是ES6 Module规范中的。ES6 Module实现了模块化的统一，你可以统一使用ES6 Module语法规范，只需要配置babel就行了。

 文章中的总结，有问题欢迎指正。
