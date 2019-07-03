>偶尔有一天觉得公司的 H5 项目在 build 的时候特别慢，看了下时间大约在 60 - 80s，而且由于需要在微信中看真机效果，所以需要经常 build，于是去网上找找资料如何缩短这个加载时间。以下是我在优化加载过程中遇到的一些坑及优化方案。 

![vue-build.png](https://upload-images.jianshu.io/upload_images/1394028-7b8f238b31e95e6b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![网络请求.png](https://upload-images.jianshu.io/upload_images/1394028-07c182d14e26c6ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从上图中我们发现编译耗时接近 80s，而且仔细看还发现 vendor.js 和 app.css 异常的大，这不仅导致我们在编译阶段很慢，而且首屏响应也会变慢，怎么去解决呢？通过查了些资料我们开始着手优化起来~

## 优化方案
+ 服务端开启 gzip 压缩
+ webpack 配置优化
+ 按需加载第三方组件

### 项目配置
---
该 H5 商城基于 vue-cli 脚手架构建，引入 vue-router 、vuex、axios 搭建 SPA 应用，UI 工具库选用的是 iView、mint-ui，以及不少第三方组件。

### 服务端 Nginx 开启 gzip
这个很重要，尽管我们现在已经做了，但是需要提醒的是服务端 Nginx 开启 gzip 压缩能大幅度提高页面加载速度，前端需要配置 compression-webpack-plugin 插件。这里注意根据  webpack 版本来选择 compression-webpack-plugin 插件版本，webpack4 使用 2.x ，webpack4 以下版本选择 1.x
+ config/index.js
```js
module.exports = {
  build: {
        ...
        // Gzip off by default as many popular static hosts such as
        // Surge or Netlify already gzip all static assets for you.
        // Before setting to `true`, make sure to:
        // npm install --save-dev compression-webpack-plugin
        productionGzip: true,  // 这里开启 gzip，默认为false
        productionGzipExtensions: ['js', 'css']   // 只压缩 js css 不压缩图片 
  }
}
```

+ build/webpack.prod.conf.js
```js
if (config.build.productionGzip) {
    const CompressionWebpackPlugin = require('compression-webpack-plugin')
    //增加浏览器CPU（需要解压缩）， 减少网络传输量和带宽消耗 （需要衡量，一般小文件不需要压缩的）
    webpackConfig.plugins.push(
        new CompressionWebpackPlugin({
            asset: '[path].gz[query]',
            algorithm: 'gzip',
            test: new RegExp(
                '\\.(' +
                config.build.productionGzipExtensions.join('|') +
                ')$'
            ),
            threshold: 10240,               // 达到10kb的静态文件进行压缩 按字节计算
            minRatio: 0.8                   // 只有压缩率比这个值小的资源才会被处理
        })
    )
}
```
![开启双端gzip后效果1.png](https://upload-images.jianshu.io/upload_images/1394028-c1c1fb7bc3370b9e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![开启双端gzip后效果2.png](https://upload-images.jianshu.io/upload_images/1394028-ec7efb94c57b446b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由图中可以看出，这里配置完后首屏加载速度提升是很明显的，``vendor 由 176kb 压缩成61kb，app.css 由 1.22M 压缩成186kb``，接下来我们可以看看如何通过配置 webpack 优化打包速度。

###  1、使用 webpack-parallel-uglify-plugin 插件来压缩代码
vue 默认使用 UglifyJsPlugin 插件进行代码压缩，由于是单线程，压缩效率较低；我们可以使用 webpack-parallel-uglify-plugin 插件，可以合理使用 CPU 资源，通过设置缓存能够大大减少非首次构建时间
```js
const ParallelUglifyPlugin = require('webpack-parallel-uglify-plugin');
    ...
    // 注释 webpack 提供的 UglifyJS 插件
    //new UglifyJsPlugin({
    //  uglifyOptions: {
    //    compress: {
    //      warnings: false
    //    }
    //  },
    //  sourceMap: config.build.productionSourceMap,
    //  parallel: true
    //}),
    // 增加 webpack-parallel-uglify-plugin 来替换
    new ParallelUglifyPlugin({
      cacheDir: '.cache/',
      uglifyJS:{
        output: {
          comments: false
        },
        compress: {
          drop_debugger: true, // 去除生产环境的 debugger 和 console.log
          drop_console: true
        }
      }
    })
```
![webpack-parallel-uglify-plugin.png](https://upload-images.jianshu.io/upload_images/1394028-d81b7daa830f798c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

此时编译发现打包时间缩短了不少，不错，总算看到点希望，接下来我们来尝试下缩小打包后的文件大小。


### 2、减少打包文件数量
随着项目的迭代，引入的第三方库也会越来越多，每次 build 都会对所有文件进行打包，耗时也会越来越久，不利于平时开发。这个时候我们可以通过减少一些不经常更新的包来单独打包，我们有两种方式，第一种是通过 CDN 引入包然后配合 externals 配置的方式也可以，抽离第三方包，只打包业务代码。第二种方式就是引入 DLL 文件，dllplugin 是 webpack DllPlugin、webpack 的简称，思路也是讲一些第三方包抽离出来单独打包，后面就不需要重新打包了。

使用方法：
+ 1、在 build 文件夹下新建 `webpack.dll.conf.js`，内容如下：
```js
const path = require('path')
const webpack = require('webpack')

module.exports = {
    entry: {
        //这地方写你想抽离的包，可以参考你的package.json文件下的 dependencies
        vendor: [
            'vue/dist/vue.common.js',
            'vue-router',
            'vuex'
        ]
    },
    output: {
        //这地方写你打包后生成文件的路径
        path: path.join(__dirname, "../dist/dll/"),
        filename: '[name].dll.js',
        library: '[name]_library'
    },
    plugins: [
        //这个插件是重点，用于打包上面entry里配置的包
        new webpack.DllPlugin({
            path: path.join(__dirname, '../dist/dll', '[name]-manifest.json'),
            name: '[name]_library'
        }),
        new webpack.optimize.UglifyJsPlugin()
    ]
}

```
这里我们先抽离出 ``vue，vue-router 和 vuex``，然后在 package.json 中 scripts 
下加入该命令  ``"build:dll": "webpack --config build/webpack.dll.conf.js"``，执行 ``yarn build:dll`` 命令将第三方依赖打包

![yarn dll.png](https://upload-images.jianshu.io/upload_images/1394028-e2432364484b9873.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到在 dist 目录下生成了一个 dll 文件夹，里面有 ``vendor-manifest.json 和 vendor.dll.js`` 这两个文件，我们打开 ``build/webpack.base.conf.js`` 文件，编辑添加如下配置
```js
const webpack = require('webpack');
module.exports = {
    ...
    plugins: [
        new webpack.DllReferencePlugin({
            manifest: require('../dist/vendor-manifest.json')
        })
    ]
    ...
}
```
接着我们只要在 index.html 中引入 ``vendor.dll.js`` 即可
```html
<body>
    <div id="app"></div>
    <script src="./dist/dll/vendor.dll.js"></script>
</body>
``` 
接下来我们执行 ``yarn build`` 发现 ``vendor`` 文件由原来的 ``1.57M`` 变成了 `931KB`，看起来效果不错
![步骤一.png](https://upload-images.jianshu.io/upload_images/1394028-22441a0e69d8415c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里我们只是预编译了三个文件，当我们把其他的第三方库打包后看看效果如何

![步骤二.png](https://upload-images.jianshu.io/upload_images/1394028-13de483ca5c8ac6f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

vendor 文件现在只有 270 kb 了，缩小了不知道多少倍~ 到这里，dll 的配置基本完事了，我们还可以把 `index.html` 中的手动引入的脚步改成自动导入。
```js
// webpack.base.conf.js
new webpack.DllReferencePlugin({
    context: path.resolve(__dirname, '..'),
    manifest: require('../dist/vendor-manifest.json')
}),
// 在htmlwebpack后插入一个 AddAssetHtmlPlugin 插件，用于将 vendor 插入打包后的页面
new AddAssetHtmlPlugin({
    filepath: require.resolve('../dist/dll/vendor.dll.js'),
    includeSourcemap: false
})
```
> 一开始踩了很多坑，配置 dll 一直配不对，还是要细心~

然后删除 `index.html` 引入的脚本即可。接下来我们可以继续优化一下 css ，从我们的 main.js 文件可以看到全局引入了很多 css，我们可以通过 CDN 方式引入。

> 多说一句，cdn 引入其实原理和 dll 一样，但是我们知道每一个 cdn 资源也是需要走请求的，如果很多 cdn 的话还是要权衡一下，同理，dll 文件如果太大，会影响首屏加载，不能光图打包爽。

### CDN 引入全局样式和字体库
我们把 main.js 中一些可以从 cdn 上找到的样式和字体都通过 cdn 引入到 index.html 中，然后再打包

![首页.png](https://upload-images.jianshu.io/upload_images/1394028-abeab88a488cc3d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![app.css.png](https://upload-images.jianshu.io/upload_images/1394028-ca6144a6f1eebb16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


我们发现 css 文件也响应缩小了一些，最终效果如下图所示：

![end.png](https://upload-images.jianshu.io/upload_images/1394028-47635e62388b3bb8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![end.png](https://upload-images.jianshu.io/upload_images/1394028-b4556e0545d2c28b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 总结
从首次打包花了 80s 到现在 47s，优化了接近一半的时间，在研究 webpack 时也遇到不少问题，基本都是自己 Google ，不断的翻阅文档和相关 issues。有的插件在 webpack3.x 和 webpack4.x 不尽相同，没有做向下兼容，需要不断地踩坑，比如这次的 `add-asset-html-webpack-plugin` 在  webpack3.x 下只能使用 2.x 版本，以及昨晚的 `compression-webpack-plugin` 插件在 webpack3.x 下只能使用 1.x 版本，这些东西其实劳心劳力但是又没有什么用处，真心不建议跟 webpack 较劲。

项目优化其实只是一个下策，更重要的是我们平时写代码需要多思考，提升效率和性能。我们写的时候需要注意引入第三方组件以及样式，不是多处用到的组件尽量按需加载，外部样式走 CDN，公用逻辑尽量提出来，不要写完就完事了。就我们目前的商城，移动端组件库就有俩，查看请求时发现有几张图片都是 1.5M（编辑同学上传时没有压缩），首页第一次加载请求有200多个，主要是图片请求，这块如何优化呢？