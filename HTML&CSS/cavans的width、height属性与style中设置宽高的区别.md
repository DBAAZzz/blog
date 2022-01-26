### canvas 的 width、height 属性与在 style 中设置宽高的区别

canvas 看起来和 img 标签一样，只是 canvas 只有两个可选的属性 width、height 属性，而没有 src、alt 属性。

canvas 是一个**画板**（通过 style 设置）和一张**画纸**（通过 canvas 属性设置），画板相当于一个容器。

画板和画纸的默认宽高都是 `300*150` ，当画板与画纸宽高相等时，图像就不会被拉伸，当画纸与画板宽高不一样时，图像就会被拉伸。

canvas 的 width、height 属性是设置**画纸的宽高**

style 样式里设定的仅仅是**画板的宽高**，如果不设置 width、height 属性，画纸的宽高还是为默认的 300*150

---

**【重点！】**

- 要设置 canvas 的画纸大小，使用的是 canvas.width 和 canvas.height
- 要设置实际渲染大小，使用的是 style 中的 width 和 height


