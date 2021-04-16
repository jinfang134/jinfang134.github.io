---
layout: post
title: vue文件不能跳转到定义的解决办法
date: 2021-02-03 12:46:59
tags:
---

在vscode里面，vue项目里的引入文件不能跳转到定义，在开发过程中，总是要手动去查找文件再打开，挺不方便的.

**解决方法：**
1. 添加jsconfig.json文件,添加[webpack的alias](https://code.visualstudio.com/docs/languages/jsconfig#_using-webpack-aliases)
    >jsconfig.json是干嘛的呢？
    >
    >它用来表明当前的目录是一个js项目的根目录，可以配置一些js服务的特性

<!-- more -->

   ```js
   {
    "compilerOptions": {
      "module": "commonjs",
      "target": "es6",
      "baseUrl": ".",
      "paths": {
        "@/*": ["./src/*"] 
      }
    },
    "exclude": ["node_modules"],
    }
    ```
2. babel.config.js里的presets添加`@vue/app`
    ```js
    module.exports = {
        presets: [
            '@vue/cli-plugin-babel/preset',
            "@babel/preset-typescript",
            '@vue/app'
        ],
        plugins:[
            "@babel/plugin-transform-runtime",
            "@babel/plugin-syntax-dynamic-import",
            "@babel/plugin-transform-sticky-regex",
            "@babel/plugin-proposal-optional-chaining",
            "@babel/plugin-proposal-nullish-coalescing-operator"
        ]
    }
    ```
