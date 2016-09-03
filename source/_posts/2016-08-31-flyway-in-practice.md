---
title: 快速掌握和使用Flyway
date: 2016-08-31 21:32:16
category: Tools
tags: [数据库, Flyway, Migration, Gradle, Spring Boot]
description: Flyway是一款开源的数据库版本管理工具，它更倾向于规约优于配置的方式。Flyway可以独立于应用实现管理并跟踪数据库变更，支持数据库版本自动升级，并且有一套默认的规约，不需要复杂的配置，不仅支持命令行和Java API，还支持Build构建工具和Spring Boot等，同时在分布式环境下能够安全可靠地升级数据库。
published: true
---

## 什么是Flyway?
> Flyway is an open-source database migration tool. It strongly favors simplicity and convention over configuration.

Flyway是一款开源的数据库版本管理工具，它更倾向于规约优于配置的方式。Flyway可以独立于应用实现管理并跟踪数据库变更，支持数据库版本自动升级，并且有一套默认的规约，不需要复杂的配置，Migrations可以写成SQL脚本，也可以写在Java代码中，不仅支持Command Line和Java API，还支持Build构建工具和Spring Boot等，同时在分布式环境下能够安全可靠地升级数据库，同时也支持失败恢复等。

Flyway主要基于6种基本命令：`Migrate`, `Clean`, `Info`, `Validate`, `Baseline` and `Repair`，稍候会逐一分析讲解。目前支持的数据库主要有：Oracle, SQL Server, SQL Azure, DB2, DB2 z/OS, MySQL(including Amazon RDS), MariaDB, Google Cloud SQL, PostgreSQL(including Amazon RDS and Heroku), Redshift, Vertica, H2, Hsql, Derby, SQLite, SAP HANA, solidDB, Sybase ASE and Phoenix.

