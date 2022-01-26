### BFC

块级格式化上下文（BFC）将根据块布局规则布局子元素。


#### 如何创建新的块级格式化上下文

`<html>` 元素不是唯一能够创建块级格式化上下文的元素。默认为块布局的任何元素也会为其后代元素创建块级格式化上下文。还有一些 CSS 属性可以使元素创建一个 BFC 

- 使用 `float` 使其浮动的元素
- 绝对定位的元素 （`position: fixed` 或 `position: sticky`）
- 使用 `display: inline-block` 的元素
- 使用 `display: flow-root` 的元素
- `块级元素`的 overflow 属性不为 `visible`
- 元素属性为 `contain: layout, content, strict`


使用 overflow 创建的 BFC 的问题在于， overflow 属性用于告诉浏览器您希望如何处理溢出的内容。在某些情况下，但您纯粹使用此属性创建 BFC 时，您会发现不需要的滚动条或剪切阴影。

##### 而使用 display: flow-root 显示创建BFC就不会有任何问题。