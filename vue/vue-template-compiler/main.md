vue-template-compiler

- An Introduction to Vue Template Compiler
		- https://masteringjs.io/tutorials/vue/template-compiler

## parseComponent

主要关注返回值

```js
const compiler = require('vue-template-compiler');
const parsed = compiler.parseComponent(`
  <template lang="html">
    <h1 class="title" @click="log">
        <span>{{setupMessage}}</span>
        <span>{{message}}</span>
    </h1>
  </template>

  <script setup>
    // 变量
    const setupMessage = 'Hello!'

    // 函数
    function log() {
        console.log(setupMessage)
    }
  </script>
  <script lang="ts">
    module.exports = {
      data: () => ({ message: 'Hello World' })
    };
  </script>

  <style lang="css">
    h1 {
        font-size: 19px;
    }
  </style>
  <style lang="less" scoped>
    h1 {
        .title {
            color: red;
        }
    }
  </style>
`);
console.dir(parsed, {
    depth: 9
});
```


```json
{
  source: '\n' +
    '  <template lang="html">\n' +
    '    <h1 class="title" @click="log">\n' +
    '        <span>{{setupMessage}}</span>\n' +
    '        <span>{{message}}</span>\n' +
    '    </h1>\n' +
    '  </template>\n' +
    '\n' +
    '  <script setup>\n' +
    '    // 变量\n' +
    "    const setupMessage = 'Hello!'\n" +
    '\n' +
    '    // 函数\n' +
    '    function log() {\n' +
    '        console.log(setupMessage)\n' +
    '    }\n' +
    '  </script>\n' +
    '  <script lang="ts">\n' +
    '    module.exports = {\n' +
    "      data: () => ({ message: 'Hello World' })\n" +
    '    };\n' +
    '  </script>\n' +
    '\n' +
    '  <style lang="css">\n' +
    '    h1 {\n' +
    '        font-size: 19px;\n' +
    '    }\n' +
    '  </style>\n' +
    '  <style lang="less" scoped>\n' +
    '    h1 {\n' +
    '        .title {\n' +
    '            color: red;\n' +
    '        }\n' +
    '    }\n' +
    '  </style>\n',
  filename: 'anonymous.vue',
  template: {
    type: 'template',
    content: '\n' +
      '<h1 class="title" @click="log">\n' +
      '    <span>{{setupMessage}}</span>\n' +
      '    <span>{{message}}</span>\n' +
      '</h1>\n',
    start: 25,
    end: 145,
    attrs: { lang: 'html' },
    lang: 'html'
  },
  script: {
    type: 'script',
    content: '\n' +
      '    module.exports = {\n' +
      "      data: () => ({ message: 'Hello World' })\n" +
      '    };\n' +
      '  ',
    start: 323,
    end: 403,
    attrs: { lang: 'ts' },
    lang: 'ts'
  },
  scriptSetup: {
    type: 'script',
    content: '\n' +
      '    // 变量\n' +
      "    const setupMessage = 'Hello!'\n" +
      '\n' +
      '    // 函数\n' +
      '    function log() {\n' +
      '        console.log(setupMessage)\n' +
      '    }\n' +
      '  ',
    start: 174,
    end: 293,
    attrs: { setup: true },
    setup: true
  },
  styles: [
    {
      type: 'style',
      content: '\nh1 {\n    font-size: 19px;\n}\n',
      start: 434,
      end: 477,
      attrs: { lang: 'css' },
      lang: 'css'
    },
    {
      type: 'style',
      content: '\nh1 {\n    .title {\n        color: red;\n    }\n}\n',
      start: 514,
      end: 583,
      attrs: { lang: 'less', scoped: true },
      lang: 'less',
      scoped: true
    }
  ],
  customBlocks: [],
  cssVars: [],
  errors: [],
  shouldForceReload: null
}
```

对于`template` 、`script`、`styles`下的单个对象，均有如下属性

- type
	- 自己，scriptSetup为script
- content
	- 解析出来的原始内容
- start
	- 开始字符
- end
	- 结束字符
- attrs
	- 标签上的属性
- lang
	- 语言：html、pug、js、ts、css、less

特有的属性
- template
	- setup
- styles
	- scoped