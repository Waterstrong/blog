---
title: 快速掌握和使用Flyway
date: 2016-08-31 21:32:16
category: Tools
tags: [数据库, Flyway, Gradle, Spring Boot]
description: Flyway是一款开源的数据库版本管理工具。
published: true
---

## 什么是Flyway?

## 为什么使用Flyway?

## Flyway如何工作的?

Concepts -> Migrations

Flyway Commands

#### Migrate

#### Clean

#### Info


#### Validate

#### Baseline


#### Repair


## 如何使用Flyway?

#### 支持的数据库

#### Flyway命令行

#### 在Gradle中的应用



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