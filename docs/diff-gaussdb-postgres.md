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

### JDBC使用逻辑复制功能

* 补充说明
PostgreSQL使用逻辑复制连接的端口与普通SQL端口一致，默认是5432, PostgreSQL需要单独配置逻辑复制权限和逻辑复制槽。

GaussDB使用的逻辑复制端口与普通SQL端口不同，通常为普通端口+1。 比如集中式版本为8001，分布式版本为CN（协调节点）和DN（数据节点）的端口+1. 

参考链接：
  * https://bbs.huaweicloud.com/forum/thread-0228180067001483008-1-1.html

### SELECT pg_type.* FROM pg_catalog.pg_type 缺少oid字段

* 补充说明

参考链接：
    * https://bbs.huaweicloud.com/forum/thread-0274178187455582064-1-1.html

### GaussDB不支持JSON和JSONB类型的隐式转换

PostgreSQL支持隐式转换
```sql
SELECT '{"a":1}'::jsonb || '{"b":2}';  -- 成功
```

GaussDB需显式转换
```sql
SELECT CAST('{"a":1}' AS JSONB) || '{"b":2}'; 
```

PostgreSQL JSON路径查询
```sql
SELECT json_query(data, '$.items[*].id' WITH WRAPPER);
```

GaussDB替代方案
```sql
SELECT json_build_array(data::json #> '{items,*,id}');
```

* 补充说明

参考链接：
    * https://bbs.huaweicloud.com/forum/thread-0211179461468355165-1-1.html

### 不支持操作与系统表名或者视图名一致的表
* GaussDB写法
```
insert into PLAN_TABLE (name,id) values (?,?)
```
* PosgreSQL写法
```
insert into PLAN_TABLE (name,id) values (?,?)
```

### 数组函数行为差异

* GaussDB返回数组形式

```
select
        ewa1_0.id,
        ewa1_0.the_array,
        ewa1_0.the_label,
        ewa1_0.the_labels 
    from
        EntityWithArrays ewa1_0 
    where
        (
            array_positions(ewa1_0.the_array, 'xyz')
        )[1]=0
```

* PostgreSQL (返回第一个匹配位置)
```
select
        ewa1_0.id,
        ewa1_0.the_array,
        ewa1_0.the_label,
        ewa1_0.the_labels 
    from
        EntityWithArrays ewa1_0 
    where
        case 
            when ewa1_0.the_array is not null 
                then coalesce(array_position(ewa1_0.the_array, 'xyz'), 0) 
        end=0
```

### array_trim(e.theArray, 1)函数表现不一致

* GaussDB写法
```
select
        array_trim(ewa1_0.the_array, 1) 
    from
        EntityWithArrays ewa1_0 
    where
        ewa1_0.id=1
```
* PosgreSQL写法
```
select
        trim_array(ewa1_0.the_array, 1) 
    from
        EntityWithArrays ewa1_0 
    where
        ewa1_0.id=1
```

### json_query(e.json, '$.theNestedObjects[*].id' with wrapper)函数表现不一致
* GaussDB写法
```
select
        json_build_array(jh1_0.json::json #> '{theNestedObjects,*,id}') 
    from
        JsonHolder jh1_0 
    where
        jh1_0.id=1
```
* PosgreSQL写法
```
select
        json_query(jh1_0.json, '$.theNestedObjects[*].id' with wrapper) 
    from
        JsonHolder jh1_0 
    where
        jh1_0.id=1
```

### json_objectagg(e.theString value e.theUuid)函数表现不一致
* GaussDB写法
```
select
        json_object_agg(CASE 
            WHEN eob1_0.the_string IS NOT NULL   
                THEN eob1_0.the_string 
        END, eob1_0.theuuid) 
    from
        EntityOfBasics eob1_0
```
* PosgreSQL写法
```
select
        json_objectagg(eob1_0.the_string:eob1_0.theuuid absent 
            on null 
    returning jsonb) from
        EntityOfBasics eob1_0
```

### json_objectagg(e.theString value e.theUuid)函数表现不一致
* GaussDB写法
```
select
        json_object_agg(CASE 
            WHEN cast(eob1_0.the_integer as varchar) IS NOT NULL   
                THEN cast(eob1_0.the_integer as varchar) 
        END, eob1_0.the_string) 
    from
        EntityOfBasics eob1_0
```
* PosgreSQL写法
```
select
        json_objectagg(cast(eob1_0.the_integer as varchar):eob1_0.the_string absent 
            on null with unique keys 
    returning jsonb) from
        EntityOfBasics eob1_0
```

