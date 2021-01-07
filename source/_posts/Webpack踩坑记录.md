---
layout: post
title: Webpack踩坑记录
date: 2021-01-07 15:54:58
tags:
---
在使用webpack进行构建打包时，经常会碰到一些莫名其妙的坑，开贴记录一下，方便以后碰到同类问题的时候查找，同时也给大家提供一个借鉴．

## 使用esj时报错
> "errorMessage": "Cannot create property 'ejs' on string 'self'",
  "errorType": "TypeError",
  
<!-- more -->

解决办法：
```js
new webpack.DefinePlugin({
    window: JSON.stringify('self'),
}),
```

## 找到不文件
> "errorMessage": "ENOENT: no such file or directory, open '/templates/encrypted-key.tpl.xml'",

这是因为用了 `xml-encryption` 这个库，里面会动态地读当前目录下的这个文件，源代码如下：
```js
var templates = {
  'encrypted-key': fs.readFileSync(path.join(__dirname, 'templates', 'encrypted-key.tpl.xml'), 'utf8'),
  'keyinfo': fs.readFileSync(path.join(__dirname, 'templates', 'keyinfo.tpl.xml'), 'utf8')
};
```
解决方法就是使用`CopyWebpackPlugin`插件在构建的时候，把找不到的文件复制到对应的目录下，这样程序在运行时，就不会再报同样的错误了．
```js
new CopyWebpackPlugin([
    {
      from: path.resolve(__dirname, '../node_modules/xml-encryption/lib/templates/encrypted-key.tpl.xml'),
      to: path.resolve(__dirname, '../dist/templates/encrypted-key.tpl.xml'),
      // ignore: ['.*'],
    },
    {
      from: path.resolve(__dirname, '../node_modules/xml-encryption/lib/templates/keyinfo.tpl.xml'),
      to: path.resolve(__dirname, '../dist/templates/keyinfo.tpl.xml'),
      // ignore: ['.*'],
    },
  ]),
```
修改以后发现文件虽然有了，但是还是失败,依然报同样的错误,问题没有解决,排查发现是因为webpack的问题,会在打包的时候将`__dirname`改写别的变量名,所以要加如下的配置
```js
node: {
  __dirname: false,
};
```


## 参考资料
1. [webpack dirname](https://codeburst.io/use-webpack-with-dirname-correctly-4cad3b265a92)