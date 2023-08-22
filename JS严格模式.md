# JS严格模式详解

## 严格模式的使用

严格模式通过在脚本或函数的头部添加 **use strict**; 表达式来声明。

## 严格模式的优势

JS除了正常的运行模式（混杂模式）外，ES5还添加了第二种运行模式：严格模式（strict mode）。顾名思义，这种模式使得javascript在更严格的语法条件下运行。其目的是为了消除javascript语法的一些不合理、不严谨之处，减少一些怪异行为，消除代码运行的一些不安全之处，保证代码运行的安全。提高编译器效率，增加运行速度，为未来新版本的javascript做好铺垫。

## 使用严格模式带来的限制

1. 严格模式下变量必须先声明再使用，使用未声明的变量会报错；

2. 在函数内部声明的严格模式，只在函数内部生效；

   ```js
   a = 3;       // 不报错
   myFunction();
   
   function myFunction() {
      "use strict";
       b = 3;   // 报错 (b 未定义)
   }
   ```

   

3. 严格模式下不允许删除变量（包括对象、函数），只有configurable设置为true的对象属性，才能被删除；

   ```js
   "use strict";
   var m = 3;
   delete m;  // 报错
   function f(a, b) {};
   delete f;   // 报错
   
   var obj = Object.create(null, {'x': {
     value: 1,
     configurable: true
   }});
   
   delete obj.x; // 删除成功
   ```

4. 对象不能有重名的属性；

   ```js
   "use strict";
   var o = {
     a: 1,
     a: 2
   }; // 报错
   ```

5. 函数不能有重名的参数；

   ```js
   "use strict";
   function f(a, a) { // 语法错误
     return ;
   }
   ```

   

6. 不允许使用八进制；

7. 不允许使用转义字符；

   ```js
   "use strict";
   var x = \010;
   ```

   

8. 不允许对只读属性赋值；

   ```js
   "use strict";
   var o = {};
   Object.defineProperty(o, "a", { value: 1, writable: false });
   o.a = 2; // 报错
   ```

   

9. 不允许对一个使用getter方法读取的属性进行赋值

   ```js
   "use strict";
   var o = {
     get v() { return 1; }
   };
   o.v = 2; // 报错
   ```

   

10. 禁止删除一个不允许删除的属性；

    ```js
    "use strict";
    delete Object.prototype; // 报错
    ```

    

11. 不允许使用with语句；

12. (创设第三种作用域—eval作用域；)   在作用域 eval() 创建的变量不能被调用；

    ```js
    "use strict";
    eval ("var a = 2");
    alert (a); // 报错
    ```

13. 禁止this关键字指向全局对象；

    ```js
    function f(){
    	return !this;
    }
    // 返回false，因为"this"指向全局对象，"!this"就是false
    function f(){
    	"use strict";
    	return !this;
    }
    // 返回true，因为严格模式下，this的值为undefined，所以"!this"为true。
    
    // 因此，使用构造函数时，如果忘了加new，this不再指向全局对象，而是报错。
    function f() {
      "use strict";
      this.a = 1;
    };
    f(); // 报错，this未定义
    ```

    

14. 严格模式下，不允许对arguments赋值，arguments不再追踪参数的变化，禁止使用arguments.callee；

15. 严格模式下，也不允许对eval赋值；

16. 严格模式新增了一些保留字：implements, interface, let, package, private, protected, public, static, yield。

    使用这些词作为变量名将会报错。

