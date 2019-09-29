# 索引

## 索引的本质

1. 需要高效获取数据：数据库查询是数据库的主要功能之一，我们都希望查询数据的速度能尽可能的快。顺序查找的时间复杂度为O（n），而二叉查找树的时间复杂度是O（logN），查找速度最快和比较次数最少。

2. 通过索引指向数据：每种查找算法都只能应用于特定的数据结构上，例如二分查找要求被检索数据有序，而二叉树查找只能应用于二叉查找树上，但是数据本身的组织结构不可能完全满足各种数据结构，所以，在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用（指向）数据，这种数据结构，就是索引。

3. 减少磁盘I/O损耗：索引本身也很大，不可能全部存储在内存中，所以索引查找过程中需要产生磁盘I/O消耗，磁盘I/O消耗成本是访问内存的十万倍左右（昂贵磁盘I/O操作的优化：预读【局部性原理】）。二叉查找树可以任意的构造，最坏的情况下磁盘I/O的次数由树的高度决定，所以减少磁盘I/O的次数就必须压缩树的高度，B-/+Tree就诞生了。B-/+Tree单节点可以存储更多相邻的元素，根据局部性原理已经将相邻数据预读到内存中，使得查询磁盘I/O次数更少。

## 索引优化

1. 选择索引列：可以考虑在where子句或者join子句中出现的列建索引；

2. 最左前缀原则：是B+树的数据结构决定的，因为B+树是按照从左到右的顺序来建立搜索树的。在建立联合索引时，尽量将经常参与查询的字段放在联合索引的最左边；

3. like的使用：一般情况下不建议使用like操作，如果非用不可，需要注意：like’%abc%’不会使用索引，而like’abc%’可以使用索引。这是最左前缀原则的一个使用场景；

4. 不能使用索引说明：mysql会按照联合索引从左到右进行匹配，直到遇到范围查询，如：>,<,between,like等就停止匹配。a = 1 and b =2 and  c > 3 and d = 4，如果建立（a,b,c,d）顺序的索引，d是不会使用索引的。但如果联合索引是（a,b,d,c）的话，则a b d c都可以使用到索引，只是最终c是一个范围值。

5. ORDER BY子句，尽量使用index方式排序：mysql支持两种排序：FileSorth和Index，后者效率高。Mysql的一次查询只会从众多索引中选择一个索引，order by满足以下情况，会使用index方式索引：

   a)          order by语句满足索引最左前缀原则；

   b)         使用where子句与order by子句条件列组合满足索引最左前缀原则；

注意mysql的max_length_for_sort_data默认为1024，如果排序的数据量大于1024，则order by不会使用索引，而是使用using filesort。