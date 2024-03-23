---
layout: post
title:  "Tree Shaking"
date:   2017-03-23 23:48:52 +0800
categories: jekyll update
---
Rich Harris 的模块打包器 Rollup 普及了 JavaScript 圈内一个重要的特性：Tree-shaking，即从模块包中排除未使用的 exports 项。Rollup 基于 ES6 模块的静态结构（引入和导出不会在运行时改变）来检测哪一个模块没有被使用。

Webpack 的 Tree-shaking 特性目前正在 beta 阶段，这篇博文解释了它是如何工作的，相关的项目代码可以在 Github 上查看：[Tree-shaking Demo](https://github.com/rauschma/tree-shaking-demo)。


### 1. Webpack 2 如何消除未使用的导出

Webpack 2 是一个尚处于 beta 阶段的新版本，它通过下面两个步骤来消除未使用的导出:

* 首先，所有 ES6 模块文件会被合并为一个打包文件。在这个文件中在任何地方都没有被引入的导出将不再会被打包（译注：`export` 语句将会直接被删除）。
其次，通过消除不可到达的代码，将打包文件精简。因此，那些既没有被导出，又没有在其模块中被使用的代码将不会出现在精简的打包文件中。如果没有第一个步骤，那些被导出的不可到达代码永远不会被删除（`export` 关键字会访问他们）。

* 只有模块系统拥有一个静态的结构时，模块加载器才能在构建阶段对没有使用过的 `export` 语句进行可靠的检测。因此，Webpack 2 可以解析并且理解所有 ES6 代码，并且仅对检测到的 ES6 模块进行 Tree-shaking。但不管怎样，只有 `import` 和 `export` 语句会被 Webpack 转译为 ES5，如果希望所有代码都转译为 ES5，你仍然需要一个转译器对剩余部分的代码进行转译。在这篇博文中，我们会使用 Babel 6。


### 2. 输入：ES6 代码

Demo 项目有两个 ES6 模块。

helpers.js 包含辅助函数：
```
// helpers.js
export function foo() {
  return 'foo';
}
export function bar() {
  return 'bar';
}
```

main.js，web 应用的入口：
```
// main.js
import { foo } from './helpers';

let elem = document.getElementById('output');
elem.innerHTML = `Output: ${foo()}`;
```
需要注意的是在 helpers 模块中导出的 bar 在整个项目的任何地方都没有被使用过。

### 3. 不使用 Tree-shaking 的输出

Babel 6 的典型选择是使用 es2015 这个 preset：
```
{
  presets: ['es2015'],
}
```

不管怎样，这一 preset 包含了 transform-es2015-modules-commonjs 这个插件，这意味着 Babel 会输出 CommonJS 规范的模块，这时 Webpack 无法进行 Tree-shaking：
```
function(module, exports) {

  'use strict';

  Object.defineProperty(exports, "__esModule", {
    value: true
  });
  exports.foo = foo;
  exports.bar = bar;
  function foo() {
    return 'foo';
  }
  function bar() {
    return 'bar';
  }

}
```
你可以看到在代码精简的过程中，`bar` 作为导出的一部分使它不能被识别为不可访问的代码。

### 4. 使用 Tree-shaking 的输出

我们需要的是 Babel 的 es2015，并不需要其中的 transform-es2015-modules-commonjs 插件。这时，实现以上需求的唯一途径是在配置中声明除 transform-es2015-modules-commonjs 以外的所有的插件。preset 的源代码托管在 Gibhub 上，所以我们这里复制粘贴需要的插件：
```
{
  plugins: [
    'transform-es2015-template-literals',
    'transform-es2015-literals',
    'transform-es2015-function-name',
    'transform-es2015-arrow-functions',
    'transform-es2015-block-scoped-functions',
    'transform-es2015-classes',
    'transform-es2015-object-super',
    'transform-es2015-shorthand-properties',
    'transform-es2015-computed-properties',
    'transform-es2015-for-of',
    'transform-es2015-sticky-regex',
    'transform-es2015-unicode-regex',
    'check-es2015-constants',
    'transform-es2015-spread',
    'transform-es2015-parameters',
    'transform-es2015-destructuring',
    'transform-es2015-block-scoping',
    'transform-es2015-typeof-symbol',
    ['transform-regenerator', { async: false, asyncGenerators: false }],
  ],
}
```
如果现在我们构建这个项目，helper 模块在打包文件中看起来就像这样：

```
function(module, exports, __webpack_require__) {

  /* harmony export */ exports["foo"] = foo;
  /* unused harmony export bar */;

  function foo() {
      return 'foo';
  }
  function bar() {
      return 'bar';
  }
}
```
现在，只有 foo 被导出了，但 bar 的代码仍然存在。经过代码精简，helpers 看起来就像这样：
```
function (t, n, r) {
  function e() {
      return "foo"
  }

  n.foo = e
}
```
