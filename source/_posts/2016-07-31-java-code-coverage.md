---
title: Java Code Coverage in Gradle
date: 2016-07-31 19:53:11
category: Tools
tags: [Java, Code Coverage, Jacoco, Cobertura]
description: 代码覆盖率是用于衡量源代码被测试粒度和程度的，在一定程序上能够衡量代码质量，通常作为发现未被测试覆盖的代码的一种手段，可以直接反映部分测试遗漏点。覆盖率主要用于尽可能减少代码缺陷和Bugs及降低出错风险，较高的测试覆盖率能够增加开发人员的信心。
published: true
---

## Code Coverage介绍
### 什么是代码覆盖率？
代码覆盖率是用于衡量源代码被测试粒度和程度的，在一定程序上能够衡量代码质量，通常作为发现未被测试覆盖的代码的一种手段，可以直接反映部分测试遗漏点。

覆盖率主要用于尽可能减少代码Defects和Bugs及降低出错风险，较高的测试覆盖率能够增加开发人员的信心，但并不代表代码测试覆盖率越高越好，也不需要实现100%的覆盖率，并且这也很难实现，通常对核心逻辑才会增加测试粒度，根据经验值，覆盖率在85%左右为宜，即增加团队成员信心，也减少不必要的工作量。

另外，代码覆盖率粒度通常分为类(Class)、方法(Method)、块(Block)、分支(Branch)、行(Line)、指令(instruction)以圈(complexity)覆盖。

需要注意的是，不应把测试覆盖率作为代码质量唯一指标，而应作为发现未被测试覆盖的代码的手段，并且代码覆盖率不能完全作为衡量代码质量的标准。

### 代码覆盖率统计原理
主流代码覆盖率工具都采用字节码插桩模式，通过钩子的方式来记录代码执行轨迹信息。以Java为例，目前常用的工具为Jacoco和Cobertura，其对字节码进行插桩，主要分为on-the-fly和offine两种模式。一般的过程为：首先执行测试用例，收集程序执行轨迹信息，并存入内存中，然后数据处理器结合程序执行轨迹信息和代码结构信息分析生成代码覆盖率报告，最后将代码覆盖率报告以图形化方式展示出来。

#### On-The-Fly插桩
On-The_Fly也可分为基于Java Agent和Class Loader两种方式。

Java Agent原理如下: 
1. JVM中通过`-javaagent`参数指定特定的jar文件启动Instrumentation的代理程序
2. 代理程序在每装载一个class文件前判断是否已经转换修改了该文件，如果没有则需要将探针插入class文件中
3. 代码覆盖率就可以在JVM执行代码的时候实时获取

Class Loader原理为：自定义Class Loader实现自己的类装载策略，在类加载之前将探针插入class文件中。

On-The-Fly模式优点在于无需修改源代码，无需提前进行字节码插桩，更加方便的获取代码覆盖率，可以在系统不停机的情况下，实时获取和收集代码覆盖率信息。

#### Offline插桩

在测试之前先对文件进行插桩，生成插过桩的class文件或者jar包，执行插过桩的class文件或者jar包之后，会生成覆盖率信息到文件，最后统一对覆盖率信息进行处理，并生成报告。Offline插桩又分为两种：
* Replace：替换方式，修改字节码生成新的class文件
* Inject：注入方式，在原有字节码文件上进行修改

Offine模式优点在于系统启动不需要额外开启代理，但只能在系统停机的情况下才能获取代码覆盖率。Offline模式适用于以下场景：
- 运行环境不支持Java Agent
- 部署环境不允许设置JVM参数
- 字节码需要被转换成其他虚拟机字节码
- 动态修改字节码过程中和其他Agent冲突
- 无法自定义用户加载类

## 在Java中应用Code Coverage
本博客将主要讲解如何在Java中实现对代码测试覆盖统计和检查，采用Gradle构建工具，以Jacoco和Cobertura覆盖率工具为例，分别给出实现步骤和代码。

### Gradle + Jacoco
Jacoco是开源的Java代码测试覆盖率检查工具，使用ASM修改字节码，插桩主要基于对字节码的on-the-fly和offline的方式，支持提供method, class, line, branch, instruction以及complexity的覆盖率报告。接下来将讲解如何在Gradle中配置Jacoco，实现对Java代码的测试覆盖率检查。

#### Apply Jacoco Plugin
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

### Gradle + Cobertura
Cobertura是开源的Java代码测试覆盖率检查工具，它主要基于对字节码offline插桩的方式实现，支持提供branch和line覆盖率报告。接下来将讲解如何在Gradle中使用Cobertura，并配置实现对Java代码的测试覆盖率检查。

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

### Jacoco vs Cobertura
Jacoco与Cobertura的区别在于插桩的方式，前者是off-line和on-the-fly，而后者只是off-line，Jacoco支持的覆盖率粒度要多于Cobertura(只支持line和branch)，二者都支持[SonarQube](http://www.sonarqube.org/)集成，报告都支持HTML和XML格式，Jacoco性能要略优于Cobertura。更多对比请参阅[Comparison of code coverage tools](https://confluence.atlassian.com/display/CLOVER/Comparison+of+code+coverage+tools)和[Code Coverage Tools Comparison in Sonar](https://onlysoftware.wordpress.com/2012/12/19/code-coverage-tools-jacoco-cobertura-emma-comparison-in-sonar/)。

## The End
总之，在开发过程中进行测试覆盖率检查在一定程度上能够保证代码的质量，可以作为发现未被测试覆盖的代码的一种手段，可以直接反映部分测试遗漏点，从而尽可能减少代码Defects和Bugs及降低出错风险，提高团队成员的信心，至于使用哪种覆盖率工具需要根据项目代码性质决定，大多数情况下建议选择Jacoco。以上就是对Jacoco和Cobertura的基本概念和实践的介绍，现在就可以自己动手试一下吧。

----
References

* [Code coverage Wiki](https://en.wikipedia.org/wiki/Code_coverage)
* [Java Code Coverage Tools](https://en.wikipedia.org/wiki/Java_Code_Coverage_Tools)
* [The JaCoCo Plugin](https://docs.gradle.org/current/userguide/jacoco_plugin.html)
* [springfox jacoco](https://github.com/springfox/springfox)
* [Gradle Logging](http://scratchpad.pietschy.com/gradle/logging.html)
* [gradle-cobertura-plugin](https://github.com/stevesaliman/gradle-cobertura-plugin)
* [net.saliman.cobertura](https://plugins.gradle.org/plugin/net.saliman.cobertura)
* [Comparison of code coverage tools](https://confluence.atlassian.com/display/CLOVER/Comparison+of+code+coverage+tools)
* [浅谈代码覆盖率](http://www.tuicool.com/articles/aq6rUz)
* [Code Coverage Tools Comparison in Sonar](https://onlysoftware.wordpress.com/2012/12/19/code-coverage-tools-jacoco-cobertura-emma-comparison-in-sonar/)
