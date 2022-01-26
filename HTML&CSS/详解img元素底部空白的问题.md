### 详解img元素底部空白的问题


首先我们要明白CSS对于 display: inline 元素的 `vertical-align` 各个值的含义。

CSS 的属性 vertial-align 用来指定行内元素（inline）或表格单元格（table-cell）元素的。


vertical-align的默认值是 baseline ，这是一个西方排版才有的概念。

![image](./image/vertical-align-1.png)


此外，对比一下vertial-align 的另外两个常见值，top 和 bottom

![image](./image/vertical-align-2.png)


我们可以看到 baseline  和 bottom 之间有一定的距离。实际上， inline 的图片下面那一道空白正是 baseline 和 bottom 之间的距离。即使只有图片没有文字。只要是 inline 的图片这段空白都会存在。


所以要解决图片底部空白的问题有：

1. 将图片的 `vertical-align` 设置为其他值。如果在同一行中有问题混行的话，那应该是用 `bottom` 或是 `middle` 比较好。
2. `vertical-align` 中的 top 和 bottom 之间的值即为 line-height。那么我们把 `line-height` 设置为 0，那么 `baseline` 与 `bottom` 之间的距离也变为 0 。那么空白也就消失了。
3. 如果没有设置 `line-height` , `line-height` 的默认值是基于 `font-size` 的（视渲染引擎有所不同，一般是乘以一个系数如1.2）。因此在没有设置 `line-height` 情况下把 `font-size` 设为 0 也可以达到同样的效果。
4. 将图片的 `display` 改为 `block` 


