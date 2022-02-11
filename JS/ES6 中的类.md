### ES6 中的类

ECMAScript2015 中引入的 JavaScript 类实质上是 JavaScript 现有的基于原型的继承的语法糖。类实际上是一个“特殊的函数”，就像你能够定义的函数表达式和函数声明一样，类声明有两个组成部分：类表达式和类声明

#### 类声明

函数声明和类声明之间的一个重要区别是**函数声明会提升**，类声明不会。

```js
class Person {
  constructor(name) {
    this.name = name;
  }

  // 等价于 Person.prototype.sayName
  sayName() {
    console.log(this.name)
  }
}
```

类声明与自定义类型之间的差异：

1. 函数声明可以被提升，而类声明与 let 声明类似，不能被提升，真正执行语句之前，他们会一直存在于临时性死区中。
2. **类声明中的所有代码将自动运行在严格模式下**，而且无法强行让代码脱离严格模式执行。
3. 在自定义类型中，需要通过 Object.defineProperty() 方法手工指定某个方法不可枚举。而**在类中，所有方法都是不可枚举的**
4. 使用关键字 new 以外的方式调用类的构造函数会导致程序抛出错误
5. 在类中修改类名会导致程序报错

---


#### 类表达式

类表达式可以是具名的或匿名的，一个具名类表达式的名称是类内的一个局部属性，他可以通过类本身的 name 属性来获取。类表达式也同样受到类声明中提到的类型提升的限制

```js
// 匿名类
let person = class {
  constructor(height, width) {
    this.height = height
    this.width = width
  }
}

// 具名类
let person = class Person {
  constructor(height, width) {
    this.height = height
    this.width = width
  }
}
```

---

#### 类的静态方法 static

类通过 `static` 关键字定义静态方法。**不能在类的实例上调用静态方法**，而应该通过类本身调用。

```js
class Person {
  static sayName() {
    console.log('我的名字是xiaoming')
  }
}

// 无需实例化就可以调用
ClassWithStaticMethod.sayName() // 我的名字是xiaoming
```

如果通过实例化对象来调用该方法的话就会报错

```js
let person1 = new Person()

// Uncaught TypeError: person1.sayName is not a function
person1.sayName();
```

**静态方法的使用场景**

不需要访问类属性的方法都可以写成静态的，这类方法一般都是 Helper 方法，即对输入进行处理再得到一个输出。




