#### 页面导入样式时，使用link和@import有什么区别？
+ link是HTML标签，@import是css提供的；
+ link引入的样式问题加载时同时加载，@import引入的样式需等页面加载完成后再加载；
+ link没有兼容性问题，@import不兼容ie5以下；
+ link可以通过js操作DOM动态引入样式表改变样式，而@import不可以