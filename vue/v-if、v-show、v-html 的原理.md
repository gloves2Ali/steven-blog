#### v-if、v-show、v-html 的原理
+ v-if会调用addIfCondition方法，生成vnode的时候会忽略对应节点，render的时候就不会渲染；

+ v-show会生成vnode，render的时候也会渲染成真实节点，只是在render过程中会在节点的属性中修改show属性值，也就是常说的display；

+ v-html会先移除节点下的所有节点，调用html方法，通过addProp添加innerHTML属性，归根结底还是设置innerHTML为v-html的值