---
date: "2019-02-25T20:18:57+08:00"
title: Jhipster 项目之架构分析之gradle.properteis
---

gradle 的属性配置文件，可以直接在gradle.build中的代码project.hasProperty判断处理。

```properteis
rootProject.name=demo
profile=dev

# Build properties
node_version=10.14.1
npm_version=6.4.1
yarn_version=1.12.3

# Dependency versions
jhipster_dependencies_version=2.0.29
# The spring-boot version should match the one managed by
# https://mvnrepository.com/artifact/io.github.jhipster/jhipster-dependencies/${jhipster_dependencies_version}
spring_boot_version=2.0.7.RELEASE
# The hibernate version should match the one managed by
# https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-dependencies/${spring-boot.version} -->
hibernate_version=5.2.17.Final
mapstruct_version=1.2.0.Final

liquibase_hibernate5_version=3.6
liquibaseTaskPrefix=liquibase

## below are some of the gradle performance improvement settings that can be used as required, these are not enabled by default

## The Gradle daemon aims to improve the startup and execution time of Gradle.
## The daemon is enabled by default in Gradle 3+ setting this to false will disable this.
## TODO: disable daemon on CI, since builds should be clean and reliable on servers
## https://docs.gradle.org/current/userguide/gradle_daemon.html#sec:ways_to_disable_gradle_daemon
## un comment the below line to disable the daemon

#org.gradle.daemon=false

## Specifies the JVM arguments used for the daemon process.
## The setting is particularly useful for tweaking memory settings.
## Default value: -Xmx1024m -XX:MaxPermSize=256m
## un comment the below line to override the daemon defaults

#org.gradle.jvmargs=-Xmx1024m -XX:MaxPermSize=256m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8

## When configured, Gradle will run in incubating parallel mode.
## This option should only be used with decoupled projects. More details, visit
## http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:decoupled_projects
## un comment the below line to enable parallel mode

#org.gradle.parallel=true

## Enables new incubating mode that makes Gradle selective when configuring projects.
## Only relevant projects are configured which results in faster builds for large multi-projects.
## http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:configuration_on_demand
## un comment the below line to enable the selective mode

#org.gradle.configureondemand=true

```

由于本篇幅比较简单，本篇文章就到次结束。