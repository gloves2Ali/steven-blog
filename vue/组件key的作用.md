#### Vue 项目时为什么要在组件中写 key，其作用是什么?

**vue和react都是采用diff算法来对比新旧虚拟节点，从而更新节点。key的作用是为了在diff算法执行时更快的找到对应的节点，提高diff速度。**

**设置key和不设置key的区别**
不设key，newCh和oldCh只会进行头尾两端的相互比较，设key后，除了头尾两端的比较外，还会从用key生成的对象oldKeyToIdx中查找匹配的节点，所以为节点设置key可以更高效的利用dom。

**深度解析**
key是给每一个vnode的唯一id，可以依靠key，更准确，更快的拿到oldVnode中对应的vnode节点
1. 更准确：
因为带key就不是就地复用了，在sameNode函数 a.key === b.key对比中可以避免就地复用的情况。所以会更加准确。
2. 更快：
利用key的唯一性生成map对象来获取对应节点，比遍历方式更快。