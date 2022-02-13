# Hash路由和History路由

## 什么是 SPA
SPA 是 `single page web application` 的简称，译为单页Web应用。

SPA 就是一个 WEB 项目只有一个 HTML 页面，一旦页面加载完成，SPA 不会因为用户的操作而进行页面的重新加载或跳转。 取而代之的是利用 JS 动态的变换 HTML 的内容，从而来模拟多个视图间跳转


## 什么是前端路由

简单的说，就是在保证只有一个 HTML 页面，且与用户交互时不刷新和跳转页面的同时，为 SPA 中的每个视图展示形式匹配一个特殊的 url。在刷新、前进、后退和SEO时均通过这个特殊的 url 来实现。

路由需要实现三个功能：
- 当浏览器地址变化时，切换页面（但浏览器不向服务器发送请求）；
- 点击浏览器【后退】、【前进】按钮，网页内容跟随变化；
- 刷新浏览器，网页加载当前路由对应内容；


接下来要介绍的 hash 模式和 history 模式，就是实现了上面的功能


## Hash路由和History路由区别

- **Hash路由**
    - 描述：监听浏览器地址hash值变化，执行相应的js切换网页；【[MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/hashchange_event)】
    - 要点
        - hash指的是地址中#号以及后面的字符，也称为散列值。hash也称作锚点，本身是用来做页面跳转定位的。如`http://localhost/index.html#abc`，这里的#abc就是hash；
        - 散列值是不会随请求发送到服务器端的，所以改变hash，不会重新加载页面；
        - 监听 window 的 hashchange 事件，当散列值改变时，可以通过 location.hash 来获取和设置hash值；
            - **触发 hashchange 事件的几种情况**
                - 浏览器地址栏散列值的变化（包括浏览器的前进、后退）会触发`window.location.hash`值的变化，从而触发 onhashchange 事件；
                - 当浏览器地址栏中URL包含哈希如 `http://www.baidu.com/#home`，这时按下输入，浏览器发送`http://www.baidu.com/`请求至服务器，请求完毕之后设置散列值为#home，进而触发onhashchange事件；
                - 当只改变浏览器地址栏URL的哈希部分，这时按下回车，浏览器不会发送任何请求至服务器，这时发生的只是设置散列值新修改的哈希值，并触发 onhashchange 事件；
                - html 中`<a>标签`的属性 href 可以设置为页面的元素ID如 #top，当点击该链接时页面跳转至该id元素所在区域，同时浏览器自动设置 `window.location.hash` 属性，地址栏中的哈希值也会发生改变，并触发 onhashchange 事件；
        - location.hash值的变化会直接反应到浏览器地址栏；

- **History路由**
    - 描述：利用 H5 的 History API 实现 url 地址改变，网页内容改变；【[History_API MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/History_API)】
    - 要点
        - window.history 属性指向 History 对象，它表示当前窗口的浏览历史。当发生改变时，只会改变页面的路径，不会刷新页面。
        - 浏览器工具栏的“前进”和“后退”按钮，其实就是对 History 对象进行操作。
        - History 对象主要有**两个属性**：
            - `History.length`：当前窗口访问过的网址数量（包括当前网页，由于安全原因，**浏览器不允许脚本读取这些网址**，但是允许在地址之间导航。）
            - `History.state`：History 堆栈最上层的状态值
        - History 对象的**几个方法**：
            - `History.back()`： 移动到上一个网址，等同于点击浏览器的后退键。对于第一个访问的网址，该方法无效果。
            - `History.forward()`:移动到下一个网址，等同于点击浏览器的前进键。对于最后一个访问的网址，该方法无效果。
            - `History.go()`: 接受一个整数作为参数，以当前网址为基准，移动到参数指定的网址。如果参数超过实际存在的网址范围，该方法无效果；如果不指定参数，默认参数为0，相当于刷新当前页面。
                ```js
                history.go(0) // 刷新当前页面
                history.go(1) // 相当于history.forward()
                history.go(-1) // 相当于history.back()
                ```
            > 注意：移动到以前访问过的页面时，页面通常是从浏览器缓存之中加载，而不是重新要求服务器发送新的网页。
            - `History.pushState()`：用于在历史中添加一条记录。pushState()方法不会触发页面刷新，只是导致 History 对象发生变化，地址栏会有变化；该方法接受三个参数，依次为：
                - `object`：是一个对象，通过 pushState 方法可以将该对象内容传递到新页面中。如果不需要这个对象，此处可以填 null。
                - `title`：指标题，几乎没有浏览器支持该参数，传一个空字符串比较安全。
                - `url`：新的网址，必须与当前页面处在**同一个域**。不指定的话则为当前的路径，如果设置了一个跨域网址，则会报错。
            - `History.replaceState()`: 用来修改 History 对象的当前记录，用法与 pushState() 方法一样。
        - **popstate 事件**： 每当 history 对象出现变化时，就会触发 popstate 事件。
            - 仅仅调用`pushState()`方法或`replaceState()`方法 ，并不会触发该事件;
            - 只有用户点击浏览器倒退按钮和前进按钮，或者使用 JavaScript 调用`History.back()`、`History.forward()`、`History.go()`方法时才会触发。
            - 另外，该事件只针对同一个文档，如果浏览历史的切换，导致加载不同的文档，该事件也不会触发。
            - 页面第一次加载的时候，浏览器不会触发popstate事件。
