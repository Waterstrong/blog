---
title: java-code-coverage
date: 2016-07-31 19:53:11
category: Tools
tags: [Java, Code Coverage, Jacoco, Cobertura]
description:
published: false
---

# Code Coverage介绍
## 什么是代码覆盖

## 代码覆盖的好处

## 代码覆盖的类型


# 在Java中应用Code Coverage
## Gradle+Jacoco



## Gradle+Cobertura
#### Apply Cobertura Plugin
首先在`build.gradle`文件开始处加入构建脚本依赖并引用cobertura插件。一般有两种方式，第一种方式支持在所有版本的Gradle中使用：
```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath "net.saliman:gradle-cobertura-plugin:2.3.2"
    }
}
apply plugin: 'net.saliman.cobertura'
```

第二种方式如下，直接引用plugins的机制，但只支持在Gradle 2.1及以后的版本使用，但特别注意，该script代码只能放在`buildscript`之后，其他script代码之前：
```
plugins {
    id 'net.saliman.cobertura' version '2.3.2'
}
```

然后可以在同一脚本`build.gradle`文件中或另外新增一个名为`cobertura.gradle`的脚本，并写入如下代码：
```
cobertura {
    coverageFormats = ['html', 'xml']
    coverageEncoding = 'UTF-8'
    coverageExcludes = ['.*Application.*']
    coverageIgnoreMethodAnnotations = []
    coverageCheckBranchRate = 90
    coverageCheckLineRate = 90
    coverageCheckPackageBranchRate = 90
    coverageCheckPackageLineRate = 90
    coverageCheckTotalBranchRate = 90
    coverageCheckTotalLineRate = 90
    coverageIgnoreTrivial = true
    coverageCheckHaltOnFailure = true
}
check.dependsOn 'coberturaCheck'
```

最后，简单写一些测试代码，在命令行中运行`./gradlew clean build`来查看代码覆盖检查是否配置成功，默认cobertura生成的报告在当前项目的`build/reports/cobertura/`下，可以查看`index.html`。
![]()

另外，如果在项目中使用cobertura作为代码测试覆盖率检查工具，但未使用[SLF4J](http://www.slf4j.org/)日志库，在运行时会报出关于slf4j的`NoClassDefFoundError`问题，只需要在`build.gradle`中添加testRuntime的依赖即可：
```
dependencies {
	testRuntime 'org.slf4j:slf4j-api:1.7.21'
}
```

#### Cobertura Configuration

#### Cobertura Tasks

## Jacoco vs Cobertura


# The End


----
References

* [Code coverage Wiki](https://en.wikipedia.org/wiki/Code_coverage)
* [Java Code Coverage Tools](https://en.wikipedia.org/wiki/Java_Code_Coverage_Tools)
* [springfox jacoco](https://github.com/springfox/springfox)
* [Gradle Logging](http://scratchpad.pietschy.com/gradle/logging.html)
* [gradle-cobertura-plugin](https://github.com/stevesaliman/gradle-cobertura-plugin)
* [net.saliman.cobertura](https://plugins.gradle.org/plugin/net.saliman.cobertura)
