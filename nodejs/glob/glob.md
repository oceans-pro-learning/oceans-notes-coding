https://github.com/isaacs/node-glob

一些特殊符号的含义如下

`*`  `Matches 0 or more characters in a single path portion
`**` If a "globstar" is alone in a path portion, then it matches zero or more directories and subdirectories searching for matches. It does not crawl symlinked directories.

注意

- 1个星只能匹配一个路径分隔符里面的内容
- 两个星星如果单独在一个路径分隔符里面（有且只有一个），则代表多层路径匹配，如果不是在一个路径里，那么两个星星其实是一个星星 因为 `2 ✖️ [0, N]` = `[0, 2N]`
	`src/**/*.js`
	`src/**`

假设目录结构为

- src
	- **src.js**
	- sub
			- **sub.js**
			- sub_sub
				- **sub_sub.js**

```js
var glob = require('glob');

// options is optional
glob(
    // '/src/*.js',
    // '/src/**.js',
    // '/src/**',
    '/src/**/*.js',
    {
        root: __dirname,
        noglobstar: false,
    },
    function (err, files) {
        files.forEach(f => {
            console.log(f.replace(__dirname, ''));
        })
        // files is an array of filenames.
        // If the `nonull` option is set, and nothing
        // was found, then files is ["**/*.js"]
        // err is an error object or null.
    }
);

```

或者
```
ls ./src/*.js
ls ./src/**.js
ls ./src/**/*.js
```