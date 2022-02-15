## Promise的resolve参数是另一个Promise实例的情况

- [Promise的resolve参数是另一个Promise实例的情况](#promise的resolve参数是另一个promise实例的情况)
  - [Promise.resolve(value)](#promiseresolvevalue)
    - [参数是一个 Promise 参数](#参数是一个-promise-参数)
    - [参数是一个 thenable 对象](#参数是一个-thenable-对象)
    - [参数不具有 then 方法的对象，或者根本不是一个对象](#参数不具有-then-方法的对象或者根本不是一个对象)
    - [不带有任何参数](#不带有任何参数)
  - [new Promise((resolve) => resolve(new Promise)) 的情况](#new-promiseresolve--resolvenew-promise-的情况)
  - [Promise.resolve(value) 不等于 new Promise((resolve) => resolve(value))](#promiseresolvevalue-不等于-new-promiseresolve--resolvevalue)


### Promise.resolve(value)

Promise.resolve(value) 方法的参数分为4种情况

#### 参数是一个 Promise 参数

如果参数是一个 Promise 参数，那么 Promise.resolve 将不做任何修改、原封不动地返回这个实例

#### 参数是一个 thenable 对象

thenable 对象是指具有 then 方法的对象，比如下面这个对象

```js
let thenable = {
  then: function(resolve, reject) {
    resolve(42)
  }
}
```

Promise.resolve 方法会将这个对象转为 Promise 对象，然后就立即执行 thenable 对象的then 方法。

```js
let thenable = {
  then: function(resolve, reject) {
    resolve(42);
  }
};

let p1 = Promise.resolve(thenable);
p1.then(function(value) {
  console.log(value);  // 42
});
```

#### 参数不具有 then 方法的对象，或者根本不是一个对象

如果参数是一个原始值，或者是一个不具备 then 方法的对象，则 Promise.resolve 方法返回一个新的 Promise 对象，状态为 `fulfilled`

```js
const p = Promise.resolve('Hello');

p.then(function (s){
  // Hello
  console.log(s)
});
```

#### 不带有任何参数

Promise.resolve 方法允许调用时不带参数，直接返回一个 fulfilled 状态的 Promise 对象

```js
// one
// two
// three

setTimeout(function () {
  console.log('three');
}, 0);

Promise.resolve().then(function () {
  console.log('two');
});

console.log('one');
```

### new Promise((resolve) => resolve(new Promise)) 的情况

当一个 Promise（p1） 的 resolve 为一个新的 Promise（p2） 时，p1 会等待 p2 的状态为 fulfilled 后再往下执行执行。


### Promise.resolve(value) 不等于 new Promise((resolve) => resolve(value))

当 value 为一个 Promise 实例的时候就会出现一些不同的地方

```js
let v = new Promise(resolve => {
  console.log("begin");
  resolve("then");
});

new Promise(resolve => {
  resolve(v);
}).then((v) => {
  console.log(v)
});

new Promise(resolve => {
  console.log(1);
  resolve();
})
  .then(() => {
    console.log(2);
  })
  .then(() => {
    console.log(3);
  })
  .then(() => {
    console.log(4);
  });
// 输出为 begin -> 1 -> 2 -> 3 -> then -> 4 
```

我们可以发现 then 推迟了两个时序

推迟原因：

浏览器会创建一个 PromiseResolveThenableJob 去处理这个 Promise 实例，这是一个微任务。

等到下次循环到来这个微任务会执行，也就是 PromiseResolveThenableJob 执行中的时候，因为这个 Promise 实例是 fulfilled 状态，所以又会注册一个它的.then()回调

又等一次循环到这个 Promise 实例它的 .then() 回调执行后，才会注册下面的这个 .then() ,于是就被推迟了两个时序

---

```js
let v = new Promise(resolve => {
  console.log("begin");
  resolve("then");
});

Promise.resolve(v).then((v) => {
  console.log(v)
});

new Promise(resolve => {
  console.log(1);
  resolve();
})
  .then(() => {
    console.log(2);
  })
  .then(() => {
    console.log(3);
  })
  .then(() => {
    console.log(4);
  });
// 输出为 begin -> 1 -> then -> 2 -> 3 -> 4 
```

我们可以看到 then 的执行时间变正常了，第一个执行的微任务就是下面这个 .then

Promise.resolve() API 如果参数是 promise 会直接返回这个 promise 实例，不会做任何处理