## npm 和 npx的区别

- [npm 和 npx的区别](#npm-和-npx的区别)
  - [npm 和 npx的区别](#npm-和-npx的区别-1)
    - [npm](#npm)
    - [npx](#npx)
  - [举例](#举例)
  - [npm 和 npx 该使用哪一个](#npm-和-npx-该使用哪一个)

### npm 和 npx的区别

#### npm

npm 是 Node Package Manager 的缩写，是 NodeJs 的默认包管理器。允许用户从 npm 服务器下载别人编写的第三方包到本地使用。

#### npx 

npx 是 Node Package eXecute， 是一个 **NPM 包运行器**，它允许开发人员执行 npm 注册表上可用的任何 Javascript 包，甚至无需安装它。npx 随 npm 5.2.0 及更高版本自动安装。

总结：`npm` 代表节点包管理器，npx 是一个执行包的工具。npx 是一个 npm 包运行器，它的工作是从注册表中执行包而不安装它。

### 举例

如果执行了 `npm install create-react-app` 的话，本地目录就会将 `create-react-app` 包下载在 `node_modules` 中。

执行 `npx create-react-app myApp` 其实是执行 `node_modules` 下 bin 目录的 `create-react-app` 可执行文件，如果当前 `node_modules` 下没有这个可执行文件，就会看有没有全局的 `create-react-app` 可执行文件，全局环境也没有该执行文件的话，就会帮我们安装 create-react-app 包再帮我们删除。

注1：`node_modules` 下 bin 目录执行文件来自依赖 `package.json` 的 "bin" 字段。

注2：我们在项目中执行脚本一般是放在 package.json 里面的 script 里，例如：

```
"scripts": {
  "serve": "vue-cli-service serve",
  "build": "vue-cli-service build"
}
```

其实 vue-cli-service 执行的也是 node_modules 中 bin 目录下的可执行文件。

### npm 和 npx 该使用哪一个

如果我们包只使用一次或两次（如：npm-upgrade），而不是每次项目运行时都需要最好使用 npx。如果我们的项目需要依赖包，则使用 npm


