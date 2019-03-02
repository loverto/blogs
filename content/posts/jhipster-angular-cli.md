---
date: "2019-02-25T20:18:57+08:00"
title: Jhipster 项目之架构分析之angular.json
---

由于我选择的项目前端开发框架用的angular，那么jhipster前端angular利用到了官方的[angular-cli](https://github.com/angular/angular-cli)来解决问题，

```
{
  "$schema": "./node_modules/@angular/cli/lib/config/schema.json",
  "version": 1,
  "newProjectRoot": "projects",
  "projects": {
    "demo": {
      "root": "",
      "sourceRoot": "src/main/webapp",
      "projectType": "application",
      "architect": {}
    }
  },
  "defaultProject": "demo",
  "cli": {
    "packageManager": "npm"
  },
  "schematics": {
    "@schematics/angular:component": {
      "inlineStyle": true,
      "inlineTemplate": false,
      "spec": false,
      "prefix": "jhi",
      "styleExt": "css"
    },
    "@schematics/angular:directive": {
      "spec": false,
      "prefix": "jhi"
    },
    "@schematics/angular:guard": {
      "spec": false
    },
    "@schematics/angular:pipe": {
      "spec": false
    },
    "@schematics/angular:service": {
      "spec": false
    }
  }
}

```
这个配置文件，根据需要，设定了项目文件的源码目录，像盲目类型，项目版本，登场用配置，具体信息我们可以参考我的博文angular-cli相关的文章