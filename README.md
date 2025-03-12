# 总体计划

|语言| 驱动名称                                                                                 | 对应PostgreSQL驱动                                                 | ORM框架                                                  |备注|
|----|--------------------------------------------------------------------------------------|----------------------------------------------------------------|--------------------------------------------------------|----|
|JAVA| [gaussdb-r2dbc](https://github.com/HuaweiCloudDeveloper/gaussdb-r2dbc)                  | [r2dbc-postgresql](https://github.com/pgjdbc/r2dbc-postgresql) | [Spring Data R2DBC](https://github.com/spring-projects/spring-data-relational/)                                  ||
|GO| [gaussdb-go](https://github.com/HuaweiCloudDeveloper/gaussdb-go)                     | [pgx](https://github.com/jackc/pgx)                            | [GORM](https://github.com/go-gorm/gorm)                ||
|Python| [gaussdb-python](https://github.com/HuaweiCloudDeveloper/gaussdb-python)             | [psycopg](https://github.com/psycopg/psycopg)                  | [SqlAlchemy](https://github.com/sqlalchemy/sqlalchemy) ||
|Python| [gaussdb-python-async](https://github.com/HuaweiCloudDeveloper/gaussdb-python-async) | [asyncpg](https://github.com/MagicStack/asyncpg)               | [SqlAlchemy](https://github.com/sqlalchemy/sqlalchemy) ||
|NodeJS| [gaussdb-node](https://github.com/HuaweiCloudDeveloper/gaussdb-node)                 | [node-postgres](https://github.com/brianc/node-postgres)       | [TypeORM](https://github.com/typeorm/typeorm)          ||
|.NET| [gaussdb-dotnet](https://github.com/HuaweiCloudDeveloper/gaussdb-dotnet)               | [npgsql](https://github.com/npgsql/npgsql)                     | [Entity Framework](https://github.com/dotnet/ef6)      ||
|PHP| [gaussdb-ph](https://github.com/HuaweiCloudDeveloper/gaussdb-php)               | [pdo_pgsql](https://github.com/php/php-src/tree/master/ext/pdo_pgsql)                     | [Laravel](https://github.com/laravel/laravel)      |代码需要推送PHP官网|

## 第一阶段目标
* 驱动可以连接gaussdb，基本功能正常
* 能够至少支撑一个ORM框架使用该驱动适配

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

# 重构步骤

## JAVA

* 下载项目，写一个简单测试程序，针对PostgreSQL编译和测试通过。
* 修改认证逻辑
* 测试程序针对GaussDB编译和测试通过
* 重构group-id, artifactid, 重构package
* 针对PostgreSQL编译和测试通过


## GO

1. 下载项目，针对PostgreSQL编译和测试通过
  * 下载 pgx 项目源码。
  * 确保项目能够在本地编译并通过所有PostgreSQL的测试用例。

2. 重构模块路径和包名
  * 将模块路径从 github.com/jackc/pgx 修改为 github.com/HuaweiCloudDeveloper/gaussdb-go。
  * 将包名从 pgx 修改为 gaussdb。
  * 更新所有导入路径和包引用。

3. 针对PostgreSQL编译和测试通过
  * 确保在修改模块路径和包名后，项目仍然能够编译并通过所有PostgreSQL的测试用例。

4. 修改认证逻辑
  * 根据GaussDB的认证机制，修改驱动中的认证逻辑。
  * 确保新的认证逻辑能够与GaussDB正常通信。

5. 针对GaussDB编译和测试通过
  * 针对GaussDB进行编译和测试，确保所有功能正常。

# 详细开发任务

详细开发任务在issues里面提交： https://github.com/HuaweiCloudDeveloper/gassdb-drivers/issues 

