---
title: 对 webpack hash 的理解
tags: [构建工具]
categories: [FrontEnd]
---

## 1、hash && chunkhash

`webpack` 提供了两种方式实现缓存

- 一种是为所有的 `chunks` 打上同样的 `hash` ，即编译 `hash`

- 一种是根据每个 `chunk`  的内容打上各自的 `hash`，即 `chunkhash`

```javascript
output: {
    path: 'output',
    filename: '[name].[hash].js'
    // or
    // filename: '[name].[chunkhash:4].js'
}
```

<!-- more -->

下面是很常见的 `webpack` 配置：

- 对 `js` 的 `[chunkhash]` 由 `webpack` 计算

- 图片/字体的 `[hash]` 由 `file-loader` 计算

- 提取的 `CSS` 的 `[contenthash]` 由 `extract-text-webpack-plugin`

```js
// production
output: {  
    filename: '[name].[chunkhash:8].js'
},
module: {  
    rules: [
        {
            test: /\.(jpe?g|png|gif|svg)$/i,
            loader: 'url-loader',
            options: {
                limit: 1000,
                name: 'assets/imgs/[name].[hash:8].[ext]'
            }
        },
        {
            test: /\.(woff2?|eot|ttf|otf)$/i,
            loader: 'url-loader',
            options: {
                limit: 10000,
                name: 'assets/fonts/[name].[hash:8].[ext]'
            }
        }
    ]
},
plugins: [  
    new ExtractTextPlugin('[name].[contenthash:8].css')
]
```

## 2、 不稳定的 `chunkhash`

编译 `hash` 是很稳定可信赖的，但是不能做持久化缓存；`chunkhash` 看样子是根据 `chunk` 内容改变 `hash` 的，但是也不一定可信

比如我们加上 `extract-text-webpack-plugin` 抽取 `css` 出来：

```javascript
/src
  |- pageA.js
  |- pageA.css

// pageA.js
require('./a.css');

// webpack.config.js
output: {
    filename: 'bundle.[chunkhash:4].js'
},

module: {
    loaders: [{
        test: /\.css$/,
        loader: ExtractTextPlugin.extract('style-loader', 'css-loader')
    }]
},
    
plugins: [  
    new ExtractTextPlugin('[name].[contenthash:8].css')
]
```

修改 `pageA.css` 前后构建结果如下：

![Alt text](/assets/img/webpack-hash-001.png)

`pageA.js` 这个 `chunk` 的输出在 `webpack` 看来是包括 `css` 文件的，只不过被我们抽取出来罢了，所以改 `css` 也就改了这个 `chunk` 的内容

## 3、 自定义 hash

