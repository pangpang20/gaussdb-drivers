本项目的目的是帮助繁荣 [GaussDB](https://www.huaweicloud.com/product/gaussdb.html) 和 [OpenGauss](https://opengauss.org/zh/) 开源生态。

项目包括常见语言的驱动，这些驱动目前都通过PostgreSQL开源驱动进行构建。GaussDB开源生态信息参考 [gaussdb-ecosystem](https://github.com/HuaweiCloudDeveloper/gaussdb-ecosystem) 。

> ***如果发现项目能帮助到您，别忘了点击右上角`star`表示鼓励***

# 项目信息

| 语言   | 驱动名称                                                                          | 对应PostgreSQL驱动                                                 | 备注                |
| ------ | --------------------------------------------------------------------------------- | ------------------------------------------------------------------ | ------------------- |
| JAVA   | [gaussdb-r2dbc](https://github.com/HuaweiCloudDeveloper/gaussdb-r2dbc)               | [r2dbc-postgresql](https://github.com/pgjdbc/r2dbc-postgresql)        |        已发布             |
| GO     | [gaussdb-go](https://github.com/HuaweiCloudDeveloper/gaussdb-go)                     | [pgx](https://github.com/jackc/pgx)                                   |       已发布           |
| .NET   | [gaussdb-dotnet](https://github.com/HuaweiCloudDeveloper/gaussdb-dotnet)             | [npgsql](https://github.com/npgsql/npgsql)                            |         已发布             |
| Python | [gaussdb-python](https://github.com/HuaweiCloudDeveloper/gaussdb-python)             | [psycopg](https://github.com/psycopg/psycopg)                         |     开发中             |
| Python | [gaussdb-python-async](https://github.com/HuaweiCloudDeveloper/gaussdb-python-async) | [asyncpg](https://github.com/MagicStack/asyncpg)                      |    开发中               |
| NodeJS | [gaussdb-node](https://github.com/HuaweiCloudDeveloper/gaussdb-node)                 | [node-postgres](https://github.com/brianc/node-postgres)              |       开发中               |
| PHP    | [gaussdb-php](https://github.com/HuaweiCloudDeveloper/gaussdb-php)                   | [pdo_pgsql](https://github.com/php/php-src/tree/master/ext/pdo_pgsql) |  开发中 |
| Rust   | [gaussdb-rust](https://github.com/HuaweiCloudDeveloper/gaussdb-rust)                 | [rust-postgres](https://github.com/sfackler/rust-postgres/)           |       开发中               |

# 项目标准规范

## gaussdb-r2dbc

* 项目信息：

  * 仓库名称：gaussdb-r2dbc
* Maven发布：

  * group-id: com.huaweicloud.gaussdb
  * artifactid: gaussdb-r2dbc
* 应用开发：

  * package: com.huaweicloud.gaussdb
  * 连接串：`r2dbc:gaussdb://<host>:<port>/<database>`
  * 驱动名称: `gaussdb`（ConnectionFactoryOptions）

## gaussdb-go

* 项目信息
  * 仓库名称: gaussdb-go
* pkg.go.dev发布
  * package name: github.com/HuaweiCloudDeveloper/gaussdb-go
* 应用开发
  * package：github.com/HuaweiCloudDeveloper/gaussdb-go
  * 连接串：`gaussdb://<username>:<password>@<host>:<port>/<database>`
  * 驱动名称：`gaussdb`

## gaussdb-dotnet

* 项目信息
  * 仓库名称：gaussdb-dotnet
* NuGet发布
  * 包名：HuaweiCloud.Driver.GaussDB
* 应用开发
  * namespace：HuaweiCloud.GaussDB
  * 连接串：`gaussdb://<username>:<password>@<host>:<port>/<database>`/`Host=<host>:<port>;Username=<username>;Password=<password>;Database=<database>`
  * 驱动名称：GaussDB

## gaussdb-node

* 项目名称: gaussdb-node
* 模块路径: github.com/HuaweiCloudDeveloper/gaussdb-node
* 包名: gaussdb


# 发布信息

## JDBC驱动

> 需要使用JDK 17及其以上版本

* 连接串：jdbc:gaussdb://host:port/database

* Driver名称： com.huawei.gaussdb.jdbc.Driver

* maven

```xml
<dependencies>
  <dependency>
    <groupId>com.huaweicloud.gaussdb</groupId>
    <artifactId>gaussdbjdbc</artifactId>
    <version>506.0.0.b058</version>
  </dependency>
</dependencies>
```

* gradle

```gradle
dependencies {
    implementation 'com.huaweicloud.gaussdb:gaussdbjdbc:506.0.0.b058'
}  
```

## R2DBC驱动

> 需要使用JDK 17及其以上版本

```xml
<dependencies>
  <dependency>
    <groupId>com.huaweicloud.gaussdb</groupId>
    <artifactId>gaussdb-r2dbc</artifactId>
    <version>1.0.0.RC1</version>
  </dependency>
</dependencies>
```

* gradle

```gradle
dependencies {
    implementation 'com.huaweicloud.gaussdb:gaussdb-r2dbc:1.0.0.RC1'
}  
```

## GO驱动

版本发布：https://pkg.go.dev/github.com/HuaweiCloudDeveloper/gaussdb-go

## .NET驱动

版本发布：https://www.nuget.org/packages/HuaweiCloud.Driver.GaussDB/

# 开源开发规范

* 按照主仓库代码模板设置IDE，确保提交的代码格式符合主仓库要求。
* 采用FORK - PR开发流程，通过PR给主仓库提交修改。
* 鼓励将功能拆解为独立的小特性，每个commit只包含一个特性，代码修改规模控制在200行以内。
* PR需要通过主仓库配置的各项代码检查（自动化测试、代码静态检查等），积极配合修复代码检查发现的问题。
* 积极配合修复代码检视意见，针对检视意见都有回应和闭环。
