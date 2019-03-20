---
date: "2019-02-25T20:18:57+08:00"
title: Jhipster 项目之架构分析之packge.json
---

由于前端的项目现在发展的日新月异，同样有类似后台的依赖管理工具，前端的依赖管理工具就是NPM（Node Package Manager），

```json
{
    "name": "demo",
    "version": "0.0.0",
    "description": "Description for demo",
    "private": true,
    "license": "UNLICENSED",
    "cacheDirectories": [
        "node_modules"
    ],
    "dependencies": {
        "@angular/common": "7.1.0",
        "@angular/compiler": "7.1.0",
        "@angular/core": "7.1.0",
        "@angular/forms": "7.1.0",
        "@angular/platform-browser": "7.1.0",
        "@angular/platform-browser-dynamic": "7.1.0",
        "@angular/router": "7.1.0",
        "@fortawesome/angular-fontawesome": "0.3.0",
        "@fortawesome/fontawesome-svg-core": "1.2.8",
        "@fortawesome/free-solid-svg-icons": "5.5.0",
        "@ng-bootstrap/ng-bootstrap": "4.0.0",
        "bootstrap": "4.1.3",
        "core-js": "2.5.7",
        "moment": "2.22.2",
        "ng-jhipster": "0.5.6",
        "ngx-cookie": "2.0.1",
        "ngx-infinite-scroll": "6.0.1",
        "ngx-webstorage": "2.0.1",
        "rxjs": "6.3.3",
        "swagger-ui": "2.2.10",
        "tslib": "1.9.3",
        "zone.js": "0.8.26",
        "ng-diff-match-patch": "2.0.6"
    },
    "devDependencies": {
        "@angular/cli": "7.0.6",
        "@angular/compiler-cli": "7.1.0",
        "@ngtools/webpack": "7.0.6",
        "@types/chai": "4.1.7",
        "@types/chai-string": "1.4.1",
        "@types/jest": "23.3.9",
        "@types/mocha": "5.2.5",
        "@types/node": "10.12.10",
        "@types/selenium-webdriver": "3.0.13",
        "angular-router-loader": "0.8.5",
        "angular2-template-loader": "0.6.2",
        "autoprefixer": "9.3.1",
        "browser-sync": "2.26.3",
        "browser-sync-webpack-plugin": "2.2.2",
        "cache-loader": "1.2.5",
        "chai": "4.2.0",
        "chai-as-promised": "7.1.1",
        "chai-string": "1.5.0",
        "codelyzer": "4.5.0",
        "copy-webpack-plugin": "4.6.0",
        "css-loader": "1.0.1",
        "file-loader": "2.0.0",
        "fork-ts-checker-webpack-plugin": "0.5.0",
        "friendly-errors-webpack-plugin": "1.7.0",
        "generator-jhipster": "5.7.1",
        "html-loader": "0.5.5",
        "html-webpack-plugin": "3.2.0",
        "husky": "1.2.0",
        "jest": "23.6.0",
        "jest-junit": "5.2.0",
        "jest-preset-angular": "6.0.1",
        "jest-sonar-reporter": "2.0.0",
        "lint-staged": "8.1.0",
        "merge-jsons-webpack-plugin": "1.0.18",
        "mocha": "5.2.0",
        "mini-css-extract-plugin": "0.4.5",
        "moment-locales-webpack-plugin": "1.0.7",
        "optimize-css-assets-webpack-plugin": "5.0.1",
        "prettier": "1.15.2",
        "protractor": "5.4.1",
        "reflect-metadata": "0.1.12",
        "rimraf": "2.6.2",
        "simple-progress-webpack-plugin": "1.1.2",
        "style-loader": "0.23.1",
        "terser-webpack-plugin": "1.1.0",
        "thread-loader": "1.2.0",
        "to-string-loader": "1.1.5",
        "ts-node": "7.0.1",
        "ts-loader": "5.3.0",
        "tslint": "5.11.0",
        "tslint-config-prettier": "1.16.0",
        "tslint-loader": "3.6.0",
        "typescript": "3.1.6",
        "postcss-loader": "3.0.0",
        "webpack": "4.26.0",
        "webpack-cli": "3.1.2",
        "webpack-dev-server": "3.1.10",
        "webpack-merge": "4.1.4",
        "webpack-notifier": "1.7.0",
        "webpack-visualizer-plugin": "0.1.11",
        "workbox-webpack-plugin": "3.6.3",
        "write-file-webpack-plugin": "4.5.0"
    },
    "engines": {
        "node": ">=8.9.0"
    },
    "lint-staged": {
        "{,src/**/}*.{md,json,ts,css,scss}": [
            "prettier --write",
            "git add"
        ]
    },
    "scripts": {
        "prettier:format": "prettier --write \"{,src/**/}*.{md,json,ts,css,scss}\"",
        "lint": "tslint --project tsconfig.json -e 'node_modules/**'",
        "lint:fix": "npm run lint -- --fix",
        "ngc": "ngc -p tsconfig-aot.json",
        "cleanup": "rimraf build/{aot,www}",
        "clean-www": "rimraf build//www/app/{src,build/}",
        "e2e": "protractor src/test/javascript/protractor.conf.js",
        "postinstall": "",
        "start": "npm run webpack:dev",
        "start-tls": "npm run webpack:dev -- --env.tls",
        "serve": "npm run start",
        "build": "npm run webpack:prod",
        "test": "npm run lint && jest --coverage --logHeapUsage -w=2 --config src/test/javascript/jest.conf.js",
        "test:watch": "npm run test -- --watch",
        "webpack:dev": "npm run webpack-dev-server -- --config webpack/webpack.dev.js --inline --hot --port=9060 --watch-content-base --env.stats=minimal",
        "webpack:dev-verbose": "npm run webpack-dev-server -- --config webpack/webpack.dev.js --inline --hot --port=9060 --watch-content-base --profile --progress --env.stats=normal",
        "webpack:build:main": "npm run webpack -- --config webpack/webpack.dev.js --env.stats=minimal",
        "webpack:build": "npm run cleanup && npm run webpack:build:main",
        "webpack:prod:main": "npm run webpack -- --config webpack/webpack.prod.js --profile",
        "webpack:prod": "npm run cleanup && npm run webpack:prod:main && npm run clean-www",
        "webpack:test": "npm run test",
        "webpack-dev-server": "node --max_old_space_size=4096 node_modules/webpack-dev-server/bin/webpack-dev-server.js",
        "webpack": "node --max_old_space_size=4096 node_modules/webpack/bin/webpack.js"
    },
    "jestSonar": {
        "reportPath": "build/test-results/jest",
        "reportFile": "TESTS-results-sonar.xml"
    }
}

```

