---
date: "2019-02-25T20:18:57+08:00"
title: Jhipster 项目之架构分析之yo-rc.json
---

jhipster generator这个工具本身使用[yeoman](https://github.com/yeoman/yo)开发的，那么yo他的开发工具生成代码，jhipster利用这个配置文件来保存生成项目时的一些配置，比如项目名称，用的什么组件之类的

```
{
    "generator-jhipster": {
        "applicationType": "monolith",
        "gitCompany": "",
        "baseName": "demo",
        "packageName": "org.ylf.demo",
        "packageFolder": "org/ylf/demo",
        "serverPort": 8080,
        "serviceDiscoveryType": false,
        "authenticationType": "jwt",
        "uaaBaseName": "../uaa",
        "cacheProvider": "ehcache",
        "enableHibernateCache": true,
        "websocket": false,
        "databaseType": "sql",
        "devDatabaseType": "h2Disk",
        "prodDatabaseType": "mysql",
        "searchEngine": false,
        "enableSwaggerCodegen": false,
        "messageBroker": false,
        "buildTool": "gradle",
        "useSass": false,
        "clientPackageManager": "npm",
        "testFrameworks": ["gatling", "cucumber", "protractor"],
        "enableTranslation": true,
        "nativeLanguage": "zh-cn",
        "languages": ["en", "zh-cn"],
        "clientFramework": "angularX",
        "jhiPrefix": "jhi",
        "jhipsterVersion": "5.7.1",
        "jwtSecretKey": "ZDY5Y2JlZjdlNTNjYTU2MTA3NzcyZjZmMTA2MDVhZWJiMjQ4YjQ2ZTQ2ODIzMzUxZjgzZTlhNzZhOTU4MjljZDBlNjY3OGY5NTBlODgyZGFhYzBlZDc5Y2YwNzFmZDMyMTA4OTdlYTRlNmU4MGZhN2RmZmZhYTI5YjkzZjViYjI=",
        "otherModules": []
    },
    "git-provider": "GitLab",
    "git-company": "flydragon",
    "repository-name": "demo",
    "generator-jhipster-entity-audit": {
        "auditFramework": "custom"
    }
}

```

内容本身并不复杂，我们知道怎么回事，知道这个文件使用来做什么的就好了，这篇文章到此结束