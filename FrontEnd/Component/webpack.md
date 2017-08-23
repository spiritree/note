# webpack3

## 什么是webpack？
![webpack.png](https://i.loli.net/2017/08/23/599d7a5b079d1.png)

webpack = module building system

在webpack看来，所有的资源文件都是模块(module)，只是处理的方式不同。

关于webpack的使用，其实和我们平时开发业务产品是一个道理。

产品需求 => 代码设计 => 提供API给开发者使用。

webpack解决的需求点就是如何更好的加载前端模块，webpack本身并不关心这个模块是什么，它只是调度配置文件中对模块处理的方式来完成这一切,而不必纠结文件类型。

比如我们会在项目中使用ES6/7/8的语法来编写JS代码，于是我们只需要配置babel-loader来转换到ES5从而实现兼容。

比如我们需要加载HTML文件获取HTML字符串，于是我们只需要配置raw-loader即可拿到对应文件的字符串。

比如我们需要将SASS/LESS文件预编译成CSS，于是我们只需要配置sass-loader/less-loader即可处理。

## webpack与gulp的区别？
**gulp 是 task runner，Webpack 是 module bundler。**

gulp 最核心的功能：

1. 任务定义和组织
2. 基于文件 stream 的构建
3. 插件体系

webpack 最核心的功能：

1. 按照模块的依赖构建目标文件
2. loader 体系支持不同的模块
3. 插件体系提供更多额外的功能

## webpack3优化实践
在用Vue-cli生成的项目中已经自带webpack配置文件，我们只需要运行`npm run build`就能打包文件，但是默认配置打包出来的文件通常过大，这就需要我们改动配置来优化。

### 图解打包文件
在`package.json`中加入如下脚本
```json
"scripts": {
    "analyze": "npm_config_preview=true  npm_config_report=true node build/build.js"
},
```
运行`npm run analyze`得到分析大小和分析图

![webpacktotal0.png](https://i.loli.net/2017/08/23/599d799d39782.png)

分析图中JS文件的大小

![webpacksize0.png](https://i.loli.net/2017/08/23/599d799d30e2a.png)

分析图

![webpackanalyze.png](https://i.loli.net/2017/08/23/599d799d458ba.png)

### 删去或替换代价大的模块

我们看到echart模块占了将近四分之一的体积，但是好像项目中并没有用到，于是定位并删除模块引用。

![echart.png](https://i.loli.net/2017/08/23/599d7a846831a.png)

还发现fontawesome图标库模块在打包过程中被webpack标记为big，先以`fa fa`关键词定位

![icon0.png](https://i.loli.net/2017/08/23/599d7a846765a.png)

在项目中查看这个图标的具体应用

![menucollapse0.png](https://i.loli.net/2017/08/23/599d799cd4852.png)

花费400KB+大小的图标库来使用一个图标有点得不偿失，寻找element自带图标替代方案

![menucollapse1.png](https://i.loli.net/2017/08/23/599d799ce5aaf.png)
![menucollapse2.png](https://i.loli.net/2017/08/23/599d799ce4aed.png)

替代完成后，删除import引用

![importfontawesome.png](https://i.loli.net/2017/08/23/599d7a8465aeb.png)

### GZIP压缩
gzip对于JS和CSS等文件压缩比很大，能进一步压缩用户访问服务器需要的体积

先安装压缩插件`npm install --save-dev compression-webpack-plugin`

在配置中设置`productionGzip: true`

![webpackgzip.png](https://i.loli.net/2017/08/23/599d873a15fc3.png)

### webpack3兼容性问题（JS压缩）

webpack2升级到webpack3的兼容性问题，JS压缩插件需要升级并重新在配置中应用

`npm install uglifyjs-webpack-plugin@beta --save-dev`

![webpackjscompress.png](https://i.loli.net/2017/08/23/599d8989c789a.png)

### 重新打包
`npm run analyze`或`npm run build`

![webpacktotal1.png](https://i.loli.net/2017/08/23/599d799d37624.png)

![webpack1.png](https://i.loli.net/2017/08/23/599d799d3abb3.png)

![webpackgzip1.png](https://i.loli.net/2017/08/23/599d799d26128.png)

**打包大小由3MB+ => 1.7MB => 500KB**