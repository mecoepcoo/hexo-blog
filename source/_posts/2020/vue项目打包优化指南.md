---
title: vue项目打包优化指南
date: 2020/06/22 23:55:00
updated: 2020/06/22 23:55:00
categories: 
- Vue
tags: 
- vue
- 打包
- 优化
- webpack
---

# 简介
vue-cli是一个非常好用的vue项目脚手架生成工具，当一个项目从开发环境上线到生产环境时，往往会遇到一些问题：为什么打包出来的文件有2Mb大？为什么页面上的资源加载非常缓慢？为什么有毫不相关的代码被打包到了同一个文件？本文主要针对vue-cli生成的项目，在打包过程中做一些优化。

<!-- more -->

本文发布时vue-cli的版本为v4。

# 按需加载路由
如果不做额外的配置，当打开项目的首页时，项目就会同时加载许多与当前页面无关的代码，这影响了我们的首页加载速度，按需加载的路由（懒加载）可以解决这个问题。我们让webpack自动识别，把不同的路由组件打包成分开的文件。

得益于vue-cli的封装，我们只需要这样写路由就可以实现按需加载的路由：
```javascript
const routes = [
  {
    path: '',
    redirect: '/home',
  },
  {
    path: '/home',
    name: 'home',
    component: () => import('@/views/home'),
    meta: { title: '首页' }
  }
  {
    path: '/about',
    name: 'about',
    component: () => import('@/views/about'),
    meta: { title: '关于' }
  },
]
```

# 异步组件
异步组件的原理跟按需加载路由类似，我们同样只需要以特定的方式书写代码：
```javascript
export default {
  name: 'MyPage',
  components: {
    test: () => import('./test') // 异步引入组件，让webpack将代码分割打包
  },
}
```

# 分析工具
要解决打包文件过大的问题，先要分析出到底是什么原因导致了文件大，哪些地方可以减小打包大小。

vue-cli中内置了 `webpack-bundle-analyzer` 插件，默认情况下，我们修改 `package.json` 中的打包命令为 `npm run build --report` 即可查看可视化打包分析：
```json
{
  "scripts": {
    "build": "vue-cli-service build --report",
  },
}
```

```bash
$ npm run build
```

执行命令后将会在 `dist` 目录中多生成一个 `report.html` 文件，把它在浏览器中打开，即可看到分析：

{% asset_img an.png pic %}

从图中可以看出，moment这个库占据了大量无用的空间。借助这个工具，我们可以分析是哪些文件占用了空间。

# 按需打包Moment.js
`moment.js` 占用空间大的原因在于，moment中包含了大量语言资源文件，我们并不需要这些。

通过webpack自身的功能即可在打包时丢弃这些无用的内容：

```javascript
// 在项目根目录新建 vue.config.js
const webpack = require('webpack')
module.exports = {
  configureWebpack: config => {
    const plugins = [
      // 只保留中文语言资源
      new webpack.ContextReplacementPlugin(/moment[/\\]locale$/, /zh-cn/),
    ]
  },
}
```

# 按需打包lodash
`lodash` 也是前端常用的一个工具库，有两种方案可以减小它的打包体积。

首先我们这么写代码，然后选择下面任意一种方法完成优化：
```javascript
import { cloneDeep } from 'lodash'

const a = { k: 'v' }
const b = cloneDeep(a)
```

## 使用lodash-es（推荐）
如果使用es模块，那么可以直接安装 `lodash-es`：

```bash
$ npm i lodash-es
```

## lodash-webpack-plugin插件
也可以使用 `lodash-webpack-plugin` 插件，自动移除没有用到的lodash代码：

> 注意：这个插件在使用 `find` 等方法时可能会删除一些必要的依赖，导致程序报错，解决方法是自己补全依赖，或者换用 `lodash-es`

```javascript
// vue.config.js
const LodashWebpackPlugin = require('lodash-webpack-plugin')

module.exports = {
  configureWebpack: config => {
    const plugins = [
      // 加入这条
      new LodashWebpackPlugin(),
    ]
  },
}
```

# 压缩代码
新版本的vue-cli是自动压缩代码的，所以不需要额外的配置。如果使用旧版cli生成项目，可以查询 [terser](https://github.com/terser/terser) 相关的用法。

> `uglifyify.js` 已经停止维护了，不要再使用这个库

# 生成gzip文件
nginx可以帮助我们将资源自动压缩为gzip文件，如果我们在打包时能够提供一份压缩好的gzip文件的话，传输会更快。

这里使用 `compression-webpack-plugin` 插件：

```bash
$ npm i -D compression-webpack-plugin
```

编辑webpack配置：
```javascript
// vue.config.js
const CompressionWebpackPlugin = require('compression-webpack-plugin')
const productionGzipExtensions = ['js', 'css']

module.exports = {
  configureWebpack: config => {
    const plugins = [
      new CompressionWebpackPlugin({
        filename: '[path].gz[query]',
        algorithm: 'gzip',
        test: new RegExp('\\.(' + productionGzipExtensions.join('|') + ')$'),
        threshold: 10240,
        minRatio: 0.8
      }),
    ]
    // 仅在生产环境压缩，提高开发时的编译速度
    if (process.env.NODE_ENV === 'production') {
      config.plugins = [...plugins, ...config.plugins]
      config.optimization = {
        splitChunks: {
          cacheGroups: {
            // 提取公共模块
            commons: {
              chunks: 'all',
              minChunks: 2,
              maxInitialRequests: 5,
              minSize: 0,
              name: 'common'
            }
          }
        },
      }
    }
  },
}
```

> 注意：gzip还需要服务端支持，一般在nginx上做配置
