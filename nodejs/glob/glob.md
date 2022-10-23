https://github.com/isaacs/node-glob
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
    '/src/**/*.js',
    {
        root: __dirname,
        noglobstar: false,
    },
    function (er, files) {
        files.forEach(f => {
            console.log(f.replace(__dirname, ''));
        })
        // files is an array of filenames.
        // If the `nonull` option is set, and nothing
        // was found, then files is ["**/*.js"]
        // er is an error object or null.
    }
);

```

或者
```
ls ./src/*.js
ls ./src/**.js
ls ./src/**/*.js
```