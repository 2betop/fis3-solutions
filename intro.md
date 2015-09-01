fis 解决方案
=================================

目前基于 fis 的解决方案越来越多，包括：

* 基于 php 和 smarty 的 [fis-plus](http://oak.baidu.com/fis-plus).
* 基于 java 和 velocity / jsp 的 [jello](http://oak.baidu.com/jello).
* 基于 node 和 [swig](https://www.npmjs.com/package/swig) 的 [yog2](https://github.com/fex-team/yog2)
* [纯 php 版本的解决方案](https://github.com/fex-team/fis3-demo/tree/master/backend-resource-manage/use-php)
* [基于 php 和 laravel blade 的解决方案](https://github.com/fis-scaffold/laravel)
* 等等...

为了更好的理解 fis 解决方案，该文档将对其定义和实现思路做必要的说明。

fis 解决方案是一个基于 fis 编译工具，结合特定后端和特定模板引擎，集成模块化开发、组件化开发、静态资源管理、目录规范、子系统拆分以及线下调试为一体的解决方案集合。

对于开发者来说，只要遵循这些简单的规则便能开发出高性能，易维护的站点。

## 模块化开发

模块化开发是复杂应用开发中有效的分治手段之一，通过模块化，可以将复杂的功能模块，根据职责划分拆成多个小模块，一来提高代码的可维护性，二来提高代码的复用性。fis 模块化包含 JS 模块化和 CSS 模块化两部分。

### JS 模块化
JS 模块化是指遵循 [CommonJs](http://wiki.commonjs.org/wiki/Modules/1.1) 或者 [AMD](https://github.com/amdjs/amdjs-api/blob/master/AMD.md) 模块化规范，将复杂的 JS 逻辑拆分成多个小模块，模块与模块之间通过 `require` 语句来关联。

大家都知道，按 [CommonJs](http://wiki.commonjs.org/wiki/Modules/1.1) 规范编写的 js 是不能直接在浏览器里面运行，在 fis 中之所以能运行，是因为借助了[mod.js](https://github.com/fex-team/mod/blob/master/mod.js)、编译工具[fis3-hook-commonjs](https://github.com/fex-team/fis3-hook-commonjs)和资源加载器。

以下将重点讲解 fis 中是如何支持 [CommonJs](http://wiki.commonjs.org/wiki/Modules/1.1) 模块化的。

#### [mod.js](https://github.com/fex-team/mod/blob/master/mod.js)

mod.js 模仿 amd 规范，实现了以下接口。

* **模块定义**

  ```js
  define(id, function(require, exports, module) {
    // 通过 exports 或者 module.exports 控制本模块需要暴露的对象。
  });
  ```

  通过指定 id 和回调函数来注册模块。此接口不直接使用，由编译工具自动生成。
* **调用模块**
  
  ```js
  // 同步用法
  // a 指向对应的模块中暴露的对象。
  var a = require(id);

  // 异步用法
  require([id1, id2], function(mod1, mod2) {
    // 当 模块 1 和 模块 2 异步加载完成后触发。
    // mod1 和 mod2 变量分别指向模块中暴露的对象。
  });

  // require([id, id2], callback) 等价于 require.async([id, id2], callback)
  ```
* **配置模块**
  
  ```js
  require.resourcemap({
    res: {
      '模块Id': {
        url: '请求地址',
        pkg: '如果资源被打包了，记录打包资源编号',
        deps: ['模块Id', '模块Id']
      }
    },
    pkg: {
      '打包资源编号': {
        url: '请求地址'
      }
    }
  });
  ```

  用来配置异步模块的信息，包括请求地址、依赖表和打包信息，当异步加载模块时， mod.js 会根据配置的信息请求资源。

  主要用来满足 js 加 md5 戳和多个模块合并成一个 js 的需求。此接口也不直接调用，有资源加载器来自动生成。

#### 编译工具 [fis3-hook-commonjs](https://github.com/fex-team/fis3-hook-commonjs)

编译工具主要包括以下几个处理。

1. **将文件路径转换成模块ID**
  
  ```js
  // 编译前
  require('./add.js'); 

  // 编译后
  require('widget/lib/add');
  ```
2. **分析依赖**

  ```js
  // 同步依赖
  require(id);

  // 异步用法
  require.async([id1, id2], callback?)

  // or require([id], callback?)
  ```

  分析项目中所有的 js 代码，将同步依赖和异步依赖，记录在静态资源表里面。对于采用后端资源加载的项目，需要产出到 map.json 文件中，交给后端程序。
3. **包裹模块化 js**

  对于标记 `isMod` 的 js, 自动用 define 包装，如：

  编译前：

  ```js
  module.exports = function(a, b) {
    return a + b;
  };
  ```

  编译后： 


  ```js
  define('widget/lib/add', function() {
    module.exports = function(a, b) {
      return a + b;
    };
  });
  ```
#### 资源加载器

资源加载器对模块化的支持的主要工作是按顺序输出以下内容。

1. 输出 `mod.js`
2. 输出页面 js 中所有的同步依赖（包括依赖中依赖）。
3. 将分析到的所有异步依赖，将其资源信息组织好通过 `require.resourceMap(info)` 告诉 mod.js
4. 输出页面中 js.

对于同步依赖的模块，实际上借助资源加载器在第2步输出 `<script>` 加载的， `mod.js` 并没有加载同步依赖。对于异步模块，则是通过资源加载器生成的资源信息，在运行时加载的。


### CSS 模块化

## 组件化开发

## 静态资源管理

## 目录规范

## 子系统拆分

## 线下调试

## 参考资料

* [前端工程--基础篇](https://github.com/fouber/blog/issues/10)

--------------



### 模板语法糖扩展

为了不污染原有 html 标签的语义，解决方案中应当基于某一模板引擎，扩展一些必要的标签（语法）以便于资源的加载和引用，以及支撑自动化性能优化、模块化开发和组件开发等。

每个解决方案中应当至少扩展 import、uri、script、style、widget、framework 和 placeholder 标签，考虑到不同模板引擎，允许扩展标签格式不一致，但是标签名字应当统一。

* `@import("静态资源ID")` 

  用来加载某一静态资源，通过 `静态资源ID` 指定。

  fis 中所有资源都可以根据配置随意部署到各种目录，或者 cdn 服务器上，最终产出的路径是相对不固定的。对于开发人员来说，使用产出路径是不可靠的，所以需要通过`静态资源ID`来指定资源。`静态资源ID` 是相对固定的，他的值为该资源在项目中的绝对路径。如：

  * `static/js/mod.js`
  * `static/css/global.css`
  * `widget/header/header.tpl`

  注意没有开头的斜线`/`，如果该项目设置了 `namespace`。

  ```js
  fis.set('namespace', 'common');
  ```

  那么`静态资源ID`会变成 `${命名空间}:${资源在该项目中绝对路径}`，如：

  * `common:static/js/mod.js`
  * `common:static/css/global.css`

  fis 编译工具，会把项目中的静态资源生成一张静态资源表存放在 `map.json` 文件中，该表通过 `静态资源ID` 记录了所有静态资源的类型、最终产出路径和依赖信息。当通过 `@import` 加载某一资源时，程序需要读取该表，将实际产出路径输出。

  同是，除了加载资源本身外，还应当进一步分析该资源依赖表，递归加载所有依赖，减少人工分析成本。这也是对 fis [三种语言能力](http://fis.baidu.com/fis3/docs/user-dev/extlang.html)中[声明依赖](http://fis.baidu.com/fis3/docs/user-dev/require.html)能力的落实。

  示例

  ```php
  ...
  @import("widget/ui/jquery/jquery.js")
  @import("static/sidebar/sidebar.css")
  ...
  ```

  `@import` 用来跨[子站点](#子站点拆分)加载资源时，需要使用完整的`静态资源ID`，但是用来加载当前[子站点](#子站点拆分)（当前项目）下的资源时，可以使用相对路径或者基于项目的绝对路径来指定。如：

  文件：/widget/header/header.tpl

  ```php
  @import("./header.js")
  @import("/static/js/lib.js")
  ```

  此类路径，需要借助编译工具，在产出给后端程序时应当被替换成完整的`静态资源ID`。
* `@uri("静态资源ID")` 

  用来输出静态资源的访问路径，并不加载该资源。

  ```php
  ...
  <div data-src="@uri('common:static/images/icon.png')">
  </div>
  ...
  ```
* `@script()@endscript` 

  与`html` 中 `<script></script>` 语法类似, 主要区别在于，通过此语法加载的 `script`, 会被收集到队列中，无论在模板什么位置使用，最终都会被合并在页面页脚处统一输出，自动性能优化。

  有了此功能，js 脚本可以按就近原则，忽略性能问题与关联的 html 写在一起，提高代码可维护性和可移植性。

  该标签支持以下三种用法：

  1. `@script() js content @endscript`

    ```php
    <div class="xxx">dom</div>
    @script()
    require(['./script.js'], function(init) {
      init('div.xxx');
    });
    @endscript
    ```
  2. `@script('远程 js 地址')@endscript` 用来加载线上 js。
  3. `@script('资源ID')@endscript` 等价于 `@import('资源ID')`
* `@style()@endstyle` 

  与`html` 中 `<style></style>` 语法类似, 主要区别在于，通过此语法加载的 `css`, 会被收集到队列中，无论在模板什么位置使用，最终都会被合并在页面头部统一输出，自动性能优化。

  有了此功能，css 内容可以按就近原则，忽略性能问题与关联的 html 写在一起，提高代码可维护性和可移植性。

  支持以下三种用法。

  1. `@style() css content @endstyle`

    ```php
    <div class="xxx">dom</div>
    @style()
    div.xxx {
      color: red;
    }
    @endstyle
    ```
  2. `@style('远程 css 地址')@endstyle` 用来加载线上 css。
  3. `@style('资源ID')@endstyle` 等价于 `@import('资源ID')`
* `@widget('子模板资源ID'[, localVars])`
  
  类似于各种模板引擎中的 `include` 功能，区别在于：

  * 支持局部变量传递。
  * 自动加载模板中依赖的资源。

  此功能主要用来支撑[组件化开发](#组件化开发)，把多个页面中可公用的部分，将 html、js、css 资源组织在同一个目录封装成组件，外部只需通过 `@widget` 引入该组件即可。

  ```php
  @if(!Auth::guest())
    @widget('/widget/userInfo/userInfo.tpl')
  @endif
  ```

* `@framework('资源ID')`
  
  指定前端运行时框架，用来支撑 CommonJs 或者 AMD 模块化开发。如果项目采用 `CommonJs` 规范请使用 [mod.js](https://github.com/fex-team/mod/blob/master/mod.js), 如果项目采用 AMD 规范请使用 `require.js`、`esl.js` 或者其他 AMD Loader。

  ```php
  ...
  @framework('/static/mod.js')
  ...
  ```

  更多信息请查看[模块化开发](#模块化开发)。
* `@placeholder('类型')` 

  后端框架需要把收集的 js 和 css 统一输出，同时为了支持[模块化开发](#模块化开发)，还需输出前端框架资源路径以及异步 js 模块资源表信息，那么具体输出在什么位置需要支持占位符来控制。

  * `@placeholder('js')` 用来控制收集到的 js 输出位置，一般都放在 body 前面。
  * `@placeholder('css')` 用来控制收集到的 css 输出位置，一般都放在 head 前面。
  * `@placeholder('framework')` 用来控制前端框架 js 输出位置。
  * `@placeholder('resource_map')` 用来控制异步 js 模块资源表输出位置。

### 线下调试



### 模块化开发

一个完整的解决方案，应该至少支持满足一种规范的模块化开发，CommonJs 规范或者 AMD 规范。

模块化开发主要有利于代码拆分管理，用来代替传统的命名空间管理方式。

如：

/widget/add.js 模块定义

```js
module.exports = function(a, b) {
  return a + b;
};
```

/page/index.js 模块使用。

```js
var add = require('/widget/add.js');

console.log(add(1, 2)); // => 3
```

对于此功能的支持工作主要集中在编译和后端运行时框架两部分，以下将详细说明如何实现结合 [mod.js](https://github.com/fex-team/mod/blob/master/mod.js) 支持 CommonJs 规范的模块化开发。

#### 编译部分

编译部分主要负责两部分工作。

1. 分析 `require` 用法，把分析到依赖信息写入到静态资源表里面，并产出静态资源表供后端运行时框架读取。

  如

  源码 module/a.js

  ```js
  var b = require('./b.js');
  var c = require('./c.js');

  module.exports = function() {
    console.log('sample');
  };
  ```

  产出的静态资源表：

  ```json
  {
    "res": {
      "module/a.js": {
        "uri": "/static/module/a.js",
        "type": "js",
        "deps": [
          "module/b.js",
          "module/c.js"
        ]
      },
      "module/b.js": {
        "uri": "/static/module/b.js",
        "type": "js"
      },
      "module/c.js": {
        "uri": "/static/module/c.js",
        "type": "js"
      },
    },
    "pkg": {}
  }
  ```
2. 将模块化的 js 用 amd 包裹如：

  源码：

  ```js
  module.exports = function(a, b) {
    return a + b;
  }
  ```

  经过编译后为：

  ```js
  define('资源ID', function(require, exports, module) {

    module.exports = function(a, b) {
      return a + b;
    }

  });
  ```

  此功能已在插件 [fis3-hook-commonjs](https://github.com/fex-team/fis3-hook-commonjs)中支持，新解决方案只需绑定此插件即可。

#### 后端框架部分

后端框架部分主要包括以下工作。

1. 记录用户通过 `@framework('/static/js/mod.js')` 指定的前端加载框架。
2. 收集页面后端渲染过程中收集的所有 js 资源，读取静态资源表，递归分析其依赖。
3. 在 `@placeholder('framework')` 位置将设置的前端加载框架输出。
4. 在 `@placeholder('resource_map')` 位置将分析到的异步 js 模块信息组织成 js 数据输出。

  ```js
  require.resourcemap({
    res: {...},
    pkg: {...}
  })
  ```
5. 把收集到的 js 以及所有同步依赖按顺序在 `@placeholder('js')` 处输出。包括模块化的和非模块化的，外联的和内联的。

从流程中可以看出，对于同步依赖的 js 实际上是后端框架分析静态资源表，提前在页面中通过 `<script>` 加载的，而不是由 `mod.js` 负责的。

对于异步 js 则是通过 `mod.js` 的接口，告知异步资源信息，在运行期结合此信息异步加载的。

### 组件化开发

在 fis 解决方案中，多个页面之间公用的部分可以封装成一个独立的组件，该组件包含后端模板片段、可选的 js 文件和可选的 css 文件。

```
└── nav
    ├── nav.tpl
    ├── nav.css
    └── nav.js
```

页面模板中通过 `@widget('组件模板'[, 局部变量])`能在相应的位置引用组件，并允许传入不同的变量, 子模板可以根据变量渲染不同的结果。

组件内部之间的资源请通过依赖或者新扩展出来的模板语法引用，作用是资源被后端框架收集，而不是马上输出，这样才能被框架自动优化。

优化包含以下内容：

1. 重复资源去重。
2. 自动分析依赖中的依赖，避免依赖缺失页面报错。
3. js 和 css 被收集，在合理位置输出，提高页面性能。

与传统的开发不一样的地方是，这种组织方式把相关的 tpl、js、css 和其他静态资源放在一个文件夹下面维护，可维护性和移植性得到很大提升。

示例：

/widget/nav/nav.tpl

```html
<div class="nav">
  <ul>
    <li><a>Link1</a></li>
    <li><a>Link2</a></li>
    <li><a>Link3</a></li>
  </ul>

  @script()
  var initNav = require('./nav.js');
  initNav('div.nav');
  @endscript
</div>
```

/widget/nav/nav.js

```js
var $ = require('/widget/lib/jquery.js');

module.exports = function(selector) {
  $(selector).on('click', function() {
    // ...
  });
};
```

/widget/nav/nav.css

```css
div.nav {
  /*样式*/
}
```

/page/index.tpl

```php
<<!DOCTYPE html>
<html>
<head>
  <title>demo</title>
  @framework('/static/js/mod.js')
  @placeholder('css')
</head>
<body>
<div>
  @widget('/widget/nav/nav.tpl')
</div>
@placeholder('framework')
@placeholder('resouce_map')
@placeholder('js')
</body>
</html>
```

产出

```php
<html>
<head>
  <title>demo</title>
  <link rel="stylesheet" type="text/css" href="/widget/nav/nav.css" />
</head>
<body>
<div>
  <div class="nav">
    <ul>
      <li><a>Link1</a></li>
      <li><a>Link2</a></li>
      <li><a>Link3</a></li>
    </ul>
  </div>
</div>
<script type="text/javascript" src="/static/js/mod.js"></script>
<script type="text/javascript" src="/widget/lib/jquery.js"></script>
<script type="text/javascript" src="/widget/nav/nav.js"></script>
<script type="text/javascript">
  var initNav = require('./nav.js');
  initNav('div.nav');
</script>
</body>
</html>
```

### 目录规范

制定目录规范可以降低项目的维护成本，每个解决方案应当预设一个合理的目录规范。

```
├── page
│   └── index.tpl
├── static
│   ├── css
│   ├── img
│   └── js
├── widget
│   ├── nav
│   └── sidebar
├── mock
│   └── sample.json
└── fis-conf.js
```

* `page` 目录用来存放页面入口模板文件。
* `static` 目录用来存放各种静态资源，如 css、图片、swf、fonts 和 js 等等。(PS: **js** 目录主要用来存放非模块化的 js，模块化 js 主要存放在 widget 目录。)
* `widget` 目录存放各类组件，组件中 js 都采用模块化方式开发。
* `mock` 用来存放各种假数据模拟文件。
* `fis-conf.js` 项目编译配置文件。

### 后端运行时框架

后端框架部分主要包含两部分工作：

1. 扩展模板语法糖，详情请见[模板语法糖扩展](#模板语法糖扩展)部分。
2. 基于静态资源表的资源管理。详情请见[模块化开发](#模块化开发)部分。

### 子站点拆分

随着业务扩展，代码越来越多，会增大项目维护成本，并且让编译工具一次编译时间增长。为了解决此问题，解决方案应当支持子站点拆分功能。

即：将完整的一个站点拆成多个 `子站点`，每个 `子站点` 为一个独立的编译单元，通过设置 `namespace` 来区分，最终将多个 `子站点` 产出合并到服务端完成整站的构建。

当 fis 项目设置了 `namespace` 后，实际上是一个 `子站点` 的角色，主要有以下区别:

（下面的例子都以 `common` 作为名字空间的值）

1. `静态资源ID` 规则发生了变化，在原来规则的基础上，以 `namespace` 作为前缀。
  
  `static/js/mod.js` => `common:static/js/mod.js`

  `子站点` 之间可以通过 `静态资源ID` 相互引用。

  如：在 home 子站点下使用 common 下面的 js.

  ```js
  var myFn = require('common:widget/comp/a.js');

  myFn('div.selector');
  ```
2. 产出的静态资源表文件名有区别。

  `map.json` => `common-map.json`

  对于后端框架来说，需要清楚此规则，根据 `资源ID` 加载相应的表。
3. 产出的文件可能需要按 `namespace` 来分目录存放。
  
  `static/js/script.js` => `static/common/js/script.js`

  `template/widget/nav/nav.tpl` => `template/common/widget/nav/nav.tpl`

4. 满足 `commonJs` 规范的模块化 js 包装成 amd 时 采用的 module Id 规范发生变化。

  原来：

  ```js
  define('widget/nav/nav', function(require, exports, module) {
    // ...
  });
  ```

  设置 `namespace` 之后 :

  ```js
  define('common:widget/nav/nav', function(require, exports, module) {
    // ...
  });
  ```
