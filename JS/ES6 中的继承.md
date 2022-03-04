## ES6 中的继承

- [ES6 中的继承](#es6-中的继承)
  - [ES6 中继承的介绍](#es6-中继承的介绍)
  - [super 关键字](#super-关键字)
    - [作为函数](#作为函数)
    - [作为对象](#作为对象)
  - [类的 prototype 属性和__proto__属性](#类的-prototype-属性和__proto__属性)
  - [参考文章](#参考文章)

### ES6 中继承的介绍

Class 可以通过extends关键字实现继承，让子类继承父类的属性和方法。extends 的写法比 ES5 的原型链继承，要清晰和方便很多。

```js
class Person {
  constructor(name, age) {
    this.namge = name
    this.age = age
  }
}

class Man extends Person {
  constructor(name, age) {
    super(name, age)
    this.sex = 'man'
  }
}
```

ES6 规定， 子类必须在 constructor() 方法中调用 super()，否则就会报错。**这是因为子类自己的 this 对象，必须先通过父类的构造函数完成塑造**，得到与父类同样的实例属性和方法，然后再对其进行加工，添加子类自己的实例属性和方法。**如果不调用 super() 方法，子类就得不到自己的 this 对象**。

为什么子类的构造函数，一定要调用 super() ？原因就在于 ES6 的继承机制，与 ES5 完全不同。

ES5 的继承机制，是先创造一个独立的子类的实例对象，然后再将父类的方法添加到这个对象上面，即“实例在前，继承在后”。

ES6 的继承机制，则是先将父类的属性和方法，加到一个空的对象上面，然后再将该对象作为子类的实例，即“继承在前，实例在后”。这就是为什么 ES6 的继承必须先调用 super() 方法，因为这一步会生成一个继承父类的 this 对象，没有这一步就无法继承父类。

这意味着新建子类实例时，父类的构造函数必定会先运行一次。

```js
class Person {
  constructor(name, age) {
    this.namge = name
    this.age = age
    console.log(1)
  }
}

class Man extends Person {
  constructor(name, age) {
    super(name, age)
    this.sex = 'man'
    console.log(2)
  }
}

let man1 = new Man('xiaoming', 18)
```

上面的示例中，子类 Man 创建一个实例时，会输出 1 和 2。原因就是子类调用构造函数调用 `super()` 时，会执行一次父类构造函数。

如果子类没有定义constructor()方法，这个方法会默认添加，并且里面会调用super()。也就是说，不管有没有显式定义，任何一个子类都有constructor()方法。

**除了私有属性，父类的所有属性和方法，都会被子类继承，其中包括静态方法。**

子类无法继承父类的私有属性，或者说，私有属性只能在定义它的 class 里面使用。


总结了几个关键点是需要注意的：

1. 在子类的构造函数中，只有调用 super() 之后，才可以使用 this 关键字，否则会报错。
2. 除了私有属性，父类的所有属性和方法，都会被子类继承，其中包括静态方法。
3. 子类无法继承父类的私有属性


### super 关键字

#### 作为函数

super这个关键字，既可以当作函数使用，也可以当作对象使用。在这两种情况下，它的用法完全不同。

第一种情况，super作为函数调用时，代表父类的构造函数。ES6 要求，子类的构造函数必须执行一次super函数。

```js
class A {}

class B extends A {
  constructor() {
    super();
  }
}
```

注意，super 虽然代表了父类A的构造函数，但是返回的是子类B的实例，即 super 内部的 this 指的是B的实例，因此 super() 在这里相当于 `A.prototype.constructor.call(this)`。

```js
class A {
  constructor() {
    console.log(new.target.name);
  }
}
class B extends A {
  constructor() {
    super();
  }
}
new A() // A
new B() // B
```

**作为函数时**，super() 只能用在子类的构造函数之中，用在其他地方就会报错。



#### 作为对象

第二种情况，super 作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类。

```js
class A {
  p() {
    return 2;
  }
}

class B extends A {
  constructor() {
    super();
    console.log(super.p()); // 2
  }
}

let b = new B();
```

上面代码中，子类B当中的super.p()，就是将super当作一个对象使用。这时，super在普通方法之中，指向A.prototype，所以super.p()就相当于A.prototype.p()。

**这里需要注意，由于 super 指向父类的原型对象，所以定义在父类实例上的方法或属性，是无法通过super调用的**。

```js
class A {
  constructor() {
    this.p = 2;
  }
}

class B extends A {
  get m() {
    return super.p;
  }
}

let b = new B();
b.m // undefined
```

上面代码中，p 是父类A实例的属性，super.p 就引用不到它。

如果属性定义在父类的原型对象上，super就可以取到。

```js
class A {}
A.prototype.x = 2;

class B extends A {
  constructor() {
    super();
    console.log(super.x) // 2
  }
}

let b = new B();
```

**ES6 规定，在子类普通方法中通过 super 调用父类的方法时，方法内部的this指向当前的子类实例**。

```js
class A {
  constructor() {
    this.x = 1;
  }
  print() {
    console.log(this.x);
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
  }
  m() {
    super.print();
  }
}

let b = new B();
b.m() // 2
```

上面代码中，`super.print() `虽然调用的是 `A.prototype.print()`，但是 `A.prototype.print()` 内部的 `this` 指向子类 B 的实例，导致输出的是 2，而不是 1。也就是说，实际上执行的是 `super.print.call(this)`。

**由于 this 指向子类实例，所以如果通过 super 对某个属性赋值，这时 super 就是 this，赋值的属性会变成子类实例的属性**。

```js
class A {
  constructor() {
    this.x = 1;
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
    super.x = 3;
    console.log(super.x); // undefined
    console.log(this.x); // 3
  }
}

let b = new B();
```

最后，由于对象总是继承其他对象的，所以可以在任意一个对象中，使用super关键字。

```js
var obj = {
  toString() {
    return "MyObject: " + super.toString();
  }
};

obj.toString(); // MyObject: [object Object]
```

### 类的 prototype 属性和__proto__属性

大多数浏览器的 ES5 实现之中，每一个对象都有__proto__属性，指向对应的构造函数的prototype属性。
Class 作为构造函数的语法糖，同时有prototype属性和__proto__属性，因此同时存在两条继承链。

**1. 子类的__proto__属性，表示构造函数的继承，总是指向父类**。
**2. 子类prototype属性的__proto__属性，表示方法的继承，总是指向父类的prototype属性**。

```js
class A {
}

class B extends A {
}

B.__proto__ === A // true
B.prototype.__proto__ === A.prototype // true
```

上面代码中，子类B的__proto__属性指向父类A，子类B的prototype属性的__proto__属性指向父类A的prototype属性。

这样的结果是因为，类的继承是按照下面的模式实现的。

```js
class A {
}

class B {
}

// B 的实例继承 A 的实例
Object.setPrototypeOf(B.prototype, A.prototype);

// B 继承 A 的静态属性
Object.setPrototypeOf(B, A);

const b = new B();
```


等同于

```js
Object.setPrototypeOf(B.prototype, A.prototype);
// 等同于
B.prototype.__proto__ = A.prototype;

Object.setPrototypeOf(B, A);
// 等同于
B.__proto__ = A;
```


### 参考文章

[xx](https://wangdoc.com/es6/class-extends.html)


