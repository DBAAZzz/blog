### requestAnimationFrame

- [requestAnimationFrame](#requestanimationframe)
  - [理解什么是requestAnimationFrame](#理解什么是requestanimationframe)
  - [event loop与requestAnimationFrame的关系](#event-loop与requestanimationframe的关系)

#### 理解什么是requestAnimationFrame

`requestAnimationFrame` 是HTML5中的一个API，类似于 `setTimeout` 定时器。
`window.requestAnimationFrame()` 告诉浏览器——你希望执行一个动画，并且要求浏览器在下次重回之前调用指定的回调函数更新动画 。该方法需要传入一个回调函数作为参数，该回调函数会在浏览器下一次重绘之前执行。

    若想在浏览器下次重绘之前继续更新下一帧动画，那么回调函数自身必须再次调用 window.requestAnimationFrame() 



语法：

```js
window.requestAnimationFrame(callback)
```


实际例子：

```js
var progress = 0; //回调函数 
function render() { 
    progress += 1; 
    //修改图像的位置 
    if (progress < 100) { 
        //在动画没有结束前，递归渲染 
        window.requestAnimationFrame(render);
    } 
 } 
//第一帧渲染 
window.requestAnimationFrame(render);
```


**使用requestAnimationFrame的优点：**

- **CPU节能**
  
  在使用 `setTimeout` 实现的动画，当页面被隐藏或最小化时，`setTimeout` 仍然在后台执行动画任务，由于此时页面处于不可见或不可用状态 ，刷新页面动画时没有意义的，浪费了CPU资源。而 `requestAnimationFrame` 则完全不同，当页面处于未激活状态下，该页面的屏幕刷新任务也会被系统暂停，因此跟着系统的步伐走的 `requestAnimationFrame` 也会停止渲染，当页面再次别激活时，动画就从上次停留的地方继续执行，有效的节省了CPU的开销。
  

- **函数节流**
  
  在高频率事件（resize、scroll等）中，为了防止在一个刷新间隔内发生多次函数执行，使用 `requestAnimationFrame` 可保证每个刷新间隔内，函数只被执行一次，这样既能保证流畅性，也能更好的节省函数执行的开销。一个刷新间隔内函数执行多次时没有意义的，因此显示器每16.7ms（1000/60hz）刷新一次，多次绘制并不会在屏幕上提现出来。


#### event loop与requestAnimationFrame的关系

浏览器中有个渲染时机的问题，也就是浏览器会根据当前的浏览上下文判断是否进行渲染，它会尽量高效，只有必要的时候才进行渲染，如果没有界面的改变就不会渲染。
按照规范里说的一样，因为考虑到硬件的刷新频率限制、页面性能以及页面是否存在后台等因素，有可能执行完 setTimeout 这个 task 之后，发现还没有到渲染时机，所以 setTimeout回调了几次之后才进行渲染。

`requestAnimationFrame` 帧动画不同之处在于，每次渲染之前都会调用。