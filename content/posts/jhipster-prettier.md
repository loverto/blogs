---
date: "2019-02-25T20:18:57+08:00"
title: Jhipster 项目之架构分析之perttier
---

在开发代码时候，前端项目如何让代码格式固定，有很多办法，批量格式化代码的神器，虽然咱们前面已经有editorconfig来保证跨开发工具的缩进啊，之类的包支持一直，但是如果需要批量格式化代码，该如何处理呢，刚好
[perttier](https://github.com/prettier/prettier)就是这个神器,可以陪和前面的huskrc在提交前，批量把之前的代码统一按配置的格式化处理

## .prettierrc

```
# Prettier configuration

# 每行行宽
printWidth: 140
# 单引号
singleQuote: true
# tab是4个空格代替
tabWidth: 4
# 不用tab字符
useTabs: false

# js and ts rules:
arrowParens: avoid

# jsx and tsx rules:
jsxBracketSameLine: false

```

通过上面的配置，就可以在前端代码中作出如下的格式化。

## .prettierignore

针对这种批量格式化任务呢，我们针对一些不必要的文件不用格式化，比如node_modules，这个是别人开发好的库文件，格式化了，没什么用，还有编译后的文件，这个不是我们源码需要处理的内容，再有就是锁定版本的文件，更是不需要更新了
```
node_modules
target
package-lock.json

```

那么通过上面两个文件，我们就可以处理批量格式化了，本篇文章到此结束