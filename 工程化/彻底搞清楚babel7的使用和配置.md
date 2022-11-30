## 彻底搞清楚 babel7 的使用和配置

- [彻底搞清楚 babel7 的使用和配置](#彻底搞清楚-babel7-的使用和配置)
  - [核心库 @babel/core](#核心库-babelcore)
  - [CLI 命令行工具 @babel/cli](#cli-命令行工具-babelcli)
  - [插件](#插件)
    - [语法插件](#语法插件)
    - [转换插件](#转换插件)
    - [插件的使用](#插件的使用)
  - [预设（preset）](#预设preset)
    - [官方 preset](#官方-preset)
    - [babel/preset-env](#babelpreset-env)
    - [polyfill](#polyfill)
    - [@babel/plugin-transform-runtime](#babelplugin-transform-runtime)
  - [Babel 插件和预设的执行顺序](#babel-插件和预设的执行顺序)
  - [参考文章](#参考文章)


### 核心库 @babel/core

`@babel/core` 是 `Babel` 进行转码的核心依赖包，不安装 `@babel/core` 就无法使用 `babel` 进行编译。

我们常用的 `babel-cli` 和 `babel-node` 都依赖于它。


### CLI 命令行工具 @babel/cli

`Babel` 提供的命令行工具，主要是提供 `babel` 这个命令，适合安装在项目里。

`@babel/node` 提供了 `babel-node` 命令，但是 `@babel/node` 更适合全局安装，不适合安装在项目里。

```
npm install --save-dev @babel/core @babel/cli
```

```
// package.json

"scripts": {
  "compiler": "babel src --out-dir lib --watch"
}
```

我们可以使用 `npm run compiler` 来进行编译，但是 `Babel` 不配置任何插件的话，编译前后的代码是没有变化的。

因为 Babel 虽然开箱即用，但是什么动作也不做，如果想要 `Babel` 做一些实际的工作，就需要为其添加插件(plugin)。

### 插件

`Babel` 构建在插件之上，使用现有的或者自己编写的插件可以组成一个转换通道，`Babel` 的插件分为两种: **语法插件**和**转换插件**。


#### 语法插件

这些插件只允许 `Babel` 解析（parse） 特定类型的语法（不是转换），可以在 AST 转换时使用，以支持解析新语法。例如：

```
import * as babel from "@babel/core";

const code = babel.transformFromAstSync(ast, {
  //支持可选链
  plugins: ["@babel/plugin-proposal-optional-chaining"],
  babelrc: false
}).code;
```

#### 转换插件

转换插件会启用相应的语法插件(因此不需要同时指定这两种插件)，这点很容易理解，如果不启用相应的语法插件，意味着无法解析，连解析都不能解析，又何谈转换呢？


#### 插件的使用

如果插件发布在 npm 上，可以直接填写插件的名称， `Babel` 会自动检查它是否已经被安装在 node_modules 目录下，在项目目录下新建 `.babelrc` 文件 (下文会具体介绍配置文件)，配置如下：

```
// .babelrc

{
  "plugins": ["@babel/plugin-transform-arrow-functions"]
}
```

此时再执行 `npm run compiler`  进行编译，我们可以看到箭头函数已经被编译。


现在，我们仅支持转换箭头函数，如果想将其它的新的 JS 特性转换成低版本，需要使用其它对应的 plugin 。如果我们一个个配置的话，会非常繁琐，因为你可能需要配置几十个插件，这显然非常不便，那么有没有什么办法可以简化这个配置呢？

有！预设（preset）！


### 预设（preset）

Babel 插件一般尽可能拆成小的力度，开发者可以按需引入。比如 ES6 转 ES5 的功能，Babel 官方就拆成了 20+ 插件。

可以简单的把 Babel Preset 视为 Babel Plugin 的集合。比如 babel-preset-es2015 就包含了所有跟 ES6 转换有关的插件。

#### 官方 preset 

- @babel/preset-env
- @babel/preset-flow
- @babel/preset-react
- @babel/preset-*cript

注: 从 Babel v7 开始，所以针对标准提案阶段的功能所编写的预设(stage preset)都已被弃用，官方已经移除了 @babel/preset-stage-x。


#### babel/preset-env

`@babel/preset-env` 主要作用是对我们所使用的并且目标浏览器中缺失的功能进行代码转换和加载 polyfill

在不进行任何配置的情况下，`@babel/preset-env` 所包含的插件将支持所有新的JS特性(ES2015,ES2016等，**不包含 stage 阶段**)，将其转换成ES5代码。

例如，如果你的代码中使用了可选链(目前，仍在 stage 阶段)，那么只配置 `@babel/preset-env`，转换时会抛出错误，需要另外安装相应的插件。

```
// .babelrc

{
  "presets": ["@babel/preset-env"]
}
```

需要说明的是，`@babel/preset-env` 会根据你配置的目标环境，生成插件列表来编译。

对于基于浏览器或 `Electron` 的项目，官方推荐使用 `.browserslistrc` 文件来指定目标环境。默认情况下，如果你没有在 `Babel` 配置文件中(如 `.babelrc`)设置 `targets` 或 `ignoreBrowserslistConfig`，`@babel/preset-env` 会使用 `browserslist` 配置源。


注：如果没有指定目标浏览器时 `Babel` 会假设你的目标是最旧的浏览器。例如，`@babel/preset-env`将所有 ES2015-ES2020 代码转换为 ES5 兼容。

所以，**如果你不是要兼容所有的浏览器和环境，推荐你指定目标环境，这样你的编译代码能够保持小。**

例如，仅包括浏览器市场份额超过0.25%的用户所需的 `polyfill` 和代码转换（忽略没有安全更新的浏览器，如 IE10 和 BlackBerry）:

```
// .browserslistrc

> 0.25%
not dead
```

#### polyfill

`@babel/preset-env` 提供了一个 `useBuiltIns` 参数，设置值为 `usage` 时，就只会包含代码需要的 `polyfill` 。有一点需要注意：配置此参数的值为 usage ，必须要同时设置 corejs (如果不设置，会给出警告，默认使用的是"corejs": 2) 

**注意: 这里仍然需要安装 @babel/polyfill（当前 @babel/polyfill 版本默认会安装 "corejs": 2）**

首先说一下使用 core-js@3 的原因，core-js@2 分支中已经不会再添加新特性，新特性都会添加到 core-js@3。例如你使用了 Array.prototype.flat()，如果你使用的是 core-js@2，那么其不包含此新特性。为了可以使用更多的新特性，建议大家使用 core-js@3

```
npm install --save core-js@3
```

前面曾提到，在 useBuiltIns 参数值为 usage 时，仍然需要安装 @babel/polyfill，虽然我们上面的代码转换中看起来并没有使用到


#### @babel/plugin-transform-runtime

`@babel/plugin-transform-runtime` 是一个可以重复使用 `Babel` 注入的帮助程序，以节省代码大小的插件。

使用 `@babel/plugin-transform-runtime` 插件，所有帮助程序都将引用模块 `@babel/runtime`，这样就可以避免编译后的代码中出现重复的帮助程序，有效减少包体积。


所以，`@babel/plugin-transform-runtime` 需要和 `@babel/runtime` 配合使用。

注：不过只需要安装 `@babel/plugin-transform-runtime` 即可，`@babel/plugin-transform-runtime` 的 `dependencies` 包括了 `@babel/runtime`

首先安装依赖，`@babel/plugin-transform-runtime` 通常仅在开发时使用，但是运行时终代码需要依赖 `@babel/runtime`，所以 `@babel/runtime` 必须要作为生产依赖被安装。

除了前文所说的，`@babel/plugin-transform-runtime` 可以减少编译后代码的体积外，我们使用它还有一个好处，它可以为代码创建一个沙盒环境，如果使用 `@babel/polyfill` 及其提供的内置程序（例如 Promise ，Set 和 Map ），则它们将污染全局范围。虽然这对于应用程序或命令行工具可能是可以的，但是如果你的代码是要发布供他人使用的库，或者无法完全控制代码运行的环境，则将成为一个问题。

`@babel/plugin-transform-runtime` 会将这些内置别名作为 core-js 的别名，因此您可以无缝使用它们，它可以将这些变量隔离在局部作用域中。

修改 .babelrc 的配置，如下:

```
//.babelrc

{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "usage",
        "corejs": 3
      }
    ]
  ],
  "plugins": [
    [
      "@babel/plugin-transform-runtime"
    ]
  ]
}
```

### Babel 插件和预设的执行顺序

- 插件比预设先执行
- 插件执行顺序是插件数组从前向后执行
- 预设执行顺序是预设数组从后向前执行


### 参考文章

[不容错过的 Babel7 知识-刘小夕](https://z.itpub.net/article/detail/7190260E1D017F7CBC90574CBA48E043)