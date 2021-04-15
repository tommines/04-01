### 1.请简述 React 16 版本中初始渲染的流程

​         要将 React 元素渲染到页面中，分为两个阶段，render 阶段和 commit 阶段

​         render 阶段负责创建 Fiber 数据结构并为 Fiber 节点打标记，标记当前 Fiber 节点要进行的 DOM 操作。

​         commit 阶段负责根据 Fiber 节点标记 ( effectTag ) 进行相应的 DOM 操作。

### 2.为什么 React 16 版本中 render 阶段放弃了使用递归

在React 15的版本中，采用了循环加递归的方式进行virtualDOM的比对，由于递归使用JavaScript自身的执行栈，一旦开始就无法停止，直到任务执行完成。如果VirtualDOM树的层级比较深，virtualDOM的比对就会长期占用JavaScript主线程，由于Javascript又是单线程的无法同时执行其他任务，所以在比对的过程中无法响应用户操作，无法即时执行元素动画，造成了页面卡顿的现象。

在React 16的版本中，放弃了Javascript 递归的方式进行virtualDOM的对比，而是采用循环模拟递归。而且比对的过程是利用浏览器的空闲时间完成的，不会长期占用主线程，这就解决了virtualDOM比对造成页面卡顿的问题。

### 3.请简述 React 16 版本中 commit 阶段的三个子阶段分别做了什么事情

​      **第一个子阶段：**  **before mutation 阶段 （执行DOM操作前）**

​          调用类组件的 getSnapshotBeforeUpdate 生命周期函数

​      **第二个子阶段：** **mutation 阶段 （执行DOM操作）**

​           根据 effectTag 执行 DOM 操作, 挂载 DOM 元素, 获取 HostRootFiber 对象,向容器中追加 | 插入到某一个节点的前面

​      **第三个子阶段：** **layout阶段（执行DOM操作后）**

​          执行渲染完成之后的回调函数,useEffect 回调函数调用

### 4.请简述 workInProgress Fiber 树存在的意义是什么

workInProgress Fiber 树存在的意义是为了实现React的双缓存技术的实现。

在 React 中，DOM 的更新采用了双缓存技术，双缓存技术致力于更快速的 DOM 更新。

什么是双缓存？举个例子，使用 canvas 绘制动画时，在绘制每一帧前都会清除上一帧的画面，清除上一帧需要花费时间，如果当前帧画面计算量又比较大，又需要花费比较长的时间，这就导致上一帧清除到下一帧显示中间会有较长的间隙，就会出现白屏。

为了解决这个问题，我们可以在内存中绘制当前帧动画，绘制完毕后直接用当前帧替换上一帧画面，这样的话在帧画面替换的过程中就会节约非常多的时间，就不会出现白屏问题。这种在内存中构建并直接替换的技术叫做双缓存。

React 使用双缓存技术完成 Fiber 树的构建与替换，实现DOM对象的快速更新。

在 React 中最多会同时存在两棵 Fiber 树，当前在屏幕中显示的内容对应的 Fiber 树叫做 current Fiber 树，当发生更新时，React 会在内存中重新构建一颗新的 Fiber 树，这颗正在构建的 Fiber 树叫做 workInProgress Fiber 树。在双缓存技术中，workInProgress Fiber 树就是即将要显示在页面中的 Fiber 树，当这颗 Fiber 树构建完成后，React 会使用它直接替换 current Fiber 树达到快速更新 DOM 的目的，因为 workInProgress Fiber 树是在内存中构建的所以构建它的速度是非常快的。

一旦 workInProgress Fiber 树在屏幕上呈现，它就会变成 current Fiber 树。

在 current Fiber 节点对象中有一个 alternate 属性指向对应的 workInProgress Fiber 节点对象，在 workInProgress Fiber 节点中有一个 alternate 属性也指向对应的 current Fiber 节点对象。