# GaussDB开源驱动介绍
本项目的目的是帮助繁荣[GaussDB](https://www.huaweicloud.com/product/gaussdb.html)和[OpenGauss](https://opengauss.org/zh/)开源生态。项目包括常见语言的驱动（早期都通过PostgreSQL开源驱动进行构建），在这些开源驱动的基础上，贡献其他开源软件，包括ORM等。

# 总体计划

|语言| 驱动名称                                                                                 | 对应PostgreSQL驱动                                                 | ORM框架                                                  |备注|
|----|--------------------------------------------------------------------------------------|----------------------------------------------------------------|--------------------------------------------------------|----|
|JAVA| [gaussdb-r2dbc](https://github.com/HuaweiCloudDeveloper/gaussdb-r2dbc)                  | [r2dbc-postgresql](https://github.com/pgjdbc/r2dbc-postgresql) | [Spring Data R2DBC](https://github.com/spring-projects/spring-data-relational/)                                  ||
|GO| [gaussdb-go](https://github.com/HuaweiCloudDeveloper/gaussdb-go)                     | [pgx](https://github.com/jackc/pgx)                            | [GORM](https://github.com/go-gorm/gorm)                ||
|Python| [gaussdb-python](https://github.com/HuaweiCloudDeveloper/gaussdb-python)             | [psycopg](https://github.com/psycopg/psycopg)                  | [SqlAlchemy](https://github.com/sqlalchemy/sqlalchemy) ||
|Python| [gaussdb-python-async](https://github.com/HuaweiCloudDeveloper/gaussdb-python-async) | [asyncpg](https://github.com/MagicStack/asyncpg)               | [SqlAlchemy](https://github.com/sqlalchemy/sqlalchemy) ||
|NodeJS| [gaussdb-node](https://github.com/HuaweiCloudDeveloper/gaussdb-node)                 | [node-postgres](https://github.com/brianc/node-postgres)       | [TypeORM](https://github.com/typeorm/typeorm)          ||
|.NET| [gaussdb-dotnet](https://github.com/HuaweiCloudDeveloper/gaussdb-dotnet)               | [npgsql](https://github.com/npgsql/npgsql)                     | [Entity Framework Core](https://github.com/HuaweiCloudDeveloper/gaussdb-dotnet-efcore)      ||
|PHP| [gaussdb-php](https://github.com/HuaweiCloudDeveloper/gaussdb-php)               | [pdo_pgsql](https://github.com/php/php-src/tree/master/ext/pdo_pgsql)                     | [Laravel](https://github.com/laravel/laravel)      |代码需要推送PHP官网|
|Rust| [gaussdb-rust](https://github.com/HuaweiCloudDeveloper/gaussdb-rust)               | [rust-postgres](https://github.com/sfackler/rust-postgres/)                     | [Diesel](https://github.com/diesel-rs/diesel)      ||

## 第一阶段目标
* 驱动可以连接gaussdb，基本功能正常
* 能够至少支撑一个ORM框架使用该驱动适配

## 第一阶段参考步骤
* 下载项目，写一个简单测试程序，针对PostgreSQL编译和测试通过。
* 修改认证逻辑
* 测试程序针对GaussDB编译和测试通过
* 重构group-id, artifactid, 重构package
* 针对PostgreSQL编译和测试通过

# 标准规范

## JAVA

* 驱动名称: gaussdb-r2dbc
* group-id: com.huaweicloud.gaussdb
* artifactid: gaussdb-r2dbc
* package: com.huaweicloud.gaussdb

## GO

* 驱动名称: gaussdb-go
* 模块路径: github.com/HuaweiCloudDeveloper/gaussdb-go
* 包名: gaussdb

## NODEJS

* 驱动名称: gaussdb-node
* 模块路径: github.com/HuaweiCloudDeveloper/gaussdb-node
* 包名: gaussdb


# 详细开发任务

详细开发任务在issues里面提交： https://github.com/HuaweiCloudDeveloper/gassdb-drivers/issues 

