# Diff算法

## React diff - 递增法

- ### tree diff
    - react对树的算法进行了分层比较，只会对比同一层级的节点（即只会对比同一个父节点下的所有子节点）。当发现节点不存在，则该节点和其子节点都会被删除。这样只需要遍历一次dom树，就完成了整个dom树的对比
    - 当进行跨层级的移动操作，react并不是简单的进行移动，而是进行了删除和创建的操作，这会影响到react性能。所以要尽量避免跨层级的操作。（例如：控制display来达到显示和隐藏，而不是真的添加和删除dom）

- ### component diff
    - 如果是同类型的组件，则直接对比virtual Dom tree
    - 如果不是同类型的组件，会直接替换掉组件下的所有子组件
    - 如果类型相同，但是可能virtual DOM 没有变化，这种情况下我们可以使用shouldComponentUpdate() 来判断是否需要进行diff

- ### element diff
    - #### 不使用key
        新旧节点对比，如果不是相同的节点则删除后重新创建

    - #### 使用key
        - 移动
            - 对比新旧两组节点（newList & prevList），首先遍历 newList 并维护一个”lastIndex“ 来记录上一个节点的位置，然后找到当前节点在 prevList 中的位置index，如果 index 大于 lastIndex 则不需要移动，否则，将该节点对应的DOM移动到 当前遍历到的 newList 节点对应的真实DOM后面
                ```
                            ↱¯¯¯¯¯¯¯¯¯↴
                DOM        A  [B]  C  D [B]
                prevList   a   b   c  d
                newList    a   c   d  b
                ```
        - 新增
            - 同样先遍历 newList，如果发现在 prevList 中找不到节点，则将该节点对应的DOM插入到当前遍历到的 newList 节点对应的真实DOM 后面
                ```
                DOM        A  B  [+C]  D
                prevList   a  b        d
                newList    a  b    c   d
                ```
        - 删除
            - 当旧的节点不在newList中时，我们就将其对应的DOM节点移除。
                ```
                DOM        A  B [-C] D
                prevList   a  b   c  d
                newList    a  b      d
                ```
- #### 时间复杂度
    目前的reactDiff的时间复杂度为O(m*n)，我们可以用空间换时间，把key与index的关系维护成一个Map，从而将时间复杂度降低为O(n)


## Vue2.x diff - 双端比较

- ### MVVM原理
    M (model)：模型对象：指的是构成界面内容的相关数据
    V(view): view: 视图对象：指的是给用户或者开发者展示数据的界面
    VM(viewmodel): 视图模型对象：是view与model之间的桥梁

    MVVM就是通过 数据劫持 + 发布订阅模式 实现双向绑定。数据驱动视图具体过程为 监听数据变化 -> 收集依赖 -> 通知依赖更新 三个步骤。

    当 new 一个 vue 实例的时候，vue会调用 `_init()` 方法进行初始化（vue的声明周期、事件、props、data、compute、watch等，最重要的是）初始化会通过 Object.defineProperty来设置 getter 和 setter 函数。（getter 函数用于依赖收集，setter函数用于通知内部所有的 Watcher 对象进行视图更新）

    当组件被渲染时，会去读取所需数据，这个读取操作会触发 getter 函数， getter函数会进行 依赖收集。依赖收集的过程就是把 Watcher 实例放到 Dep（依赖收集器）中（Dep 调用 addSub方法）。
    当数据被修改时，会触发 setter 函数，setter 函数会通知 Dep（依赖收集器）有数据变化了，Dep会找到对应的 Watcher 进行更新（Dep 调用 notify 方法，调用 Watcher update方法）

