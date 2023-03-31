## 学习 Webpack

### webpack 方法会返回 compiler 对象

根据传入的 `options` 参数来判断 `compiler` 对象由什么方法创建，`options` 参数是数组就调用 `createMultiCompiler` 否则调用 `createCompiler`

```js
// webpack.js
if (Array.isArray(options)) {
  /** @type {MultiCompiler} */
  compiler = createMultiCompiler(
    options,
    /** @type {MultiCompilerOptions} */ (options)
  )
  watch = options.some((options) => options.watch)
  watchOptions = options.map((options) => options.watchOptions || {})
} else {
  const webpackOptions = /** @type {WebpackOptions} */ (options)
  /** @type {Compiler} */
  compiler = createCompiler(webpackOptions)
  watch = webpackOptions.watch
  watchOptions = webpackOptions.watchOptions || {}
}
```

在 `createCompiler` 方法中通过 `getNormalizeWebpackOptions`、`applyWebpackOptionsBaseDefaults` 等方法合并默认参数到 `options`

```js
const createCompiler = (rawOptions) => {
  // ...
  const options = getNormalizedWebpackOptions(rawOptions)
  applyWebpackOptionsBaseDefaults(options)
  const compiler = new Compiler(options.context, options)
  // ...
}
```

注册 NodeEnvironmentPlugin 插件

遍历挂载 options 中传入的 plugins

```js
if (Array.isArray(options.plugins)) {
  for (const plugin of options.plugins) {
    if (typeof plugin === 'function') {
      plugin.call(compiler, compiler)
    } else {
      plugin.apply(compiler)
    }
  }
}
```

执行 tapable 插件的钩子，使用 `new WebpackOptionsApply().process(options, compiler)` 初始化基础插件，这些基础插件都是 webpack 的内置模块。

其中就包括了 `EntryOptionPlugin`，`EntryOptionPlugin`的主要功能就是挂载 `EntryPlugin` 插件。

`EntryPlugin` 插件的作用是 `make` 事件触发时分析入口文件，创建模块对象等

```js
compiler.hooks.environment.call()
compiler.hooks.afterEnvironment.call()
new WebpackOptionsApply().process(options, compiler)
compiler.hooks.initialize.call()
return compiler
```

### 返回的 compiler 对象执行 run 方法

`run` 方法定义在 compiler.js 文件中，部分代码如下：

```js
class Compiler {
  run(callback) {
    const run = () => {
      // 调用 beforeRun 钩子，如果订阅 beforeRun 钩子函数没有发生错误，就继续往下执行
      this.hooks.beforeRun.callAsync(this, (err) => {
        if (err) return finalCallback(err)
        // 调用 run 钩子
        this.hooks.run.callAsync(this, (err) => {
          if (err) return finalCallback(err)

          this.readRecords((err) => {
            if (err) return finalCallback(err)
            // 开始编译
            this.compile(onCompiled)
          })
        })
      })
    }

    if (this.idle) {
      this.cache.endIdle((err) => {
        if (err) return finalCallback(err)

        this.idle = false
        run()
      })
    } else {
      run()
    }
  }
}
```

run 方法先是执行了 beforeRun 和 run 钩子，然后就执行 compile 方法进入编译阶段，compile 的部分代码如下：

```js
class Compiler {
  compile(callback) {
    const params = this.newCompilationParams()
    this.hooks.beforeCompile.callAsync(params, (err) => {
      if (err) return callback(err)

      // 调用 compile 钩子
      this.hooks.compile.call(params)

      // 创建 compilation 对象
      const compilation = this.newCompilation(params)

      // 调用 make 钩子
      this.hooks.make.callAsync(compilation, (err) => {
        if (err) return callback(err)

        this.hooks.finishMake.callAsync(compilation, (err) => {
          if (err) return callback(err)

          process.nextTick(() => {
            compilation.finish((err) => {
              if (err) return callback(err)

              compilation.seal((err) => {
                if (err) return callback(err)

                this.hooks.afterCompile.callAsync(compilation, (err) => {
                  if (err) return callback(err)

                  return callback(null, compilation)
                })
              })
            })
          })
        })
      })
    })
  }
}
```

发射事件顺序为：

- compiler.beforeRun
- compiler.run
- compiler.compile
- compiler.thisCompilation
- compiler.compilation
- compiler.make

### make 事件触发后，EntryPlugin 做了哪些事情

```js
class EntryPlugin {
  apply(compiler) {
    compiler.hooks.compilation.tap(
      'EntryPlugin',
      (compilation, { normalModuleFactory }) => {
        compilation.dependencyFactories.set(
          EntryDependency,
          normalModuleFactory
        )
      }
    )

    const { entry, options, context } = this
    const dep = EntryPlugin.createDependency(entry, options)

    compiler.hooks.make.tapAsync('EntryPlugin', (compilation, callback) => {
      // 执行 compilation 对象的 addEntry 方法
      compilation.addEntry(context, dep, options, (err) => {
        callback(err)
      })
    })
  }

  static createDependency(entry, options) {
    const dep = new EntryDependency(entry)
    // TODO webpack 6 remove string option
    dep.loc = { name: typeof options === 'object' ? options.name : options }
    return dep
  }
}
```

compilation 对象的 addEntry 方法做的事情主要是创建依赖模块树
