---
date: "2019-02-25T20:18:57+08:00"
title: Jhipster 项目之架构分析之EditorConfig
---

项目开发时，随着人员的增多，需要大家规范开发中的代码格式，虽然各个开发工具都有格式代码的工具，但是人员不同个人喜好的开发工具也不同，为了遵循大家自由的选择开发工具，而又可以满足维护一致的编码样式，该如何处理呢，
EditorConfig刚好就是为这种场景而生的工具

## .editorconfig

```
# EditorConfig helps developers define and maintain consistent
# coding styles between different editors and IDEs
# editorconfig.org

root = true

[*]

# Change these settings to your own preference
# 缩进格式为4个空格
indent_style = space
indent_size = 4

# We recommend you to keep these unchanged
# 设置结尾符号为LF，统一结尾符号，避免window上跟linux或mac上结尾不同
end_of_line = lf
# 统一字符编码为UTF-8
charset = utf-8
# 删除换行符号前面的所有空白字符
trim_trailing_whitespace = true
# Unix-style 风格的换行
insert_final_newline = true

[*.md]
# 针对markdown文件不去除换行福前面的空白字符
trim_trailing_whitespace = false

[package.json]
# package.json文件缩进为两个空格
indent_style = space
indent_size = 2

```

这是这个配置文件的简单介绍。本篇先到此为止