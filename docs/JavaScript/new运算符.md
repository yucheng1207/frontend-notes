# new 操作符

## new 一个函数做了什么事情
1. 创建一个空的简单JavaScript对象（即{}）；
2. 为步骤1新创建的对象添加属性__proto__，将该属性链接至构造函数的原型对象 ；
3. 将步骤1新创建的对象作为this的上下文 ；
4. 如果该函数没有返回对象，则返回this。

## new操作符模拟实现
[JavaScript深入之new的模拟实现](https://github.com/mqyqingfeng/Blog/issues/13)中的示例：
```js
function objectFactory() {
    var obj = new Object(); // 用new Object() 的方式新建了一个对象 obj
    Constructor = [].shift.call(arguments); // 取出第一个参数，就是我们要传入的构造函数。此外因为 shift 会修改原数组，所以 arguments 会被去除第一个参数
    obj.__proto__ = Constructor.prototype; // 将 obj 的原型指向构造函数，这样 obj 就可以访问到构造函数原型中的属性

    var ret = Constructor.apply(obj, arguments); // 使用 apply，改变构造函数 this 的指向到新建的对象，这样 obj 就可以访问到构造函数中的属性

    return typeof ret === 'object' ? ret : obj; // 如果构造函数返回类型为对象，则返回对象；如果返回是基础类型，则返回基础类型
};
```

## 函数返回值
1、一般的调用函数的返回值依赖于return 语句，若没有return语句 将在代码执行完后返回undefined；

2、函数作为构造函数(使用new调用)，如果没有return语句，或者return后是基本数据类型，会将this作为返回值；反之如果return了对象，则以此对象为返回值。


# 参考
- [JavaScript深入之new的模拟实现](https://github.com/mqyqingfeng/Blog/issues/13)
- [new 运算符MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new)
