
## 一个普通的file对象

```js
const root = fis.project.getProjectPath();
var file = fis.file(root, 'static/foo.js');

fis.log.info(file)
```

```js
{
    origin: '/Users/linyangyang/dir/static/foo.js',
    rest: '/Users/linyangyang/dir/static/foo',


    hash: '',
    query: '',


    fullname: '/Users/linyangyang/dir/static/foo.js',
    dirname: '/Users/linyangyang/dir/static',

    ext: '.js',
    filename: 'foo',
    basename: 'foo.js',


    realpath: '/Users/linyangyang/dir/static/foo.js',
    realpathNoExt: '/Users/linyangyang/dir/static/foo',


    subpath: '/static/foo.js',
    subdirname: '/static',
    subpathNoExt: '/static/foo',

    rExt: '.js',

    useCompile: true,
    useCache: true,
    useHash: false,
    useMap: true,

    domain: '',
    requires: [],
    links: [],
    asyncs: [],
    derived: [],
    extras: {},

    _isImage: false,
    _isText: true,
    _likes: { isHtmlLike: false, isJsLike: true, isCssLike: false },

    charset: 'utf8',
    release: '/static/foo.js',
    url: '/static/foo.js',

    id: 'static/foo.js',
    moduleId: 'static/foo'
}
```


# id vs moduleId

moduleId是前端用的（可以联想 mod.js）
id是后端用的

```js
module.exports = function (content, file, conf = {}) {
    file.id = 'id-' + file.id;
    file.moduleId = 'moduleId-' + file.moduleId;
}
```

```js
{
	res: {
		'id-src/abc.js': {
			uri: '/src/abc.js',
			type: 'js',
			extras: {
				moduleId: 'moduleId-src/abc',
			},
		},
	},
	pkg: {},
}
```
