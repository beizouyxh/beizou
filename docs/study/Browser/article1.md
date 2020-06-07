# 浏览器工作原理之渲染流程

最新的Chrome浏览器包括：1个浏览器主进程，一个GPU进程，一个网络进程，多个渲染进程和多个插件进程。

- **浏览器主进程**：主要负责界面展示，用户交互，子进程管理，提供存储等功能
- **渲染进程**：核心任务是将HTML，CSS，JavaScript 转化为用户可以与之交互的网页，排版引擎Blink和js引擎V8都是在该进程中运行，默认一个Tab标签对应一个渲染进程，但同一站点共(协议，根域名相同)享一个渲染进程。
- **GPU进程**：绘制网页的UI界面
- **网络进程**：负责网络资源的加载
- **插件进程**：负责插件的运行，保证插件进程崩溃不会对浏览器和页面造成影响

**优点**：多进程架构提升了浏览器的稳定性，安全性，和流畅性。

**缺点**：

1. 更高的资源占用
2. 更复杂的体系架构

未来的chrome将采用**面向服务的架构**，从而构建一个更加内聚，松耦合，易于维护和扩展的系统。



- **用户输入**

  浏览器首先会输入的搜索内容还是URL，如果是搜索内容则使用默认搜索引擎，合成新的带搜索关键字的URL，否则将会补全协议，合成完整的URL。

- **URL请求过程**

  浏览器会通过进程间通信（IPC）将URL请求发送至网络进程，由网络进程发起请求。

  **流程：**

   首先查看本地缓存，没有则进行DNS解析，获取服务器IP地址，https请求还要建立TLS连接，

  然后建立TCP连接，连接建立之后，浏览器会构建请求行，请求头等消息，服务器接收到请求信息后，根据信息生成响应数据，发送给网络进程。

  重定向：**在导航过程中，如果服务器响应行的状态码包含了 301、302 一类的跳转信息，浏览器会跳转到新的地址继续导航；如果响应行是 200，那么表示浏览器可以继续处理该请求**。

  响应数据类型处理：浏览器根据**Content-Type**区分数据类型，进行相应处理。

- **准备渲染进程**

  为页面分配渲染进程，如果是同一站点，则复用渲染进程（process-per-site-instance）

- **提交文档(响应体数据)**

  - 浏览器主进程发出”提交文档”消息，渲染进程收到后，和网络进程建立传输数据**管道**
  - 文档数据传输完毕后，渲染进程返回”**确认提交**“给浏览器主进程
  - 浏览器收到后，更新浏览器界面状态和Web页面

- **渲染阶段**

  **渲染流水线**：构建DOM树，样式计算，布局阶段，分层，绘制，分块，光栅化和合成。

  每个阶段都有三部分：

  - 开始每个子阶段都有其**输入的内容**；
  - 然后每个子阶段有其**处理过程**；
  - 最终每个子阶段会生成**输出内容**。

  **构建DOM树**

   **浏览器无法直接识别HTML，因此要将HTML转化为浏览器能够理解的结构——DOM树**

  **样式计算**

  1. **把CSS转化为浏览器能够理解的结构**

     CSS来源：link标签引入，style标记，元素内嵌的CSS

     **当渲染引擎接收到 CSS 文本时，会执行一个转换操作，将 CSS 文本转换为浏览器可以理解的结构——styleSheets**。

  2. **属性值标准化**

     CSS文本中有很多属性值，如 em，bold，blue，这些属性值不被渲染引擎理解，因此**需要将所有值转换为渲染引擎容易理解的、标准化的计算值**

  3. **计算DOM树每个节点的具体样式**

     两个规则：**继承和重叠**

     **继承**：**每个 DOM 节点都包含有父节点的样式**

     **层叠**：**层叠是 CSS 的一个基本特征，它是一个定义了如何合并来自多个源的属性值的算法。它在 CSS 处于核心地位，CSS 的全称“层叠样式表”正是强调了这一点**。

  **布局阶段**

   有了DOM树和样式之后，需要计算DOM树中可见元素的几何位置，这个计算过程叫做布局。

  1. 创建布局树
     - 遍历 DOM 树中的所有可见节点，并把这些节点加到布局中；
     - 不可见的节点会被布局树忽略掉
  2. **布局计算**

  **分层**

   由于页面中有很多复杂的效果，因此**渲染引擎还需要为特定的节点生成专用的图层，并生成一棵对应的图层树**，这些图层叠加在一起就构成了最终的页面图像。

   **浏览器的页面实际上被分成了很多图层，这些图层叠加后合成了最终的页面**

   通常情况下，**并不是布局树的每个节点都包含一个图层，如果一个节点没有对应的层，那么这个节点就从属于父节点的图层**。

  但满足以下任意一点就会被提升为一个单独的图层

  1. **拥有层叠上下文属性的元素会被提升为单独的一层**。

     明确定位属性的元素、定义透明属性的元素、使用 CSS 滤镜的元素等，都拥有层叠上下文属性。

  2. **需要剪裁（clip）的地方也会被创建为图层。**

  **图层绘制**

   完成图层树的构建之后，渲染引擎会对图层树的每个图层进行绘制。

  渲染引擎会把每一个图层的绘制拆分成很多的**绘制指令**，然后将这些指令组成一个待绘制列表。

  **栅格化操作**

   当图层的绘制列表准备好之后，主线程会把该绘制列表**提交（commit）**给合成线程

   有些情况下，有的图层很大，但是通过视口，用户只能看到页面的很小的一部分，因此，**合成线程会将图层划分为图块**。

   然后**合成线程会按照视口附近的图块来优先生成位图，实际生成位图的操作是由栅格化来执行的。所谓栅格化，是指将图块转换为位图**。

   渲染进程把生成图块的指令发送给 GPU，然后在 GPU 中执行生成图块的位图，并保存在 GPU 的内存中。

  **合成和显示**

   一旦所有图块都被光栅化，合成线程就会生成一个绘制图块的命令——“DrawQuad”，然后将该命令提交给浏览器进程。浏览器的viz组件根据 DrawQuad 命令，将其页面内容绘制到内存中，最后再将内存显示在屏幕上。