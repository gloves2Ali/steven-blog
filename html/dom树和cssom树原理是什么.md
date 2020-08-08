### dom树和cssom树原理是什么
#### DOM 模型
1、DOM 标准
DOM (Document Object Model) 的全称是文档对象模型，它可以以一种独立于平台和语言的方式访问和修改一个文档的内容和结构。比如，Web开发中，用 JavaScript 语言来访问、创建、删除或者修改 HTML 的文档结构。

2、 DOM 树
在介绍 DOM 树之前，首先要清楚，DOM 规范中，对于文档的表示方法并没有任何限制，因此，DOM 树只是多种文档结构中的一种较为普遍的实现方式。

DOM 结构构成的基本要素是 “节点“，而文档的结构就是由层次化的节点组成。在 DOM 模型中，节点的概念很宽泛，整个文档 （Document） 就是一个节点，称为文档节点。除此之外还有元素（Element）节点、属性节点、Entity节点、注释（Comment）节点等。
了解了 DOM 的结构是由各种的子节点组成的，那么以 HTMLDocument 为根节点，其余节点为子节点，组织成一个树的数据结构的表示就是 DOM树。

#### HTML 解释器

1、解释过程
HTML 解释器的工作就是将网络或者本地磁盘获取的 HTML 网页和资源从字节流解释成 DOM 树结构。
从资源的字节流到 DOM 树
通过上图可以清楚的了解这一过程：首先是字节流，经过解码之后是字符流，然后通过词法分析器会被解释成词语（Tokens），之后经过语法分析器构建成节点，最后这些节点被组建成一颗 DOM 树。
在这个过程中，每一个环节都会调用对应的类去处理

+ 词法分析： HTMLTokenizer 类
+ 词语验证：XSSAuditor 类
+ 从词语到节点： HTMLDocumentParser 类、 + HTMLTreeBuilder 类
+ 从节点到 DOM 树： HTMLConstructionSite 类

对于线程化的解释器，字符流后的整个解释、布局和渲染过程基本会交给一个单独的渲染线程来管理（不是绝对的）。由于 DOM 树只能在渲染线程上创建和访问，所以构建 DOM 树的过程只能在渲染线程中进行。但是，从字符串到词语这个阶段可以交给单独的线程来做，Chromium 浏览器使用的就是这个思想。在解释成词语之后，Webkit 会分批次将结果词语传递回渲染线程。

#### JavaScript 的执行
在 HTML 解释器的工作过程中，可能会有 JavaScript 代码需要执行，它发生在将字符串解释成词语之后、创建各种节点的时候。这也是为什么全局执行的 JavaScript 代码不能访问 DOM 的原因——因为 DOM 树还没有被创建完呢。

WebKit 将 DOM 树创建过程中需要执行的 JavaScript 代码交由 HTMLScriptRunner 类来负责，其利用 JavaScript 引擎来执行 Node 节点中包含的代码。

因为 JavaScript 代码可能会修改文档结构，所以代码的执行会阻碍后面节点的创建，同时也会阻碍后面的资源下载，这样就会导致资源不能并发下载的性能问题。所以一般建议：

1、在 “script“ 标签上加上 “async“ 或 “defer“ 属性。
2、将 “script“ 元素放在 “body“ 元素后面。

对于此，WebKit 也通过预扫描和预加载来实现对资源并发下载的优化。

具体过程就是当需要执行 JavaScript 代码的时候，WebKit 先暂停代码的执行，使用预扫描器 HTMLPreloadScanner 类来扫描后面的词语， 如果发现需要使用其他资源，那么就会使用与资源加载器 HTMLResourcePreloader 类来发送请求，在这之后，才执行 JavaScript 代码。由于预扫描器本身并不创建节点对象，也不会构建 DOM 树，所以速度比较快。就算如此，还是推荐不要在头部写入大量 JavaScript 代码，毕竟不是所有渲染引擎都做了这样的优化。

在 DOM 树构建完成后，WebKit 会触发 “DOMContentLoaded” 事件，当所有资源都被加载完成后，会触发 “onload” 事件。