- 如何选择
    - Hash路由兼容性更好，可以兼容到IE8，无需服务端配合处理非单页的url地址，但是由于所有操作都在客户端完成，路由变化不算一次http请求，这种模式不利于SEO
    - History看起来更美观（没有“#”），使用时需要**通过服务端来允许地址可访问**，如果没有设置，就很容易导致出现 404 的局面。（服务端支持：需要后端配置 Nginx）
    - 当我们不需要兼容老版本IE浏览器，并且可以控制服务端覆盖所有情况的候选资源时，我们可以愉快的使用 history 模式了。反之，很遗憾，只能使用丑陋的 hash 模式~




## 演示
```js
//http://127.0.0.1:8001/01-hash.html?a=100&b=20#/aaa/bbb
location.protocal // 'http:'
localtion.hostname // '127.0.0.1'
location.host // '127.0.0.1:8001'
location.port //8001
location.pathname //'01-hash.html'
location.serach // '?a=100&b=20'
location.hash // '#/aaa/bbb'
```

## 模拟实现

> 以下代码拷贝至[面试官: 你了解前端路由吗?](https://juejin.cn/post/6844903589123457031)

### Hash路由
> [在线例子](https://codepen.io/xiaomuzhu/pen/VXVrxa/)
```js
class Routers {
  constructor() {
    // 储存hash与callback键值对
    this.routes = {};
    // 当前hash
    this.currentUrl = '';
    // 记录出现过的hash
    this.history = [];
    // 作为指针,默认指向this.history的末尾,根据后退前进指向history中不同的hash
    this.currentIndex = this.history.length - 1;
    this.refresh = this.refresh.bind(this);
    this.backOff = this.backOff.bind(this);
    // 默认不是后退操作
    this.isBack = false;
    window.addEventListener('load', this.refresh, false);
    window.addEventListener('hashchange', this.refresh, false);
  }

  route(path, callback) {
    this.routes[path] = callback || function() {};
  }

  refresh() {
    this.currentUrl = location.hash.slice(1) || '/';
    if (!this.isBack) {
      // 如果不是后退操作,且当前指针小于数组总长度,直接截取指针之前的部分储存下来
      // 此操作来避免当点击后退按钮之后,再进行正常跳转,指针会停留在原地,而数组添加新hash路由
      // 避免再次造成指针的不匹配,我们直接截取指针之前的数组
      // 此操作同时与浏览器自带后退功能的行为保持一致
      if (this.currentIndex < this.history.length - 1)
        this.history = this.history.slice(0, this.currentIndex + 1);
      this.history.push(this.currentUrl);
      this.currentIndex++;
    }
    this.routes[this.currentUrl]();
    console.log('指针:', this.currentIndex, 'history:', this.history);
    this.isBack = false;
  }
  // 后退功能
  backOff() {
    // 后退操作设置为true
    this.isBack = true;
    this.currentIndex <= 0
      ? (this.currentIndex = 0)
      : (this.currentIndex = this.currentIndex - 1);
    location.hash = `#${this.history[this.currentIndex]}`;
    this.routes[this.history[this.currentIndex]]();
  }
}
```

### History路由
> [在线例子](https://codepen.io/xiaomuzhu/pen/QmJorQ/)
```js
class Routers {
  constructor() {
    this.routes = {};
    // 在初始化时监听popstate事件
    this._bindPopState();
  }
  // 初始化路由
  init(path) {
    history.replaceState({path: path}, null, path);
    this.routes[path] && this.routes[path]();
  }
  // 将路径和对应回调函数加入hashMap储存
  route(path, callback) {
    this.routes[path] = callback || function() {};
  }

  // 触发路由对应回调
  go(path) {
    history.pushState({path: path}, null, path);
    this.routes[path] && this.routes[path]();
  }
  // 监听popstate事件
  _bindPopState() {
    window.addEventListener('popstate', e => {
      const path = e.state && e.state.path;
      this.routes[path] && this.routes[path]();
    });
  }
}
```

# 参考
- [前端路由的两种模式：hash模式和 history模式](https://blog.csdn.net/Charissa2017/article/details/104779412)
- [面试官: 你了解前端路由吗?](https://juejin.cn/post/6844903589123457031)
    - hash路由和history路由的模拟实现
- [「前端进阶」彻底弄懂前端路由](https://juejin.cn/post/6844903890278694919)
- [浅谈前端路由原理hash和history](https://juejin.cn/post/6993840419041706014)