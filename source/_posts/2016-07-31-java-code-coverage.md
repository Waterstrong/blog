---
title: Java Code Coverage in Gradle
date: 2016-07-31 19:53:11
category: Tools
tags: [Java, Code Coverage, Jacoco, Cobertura]
description: Code Coverage is ...
published: true
---

# Code Coverage介绍
## 什么是代码覆盖率

## 代码覆盖率的好处

## 代码覆盖率的类型


# 在Java中应用Code Coverage
## Gradle+Jacoco
Jacoco是开源的Java代码测试覆盖率检查工具，使用ASM修改字节码，插桩主要基于对字节码的on-the-fly和offline的方式，支持提供method, class, line, branch, instruction以及complexity的覆盖率报告。接下来将讲解如何在Gradle中配置Jacoco，实现对Java代码的测试覆盖率检查。

##### Apply Jacoco Plugin
首先，新建一个名为[jacoco.gradle](/assets/java-code-coverage/jacoco.gradle)的文件，并加入以下脚本代码：
```
apply plugin: 'jacoco'

ext {
    limits = [
            'instruction': 95,
            'branch'     : 90,
            'line'       : 90,
            'complexity' : 90,
            'method'     : 95,
            'class'      : 95
    ]
}

jacocoTestReport {
    group = "Reporting"
    description = "Generate and check jacoco coverage reports after running tests."

    reports {
        xml.enabled true
        html.enabled true
        csv.enabled false
    }

    afterEvaluate {
        classDirectories = files(classDirectories.files.collect {
            fileTree(dir: it, exclude: ['**/Application**'])
        })
    }

    doLast {
        new TestCoverage(jacoco).check(limits)
    }
}

check.dependsOn jacocoTestReport
```

其中，`limits`用于配置代码覆盖率检查满足的最小阈值，可以根据项目需要自定义修改，在`exclude`中也可以配置不接收覆盖率检查的Package或Class。

#### Create TestCoverage Class
另外，还需要创建一个用于测试覆盖率检查的类，可以在[jacoco.gradle](/assets/java-code-coverage/jacoco.gradle)中追加以下代码：
```
import org.slf4j.Logger
import static org.slf4j.LoggerFactory.getLogger

class TestCoverage {
    private static Logger logger = getLogger(TestCoverage.class);
    def xmlReport
    def htmlReport
    def metrics

    TestCoverage(jacoco) {
        xmlReport = new File("${jacoco.reportsDir}/test/jacocoTestReport.xml")
        htmlReport = new File("${jacoco.reportsDir}/test/html/index.html")
        initMetrics()
    }

    void check(limits) {
        logger.lifecycle("Checking coverage results: ${xmlReport}")
        logger.lifecycle("Html report: ${htmlReport}")
        showResult checkMetrics(limits)
    }

    private void showResult(failures, improvements) {
        if (failures) {
            logger.quiet("--------------------- Jacoco Code Coverage Failed ---------------------")
            failures.each {
                logger.quiet(it)
            }
            logger.quiet("-----------------------------------------------------------------------")
            throw new GradleException("Jacoco Code coverage failed")
        }
        if (improvements) {
            logger.quiet("-------------------- Jacoco Code Coverage Improved --------------------")
            improvements.each {
                logger.quiet(it)
            }
            logger.quiet("-----------------------------------------------------------------------")
        }
    }

    private List checkMetrics(limits) {
        def failures = []
        def improvements = []
        metrics.each { key, value ->
            def limit = limits[key] as Double
            if (value < limit) {
                failures.add("- ${key} coverage rate is: ${value}%, minimum is ${limit}%")
            }
            if (value > limit + 1) {
                improvements.add("- $key coverage rate is: ${value}%, minimum is ${limit}%")
            }
        }
        [failures, improvements]
    }

    private void initMetrics() {
        def parser = new XmlParser()
        parser.setFeature("http://apache.org/xml/features/nonvalidating/load-external-dtd", false);
        parser.setFeature("http://apache.org/xml/features/disallow-doctype-decl", false)
        def counters = parser.parse(xmlReport).counter
        def percentage = {
            def covered = it ? it.'@covered' as Double : 0
            def missed = it ? it.'@missed' as Double : 0
            ((covered / (covered + missed)) * 100).round(2)
        }
        metrics = [:]
        metrics << [
                'instruction': percentage(counters.find { it.'@type'.equals('INSTRUCTION') }),
                'branch'     : percentage(counters.find { it.'@type'.equals('BRANCH') }),
                'line'       : percentage(counters.find { it.'@type'.equals('LINE') }),
                'complexity' : percentage(counters.find { it.'@type'.equals('COMPLEXITY') }),
                'method'     : percentage(counters.find { it.'@type'.equals('METHOD') }),
                'class'      : percentage(counters.find { it.'@type'.equals('CLASS') })
        ]
    }
}
```

#### Use Custom Jacoco Script
最后需要在`build.gradle`中引用自定义脚本和依赖：
```
apply from: 'jacoco.gradle'
...

dependencies {
	...
	testRuntime 'org.slf4j:slf4j-api:1.7.21'
}
```

在命令行中运行`./gradlew build`可以生成代码覆盖率报告并检查覆盖率是否通过。
![](/assets/java-code-coverage/jacoco_console.png)
![](/assets/java-code-coverage/jacoco_report.png)

