## 原型链中 Function、Object、Null 的关系


我们知道原型链的尽头是指向 null，那么 `Function.prototype`、`Object.prototype`、`null`、`Function.prototype.__proto__`、`Object.prototype.__proto__`、`function`、`object` 之间的关系是什么？

### Object

Object 构造函数创建一个对象包装器。JavaScript中的所有对象都来自 Object；所有对象从Object.prototype继承方法和属性，尽管它们可能被覆盖。

**Object 作为构造函数时，其 `[[Prototype]]` 内部属性值指向 Function.prototype**


```js
Object.__proto__ === Function.prototype; // true
```


#### Object.prototype

Object.prototype 表示 Object 的原型对象，其 `[[Prototype]]` 属性是 null，访问器属性 __proto__ 暴露了一个对象的内部 `[[Prototype]]` 。Object.prototype 浏览器底层根据 ECMAScript 规范创造的一个对象。


#### object

**通过字面量实例化一个object，它的__proto__指向Object.prototype。**

**而通过new Object实例化一个object，它的__proto__指向Object.prototype。**

```js
var obj = {};
obj.__proto__ === Object.prototype; // true
```

```js
var obj = new Object;
obj.__proto__ === Object.prototype; // true
```

### Function


#### Function.prototype

Function 构造函数创建一个新的 Function 对象。在 JavaScript 中每个函数实际上都是一个 Function 对象。

全局的 Function 对象没有自己的属性和方法, **但是因为它本身也是函数**，所以它也会通过原型链从 Function.prototype 上继承部分属性和方法。Function.prototype 也是一个“函数对象“，其 `[[prototype]]` 内部属性值指向 Object.prototype

`Function.prototype` 是引擎创建出来的函数，引擎认为不需要给这个函数对象添加 prototype 属性，不然 `Function.prototype.prototype…` 将无休无止并且没有存在的意义。

`Function.prototype.__proto__` 指向 `Object.prototype`。

```js
Function.prototype.__proto__ === Object.prototype; // true
```

#### `Function.__proto__`

Function 构造函数是一个函数对象，其 `[[Class]]` 属性是 Function。Function 的 `[[Prototype]]` 属性指向了 Function.prototype，即

```js
Function.__proto__ === Function.prototype; // true
```

### 解答

```js
Function.__proto__ === Function.prototype; // true
```

```js
Function instanceof Object; // true
Object instanceof Function; // true
Object instanceof Object; // true
Function instanceof Function; // true
```

