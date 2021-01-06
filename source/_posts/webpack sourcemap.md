---
title: Webpack SourceMap 配置
date: 2020-12-20 19:11:15
categories: 
  - coding
tags: 
  - sourcemap
  - webpack
---
最近项目中遇到了sourcemap不生效的问题,而且是有的项目生效,有的项目不生效, 经过一番查找资料,最终将问题解决,因此写篇文章以作备忘,也给后来者一个参考

## 关于webpack sourcemap
其实[官方文档](https://webpack.js.org/configuration/devtool/)已经写得比较详细了, 需要注意的点是：
1. 当`mode: production`时不能用 `Development` 模式的sourcemap, 否则文件名都不会显示出来, 我测试时在production模式下使用`eval-source-map`时就出现了这种情况。
2. `mode: production` 时推荐使用 `devtool: 'source-map'`。
3. `mode: development` 时推荐使用 `devtool: 'eval-source-map'`, 因为可以显示行号和源文件名,便于排查问题, 但这个模式初始化的时候要稍微慢一点。
<!-- more -->
## Typescript项目
对于typescript项目, 如果你用的`ts-loader`, 请务必注意两个地方:
1. tsconfig.json当中配置`sourceMap: true`, 以下为一个示例配置
```js
{
  "compilerOptions": {
    "outDir": "./dist",
    "module": "commonjs",
    "target": "es6",
    "forceConsistentCasingInFileNames": true,
    "allowJs": true,
    "allowSyntheticDefaultImports": true,
    "sourceMap": true,
    "baseUrl": ".",
    "paths": {
      "~common/*": ["src/common/*"],
      "~model/*": ["src/model/*"],
      "~orm/*": ["src/orm/*"],
      "~load/*": ["src/load/*"]
    },
    "lib": [
      "es5",
      "es6",
      "dom",
      "es2015",
      "esnext.asynciterable"
    ],
    "types": ["node"],
    "typeRoots": [
      "node_modules/@types"
    ]
  },
  "exclude": [
    "node_modules"
  ],
  "include": ["src/server/handler.ts"]
}
```
2. 正确配置`ts-loader`
我们的ts项目sourcemap无法显示行号和源代码路径,就是挂在这里,对此,`ts-loader`官方github里有一个[example](https://github.com/TypeStrong/ts-loader/tree/master/examples/fork-ts-checker-webpack-plugin), ts-loader一定要记得配置`transpileOnly: true`
```js
  module: {
        rules: [
            {
                test: /.tsx?$/,
                use: [
                    { loader: 'ts-loader', options: { transpileOnly: true } }  
                ],
            }
        ]
    },
```