解决这个问题可以使用 [webpack-md5-hash](https://github.com/erm0l0v/webpack-md5-hash/)：排序 `chunk` 的所有依赖模块，并将这些排序后的模块源代码拼接，最后用 `MD5` 拼接后内容的 `chunkhash`

PS：为什么要排序呢？因为比如一个模块 `b` 先后引入了 `c, d`，后来我们换了位置变成了 `d, c` 实际内容没有变所以 `hash` 也不应该改变的

```js
function compareMod(modA, modB) {
   var modAPath = getModFilePath(modA);
   var modBPath = getModFilePath(modB);
   return modAPath > modBPath ? 1 : modAPath < modBPath ? -1 : 0;
}

function getModSrc(mod) {
   return mod._source && mod._source._value || '';
}

compiler.plugin('compilation', function (compilation) {
   compilation.plugin('chunk-hash', function (chunk, chunkHash) {
       var source = chunk.modules.sort(compareMod).map(getModSrc).join('');
       chunkHash.digest = function() {
           return md5(source);
       };
   });
});
```

这样就能解决前面修改 `ExtractTextPlugin` 的问题，但是依然存在问题

## 4、发布版本不一致

有 `a.js, b.js, index.js` 如下：

```js
// a.js
require('./index')
var a = 1424;

// b.js
require('./index')

// webpack.config.js
new webpack.optimize.CommonsChunkPlugin({
    name: 'common',
    minChunks: 2,
    chunks: ['a', 'b'],
})
```
第一次编译：

![Alt text](/assets/img/webpack-hash-002.png)

修改 `a.js` 再次编译：

![Alt text](/assets/img/webpack-hash-003.png)

可以看见 `common.xx.js` 也跟着改变了，但是实际上我们仅仅修改了 `a.js` 不应该 `common.js` 也改变了才对

打开它的文件，发现这一段不一样：

```js
script.src = __webpack_require__.p + "" + chunkId + "." + {"0":"81a79c6cc1fd236fb9ae","1":"6a749561f8eb4005a925"}[chunkId] + ".js";

// 修改后
script.src = __webpack_require__.p + "" + chunkId + "." + {"0":"81a79c6cc1fd236fb9ae","1":"9ce6f42793d52c747c1a"}[chunkId] + ".js";
```

这个可以在 `webpack` 源码中找到原因，为什么 `common.xx.js` 的 `hash` 也发生变化了

```js
// compilation.js
for (i = 0; i < chunks.length; i++) {
    chunk = chunks[i];
    var chunkHash = require("crypto").createHash(hashFunction);

    // 根据chunk内容生成 chunkhash
    chunk.updateHash(chunkHash);
    // 这两句话用来生成要加密的信息
    // 对于入口文件，走的是 chunkTemplate
    // 对抽取的公共文件如上文的 common.js, 走的是 mainTemplate
    if (chunk.entry) {
        this.mainTemplate.updateHashForChunk(chunkHash, chunk);
	}
    else {
        this.chunkTemplate.updateHashForChunk(chunkHash);
    }

    // webpack-md5-plugin 就是在 chunk-hash 触发的
    this.applyPlugins("chunk-hash", chunk, chunkHash);

    chunk.hash = chunkHash.digest(hashDigest);

    hash.update(chunk.hash);
}

mainTemplate.plugin("hash-for-chunk", function(hash, chunk) {
    var outputOptions = this.outputOptions;
    var chunkFilename = outputOptions.chunkFilename || outputOptions.filename;

    // 这里会输出chunk、chunkhash对应的关系，拿上文的例子来说：
    // getChunkMaps() 得到一个对象
    // {
    //     hash: {
    //         0: 81a79c6cc1fd236fb9ae,
    //         1: 9ce6f42793d52c747c1a
    //     },
    //     name: {
    //         0: 'a',
    //         1: 'b'
    //     }
    // }
    // 因为我们修改了 a.js 内容发生变化，所以 hash 发生变化，从而上面的对象发生变化，所以 common.js 内容改变 hash 也因此变化
    if(REGEXP_CHUNKHASH_FOR_TEST.test(chunkFilename))
        hash.update(JSON.stringify(chunk.getChunkMaps(true, true).hash));

    if(REGEXP_NAME_FOR_TEST.test(chunkFilename))
        hash.update(JSON.stringify(chunk.getChunkMaps(true, true).name));
});
```

通过这个源码可以知道在生成 `chunkhash` 的过程中，`common.js` 依赖 `chunk` 的 `hash` 值，这里的例子就是 `a.js, b.js`，因为修改了 `a.js` 导致其 `chunkhash` 变化，从而导致 `common.js` 内容变化从而其 `hash` 变化，但是 `common.js` 的本质内容并没有发生变化

在使用前面的自定义 `hash` 插件重复上述操作之后，效果如下：

![Alt text](/assets/img/webpack-hash-004.png)

看样子 `hash` 是没有变化了，但是当我们打开 `common.js` 会发现内容是发生变化的：

![Alt text](/assets/img/webpack-hash-005.png)

但我们的 `chunkhash` 并没有改变，这就导致 `common.js` 可能不能上传到线上，导致线上的 `common.js` 版本依然是旧的，这样就会出错

## 5、解决方法

## 6、调试 webpack

- 下载 `webstorm` 

- 全局/项目安装 `npm install webpack-webstorm-debugger-script -g/--save-dev`

- 配置 `webstorm`

![Alt text](/assets/img/webpack-hash-007.png)

## refer

- [你用 webpack 1.x 输出的 hash 靠谱不](https://github.com/zhenyong/Blog/issues/1)
- [用 webpack 实现持久化缓存](https://sebastianblade.com/using-webpack-to-achieve-long-term-cache/)
- [webpack 源码分析](https://lihuanghe.github.io/2016/05/30/webpack-event.html)
- [如何十倍提升你的 webpack 构建效率](https://mp.weixin.qq.com/s?__biz=MzI2NzExNTczMw==&mid=2653284910&idx=1&sn=77f0675205bcb2265745b377a2c331d5&key=18e81ac7415f67c418b948acc5b6451858d3cb10a88c3627d82ddb8cdd4002391c24f5d91c9a0ce7c832edd4d6789982&ascene=4&uin=MjIzNzU4MjAw&devicetype=iPhone+OS9.3.2&version=16031312&nettype=WIFI&fontScale=100&pass_ticket=LMn24789re7A38N05l0uGSnLLAlBnEFoJ10B/eQLhR0=)
- [long term caching webpack1](https://webpack.github.io/docs/long-term-caching.html)
- [cache webpack2](https://webpack.js.org/guides/caching/#generating-unique-hashes-for-each-file)
