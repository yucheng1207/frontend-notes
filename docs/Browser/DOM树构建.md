# DOM树构建

## 什么是DOM
从网络传给渲染引擎的 HTML 文件字节流是无法直接被渲染引擎理解的，所以要将其转化为渲染引擎能够理解的内部结构，这个结构就是 DOM。DOM 提供了对 HTML 文档结构化的表述。在渲染引擎中，DOM 有三个层面的作用
- 从页面的视角来看，DOM 是生成页面的基础数据结构。
- 从 JavaScript 脚本视角来看，DOM 提供给 JavaScript 脚本操作的接口，通过这套接口，JavaScript 可以对 DOM 结构进行访问，从而改变文档的结构、样式和内容。
- 从安全视角来看，DOM 是一道安全防护线，一些不安全的内容在 DOM 解析阶段就被拒之门外了。

简言之，DOM 是表述 HTML 的内部数据结构，它会将 Web 页面和 JavaScript 脚本连接起来，并过滤一些不安全的内容。

## DOM树是怎么生成的
> HTML 解析器并不是等整个文档加载完成之后再解析的，而是网络进程加载了多少数据，HTML 解析器便解析多少数据。

1. **解析文件并创建渲染进程**
    `网络进程`接收到响应头并根据响应头中的 `content-type` 字段来判断文件的类型，如果文件类型为“text/html”则是HTML文件的，则选择或者创建一个`渲染进程`。
2. **网络进程将数据(字节流)传给渲染进程的 HTML解析器**
    `网络进程`和`渲染进程`之间会建立一个共享数据的管道，网络进程接收到数据后就往这个管道里面放，而`渲染进程`则从管道的另外一端不断地读取数据，并同时将读取的数据“喂”给 HTML 解析器。
3. **HTLM解析器通过分词器将字节流转换为Token**
    Token又可以分为以下几种类型
    - `Tag Token`：`Tag Token`又分 `StartTag` 和 `EndTag`
    - `文本 Token`
4. **HTLM解析器继续将Token解析为DOM节点，并将DOM节点添加到DOM树中**
    解析步骤为：
    a. 创建了一个根为`document`的空`DOM结构`，并将`document`压入 `Token栈`底（`StartTag document`入栈）
    b. 创建一个 `html DOM节点`，添加到 `document节点`（`StartTag html`入栈）
    c. 创建一个 `body DOM节点`，添加到上面创建的`html节点`中 （`StartTag body`入栈）
    d. 以此类推继续解析剩余的Token
    e. 当解析到`文本 Token`时创建一个文本节点并将该节点添加到离他最近的`Token节点`（`Tag Token`创建的节点）中
    f. 当解析到`EndTag`时将对应的`StartTag`出栈完成`Tag Token`该节点的解析
    g. 根据上述步骤完成整个`DOM树`的构建

