# 阿里巴巴Java开发手册编程规约摘录

## 命名风格

1. POJO类中布尔类型的变量，都不要加is，否则部分框架解析会引起序列化错误。如Boolean isDeleted；属性的方法也是iseleted()，RPC框架在反向解析的时候，以为对应的属性名称是deleted()；

2. 【参考】各层命名规约：

   A) Service/DAO层方法命名规约 

   1） 获取单个对象的方法用get做前缀。 

   2） 获取多个对象的方法用list做前缀。 

   3） 获取统计值的方法用count做前缀。 

   4） 插入的方法用save/insert做前缀。 

   5） 删除的方法用remove/delete做前缀。 

   6） 修改的方法用update做前缀。 

   B) 领域模型命名规约 

   1） 数据对象：xxxDO，xxx即为数据表名。 

   2） 数据传输对象：xxxDTO，xxx为业务领域相关的名称。 

   3） 展示对象：xxxVO，xxx一般为网页名称。 

   4） POJO是DO/DTO/BO/VO的统称，禁止命名成xxxPOJO。

## 代码格式

运算符与下文一起换行；方法调用的点符号与下文一起换行；方法调用时，多个参数在逗号后换行。

```java
method(args1, args2, args3, ..., 
       argsX){
StringBuffer sb = new StringBuffer();
// 超过120个字符的情况下，换行缩进4个空格，点号和方法名称一起换行
sb.append("zi").append("xin")...
    .append("huang");
}
```



## OPP规约

1. Object的equals方法容易抛空指针异常，应使用常量或确定有值的对象来调用equals：”test”.equals(object)

2. 所有相同类型的包装类对象之间值的比较，全部使用equals方法比较。

3. 循环体内，字符串的连接方式，使用StringBuilder的append方法进行扩展，不要使用+连接

4. 不要在foreach循环里进行元素的remove/add操作。remove元素请使用Iterator方式，如果并发操作，需要对Iterator对象加锁。

5. 集合初始化时，指定集合初始化大小：

   HashMap（int ininialCapacity）初始化，ininialCapacity=（需要存储的元素个数/负载因子）+1。

   负载因子（loader factor）默认为0.75，如果暂时不确定初始值大小，设置为16（默认值）

## 并发处理

1. 线程资源必须通过线程池提供，不允许在应用中自行显式创建线程。

2. 如果是JDK8，使用Instant代替Date，LocalDateTime代替Calendar，DateTimeFormatter代替SimpleDateFormat

3. 并发修改同一记录时，避免更新丢失，需要加锁。要么在应用层加锁，要么在缓存加锁，要么在数据库层使用乐观锁，使用version作为更新依据，乐观锁的重试次数不得小于3次。

## 索引规约

1. 业务上具有唯一特性的字段，即使是多个字段的组合，也必须建成唯一索引。

2. 超过三个表禁止join。需要join的字段，数据类型必须绝对一致；多表关联查询时，保证被关联的字段需要有索引。

3. 页面搜索严禁左模糊或者全模糊，如果需要请走搜索引擎来解决。

4. 如果有order by的场景，请注意利用索引的有序性。order by最后的字段是组合索引的一部分，并且放在索引组合顺序的最后，避免出现file_sort的情况，影响查询性能。

5. 利用延迟关联或者子查询优化超过分页场景。

   说明：Mysql并不是跳过offset行，而是取offset+N行，然后返回放弃前offset行，返回N行。先快速定位需要获取的id段，然后再关联：

   ```mysql
   SELECT a.* FROM 表1 a,(select id from 表1 where 条件 LIMIT 100000,20) b where a.id=b.id
   ```

6. 建组合索引的时候，区分度最高的在最左边。

## SQL语句

1. count(*)会统计值为NULL的行，而count(列名)不会。

2. 当某一列的值全为NULL时，count(col)的返回结果为0，但sum(col)的返回结果为NULL，因此使用sum()时需注意NPE问题。避免方法：

   ```mysql
   SELECT IF(ISNULL(SUM(g)),0,SUM(g)) FROM table;
   ```

3. 使用ISNULL()来判断是否为NULL值。NULL与任何值的直接比较都为NULL：NULL<>NULL,NULL=NULL,NULL<>1均返回NULL

4. 不得使用外键级联，一切外键概念必须在应用层解决。外键与级联更新适用于单机低并发，不适合分布式、高并发集群；级联更新是强阻塞，存在数据库更新风暴的风险；外键影响数据库的插入速度。

5. 禁止使用存储过程，存储过程难以调试和扩展，更没有移植性。

6. 数据删除和修改时，要先select，避免出现误删除，确认无误才能执行更新语句。

## ORM映射

1. 在表查询中，一律不要使用*作为查询的字段列表，需要哪些字段必须明确写。

   说明：1）增加查询分析器解析成本。2）增减字段容易与resultMap配置不一致。

2. POJO类的布尔属性不能加is，而数据库字段必须加is_。

3. sql.xml配置参数使用：#{}，#param# 不要使用${}，此种方式容易出现SQL注入。