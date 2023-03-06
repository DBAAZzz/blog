## 运行时 chunk 加载分析

还是一样，我们用一个简单项目来验证一下：

```js
// index.js
import("./sum").then((m) => {
  console.log(m.default(3, 4))
});
```

```js
// sum.js
const sum = (x, y) => x + y;
export default sum;
```

```js
// build.js
const webpack = require('webpack')
const path = require('path')

// 通过配置 deterministic，可获得确定的 moduleId 与 chunkId
// 在不同的编译中不变的短数字 id。有益于长期缓存。在生产模式中会默认开启。
function f1() {
  return webpack({
    entry: './code-example/code1/index.js',
    mode: 'none',
    output: {
      filename: '[name].[contenthash].js',
      chunkFilename: '[name].[id].[contenthash].chunk.js',
      path: path.resolve(__dirname, 'dist/deterministic'),
      clean: true,
    },
    optimization: {
      moduleIds: 'deterministic',
      chunkIds: 'deterministic',
    },
  })
}


f1().run((err, stat) => {
  if (err) {
    console.log(err)
  }
})

```

编译后的文件有两个，分别是 `main.js` 和动态引入sum文件后打包的 `chunk.js` 文件，核心代码为：

```js
// 513.chunk.js
"use strict";
(self["webpackChunkwebpack"] = self["webpackChunkwebpack"] || []).push([[513],{

/***/ 513:
/***/ ((__unused_webpack_module, __webpack_exports__, __webpack_require__) => {
  debugger
console.log('__webpack_require__', __webpack_require__)
__webpack_require__.r(__webpack_exports__);
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   "default": () => (__WEBPACK_DEFAULT_EXPORT__)
/* harmony export */ });
const sum = (x, y) => x + y;
/* harmony default export */ const __WEBPACK_DEFAULT_EXPORT__ = (sum);

/***/ })
}]);
```

```js
// main.js
var __webpack_exports__ = {};
__webpack_require__.e(/* import() */ 513).then(__webpack_require__.bind(__webpack_require__, 513)).then((m) => {
  console.log(m.default(3, 4))
});
```

以下两个数据结构是加载 chunk 的关键：

1. `__webpack_require__.e`: 加载 chunk。该函数将使用 `document.createElement('script')` 异步加载 `chunk` 并封装为 `Promise。`
2. `self["webpackChunk"].push`: JSONP cllaback，收集 `modules` 至 `__webpack_modules__`，并将 `__webpack_require__.e` 的 Promise 进行 resolve。


我们可以看到 chunk 文件中这段语句 `(self["webpackChunkwebpack"] = self["webpackChunkwebpack"] || []).push`， webpack 中通过重写 push 方法来实现 JsonP 的 callback。

核心代码如下：

```js
/******/ 		var webpackJsonpCallback = (parentChunkLoadingFunction, data) => {
/******/ 			var [chunkIds, moreModules, runtime] = data;
/******/ 			// add "moreModules" to the modules object,
/******/ 			// then flag all "chunkIds" as loaded and fire callback
/******/ 			var moduleId, chunkId, i = 0;
/******/ 			if(chunkIds.some((id) => (installedChunks[id] !== 0))) {
/******/ 				for(moduleId in moreModules) {
/******/ 					if(__webpack_require__.o(moreModules, moduleId)) {
/******/ 						__webpack_require__.m[moduleId] = moreModules[moduleId];
/******/ 					}
/******/ 				}
/******/ 				if(runtime) var result = runtime(__webpack_require__);
/******/ 			}
/******/ 			if(parentChunkLoadingFunction) parentChunkLoadingFunction(data);
/******/ 			for(;i < chunkIds.length; i++) {
/******/ 				chunkId = chunkIds[i];
/******/ 				if(__webpack_require__.o(installedChunks, chunkId) && installedChunks[chunkId]) {
/******/                                        // 在执行 __webpack_require__.e 方法中折行了这句 
/******/                                        // var promise = new Promise((resolve, reject) => (installedChunkData = installedChunks[chunkId] = [resolve, reject]));
/******/                                        // 调用 promise 的 resolve 方法
/******/ 					installedChunks[chunkId][0]();
/******/ 				}
/******/ 				installedChunks[chunkId] = 0;
/******/ 			}
/******/ 		
/******/ 		}
/******/ 		
/******/ 		var chunkLoadingGlobal = self["webpackChunkwebpack"] = self["webpackChunkwebpack"] || [];
/******/ 		chunkLoadingGlobal.forEach(webpackJsonpCallback.bind(null, 0));
/******/ 		chunkLoadingGlobal.push = webpackJsonpCallback.bind(null, chunkLoadingGlobal.push.bind(chunkLoadingGlobal));
/******/ 	})();
```