![browser-DOM树构建-元素弹出Token栈](https://raw.githubusercontent.com/Coder-1024/image-host/main/imgs/frontend_notes/browser-DOM树构建-元素弹出Token栈.jpg)


## JavaScript 是如何影响 DOM 生成的
```html
<html>
<body>
    <div>1</div>
    <script>
    let div1 = document.getElementsByTagName('div')[0]
    div1.innerText = 'time.geekbang'
    </script>
    <div>test</div>
</body>
</html>
```
如上代码，当 DOM 解析的过程中遇到了 js 脚本，会暂停 DOM 解析，JavaScript 引擎介入，并执行 script 标签中的的脚本后再恢复 HTML 解析器，继续解析后续的内容，直至生成最终的 DOM。

```html
<html>
<body>
    <div>1</div>
    <script type="text/javascript" src='test.js'></script>
    <div>test</div>
</body>
</html>
```

但如果 html 中遇到了 js 文件 (如上代码)，流程就会变得相对复杂了。因为要解析 js 文件需要先把文件下载下来，如果等下载完再去解析 DOM 会造成 阻塞；Chrome 浏览器对此做了很多优化，其中一个主要的优化是预解析操作。当渲染引擎收到字节流之后，会开启一个预解析线程，用来分析 HTML 文件中包含的 JavaScript、CSS 等相关文件，解析到相关文件之后，预解析线程会**提前下载**这些文件。

> 引入 JavaScript 线程会阻塞 DOM，下面是一些 js 加载优化手段
> - CDN 来加速
> - 压缩 js 文件
> - 如果 JavaScript 文件中没有操作 DOM 相关代码，就可以将该 JavaScript 脚本设置为异步加载，通过 async 或 defer 来标记代码
>   ```js
>   <script async type="text/javascript" src='foo.js'></script>
>   <script defer type="text/javascript" src='foo.js'></script>
>   ```
>   async 和 defer 虽然都是异步的，不过还有一些差异，使用 async 标志的脚本文件一旦加载完成，会立即执行；而使用了 defer 标记的脚本文件，需要在 DOMContentLoaded 事件之前执行。

### 其他情况
```html
<html>
<head>
    <style src='theme.css'></style>
</head>
<body>
    <div>1</div>
    <script>
            let div1 = document.getElementsByTagName('div')[0]
            div1.innerText = 'time.geekbang' // 需要 DOM
            div1.style.color = 'red'  // 需要 CSSOM
        </script>
    <div>test</div>
</body>
</html>
```
该示例中，JavaScript 代码出现了 div1.style.color = ‘red' 的语句，它是用来操纵 CSSOM 的，所以在执行 JavaScript 之前，需要先解析 JavaScript 语句之上所有的 CSS 样式。所以如果代码里引用了外部的 CSS 文件，那么在执行 JavaScript 之前，还需要等待外部的 CSS 文件下载完成，并解析生成 CSSOM 对象之后，才能执行 JavaScript 脚本。

而 JavaScript 引擎在解析 JavaScript 之前，是不知道 JavaScript 是否操纵了 CSSOM 的，所以渲染引擎在遇到 JavaScript 脚本时，不管该脚本是否操纵了 CSSOM，都会**先执行 CSS 文件下载**，解析操作，再执行 JavaScript 脚本。

通过上面的分析，我们知道了 **JavaScript 会阻塞 DOM 生成**，而**样式文件又会阻塞 JavaScript 的执行**，所以在实际的工程中需要重点关注 JavaScript 文件和样式表文件，使用不当会影响到页面性能的


> 额外说明一下，渲染引擎还有一个安全检查模块叫 XSSAuditor，是用来检测词法安全的。在分词器解析出来 Token 之后，它会检测这些模块是否安全，比如是否引用了外部脚本，是否符合 CSP 规范，是否存在跨站点请求等。如果出现不符合规范的内容，XSSAuditor 会对该脚本或者下载任务进行拦截。

# Q & A
### 为什么要将 CSS 放在文件头部，JavaScript 文件放在底部
- CSS 执行会阻塞渲染，阻止 JS 执行
- JS 加载和执行会阻塞 HTML 解析，阻止 CSSOM 构建

如果这些 CSS、JS 标签放在 HEAD 标签里，并且需要加载和解析很久的话，那么页面就空白了。所以 JS 文件要放在底部（不阻止 DOM 解析，但会阻塞渲染），等 HTML 解析完了再加载 JS 文件，尽早向用户呈现页面的内容。

那为什么 CSS 文件还要放在头部呢？

因为先加载 HTML 再加载 CSS，会让用户第一时间看到的页面是没有样式的、“丑陋”的，为了避免这种情况发生，就要将 CSS 文件放在头部了。

另外，JS 文件也不是不可以放在头部，只要给 script 标签加上 defer 属性就可以了，异步下载，延迟执行。

参考：[使用 JavaScript 添加交互](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/adding-interactivity-with-javascript)




# 参考
- [DOM树：JavaScript是如何影响DOM树构建的](https://blog.poetries.top/browser-working-principle/guide/part5/lesson22.html#%E4%BB%80%E4%B9%88%E6%98%AF-dom)