### json_objectagg(e.theString value e.theUuid)函数表现不一致
* GaussDB写法
```
select
        json_merge('{"a":456, "b":[1,2], "c":{"a":1}}', '{"a":null, "b":[4,5], "c":{"b":1}}')
```
* PosgreSQL写法
```
select
        (with recursive args(d0, d1) as(select
            cast('{"a":456, "b":[1,2], "c":{"a":1}}' as jsonb), cast('{"a":null, "b":[4,5], "c":{"b":1}}' as jsonb)),
        val0(p, k, v) as (select
            '{}'::text[], s.k, t.d0->s.k 
        from
            args t 
        join
            lateral jsonb_object_keys(t.d0) s(k) 
                on 1=1 
        union
        select
            v.p||v.k, s.k, v.v->s.k 
        from
            val0 v 
        join
            lateral jsonb_object_keys(v.v) s(k) 
                on jsonb_typeof(v.v)='object'), val1(p, k, v) as (select
            '{}'::text[], s.k, t.d1->s.k 
    from
        args t 
    join
        lateral jsonb_object_keys(t.d1) s(k) 
            on 1=1 
    union
    select
        v.p||v.k, s.k, v.v->s.k 
    from
        val1 v 
    join
        lateral jsonb_object_keys(v.v) s(k) 
            on jsonb_typeof(v.v)='object'), res(v, p, l) as(select
        jsonb_object_agg(coalesce(v1.k, v0.k), coalesce(v1.v, v0.v)), coalesce(v1.p, v0.p), cardinality(coalesce(v1.p, v0.p)) 
from
    val0 v0 
full join
    val1 v1 
        on v0.p=v1.p 
        and v0.k=v1.k 
where
    cardinality(coalesce(v1.p, v0.p))=(select
        cardinality(v.p) 
    from
        val0 v 
    union
    select
        cardinality(v.p) 
    from
        val1 v 
    order by
        1 desc 
    limit
        1) 
    and jsonb_typeof(coalesce(v1.v)) is distinct 
from
    'null' 
group by
    coalesce(v1.p, v0.p), cardinality(coalesce(v1.p, v0.p)) 
union
all select
    jsonb_object_agg(coalesce(v1.k, v0.k), coalesce(case 
        when coalesce(v1.k, v0.k)=r.p[cardinality(r.p)] 
            then r.v 
    end, v1.v, v0.v)) filter (
where
    coalesce(case 
        when coalesce(v1.k, v0.k)=r.p[cardinality(r.p)] 
            then r.v 
    end, v1.v, v0.v) is not null), coalesce(v1.p, v0.p), r.l-1 
from
    val0 v0 
full join
    val1 v1 
        on v0.p=v1.p 
        and v0.k=v1.k 
join
    (select
        * 
    from
        res r 
    order by
        r.l 
    fetch
        first 1 rows with ties) r 
        on cardinality(coalesce(v1.p, v0.p))=r.l-1 
        and jsonb_typeof(coalesce(v1.v)) is distinct 
from
    'null' 
    and r.l<>0 
group by
    coalesce(v1.p, v0.p), r.l-1) select
    r.v 
from
    res r 
where
    r.l=0
)
```

### GaussDB NUMERIC 类型精度没PostgreSQL高
* 补充说明

参考链接：
* https://bbs.huaweicloud.com/forum/thread-0211180070242768009-1-1.html

### GaussDB数值类型（numeric）不支持Infinity（正无穷）和-Infinity（负无穷）
* 补充说明

参考链接：
* https://bbs.huaweicloud.com/forum/thread-0202180070831076010-1-1.html

### GaussDB的SEQUENCE不支持修改类型
GaussDB会报: `syntax error at or near "AS"`
* PosgreSQL写法  
```sql
ALTER SEQUENCE IF EXISTS events_uid_seq AS integer
```

参考连接:  
https://doc.hcs.huawei.com/db/en-us/gaussdb/24.1.30/devg-dist/gaussdb-12-0504.html

### GaussDB创建表必须至少有一列
GaussDB会报: `ERROR:  There must be at least one column.`
* PosgreSQL写法
```sql
CREATE TABLE "objects" ()
```

### 除法返回结果差异
执行`select 4/3;`

GaussDB会返回: `1.33333333333333326`

PosgreSQL会返回:1

参考链接：
* https://bbs.huaweicloud.com/forum/thread-0233180004755556002-1-1.html


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

