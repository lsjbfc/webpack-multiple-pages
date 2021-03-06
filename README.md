
# Webpack Multiple Pages

**注：目前侧重于移动端支持，调整webpack配置后可支持PC端**

- - -

### 要解决的问题

1. 资源的拆分与缓存的利用

	通常webpack会把所有公共代码打成一个大包，多页面情况下，对公共部分的使用情况稍微复杂，不是每个页面都需要用到完整的公共包

	常见的方法是：每一个入口html会引用整个大包，但对于未完全使用大包的入口来说，引入了多余的资源，浪费性能

	而且如果大包中有频繁变动的业务公共代码，可能大包中1%的代码发生了变动，缓存失效会导致剩余的99%需要重新下载

	解决的办法是把大包有计划地拆成多个小包（如可配置地单独打包react相关资源、helpers资源），每个页面按自己的需求有针对性地引用，可以大致解决上面的问题

2. 入口资源按需引用

	但最后一步中“每个页面按自己的需求有针对性地引用”，webpack中无法直接得知每个资源之间的引用关系，html-webpack-plugin为各入口生成html时，需要手动配置html的依赖资源，这显然不够智能

	所以花了一些时间来研究webpack运行过程，检查文件之间的依赖关系如何保存，催生了一个提取依赖关系的插件（`build/utils/auto-inject-plugin`），目前只针对于当前方案。

3. 如何配置多入口

	一般采用配置表来指定入口，较为麻烦

	此处采用了`约定大于配置`的方案，以约定好的路径命名的方式来自动采集入口

- - -

### 特性介绍

