基础配置
https://github.com/ccqgithub/fis3-parser-vue-component
vue-loader
https://vue-loader.vuejs.org/zh/guide/
https://www.npmjs.com/package/vue-loader

```js
fis.match('src/**.vue', {
  isMod: true,
  rExt: 'js',
  useSameNameRequire: true,
  parser: [
    fis.plugin('vue-component', {
      // vue@2.x runtimeOnly
      runtimeOnly: true,          // vue@2.x 有runtimeOnly模式，为true时，template会在构建时转为render方法

      // styleNameJoin
      styleNameJoin: '',          // 样式文件命名连接符 `component-xx-a.css`

      extractCSS: true,           // 是否将css生成新的文件, 如果为false, 则会内联到js中

      // css scoped
      cssScopedIdPrefix: '_v-',   // hash前缀：_v-23j232jj
      cssScopedHashType: 'sum',   // hash生成模式，num：使用`hash-sum`, md5: 使用`fis.util.md5`
      cssScopedHashLength: 8,     // hash 长度，cssScopedHashType为md5时有效

      cssScopedFlag: '__vuec__',  // 兼容旧的ccs scoped模式而存在，此例子会将组件中所有的`__vuec__`替换为 `scoped id`，不需要设为空
    })
  ],
});
```


可以看到，大部分的配置是与样式有关的！只有`runtimeOnly`是非样式相关的配置呢。


## 工具方法分析

### template-compiler

注意看
```js
// runtimeOnly
if(configs.runtimeOnly){
  var result = compileTemplate(templateContent);
  if(result){
	scriptStr += '__vue__options__.render =' + result.render + '\n';
	scriptStr += '__vue__options__.staticRenderFns =' + result.staticRenderFns + '\n';
  }
}else{
  // template
  scriptStr += '__vue__options__.template = ' + JSON.stringify(templateContent) + '\n';
}
```

作用：大部分是报错输出代码，本质的代码只是调用`vue-template-compiler`和`vue-template-es2015-compiler`。
返回 `render`和`staticRenderFns`，这俩其实是`vue-template-compiler`库返回的对象。

- vue-template-compiler
	- https://www.npmjs.com/package/vue-template-compiler
- vue-template-es2015-compiler
	- https://www.npmjs.com/package/vue-template-es2015-compiler
- buble
	- 可以替换babel的一个产品
	- https://buble.surge.sh/
```js
var chalk = require('chalk')
var vueCompiler = require('vue-template-compiler')
var transpile = require('vue-template-es2015-compiler')

module.exports = function compileTemplate (template) {
  var compiled = vueCompiler.compile(template)
  if (compiled.errors.length) {
    compiled.errors.forEach(function (msg) {
      console.error('\n' + chalk.red(msg) + '\n')
    })
    throw new Error('Vue template compilation failed')
  } else {
    return {
      render: toFunction(compiled.render),
      staticRenderFns: '[' + compiled.staticRenderFns.map(toFunction).join(',') + ']'
    }
  }
}

function toFunction (code) {
  return transpile('function render () {' + code + '}')
}

```

runtimeOnly设置为true
```js
__vue__options__.render = function render() {
	var _vm = this;
	var _h = _vm.$createElement;
	var _c = _vm._self._c || _h;
	return _c('div', [
		_c('span', [_vm._v('content')]),
		_vm._v(' '),
		_c('span', [_vm._v(_vm._s(_vm.age))]),
	]);
};
__vue__options__.staticRenderFns = [];
```
否则
```js
__vue__options__.template =
	'\n<div>\n    <span>content</span>\n    <span>{{age}}</span>\n</div>\n';
```
### gen-id.js

- hash-sum
	- https://github.com/bevacqua/hash-sum

哈希操作要进行IO操作，所以这里缓存了一下子
```js
// utility for generating a uid for each component file
// used in scoped CSS rewriting
var hash = require('hash-sum')
var cache = Object.create(null)

// module.exports = function genId (file) {
//   return cache[file] || (cache[file] = hash(file))
// }

module.exports = function genId(file, configs){
  if(cache[file.subpath]){
    return cache[file.subpath];
  }

  var scopeId;

  // scope replace
  if (configs.cssScopedType == 'sum') {
    scopeId = hash(file.subpath);
  } else {
    scopeId = fis.util.md5(file.subpath, configs.cssScopedHashLength);
  }

  return cache[file.subpath] = scopeId;
};
```

###  insert-css.js

功能：生成一段可以插入CSS的脚本字符串。

依赖：
- uglify-js
	- https://github.com/mishoo/UglifyJS
	- https://lisperator.net/uglifyjs/

https://stackoverflow.com/questions/524696/how-to-create-a-style-tag-with-javascript

