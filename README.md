# bifypack

基于[gulp](http://gulpjs.com/)和[browserify](http://browserify.org/)的项目构建工具。

## 安装使用

### 全局安装：

```
npm install -g bifypack
```

然后在项目根目录下创建一个`bifypackfile`的`js`文件，如同`gulp`的初始化`gulpfile.js`一样，该文件就是构建配置文件。

有了配置文件，然后就是执行了，`bifypack`是依赖于`gulp`的，也是基于任务式的，但是非编程。目前执行很简单：

```
bifypack <task>
```

### 本地安装

```
npm install bifypack --save-dev
```

然后在`package.json`中增加如下`script`：

```
"scripts": {
  "dev": "bifypack reload",
  "build": "bifypack rev"
}
```

一般场景都适用，开发执行`npm run dev`，上线执行`npm run build`即可。

下面就看默认提供的`task`。

## task

基础`task`：

* `eslint`: 利用[eslint](https://github.com/eslint/eslint)做语法检查，规则文件就是项目根目录下的`.eslintrc`文件，参考<http://eslint.org/docs/user-guide/configuring#using-configuration-files>

* `browserify`: [browserify](http://browserify.org/) task，主要根据配置文件中指定入口文件转为可以在浏览器中执行的代码，如果配置了`bundle`，还会根据规则利用[factor-bundle](https://github.com/substack/factor-bundle)插件生成公共代码。

* `watchify`: 利用[watchify](https://www.npmjs.com/package/watchify)提升`browserify`编译效率。

* `clean`: 清空配置文件中指定的`product`目录内容。

* `cleanDev`: 清空配置文件中指定的`dev`目录内容。

* `html`: 主要把配置的`html`文件复制到配置的`dev`目录下；同时这个过程还会将`placeholder`中的规则做替换生成标签插入到html中，还会把利用[factor-bundle](https://github.com/substack/factor-bundle)插件生成公共代码插入到html中。

* `img`: 主要是将配置的`img`资源复制到配置的`dev`目录下（注：1.x的版本会利用[gulp-imagemin](https://www.npmjs.com/package/gulp-imagemin)来压缩图片，2.x的则不会）。

* `style`: 将指定的入口css文件利用[gulp-minify-css](https://www.npmjs.com/package/gulp-minify-css)压缩（注：2.x的版本如果自己设置了`plugins`，那么则不会进行压缩），然后复制到配置的`dev`目录下，同时会增加`.map`文件，便于开发调试。一般会配合用于将html文件中的占位符替换为`<link rel="stylesheet">`标签。

* `script`: 将指定的浏览器可以直接执行的代码复制（合并）到配置的`dev`目录下，一般会配合用于将html文件中的占位符替换为`<script>`标签。

* `copy`: 复制文件到配置的`dev`目录下。

集成`task`：

* `static`: 也是默认`default`的`task`，执行`browserify`, `html`, `style`, `img`, `script`, `copy`任务task。

* `watch`: 主要watch执行`html`, `style`, `img`, `copy`任务，以及`watchify`任务。

* `reload`: 主要利用`watch`task和[browser-sync](http://www.browsersync.io/)实现便利开发。

* `rev`: 会先执行`static`任务，把文件编译到指定`dev`目录，然后将指定`dev`目录下的文件利用[gulp-rev-all](https://www.npmjs.com/package/gulp-rev-all)给文件名增加hash值且替换引用地址；将最终结果编译到指定的`product`目录。

对于一般开发而言，只需要在开发时执行`bifypack reload`，就可以很方便的开一个服务进行开发了。如果需要增加hash值，编译至指定`product`目录的话，则只需要执行`bifypack rev`就好了。

## bifypackfile

这个是`bifypack`的配置文件，这里主要来看一个示例：

```js
var path = require('path')

// 插件们
var less = require('gulp-less')
var LessPluginCleanCSS = require('less-plugin-clean-css')
var htmlmin = require('gulp-htmlmin')
var imagemin = require('gulp-imagemin')
var cleanCSSPlugin = new LessPluginCleanCSS({
	advanced: true,
	compatibility: 'ie8'
})

var src = 'src/'
var dev = 'dev/' // 加入到 .gitignore
var product = 'dist/' // 加入到 .gitignore
var bundleMapFile = 'bifypack.bundleMap.json' // 加入到 .gitignore
var externals = {
	jquery: '$'
}
var sexts = ['gif', 'png', 'jpg', 'jpeg'].join(',')

var config = {
	src: src, // 源目录 默认 src/
	dev: dev, // 开发目录 默认 dev/
	product: product, // 生产目录 默认 dist/
	bundleMapFile: bundleMapFile, // 默认bifypack.bundleMap.json，这个比较特殊，如果说利用factor-bundle插件生成公共代码文件的话，才会生成这个文件，主要是为了在html中增加新生成的这个公共代码文件的`<script>`标签

  browserSync: {
    // 可以省略不写 默认 { server: config.dev }
    // 详细的参数配置见 https://browsersync.io/docs/options
    // 当前依赖 browsersync 的版本为 2.15.0
    // 例如 普遍的场景可能会有 port proxy open 等
    port: 3555, // 端口
    open: true, // 是否默认打开浏览器
    proxy: 'localhost/path/' // 代理配置
  },

	rev: {
		manifest: true, // 是否在product目录生成 rev-manifest.json 文件
		// 正则 如果是字符串的话 会转成正则
		dontRenameFile: [/\/index\.html$/ig, '.map', '.jpg', '.jpeg', '.png', '.gif'], // gulp-rev-all的配置，这些文件不重命名
		prefix: { // 根据扩展名的前缀处理
			'.js': 'http://sj.domain.com/',
			'.css': 'http://sc.domain.com/',
			'.{jpg,jpeg,png,gif}': 'http://si.domain.com/',
		}
		/**
		 * prefix 还可以是
		prefix: 'http://s.domain.com/'
		 */
	},
	script: { // js相关

		eslint: ['pages/**/*.js', 'common/**/*.js', 'components/**/*.js'], // 检查

		browserify: { // browserify配置

			src: ['pages/home/*.js'], // 不考虑利用 factor-bundle 来生成公共文件的入口文件

			bundle: { // factor-bundle 生成公共文件 apps/xx/common.js
			 	'apps/xx/common.js': 'apps/xx/*/*.js'
			},

			// 两个用途：
			// 1. browserify.externals（获取externals的key）
			// 2. exposify（内置的） transform的expose参数，用于将externals指定require的key替换为全局的对应的值
			//    也就是说，此时的例子，源码：`var $ = require('jquery')`转换的结果为`var $ = (typeof window !== "undefined" ? window['$'] : typeof global !== "undefined" ? global['$'] : null)`
			externals: externals,

			/**plugins和transforms都需要增加到自己项目的依赖中**/
			// https://www.npmjs.com/browse/keyword/browserify-plugin
			plugins: [ // 插件 内置的插件是bundle-collapser
				'xx-plugin',
				// 还可以：
				{
					name: 'xx',
					options: {
						xxx: 'xxx'
					}
				}
			],

			// https://github.com/substack/node-browserify/wiki/list-of-transforms
			transforms: [ // transforms 已经内置了exposify这个transform
				{
					name: 'node-lessify', // 这个是dolymood/node-lessify版本，主要解决插入到页面上的css中的图片路径问题，如果是绝对路径则使用默认node-lessify即可
					options: {
						root: path.join(process.cwd(), src), // dolymood/node-lessify的配置
						prefix: '/' // dolymood/node-lessify的配置
					}
				},
				'partialify' // partialify transform，注意和其他transform的冲突，例如 cssify browserify-css 等
			]

		},

		normal: { // 正常的通过script任务执行的，主要操作是复制，合并，重命名
			'lib/jquery.js': '!node_modules/jquery/dist/jquery.j', // 注意这里用了! ，当后边跟着的是确定的一个地址的时候才可以使用，也就是说 !xx/*.js 是不可以的
			'lib/jquery-ui.js': 'lib/jquery-ui/*.js',
			'*': 'lib/avalon/*.js' // 如果key是*的话，那么就会按照原有目录、文件名输出
		}

	},

	placeholder: { // html中的占位符替换规则，替换为对应的script标签或者link标签，都为js或css的各个入口文件
		script: {
			// <!--SCRIPT_LIB_PLACEHOLDER--> 的结果就是 加入四个script标签，相对地址会做处理
			'<!--SCRIPT_LIB_PLACEHOLDER-->': [
				'lib/jquery.js',
				'lib/jquery-ui.js',
				'lib/avalon.js',
				'lib/avalon.ui.js'
			]
		},
		style: {
			// <!--STYLE_COMMON_PLACEHOLDER--> 的结果就是 加入n个link标签，相对地址同样会做处理
			'<!--STYLE_COMMON_PLACEHOLDER-->': [
				'lib/**/*.css',
				'common/common.css'
			]
		}
	},
  copy: ['font/*.*'],

  // 注：
  // 2.x 起新增插件机制
  // 关于 html style img 三个配置可以配置为如下形式 {src: ['xx'], plugins: []}
  // 还可以是传统的 1.x 的配置方式，例如：
  // html: ['**/*.html']
  // 同样的 style 和 img 也一样可以：
  // style: ['common/css/g.css']
  // img: ['components/**/*.{' + exts + '}']
  html: {
		src: ['**/*.html'], // html tasks
		plugins: [
			function (styleConfig, config, utils) {
				return htmlmin({
					collapseWhitespace: true
				})
			}
		]
	},
	style: {
		src: [
			'common/css/g.less',
			'app/**/*.less'
		],
		plugins: [
			function (styleConfig, config, utils) {
				return less({
					plugins: [cleanCSSPlugin]
				})
			}
		]
	},
	img: {
		src: [
			'components/**/*.{' + exts + '}',
			'app/**/*.{' + exts + '}'
		],
		plugins: [
			function (imgConfig, config, utils) {
				return imagemin()
			}
		]
	}
}

module.exports = config

```

## 后续

目前只是支持基础的一些配置，还有额外的很多配置；目前还可以通过自建额外`task`的方式进行增强：

1. 如果是全局安装了`bifypack`的话，需要在项目根目录下执行`npm link bifypack`；如果是本地安装则不需要执行这一步了。

2. 在某目录下修改原有的task

```js
// 放在tasks目录下的其中task文件
var marked = require('gulp-marked')
var bifypack = require('bifypack')
var gulp = bifypack.gulp
var utils = bifypack.utils
gulp.task('marked', ['static'], function () {
	return gulp.src(utils.getSrc(bifypack.config.marked))
		.pipe(marked())
		.pipe(gulp.dest(bifypack.config.dev))
})
```

如果你选择了全局安装，那么直接执行`bifypack marked -e tasks`即可（`tasks`也就是任务目录名）；

如果你选择的是本地安装，那么则需要在`package.json`中增加如下`script`：

```
"scripts": {
  /** 其他script **/
  "marked": "bifypack marked -e tasks"
}
```
