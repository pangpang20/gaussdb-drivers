# Functional and Syntactic Differences Between GaussDB and PostgreSQL

## Table of Contents
1. **[Core Functional Differences](#core-functional-differences)**  
   1.1 Upsert Syntax  
   1.2 Real-time Data Replication  
   1.3 JSON Handling Differences  
   1.4 Array Function Behavior

2. **[Unsupported Features](#unsupported-features)**  
   2.1 Keyword Restrictions  
   2.2 Missing Data Types  
   2.3 Management Feature Gaps

3. **[Syntax Compatibility Issues](#syntax-compatibility-issues)**  
   3.1 Table Operation Limitations  
   3.2 Function Syntax Differences  
   3.3 Complex Query Support

4. **[Known Product Defects](#known-product-defects)**  
   4.1 Lock Mechanism Anomalies  
   4.2 Arithmetic Precision Issues  
   4.3 Metadata Management Differences

5. **[JDBC Driver Issues](#jdbc-driver-issues)**  
   5.1 Batch Operation Return Values  
   5.2 Type Mapping Anomalies  
   5.3 Advanced Feature Limitations

---

## Core Functional Differences

### 1.1 Upsert Syntax
**Scenario**  
Atomic "Update if exists, else Insert" operations.

```sql
-- PostgreSQL (ON CONFLICT)
INSERT INTO distributors (did, dname)
VALUES (5, 'Gizmo')
ON CONFLICT (did) 
DO UPDATE SET dname = EXCLUDED.dname
WHERE zipcode <> '21201';

-- GaussDB (ON DUPLICATE KEY)
INSERT INTO distributors (did, dname)
VALUES (5, 'Gizmo')
ON DUPLICATE KEY UPDATE 
dname = VALUES(dname)
WHERE zipcode <> '21201';
```

**Key Differences**
- Conflict detection: PostgreSQL uses `ON CONFLICT` with constraint names, GaussDB uses `ON DUPLICATE KEY`
- Value reference: PostgreSQL uses `EXCLUDED` pseudo-table, GaussDB uses `VALUES()` function

> Reference: [Huawei Cloud Upsert Documentation](https://support.huaweicloud.com/centralized-devg-v8-gaussdb/gaussdb-42-0653.html#section4)

---

### 1.2 Real-time Data Replication
**Logical Replication Comparison**

```sql
-- PostgreSQL Publication/Subscription
CREATE PUBLICATION all_tables FOR ALL TABLES;
CREATE SUBSCRIPTION sub_1 
CONNECTION 'host=10.0.0.1 port=5432 user=rep'
PUBLICATION all_tables;

-- GaussDB Implementation
/* Requires Huawei proprietary toolchain */
```

**Key Limitations**
- GaussDB doesn't support `ALL TABLES` wildcard publications
- Requires custom connection parameters
- Different management interfaces (Huawei Cloud Console required)

> Reference: [GaussDB Logical Replication Guide](https://doc.hcs.huawei.com/db/zh-cn/gaussdbqlh/24.7.30.10/fg-dist/gaussdb-18-0030.html)

---

### 1.3 JSON Handling Differences
#### Type Conversion
```sql
-- PostgreSQL implicit conversion
SELECT '{"a":1}'::jsonb || '{"b":2}';  -- Works

-- GaussDB explicit cast required
SELECT CAST('{"a":1}' AS JSONB) || '{"b":2}'; 
```

#### Field Query Differences
```sql
-- PostgreSQL JSON Path Query
SELECT json_query(data, '$.items[*].id' WITH WRAPPER);

-- GaussDB Alternative
SELECT json_build_array(data::json #> '{items,*,id}');
```

**Important Notes**
- GaussDB lacks `json_table` function support
- Deep nesting requires manual parsing with `#>` operator

---

### 1.4 Array Function Behavior
#### Element Positioning
```sql
-- PostgreSQL (returns first match index)
SELECT array_position(array['a','b','c'], 'b');  -- Returns 2

-- GaussDB (returns array of positions)
SELECT (array_positions(array['a','b','c'], 'b'))[1]; 
```

#### Array Trimming
```sql
-- PostgreSQL Standard
SELECT trim_array(array[1,2,3,4], 1);

-- GaussDB Specific
SELECT array_trim(array[1,2,3,4], 1);
```

---

## Unsupported Features

### 2.1 Keyword Restrictions
| Feature             | PostgreSQL Support       | GaussDB Workaround       |
|---------------------|--------------------------|--------------------------|
| Cursor Operations   | DECLARE refcursor        | Not Supported            |
| Superuser Privilege | CREATE USER ... SUPERUSER| Use SYSADMIN role        |
| Password Encryption | password_encryption=md5  | password_encryption_type  |

---

### 2.2 Missing Data Types
| Type         | Description               | Migration Impact         |
|--------------|---------------------------|--------------------------|
| line         | Planar line geometry      | GIS system rework needed |
| ltree        | Hierarchical tree extension | Requires adjacency list  |
| MultiRange   | Composite range types     | Temporal logic rework    |

---

### 2.3 Management Feature Gaps
**Key Limitations**
1. **DOMAIN Types**  
   `CREATE/ALTER/DROP DOMAIN` not supported, use CHECK constraints instead

2. **Large Objects**  
   LOB API unavailable, use BYTEA instead

3. **Isolation Levels**  
   SERIALIZABLE level not supported, maximum REPEATABLE READ

> Reference: [Isolation Level Documentation](https://support.huaweicloud.com/intl/zh-cn/centralized-devg-v2-gaussdb/gaussdb_42_0501.html)

---

## Syntax Compatibility Issues

### 3.1 Table Operation Limitations
**System Table Naming Conflicts**
```sql
-- When PLAN_TABLE exists in both systems
/* PostgreSQL works normally */
INSERT INTO plan_table (...) VALUES (...);

-- GaussDB prioritizes system tables
ORA-00942: Table or view does not exist
```

**Solutions**
- Rename user tables to avoid system table conflicts
- Use schema-qualified access

---

### 3.2 Function Syntax Differences
**Mathematical Functions**
```sql
-- PostgreSQL Supported
SELECT log(10, 100);  -- Returns 2
SELECT sinh(1.0);     -- Hyperbolic sine

-- GaussDB Error
[20100] ERROR: Unsupported function
```

**Workarounds**
- Use `ln()` for natural logarithm
- Implement hyperbolic functions via UDFs

---

### 3.3 Complex Query Support
**JSON Aggregation**
```sql
-- PostgreSQL Standard
SELECT json_objectagg(key:value);

-- GaussDB Implementation
SELECT json_object_agg(
  CASE WHEN key IS NOT NULL THEN key END, 
  value
);
```

**Key Notes**
- NULL key handling mandatory
- Explicit type casting required
- Advanced options like `WITH UNIQUE KEYS` unavailable

---

## Known Product Defects

### 4.1 Lock Mechanism Anomalies
**Symptom**
```sql
SET LOCK_TIMEOUT = 3000;  -- Syntax error in GaussDB
```

**Workaround**  
Use connection parameter:
```ini
# Connection configuration
lock_wait_timeout = 3s
```

---

### 4.2 Arithmetic Precision Issues
**Integer Division**
```sql
SELECT 4/3;  
-- PostgreSQL returns 1
-- GaussDB returns 1.33333333333333326
```

**Impact**
- Financial calculation errors
- Pagination offset miscalculations

**Solution**  
Explicit integer division:
```sql
SELECT div(4,3);  -- Returns 1
```

---

### 4.3 Metadata Management Differences
**Missing System Columns**
```sql
SELECT oid FROM pg_type;  -- Missing in GaussDB
```

**Workaround**  
Use Huawei's alternative view:
```sql
SELECT * FROM hg_catalog.hg_type;
```

---

## JDBC Driver Issues

### 5.1 Batch Operation Return Values
**Specification Violation**
```java
int[] counts = statement.executeBatch();
// Returns -2 (UNKNOWN_COUNT) instead of actual row count
```

**Impact**
- Batch operation results indeterminate
- ORM framework batch processing fails

---

### 5.2 Type Mapping Anomalies
**Date Type Mapping**
```sql
CREATE TABLE test (date_col DATE);  
-- Actually creates timestamp(0) in GaussDB
```

**Consequences**
- Framework type handling errors
- Unexpected timestamp precision loss

---

### 5.3 Advanced Feature Limitations
**Binary Array Support**
```java
byte[][] data = resultSet.getArray(1).getArray();  
// Throws SQLFeatureNotSupportedException
```

**Workaround**  
Manual BYTEA parsing:
```java
String hexStr = resultSet.getString(1);
byte[] data = Hex.decodeHex(hexStr);
```
