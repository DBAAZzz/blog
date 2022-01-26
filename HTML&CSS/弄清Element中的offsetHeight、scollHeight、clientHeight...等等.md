### 弄清Element中的offsetHeight、scollHeight、clientHeight...等等


#### 相关的属性

offsetWidth / offsetHeight
clientWidth / clientHeight
scrollWidth / scrollHeight
offsetTop / scrollTop

#### offsetXXX属性

`offsetWidth / offsetHeight` 是指一个元素的CSS【标准宽高】，它包括了边框、内边距、元素内容以及滚动条（如果存在的话）。
借助MDN上的图片可以看到元素的 `offsetWidth` ，这就是这个元素在页面中【实际占有的空间】，同时当元素的形状发生变化时（放大、缩小） `offsetWidth` 就代表元素的布局宽高，并不代表实际渲染出来的宽高，如需获取渲染出来的宽高请使用 `getBoundingClientRect()` 方法来计算得到真正的 `offsetWidth `。


#### clientXXX属性

`clientWidth / clientHeight` 就表示一个元素内容区域的实际大小，包括元素内容以及内边距padding。
跟上面的 `offsetWidth / offsetHeight` 相比，不包括边框和滚动条（如果存在的话）


#### scrollXXX属性

`scrollHeight / scrollWidth` 表示一个元素内容区域的实际大小，包括不在页面中的可滚动部分（内容和内边距）
跟上面的 `clientHeight/clientWidth` 相比，是整个内容的大小


#### xxxTop属性

`offsetTop` 是当前元素顶部距离最近父元素顶部的距离，和有没有滚动条没有关系
`scrollTop` 在有滚动条的情况下，为元素可视区域距离远离顶部的像素，也就是已经滚动了多少距离

#### 6、其他属性

`window.screen.availHeight`  浏览器窗口可用的高度
`window.innerHeight` 获取浏览器视觉视口高度（包括垂直滚动条）