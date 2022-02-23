## preload和prefetch的区别

- [preload和prefetch的区别](#preload和prefetch的区别)
  - [preload](#preload)
    - [preload特点](#preload特点)
  - [prefetch](#prefetch)
    - [prefetch 特点](#prefetch-特点)
    - [preload 和 prefetch 的对比](#preload-和-prefetch-的对比)

preload 和 prefetch 的出现为我们提供了可以更加细粒度地控制浏览器加载资源的方法。

### preload

link 标签的 preload 是一种声明式的资源获取请求方式，用于提前加载一些需要的依赖，并且不会影响页面的 onload 事件。

使用方式如下：

```html
<link rel="preload" as="script" href="test.js" onload="handleOnload()" onerror="handlepreloadError()" >
```

- rel 属性为 preload
- as 属性用于规定资源的类型，并根据资源类型设置 Accep 请求头，以便能够使用正常的策略去请求对应的资源
- href 属性为资源请求地址
- onload 和 onerror 则分别是资源加载成功和失败后的回调函数

#### preload特点

1. preload 加载的资源是在浏览器渲染机制之前进行处理的，并且不会阻塞 onload 事件
2. preload 可以支持加载多种类型的资源，并且可以加载跨域资源
3. preload 加载的 js 脚本其加载和执行过程是分离的。即 preload 会预加载相应的脚本代码，待到需要时自行调用


### prefetch

prefetch 是一种利用浏览器的空闲时间加载页面将来可能用到的资源的一种机制。通常可以用于加载非首页的其他页面所需要的资源，以便加快后续页面首屏速度。

#### prefetch 特点

prefetch 可以加载非当前页面的所需要的资源，并且将其放入缓存至少5分钟。当页面跳转时，未完成的 prefetch 请求不会被中断。


#### preload 和 prefetch 的对比

- preload 和 prefetch 都没有同域名的限制
- preload 主要用于预加载当前页面需要的资源，而 prefetch 主要用于加载将来页面所需要的资源
- preload 需要使用 as 属性指定特定的资源类型以便浏览器为其分配一定的优先级，并能够正确加载资源


