---
date: "2019-02-25T20:18:57+08:00"
title: Jhipster 项目之架构分析之build.gradle
---
java开发的项目用gradle来管理项目依赖，项目开发，我们这里面还有前端的一些东西，这些东西在不同平台上如何处理，比如window上的空格跟linux上的处理方式就不同该怎么办？jhipster通过他们丰富的经验在这里给出了模板
下面我们会详细的讲解他们这座做的原因

```gradle
// 导入操作系统类，方便针对个别系统进行处理，比如window下的空格，就需要判断操作系统
import org.gradle.internal.os.OperatingSystem

// 
buildscript {
    //定义的仓库
    repositories {
        // mavnen的本地仓库
        mavenLocal()
        // maven的中央仓库
        mavenCentral()
        // spring的plugin 插件库
        maven { url "http://repo.spring.io/plugins-release" }
        // spring的里程碑库
        maven { url "https://plugins.gradle.org/m2/" }
    }
    //定义的依赖类路径
    dependencies {
        // 插件依赖，
        classpath "org.springframework.boot:spring-boot-gradle-plugin:${spring_boot_version}"
        classpath "io.spring.gradle:propdeps-plugin:0.0.10.RELEASE"
        classpath "gradle.plugin.com.gorylenko.gradle-git-properties:gradle-git-properties:1.5.2"
        //jhipster-needle-gradle-buildscript-dependency - JHipster will add additional gradle build script plugins here
    }
}
// 应用的插件
plugins {
    // 代码质量检查插件
    id "org.sonarqube" version "2.6.2"
    // 为什么这个地方会用apt的eclipse插件，可能为了方便把项目导入到eclipse，那么apt是什么东西呢
    id "net.ltgt.apt-eclipse" version "0.19"
    // apt 插件 idea版本的 AnnotationProcessingTool
    id "net.ltgt.apt-idea" version "0.19"
    // apt 插件
    id "net.ltgt.apt" version "0.19"
    // Spring依赖管理插件，可以不用再关心版本，只关心引用
    id "io.spring.dependency-management" version "1.0.6.RELEASE"
    // node 插件，可以在gradle的任务中执行nodejs的任务，本质上是嵌入了一个nodejs
    id "com.moowork.node" version "1.2.0"
    // 数据库版本控制工具的gradle插件，
    id 'org.liquibase.gradle' version '2.0.1'
    //jhipster-needle-gradle-plugins - JHipster will add additional gradle plugins here
    // lombok 一个根据注释动态生成代码工具，jhipster官方不建议用，这个，原因容易出现难以复现的bug
    id 'io.franzbecker.gradle-lombok' version '1.14'
}

// 应用插件
apply plugin: 'java'
// Apt 处理注解，在运行时生成java代码，比如lombok，和mapstruct这个几个技术
apply plugin: 'net.ltgt.apt'
// 设定源码版本
sourceCompatibility=1.8
// 设置并以后class的jvm版本
targetCompatibility=1.8
// Until JHipster supports JDK 9
assert System.properties['java.specification.version'] == '1.8'

// maven 插件
apply plugin: 'maven'
// 引用springboot 的插件
apply plugin: 'org.springframework.boot'
// web项目，所以要应用war插件，来生成war包
apply plugin: 'war'
// 忘了这个是干嘛的
// 为Gradle 提供附加optional和provided依赖配置以及生成Maven POM支持。
apply plugin: 'propdeps'
// node插件
apply plugin: 'com.moowork.node'
// Spring 依赖管理
apply plugin: 'io.spring.dependency-management'
// idea 插件
apply plugin: 'idea'

//依赖管理导入
dependencyManagement {
  imports {
    mavenBom 'io.github.jhipster:jhipster-dependencies:' + jhipster_dependencies_version
    //jhipster-needle-gradle-dependency-management - JHipster will add additional dependencies management here
  }
}

// 默认任务
defaultTasks 'bootRun'
// 声明项目的group 名称，该值同maven中的groupid一样
group = 'org.ylf.demo'
// 声明项目的版本号，方便项目管理
version = '0.0.1-SNAPSHOT'
// 项目描述
description = ''
// Spring Boot如果生成war可执行包时，需要一个可执行的主类
bootWar {
   mainClassName = 'org.ylf.demo.DemoApp'
}
// war包任务，配置
war {
    // 设置webapp的目录文件
    webAppDirName = 'build/www/'
    // 是否启用，默认应该是没启用
    enabled = true
    // 指定war包的后缀格式
    classifier = 'original'
}
// SpringBoot 任务
springBoot {
    // 指定主类方法
    mainClassName = 'org.ylf.demo.DemoApp'
    //构建信息
    buildInfo()
}

// 处理操作系统版本，在window下处理空格，如果不把空格转换为UTF-8字符的话，Java运行时会报错
if (OperatingSystem.current().isWindows()) {
    // https://stackoverflow.com/questions/40037487/the-filename-or-extension-is-too-long-error-using-gradle
    task classpathJar(type: Jar) {
        dependsOn configurations.runtime
        appendix = 'classpath'

        doFirst {
            manifest {
                attributes 'Class-Path': configurations.runtime.files.collect {
                    it.toURI().toURL().toString().replaceFirst(/file:\/+/, '/').replaceAll(' ', '%20')
                }.join(' ')
            }
        }
    }

    bootRun {
        dependsOn classpathJar
        doFirst {
            classpath = files("$buildDir/classes/java/main", "$buildDir/resources/main", classpathJar.archivePath)
        }
    }
}
// 测试任务
test {
    // 排除行为测试
    exclude '**/CucumberTest*'

    // uncomment if the tests reports are not generated
    // see https://github.com/jhipster/generator-jhipster/pull/2771 and https://github.com/jhipster/generator-jhipster/pull/4484
    // ignoreFailures true
    reports.html.enabled = false
}
//行为测试
task cucumberTest(type: Test) {
    // 行为测试描述
    description = "Execute cucumber BDD tests."
    // 校验
    group = "verification"
    //包含那些的
    include '**/CucumberTest*'

    // uncomment if the tests reports are not generated
    // see https://github.com/jhipster/generator-jhipster/pull/2771 and https://github.com/jhipster/generator-jhipster/pull/4484
    // ignoreFailures true
    reports.html.enabled = false
}
// 检查需要依赖 行为测试
check.dependsOn cucumberTest
// 测试报告
task testReport(type: TestReport) {
    // 指定测试报告路径
    destinationDir = file("$buildDir/reports/tests")
    reportOn test
}

// 行为测试报告
task cucumberTestReport(type: TestReport) {
    // 指定行为测试报告路径
    destinationDir = file("$buildDir/reports/tests")
    reportOn cucumberTest
}

// 引用docker的gradle配置
apply from: 'gradle/docker.gradle'
//  引用质量分析的的gradle配置
apply from: 'gradle/sonar.gradle'
//jhipster-needle-gradle-apply-from - JHipster will add additional gradle scripts to be applied here


// 通过判断系统属性来设置引用
if (project.hasProperty('prod')) {
    apply from: 'gradle/profile_prod.gradle'
} else {
    apply from: 'gradle/profile_dev.gradle'
}

// liquibase的配置，如果没有指定具体用那个就默认用main的配置
if (!project.hasProperty('runList')) {
    project.ext.runList = 'main'
}
// liquibase生成changelog文件的目录及生成文件的命名格式
project.ext.diffChangelogFile = 'src/main/resources/config/liquibase/changelog/' + new Date().format('yyyyMMddHHmmss') + '_changelog.xml'
// liquibase任务的配置
liquibase {
    //激活
    activities {
        // main 配置
        main {
            // 驱动类
            driver 'org.h2.Driver'
            // jdbc配置
            url 'jdbc:h2:file:./target/h2db/db/demo'
            // 用户名
            username 'demo'
            // 密码
            password ''
            // changelog文件
            changeLogFile 'src/main/resources/config/liquibase/master.xml'
            // 默认的schema
            defaultSchemaName ''
            // 日志级别为debug
            logLevel 'debug'
            //  类路径
            classpath 'src/main/resources/'
        }
        // 日志差异
        diffLog {
            // jdbc驱动类
            driver 'org.h2.Driver'
            // jdbc url
            url 'jdbc:h2:file:./target/h2db/db/demo'
            // 数据库用户名
            username 'demo'
            // 数据库密码
            password ''
            // 生成差异日志的文件
            changeLogFile project.ext.diffChangelogFile
            // 引用url
            referenceUrl 'hibernate:spring:org.ylf.demo.domain?dialect=org.hibernate.dialect.H2Dialect&amp;
            // hibernate 物理命名策略
            hibernate.physical_naming_strategy=org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy&amp;
            // hibernate 隐式命名策略
            hibernate.implicit_naming_strategy=org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy'
            defaultSchemaName ''
            // 日志级别
            logLevel 'debug'
            // 类路径
            classpath "$buildDir/classes/java/main"
        }
    }
    // 运行List
    runList = project.ext.runList
}

// 配置
configurations {
    // 由运行环境提供
    providedRuntime
    // 排除tomcat
    compile.exclude module: "spring-boot-starter-tomcat"
}

// 仓库
repositories {
    mavenLocal()
    mavenCentral()
    jcenter()
    //jhipster-needle-gradle-repositories - JHipster will add additional repositories
}

dependencies {
    // Use ", version: jhipster_dependencies_version, changing: true" if you want
    // to use a SNAPSHOT release instead of a stable release
    compile group: "io.github.jhipster", name: "jhipster-framework"
    compile "org.springframework.boot:spring-boot-starter-cache"
    compile "io.dropwizard.metrics:metrics-core"
    compile "io.dropwizard.metrics:metrics-jcache"
    compile "io.dropwizard.metrics:metrics-json"
    compile "io.dropwizard.metrics:metrics-jvm"
    compile "io.dropwizard.metrics:metrics-servlet"
    compile "io.dropwizard.metrics:metrics-servlets"
    compile "io.prometheus:simpleclient"
    compile "io.prometheus:simpleclient_dropwizard"
    compile "io.prometheus:simpleclient_servlet"
    compile "net.logstash.logback:logstash-logback-encoder"
    compile "com.fasterxml.jackson.datatype:jackson-datatype-hppc"
    compile "com.fasterxml.jackson.datatype:jackson-datatype-jsr310"
    compile "com.fasterxml.jackson.datatype:jackson-datatype-hibernate5"
    compile "com.fasterxml.jackson.core:jackson-annotations"
    compile "com.fasterxml.jackson.core:jackson-databind"
    compile "com.fasterxml.jackson.module:jackson-module-afterburner"
    compile "com.ryantenney.metrics:metrics-spring"
    compile "javax.cache:cache-api"
    compile "org.hibernate:hibernate-core"
    compile "com.zaxxer:HikariCP"
    compile "org.apache.commons:commons-lang3"
    compile "commons-io:commons-io"
    compile "javax.transaction:javax.transaction-api"
    compile "org.ehcache:ehcache"
    compile "org.hibernate:hibernate-jcache"
    compile "org.hibernate:hibernate-entitymanager"
    compile "org.hibernate:hibernate-envers"
    compile "org.hibernate.validator:hibernate-validator"
    compile "org.liquibase:liquibase-core"
    compile "com.mattbertolini:liquibase-slf4j"
    liquibaseRuntime "org.liquibase:liquibase-core"
    liquibaseRuntime "org.liquibase.ext:liquibase-hibernate5:${liquibase_hibernate5_version}"
    liquibaseRuntime sourceSets.main.compileClasspath
    compile "org.springframework.boot:spring-boot-loader-tools"
    compile "org.springframework.boot:spring-boot-starter-mail"
    compile "org.springframework.boot:spring-boot-starter-logging"
    compile "org.springframework.boot:spring-boot-starter-actuator"
    compile "org.springframework.boot:spring-boot-starter-aop"
    compile "org.springframework.boot:spring-boot-starter-data-jpa"
    compile "org.springframework.boot:spring-boot-starter-security"
    compile ("org.springframework.boot:spring-boot-starter-web") {
        exclude module: 'spring-boot-starter-tomcat'
    }
    compile "org.springframework.boot:spring-boot-starter-undertow"
    compile "org.springframework.boot:spring-boot-starter-thymeleaf"
    compile "org.zalando:problem-spring-web:0.24.0-RC.0"
    compile "org.springframework.boot:spring-boot-starter-cloud-connectors"
    compile "org.springframework.security:spring-security-config"
    compile "org.springframework.security:spring-security-data"
    compile "org.springframework.security:spring-security-web"
    compile "io.jsonwebtoken:jjwt-api"
    runtime "io.jsonwebtoken:jjwt-impl"
    runtime "io.jsonwebtoken:jjwt-jackson"
    compile ("io.springfox:springfox-swagger2") {
        exclude module: 'mapstruct'
    }
    compile "io.springfox:springfox-bean-validators"
    compile "mysql:mysql-connector-java"
    liquibaseRuntime "mysql:mysql-connector-java"
    compile "org.mapstruct:mapstruct-jdk8:${mapstruct_version}"
    annotationProcessor "org.mapstruct:mapstruct-processor:${mapstruct_version}"
    annotationProcessor "org.hibernate:hibernate-jpamodelgen"
    annotationProcessor ("org.springframework.boot:spring-boot-configuration-processor") {
        exclude group: 'com.vaadin.external.google', module: 'android-json'
    }
    testCompile "com.jayway.jsonpath:json-path"
    testCompile "io.cucumber:cucumber-junit"
    testCompile "io.cucumber:cucumber-spring"
    testCompile ("org.springframework.boot:spring-boot-starter-test") {
        exclude group: 'com.vaadin.external.google', module: 'android-json'
    }
    testCompile "org.springframework.security:spring-security-test"
    testCompile "org.springframework.boot:spring-boot-test"
    testCompile "org.assertj:assertj-core"
    testCompile "junit:junit"
    testCompile "org.mockito:mockito-core"
    testCompile "com.mattbertolini:liquibase-slf4j"
    testCompile "org.hamcrest:hamcrest-library"
    testCompile "com.h2database:h2"
    liquibaseRuntime "com.h2database:h2"
    //jhipster-needle-gradle-dependency - JHipster will add additional dependencies here


    compile "com.alipay.sdk:alipay-sdk-java:3.4.49.ALL"
    compile group: 'com.google.code.gson', name: 'gson', version: '2.8.5'
    compile group: 'javax.interceptor', name: 'javax.interceptor-api', version: '1.2'
    // https://mvnrepository.com/artifact/cn.minsin/mutils-spring-boot-starter
    compile group: 'com.github.wxpay', name: 'wxpay-sdk', version: '0.0.3'
    // https://mvnrepository.com/artifact/commons-codec/commons-codec
    compile group: 'commons-codec', name: 'commons-codec', version: '1.11'

}

// 清理资源
task cleanResources(type: Delete) {
    // 删除清理资源
    delete 'build/resources'
}

// 设置 gradle 的版本
wrapper {
    gradleVersion = '4.10.2'
}

// stage任务依赖bootWar
task stage(dependsOn: 'bootWar') {
}

// 如果由nodeInstall属性的话，就设置node的属性
if (project.hasProperty('nodeInstall')) {
    node {
        version = "${node_version}"
        npmVersion = "${npm_version}"
        yarnVersion = "${yarn_version}"
        download = true
    }
}

bootWar.dependsOn war
compileJava.dependsOn processResources
processResources.dependsOn cleanResources,bootBuildInfo
bootBuildInfo.mustRunAfter cleanResources

```

这个配置文件，主要配置了Node和SpringBoot相关的配置，还有liquibase的配置，以及gradle的wrapper的版本信息，以及单元测试和行为测试的相关配置，中间还根据情况引用了docker的配置和sonar的配置。这个例子做的相当全面，是一个全面的教材之一。本篇博客的介绍内容就到这里，具体的信息就看配置文件中的注释