```js
var uglifyJS = require('uglify-js');

function insertCSS(styleContent) {
  var styleNode = document.createElement("style");
  styleNode.setAttribute("type", "text/css");
  // IE
  if (styleNode.styleSheet) {
    styleNode.styleSheet.cssText = styleContent;
  }
  else {
	  //w3c浏览器中只要创建文本节点插入到style元素中就行了
	  styleNode.appendChild(document.createTextNode(styleContent));
  }
  document.getElementsByTagName("head")[0].appendChild(styleNode);
};

module.exports = uglifyJS.minify(insertCSS.toString(), {fromString: true}).code;

```

### replace-scoped-flag.js

作用：是要兼容旧版本。

cssScopedFlag之前应该是需要在代码里面去手动写的，比如手动写一个，`__vuec__`。现在要把手动写的这个改成vuecId了。
```js
module.exports = function(str, cssScopedFlag, vuecId) {
  var reg = new RegExp('([^a-zA-Z0-9\-_])('+ cssScopedFlag +')([^a-zA-Z0-9\-_])', 'g');
  str = str.replace(reg, function($0, $1, $2, $3) {
    return $1 + vuecId + $3;
  });
  return str;
}
```

### style-rewriter

- postcss
	- https://www.npmjs.com/package/postcss
	- https://github.com/postcss/postcss
	- https://postcss.org/
- postcss-selector-parser
	- https://www.npmjs.com/package/postcss-selector-parser
	- https://github.com/postcss/postcss-selector-parser/blob/master/API.md
- lru-cache
	- https://www.npmjs.com/package/lru-cache

```js
var postcss = require('postcss')
var selectorParser = require('postcss-selector-parser')
var cache = require('lru-cache')(100)
var assign = require('object-assign')

var deasync = require('deasync')

var currentId
var addId = postcss.plugin('add-id', function () {
  return function (root) {
    root.each(function rewriteSelector (node) {
      if (!node.selector) {
        // handle media queries
        if (node.type === 'atrule' && node.name === 'media') {
          node.each(rewriteSelector)
        }
        return
      }
      node.selector = selectorParser(function (selectors) {
        selectors.each(function (selector) {
          var node = null
          selector.each(function (n) {
            if (n.type !== 'pseudo') node = n
          })
          selector.insertAfter(node, selectorParser.attribute({
            attribute: currentId
          }))
        })
      }).process(node.selector).result
    })
  }
})

/**
 * Add attribute selector to css
 *
 * @param {String} id
 * @param {String} css
 * @param {Boolean} scoped
 * @param {Object} options
 * @return {Promise}
 */

module.exports = deasync(function (id, css, scoped, options, cbk) {
  var key = id + '!!' + scoped + '!!' + css
  var val = cache.get(key)
  if (val) {
    cbk(null, val)
  } else {
    var plugins = []
    var opts = {}

    // // do not support post plugins here, use fis3 style plugins
    // if (options.postcss instanceof Array) {
    //   plugins = options.postcss.slice()
    // } else if (options.postcss instanceof Object) {
    //   plugins = options.postcss.plugins || []
    //   opts = options.postcss.options
    // }

    // scoped css rewrite
    // make sure the addId plugin is only pushed once
    if (scoped && plugins.indexOf(addId) === -1) {
      plugins.push(addId)
    }

    // remove the addId plugin if the style block is not scoped
    if (!scoped && plugins.indexOf(addId) !== -1) {
      plugins.splice(plugins.indexOf(addId), 1)
    }

    // // do not support cssnano minification here, use fis3 style plugins
    // // minification
    // if (process.env.NODE_ENV === 'production') {
    //   plugins.push(require('cssnano')(assign({
    //     safe: true
    //   }, options.cssnano)))
    // }
    currentId = id

    postcss(plugins)
      .process(css, opts)
      .then(function (res) {
        cache.set(key, res.css)
        cbk(null, res.css)
      })
  }
})
```

## 核心入口文件源码分析

- `_scopeId`
	- https://segmentfault.com/a/1190000039677087
	- 只适用于vue2

