---
title: Groovy SQL Batch
date: 2016-09-14 22:38:59
category: Languages
tags: [Groovy, SQL, Batch]
description: 通常，如果需要写一些针对数据库临时执行特定操作的脚本时，Groovy是一个不错的选择，如果数据量比较大的情况下需要用到Batch方式，接下来将介绍使用Groovy SQL快速实现一个简单的数据库Batch操作Demo。
published: true
---

通常，如果需要写一些针对数据库临时执行特定操作的脚本时，Groovy是一个不错的选择，如果数据量比较大的情况下需要用到Batch方式，接下来就我们就使用Groovy SQL快速实现一个简单的数据库批量操作Demo，如对已有User的姓名进行修饰更新。

## Preparation
准备阶段主要Import依赖，以及配置数据库各项属性并创建连接：
``` groovy
import groovy.sql.Sql
import org.springframework.test.context.jdbc.Sql
# Also can set to project.ext.sql
def sql = Sql.newInstance('url', 'user', 'pwd', 'driver')

# Set the variables
ext {
    batchSize = 100
    customizeFile = file 'file.txt'
}
```

## Statement
创建语句并执行，比如写一个查询和更新：
``` groovy
def getUsers() {
    println 'Start to find users...'
    sql.rows """
        SELECT USER_ID AS userId, USER_NAME AS userName FROM TB_USER WITH UR
    """
}

String getUpdateStatement() {
    """
        UPDATE TB_USER SET USER_NAME=? WHERE USER_ID=?
    """
}
```

## Extraction
Extration可以从查询结果中提取值并重新赋值给新对象，同时写入记录到文件中，建议采用`withWriter`高效写文件的方式：
``` groovy
List extractUserIds(users) {
    println 'Start to extract users...'
    def userIds = [:]
    customizeFile.withWriter { writer ->
        users.each {
            userIds.put(it.userId, it.userNames)
            writer.println it.userId
        }
    }
    userIds
}
```

## Transaction & Batch
对于指执行，建议采用先准备好Sql脚本并加入到Batch中的方式，这样可以按批发送给数据库执行，提升效率，同时在最外层加上事务管理：
``` groovy
def processBatchUpdate(userIds) {
    println "Start to batch(size: $batchSize) update..."
    sql.withTransaction {
        sql.withBatch(batchSize, updateStatement) { statement ->
            userIds.each { statement.addBatch decorate(it.userName), it.userId }
        }
    }
    println "${userIds.size()} users updated"
}
```

## Default Task
默认会被执行的任务，通常是入口，并且执行完毕可应关闭数据库Session：
``` groovy
defaultTasks = ['defaultMain']

# Generally dependsOn: initialiseDriver
task defaultMain() {
    onlyIf { !customizeFile.exitsts() }
    doLast {
        def userIds = extractUserIds(users)
        processBatchUpdate userIds
        sql.close()
    }
}
```

更多关于`Groovy SQL`的用法可以参考官网资料[Groovy Class Sql](http://docs.groovy-lang.org/latest/html/api/groovy/sql/Sql.html)。

----
References
* [Groovy Class Sql](http://docs.groovy-lang.org/latest/html/api/groovy/sql/Sql.html)
* [GROOVY SQL – AN EASY WAY TO DATABASE SCRIPTING](http://www.schibsted.pl/blog/groovy-sql-an-easy-way-to-database-scripting/)