### 不支持临时表Serial

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

### 不支持ltree的extenstion
会报类似错误: `ERROR:  Failed to open the extension control file: No such file or directory.`

PosgreSQL:
```sql
postgres=# create extension ltree;
CREATE EXTENSION
postgres=#
```
参考链接：
* https://bbs.huaweicloud.com/forum/thread-0211180003903912003-1-1.html

### 不支持SUPERUSER关键字
需要使用`SYSADMIN`代替.

PosgreSQL:
```sql
postgres=# create user pgx_pw with SUPERUSER PASSWORD 'Gaussdb@123!';
CREATE ROLE
postgres=#
```

参考链接：
* https://bbs.huaweicloud.com/forum/thread-0238179049931706106-1-1.html

### 不支持password_encryption关键字
需要使用`password_encryption_type`代替, 也不能动态设置.

PosgreSQL:
```sql
postgres=# set password_encryption = md5;
SET
postgres=#
```
参考链接：
* https://doc.hcs.huawei.com/db/zh-cn/gaussdb/24.1.30/devg-dist/gaussdb-12-1691.html
* https://bbs.huaweicloud.com/forum/thread-0207180004302613002-1-1.html

### 不支持unix_socket_directories关键字
需要使用`unix_socket_directory`代替.

PosgreSQL:
```sql
postgres=# show unix_socket_directories;
 unix_socket_directories
-------------------------
 /var/run/postgresql
(1 row)
```
参考链接：
* https://doc.hcs.huawei.com/db/zh-cn/gaussdb/24.1.30/devg-dist/gaussdb-12-1690.html


### 不支持**DISCARD**关键字

- Discard非保留关键字，且在GaussDB中执行后出现异常 "DISCARD statement is not yet supported."

参考连接：

https://support.huaweicloud.com/centralized-devg-v3-gaussdb/gaussdb-42-0327.html


### 不支持**UNLISTEN**关键字

- UNLISTEN非保留关键字，且在GaussDB中执行后出现异常 "UNLISTENstatement is not yet supported."

参考连接：

https://support.huaweicloud.com/centralized-devg-v3-gaussdb/gaussdb-42-0327.html

### unnest语法不支持
* GaussDB写法
```
select
        b1_0.id,
        p1_0.name,
        l1_0.name,
        l1_0.val 
    from
        Book b1_0 
    join
        unnest(b1_0.publishers) p1_0 
            on true 
    join
        unnest(b1_0.labels) l1_0 
            on true 
    order by
        b1_0.id,
        p1_0.name asc nulls first,
        l1_0.name asc nulls first,
        l1_0.val asc nulls first
```
* PosgreSQL写法
```
select
        b1_0.id,
        p1_0.name,
        l1_0.name,
        l1_0.val 
    from
        Book b1_0 
    join
        lateral unnest(b1_0.publishers) p1_0 
            on true 
    join
        lateral unnest(b1_0.labels) l1_0 
            on true 
    order by
        b1_0.id,
        p1_0.name asc nulls first,
        l1_0.name asc nulls first,
        l1_0.val asc nulls first
```

### json_table不支持
* GaussDB写法
```
select
        t1_0.theInt,
        t1_0.theFloat,
        t1_0.theString,
        t1_0.theBoolean,
        t1_0.theNull,
        t1_0.theObject,
        t1_0.theNestedInt,
        t1_0.theNestedFloat,
        t1_0.theNestedString,
        t1_0.arrayIndex,
        t1_0.arrayValue,
        t1_0.nonExisting 
    from
        EntityWithJson ewj1_0 
    join
        json_table(ewj1_0.json, '$' columns(theInt integer path '$.theInt', theFloat float path '$.theFloat', theString text path '$.theString', theBoolean boolean path '$.theBoolean', theNull text path '$.theNull', theObject jsonb path '$.theObject', theNestedInt integer path '$.theObject.theInt', theNestedFloat float path '$.theObject.theFloat', theNestedString text path '$.theObject.theString', nested '$.theArray[*]' columns(arrayIndex for ordinality, arrayValue text path '$'), nonExisting boolean exists path '$.nonExisting')) t1_0 
            on true 
    order by
        ewj1_0.id,
        t1_0.arrayIndex
```
* PosgreSQL写法
```
select
        t1_0.theInt,
        t1_0.theFloat,
        t1_0.theString,
        t1_0.theBoolean,
        t1_0.theNull,
        t1_0.theObject,
        t1_0.theNestedInt,
        t1_0.theNestedFloat,
        t1_0.theNestedString,
        t1_0.arrayIndex,
        t1_0.arrayValue,
        t1_0.nonExisting 
    from
        EntityWithJson ewj1_0 
    join
        lateral json_table(ewj1_0.json, '$' columns(theInt integer path '$.theInt', theFloat float path '$.theFloat', theString text path '$.theString', theBoolean boolean path '$.theBoolean', theNull text path '$.theNull', theObject jsonb path '$.theObject', theNestedInt integer path '$.theObject.theInt', theNestedFloat float path '$.theObject.theFloat', theNestedString text path '$.theObject.theString', nested '$.theArray[*]' columns(arrayIndex for ordinality, arrayValue text path '$'), nonExisting boolean exists path '$.nonExisting')) t1_0 
            on true 
    order by
        ewj1_0.id,
        t1_0.arrayIndex
```