- ### Vue.js（2.x）运行机制

    1. **初始化**：调用 `new Vue()` 后 vue 会调用 `_init()` 函数进行初始化，它会初始化生命周期、事件、 props、 methods、 data、 computed 与 watch 等。其中最重要的是**通过 Object.defineProperty 设置 setter 与 getter 函数，用来实现「响应式」以及「依赖收集」**

    2. **挂载组件**：初始化之后会调用 `$mount` 挂载组件，如果是运行时编译，即不存在 render function 但是存在 template 的情况，需要进行 compile 步骤

    3. **编译**：compile 编译可以分成 parse、optimize 与 generate 三个阶段，最终得到渲染 VNode 所需的 render function
        - parse：用正则等方式解析 template 模板中的指令、class、style等数据，形成AST
            - 维护一个 stack 栈来解析标签，文本的话可以直接取出
        - optimize：主要作用是标记 static 静态节点，这是 Vue 在编译过程中的一处优化，后面当 update 更新界面时，会有一个 patch 的过程， diff 算法会直接跳过静态节点，从而减少了比较的过程，优化了 patch 的性能。
        - generate： 将 AST 转化成 render function 字符串的过程，得到结果是 render 的字符串以及 staticRenderFns 字符串。

    4. **生成虚拟DOM&响应式**
        - 生成虚拟DOM：render function 会被转化成 VNode 节点。Virtual DOM 其实就是一棵以 JavaScript 对象（ VNode 节点）作为基础的树，用对象属性来描述节点，实际上它只是一层对真实 DOM 的抽象。最终可以通过一系列操作使这棵树映射到真实环境上。由于Virtual DOM 是以 JavaScript 对象为基础而不依赖真实平台环境，所以使它具有了跨平台的能力，比如说浏览器平台、Weex、Node 等。
            - VNode 就是一个 JavaScript 对象，用 JavaScript 对象的属性来描述当前节点的一些状态，用 VNode 节点的形式来模拟一棵 Virtual DOM 树。
            - 标签节点
            - 文本节点
        - 响应式：setter -> Dep -> Watcher -> patch -> 视图
            - 依赖收集：当 render function 被渲染时，会读取所需对象的值，由于进行了读取操作，所以会触发 getter 函数进行 ”依赖收集“。「依赖收集」的过程就是把 Watcher 实例存放到对应的 Dep 对象中去。get 方法可以让当前的 Watcher 对象（Dep.target）存放到它的 subs 中（addSub）方法，在数据变化时，set 会调用 Dep 对象的 notify 方法通知它内部所有的 Watcher 对象进行视图更新（update）。
            - 当在修改对象的值的时候，会触发对应的 setter， setter 通知之前「依赖收集」得到的 Dep 中的每一个 Watcher，告诉它们自己的值改变了，需要重新渲染视图。这时候这些 Watcher 就会开始调用 update 来更新视图，当然这中间还有一个 patch 的过程以及使用队列来异步更新的策略。
    5. patch：在对 model 进行操作对时候，会触发对应 Dep 中的 Watcher 对象。Watcher 对象会调用对应的 update 来修改视图。最终是将新产生的 VNode 节点与老 VNode 进行一个 patch 的过程，比对得出「差异」，最终将这些「差异」更新到视图上。
        - diff算法：如下

- ### diff算法
    vue2.x使用的是双端比较，所谓双端比较就是新列表和旧列表两个列表的头与尾互相对比，，在对比的过程中指针会逐渐向内靠拢，直到某一个列表的节点全部遍历过，对比停止。
    - 理想情况
        - 先用四个指针指向两个列表的头尾，对比指针对应的节点，一共有四种情况，对比顺序如下：
            - prevList 的头节点跟 newList 的 头节点作对比（a↔d）
            - prevList 的尾节点跟 newList 的 尾节点作对比（d↔c）
            - prevList 的头节点跟 newList 的 尾节点作对比（a↔c）
            - prevList 的尾节点跟 newList 的 头节点作对比（d↔d）
            ```
            DOM        A   B   C   D
                    ↓           ↓
            prevList   a   b   c   d

            newList    d   b   a   c
                    ↑           ↑
            ```
        - 同过上面四种情况对比找出相同的节点，找到相同的节点则对真实DOM进行移动，然后将对应的指针向内移动，如上情况经过一次对比后会变成
            ```
                        ↙¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯↖
            DOM        [+D]  A   B   C  [-D]
                            ↓       ↓
            prevList         a   b   c   d

            newList      d   b   a   c
                            ↑       ↑
            ```
        - 继续对比，发现相同的节点为c，真实DOM不需要移动，继续将指针向内移动
            ```
            DOM          D   A   B   C
                            ↓   ↓
            prevList         a   b   c   d

            newList      d   b   a   c
                            ↑   ↑
            ```
        - 继续对比，发现相同节点 prevList头节点a跟newList尾节点a相同（根据上面的对比顺序），移动对应的真实DOM并将指针向内移动
            ```
                            ↗¯¯¯¯¯¯¯↘
            DOM          D [-A]  B  [+A]  C
                                ↓
            prevList         a   b        c   d

            newList      d   b   a        c
                            ↑
            ```
        - 最后发现prevList头节点跟newList头节点相同，不需要移动，结束对比
    - 非理想情况
        - 上面每次对比都能找到相同的节点，那要是**没有找到相同节点**怎么办呢
            ```
            DOM         A   B   C   D
                        ↓           ↓
            prevList    a   b   c   d

            newList     b   e   a   c   f
                        ↑               ↑
            ```
        - 拿 newList 的第一个节点**去 prevList 找相同的节点**，如果**找到了**则将找到的节点设置为undefined（后面遇到这个undefined直接跳过）并将其对应的DOM移动到开头，然后将 newList 头指针向内移动
            ```

            DOM      [+B]  A  [-B]  C   D
                           ↓            ↓
            prevList       a    -   c   d

            newList        b    e   a   c   f
                                ↑           ↑
            ```
        - 继续对比发现**没有找到**对应的节点，直接创建一个新的节点放到最前面就可以了并将 newList 头指针向内移动
            ```

            DOM      B  [+E]  A        C   D
                              ↓            ↓
            prevList          a    -   c   d

            newList           b    e   a   c  f
                                       ↑      ↑
            ```
        - **删除节点**：继续对比最后会变成如下情况，此时newList的 尾节点index 小于 头节点index，表示对比结束，将**剩余节点删除**即可
            ```

            DOM         B  E  A        C   F   D
                                               ↓↓
            prevList          a    -   c       d

            newList           b    e   a   c   f
                                               ↑   ↑(头index)
            ```
        - **添加节点**：假设最后的情况如下，prevList 的尾节点 index 小于 头节点 index，则将 newList 中剩余的节点插入到 prevList头节点对应的DOM之前即可
            ```

            DOM             A   B   C
                (尾index)↓  ↓
            prevList        a   b   c

            newList         d   a   b   c
                           ↑↑
            ```


