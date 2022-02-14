### BFC&FFC

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

#### flex 布局下 margin: auto 水平垂直居中的理解

在传统的 `display: block `（BFC）下，margin: auto 只能在水平方向上居中，而不能在垂直方向上居中。

查看 CSS 文档可以知道，在块级格式化上下文中，如果 `margin-left` 和 `margin-right` 都是 auto ，则它们的`表达值`相等，从而导致元素的居中。（这里的计算值为元素剩余可用剩余空间的一半）

而如果 margin-top 和 margin-bottom 都是 auto ，则它们的值都是为 0 ，就无法实现垂直方向上的居中。


而在 FFC （flex formatting context）下可以通过 margin: auto 的原因是：

**`在 flex 格式化上下文中，设置了 margin: auto 的元素，在通过 justify-content 和 align-self 进行对齐之前，任何处于空闲的空间都会被分配到该方向中的自动 margin 中去。`**

这里，很重要的一点是，margin: auto 的生效不仅是水平方向，垂直方向也会自动去分配到这个剩余空间。

**重点！！！**

任意方向上的可用空间分配给了该方向的自动 margin，则对齐属性（justify-content/align-self）在该方向上不起作用，因为 margin 将在排布后窃取该方向剩余的所有可用空间。


也就是说，**使用了自动 margin 的 flex 子项目，那么它们父元素设置的 justify-content 和子项目本身的 align-self 属性就不再生效**。margin: auto 的优先级会比较高

