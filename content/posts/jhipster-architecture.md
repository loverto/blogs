---
date: "2019-02-25T20:18:57+08:00"
title: Jhipster 项目之架构分析之gradle目录结构
---

jhipster是一个快速生成项目的脚手架，能够快速提高开发效率，本身并不是一个开发技术，但是这个东西本身实际上是一个最佳实践，我们可以看看他的最佳实践做一些学习。

## 项目目录结构

![项目的目录结构](https://ws1.sinaimg.cn/large/61411417ly1g0k8i05y8pj20ja0j8q4c.jpg)

项目的目录结构如上图所示，接下来，咱们详细的讲解一下目录结构的作用。
其中`.gradle`,`.idea`,`node_modules`,`out`，`build`这几个文件是工具自动生成文件，咱们不需要了解，就不细说这个了，

### .jhipster目录

这个目录中都是定义的实体配置，还有就是jhi提供的钩子，可以在开发插件时利用这些钩子做一些有意思的事情，比如给实体添加实体审核的,这里面的文件大部分情况下我们是不需要动地，因为这些文件是jhipster自动帮忙生成的。

```json
{
    "name": "About",
    "fields": [
        {
            "fieldName": "info",
            "fieldType": "String"
        },
        {
            "fieldName": "content",
            "fieldType": "String"
        },
        {
            "fieldName": "order",
            "fieldType": "Long"
        }
    ],
    "relationships": [],
    "changelogDate": "20190105111036",
    "entityTableName": "about",
    "dto": "no",
    "pagination": "no",
    "service": "no",
    "jpaMetamodelFiltering": false,
    "fluentMethods": true,
    "clientRootFolder": "",
    "applications": "*",
    "enableEntityAudit": true
}

```

这里面有关系设置，记录liqubase数据库中的变更日期集，实体表明，是否用dto，是否用分页，是否用service，是否开启jpa元数据模型过滤，客户端 根目录，是否启用实体审核等信息。

```json
[
    {
        "name": "Entity Audit generator",
        "npmPackageName": "generator-jhipster-entity-audit",
        "description": "Add support for entity audit and audit log page",
        "hookFor": "entity",
        "hookType": "post",
        "generatorCallback": "jhipster-entity-audit:entity"
    }
]
```

插件类型，执行时间，执行回调

### gradle目录

本部分内容需要有gradle知识，作为铺垫，如果不了解gradle，可以看我有关gradle中的篇幅，gradle目录本身作用是存放gradle-wrapper的属性，这些东西是，但是jhipster把gradle中的一些工用的gradle相关的配置单独分离出来，放到这里，例如`docker`，`profile_dev`,`profile_prod`,`sonar`,`zipkin`配置
下面咱们分别讲一下里面的内容，首先讲docker.gralde，目录中的文章

#### docker.gralde

```gradle
buildscript {
    repositories {
        gradlePluginPortal()
    }
    dependencies {
        //设置jib的依赖jar包，这个地方设置依赖后，下面就可以用apply plugin 引入插件
        classpath "gradle.plugin.com.google.cloud.tools:jib-gradle-plugin:0.9.11"
    }
}
//利用jib来生成docker
apply plugin: com.google.cloud.tools.jib.gradle.JibPlugin

jib {
    from {
        // 基于openjdk8作为底层镜像
        image = 'openjdk:8-jre-alpine'
    }
    to {
        //最后生成的镜像名称
        image = 'demo:latest'
    }
    container {
        //由于jib生成的镜像没有sh，所以需要在这个地方设置一个入口执行脚本，脚本会在项目目录下src/main/jib/entrypoinit.sh
        entrypoint = ['sh', '-c', 'chmod +x /entrypoint.sh && sync && /entrypoint.sh']
        // 设置端口
        ports = ['8080']
        // 设置环境变量
        environment = [
            SPRING_OUTPUT_ANSI_ENABLED: 'ALWAYS',
            JHIPSTER_SLEEP: '0'
        ]
        // 使用当前时间戳
        useCurrentTimestamp = true
    }
}
// 定义任务来处理静态资源到镜像中，自定义文件资源复制
task copyWwwIntoStatic (type: Copy) {
    from 'build/www/'
    into 'build/resources/main/static'
}
// 设置构建依赖，
jibDockerBuild.dependsOn copyWwwIntoStatic

```

这个生成docker镜像的工具用的是谷歌去年开源的[jib](https://github.com/GoogleContainerTools/jib)工具

#### profile_dev.gradle

开发环境中的默认配置，这个文件的作用，激活spring中的profile文件，在开发中会设置dev（开发中的环境），prod（生产的环境），profile_dev激活dev的配置文件，并且做开发环境中的相关设置，下面我们会详细介绍，会在注释中说明，

```gradle
// 导入操作系统相关的配置，这个导入暂时并没有起作用
import org.gradle.internal.os.OperatingSystem

//使用springframework.boot的插件，设置bootRun
apply plugin: 'org.springframework.boot'
// 使用node插件，可以处理node相关的任务，处理webpack构建
apply plugin: 'com.moowork.node'

dependencies {
    // spring boot的开发工具，方便热部署
    compile "org.springframework.boot:spring-boot-devtools"
    //  h2数据库驱动依赖，一般h2的数据库会在开发环境是使用，由于使用的是jpa，开发的代码无关数据库，这样在做开发，测试是更方便
    compile "com.h2database:h2"
}
// 定义
def profiles = 'dev'
// project.hasProperty 是判断gradle.properties中的属性，
if (project.hasProperty('no-liquibase')) {
    profiles += ',no-liquibase'
}
if (project.hasProperty('tls')) {
    profiles += ',tls'
}
// bootRun springboot中的任务，这里启动时，没有设置其他的参数
bootRun {
    args = []
}

//新建任务构建webpack开发
task webpackBuildDev(type: NpmTask, dependsOn: 'npm_install') {
     // 设置输入目录
    inputs.dir("src/main/webapp/")
    // 设置输入文件，从目录中获取所有文件树
    inputs.files(fileTree('src/main/webapp/'))
    // 设置构建目录
    outputs.dir("build/www/")
    // 设置输出出文件
    outputs.file("build/www/app/main.bundle.js")
    args = ["run", "webpack:build"]
}

// 复制到静态文件
task copyIntoStatic (type: Copy) {
    // 从构建的输出结果
    from 'build/www/'
    // 复制指定的静态资源目录
    into 'build/resources/main/static'
}

// 处理资源文件
processResources {
    // 过滤资源文件，替换资源文件中定义的值
    filesMatching('**/application.yml') {
        filter {
          // 版本的值在build.gradle设置的值
            it.replace('#project.version#', version)
        }
        filter {
            // 替换spring激活的属性
            it.replace('#spring.profiles.active#', profiles)
        }
    }
}
// 处理资源文件，构建webpack
processResources.dependsOn webpackBuildDev
// 复制静态文件，处理静态文件
copyIntoStatic.dependsOn processResources
// 生成boot的jar ，复制静态文件
bootJar.dependsOn copyIntoStatic

```

#### profile_prod.gradle

```gradle
// springboot插件
apply plugin: 'org.springframework.boot'
// git插件
apply plugin: 'com.gorylenko.gradle-git-properties'
// node插件
apply plugin: 'com.moowork.node'

dependencies {
    // 测试依赖jar包中h2
    testCompile "com.h2database:h2"
}

// 定义profile，设置默认值为prod
def profiles = 'prod'
// 设置忽略liquibase
if (project.hasProperty('no-liquibase')) {
    profiles += ',no-liquibase'
}

// 设置swagger
if (project.hasProperty('swagger')) {
    profiles += ',swagger'
}

// boot启动设置
bootRun {
    args = []
}
// webpack测试
task webpack_test(type: NpmTask, dependsOn: 'npm_install') {
    args = ["run", "webpack:test"]
}

// webpack生产
task webpack(type: NpmTask, dependsOn: 'npm_install') {
    args = ["run", "webpack:prod"]
}

//复制静态资源
task copyIntoStatic (type: Copy) {
    from 'build/www/'
    into 'build/resources/main/static'
}

// 处理资源
processResources {
    filesMatching('**/application.yml') {
        filter {
            //替换项目版本
            it.replace('#project.version#', version)
        }
        filter {
            // 替换激活的profile
            it.replace('#spring.profiles.active#', profiles)
        }
    }
}

// 生成git属性，这个信息暂时不知道哪里有处理，如果有知道，可以告诉我
generateGitProperties {
    onlyIf {
        // 源文件不为空时生成git属性信息
        !source.isEmpty()
    }
}

// git属性，包括git.branch（git的分支信息）,git.commit.id.abbrev（git提交的信息）,git.commit.id.describe（git提交新的描述信息）,后两个属性是干嘛的
gitProperties {
    keys = ['git.branch', 'git.commit.id.abbrev', 'git.commit.id.describe']
}
// 测试依赖webpack_test(就是要webpack_test先执行)
test.dependsOn webpack_test
// 处理资源（处理资源需要依赖webpack，让webpack先执行）
processResources.dependsOn webpack
// 复制静态资源(复制静态资源文件前，先执行处理资源)
copyIntoStatic.dependsOn processResources
// 生成springboot格式的jar包（生成springboot格式的jar包前先把静态资源的任务执行了）
bootJar.dependsOn copyIntoStatic

```

#### sonar.gradle

对项目进行扫描

```gradle
// 设置sonar插件（代码质量扫描插件）
apply plugin: "org.sonarqube"
// 设置代码覆盖插件
apply plugin: 'jacoco'

// 设置工具版本
jacoco {
    toolVersion = '0.8.2'
}

// 代码扫描结果，设置报告结果为xml格式的
jacocoTestReport {
    reports {
        xml.enabled true
    }
}

// 代码质量扫描插件设置
sonarqube {
    properties {
        // 设置代码质量的服务器路径
        property "sonar.host.url", "http://localhost:9001"
        // 设置排除扫描的路径
        property "sonar.exclusions", "src/main/webapp/content/**/*.*,src/main/webapp/i18n/*.js, build/www/**/*.*"
        // 设置问题忽略标准，例如，没有文档的api，
        property "sonar.issue.ignore.multicriteria", "S3437,S4502,S4684,UndocumentedApi,BoldAndItalicTagsCheck"

        // Rule https://sonarcloud.io/coding_rules?open=Web%3ABoldAndItalicTagsCheck&rule_key=Web%3ABoldAndItalicTagsCheck is ignored. Even if we agree that using the "i" tag is an awful practice, this is what is recommended by http://fontawesome.io/examples/
        property "sonar.issue.ignore.multicriteria.BoldAndItalicTagsCheck.resourceKey", ">src/main/webapp/app/**/*.*"
        property "sonar.issue.ignore.multicriteria.BoldAndItalicTagsCheck.ruleKey", "Web:BoldAndItalicTagsCheck"

        // Rule https://sonarcloud.io/coding_rules?open=squid%3AS3437&rule_key=squid%3AS3437 is ignored, as a JPA-managed field cannot be transient
        property "sonar.issue.ignore.multicriteria.S3437.resourceKey", "src/main/java/**/*"
        property "sonar.issue.ignore.multicriteria.S3437.ruleKey", "squid:S3437"

        // Rule https://sonarcloud.io/coding_rules?open=squid%3AUndocumentedApi&rule_key=squid%3AUndocumentedApi is ignored, as we want to follow "clean code" guidelines and classes, methods and arguments names should be self-explanatory
        property "sonar.issue.ignore.multicriteria.UndocumentedApi.resourceKey", "src/main/java/**/*"
        property "sonar.issue.ignore.multicriteria.UndocumentedApi.ruleKey", "squid:UndocumentedApi"
        // Rule https://sonarcloud.io/coding_rules?open=squid%3AS4502&rule_key=squid%3AS4502 is ignored, as for JWT tokens we are not subject to CSRF attack
        property "sonar.issue.ignore.multicriteria.S4502.resourceKey", "src/main/java/**/*"
        property "sonar.issue.ignore.multicriteria.S4502.ruleKey", "squid:S4502"

        // Rule https://sonarcloud.io/coding_rules?open=squid%3AS4684&rule_key=squid%3AS4684
        property "sonar.issue.ignore.multicriteria.S4684.resourceKey", "src/main/java/**/*"
        property "sonar.issue.ignore.multicriteria.S4684.ruleKey", "squid:S4684"

        // 设置代码覆盖报告路径
        property "sonar.jacoco.reportPaths", "${project.buildDir}/jacoco/test.exec"
        // 设置代码覆盖插件为jacoco
        property "sonar.java.codeCoveragePlugin", "jacoco"
        // 代码执行报告路径
        property "sonar.testExecutionReportPaths", "${project.buildDir}/test-results/jest/TESTS-results-sonar.xml"
        // typescript报告路径
        property "sonar.typescript.lcov.reportPaths", "${project.buildDir}/test-results/lcov.info"
        // 单元测试报告路径
        property "sonar.junit.reportPaths", "${project.buildDir}/test-results/test"
        //  设置源代码路径
        property "sonar.sources", "${project.projectDir}/src/main/"
        //  设置测试代码路径
        property "sonar.tests", "${project.projectDir}/src/test/"
    }
}
```

#### zipkin.gradle

zipkin 微服务项目调用链跟踪的jar

```gradle
dependencies {
    // spring cloud的zipkin的依赖
    compile "org.springframework.cloud:spring-cloud-starter-zipkin"
}
```

介于篇幅，本篇介绍的内容到此为止，下篇博客会讲src目录中的相关内容