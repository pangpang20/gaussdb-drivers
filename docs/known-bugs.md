# GaussDB、OpenGauss、JDBC驱动已知缺陷

1. OpenGauss已知缺陷
2. GaussDB已知缺陷
3. JDBC(gaussjdbc.jar)已知缺陷

请按照如下模板更新本文档
> 
----------
* 问题描述

XXXXX

* 缺陷版本: XXXXX
* 修复版本: XXXXX
* 参考信息：XXXXX

>
----------

## OpenGauss已知缺陷

###  to_char函数预期结果不对
* 问题描述

`select to_char(time '12:13:14', 'HH:MI:SS')`预期结果为 `12:13:14`，实际为 `+00 12:13:14` 。

* 缺陷版本：7.0.0-RC1.B023
* 修复版本：7.0.0RC2B017

* 参考信息：https://gitcode.com/opengauss/openGauss-server/issues/7284

## GaussDB已知缺陷

### SET LOCK_TIMEOUT提示错误

* 补充说明

参考链接：
    * https://bbs.huaweicloud.com/forum/thread-02127178278029106066-1-1.html

### FETCH FIRST n ROWS绑定参数n时提示语法错误

* 补充说明

参考链接：
    * https://bbs.huaweicloud.com/forum/thread-02104174364146264084-1-1.html

### 启动statement cache并且表结构发生变化时，数据库返回错误码不正确

参考链接：
    * https://bbs.huaweicloud.com/forum/thread-02127179062938863125-1-1.html

应用程序在这个场景会捕获到数据库异常，最佳的状态是能够自动恢复。客户端驱动如果要做到自动恢复，需要数据库针对这个异常返回不一样的错误码。

### 字符串转为float4/float8的时候, 跟原值不一样
PostgreSQL是与原值一致的, GaussDB会在不同的目标类型返回不同的值

PostgreSQL:

```sql
-- GaussDB返回1.23000002
postgres=# select '1.23'::float4;
 float4
--------
   1.23
(1 row)

-- GaussDB返回1.22999999999999998
postgres=# select '1.23'::float8;
 float8
--------
   1.23
(1 row)
```

参考链接
*  https://bbs.huaweicloud.com/forum/thread-0204180071486749008-1-1.html

### 通过libpq库以BINARY格式执行COPY操作时，偶尔会出现数据读取错误

在psycopg3中执行以下COPY语句后进行数据读取，底层会调用libpq库的PQgetCopyData函数，可能会报错：`receiving copy data failed: no COPY in progress`

```sql
create table "fake_table" (id serial primary key, "fld_0" "int2", "fld_1" "int4", "fld_2" "bytea", "fld_3" "uuid", "fld_4" "bytea", "fld_5" "int4", "fld_6" "bytea", "fld_7" "jsonb", "fld_8" "float4", "fld_9" "int4", "fld_10" "jsonb", "fld_11" "numeric", "fld_12" "daterange", "fld_13" "uuid", "fld_14" "int8", "fld_15" "bytea", "fld_16" "int4", "fld_17" "int2", "fld_18" "int8range", "fld_19" "int4");

copy (select "fld_0", "fld_1", "fld_2", "fld_3", "fld_4", "fld_5", "fld_6", "fld_7", "fld_8", "fld_9", "fld_10", "fld_11", "fld_12", "fld_13", "fld_14", "fld_15", "fld_16", "fld_17", "fld_18", "fld_19" from "fake_table" order by id) to stdout (format BINARY);
```

* 补充说明

参考连接：

* https://bbs.huaweicloud.com/forum/thread-0255184575549538002-1-1.html

## JDBC(gaussjdbc.jar)已知缺陷



### 创建Date类型字段的表，实际类型为timestamp(0)

* 补充说明

参考链接：
    * https://bbs.huaweicloud.com/forum/thread-0234179025026914116-1-1.html

### gaussdbjdbc.jar的executeBatch返回值不符合JDBC规范

* 补充说明

参考链接：
    * https://bbs.huaweicloud.com/forum/thread-02104174303512776081-1-1.html

### 获取byte二位数组失败

```
final byte[][] byteArray = session.find( EntityWithDoubleByteArray.class, id ).getByteArray();

java.sql.SQLFeatureNotSupportedException: Method com.huawei.gaussdb.jdbc.jdbc.PgArray.getArrayImpl(long,int,Map) is not yet implemented.
```