## Vue3.x diff - 最长递增子序列

1. 前置与后置元素的预处理
    - 首先从 newList 头节点开始比较，遇到相同的节点指针向内移动，否则跳出循环
    - 从 newList 尾节点开始比较，遇到相同节点指针向内移动，否则跳出循环
2. 剩余的节点就是不确定元素了，使用最长递增子序列进行节点移动、新增或删除

### 最长递增子序列
- [最长递增子序列算法题](https://leetcode-cn.com/problems/longest-increasing-subsequence/)

经过前置与后置元素预处理后就可以计算出剩余的节点的最长递增子序列了，具体操作如下
1. 先根据新列表剩余的节点数量，创建一个source数组，并将数组填满-1。
```
    DOM        A   B   C   D   F   E
                   ↓           ↓
    prevList   a   b   c   d   f   e

    newList    a   c   d   b   g   e
                   ↑           ↑
    source       [-1  -1  -1  -1]
```
2. 然后根据新节点在旧列表的位置存储在该数组中，如果旧节点在新列表中没有的话，直接删除（如F），对于全新的节点，其在source中的值为-1，通过这一步我们可以区分出来哪个为全新的节点，哪个是可复用的。
```
    DOM        A   B   C   D [-F]  E
                   ↓           ↓
    prevList   a   b   c   d   f   e

    newList    a   c   d   b   g   e
                   ↑           ↑
    source       [ 2   3   1  -1]
```
3. 计算出source的最长递增子序列
```
    DOM        A   B   C   D [-F]  E
                   ↓           ↓
    prevList   a   b   c   d   f   e

    newList    a   c   d   b   g   e
                   ↑           ↑
    source       [ 2   3   1  -1]
    Lis          [ 0   1 ]
```
4. 从后向前进行遍历source每一项，此时会出现三种情况：
    - 当前的值为-1，这说明该节点是全新的节点，又由于我们是从后向前遍历，我们直接创建好DOM节点插入到队尾就可以了。
    - 当前的索引为最长递增子序列中的值，也就是i === seq[j]，这说说明该节点不需要移动
    - 当前的索引不是最长递增子序列中的值，那么说明该DOM节点需要移动，这里也很好理解，我们也是直接将DOM节点插入到队尾就可以了，因为队尾是排好序的。
```
                   ↗¯¯¯¯¯¯¯¯¯¯↘
    DOM        A [-B]  C   D [+B] [+G] E
                   ↓               ↓
    prevList   a   b   c   d       f   e

    newList    a   c   d   b       g   e
                   ↑               ↑
    source       [ 2   3   1      -1]
    Lis          [ 0   1 ]
```



# 参考
- [React、Vue2、Vue3的三种Diff算法](https://juejin.cn/post/6919376064833667080)
- [聊一聊Diff算法（React、Vue2.x、Vue3.x）](https://zhuanlan.zhihu.com/p/149972619)
- [剖析 Vue.js 内部运行机制 - 染陌同学](https://juejin.cn/book/6844733705089449991)