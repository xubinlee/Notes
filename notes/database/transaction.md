# 事务

## 事务的四大特性（ACID）

1. 原子性（Atomicity）：事务包含的所有操作要么全部成功，要么全部回滚；

2. 一致性（Consistency）：一个事务执行之前和执行之后的状态必须一致（比如转账金额）；

3. 隔离性（Isolation）：多个并发事务直接要相互隔离，不能被其他事务的操作所干扰；

4. 持久性（Durability）：事务一旦提交，数据库中数据的改变是永久性的。 

## 事务并发问题

1. 脏读：事务A读到事务B未提交的数据；假如B回滚，A读到的数据是无效的；

2. 不可重复读：事务A两次查询一行记录结果不一样，中途事务B修改了这行记录；

3. 幻读：事务A同一查询条件两次查询返回结果集记录数不相同，中途事务B提交修改；

## 事务的隔离级别

1. Read uncommitted（读未提交）：最低级别，任何情况都无法保证；

2. Read committed（读已提交）：可避免脏读；

3. Repeatable read（可重复读）：避免脏读、不可重复读；

4. Serializable（串行化）：可避免脏读、不可重复读、幻读。

**总结：**

1. 数据库默认隔离级别：mysql：repeatable，oracle、sql server：read committed；

2. Mysql binlog的三种格式：statement、row、mixed；

3. 为什么mysql默认用repeatable而不是read committed？

   在mysql 5.0之前只有statement一种格式，而主从复杂存在了大量的不一致，故选用repeatable。

4. 为什么默认级别都会选用read committed？

   1)         repeatable存在间隙锁会使死锁的概率增大；

   2)         在repeatable级别下，条件列为命中索引会锁表，而在read committed下只锁行。

5. 在read committed级别下，主从复制用什么binlog格式：row格式，是基于行的复杂

**备注：**

1. 在read committed级别下，不可重复读怎么解决？

不用解决，这个问题是可以接受的，毕竟数据都已经提交，读出来本身就没有太大问题。

2. 互联网项目请用read committed（读已提交）这个隔离级别。

## 事务的传播行为

1. PROPAGATION_REQUIRED：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设置。

2. PROPAGATION_SUPPORTS：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。

3. PROPAGATION_MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。

4. PROPAGATION_REQUIRES_NEW：创建新事务，无论当前存不存在事务，都创建新事务。

5. PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。

6. PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。

7. PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。

## Spring事务的使用及注意事项

1. 不要在接口上声明@Transactional，而要在具体类的方法上使用 @Transactional 注解，否则注解可能无效。

2. 不要图省事，将@Transactional放置在类级的声明中，放在类声明，会使得所有方法都有事务。故@Transactional应该放在方法级别，不需要使用事务的方法，就不要放置事务，比如查询方法。否则对性能是有影响的。

3. @Transactional 的事务开启 ，或者是基于接口的 或者是基于类的代理被创建。所以在同一个类中一个方法调用另一个方法有事务的方法，事务是不会起作用的 。（AopContext.currentProxy()）

4. 用REQUIRES_NEW的时候，发现没有起作用，原因是A方法（有事务）调用B方法（要独立新事务），如果两个方法写在同一个类里，spring的事务会只处理能同一个。

5. 使用了@Transactional的方法，只能是public，@Transactional注解的方法都是被外部其他类调用才有效，故只能是public。故在 protected、private 或者 package-visible 的方法上使用 @Transactional 注解，它也不会报错，但事务无效。

6. 由于使用了多数据源，所以在使用事务时需要指定事务的value，tradeTransactionManager就是相应的value值。

7. 默认只有RuntimeExcetion会触发回滚，经验证明确实如此，需设置rollbackFor = {Exception.class}，当然具体业务具体处理，可能有的业务抛出的某些异常并不需要触发回滚，所以此时应该细化处理异常。

8. 多数据源事务场景：

   1)         不同类不同数据源，B发生异常，同时回滚；

   2)         不同类不同数据源，A发生异常，B不回滚；

   3)         同类不同数据源，A发生异常，B不回滚；

   4)         同类不同数据源，B发生异常，B不回滚；

   5)         同类同数据源，发生异常，同时回滚；

   6)         不同类同数据源，发生异常，同时回滚；

   总结：

   Ø  主方法在任何情况下事务都会回滚；

   Ø  两个方法只要同数据源，均能正常回滚；

   Ø  如果不同数据源，调用时，应将自身出现异常情况排查后再调用子方法。

9. MySQL数据库表引擎应为InnoDB，否则不支持事务。

## 分布式事务

1. @RedisTransactional使用悲观锁实现，此注解只能使用在方法上，在方法执行前会被上锁，执行完成会解锁。（AOP的环绕通知实现）

   ![](https://github.com/xubinlee/Notes/blob/master/assets/redis-transactional.png?raw=true)

2. 如果多个方法对同一分布式对象可能有并发事务操作时，则应该使用同一个锁名；

3. 如果只有一个方法操作分布式对象，则可以使用默认的锁名。

