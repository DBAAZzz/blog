# 分享一些比较少见但有用的CSS属性


- [分享一些比较少见但有用的CSS属性](#分享一些比较少见但有用的css属性)
  - [will-change](#will-change)
    - [浏览器页面展示过程](#浏览器页面展示过程)
    - [浏览器渲染原理](#浏览器渲染原理)
    - [Graphics Layer](#graphics-layer)
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

### 浏览器页面展示过程

首先，我们需要了解一下浏览器的页面展示过程

![图 1](images/4a1b505d17f75478a7bcf243a7bbd5281092868ed4fd94c4acac760ad3175a8e.png)  


- Javascript：主要负责业务交互逻辑。
- Style: 根据 CSS 选择器，对每个 DOM 元素匹配对应的 CSS 样式。
- Layout: 具体计算 DOM 元素显示在屏幕上的大小及位置。
- Paint: 实现一个 DOM 元素的可视效果(颜色、边框、阴影等)，一般来说由多个渲染层完成。
- Composite: 当每个层绘制完成后，浏览器会将所有层按照合理顺序合并为一个图层，显示到屏幕。 本文我们将重点关注 Composite 过程。


### 浏览器渲染原理

在讨论 **Composite** 之前，我们还需要了解一下浏览器渲染原理

![图 2](images/b783651500f09dc3cffe66ebbbc6e117da9ba1acd66d42db4a5a551130d4daa2.png)  


- **DOM** 元素与 **Layout Object** 存在一一对应的关系
- 一般来说，拥有相同坐标空间的 **Layout Object** 属于同一个 **Paint Layer** (渲染层)，通过 **position、opacity、filter** 等 CSS 属性可以创建新的 **Paint Layer**
- 某些特殊的 **Paint Layer** 会被认为是 **Composite Layer** (合成层/复合层)。Composite Layer 拥有单独的 Graphics Layer (图形层)，而那些非 Composite Layer 的 Paint Layer，会与拥有 Graphics Layer 的父层共用一个。
  

### Graphics Layer

我们日常生活中所看到屏幕可视效果可以理解为：由多个位图通过 GPU 合成渲染到屏幕上，而位图的最小单位是像素。如下图：

![图 3](images/eacec48c138379b4d013d1b11cce59d77dc713abbc3cacb28c1944f59b980bcc.png)  


那么位图是怎么获得的呢，**Graphics Layer** 便起到了关键作用,每个 **Graphics Layer** 都有一个 **Graphics Context**, 位图是存储在共享内存中，**Graphics Context** 会负责将位图作为纹理上传到GPU中，再由GPU进行合成渲染。

![图 4](images/4bc445a14ab229b13ec7715d4da2d5559b77c54195ebb033583ec35143cbcb62.png)  


### CSS在浏览器渲染层面承担了怎样的角色

大多数人对于CSS3的第一印象，就是可以通过 **3D（如transform）** 属性来开启硬件加速，许多同学在重构某一个项目时，考虑到动画性能问题，都会倾向将 2D 属性改为 3D 属性，但开启硬件加速的底层原理其实就在于将 **Paint Layer** 提升到了 **Composite Layer**


### 渲染层提升为合成层的方式

- 3D 属性开启硬件加速（3d-transform）
- will-change：（opacity、transform、top、left、bottom、right），其中 top、bottom 等属性需要设置明确的定位属性，如 relative 
- 使用 fixed 或 sticky 定位
- 对 opacity、transform、fitler 应用了 animation（actived）或者 transform（actived），animation 和 transform 需要是激活状态的才行


### 提升合成层的优缺点

#### 优点

- 合成层的位图，会交给 GPU 合成，比 CPU 处理要快
- 当需要 repaint 时，只需要 repaint 本身，不会影响到其他的层，使页面更加流畅
- 对于 transform 和 opacity 效果，部分浏览器不会触发 Layout 和 Paint

#### 缺点

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

## caret-color

caret-color 可以设置插入符号的颜色，即下一个字符的可见标记，也称为文本输入光标。
插入符号出现在诸如 input 或者具有 contenteditable 属性的元素中。


## scroll-behavior

当用户手动导航或者 CSSOM scrolling API 触发滚动操作时，CSS 属性 scroll-behavior 为滚动框执行滚动行为。
那些由于用户行为而产生的的滚动，不受这个属性的影响。


## :target

:target 伪类可以让我们不使用任何 JavaScript 代码，只使用 :target 伪类创建一个加亮框。该技术依赖于初始化时就隐藏在页面中的链接到指定元素的锚。一旦定位，CSS 就会更改其 display 以便显示他们。

## :focus-within

:focus-within 是一个 CSS 伪类，表示一个元素获得焦点或者该元素的后代获得焦点。换句话说，元素本身或者它的后代匹配 :focus 伪类。

该选择器非常实用。举个通俗的例子：表单中的某个 **input** 字段获得焦点时，整个表单的 **form** 元素都可被高亮。

## :placeholder-shown

:placeholder-shown CSS 伪类 在 input 或 textarea 元素显示 placeholder text 时生效


## @supports

@support 由一组样式声明和一条支持条件构成。
允许在使用之前检查浏览器是否支持特定属性或属性（属性/值组合）

## attr()

attr() 理论上能用于所有的 CSS 属性但目前支持的仅有伪元素的 content 属性，其他的属性和高级特性都是实验性的。



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



## aspect-ratio               

https://developer.mozilla.org/zh-CN/docs/Web/CSS/aspect-ratio
https://www.smashingmagazine.com/2019/03/aspect-ratio-unit-css/

## contain

https://developer.mozilla.org/zh-CN/docs/Web/CSS/contain

https://web.dev/quintoandar/
https://www.smashingmagazine.com/2020/03/setting-height-width-images-important-again/



