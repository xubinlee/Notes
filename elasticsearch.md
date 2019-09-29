# Elasticsearch原理

l   **基本概念**：Elasticsearch是面向文档型数据库，一条数据就是一个文档，用JSON作为文档序列化的格式。

Elasticsearch和关系型数据库术语对照表：

关系数据库       ⇒ 数据库       ⇒ 表         ⇒ 行             ⇒ 列(Columns) 

Elasticsearch ⇒ 索引(Index) ⇒ 类型(type) ⇒ 文档(Docments) ⇒ 字段(Fields)

l   **索引**

Elasticsearch索引的精髓：一切设计都是为了提高搜索的性能。为此牺牲插入/更新等方面的性能

※  **倒排索引**：

![](https://github.com/xubinlee/Notes/blob/master/assets/inverted-index.png?raw=true)                                                  

![](https://github.com/xubinlee/Notes/blob/master/assets/table.png?raw=true)       ![](https://github.com/xubinlee/Notes/blob/master/assets/posting-list.png?raw=true)       

**Term**：Kate，John，24，Female这些叫term，存储在磁盘里；

**Posting List**：Elasticsearch为每个field建立一个倒排索引，Posting list就是一个int数组，存储符合某个team的文档id；

**Term Dictionary**：为快速找到term，将所有term排序，二分法查询，查找效率logN。Term Dirctonary会很大，整个放内存不现实，于是就有了Term index；

**Term Index**：Term index存下term的一些前缀与Term Dictionary的block之间的映射关系，再结合FST（特定是非常节省内存）压缩技术，可以使Term index以树的形式缓存到内存中。找到对应的block位置之后，再到磁盘上找term，大大减少磁盘随机读的次数。

**压缩技巧：**

除了用FST压缩Term index外，对Posting list也有压缩技巧:

![](https://github.com/xubinlee/Notes/blob/master/assets/frame-of-reference.png?raw=true)   Frame Of Reference

![](https://github.com/xubinlee/Notes/blob/master/assets/roaring-bitmaps.png?raw=true)   Roaring bitmaps

l   **联合索引**

假设要查询arg=18，gender=女组合条件：

过程是先从term index找到18的term dictionary的大概位置，然后再从term dictionary里精确地找到18这个term，然后得到posting list或者一个指向posting list位置的指针。然后再查询gender=女的过程也是类似的。最后得出age=18 AND gender=女就是把两个posting list做一个“与”的合并。

联合使用两个索引有两种办法：

▷  使用skip list（跳表）数据结构，同时遍历gender和age的posting list，互相skip；

Skip list合并过程使用Frame Of Reference压缩，非常高效。

▷  使用bitset（位集）数据结构，对gender和age两个filter分别求出bitset，对两个bitset做AND操作。

Bitset合并过程使用Roaring bitmaps压缩。

**注**：对于简单的相等条件的过滤缓存成纯内存的bitset还不如需要访问磁盘的skip的方式要快。

l   **减少文档数**

使用嵌套文档，对于term的posting list只需要保存父文档的doc id就可以了。如果可以在一个父文档里塞入50个嵌套文档，那么posting list就可以变成之前的1/50。