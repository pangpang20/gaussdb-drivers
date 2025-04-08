# GaussDB和PostgresSQL功能和语法差异

## GaussDB与PostgreSQL存在差异

### 实现 Upsert(update + insert) 功能语法差异

* PosgreSQL写法

```
INSERT INTO distributors (did, dname)
VALUES (5, 'Gizmo Transglobal'), 
(6, 'Associated Computing, Inc')
ON CONFLICT (did) DO UPDATE SET
dname = EXCLUDED.dname
WHERE zipcode <> '21201';
```

* GaussDB写法

```
INSERT INTO distributors (did, dname)
VALUES (5, 'Gizmo Transglobal'), 
(6, 'Associated Computing, Inc')
ON DUPLICATE KEY UPDATE 
dname = VALUES(dname)
WHERE zipcode <> '21201';
```

* 补充说明

参考链接：
    * https://www.postgresql.org/docs/current/sql-insert.html
    * https://support.huaweicloud.com/centralized-devg-v8-gaussdb/gaussdb-42-0653.html#section4

### 实现数据实时复制功能

* PosgreSQL写法

```
CREATE PUBLICATION alltables FOR ALL TABLES; -- 发布所有表
CREATE PUBLICATION mypublication FOR TABLE table_name; -- 发布单张表
CREATE SUBSCRIPTION mysub CONNECTION 'host=XX.XX.XX.XXX port=8000 user= user_name dbname= dbname password= ***'
  PUBLICATION mypublication; -- 订阅名称为'mypublication'的发布
```

* GaussDB写法

参考 [逻辑复制](https://doc.hcs.huawei.com/db/zh-cn/gaussdbqlh/24.7.30.10/fg-dist/gaussdb-18-0030.html)

* 补充说明

参考链接：
    * https://bbs.huaweicloud.com/forum/thread-0211178860880205142-1-1.html

### gaussdbjdbc.jar的executeBatch返回值不符合JDBC规范

* 补充说明

参考链接：
    * https://bbs.huaweicloud.com/forum/thread-02104174303512776081-1-1.html

### SELECT pg_type.* FROM pg_catalog.pg_type 缺少oid字段

* 补充说明

参考链接：
    * https://bbs.huaweicloud.com/forum/thread-0274178187455582064-1-1.html

### GaussDB不支持JSON和JSONB类型的隐式转换

* 补充说明

参考链接：
    * https://bbs.huaweicloud.com/forum/thread-0211179461468355165-1-1.html

## GaussDB不存在的功能

### 不支持refcursor关键字

* 补充说明

参考链接：
* https://bbs.huaweicloud.com/forum/thread-0211178353127283084-1-1.html

### 不支持 CREATE/ALTER/DROP DOMAIN

* 补充说明

参考链接：
* https://bbs.huaweicloud.com/forum/thread-0210178360425351069-1-1.html

### 不支持 MultiRange

* 补充说明

参考链接：
* https://bbs.huaweicloud.com/forum/thread-0211178940977329145-1-1.html

### 不支持 LargeObject

* 补充说明

参考链接：
* https://bbs.huaweicloud.com/forum/thread-0211178941356334146-1-1.html

### 不支持 Serializable

* 补充说明

参考链接：
* https://bbs.huaweicloud.com/forum/thread-0213178941810463121-1-1.html
* https://support.huaweicloud.com/intl/zh-cn/centralized-devg-v2-gaussdb/gaussdb_42_0501.html

### 不支持 临时表Serial

* 补充说明

参考链接：
* https://bbs.huaweicloud.com/forum/thread-0213178941810463121-1-1.html

### 不支持 LISTEN/NOFITY statement

* 补充说明

参考链接：
* https://bbs.huaweicloud.com/forum/thread-0210178943513179110-1-1.html
* https://bbs.huaweicloud.com/forum/thread-0234178943691111112-1-1.html

### 不支持 line 类型

line类型是PostgreSQL新增加的用于描述平面直线的数据类型，GaussDB不存在对应的类型。 

## GaussDB已知缺陷

### SET LOCK_TIMEOUT提示错误

* 补充说明

参考链接：
    * https://bbs.huaweicloud.com/forum/thread-02127178278029106066-1-1.html

### 启动statement cache并且表结构发生变化时，数据库返回错误码不正确

参考链接：
    * https://bbs.huaweicloud.com/forum/thread-02127179062938863125-1-1.html

应用程序在这个场景会捕获到数据库异常，最佳的状态是能够自动恢复。客户端驱动如果要做到自动恢复，需要数据库针对这个异常返回不一样的错误码。

## JDBC(gaussjdbc.jar)已知缺陷

### FETCH FIRST n ROWS提示语法错误

* 补充说明

参考链接：
    * https://bbs.huaweicloud.com/forum/thread-02104174364146264084-1-1.html

### 创建Date类型字段的表，实际类型为timestamp(0)

* 补充说明

参考链接：
    * https://bbs.huaweicloud.com/forum/thread-0234179025026914116-1-1.html
