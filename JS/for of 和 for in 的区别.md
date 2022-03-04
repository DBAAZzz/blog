## for of 和 for in 的区别

- [for of 和 for in 的区别](#for-of-和-for-in-的区别)
  - [for...in](#forin)
  - [Object.keys()](#objectkeys)
  - [for...of](#forof)
  - [for...of 和 for...in 的区别](#forof-和-forin-的区别)

### for...in

**for...in 循环只遍历可枚举属性**（包括它的原型链上的可枚举属性），**获取的是键**。像 Array 和 Object 使用内置构造函数创建的对象都会继承 Object.prototype 和 String.prototype 的不可枚举属性。

```js
var obj = {a:1, b:2, c:3};
    
for (let key in obj) {
  console.log(key);  // a b c
}
```

### Object.keys()

`Object.keys()` 方法会返回一个由一个给定对象的**自身可枚举属性**组成的数组，数组中属性名的排列顺序和正常循环遍历该对象时返回的顺序一致 。

```js
// array like object
var obj = { 0: 'a', 1: 'b', 2: 'c' };
console.log(Object.keys(obj)); // console: ['0', '1', '2']
```


### for...of

for...of 语句在可迭代对象（包括Array、Map、Set、String、TypeArray、argument对象上等等）上创建一个迭代循环，调用自定义迭代钩子，并为每个不同属性的值执行语句。

**注意：for...of 不可以遍历对象，想要遍历对象的属性，可以用for in 循环，或者内建的 Object.keys() 方法**

```js
const array1 = ['a', 'b', 'c'];

for (const val of array1) {
  console.log(val);
}
```


### for...of 和 for...in 的区别

for...in 语句以任意顺序迭代对象的可枚举属性。

for...of 语句遍历可迭代对象定义要迭代的数据。

以下示例显示了与Array一起使用时，for...of循环和for...in循环之间的区别。

```js
Object.prototype.objCustom = function() {}; 
Array.prototype.arrCustom = function() {};

let iterable = [3, 5, 7];
iterable.foo = 'hello';

for (let i in iterable) {
  console.log(i); // 0, 1, 2, "foo", "arrCustom", "objCustom"
}

for (let i in iterable) {
  if (iterable.hasOwnProperty(i)) {
    console.log(i); // 0, 1, 2, "foo"
  }
}

for (let i of iterable) {
  console.log(i); // logs 3, 5, 7
}
```

**总结：for in 一般用来遍历对象的key、for of 一般用来遍历数组的value**