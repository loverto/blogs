---
date: "2019-02-25T20:18:57+08:00"
title: Jhipster 项目之架构分析之gitattributes
---

在开发一个web项目时，如果用git管理一个项目，项目中有几十中格式的文件时，git能否正常的处理这些文件呢？比如图片这种二进制，比如window平台下的bat文件在git仓库上是否应该是一样的，比如jar包这种文件，等等，像这种繁多的文件有时候并不是git默认配置情况下可以处理的非常优秀的，那么下面可以用[git的属性](https://git-scm.com/book/zh/v1/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git%E5%B1%9E%E6%80%A7),
的配置，让git了解这些文件具体该如何处理，并且配置什么类型的文件应该是什么格式，什么结尾，这个文件定义了常见的文件格式

```
# This file is inspired by https://github.com/alexkaratarakis/gitattributes
#
# Auto detect text files and perform LF normalization
# http://davidlaing.com/2012/09/19/customise-your-gitattributes-to-become-a-git-ninja/
* text=auto

# The above will handle all files NOT found below
# These files are text and should be normalized (Convert crlf => lf)

*.bat           text eol=crlf
*.coffee        text
*.css           text
*.cql           text
*.df            text
*.ejs           text
*.html          text
*.java          text
*.js            text
*.json          text
*.less          text
*.properties    text
*.sass          text
*.scss          text
*.sh            text eol=lf
*.sql           text
*.txt           text
*.ts            text
*.xml           text
*.yaml          text
*.yml           text

# Documents
*.doc           diff=astextplain
*.DOC           diff=astextplain
*.docx          diff=astextplain
*.DOCX          diff=astextplain
*.dot           diff=astextplain
*.DOT           diff=astextplain
*.pdf           diff=astextplain
*.PDF           diff=astextplain
*.rtf           diff=astextplain
*.RTF           diff=astextplain
*.markdown      text
*.md            text
*.adoc          text
*.textile       text
*.mustache      text
*.csv           text
*.tab           text
*.tsv           text
*.txt           text
AUTHORS         text
CHANGELOG       text
CHANGES         text
CONTRIBUTING    text
COPYING         text
copyright       text
*COPYRIGHT*     text
INSTALL         text
license         text
LICENSE         text
NEWS            text
readme          text
*README*        text
TODO            text

# Graphics
*.png           binary
*.jpg           binary
*.jpeg          binary
*.gif           binary
*.tif           binary
*.tiff          binary
*.ico           binary
# SVG treated as an asset (binary) by default. If you want to treat it as text,
# comment-out the following line and uncomment the line after.
*.svg           binary
#*.svg          text
*.eps           binary

# These files are binary and should be left untouched
# (binary is a macro for -text -diff)
*.class         binary
*.jar           binary
*.war           binary

## LINTERS
.csslintrc      text
.eslintrc       text
.jscsrc         text
.jshintrc       text
.jshintignore   text
.stylelintrc    text

## CONFIGS
*.conf          text
*.config        text
.editorconfig   text
.gitattributes  text
.gitconfig      text
.gitignore      text
.htaccess       text
*.npmignore     text

## HEROKU
Procfile        text
.slugignore     text

## AUDIO
*.kar           binary
*.m4a           binary
*.mid           binary
*.midi          binary
*.mp3           binary
*.ogg           binary
*.ra            binary

## VIDEO
*.3gpp          binary
*.3gp           binary
*.as            binary
*.asf           binary
*.asx           binary
*.fla           binary
*.flv           binary
*.m4v           binary
*.mng           binary
*.mov           binary
*.mp4           binary
*.mpeg          binary
*.mpg           binary
*.swc           binary
*.swf           binary
*.webm          binary

## ARCHIVES
*.7z            binary
*.gz            binary
*.rar           binary
*.tar           binary
*.zip           binary

## FONTS
*.ttf           binary
*.eot           binary
*.otf           binary
*.woff          binary
*.woff2         binary

```

上面的git属性配置文件分别定义了，常见文本，常见文档，常见图片，常见视频，java开发打包的文件，常见规则校验文件，常见的配置文件，常见归档文件格式，常见字体格式，针对这些格式，jhipster给与了默认合理的配置，配置了，文件应该是什么格式，以什么结束符号结束的，
word等常见格式用astextplain来比较差异，这个配置文件的作用介绍到此结束。其他更详细的git操作可以参考[git权威指南](https://git-scm.com/book/zh/v1/)