### 1、简介
我们知道vue生成dom树的过程可以概括为3步：
1.编译出render函数
2.生成虚拟dom
3.生成真实dom


### 2.一个简单的例子
通过一个简单的例子说明父子组建生成的过程：
```javascript
new Vue({
  template: `<child></child>`,
  components: {
    child: {
      template: `<div></div>`
    }
  }
}).$mount();
```

#### 2.1父组件生成render函数
父组件编译模版，生成render函数，看下结果：
parentVM属性说明：
+ 0是实例id
+ _vnode（虚拟节点）是null
+ render作为$options的一个属性保存
+ $parent为空
+ $children为空数组
编译过程不是本文重点，所以直接给出render函数：
```javascript
function anonymous() {
  with(this) {return _('child')}
}
```
注意
+ 这是_父组件_的render函数
+ 此时没有_子组件_的实例，更加没有_子组件_的render函数

#### 2.2父组件生成虚拟dom
_父组件_运行_render函数（内部调用render）生成虚拟dom，看下结果：

parentVnode属性说明：

+ tag是使用特殊的格式拼接的
+ data包含hook属性
+ 添加虚拟节点所属的实例，即_parentVM_
+ componentOptions包含Ctor属性（明显就是子组件的构造函数）
+ componentInstance实例引用，暂时为空
parentVM说明：
+ 添加生成的虚拟dom，即_parentVnode_
注意
+ 只有组件类型的虚拟dom data才有hook属性

#### 2.3父组件生成真实dom
现在_父组件_创建真实节点了，在创建child节点时，发现child是一个组件，则重复父组件的3步：

2.3.1子组件生成render函数
子组件的render函数：
```javascript
function anonymous() {
  with(this) {return _('div')}
}
```
此时父子组件实例：
parentVM说明：
+ $children添加了对childVM的引用
  childVM说明：
+ 实例id
+ 暂无虚拟dom
+ $options包含了render属性
+ $parent是对parentVM的引用
+ 无子组件

2.3.2子组件生成新的虚拟dom
父子组件实例，以及各自的虚拟dom的状态：

childVnode说明：

+ tag是div
+ data属性并无hook属性
+ 绑定了childVM实例
+ 无组件选项
+ 无子组件实例
+ parentVnode说明：
+ 添加了对childVM的引用

2.3.3子组件生成真实dom
调用api创建div元素。
因为本例中_父组件_没有其他节点需要创建，所以_子组件_完成后，_父组件_也完成了。

2.3.4小结
+ 父组件实例的虚拟dom不会包含子组件实例的虚拟dom
+ 组件类型的虚拟dom有一些特殊属性（用于实例化子组件），比较重要的有data.hook,componentOptions.Ctor
+ 生成真实dom的过程是深度优先的，即从父组件到子组件

### 3 代码分析
本节从代码角度找出对组件做处理的地方，还是围绕3步走。

#### 3.1父组件生成render函数
编译函数是判断不出字符串模版中的标签是html标签还是组件名称的，所以基本没有特殊逻辑，唯一需要注意的就是对is属性的处理：
```javascript
// src/compiler/parse/index.ts
function processComponent(el: ASTElement) {
  let component = getBindAttr(el, "is");
  if (component) {
    el.component = component;
  }
}
```
简单的说就是得到is值，生成虚拟dom时就可以用作标签了。

#### 3.2生成虚拟dom
此处肯定有对组件类型的虚拟dom有特殊处理（废话嘛），看下代码：
```javascript
// src/core/vnode/create-element.ts
function _createElement(
  context: Vue,
  tag?: string,
  data?: VNodeData,
  children?: any,
  normalizeType?: number
): VNode {
  if ((Ctor = resolveAsset(
      context.$options, 
      “components”,
      tag))
  ) {
    vnode = 
      createComponent(Ctor, data, context, tag);
  } else {
    vnode = 
      new VNode(tag, data, children, 
        undefined, context);
  }
  return vnode;
}
```
_createElement就是创建虚拟dom的具体函数，首先判断当前tag是不是组件名，如果是的话创建组件类型的虚拟dom，反之创建html标签对应的虚拟dom。
判断的方式也很简单，本例中暂时只判断了components选项有没有对应的属性（实际情况会复杂一些，因为vue还有全局组件，内置组件）。
下面看下创建组件类型的虚拟dom：
```javascript
// src/core/vnode/create-component.ts
function createComponent(
  Ctor: typeof Vue | Object,
  data: VNodeData,
  context: Vue,
  tag: string
): VNode {
  const base = context.$options._base;

  if (isPlainObject(Ctor)) {
    Ctor = base.extend(Ctor);
  }
  if (typeof Ctor !== 'function') {
    return;
  }

  resolveConstructorOptions(Ctor);

  data = data || {};

  let propsData = extractPropsForVnode(data, Ctor);

  let listener = data.on;

  // 添加组件类型的虚拟dom特有的钩子函数
  mergeHooks(data);

  return new VNode(
    `vue-component-${Ctor.cid}-${tag}`,
    data, null, null, context,
    {
      Ctor,
      propsData,
      tag
    }
  );
}
```
目的就是将准备好实例化子组件需要的所有数据 保存在虚拟dom对象中，为实例化做准备。需要什么数据呢？不妨思考下创建一个子组件需要哪些数据？构造函数？父组件传来的props？监听事件？