1. 自由的代码分割，在`build/entries.js`文件中配置代码分割

	- 多页面（*project*），每个页面一个js
	- 不同第三方库（*lib*）可定制化打包成某个js
	- 支持多个公共业务库（*vendor*），每个库一个js
	- 使用了[`AutoDllPlugin`](https://github.com/asfktz/autodll-webpack-plugin)功能加快编译速度，**但dll文件可能会对lib文件依赖造成影响，未了解其中原理需慎用，在不明确**

2. 稳定的模块版本控制，各模块改动时，尽可能不影响其他模块的hash值，保证对资源缓存的最大利用

3. 支持多框架共存（目前内置了[`react`](https://reactjs.org/)、[`vue`](https://cn.vuejs.org/)），如A页面用react，B页面用vue，并引入了常用的工具，如下

	- react：
		- [`react-router`](https://github.com/ReactTraining/react-router)
		- [`antd-mobile`](https://github.com/ant-design/ant-design-mobile)
			- 按需打包
			-	支持主题自定义，主题配置文件为`build/utils/antd-mobile/default.less`
			- 默认高清方案
		- [`redux`](http://www.redux.org.cn/)（集成了以下中间件）
			-	[`redux-thunk`](https://github.com/gaearon/redux-thunk)
			- [`redux-promise`](https://github.com/redux-utilities/redux-promise)
			- [`redux-actions`](https://github.com/redux-utilities/redux-actions)
			- [`redux-persist`](https://github.com/rt2zz/redux-persist)
		- [`immutable`](http://facebook.github.io/immutable-js/docs/#/)
		- [`mobx`](http://cn.mobx.js.org/)

	- vue：
		- [`vue-router`](https://router.vuejs.org/zh-cn/)
		- [`mint-ui`](http://mint-ui.github.io/#!/zh-cn)（按需打包）
		- [`vuex`](https://vuex.vuejs.org/zh-cn/)

4. [`eslint`](http://eslint.cn/)代码校验，[`prettier`](https://prettier.io/)格式化（commit时自动格式化）

5. 内置了[`babel-polyfill`](https://babeljs.io/docs/usage/polyfill/)垫片库，支持es各种新特性（装饰器、async/await等）

6. 样式支持（抽取成独立的css文件）
	
	- 业务样式支持scss、sass、css
	- 第三方样式支持css、less
	- 支持[`cssModule`](http://www.ruanyifeng.com/blog/2016/06/css_modules.html)，需要使用.m.css、.m.scss、.m.sass文件后缀

7. 引入[`postcss`](https://github.com/postcss/postcss/blob/master/README.cn.md)，启用以下插件

	- [`cssnext`](http://cssnext.io/)支持最新css语法，内置了[`autoprefixer`](https://github.com/postcss/autoprefixer)，兼容浏览器为 [`[iOS > 7, Android >= 4.0]`](https://github.com/ai/browserslist#queries)

	- [`cssnano`](http://cssnano.co/)对样式文件进行压缩

	- [`postcss-combine-duplicated-selectors`](https://github.com/ChristianMurphy/postcss-combine-duplicated-selectors)对样式或属性进行合并

	- [`postcss-viewport-units`](https://github.com/springuper/postcss-viewport-units)配合[`viewport-units-buggyfill`](https://github.com/rodneyrehm/viewport-units-buggyfill)修复浏览器对`vw`、`vh`、`vmin`、`vmax`的兼容性

	- [`postcss-px-to-viewport`](https://github.com/evrone/postcss-px-to-viewport)自动将px转换为vw以支持高清方案（未全局开启）

	- [`postcss-write-svg`](https://github.com/jonathantneal/postcss-write-svg)支持css内编写svg，用以实现[1px-border](https://www.w3cplus.com/css/fix-1px-for-retina.html)

8. 集成了部分常用工具：[`lodash`](https://lodash.com/)、[`lodash/fp`](https://github.com/lodash/lodash/wiki/FP-Guide)、[`axios`](https://github.com/axios/axios)、[`date-fns`](https://date-fns.org/)、[`md5`](https://github.com/pvorb/node-md5)、[`fastclick`](https://github.com/ftlabs/fastclick)

9. 高清方案方面，[`flexible.js`](https://github.com/amfe/lib-flexible)官方推荐使用viewport方案代替rem，参考[《如何在Vue项目中使用vw实现移动端适配》](https://www.w3cplus.com/mobile/vw-layout-in-vue.html)，目前脚手架中已经**默认启用了此方案，如使用px单位会自动转为vw，设计稿尺寸为750px**，原`flexible.js`方案可使用但未开启

10. 默认采用根目录下的`template.html`作为各入口html模板，可在入口目录下使用`index.html`文件自定义html，需要注意保留资源的动态注入模板
- - -

### 业务入口约定

- 业务代码存放在`src`目录中，每个入口文件夹名称必须以`空格@`标识结尾，入口文件必须为`index.js`

- `src/project`中为多页面入口代码

- `src/vendor`中为业务公共部分代码

- - -

### 开发构建启动方式

1. 使用以下命令进行开发构建

```bash
npm start
```

2. 启动服务器后，浏览器打开`localhost:10001/wmp/任意页面名` 来进行浏览

	例如 [`localhost:10001/wmp/reduxCouterDemo`](http://localhost:10001/wmp/reduxCouterDemo)

- - -

### 编译方式

1. 使用以下命令进行开发构建

```bash
npm run build
```

2. 若要启动dist服务器，使用以下命令，服务器默认运行在10000端口

```bash
npm run server:dist
```
- - -

### 开发时构建的提速说明

1. `start`命令中包含了`dll`命令，作用是尽可能将使用到的第三方资源打包成静态包，加快开发构建的速度

	所以在第三方资源不变动的情况下，`start`命令只需执行一次，后续开发构建的启动直接使用以下命令

	```bash
	npm run dev
	```

	如果明确得知第三方资源发生了变动，需再执行一次`start`命令（不执行也可，不过`dev`速度会受到影响）

2. 我们在开发多页应用时，一般来说不会同时关注所有页面的开发工作，一次只会负责一个或数个页面的开发

	针对这个特性，我们在开发构建时可以指定（或忽略）部分入口以加快构建速度，具体操作如下

	在根目录下创建名为`__entry.config.js`的文件（此文件被git忽略），文件内容示例为：

	```javascript
	module.exports = {
	  include: [
	    /demo/
	  ],
	  exclude: [
	    /vue/
	  ]
	}
	```

	`include`为指定包含的入口，`exclude`为需要排除的入口，每个项的内容为`匹配入口路径的正则表达式`
