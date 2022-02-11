### flex缩写属性

我们在面试过程中，经常会被问到 flex 属性是什么属性的缩写？


**flex 属性是 flex-grow、flex-shrink、flex-basis 三个属性值的缩写**

**flex-grow 属性**

`flex-grow` 属性定义项目的放大比例，默认为 `0`，即如果存在剩余空间，也不放大。

**flex-shrink 属性**

`flex-shrink` 属性定义了项目的缩小比例，默认为 `1`，即如果空间不足，该项目将缩小。

**flex-basis 属性**

`flex-basis` 属性定义了在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为auto，即项目的本来大小。

#### flex 属性的值

flex 属性值可以只指定一个属性的值，而另外的属性值采用各自在 flex 属性中的**初始值**，但是有一点要主要的是： flex 属性中 flex-grow 和 flex-basis 的初始值和他们原先的默认值不同。至于为什么不同，MDN 中有明确的说过 这样设计是为了让 flex 缩写在最常见的情景下好用


#### flex 的不同取值

flex 的值的完整写法是 `[<flex-grow> <flex-shrink> <flex-basis>]`

**flex值为none**

当flex为none时,等同于flex: 0 0 auto;

**flex值为auto**

当flex为auto时，等同于flex: 1 1 auto;

**flex值为一个非负数字**

当flex取值为一个数字，则该数字是设置的flex-grow值，其它两个属性用初始值，如flex:1时，等同于flex: 1 1 0%