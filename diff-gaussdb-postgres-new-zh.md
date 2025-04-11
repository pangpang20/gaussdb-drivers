# GaussDB与PostgreSQL功能及语法差异指南

## 目录
1. **[核心功能差异](#核心功能差异)**  
   1.1 Upsert操作语法  
   1.2 数据实时复制实现  
   1.3 JSON处理差异  
   1.4 数组函数行为差异

2. **[不支持的功能特性](#不支持的功能特性)**  
   2.1 关键字限制  
   2.2 数据类型缺失  
   2.3 管理功能差异

3. **[语法兼容性问题](#语法兼容性问题)**  
   3.1 表操作限制  
   3.2 函数语法差异  
   3.3 复杂查询支持

4. **[已知产品缺陷](#已知产品缺陷)**  
   4.1 锁机制异常  
   4.2 运算精度问题  
   4.3 元数据管理差异

5. **[JDBC驱动问题](#jdbc驱动问题)**  
   5.1 批量操作返回值  
   5.2 类型映射异常  
   5.3 高级功能缺失

---

## 核心功能差异

### 1.1 Upsert操作语法
**场景说明**  
实现存在更新/不存在插入的原子操作

```sql
-- PostgreSQL (ON CONFLICT语法)
INSERT INTO distributors (did, dname)
VALUES (5, 'Gizmo')
ON CONFLICT (did) 
DO UPDATE SET dname = EXCLUDED.dname
WHERE zipcode <> '21201';

-- GaussDB (ON DUPLICATE KEY语法)
INSERT INTO distributors (did, dname)
VALUES (5, 'Gizmo')
ON DUPLICATE KEY UPDATE 
dname = VALUES(dname)
WHERE zipcode <> '21201';
```

**关键差异**
- 冲突检测：PostgreSQL使用`ON CONFLICT`+约束名，GaussDB使用`ON DUPLICATE KEY`
- 更新值引用：PostgreSQL通过`EXCLUDED`伪表，GaussDB使用`VALUES()`函数

> 参考链接：[华为云Upsert说明](https://support.huaweicloud.com/centralized-devg-v8-gaussdb/gaussdb-42-0653.html#section4)

---

### 1.2 数据实时复制实现
**逻辑复制配置对比**

```sql
-- PostgreSQL发布订阅
CREATE PUBLICATION all_tables FOR ALL TABLES;
CREATE SUBSCRIPTION sub_1 
CONNECTION 'host=10.0.0.1 port=5432 user=rep'
PUBLICATION all_tables;

-- GaussDB逻辑复制
/* 需使用华为自研工具链配置 */
```

**主要限制**
- GaussDB不支持ALL TABLES通配符发布
- 需要配置专属连接参数
- 管理接口不同（需使用华为云控制台）

> 参考文档：[GaussDB逻辑复制指南](https://doc.hcs.huawei.com/db/zh-cn/gaussdbqlh/24.7.30.10/fg-dist/gaussdb-18-0030.html)

---

### 1.3 JSON处理差异
#### 类型转换限制
```sql
-- PostgreSQL支持隐式转换
SELECT '{"a":1}'::jsonb || '{"b":2}';  -- 成功

-- GaussDB需显式转换
SELECT CAST('{"a":1}' AS JSONB) || '{"b":2}';  -- 需显式转换
```

#### 字段查询差异
```sql
-- PostgreSQL JSON路径查询
SELECT json_query(data, '$.items[*].id' WITH WRAPPER);

-- GaussDB替代方案
SELECT json_build_array(data::json #> '{items,*,id}');
```

**注意**
- GaussDB缺少`json_table`函数支持
- 深度嵌套查询需使用`#>`操作符手动解析

---

### 1.4 数组函数行为差异
#### 元素定位
```sql
-- PostgreSQL (返回第一个匹配位置)
SELECT array_position(array['a','b','c'], 'b');  -- 返回2

-- GaussDB (返回数组形式)
SELECT (array_positions(array['a','b','c'], 'b'))[1];  -- 需取数组元素
```

### 1.5 复杂类型操作限制
#### JSON字段更新限制
```sql
-- Hibernate写法
update JsonHolder b set b.aggregate.theString = null
-- PostgreSQL写法
update JsonHolder jh1_0 set aggregate=coalesce(jh1_0.aggregate,'{}')||jsonb_build_object('theString',to_jsonb(cast(null as varchar)))

-- GaussDB不支持
```

---

#### 数组截断
```sql
-- PostgreSQL标准语法
SELECT trim_array(array[1,2,3,4], 1);

-- GaussDB特有函数
SELECT array_trim(array[1,2,3,4], 1);
```

---

## 不支持的功能特性

### 2.1 关键字限制
| 功能                | PostgreSQL支持          | GaussDB替代方案       |
|---------------------|-------------------------|-----------------------|
| 游标操作            | DECLARE refcursor       | 不支持                |
| 超级用户权限        | CREATE USER ... SUPERUSER | 使用SYSADMIN角色      |
| 密码加密类型设置    | password_encryption=md5 | password_encryption_type参数 |

---

### 2.2 数据类型缺失
| 类型            | 用途描述                  | 影响场景示例          |
|-----------------|--------------------------|-----------------------|
| line            | 平面直线几何类型          | GIS系统迁移需改造      |
| ltree           | 层次树结构扩展            | 需改用邻接表或闭包表  |
| MultiRange       | 复合范围类型              | 时间区间处理需重构    |

---

### 2.3 管理功能差异
**重要限制项**
1. **DOMAIN类型**  
   不支持CREATE/ALTER/DROP DOMAIN，需改用CHECK约束

2. **大对象存储**  
   无法使用LargeObject API，需改用BYTEA类型

3. **事务隔离级别**  
   不支持SERIALIZABLE级别，最高支持REPEATABLE READ

> 参考问题：[隔离级别说明](https://support.huaweicloud.com/intl/zh-cn/centralized-devg-v2-gaussdb/gaussdb_42_0501.html)

---

## 语法兼容性问题

### 3.1 表操作限制
**系统同名表冲突**
```sql
-- 两库均存在PLAN_TABLE时
/* PostgreSQL可正常操作 */
INSERT INTO plan_table (...) VALUES (...);

-- GaussDB会优先访问系统表
ORA-00942: 表或视图不存在
```

**解决方案**
- 重命名用户表避免与系统表同名
- 使用schema限定访问路径

---

### 3.2 函数语法差异
**数学函数支持情况**
```sql
-- PostgreSQL支持
SELECT log(10, 100);  -- 返回2
SELECT sinh(1.0);     -- 返回双曲正弦值

-- GaussDB报错
[20100] ERROR: Unsupported function
```

**替代方案**
- 使用`ln()`代替自然对数
- 通过UDF实现双曲函数

---

### 3.3 复杂查询支持
**JSON聚合处理**
```sql
-- PostgreSQL标准语法
SELECT json_objectagg(key:value);

-- GaussDB实现方式
SELECT json_object_agg(
  CASE WHEN key IS NOT NULL THEN key END, 
  value
);
```

**注意点**
- 必须处理NULL键值
- 需要显式类型转换
- 不支持WITH UNIQUE KEYS等高级选项

---

### 3.4 高级函数限制
#### JSON_TABLE不支持
```sql
-- PostgreSQL支持标准语法
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

-- GaussDB 不支持
```

**影响范围**
- 复杂JSON解析需手动拆分步骤
- 嵌套数据遍历效率降低
- ORM框架自动映射失效

---

## 已知产品缺陷

### 4.1 锁机制异常
**问题现象**
```sql
SET LOCK_TIMEOUT = 3000;  -- GaussDB报语法错误
```

**临时方案**  
使用华为特有参数配置：
```ini
# 连接参数中设置
lock_wait_timeout = 3s
```

---

### 4.2 运算精度问题
**整数除法差异**
```sql
SELECT 4/3;  
-- PostgreSQL返回1
-- GaussDB返回1.33333333333333326
```

**影响范围**
- 财务计算精度异常
- 分页计算偏移量错误

**解决方案**  
显式使用整数除法：
```sql
SELECT div(4,3);  -- 返回1
```

---

### 4.3 元数据管理差异
**系统表字段缺失**
```sql
SELECT oid FROM pg_type;  -- GaussDB无oid列
```

**应对措施**  
使用华为提供的替代视图：
```sql
SELECT * FROM hg_catalog.hg_type;
```

---

## JDBC驱动问题

### 5.1 批量操作返回值
**规范不符现象**
```java
// 执行批量插入
int[] counts = statement.executeBatch();
// GaussDB返回-2(UNKNOWN_COUNT)而非实际影响行数
```

**影响范围**
- 无法准确判断批量操作结果
- ORM框架的批处理失效

---

### 5.2 类型映射异常
**日期类型问题**
```sql
CREATE TABLE test (date_col DATE);  
-- GaussDB实际创建为timestamp(0)
```

**导致后果**
- MyBatis等框架类型处理错误
- 时间精度意外丢失

---

### 5.3 高级功能缺失
**二进制数组支持**
```java
byte[][] data = resultSet.getArray(1).getArray();  
// 抛出SQLFeatureNotSupportedException
```

**临时方案**  
手动解析BYTEA格式：
```java
String hexStr = resultSet.getString(1);
byte[] data = Hex.decodeHex(hexStr);
```

