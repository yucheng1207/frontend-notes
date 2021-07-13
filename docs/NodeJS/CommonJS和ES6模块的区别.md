# [CommonJS和ES6模块的区别](https://juejin.cn/post/6844904067651600391)
- 因为CommonJS的require语法是同步的，所以就导致了CommonJS模块规范只适合用在服务端，而ES6模块无论是在浏览器端还是服务端都是可以使用的，但是在服务端中，还需要遵循一些特殊的规则才能使用 ；
- CommonJS 模块输出的是一个值的拷贝，而ES6 模块输出的是值的引用；
- CommonJS 模块是运行时加载，而ES6 模块是编译时输出接口，使得对JS的模块进行静态分析成为了可能；
- 因为两个模块加载机制的不同，所以在对待循环加载的时候，它们会有不同的表现。CommonJS遇到循环依赖的时候，只会输出已经执行的部分，后续的输出或者变化，是不会影响已经输出的变量。而ES6模块相反，使用import加载一个变量，变量不会被缓存，真正取值的时候就能取到最终的值；
- 关于模块顶层的this指向问题，在CommonJS顶层，this指向当前模块；而在ES6模块中，this指向undefined；
- 关于两个模块互相引用的问题，在ES6模块当中，是支持加载CommonJS模块的。但是反过来，CommonJS并不能requireES6模块，在NodeJS中，两种模块方案是分开处理的。

# 使用
## ES6加载（引入）一个模块
```
// 加载另一个es6模块
import example from 'es6-package'; // 引入 export default
import { method } from 'es6-package'; // 引入 export 方法
import * as example from 'es6-package'; // as 集合成对象导出
const example = require('es6-package'); // 等同于import * as

// 加载另一个CommonJs模块时只能整体加载
const example = require('commonjs-package');
import example from 'commonjs-package';
const { method } = example;
import { method } from 'commonjs-package';  // 报错, 在ES6模块中可以直接加载CommonJS模块，但是只能整体加载，不能加载单一的输出项
```
## ES6导出一个模块
ES6中可以使用export和export default进行导出，他们之间有以下区别
- export与export default均可用于导出常量、函数、文件、模块等
- 在一个文件或模块中，export、import可以有多个，export default仅有一个
- 通过export方式导出，在导入时要加{ }，export default则不需要
- export能直接导出变量表达式，export default不行。
```
// 导出变量
export const a = '100';

// 导出方法
export const dogSay = function(){
    console.log('wang wang');
}

// 导出方法第二种
function catSay(){
   console.log('miao miao');
}
export { catSay };

// export default导出
const m = 100;
export default m;
// export defult const m = 100; // 这里不能写这种格式。
```

## CommonJS加载（引入）一个模块
```
const example = require('commonjs-package');
```
// CommonJS 的require()命令不能加载 ES6 模块，会报错，只能使用import()这个方法加载，具体看这篇文章：[Node.js 如何处理 ES6 模块](http://www.ruanyifeng.com/blog/2020/08/how-nodejs-use-es6-module.html)
```
(async () => {
  await import('./my-app.mjs');
})();
```

## CommonJS导出一个模块
```
const a = function () {
	console.log('aaa')
}

const b = function () {
	console.log('bbb')
}

exports.a = a // 这种写法等同于module.exports.a = a
module.exports.b = b


或者

module.exports = {
    a,
    b,
}

```
### exports 和 module.exports区别
在一个node执行一个文件时，会给这个文件内生成一个 exports和module对象，而module又有一个exports属性。他们之间的关系如下，都指向一块{}内存区域。
```
exports = module.exports = {};

exports ===>  {}  <=== module.exports
```
使用时只需要记住一点**真正被require的内容永远都是module.exports**，exports只是用于辅助module.exports操作内存中的数据。看下以下几种情况可以更好的理解这句话
```
// 对exports直接赋值不会影响到module.exports，所以最终输出还是{}
exports = {
    ...
}

// module.exports被重新赋值了，所以最终只有b会输出，“exports.a = a”这一行就没有起作用了
exports.a = a
module.exports = {
    b,
}

// 最终a和b都被输出了
exports.a = a // 这种写法等同于module.exports.a = a
module.exports.b = b
```

## 使用范围
> require: node 和 es6 都支持的引入
> export / import : 只有es6 支持的导出引入
> module.exports / exports: 只有 node 支持的导出


# [Node.js 如何处理 ES6 模块](http://www.ruanyifeng.com/blog/2020/08/how-nodejs-use-es6-module.html)
.mjs文件总是以 ES6 模块加载，.cjs文件总是以 CommonJS 模块加载，.js文件的加载取决于package.json里面type字段的设置。
注意，ES6 模块与 CommonJS 模块尽量不要混用。require命令不能加载.mjs文件，会报错，只有import命令才可以加载.mjs文件。反过来，.mjs文件里面也不能使用require命令，必须使用import。

# 参考
- [exports、module.exports和export、export default到底是咋回事](https://segmentfault.com/a/1190000010426778)
- [require和import的区别](https://segmentfault.com/a/1190000021911869)
- [CommonJS和ES6模块的区别](https://juejin.cn/post/6844904067651600391)
- [Node.js 如何处理 ES6 模块](http://www.ruanyifeng.com/blog/2020/08/how-nodejs-use-es6-module.html)