#### VNode  

**<u>抽象 Dom 树</u>**
在Vue中，我们把真实Dom树抽象一棵以javascript对象构成的抽象树，在修改抽象树数据后将抽象树转化成真实Dom重绘到页面上呢？于是虚拟Dom出现了，它是真实Dom的一层抽象，用属性描述真实Dom的各个特性。当它发生变化时候，就会去修改视图。  

但是这样的js操作Dom进行重绘整个视图是非常消耗性能的，我们是不是可以做到每次只更新它的修改呢？所以Vue将Dom抽象成一个以javascript对象为节点的虚拟Dom树，以VNode节点模拟真实Dom，可以对这颗抽象树进行创建节点、删除节点以及修改节点等操作，在这个过程中都不需要操作真实Dom，只需要操作javascript对象即可，大大提升了性能。修改以后经过diff算法得出一些需要修改的最小单位，再将这些小单位的视图进行更新。这样做减少了很多不需要的Dom操作，大大提高了性能。  

Vue就使用了这样的抽象节点VNode，它对真实Dom的一层抽象，而不依赖某个平台，它可以是浏览器平台，也可以是weex，甚至是node平台也可以对这样一棵抽象Dom树进行创建删除修改等操作，这也为前后端同构提供了可能。
  
**<u>VNode 基类</u>**
先来看一下vue源码中对VNode类的定义
``` javascript
  export default class VNode {
  tag: string | void;
  data: VNodeData | void;
  children: ?Array<VNode>;
  text: string | void;
  elm: Node | void;
  ns: string | void;
  context: Component | void; // rendered in this component's scope
  functionalContext: Component | void; // only for functional component root nodes
  key: string | number | void;
  componentOptions: VNodeComponentOptions | void;
  componentInstance: Component | void; // component instance
  parent: VNode | void; // component placeholder node
  raw: boolean; // contains raw HTML? (server only)
  isStatic: boolean; // hoisted static node
  isRootInsert: boolean; // necessary for enter transition check
  isComment: boolean; // empty comment placeholder?
  isCloned: boolean; // is a cloned node?
  isOnce: boolean; // is a v-once node?

  constructor (
    tag?: string,
    data?: VNodeData,
    children?: ?Array<VNode>,
    text?: string,
    elm?: Node,
    context?: Component,
    componentOptions?: VNodeComponentOptions
  ) {
    /*当前节点的标签名*/
    this.tag = tag
    /*当前节点对应的对象，包含了具体的一些数据信息，是一个 VNodeData 类型，可以参考 VNodeData 类型中的数据信息*/
    this.data = data
    /*当前节点的子节点，是一个数组*/
    this.children = children
    /*当前节点的文本*/
    this.text = text
    /*当前虚拟节点对应的真实 dom 节点*/
    this.elm = elm
    /*当前节点的名字空间*/
    this.ns = undefined
    /*编译作用域*/
    this.context = context
    /*函数化组件作用域*/
    this.functionalContext = undefined
    /*节点的 key 属性，被当作节点的标志，用以优化*/
    this.key = data && data.key
    /*组件的 option 选项*/
    this.componentOptions = componentOptions
    /*当前节点对应的组件的实例*/
    this.componentInstance = undefined
    /*当前节点的父节点*/
    this.parent = undefined
    /*简而言之就是是否为原生 HTML 或只是普通文本，innerHTML 的时候为 true，textContent 的时候为 false*/
    this.raw = false
    /*静态节点标志*/
    this.isStatic = false
    /*是否作为跟节点插入*/
    this.isRootInsert = true
    /*是否为注释节点*/
    this.isComment = false
    /*是否为克隆节点*/
    this.isCloned = false
    /*是否有 v-once 指令*/
    this.isOnce = false
  }

  // DEPRECATED: alias for componentInstance for backwards compat.
  /* istanbul ignore next https://github.com/answershuto/learnVue*/
  get child (): Component | void {
    return this.componentInstance
  }
}
```
这是一个最基础的VNode节点，作为其他派生Vnode类的基类，里面定义了下面这些数据。
```javascript   
  tag：当前节点的标签名
  data：当前节点对应的对象，包含了具体的一些数据信息，是一个VNodeData类型，可以参考VNodeData类型中的数据信息
  children：当前节点的子节点，是一个数组
  text：当前节点的文本
  elm：当前虚拟节点对应的真实dom节点
  ns：当前节点的名字空间
  context：当前节点的编译作用域
  functionalContext：函数化组件作用域
  key：节点的key属性，被当作节点的标志，用以优化
  componentOptions：组件的option选项
  componentInstance：当前节点对应的组件的实例
  parent：当前节点的副节点
  raw：简而言之就是是否为原生HTML或只是普通文本，innerHTMl的时候为true，textContent的时候为false
  isStatic：是否为静态节点
  isRootInsert：是否作为跟节点插入
  isCOmment：是否为注释节点
  isCloned：是否为克隆节点
  isOnce：是否有v-once指令
```
以下是一个VNode树
```javascript
{
  tag: 'div'
  data: {
    class: 'btn',
    id: 'Btn'
  },
  children: [
    tag: 'p',
    data: {
      class: 'p-btn'
    }
    text: 'vnode'
  ]
}
```
渲染之后的结果就是这样
```javascript
<div class="btn" id="Btn">
  <p class="p-btn">vnode</p>
</div>
```