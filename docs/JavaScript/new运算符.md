# new 操作符

## new 一个函数做了什么事情
1. 创建一个空的简单JavaScript对象（即{}）；
2. 为步骤1新创建的对象添加属性__proto__，将该属性链接至构造函数的原型对象 ；
3. 将步骤1新创建的对象作为this的上下文 ；
4. 如果该函数没有返回对象，则返回this。


# 参考
- [JavaScript深入之new的模拟实现](https://github.com/mqyqingfeng/Blog/issues/13)
- [new 运算符MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new)
