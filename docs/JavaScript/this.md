# This

## 从底层原理分析
看[JavaScript深入之从ECMAScript规范解读this](https://github.com/mqyqingfeng/Blog/issues/7#)， 下面梳理下这篇文章的一些知识点：

#### ECMAScript 5.1 规范
- 英文版：http://es5.github.io/#x15.1
- 中文版：http://yanhaijing.com/es5/#115

ECMAScript 的类型分为语言类型和规范类型。
- 语言类型：语言类型是开发者直接使用 ECMAScript 可以操作的。其实就是我们常说的Undefined, Null, Boolean, String, Number, 和 Object。
- 规范类型：规范类型相当于 meta-values，是用来用算法描述 ECMAScript 语言结构和 ECMAScript 语言类型的。规范类型包括：Reference, List, Completion, Property Descriptor, Property Identifier, Lexical Environment, 和 Environment Record。

没懂？没关系，我们只要知道在 ECMAScript 规范中还有一种只存在于规范中的类型，它们的作用是用来描述语言底层行为逻辑。

今天我们要讲的重点是便是其中的 Reference 类型。它与 this 的指向有着密切的关联。

#### Reference

要理解`this`，先要理解`Reference`，它与`this`的指向有密切的关系。`Reference`由三部分组成
- base value
    - 属性所在的对象或者就是 EnvironmentRecord，它的值只可能是 undefined, an Object, a Boolean, a String, a Number, or an environment record 其中的一种
- referenced name
    - 属性的名称
- strict reference

而且规范中还提供了获取`Reference`组成部分的方法，比如 `GetBase` 和 `IsPropertyReference。`

- GetBase： 返回 reference 的 base value。
- IsPropertyReference：如果 base value 是一个对象，就返回true。

规范中还有一个用于从`Reference`类型获取对应值的方法： `GetValue`。

- GetValue：`GetValue`返回对象属性真正的值，但是要注意：**调用 GetValue，返回的将是具体的值，而不再是一个 Reference**

#### 如何确定this值
1. 计算 `MemberExpression` 的结果赋值给 `ref`
    MemberExpression就是`()`左边的部分
    ```
    - PrimaryExpression // 原始表达式 可以参见《JavaScript权威指南第四章》
    - FunctionExpression // 函数定义表达式
    - MemberExpression [ Expression ] // 属性访问表达式
    - MemberExpression . IdentifierName // 属性访问表达式
    - new MemberExpression Arguments // 对象创建表达式
    ```
2. 判断 `ref` 是不是一个 `Reference` 类型
    - 判读方法：
        ```
        1. 如果 ref 是 Reference，并且 IsPropertyReference(ref) 是 true, 那么 this 的值为 GetBase(ref)
        2. 如果 ref 是 Reference，并且 base value 值是 Environment Record, 那么this的值为 ImplicitThisValue(ref)
        3. 如果 ref 不是 Reference，那么 this 的值为 undefined
        ```
    - 例子：
        ```js
        var value = 1;

        var foo = {
        value: 2,
        bar: function () {
                return this.value;
            }
        }

        // 示例1
        console.log(foo.bar());
        // MemberExpression 为 foo.bar，foo.bar是一个Reference，将 foo.bar 赋值给 ref
        // IsPropertyReference(ref) 为 true，那么 this 的值为 GetBase(ref) ，而 GetBase 返回的是 reference 的 base value。
        // 所以 GetBase(ref) => GetBase(foo.bar) => foo.bar的base value => foo
        // 所以 this 为 foo ！！！
        // 所以结果为 2


        //示例2
        console.log((foo.bar)());
        // () 并没有对 MemberExpression 进行计算，所以其实跟示例 1 的结果是一样的
        // 结果为 2

        //示例3
        console.log((foo.bar = foo.bar)());
        // 有赋值操作符，使用了 GetValue，所以返回的值不是 Reference 类型
        // this 为 undefined，非严格模式下，this 的值为 undefined 的时候，其值会被隐式转换为全局对象。
        // 结果为 1

        //示例4
        console.log((false || foo.bar)());
        // 同上，使用了 || 操作符， this 为 undefined
        // 结果为 1

        //示例5
        console.log((foo.bar, foo.bar)());
        // 同上，使用了 , 操作符， this 为 undefined
        // 结果为 1
        ```
        ```js
        function foo() {
            console.log(this)
        }

        foo();

        // MemberExpression 是 foo，foo是一个Reference，将 foo 赋值给 ref
        // base value 是 EnvironmentRecord，则IsPropertyReference(ref)为false
        // 根据上面的判断方法2，this的值为 ImplicitThisValue(ref)
        // ImplicitThisValue 始终返回 undefined，所以 this 为 undefined
        ```

## 从调用场景分析
- 作为对象调用时，指向该对象 obj.b(); // 指向obj
- 作为函数调用, var b = obj.b; b(); // 指向全局window
- 作为构造函数调用 var b = new Fun(); // this指向当前实例对象
- 作为call与apply调用 obj.b.apply(object, []); // this指向当前的object

[JavaScript 的 this 原理 - 阮一峰](https://www.ruanyifeng.com/blog/2018/06/javascript-this.html)有内存数据结构和调用环境的讲解，比较简单易懂


## call 和 apply
`call`和`apply`都可以改变函数的执行上下文，调用 `call` 和 `apply` 的对象，必须是一个函数 Function

#### call
**特征：**
- 调用 call 的对象，必须是个函数 Function。
- call 的第一个参数，是一个对象。 Function 的调用者，将会指向这个对象。如果不传，则默认为全局对象 window。
- 第二个参数开始，可以接收任意个参数。每个参数会映射到相应位置的 Function 的参数上。但是如果将所有的参数作为数组传入，它们会作为一个整体映射到 Function 对应的第一个参数上，之后参数都为空。
```js
Function.call(obj,[param1[,param2[,…[,paramN]]]])
```

**使用场景**
- 对象的继承，subClass 通过 call 方法，继承了 superClass 的 print 方法和 a 变量。此外，subClass 还可以扩展自己的其他方法。
    ```js
    function superClass () {
        this.a = 1;
        this.print = function () {
            console.log(this.a);
        }
    }

    function subClass () {
        superClass.call(this);
        this.print();
    }

    subClass();
    ```
- 借用方法。如下**类数组借用数组的方法**，arguments 是类数组，并不具备数组的 forEach() 方法，那么我们可以通过 call() 调用数组的该方法，同时将方法里面的 this 绑定到 arguments 上
    ```js
    // 上面代码等同于
    var arr = [].slice.call(arguments)；

    // ES6:
    let arr = Array.from(arguments);
    let arr = [...arguments];

    // example 1
    Array.prototype.forEach.call(arguments,function(item){
        console.log(item);
    });

    // example 2
    let domNodes = Array.prototype.slice.call(document.getElementsByTagName("*")); // 这样，domNodes 就可以应用 Array 下的所有方法了。
    ```
- 判断数据类型
    ```js
    var a = "abc";
    var b = [1,2,3];
    Object.prototype.toString.call(a) == "[object String]" //true
    Object.prototype.toString.call(b) == "[object Array]"  //true
    ```
- 模拟浅拷贝
    模拟浅拷贝的过程中，需要剔除原型链上的属性，考虑到源对象可能基于 Object.create() 创建，而这样的对象是没有 hasOwnProperty() 方法的，因此我们不在源对象身上直接调用该方法，而是通过 Object.prototype.hasOwnProperty.call() 的方式去调用，因为 Object 一定是有这个方法的，我们可以借用一下。
    ```js
    if (Object.prototype.hasOwnProperty.call(nextSource, nextKey)) {
        to[nextKey] = nextSource[nextKey];
    }
    ```

**模拟实现**
```js
Function.prototype.call = function (context) {
  context = context || window;
  context.fn = this;

  let args = [...arguments].slice(1);
  let result = context.fn(...args);

  delete context.fn
  return result;
}
```

#### apply
**特征：**
- 它的调用者必须是函数 Function，并且只接收两个参数，第一个参数的规则与 call 一致。
- 第二个参数，必须是数组或者类数组，它们会被转换成类数组，传入 Function 中，并且会被映射到 Function 对应的参数上。这也是**call 和 apply 之间，很重要的一个区别**。
```
Function.apply(obj[,argArray])
```

**使用场景**
- 对象的继承，同call
- 求数组的最值。
    ```
    let max = Math.max.apply(null, array);
    let min = Math.min.apply(null, array);
    ```
- 实现两个数组合并
    ```
    let arr1 = [1, 2, 3];
    let arr2 = [4, 5, 6];

    Array.prototype.push.apply(arr1, arr2);
    console.log(arr1); // [1, 2, 3, 4, 5, 6]
    ```

**模拟实现**
```js
Function.prototype.apply = function (context, arr) {
    context = context || window;
    context.fn = this;

    let result;
    if (!arr) {
        result = context.fn();
    } else {
        result = context.fn(...arr);
    }

    delete context.fn
    return result;
```

## bind
bind() 方法会创建一个新函数。当这个新函数被调用时，bind() 的第一个参数将作为它运行时的 this，之后的一序列参数将会在传递的实参前传入作为它的参数。(来自于 MDN )

**bind vs (call & apply)**
`call()` 和 `apply()` 返回函数应该返回的值，`bind()` 返回一个经过硬绑定的新函数。
`call()` 和 `apply()` 一经调用则立即执行函数，而 `bind()` 则只是完成了函数的 this 绑定

**模拟实现**
```js
Function.prototype.bind2 = function (context) {
    if (typeof this !== "function") {
      throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
    }

    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);

    var fNOP = function () {};

    var fBound = function () {
        var bindArgs = Array.prototype.slice.call(arguments);
        return self.apply(this instanceof fNOP ? this : context, args.concat(bindArgs));
    }

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();
    return fBound;
}
```


## 箭头函数
> [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions#syntax)

**语法**
- 返回对象字面量时需要用圆括号抱起来
    ```
    //加括号的函数体返回对象字面量表达式：
    params => ({foo: bar})
    ```
- 箭头函数在参数和箭头之间不能换行。
    ```
    var func = ()
           => 1;
    // SyntaxError: expected expression, got '=>'
    ```
- 虽然箭头函数中的箭头不是运算符，但箭头函数具有与常规函数不同的特殊运算符优先级解析规则。
    ```
    let callback;

    callback = callback || function() {}; // ok

    callback = callback || () => {};
    // SyntaxError: invalid arrow-function arguments

    callback = callback || (() => {});    // ok
    ```
- 函数体内{}不使用var定义的变量是全局变量，函数体内{} 用var定义的变量是局部变量
    ```
    var test = () => { a = 1 }
    test() // 运行test会创建全局变量a
    console.log(a) // 1

    var test = () => { var a = 1 }
    test() // 运行test会创建局部变量a
    console.log(a);    // ReferenceError: now is not defined
    ```

**高级语法**
```
//支持剩余参数和默认参数
(param1, param2, ...rest) => { statements }
(param1 = defaultValue1, param2, …, paramN = defaultValueN) => {
statements }

//同样支持参数列表解构
let f = ([a, b] = [1, 2], {x: c} = {x: a + b}) => a + b + c;
f();  // 6

// 使用三元运算符
var simple = a => a > 15 ? 15 : a;
```
**箭头函数与普通函数的区别**
- 语法更加简洁、清晰
- 箭头函数没有自己的`this`，`arguments`，`super`或`new.target`。
    - 箭头函数不会创建自己的this，它只会从自己的作用域链的上一层继承this
    - 箭头函数没有自己的arguments，在箭头函数中访问arguments实际上获得的是外层局部（函数）执行环境中的值。
- .call()/.apply()/.bind()无法改变箭头函数中this的指向，他们的第一个参数会被忽略。
- 箭头函数不能作为构造函数使用，和 new一起用会抛出错误。
- 箭头函数没有原型prototype
- 箭头函数不能用作Generator函数，不能使用yeild关键字

- 箭头函数表达式的语法比函数表达式更简洁，并且**没有自己的this，arguments，super或new.target**。箭头函数表达式更适用于那些本来需要匿名函数的地方，并且它不能用作构造函数，和 new一起用会抛出错误。
- 箭头函数没有prototype属性。
- 箭头函数不会创建自己的this,它只会从自己的作用域链的上一层继承this
- 由于 箭头函数没有自己的this指针，通过 call() 或 apply() 方法调用一个函数时，只能传递参数（不能绑定this---译者注），他们的第一个参数会被忽略。（这种现象对于bind方法同样成立---译者注）



# 参考
- [JavaScript深入之从ECMAScript规范解读this](https://github.com/mqyqingfeng/Blog/issues/7#)
- [JavaScript深入之bind的模拟实现](https://github.com/mqyqingfeng/Blog/issues/12)
- [JavaScript 的 this 原理 - 阮一峰](https://www.ruanyifeng.com/blog/2018/06/javascript-this.html)
- [深度解析 call 和 apply 原理、使用场景及实现](https://github.com/frontend9/fe9-library/issues/206)
- [ES6 - 箭头函数、箭头函数与普通函数的区别](https://juejin.cn/post/6844903805960585224)