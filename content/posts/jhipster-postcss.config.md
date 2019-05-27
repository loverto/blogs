---
date: "2019-02-25T20:18:57+08:00"
title: Jhipster 项目之架构分析之postcss.config.js
---

postcss 的一大特点是，具体的编译插件甚至是css书写风格，可以根据自己的需要进行安装，选择自己需要的特性：嵌套，函数，变量。自动补全，CSS新特性等等，而不用像less或者scss一样的大型全家桶，因此不需要专门学习less或者sass的语阿伐了，只要选择自己喜欢的特性，可以只写css文件，但依旧可以写嵌套或者函数，然后根据情况选择和是的插件就行了。

## postcss.config.js配置

配置简单，只是用了autoprefixer，进行浏览器兼容补全

```json
module.exports = {
    plugins: [
        require('autoprefixer')
    ]
}
```

## webpack 配合

在packjson.json 中配置webpackloader，具体配置如下

```json
   "postcss-loader": "3.0.0",
```

配合scss做一些工作，比如mincss配置

```json
{
            test: /\.scss$/,
            use: ['to-string-loader', 'css-loader', {
              loader: 'sass-loader',
              options: { implementation: sass }
            }],
            exclude: /(vendor\.scss|global\.scss)/
          },
          {
            test: /(vendor\.scss|global\.scss)/,
            use: [
              {
                loader: MiniCssExtractPlugin.loader,
                options: {
                  publicPath: '../'
                }
              },
              'css-loader',
              'postcss-loader',
              {
                loader: 'sass-loader',
                options: { implementation: sass }
              }
            ]
          },
          {
            test: /\.css$/,
            use: ['to-string-loader', 'css-loader'],
            exclude: /(vendor\.css|global\.css)/
          },
          {
            test: /(vendor\.css|global\.css)/,
            use: [
              {
                loader: MiniCssExtractPlugin.loader,
                options: {
                  publicPath: '../'
                }
              },
              'css-loader',
              'postcss-loader'
            ]
          }
```