+ 构造函数Ctor：目前我们考虑的是局部注册组件，所以Ctor一开始是对象，需要通过context.$options.extend()转换成Vue的子类。
+ 父组件传来的props propsData：根据当前组件的props对象的key，如果父组件也设置该key，则保存。后续子组件实例化时定义成响应式。
+ 最重要的：mergeHooks函数为data添加了hook属性。

#### 3.3生成真实dom
关注下处理组件类型的虚拟dom的函数：
```javascript
// src/core/vnode/patch.ts
  function createElm(
    vnode: VNode, 
    parentElm: Node, 
    refElm: Node
  ) {
    if (createComponent(vnode, parentElm, refElm)) {
      return;
    } else {
    }
    // 省略
  }
```
如果成功创建了组件类型的虚拟dom，就直接返回了。createComponent函数：
```javascript
// src/core/vnode/patch.ts
function createComponent(
  vnode: VNode, 
  parentElm: Node, 
  refElm: Node
) {
 let i: any;
  if (
    (i = vnode.data) &&
    (i = i.hook) &&
    (i = i.init)
  ) {
    i(vnode, parentElm, refElm);
   if (vnode.componentInstance) {
      initComponent(vnode);
      return true;
    }
  }
}
```
调用了vnode.data.hook.init函数，明显就是实例化子组件嘛，如果成功了，那么componentInstance一定有值了，继而又做了些初始化工作。先看下hook.init函数：
```javascript
// src/core/vnode/create-component.ts
function init (
  vnode: VNode,
  parentElm: HTMLElement,
  refElm: HTMLElement
) {
  if (!vnode.componentInstance || 
      vnode.context._isDestroyed
  ) {
    const child = 
      vnode.componentInstance =
      createComponentInstanceForVnode(
        vnode,
        activeInstance,
        parentElm,
        refElm
      );
    child.$mount(<Element>vnode.elm);
  }
}

function createComponentInstanceForVnode(
  vnode: VNode,
  parent: Vue,
  parentElm: HTMLElement,
  refElm: HTMLElement
): Vue {
  const componentOption = vnode.componentOptions;
  const options = {
    _isComponent: true,
    propsData: componentOption.propsData,
    parent,
    _parentVnode: vnode,
    _parentElm: parentElm,
    _refElm: refElm
  }
  return new componentOption.Ctor(options);
}
```
终于看到了子组件的实例化和挂载，然后子组件重复3步走，如此递归，完成所有节点的创建。

#### 3.4小结
组件类型的虚拟dom比一般的虚拟dom多了创建组件的属性
data.hook比较关键，用于判断，也用于子组件实例化

### 4 删除组件
本节看下生成的子组件删除的处理：
```javascript
function destroy(vnode: VNode) {
  if (!vnode.componentInstance._isDestroyed) {
    vnode.componentInstance.$destroy();
  }
}
```
删除组件类型的虚拟dom就是消除一个实例。

### 5 更新组件
本节只讨论组件类型的虚拟dom修改时对应的处理，因为对于html节点的更新，网上已经有文章了。
这里分成2种情况：

+ 新旧虚拟dom是同一类型的组件
+ 新旧虚拟dom是非同一类型的组件

#### 5.1 新旧虚拟dom是同一类型的组件
直接看处理函数：
```javascript
// src/core/vnode/pathc.ts
function patchVnode(oldVnode: VNode, vnode: VNode) {
  const elm = vnode.elm = oldVnode.elm;

  let i: any;
  if (
    (i = vnode.data) && 
    (i = i.hook) && 
    (i = i.prePatch)) 
  {
    i(oldVnode, vnode);
  }

  if (isDef(vnode.data)) {
    for (let i = 0; i < cbs.update.length; i++) {
      cbs.update[i](oldVnode, vnode);
    }
  }

  // 省略
}
```
此函数处理同一类型的结点，即更新自身及子节点的属性，可以看到，还是data.hook.prePatch对组件类型的虚拟dom做了处理：
```javascript
function prePatch (oldVnode: VNode, vnode: VNode) {
    const options = vnode.componentOptions;
    const child 
      = vnode.componentInstance 
      = oldVnode.componentInstance;
    updateChildComponents(child, options.propsData, vnode);
  },

function updateChildComponents(
  vm: vueInstance,
  propsData: {[key: string]: any},
  parentVnode: VNode
) {
  vm.$vnode = parentVnode;
  vm.$options._parentVnode = parentVnode; // set for hoc
  if(vm._vnode) {
    vm._vnode.parent = parentVnode; // set for patch
  }

  if(propsData && vm.$options.props) {
    observeState.shouldObserve = false;
    for(let key in vm._props) {
      vm._props[key] = propsData[key];
    }
    observeState.shouldObserve = true;
  }
}
```
主要目的还是很明切的：更新子组件的props，因为props的属性和data一样是响应式的，所以可以起到更新子组件的目的。