关于Flyway的优势，支持的数据库以及与其他数据库版本工具的对比，可以阅读[Flyway官网介绍](https://flywaydb.org/)。

## 为什么使用Flyway?
通常在项目开始时会针对数据库进行全局设计，但在开发产品新特性过程中，难免会遇到需要更新数据库Schema的情况，比如：添加新表，添加新字段和约束等，这种情况在实际项目中也经常发生。那么，当开发人员完成了对数据库更的SQL脚本后，如何快速地在其他开发者机器上同步？并且如何在测试服务器上快速同步？以及如何保证集成测试能够顺利执行并通过呢？

假设以Spring Boot技术栈项目为例，可能有人会说，本地使用Hibernate自动更新数据库Schema模式，然后让QA或DEV到测试服务器上手动执行SQL脚本，同时可以写一个Gradle任务自动执行更新。

个人觉得，对于Hibernate自动更新数据库，感觉不靠谱，不透明，控制自由度不高，而且有时很容易就会犯错，比如：用SQL创建的某个字段为VARCHAR类型，而在Entity中配置的为CHAR类型，那么在运行集成测试时，自动创建的数据库表中的字段为CHAR类型，而实际SQL脚本期望的是VARCHAR类型，虽然测试通过了，但不是期望的行为，并且在本地bootRun或服务器上运行Service时都会失败。另外，到各测试服务器上手动执行SQL脚本费时费神费力的，干嘛不自动化呢，当然，对于高级别和PROD环境，还是需要DBA手动执行的。最后，写一段自动化程序来自动执行更新，想法是很好的，那如果已经有了一些插件或库可以帮助你更好地实现这样的功能，为何不好好利用一下呢，当然，如果是为了学习目的，重复造轮子是无可厚非的。

其实，以上问题可以通过Flyway工具来解决，Flyway可以实现自动化的数据库版本管理，并且能够记录数据库版本更新记录，接下来就一起看看Flyway是如何工作的。

## Flyway如何工作的?
Flyway对数据库进行版本管理主要由六种命令完成，每种命令功能和解决的问题范围不一样，以下分别对这些命令进行阐述，其中的示意图都来自Flyway的官方文档。

#### Migrate
Migrate是指把数据库Schema迁移到最新版本，是Flyway工作流的核心功能，Flyway在Migrate时会检查Metadata(元数据)表，如果不存在会创建Metadata表，Metadata表主要用于记录版本变更历史以及Checksum之类的。
![](/assets/flyway-in-practice/command_migrate.png)

Migrate时会扫描指定文件系统或Classpath下的Migrations(可以理解为数据库的版本脚本)，并且会逐一比对Metadata表中的已存在的版本记录，如果有未应用的Migrations，Flyway会获取这些Migrations并按次序Apply到数据库中，否则不需要做任何事情。另外，通常在应用程序启动时应默认执行Migrate操作，从而避免程序和数据库的不一致性。

#### Clean
Clean相对比较容易理解，即清除掉对应数据库Schema中的所有对象，包括表结构，视图，存储过程，函数以及所有的数据等都会被清除。
![](/assets/flyway-in-practice/command_clean.png)

Clean操作在开发和测试阶段是非常有用的，它能够帮助快速有效地更新和重新生成数据库表结构，但特别注意的是：不应在Production的数据库上使用！

#### Info
Info用于打印所有Migrations的详细和状态信息，其实也是通过Metadata表和Migrations完成的，下图很好地示意了Info打印出来的信息。
![](/assets/flyway-in-practice/command_info.png)

Info能够帮助快速定位当前的数据库版本，以及查看执行成功和失败的Migrations。

#### Validate
Validate是指验证已经Apply的Migrations是否有变更，Flyway是默认是开启验证的。
![](/assets/flyway-in-practice/command_validate.png)

Validate原理是对比Metadata表与本地Migrations的Checksum值，如果值相同则验证通过，否则验证失败，从而可以防止对已经Apply到数据库的本地Migrations的无意修改。

#### Baseline
Baseline针对已经存在Schema结构的数据库的一种解决方案，即实现在非空数据库中新建Metadata表，并把Migrations应用到该数据库。
![](/assets/flyway-in-practice/command_baseline.png)

Baseline可以应用到特定的版本，这样在已有表结构的数据库中也可以实现添加Metadata表，从而利用Flyway进行新Migrations的管理了。

#### Repair
Repair操作能够修复Metadata表，该操作在Metadata表出现错误时是非常有用的。
![](/assets/flyway-in-practice/command_repair.png)

Repair会修复Metadata表的错误，通常有两种用途：
- 移除失败的Migration记录，该问题只是针对不支持DDL事务的数据库。
- 重新调整已经应用的Migratons的Checksums值，比如：某个Migratinon已经被应用，但本地进行了修改，又期望重新应用并调整Checksum值，不过尽量不要这样操作，否则可能造成其它环境失败。

## 如何使用Flyway?

#### 运用Migrations
![](/assets/flyway-in-practice/sql_migration_base_dir.png)

![](/assets/flyway-in-practice/sql_migration_naming.png)


#### 支持的数据库

#### Flyway命令行

#### 在Gradle中的应用
首先需要在Gradle中引入Flyway插件，通常有两种方式：
- 方式一：采用buildscript依赖方式。
```
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.flywaydb:flyway-gradle-plugin:4.0.3")
    }
}
apply plugin: 'org.flywaydb.flyway'
```

- 方式二（推荐）：采用DSL方式引用Plugins。
```
plugins {
    id "org.flywaydb.flyway" version "4.0.3"
}
```

而在Gradle中配置Flyway Properties也有两种方式：
- 方式一：在`build.gradle`中配置Flyway Properties。
```
flyway{
	url = jdbc:h2:./.tmp/testdb
	user = sa
	password = 
}
```

- 方式二：在`gradle.properties`中配置Flyway Properties。
```
flyway.url = jdbc:h2:./.tmp/testdb
flyway.user = sa
flyway.password =
```

如果期望在运行Gradle Clean/Build Tasks时自动执行Flyway的某些任务，可以设置`dependsOn`，若不期望隐式执行Flyway任务，可以不配置。
```
clean.dependsOn flywayRepair  # To repair the Flyway metadata table
build.dependsOn flywayMigrate  # To migrate the schema to the latest version
```

在使用Spring Boot时，运行`./gradlew bootRun`会自动检查并加载最新的db.migration脚本。
**特别注意：**在Production环境中不应执行`./gradlew flywayClean`，除非你知道自己的行为和目的，因为该命令会清除所有的数据库对象，相当危险。

更多关于Flyway在Gradle中的使用请参阅[Flyway Gradle Plugin](https://flywaydb.org/documentation/gradle/): `flywayInfo`, `flywayValidate`, `flywayBaseline`。

#### 与Spring Boot集成
在Spring Boot中，如果加入Flyway的依赖，则会<u>自动引用Flyway并使用默认值</u>，但可以修改并配置[FlywayProperties](https://github.com/spring-projects/spring-boot/tree/v1.4.0.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/flyway/FlywayProperties.java)。
```
flyway.baseline-description= #
flyway.baseline-version=1 # version to start migration
flyway.baseline-on-migrate=false # Whether to execute migration against a non-empty schema with no metadata table
flyway.check-location=false # Check that migration scripts location exists.
flyway.clean-on-validation-error=false # will clean all objects. Warning! Do NOT enable in production!
flyway.enabled=true # Enable flyway.
flyway.encoding= #
flyway.ignore-failed-future-migration= #
flyway.init-sqls= # SQL statements to execute to initialize a connection immediately after obtaining it.
flyway.locations=classpath:db/migration # locations of migrations scripts
flyway.out-of-order=false # Allows migrations to be run "out of order"
flyway.placeholder-prefix= #
flyway.placeholder-replacement= #
flyway.placeholder-suffix= #
flyway.placeholders.*= #
flyway.schemas= # schemas to update
flyway.sql-migration-prefix=V #
flyway.sql-migration-separator= #
flyway.sql-migration-suffix=.sql #
flyway.table= #
flyway.url= # JDBC url of the database to migrate. If not set, the primary configured data source is used.
flyway.user= # Login user of the database to migrate. If not set, use spring.datasource.username value
flyway.password= # JDBC password if you want Flyway to create its own DataSource
flyway.validate-on-migrate=true # Validate sql migration CRC32 checksum in classpath
```

若使用Gradle，通常在`build.gradle`引入`org.flywaydb:flyway-core:4.0.3`依赖后即可使用。可能会有以下几种需求：

- 在本地Run和Tests都会使用内存数据库，其中的`spring.jpa.hibernate.ddl-auto`都设置为`validate`，Schema不需要Hibernate自动生成，并期望使用Flyway，而在线上环境会使用真实数据库，并不期望使用Flyway，如何实现呢？
**解决方案：**可以在`common.properties`中配置`flyway.enabled=false`，然后在local或dev的配置中启用Flyway即可。通常推荐使用此模式，毕竟可以对不同的环境进行控制，另外本地Run不会依赖真实数据库，又能保证数据库Schema是按脚本创建的。

- 在运行Tests会使用内存数据库，有单独的配置文件，不使用Flyway，而在本地bootRun时会使用真实数据库，使用Flyway，毕竟不想每次Schema改后都在本地手动去执行脚本，如何实现？
**解决方案：**设置`bootRun.dependsOn`动态添加Flyway的依赖即可：
```
addFlywayDenpendency {
	doLast {
		dependencies {
			compile('org.flywaydb:flyway-core:4.0.3')
		}
		
	}
}

bootRun.dependsOn=addFlywayDenpendency
```

- 若项目有多个团队同时开发不同的功能，需要新建多个分支，并且都会涉及到数据库Schema更改，当后期Merge时，Migration的版本如何控制并且不会产生数据库更改的冲突呢？
**解决方案：**如果两个分支的数据库更改有冲突，要么最初数据库设计不合理，要么目前数据库更改不合理，所以需要团队进行全局考虑和协调。而针对数据库在同一段时间有修改，但不会造成冲突的情况，通常实际项目中主要存在这样的情况，那可以设置`flyway.out-of-order=true`，这样允许当v1和v3已经被应用后，v2出现时同样也可以被应用。其实在本地使用内存数据库不会存在该问题，因为数据库所有对象会自动清除掉，而在local或dev中使用真实数据库时可遇到这样的问题，因此需要注意一下了。
另外，值得一提的是Flyway的参数`ignore-failed-future-migration`默认为`true`，使用情形为：当Rollback数据库更改到旧版本，而metadata表中已存在了新版本时，Flyway会忽略此错误，只会显示警告信息。

## 结束语


----
References
* [Flyway Documentation](https://flywaydb.org/documentation/)
* [Gradle Plugin: Flyway](https://flywaydb.org/documentation/gradle/)
* [Gradle Task: flywayMigrate](https://flywaydb.org/documentation/gradle/migrate)
* [Spring Common application properties](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html)