# GaussDB和OpenGauss功能和语法差异

1. GaussDB与OpenGauss存在差异
2. GaussDB不存在的功能
3. OpenGauss不存在的功能


## GaussDB与OpenGauss存在差异

### `escape ''`语法行为差异

默认情况下， 语句 `select * from BasicEntityTest where data like '%\%' escape '';` 结果不同。

* OpenGauss
  无论什么情况，查询结果都是空。
 
* GaussDB
  当data字段包含`\`字符的时候，查询结果不为空。 

* 参考信息：https://gitcode.com/opengauss/openGauss-server/issues/7263

## GaussDB不存在的功能

## OpenGauss不存在的功能

