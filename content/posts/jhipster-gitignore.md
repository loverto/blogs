---
date: "2019-02-25T20:18:57+08:00"
title: Jhipster 项目之架构分析之gitignore
---

不论是用什么源码控制软件来控制源码，项目中会有可忽略的文件，比如一些需要运行时才生成的文件，这种文件又大，又不属于源码的范围，这种文件不太建议保存到。那么svn中有ignore的配置文件，而我们用git的中也有类似的配置文件
文件名叫.gitignore文件。
格式主要有以下两种，一种就是排除某个格式，另一种就是在排除的目录中再，

* 空行不匹配任何文件
* #开头表示是注释内容,对于#字符如果想用，可以用转移字符\
* 特殊字符，可以用转移字符来处理\
* !开头表示，非，可以用这个想排除目录中不想排除的文件设置出来
* 以/结尾表示忽略整个目录
* 如果格式不包含/就相当于在根目录下匹配
* *匹配出/之外的任何内容，？匹配除了/之外的任何一个字符，[]匹配所选字符
* /*.c表示匹配c结尾的格式文件
* **/foo表示匹配任意目录中, 
* /** 匹配某个目录下的所有内容
* a/**/b 匹配`a/x/b`,`a/x/y/b`

上面的条目就是git的格式

```gitconfig
######################
# Project Specific
######################
/build/www/**
/src/test/javascript/coverage/

######################
# Node
######################
/node/
node_tmp/
node_modules/
npm-debug.log.*
/.awcache/*
/.cache-loader/*

######################
# SASS
######################
.sass-cache/

######################
# Eclipse
######################
*.pydevproject
.project
.metadata
tmp/
tmp/**/*
*.tmp
*.bak
*.swp
*~.nib
local.properties
.classpath
.settings/
.loadpath
.factorypath
/src/main/resources/rebel.xml

# External tool builders
.externalToolBuilders/**

# Locally stored "Eclipse launch configurations"
*.launch

# CDT-specific
.cproject

# PDT-specific
.buildpath

######################
# Intellij
######################
.idea/
*.iml
*.iws
*.ipr
*.ids
*.orig
classes/
out/

######################
# Visual Studio Code
######################
.vscode/

######################
# Maven
######################
/log/
/target/

######################
# Gradle
######################
.gradle/
/build/

######################
# Package Files
######################
*.jar
*.war
*.ear
*.db

######################
# Windows
######################
# Windows image file caches
Thumbs.db

# Folder config file
Desktop.ini

######################
# Mac OSX
######################
.DS_Store
.svn

# Thumbnails
._*

# Files that might appear on external disk
.Spotlight-V100
.Trashes

######################
# Directories
######################
/bin/
/deploy/

######################
# Logs
######################
*.log*

######################
# Others
######################
*.class
*.*~
*~
.merge_file*

######################
# Gradle Wrapper
######################
!gradle/wrapper/gradle-wrapper.jar

######################
# Maven Wrapper
######################
!.mvn/wrapper/maven-wrapper.jar

######################
# ESLint
######################
.eslintcache

```

git的忽略配置文件和jhipster中的实力文件已经列出来了