### index无法内聚为一个函数
* hibernate
```
select index(e), e from generate_series(2, 3, 1) e order by index(e)
* 
```
* PosgreSQL写法
```
select
        e1_0.ordinality,
        e1_0.e1_0 
    from
        generate_series(2, 3, 1) with ordinality e1_0 
    order by
        e1_0.ordinality
```

### 日期操作内存溢出
* GaussDB写法 
```
select (current_date-cast(? as date))*86400*1e9
```
* PosgreSQL写法
```
select (current_date-cast(? as date))*86400*1e9
```
* 补充说明 
```
产品特性，不同兼容模式有不同的行为。默认的兼容模式是内存溢出
```

### 对结构性字段如json的某字段的更新和查询不支持
* GaussDB写法 
```
update JsonHolder jh1_0 set aggregate=coalesce(jh1_0.aggregate,'{}')||jsonb_build_object('theString',to_jsonb(cast(null as varchar)))
```
* PosgreSQL写法
```
update
        JsonHolder jh1_0 
    set
        aggregate=coalesce(jh1_0.aggregate, '{}')||jsonb_build_object('theString',
        to_jsonb(cast(null as varchar)))
```

### 不支持在临时表上创建trigger
会报类似的错误: `ERROR:  Only support CREATE TRIGGER on regular row table.`

PosgreSQL:
```sql
postgres=# create temporary table sentences(
        t text,
        ts tsvector
);
CREATE TABLE
postgres=# create function pg_temp.sentences_trigger() returns trigger as $$
        begin
          new.ts := to_tsvector(new.t);
                return new;
        end
        $$ language plpgsql;
CREATE FUNCTION
postgres=# create trigger sentences_update before insert on sentences for each row execute procedure pg_temp.sentences_trigger();
CREATE TRIGGER
postgres=#
```
参考链接：
* https://bbs.huaweicloud.com/forum/thread-0228180004564653006-1-1.html

### 连接成功后,TCP包里返回的PID字段并不是当前连接的PID
wireshark抓包`Backend key data`的时候PostgerSQL会返回当前连接的PID, GaussDB也返回了这个字段,但是并不是当前连接的PID

参考链接:
* https://bbs.huaweicloud.com/forum/thread-0259179656679211013-1-1.html

### 不支持空字符转为xml的数据类型
会报类似的错误: `ERROR: invalid XML content (SQLSTATE 2200N)`

PosgreSQL:
```sql
postgres=# select ''::xml;
 xml
-----

(1 row)
```
参考链接:
* https://bbs.huaweicloud.com/forum/thread-0204180007865671004-1-1.html

### drop function的时候需要指定parameter
需要在函数名后面加上`(parameter)`
会报类似的错误: `ERROR:  syntax error at or near "cascade"`

PosgreSQL:
```sql
postgres=# create function test_trigger() returns trigger language plpgsql as $$
        begin
        if new.n = 4 then
                raise exception 'n cant be 4!';
        end if;
        return new;
end$$;
CREATE FUNCTION
postgres=# drop function if exists test_trigger cascade;
DROP FUNCTION
postgres=#
```
参考链接:
* https://doc.hcs.huawei.com/db/en-us/gaussdb/24.7.30.10/devg-dist/gaussdb-12-0595.html
* https://bbs.huaweicloud.com/forum/topicpost/1350

### 约束触发器的时候要使用`procedure`
PostgreSQL使用`function`关键字