# package.json 中的内容简介

下面介绍一下命令中的内容

`name` 项目名称
`version` 项目版本
`description` 项目描述
`private` 私有项目
`license` 自己开发的项目不打算开源，所以不写授权协议
`cacheDirectories` heroku 使用的缓存目录
`dependencies` 生产环境所需要的依赖
`devDependencies` 开发环境中所需要的依赖
`engines` 控制nodejs版本
`lint-staged` 校验
`scripts` 开发中常用到的脚本汇集
`jestSonar` 测试报告

## 看一下生产环境依赖的包的含义

`@angular/common` 实现基本的Angular 指令和管道，例如NgIf，NgForOf，DecimalPipe 等包含了 Http 和http/testing和testing
`@angular/compiler` 用于在运行时运行Angular编译器以创建ComponentFactorys 的低级服务，稍后可用于创建和呈现Component实例。
`@angular/core` 实现Angular的核心功能，低级服务和实用程序。
定义组件，视图层次结构，更改检测，呈现和事件处理的类基础结构。
定义为Angular构造提供元数据和上下文的装饰器。
定义依赖注入（DI），国际化（i18n）以及各种测试和调试工具的基础结构。
`@angular/forms`
在构建表单以捕获用户输入时，实现一组指令和提供程序以与本机DOM元素进行通信。

使用此API来注册指令，构建表单和数据模型，并为表单提供验证。根据您的使用情况，验证器可以是同步的或异步的。您还可以使用接口和标记来扩展Angular中表单提供的内置功能，以创建自定义验证器和输入元素。

Angular形式允许您：

捕获表单的当前值和验证状态。
跟踪并监听表单数据模型的更改。
验证用户输入的正确性。
创建自定义验证器和输入元素。
您可以通过以下两种方式之一构建表单：

反应表单使用a的现有实例FormControl或FormGroup构建表单模型。此表单模型通过指令与表单输入元素同步，以跟踪并将更改传递回表单模型。对象的值和状态的更改作为可观察对象提供。
模板驱动的形式依赖于指令，例如NgModel和NgModelGroup用于创建表单模型，所以对形式的任何更改都通过模板沟通。

`@angular/platform-browser`支持在不同支持的浏览器上交付Angular应用程序。

在BrowserModule默认情况下，在通过CLI创建的任何应用程序包括在内，并转口CommonModule和ApplicationModule出口，使得提供给应用程序的基本角功能。

`@angular/platform-browser-dynamic` angular 的入口

`@angular/router` 实现Angular Router服务，该服务允许在用户执行应用程序任务时从一个视图导航到下一个视图。

定义将RouteURL路径映射到组件的对象，以及RouterOutlet用于在模板中放置路由视图的指令，以及用于配置，查询和控制路由器状态的完整API。

`@fortawesome/angular-fontawesome` angular 字体图标组件
`@fortawesome/fontawesome-svg-core` 字体svg的核心组件
`@fortawesome/free-solid-svg-icons` 免费字体svg图标

介于篇幅限制，本篇博客先到次结束，后面引用的组件下篇慢慢介绍
