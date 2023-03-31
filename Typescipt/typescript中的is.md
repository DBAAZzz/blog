## typescript 中的 is

### 前言

TypeScript 里有类型保护机制。要定义一个类型保护，我们只要简单地定义一个函数，它的返回值是一个类型谓词（如下面的 test is string）：

```ts
// 类型保护
function isString(test: any): test is string {
  return typeof test === 'string'
}
```

上述写法与写一个返回值为 boolean 值函数的区别在哪里呢？

```ts
function isString(test: any): boolean {
  return typeof test === 'string'
}
```

### 使用 is 类型保护

```ts
// 使用 is 类型保护
function isString(test: any): test is string {
  return typeof test === 'string'
}

function example(foo: any) {
  if (isString(foo)) {
    console.log('it is a string' + foo)
    console.log(foo.length) // string function
    // 如下代码编译时会出错，运行时也会出错，因为 foo 是 string 不存在toExponential方法
    console.log(foo.toExponential(2))
  }
  // 编译不会出错，但是运行时出错
  console.log(foo.toExponential(2))
}
```

```ts
// 使用返回值为 boolean
function isString(test: any): boolean {
  return typeof test === 'string'
}

function example(foo: any) {
  if (isString(foo)) {
    console.log('it is a string' + foo)
    console.log(foo.length) // string function
    // foo 为 any，编译正常。但是运行时会出错，因为 foo 是 string 不存在toExponential方法
    console.log(foo.toExponential(2))
  }
}
example('hello world')
```

### 结论

结论是 **test is string**（类型谓词）用于编译阶段告诉开发人员将有机会出现运行时错误。对于 javascript ，开发人员在编译时不会知道错误。这就是使用 Typescript 的优势。

- 在使用类型保护时，TypeScript 会进一步缩小变量的类型。例子中，将类型从 any 缩小至 string
- 类型保护作用的作用域仅仅是在 if 后的块级作用域中生效
