```js
/**
 * 用来包裹文件，输入可以是路径也可以是文件对象，输出统一为文件对象。
 * @param  {Mixed} file 文件路径，或者文件对象
 * @return {@link fis.file~File File 对象}
 * @example
 * var file = fis.file.wrap('path of file');
 * var file2 = fis.file.wrap(file);
 * @memberOf fis.file
 * @name wrap
 */
module.exports.wrap = function(file) {
  if (typeof file === 'string') {
    return new File(file);
  } else if (file instanceof File) {
    return file;
  } else {
    fis.log.error('unable to convert [%s] to [File] object.', (typeof file));
  }
};
```

# fis.plugin

这个方法只是处理配置罢了。

```js
/**
 * 仅限于 fis-conf.js 中使用，用来包装 fis 插件配置。
 *
 * 需要在对应的插件扩展点中配置才有效，否则直接执行 fis.plugin 没有任何意义。
 *
 * 单文件扩展点：
 * - lint
 * - parser
 * - preprocess
 * - standard
 * - postprocess
 * - optimizer
 *
 * 打包阶段扩展点：
 * - prepacakger
 * - sprite
 * - packager
 * - postpackager
 *
 * @example
 * fis.match('*.scss', {
 *   parser: fis.plugin('sass', {
 *     include_paths: [
 *       './static/scss/libaray'
 *     ]
 *   })
 * });
 *
 * fis.match('::packager', {
 *   postpackager: fis.plugin('loader', {
 *     allInOne: true
 *   });
 * })
 * @memberOf fis
 * @param {String} pluginName 插件名字。
 *
 * 说明：pluginName 不是对应的 npm 包名，而是对应 npm 包名去掉 fis 前缀，去掉插件扩展点前缀。
 *
 * 如：fis-parser-sass 包，在这里面配置就是:
 *
 * ```js
 * fis.match('*.scss', {
 *   parser: fis.plugin('sass')
 * });
 * ```
 * @param {Object} options 插件配置项，具体请参看插件说明。
 * @param {String} [position] 可选：'prepend' | 'append'。默认为空，当给某类文件配置插件时，都是覆盖式的。而通过设置此插件可以做到，往前追加和往后追加。
 *
 * 如：
 *
 * ```js
 * fis.match('*.xxx', {
 *   parser: fis.plugin('a')
 * });
 *
 * // 保留 plugin a 同时，之后再执行 plugin b
 * fis.match('*.xxx', {
 *   parser: fis.plugin('b', null, 'append')
 * });
 * ```
 */
fis.plugin = function(key, options, position) {
  // 兼容这种情况：当传入两个参数时，第一个是插件名，第二个是位置。
  if (arguments.length === 2 && !_.isPlainObject(options)) {
    position = options;
    options = null;
  }

  options = options || {};
  options.__name = options.__plugin = key;
  options.__pos = position;
  options.__isPlugin = true;
  return options;
};
```
# fis.log
