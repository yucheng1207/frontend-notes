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

# 参考
- [JavaScript深入之从ECMAScript规范解读this](https://github.com/mqyqingfeng/Blog/issues/7#)
- [JavaScript 的 this 原理 - 阮一峰](https://www.ruanyifeng.com/blog/2018/06/javascript-this.html)