PosgreSQL:
```sql
postgres=# create temporary table defer_test (
                        id text primary key,
                        n int not null, unique (n),
                        unique (n) deferrable initially deferred );
CREATE TABLE
postgres=# create function test_trigger() returns trigger language plpgsql as $$
        begin
        if new.n = 4 then
                raise exception 'n cant be 4!';
        end if;
        return new;
end$$;
CREATE FUNCTION
postgres=# create constraint trigger test
                        after insert or update on defer_test
                        deferrable initially deferred
                        for each row
                        execute function test_trigger();
CREATE TRIGGER
```
参考链接：
* https://bbs.huaweicloud.com/forum/thread-0233180008573088003-1-1.html

### 密码错误时返回的`Severity`是`ERROR`
用wireshark抓包时，发现返回的`Severity`是`ERROR`，而PosgreSQL返回的是`FATAL`。

参考链接
* https://bbs.huaweicloud.com/forum/thread-0207180008754310003-1-1.html

### 没有`pg_stat_ssl`这个视图
类似的视图也未找到

PosgreSQL:
```sql
postgres=# select ssl from pg_stat_ssl;
 ssl
-----
 f
 f
 f
(3 rows)
```
参考链接
* https://bbs.huaweicloud.com/forum/thread-0202180008880802008-1-1.html

### notice的返回里没有`V`这个field
通过wireshark抓包, 发现返回的`notice`里没有`V`这个field. 这个`V`的field在PostgreSQL里的作用是其value不会被localize

参考链接
* https://bbs.huaweicloud.com/forum/thread-0233180009314921004-1-1.html

### 一些错误返回信息的首字母是大写
不太清楚是所有的错误都是大写的, 还是只有一部分的错误是

比如: 
- 查询不存在的字段
    - PostgreSQL: `ERROR: relation "baz" does not exist on gaussdb (SQLSTATE 42P01)`
    - GaussDB: `ERROR: Relation "baz" does not exist on gaussdb (SQLSTATE 42P01)`
- 类型转换为不存在的类型: `select 'a'::badtype;`
    - PostgreSQL: `type "badtype" does not exist`
    - GaussDB: `Type "badtype" does not exist`

参考链接
*  https://bbs.huaweicloud.com/forum/thread-0202180086482842011-1-1.html


### 不支持 macaddr8类型

* 补充说明

参考链接：
* https://bbs.huaweicloud.com/forum/thread-0211180066801704005-1-1.html



### pg_attribute 中没有 attidentity字段

- 补充说明

  在.NET Npgsql.Schema GenerateColumnsQuery中会查询pg_attribute的attidentity，导致大量相关测试用例失败。



## GaussDB已知缺陷

### SET LOCK_TIMEOUT提示错误

* 补充说明

参考链接：
    * https://bbs.huaweicloud.com/forum/thread-02127178278029106066-1-1.html

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

### FETCH FIRST n ROWS提示语法错误

* 补充说明

参考链接：
    * https://bbs.huaweicloud.com/forum/thread-02104174364146264084-1-1.html

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

### bigint类型对''的处理不同

* GaussDB
数据库使用默认兼容模式A，即Oracle兼容模式，bigint类型对空字符串的处理不同。

```sql
test02=> create table testmany (a bigint, b bigint);
CREATE TABLE
test02=>  insert into testmany values (1, '');
INSERT 0 1
```

* PosgreSQL

```sql
postgres=# create table testmany (a bigint, b bigint);
CREATE TABLE
postgres=# insert into testmany values (1, '');
ERROR:  invalid input syntax for type bigint: ""
LINE 1: insert into testmany values (1, '');
```


### 嵌套事务隔离级别不同
* 测试脚本
```sql
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT 'Outer: ' || current_setting('transaction_isolation') AS isolation_level_outer;

SAVEPOINT sp1;

SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT 'Inner: ' || current_setting('transaction_isolation') AS isolation_level_inner;

ROLLBACK;
```  

* GaussDB
在 GaussDB 上，执行脚本不会报错，而且隔离级别会被更改：
```text
 isolation_level_outer 
-----------------------
 repeatable read
(1 row)

 isolation_level_inner 
-----------------------
 serializable
(1 row)
```

* PosgreSQL
在 PostgreSQL 上，执行 SET TRANSACTION ISOLATION LEVEL SERIALIZABLE; 时会报错：
```text
ERROR:  SET TRANSACTION ISOLATION LEVEL must be called before any query
```


### PostgreSQL与GaussDB中SET SESSION AUTHORIZATION 不同
参考链接：https://bbs.huaweicloud.com/forum/thread-0236186944607404002-1-1.html
