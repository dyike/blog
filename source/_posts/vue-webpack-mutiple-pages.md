---
title: WebPack配置vue多页面
date: 2018-01-03 22:53:57
tags: vue
---

WebPack虐我千百遍，我带她如初恋。一个项目前台页面写差不多了，webpack几乎零配置，也算work起来了。现在需要编写后台管理界面，另起一个单独的项目，那是不存在的。于是网上了搜了大把大把的文章，很多都是修改了项目的结构，讨厌，vue-cli搞的那一套，干嘛要修改来修改去的。像我这种前端小萌新，webpack的配置改着就把前台部分run不起来了。。。

于是就有了这个笔记：


先看看项目的结构：

```bash
├── build
├── config
├── src
│   ├── api
│   ├── assets
│   ├── components
│   ├── pages
│   ├── router
│   ├── utils
│   ├── vuex
│   ├── App.vue
│   ├── main.js
│   ├── admin.js
│   └── Admin.vue
├── static
│   └── images
├── README.md
├── admin.html
├── index.html
├── package.json
└── yarn.lock
```

我相信这样的结构大家一定很熟悉，除了`admin.html`和src文件夹下面的`Admin.vue`、`admin.js`，还有一些api，pages，vuex等文件夹，就是最常见的一个vue-cli初始化的项目结构。


我想要就是新增一个后台管理界面的入口admin.html，其他能够共用的还是共用，进入正题：

## 修改webpack的配置文件

### 修改 webpack.base.conf.js

打开 `~\build\webpack.base.conf.js` ，找到entry，添加多入口:
```js
  entry: {
    app: './src/main.js',
    admin: './src/admin.js'   //新增
  },
```

这样运行编译的时候，每一个入口都会对应一个chunk。

## run dev配置的修改

打开 ·~\build\webpack.dev.conf.js· ，在plugins下找到`HtmlWebpackPlugin`，在其后面添加对应的多页，并为每个页面添加Chunk配置如下:

```js
    new HtmlWebpackPlugin({
      filename: 'index.html', //生成的html
      template: 'index.html', //来源html
      inject: true,   
      chunks: ['app']//需要引入的Chunk，不配置就会引入所有页面的资源
    }),
    new HtmlWebpackPlugin({
      filename: 'admin.html',
      template: 'admin.html',
      inject: true,
      chunks: ['admin']
    }),
```

## run build配置的修改

### 修改config/index.js

打开`~\config\index.js`，找到build下的`index: path.resolve(__dirname, '../dist/index.html')`，在其后添加多页:

```js
admin: path.resolve(__dirname, '../dist/admin.html'),
```

### 修改`webpack.prod.conf.js`

打开`~\build\webpack.prod.conf.js`，在plugins下找到`HtmlWebpackPlugin`，在其后面添加对应的多页，并为每个页面添加Chunk配置:

```js
    new HtmlWebpackPlugin({
      filename: config.build.index,
      template: 'index.html',
      inject: true,
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
        // more options:
        // https://github.com/kangax/html-minifier#options-quick-reference
      },
      // necessary to consistently work with multiple chunks via CommonsChunkPlugin
      chunksSortMode: 'dependency',
      chunks: ['manifest', 'vendor', 'app']
    }),

    new HtmlWebpackPlugin({
      filename: config.build.admin,
      template: 'admin.html',
      inject: true,
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
      },
      chunksSortMode: 'dependency',
      chunks: ['manifest', 'vendor', 'admin']
    }),
```

## End

恩，没有了，就不修改什么项目结构了，过程越复杂越容易出错。上面webpack的配置简单能看懂。Enjoy~~~
