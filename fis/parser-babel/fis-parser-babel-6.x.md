
# 预备知识

## babel-core

	- https://www.npmjs.com/package/babel-core

babel.transform

```js
var babel = require("babel-core");
import * as babel from 'babel-core';
var result = babel.transform("code();", options);
result.code;
result.map;
result.ast;
```

## presets

- babel-preset-es2015
	- `npm WARN deprecated babel-preset-es2015@6.24.1: 🙌  Thanks for using Babel: we recommend using babel-preset-env now: please read https://babeljs.io/env to update!`
	- https://zhuanlan.zhihu.com/p/29506685
- babel-preset-stage-3
	- https://babel.docschina.org/docs/en/6.26.3/babel-preset-stage-3/
- babel-preset-react

## fis.util.extend

```js
var lodash = require('lodash');
/**
 * fis 中工具类操作集合。{@link https://lodash.com/ lodash} 中所有方法都挂载在此名字空间下面。
 * @param  {String} path
 * @return {String}
 * @example
 *   /a/b//c\d/ -> /a/b/c/d
 * @namespace fis.util
 */
var _ = module.exports = function(path) {
  var type = typeof path;
  if (arguments.length > 1) {
    path = Array.prototype.join.call(arguments, '/');
  } else if (type === 'string') {
    //do nothing for quickly determining.
  } else if (type === 'object') {
    path = Array.prototype.join.call(path, '/');
  } else if (type === 'undefined') {
    path = '';
  }
  if (path) {
    path = pth.normalize(path.replace(/[\/\\]+/g, '/')).replace(/\\/g, '/');
    if (path !== '/') {
      path = path.replace(/\/$/, '');
    }
  }
  return path;
};

// 将lodash内部方法的引用挂载到utils上，方便使用
lodash.assign(_, lodash);

_.is = function(source, type) {
  return toString.call(source) === '[object ' + type + ']';
};

// ...
```


# 源码

```js
'use strict';

var babel = require('babel-core');
var preset2015 = require('babel-preset-es2015');
var presetstage3 = require('babel-preset-stage-3');
var react = require('babel-preset-react');


module.exports = function (content, file, conf) {
   // 添加 useBabel 配置项，如果 useBabel 为 false 则不进行编译
    if (file.useBabel === false) {
        return content;
    }

    conf = fis.util.extend({
        presets: [
            preset2015,
            presetstage3,
            react
        ]
    }, conf);

    // 添加 jsx 的 html 语言能力处理
    if (fis.compile.partial && file.ext === '.jsx') {
        content = fis.compile.partial(content, file, {
            ext: '.html',
            isHtmlLike: true
        });
    }

    // 出于安全考虑，不使用原始路径
    // conf.filename = file.subpath;

    var result = babel.transform(content, conf);
    return result.code;
};

```

babel7

react-jsx-runtime.production.min
```json
"@babel/core": "^7.12.8",
"@babel/preset-env": "^7.12.7",
"@babel/preset-react": "^7.12.7",
```


```js
'use strict';

var babel = require('@babel/core');
var presetEnv = require('@babel/preset-env');
var react = require('@babel/preset-react');


module.exports = function (content, file, conf) {
   // 添加 useBabel 配置项，如果 useBabel 为 false 则不进行编译
    if (file.useBabel === false) {
        return content;
    }

    conf = fis.util.extend({
        presets: [
            presetEnv,
            react
        ]
    }, conf);

    // 添加 jsx 的 html 语言能力处理
    if (fis.compile.partial && file.ext === '.jsx') {
        content = fis.compile.partial(content, file, {
            ext: '.html',
            isHtmlLike: true
        });
    }

    // 出于安全考虑，不使用原始路径
    // conf.filename = file.subpath;

    var result = babel.transform(content, conf);
    return result.code;
};

```