## Gradle+Cobertura
Cobertura是开源的Java代码测试覆盖率检查工具，它主要基于对字节码离线插桩的方式实现，支持提供branch和line覆盖率报告。接下来将讲解如何在Gradle中使用Cobertura，并配置实现对Java代码的测试覆盖率检查。

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

第二种方式如下，直接引用plugins DSL的机制，但只支持在Gradle 2.1及以后的版本使用，但特别注意，该script代码只能放在`buildscript`之后，其他script代码之前：
```
plugins {
    id 'net.saliman.cobertura' version '2.3.2'
}
```

然后可以在同一脚本`build.gradle`文件中或另外新增一个名为[cobertura.gradle](/assets/java-code-coverage/cobertura.gradle)的脚本，并写入如下代码：
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

最后，简单写一些测试代码，在命令行中运行`./gradlew clean build`来查看代码覆盖率检查是否配置成功，默认cobertura生成的报告在当前项目的`build/reports/cobertura/`下，可以查看`index.html`。
![](/assets/java-code-coverage/cobertura_report.png)

另外，如果在项目中使用cobertura作为代码测试覆盖率检查工具，但未使用[SLF4J](http://www.slf4j.org/)日志库，在运行时会报出关于slf4j的`NoClassDefFoundError`问题，只需要在`build.gradle`中添加testRuntime的依赖即可：
```
dependencies {
	testRuntime 'org.slf4j:slf4j-api:1.7.21'
}
```

#### Cobertura Configuration
Cobertura的行为是由`cobertura`块中的设置选项控制的，以下将列举常用的选项并简单解释：

* `coberturaVersion = <version>`: 指定使用哪个版本的Cobertura来Run测试覆盖率报告，默认值为2.1.1。
* `coverageDirs = [ <dirnames> ]`: 指定包含代码覆盖率检查的一个或多个Class目录，默认为[ project.sourceSets.main.classesDir.path ]。
* `coverageReportDir = <dir>`: 代码覆盖率报告生成目录。
* `coverageFormats = [ <formats> ]`: 指定Cobertura生成报告的格式，支持`html`和`xml`，默认为html。
* `coverageEncoding`: 生成覆盖率报告时的编码，比如`UTF-8`，如果未设置将自动跟随操作系统编码。
* `coverageSourceDirs = <set of directories>`: 指定需要进行覆盖率检查以及将include在报告中的源文件，默认为project.sourceSets.main.java.srcDirs, project.sourceSets.main.groovy.srcDirs和project.sourceSets.main.scala.srcDirs。
* `coverageIncludes = [ <regexes> ]`: 指定包含满足正则表达式的类文件，比如`coverageIncludes = ['.*net.saliman.someapp.logger.*']`。
* `coverageExcludes = [ <regexes> ]`: 指定不需要进行覆盖检查的类文件，比如`coverageExcludes = ['.*net.saliman.someapp.logger.*']`。
* `coverageIgnores = [ <regexes> ]`: 指定在源代码级别忽略的正则表达语句，比如一些用于日志记录的代码。
* `coverageIgnoreTrivial = <true|false>`: 在版本2.0中的新选项，指定是否忽略简单的getters和setters，默认为false。
* `coverageIgnoreMethodAnnotations = [ <annotations> ]`: 在2.0版本中，可以指定忽略的方法级注解。
* `coverageCheckBranchRate = <percent>`: 指定每个Class的分支覆盖率阈值，0~100的整型值，在运行`coberturaCheck`时有效。
* `coverageCheckLineRate = <percent>`: 指定每个Class的行覆盖率阈值，其余同上。
* `coverageCheckPackageBranchRate = <percent>`: 指定每个Packge的分支覆盖率阈值，其余同上。
* `coverageCheckPackageLineRate = <percent>`: 指定每个Packge的行覆盖率阈值，其余同上。
* `coverageCheckTotalBranchRate = <percent>`: 指定整体分支覆盖率阈值，其余同上。
* `coverageCheckTotalLineRate = <percent>`: 指定整体行覆盖率阈值，其余同上。
* `coverageCheckRegexes = [ <regexes> ]`: 用于更细粒度的控制，可以指定每个独立类的分支和行覆盖率阈值，每个表达式包含三个键值对，比如:`coverageCheckRegexes = [ [ regex: 'com.example.reallyimportant.*', branchRate: 80, lineRate: 90 ], [ regex: 'com.example.boringcode.*', branchRate: 40, lineRate: 30 ] ]`。
* `coverageCheckHaltOnFailure = <true|false>`: 指定`coberturaCheck`在不满足最小覆盖率时是否失败的开关，默认为false。

#### Cobertura Tasks
Cobertura创建了三个Tasks用于生成和检查覆盖率报告：

* `coberturaReport`: 只用于在测试后生成覆盖率报告，该Task不会触发运行测试，需要Gradle单独处理test，通常用于multi-project中需要合并测试报告的情况。
* `cobertura`: 运行`test`任务并生成覆盖率报告，包含了`coberturaReport`任务。
* `coberturaCheck`: 在生成覆盖率报告后进行覆盖率检查，但不会运行test。

如果需要了解更多，可以参阅[Gradle Cobertura Plugin](https://github.com/stevesaliman/gradle-cobertura-plugin)。

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