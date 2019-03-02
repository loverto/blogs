---
date: "2019-02-25T20:18:57+08:00"
title: Jhipster 项目之架构分析之huskrc
---

在开发的时候，我们有的时候需要在提交的时候做一些校验，比如控制一些不规范的提交，不规范的推送，这里刚好又这么个校验工具[huskrc](https://github.com/typicode/husky)
这个工具现在也是有很多有名的开源软件在使用，比如

```
jQuery
babel
create-react-app
Next.js
Hyper
Kibana
JSON Server
Hotel
```

Jhipster同样采用了，这个工具来做前端的一些lint校验，具体配置如下所示

```json
{
  "hooks": {
     "pre-commit": "lint-staged"
  }
}
```
钩子的内容，在提交前执行lint-staged，好的本篇文章就降到这里，具体的huskrc可以看我的关于huskrc详细介绍的博文