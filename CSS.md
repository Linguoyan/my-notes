# CSS属性

### text-transform

控制文本大小写

- capitalize：无论字母是大写还是小写，全部都转为首字母大写
- uppercase：无论字母是大写还是小写，全部都转为字母全部大写
- lowercase：无论字母是大写还是小写，全部被CSS转为字母全部小写



## 伪元素

通过在选择器的末尾添加伪元素关键词，来修改所选择元素的特定部分的样式。

一个选择器只能使用一个伪元素

伪元素使用 `::`，伪类使用 `:`，要区分开来



`::after`

创建一个伪元素，作为匹配元素的最后一个子元素，通常配合 `content` 属性来为作为该元素的内容，默认是行内元素



`::before`

创建一个伪元素，作为匹配元素的第一个子元素



`::marker`

作用在设置了 `display: list-item` 的元素或伪元素上，如 `<li>` 标签，小部分样式可用



`::first-letter`

选中某个块级元素第一行的第一个字母，对其设置样式，部分样式可用



`::first-line`

选中某个块级元素的第一行内容，对其设置样式，部分样式可用



`::placeholder`

表单元素占位文本的样式，即 `placeholder` 指定内容的样式，仅有一小部分样式可用



`::selection`

对文档中被用户选择文本的样式进行设置



## 伪类



伪类是添加到选择器的关键字，作用是对指定选择的元素的特殊状态进行样式设置



`:active`

匹配被用户激活的元素，使用鼠标交互时，它代表的是用户按下按键和松开按键之间的时间

一般用在 `<a>` 和 `<button>`



`:any-link`

匹配每一个带有 `href` 属性的 `<a>` `<area>` `<link>` 元素，因此会匹配所有的 `:link` `:visited`



 