#### 5.2新旧虚拟dom是非同一类型的组件
这里可以细分成3种情况的：

+ 旧虚拟dom是 空或是html结点，新虚拟dom是 组件dom
+ 旧虚拟dom是 组件dom，新虚拟dom是 空或是html结点
+ 新旧虚拟dom都是组件dom，但并非同一类型
不管是哪种情况，处理方式都是一样的，新的虚拟dom创建新的结点，旧的虚拟dom删除结点。

#### 5.3小结
对于同一虚拟组件dom，则更新props，否则走新建、删除流程。

### 6 对子组件设置class和style
我们知道模版中子组件上设置class、style是可以设置到子组件的根节点上的，这是如何实现的呢？
在createComponent函数（3.3生成真实dom）,调用init生成子实例后，紧接这执行了initComponent函数：
```javascript
function initComponent(vnode: VNode) {
  vnode.elm = vnode.componentInstance.$el;
  invokeCreateHooks(vnode);
}

function invokeCreateHooks(vnode: VNode) {
  for (let i = 0; i < cbs.create.length; i++) {
    cbs.create[i](emptyVnode, vnode);
  }
}
```
可见，父组件先保存了子组件的html结点，然后调用了cbs.create函数：
```javascript
function updateClass(oldVnode: VNode, vnode: VNode) {
  let elm = <Element>vnode.elm;
  let cls = genClassFromVnode(vnode);
  elm.setAttribute("class", cls);
}

function genClassFromVnode(vnode: VNode): string {
  let data: {
    staticClass?: string;
    class?: any;
  } = vnode.data;


  let childVnode = vnode;
  while (isDef(childVnode.componentInstance)) {
    childVnode = vnode.componentInstance._vnode;
    data = mergeClass(childVnode.data, data);
  }

  return renderClass(data.staticClass, data.class);
}

function mergeClass(child: VNodeData, parent: VNodeData): {
  staticClass?: string;
  class?: any;
} {
  return {
    staticClass: concat(child.staticClass, parent.staticClass),
    class: isDef(child.class)
      ? [child.class, parent.class]
      : parent.class
  }
}

function concat(a?: string, b?: string): string {
  return a
    ? b
      ? a + ' ' + b
      : a
    : (b || '')
}
```
这里注意2点：

+ genClassFromVnode会递归出componentInstance上的class的值，并进行合并
+ 不管是静态、还是动态的值的合并都是父组件的值在后

### 7 vue中对hoc的判断

本节看下vue对hoc的判断，还是举个例子：
```javascript
new Vue({
  template: `<child></child>`,
  components: {
    child: {
      data() {return {tag: 'div'}}
      template: `<comp :is=tag></comp>`
    }
  }
}).$mount(); 
```
我们让子组件的tag是动态的，那么如果我将tag修改成span，因为子组件内部是响应式的，会更新结点，但是父组件保存的结点如何更新呢？
```javascript
Vue.prototype._update = function (vnode: VNode) {
  const vm = this;
  if (vm._isMounted) {
    callHook(vm, 'beforeUpdate');
  }
  // patch
  vm.$el = vm.__patch__(prevNode, vnode);

  if (
    vm.$vnode &&
    vm.$parent &&
    vm.$vnode === vm.$parent._vnode
  ) {
    // update parent $el
    vm.$parent.$el = vm.$el;
  }
}
```
在_update函数内部，更新完当前实例的根节点以后，有一个判断，就是自身的$vnode和$parent._vnode是一个引用，则更新父组件的根节点为当前实例的根节点。
$parent._vnode我们知道，就是实例的虚拟dom，当执行_render得到。
$vnode在updateChildComponents（5.1新旧虚拟dom是同一类型的组件）中被设置，即当父实例引起子实例更新时，会设置成父组件的虚拟dom。
所以仅当子组件更新时，vm.$vnode === vm.$parent._vnode是可以成立的。