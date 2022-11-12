
```js
const babel = require('@babel/core');
const findBabelConfig = require('find-babel-config');

module.exports = function (content, file, conf) {
    console.log( file.subpath);
    const root = fis.project.getProjectPath();

    const babelrc = findBabelConfig.sync(root).config;

    // 合并配置
    // 权重: 插件配置 > .babelrc
    const defaultBabelrc = {
        // sourceMaps: true,
        filename: file.subpath,
        // sourceMaps: true,
        sourceMaps: 'inline',
    };
    babelConf = Object.assign({}, defaultBabelrc, babelrc, conf);
    console.log(babelConf);

   // 添加 useBabel 配置项，如果 useBabel 为 false 则不进行编译
    if (file.useBabel === false) {
        return content;
    }

    // 添加 jsx 的 html 语言能力处理 http://fis.baidu.com/fis3/docs/user-dev/extlang.html#html
    // 资源定位 内容嵌入 依赖声明
    if (file.ext === '.jsx' || file.ext === '.tsx') {
        content = fis.compile.partial(content, file, {
            isHtmlLike: true
        });
    }

    
    // 出于安全考虑，不使用原始路径


    const {code, map} = babel.transformSync(content, babelConf);

    let resCode = code;
    const sourceMapRelative = true;

    // 添加 sourceMap 输出
    if (0 && map) {
        var mapping = fis.file.wrap(file.dirname + '/' + file.filename + file.rExt + '.map');
        mapping.setContent(JSON.stringify(map, null, 4));
        // var url = babelConf.sourceMapRelative ? ('./' + file.basename + '.map').replace('jsx', 'js') :
        
        //     mapping.getUrl(fis.compile.settings.hash, fis.compile.settings.domain);

        var url = ('./' + file.basename + '.map').replace('jsx', 'js');
        var url = ('./' + file.basename + '.map').replace('tsx', 'js');

        resCode = resCode.replace(/\n?\s*\/\/#\ssourceMappingURL=.*?(?:\n|$)/g, '');
        resCode += '\n//# sourceMappingURL=' + url + '\n';
        file.derived.push(mapping);
    }

    return resCode;
};
```