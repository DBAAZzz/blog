# 分享一些比较少见但有用的CSS属性


- [分享一些比较少见但有用的CSS属性](#分享一些比较少见但有用的css属性)
  - [will-change](#will-change)
  - [浏览器交互渲染过程](#浏览器交互渲染过程)
  - [浏览器渲染原理](#浏览器渲染原理)
  - [节点（Nodes）和渲染对象（Render Object）](#节点nodes和渲染对象render-object)
  - [渲染对象（RenderObjects）和渲染图层（RenderLayers）](#渲染对象renderobjects和渲染图层renderlayers)
  - [RenderLayers 与 GraphicsLayers 的关系](#renderlayers-与-graphicslayers-的关系)
  - [CSS在浏览器渲染层面承担了怎样的角色](#css在浏览器渲染层面承担了怎样的角色)
  - [渲染层提升为合成层的方式](#渲染层提升为合成层的方式)
  - [提升合成层的优缺点](#提升合成层的优缺点)
    - [优点](#优点)
    - [缺点](#缺点)
  - [display 的几种值](#display-的几种值)
    - [flow-root](#flow-root)
    - [table-cell](#table-cell)
  - [caret-color](#caret-color)
  - [scroll-behavior](#scroll-behavior)
  - [:target](#target)
  - [:focus-within](#focus-within)
  - [:placeholder-shown](#placeholder-shown)
  - [@supports](#supports)
  - [attr()](#attr)
  - [aspect-ratio](#aspect-ratio)
  - [contain](#contain)

## will-change

MDN 上的解释是：

CSS 属性 will-change 为 web 开发者提供了一种告知浏览器该元素会有哪些变化的方法，这样浏览器可以在元素属性真正发生变化之前提前做好对应的优化准备工作。
这种优化可以将一部分复杂的计算工作提前准备好，使页面的反应更为快速灵敏。

## 浏览器交互渲染过程

首先，我们需要了解一下浏览器的页面展示过程

![图 1](images/4a1b505d17f75478a7bcf243a7bbd5281092868ed4fd94c4acac760ad3175a8e.png)  


- Javascript：主要负责业务交互逻辑。
- Style: 根据 CSS 选择器，对每个 DOM 元素匹配对应的 CSS 样式。
- Layout: 具体计算 DOM 元素显示在屏幕上的大小及位置。
- Paint: 实现一个 DOM 元素的可视效果(颜色、边框、阴影等)，一般来说由多个渲染层完成。
- Composite: 当每个层绘制完成后，浏览器会将所有层按照合理顺序合并为一个图层，显示到屏幕。 本文我们将重点关注 Composite 过程。


## 浏览器渲染原理

在讨论 **Composite** 之前，我们还需要了解一下浏览器渲染原理

![图 3](images/e2927f57921964db94367388ade8ef8b8a965503e09c26434012c33c7df4f5fd.png)  

![图 4](images/da3012a3b2482e570c136543d652382e081c5324bc28b9d230aa8c8b1e02bb73.png)  


## 节点（Nodes）和渲染对象（Render Object）

DOM 树中每个可以显示的节点（Node），都对应一个渲染对象（Render Object），渲染对象被存储在一个有层级的树形结构中，叫做渲染对象树（Render Tree）。

某些情况下 Webkit 需要建立匿名的 RenderObject 节点，这样的节点不对应于 DOM 树中的任何节点，而是 Webkit 处理上的需要，典型的例子就是匿名用于表示块元素的 RenderBlock 节点。

**DOM** 元素与 **Layout Object** 不是一一对应的关系


## 渲染对象（RenderObjects）和渲染图层（RenderLayers）

每个渲染对象（RenderObject）都由其对应的渲染图层（RenderLayer），直接关联或者通过其根对象（其根级 RenderObject）间接关联。

**渲染图层（RenerLayer）和渲染对象（RenderObject）不是一一对应的关系，而是一对多的关系。**

一般来说，拥有相同坐标空间的渲染对象会属于同一个渲染图层。渲染图层可以保证渲染对象在重叠、半透明等情况时， 能以正确层叠次序进行绘制。

某些特殊的 **RenerLayer** 会被认为是 **GraphicsLayers** (合成层/复合层)。Composite Layer 拥有单独的 Graphics Layer (图形层)，而那些非 Composite Layer 的 Paint Layer，会与拥有 Graphics Layer 的父层共用一个。

![图 1](images/b782c65610f4dd75f8b8a5b9d92c86d8c825455e8f146111d85bb81f2fecb5f1.png)  


## RenderLayers 与 GraphicsLayers 的关系

每个 GraphicsLayer 都有一个 GraphicsContext，用于为该 GraphicsLayer 开辟一段位图，也就意味着每个 GraphicsLayer 都拥有一个独立的位图，GraphicsLayer 负责将自己的 RenderLayer 及其包含的 RenderObject 绘制到位图里，位图是存储在共享内存中，作为纹理上传到 GPU 中，最后由 GPU 将多个位图进行合成，然后画到屏幕上。


![图 4](images/4bc445a14ab229b13ec7715d4da2d5559b77c54195ebb033583ec35143cbcb62.png) 


## CSS在浏览器渲染层面承担了怎样的角色

大多数人对于CSS3的第一印象，就是可以通过 **3D（如transform）** 属性来开启硬件加速，许多同学在重构某一个项目时，考虑到动画性能问题，都会倾向将 2D 属性改为 3D 属性，但开启硬件加速的底层原理其实就在于将 **RenerLayer** 提升到了 **GraphicsLayers**


## 渲染层提升为合成层的方式

- 3D 属性开启硬件加速（3d-transform）
- will-change：（opacity、transform、top、left、bottom、right），其中 top、bottom 等属性需要设置明确的定位属性，如 relative 
- 使用 fixed 或 sticky 定位
- 对 opacity、transform、fitler 应用了 animation（actived）或者 transform（actived），animation 和 transform 需要是激活状态的才行


## 提升合成层的优缺点

### 优点

- 合成层的位图，会交给 GPU 合成，比 CPU 处理要快
- 当需要 repaint 时，只需要 repaint 本身，不会影响到其他的层，使页面更加流畅
- 对于 transform 和 opacity 效果，部分浏览器不会触发 Layout 和 Paint

### 缺点

- 创建一个新的合成层并不是免费的，它得消耗额外的内存和管理资源
- 纹理上传后交由 GPU 处理，因此我们还需要考虑 CPU 和 GPU 之间的带宽问题，以及有多大内存供 GPU 处理这些纹理的问题


## display 的几种值

### flow-root

可以让元素拥有块级作用域，也就是 BFC。

兼容性：

[caniuse](https://caniuse.com/flow-root)

![图 1](images/ebef9d8eeb0cc6b3ffeda303fa554d7a57258aa15bea946f623dab8f2e3aac7c.png)  



### table-cell

可以给出类似于 table 标记

![图 10](images/7d84d08f06617b6e095bea6a5225b22b3e1bc5fc28bf452b0a605ee182070c19.png)  

## caret-color

caret-color 可以设置插入符号的颜色，即下一个字符的可见标记，也称为文本输入光标。
插入符号出现在诸如 input 或者具有 contenteditable 属性的元素中。

![图 9](images/c42674a57852b91864176385800d02b3b85d3b1ac1399da577aeb28102521282.png)  

## scroll-behavior

当用户手动导航或者 CSSOM scrolling API 触发滚动操作时，CSS 属性 scroll-behavior 为滚动框执行滚动行为。
那些由于用户行为而产生的的滚动，不受这个属性的影响。

![图 8](images/2bdb675284ce5d9869c48121173ccff0293bc061f382cdce6a76ec4d1dccee96.png)  



## :target

:target 伪类可以让我们不使用任何 JavaScript 代码，只使用 :target 伪类创建一个弹窗。该技术依赖于初始化时就隐藏在页面中的链接到指定元素的锚。一旦定位，CSS 就会更改其 display 以便显示他们。

![图 7](images/e5053e0b714c0b9cda1909a1b8df95c83f0a3bdfbb08a85c7d0f80118007a876.png)  


## :focus-within

:focus-within 是一个 CSS 伪类，表示一个元素获得焦点或者该元素的后代获得焦点。换句话说，元素本身或者它的后代匹配 :focus 伪类。

该选择器非常实用。举个通俗的例子：表单中的某个 **input** 字段获得焦点时，整个表单的 **form** 元素都可被高亮。

![图 6](images/00185925313f1d7bdae2efb054966dd59630ca45b51f1a9fd4597f4e121b5c69.png)  


## :placeholder-shown

:placeholder-shown CSS 伪类 在 input 或 textarea 元素显示 placeholder text 时生效

![图 5](images/82d1499045873e632f5f170cabc93c0a2b06f823d26b5e0f523db7220769c7bc.png)  

## @supports

@support 由一组样式声明和一条支持条件构成。
允许在使用之前检查浏览器是否支持特定属性或属性（属性/值组合）

```css
/* check support for `grid` and `image-rendering` properties */
@supports (display: grid) {
  section {
    display: grid;
  }
}

@supports (image-rendering) {
  img {
    image-rendering: pixelated;
  }
}
```

![图 11](images/efc57e89cd053a2e53490f6b7ea6ca60ccc8462e6244cde6c9c8ec167166e2f7.png)  


## attr()

attr() 理论上能用于所有的 CSS 属性但目前支持的仅有伪元素的 content 属性，其他的属性和高级特性都是实验性的。

![图 12](images/5f24e2410f66cc25f060c4a3cc7d9b6ec60e5318d20f6503e2c3871a9fae70fa.png)  


## aspect-ratio    

https://developer.mozilla.org/zh-CN/docs/Web/CSS/aspect-ratio
https://www.smashingmagazine.com/2019/03/aspect-ratio-unit-css/

## contain

https://developer.mozilla.org/zh-CN/docs/Web/CSS/contain

https://web.dev/quintoandar/
https://www.smashingmagazine.com/2020/03/setting-height-width-images-important-again/



