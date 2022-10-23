
# 更好的测试

```sh
npm i -D fis3 fis3-hook-commonjs vue-template-compiler
```

```js
// fis-conf.js
fis.set('project.files', ['/src/**']);

fis.unhook('components');
fis.hook('commonjs');

fis.match('/src/**.vue', {
    isMod: true,
    // useHash: true,
    rExt: '.js',
    parser: [require('./plugins/parser-vue.js')],
});
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="./public/mod.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/vue@2.7.12/dist/vue.min.js"></script>
    <script src="./output/src/test.js"></script>
</head>
<body>
    <div id="app"></div>
    <script>
        const TestApp = require('src/test.vue')
        new Vue(TestApp).$mount('#app')
    </script>
</body>
</html>
```

## version1 ~ 解析vue2简易版本

```js
// plugins/parser-vue.js
var compiler = require('vue-template-compiler');

function insertCSSFn(styleContent) {
    var styleNode = document.createElement('style');
    styleNode.setAttribute('type', 'text/css');
    if (styleNode.styleSheet) {
        styleNode.styleSheet.cssText = styleContent;
    } else {
        styleNode.appendChild(document.createTextNode(styleContent));
    }
    document.getElementsByTagName('head')[0].appendChild(styleNode);
}

var insertCSSFnStr = insertCSSFn.toString();

// exports
module.exports = function (content, file) {
    var scriptStr = '';
    var output, jsLang;

    content = content.toString();
    var output = compiler.parseComponent(content.toString(), {pad: true});
    if (output.script) {
        scriptStr = output.script.content;
        jsLang = output.script.lang || 'js';
    } else {
        scriptStr += 'module.exports = {}';
        jsLang = 'js';
    }

    scriptStr = fis.compile.partial(scriptStr, file, {
        ext: jsLang,
        isJsLike: true,
    });

    scriptStr += '\nvar __vue__options__;\n';
    scriptStr += 'if(exports && exports.__esModule && exports.default){\n';
    scriptStr += '  __vue__options__ = exports.default;\n';
    scriptStr += '}else{\n';
    scriptStr += '  __vue__options__ = module.exports;\n';
    scriptStr += '}\n';

    if (output.template) {
        var templateContent = fis.compile.partial(
            output.template.content,
            file,
            {
                ext: output.template.lang || 'html',
                isHtmlLike: true,
            }
        );
        scriptStr +=
            '__vue__options__.template = ' +
            JSON.stringify(templateContent) +
            '\n';
    }

    // style
    output.styles.forEach(function (item, index) {
        var styleContent = fis.compile.partial(item.content, file, {
            ext: item.lang || 'css',
            isCssLike: true,
        });

        scriptStr +=
            '\n;(' +
            insertCSSFnStr +
            ')(' +
            JSON.stringify(styleContent) +
            ');\n';
    });

    return scriptStr;
};
```