```js
var path = require('path');
var objectAssign = require('object-assign');
var hashSum = require('hash-sum');
var compiler = require('vue-template-compiler');

var genId = require('./lib/gen-id');
var rewriteStyle = require('./lib/style-rewriter');
var compileTemplate = require('./lib/template-compiler');
var insertCSS = require('./lib/insert-css');
var replaceScopedFlag = require('./lib/replace-scoped-flag');

// exports
module.exports = function(content, file, conf) {
  var scriptStr = '';
  var output, configs, jsLang;

  // configs
  configs = objectAssign({
    cssScopedFlag: '__vuec__',

    extractCSS: true,
    cssScopedIdPrefix: '_v-',
    cssScopedHashType: 'sum',
    cssScopedHashLength: 8,
    styleNameJoin: '',

    runtimeOnly: false,
  }, conf);

  // 兼容content为buffer的情况
  content = content.toString();

  // generate css scope id
  var id = configs.cssScopedIdPrefix + genId(file, configs);

  // 兼容旧的cssScopedFlag
  if (configs.cssScopedFlag) {
    content = replaceScopedFlag(content, configs.cssScopedFlag, id);
  }

  // parse
  var output = compiler.parseComponent(content.toString(), { pad: true });

  // check for scoped style nodes
  var hasScopedStyle = output.styles.some(function (style) {
    return style.scoped
  });

  // script
  if (output.script) {
    scriptStr = output.script.content;
    jsLang = output.script.lang || 'js';
  } else {
    scriptStr += 'module.exports = {}';
    jsLang = 'js';
  }

  scriptStr = fis.compile.partial(scriptStr, file, {
    ext: jsLang,
    isJsLike: true
  });

  scriptStr += '\nvar __vue__options__;\n';
  scriptStr += 'if(exports && exports.__esModule && exports.default){\n';
  scriptStr += '  __vue__options__ = exports.default;\n';
  scriptStr += '}else{\n';
  scriptStr += '  __vue__options__ = module.exports;\n';
  scriptStr += '}\n';

  if(output.template){
    var templateContent = fis.compile.partial(output.template.content, file, {
      ext: output.template.lang || 'html',
      isHtmlLike: true
    });
    // runtimeOnly
    if(configs.runtimeOnly){
      var result = compileTemplate(templateContent);
      if(result){
        scriptStr += '__vue__options__.render =' + result.render + '\n';
        scriptStr += '__vue__options__.staticRenderFns =' + result.staticRenderFns + '\n';
      }
    }else{
      // template
      scriptStr += '__vue__options__.template = ' + JSON.stringify(templateContent) + '\n';
    }
  }

  if(hasScopedStyle){
    // template
    scriptStr += '__vue__options__._scopeId = ' + JSON.stringify(id) + '\n';
  }

  // style
  output.styles.forEach(function(item, index) {
    if(!item.content){
      return;
    }

    // empty string, or all space line
    if(/^\s*$/.test(item.content)){
      return;
    }

    // css也采用片段编译，更好的支持less、sass等其他语言
    var styleContent = fis.compile.partial(item.content, file, {
      ext: item.lang || 'css',
      isCssLike: true
    });

    styleContent = rewriteStyle(id, styleContent, item.scoped, {})

    if(!configs.extractCSS){
      scriptStr += '\n;(' + insertCSS + ')(' + JSON.stringify(styleContent) + ');\n';
      return;
    }
    // 更多是将css单独提取到文件中

    var styleFileName, styleFile;

    if (output['styles'].length == 1) {
      styleFileName = file.realpathNoExt + configs.styleNameJoin + '.css';
    } else {
      styleFileName = file.realpathNoExt + configs.styleNameJoin + '-' + index + '.css';
    }

    styleFile = fis.file.wrap(styleFileName);

    styleFile.cache = file.cache;
    styleFile.isCssLike = true;
    styleFile.setContent(styleContent);
    fis.compile.process(styleFile);
    styleFile.links.forEach(function(derived) {
      file.addLink(derived);
    });
    file.derived.push(styleFile);
    file.addRequire(styleFile.getId());
  });

  return scriptStr;
};

```

### extractCSS模式

预备知识点补充：

- fis.file.wrap
- file.setContent
- file.addLink
- file.derived.push
- file.addRequire
- fis.compile.partial
- fis.compile.process

```js
// 更多是将css单独提取到文件中

var styleFileName, styleFile;

if (output['styles'].length == 1) {
  styleFileName = file.realpathNoExt + configs.styleNameJoin + '.css';
}
else {
  styleFileName = file.realpathNoExt + configs.styleNameJoin + '-' + index + '.css';
}

    styleFile = fis.file.wrap(styleFileName);

    styleFile.cache = file.cache;
    styleFile.isCssLike = true;
    styleFile.setContent(styleContent);
    fis.compile.process(styleFile);
    styleFile.links.forEach(function(derived) {
      file.addLink(derived);
    });
    file.derived.push(styleFile);
    file.addRequire(styleFile.getId());
  });

```
### 验证`_scopeId`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="https://cdn.jsdelivr.net/npm/vue@2.7.12/dist/vue.min.js"></script>
</head>
<body>
    <div id="app"></div>
    <script>
        new Vue({
            template: `
                <div>
                    vue
                    <span>{{age}}</span>
                </div>
            `,
            data: {
                age: 1,
            },
            _scopeId: 'v_id_123',
        }).$mount('#app')
    </script>
</body>
</html>
```


