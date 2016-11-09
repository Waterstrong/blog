---
title: Java Checkstyle in Gradle
date: 2016-08-11 22:31:37
category: Tools
tags: [Java, Checkstyle]
thumbnailImage: /assets/java-checkstyle/book.jpg
description: Checkstyle是一个帮助Java开发者遵守某些编码规范的工具，它能够自动化代码规范检查过程，从而使得开发者从这项重要但枯燥的任务中解脱出来，Checkstyle通常适合那些需要强制执行编码规范标准的项目。
published: true
---

Checkstyle是一个帮助Java开发者遵守某些编码规范的工具，它能够自动化代码规范检查过程，从而使得开发者从这项重要但枯燥的任务中解脱出来，Checkstyle通常适合那些需要强制执行编码规范标准的项目。

> Checkstyle is a development tool to help programmers write Java code that adheres to a coding standard. It automates the process of checking Java code to spare humans of this boring (but important) task. This makes it ideal for projects that want to enforce a coding standard.

通常的检查项目包括Javadoc注释、命名约定、Title标题、Import语句、文件大小、空白、修饰符、代码块、类设计、混合检查等。可以阅读[Checkstyle Checks](http://checkstyle.sourceforge.net/checks.html)了解更多。

在Gradle中，实现对Java的编码规范检查，只需要三步即可：
* 1) 在`build.gradle`中引入Checkstyle插件`apply plugin: 'checkstyle'`，默认执行check会触发编码规范检查`checkstyleMain`和`checkstyleTest`。
* 2) 在项目目录下添加默认的文件夹`config/checkstyle`，然后新建默认配置文件`checkstyle.xml`文件，并配置编码规范检查规则，比如这里给出一个编码规范配置的示例：[checkstyle.xml](/assets/java-checkstyle/checkstyle.xml)，对于每个Module的解释可以在[Checks](http://checkstyle.sourceforge.net/checks.html)中找到。
当然，除了使用默认文件名外，也可以在`build.gradle`中自定义规范检查的配置文件，并且可以针对Main和Test类代码的编码规范分别配置，因为通常对Test类代码的编码规范检查并没有那么严格。
``` gradle
checkstyle {
	configProperties.projectDir = project.projectDir
	checkstyleMain.configFile = new File(project.projectDir, '/config/checkstyle/checkstyle-main.xml')
	checkstyleTest.configFile = new File(project.projectDir, '/config/checkstyle/checkstyle-test.xml')
}
```

* 3) 在同一目录下新建默认忽略检查规范的文件`suppressions.xml`，可以配置忽略对某个文件的某个规则的检查，比如一个配置示例：[suppressions.xml](/assets/java-checkstyle/suppressions.xml)。
该`suppressions.xml`需要在`checkstyle.xml`文件中配置，另外，也可以配置`class-header.txt`文件来要求每个类都包括header信息：
``` xml
<module name="Checker">
    <module name="SuppressionFilter">
        <property name="file" value="${projectDir}/config/checkstyle/suppressions.xml" />
    </module>

    <module name="RegexpHeader">
    	<property name="headerFile" value="${projectDir}/config/checkstyle/class-header.txt" />
    </module>

    <module name="TreeWalker">
    	<property name="cacheFile" value="${projectDir}/config/checkstyle/main.cache" />

    	...

    	<module name="ImportControl">
    		<property name="file" value="${projectDir}/config/checkstyle/import-control.xml" />
    	</module>
    </module>
    ...

</module>
```


OK，配置完成后可以放心写代码了，再也不用担心团队成员编码规范不统一的问题了，Enjoy~


----
References

* [Sourceforge Checkstyle](http://checkstyle.sourceforge.net/)
* [Google Java Style](http://checkstyle.sourceforge.net/reports/google-java-style.html)
* [The Checkstyle Plugin](https://docs.gradle.org/current/userguide/checkstyle_plugin.html)
