---
layout: post
title:  数据库基本存储原理
date:   2017-03-04 20:35:48 +0800
categories: technology
tags: [Database]
---
# 一、基本存储单元——页

数据库文件存储是以页为存储单元的，一个页是8K（8192Byte），一个页就可以存放N行数据。我们表里的数据都是存放在页上的，这种叫数据页。还有一种页存放索引数据的，叫索引页。

同时，页也是IO读取的最小单元（物理IO上不是按行读取），也是所有权的最小单位。如果一页中包含了表A的一行数据，这页就只能存储表A的行数据了。或是一页中包含了索引B的条目，那这页也仅仅只能存储索引B的条目了。每页中除去存储数据之外，还存储一些页头信息以及行偏移以便SQL Server知道具体每一行在页中的存储位置。

数据库的基本物理存储单元是页，一个表由很多个页组成，那这些页又是如何组织的呢？我们一般都会对表创建索引，这些索引又是如何存储的呢？

# 二、表/索引的存储结构

是一个B树（二叉搜索树）的示例，都是小的元素放左边，大的元素放右边，依次构造的，比如要查找元素9，从根节点开始，只要比较三次就找到他了，查询效率是非常高的。

B+树和B-树都是B树的变种，或者说是更加高级的复杂版本。B+树和B-树是数据库中广泛应用的索引存储结构，它可以极大的提高数据查找的效率。前面说了数据库存储的基本单元是页，因此，索引树上的节点就是页了。

为了保证数据的查询效率，当新增、修改、删除数据的时候，都需要维护这颗索引树，就可能会出现分裂、合并节点（页）的情况（这是树的结构所决定的，想要更好理解这一点，可以尝试自己代码实现一下B-树B+树）。好！重点来了！

## 索引的缺点：

- 当新增、修改、删除数据的时候，需要维护索引树，有一定的性能影响；
- 同上面，在频繁的树维护过程中，B树的页拆分、合并会造成大量的索引碎片，又会极大的印象查询效率 ，因此索引还需要维护；
- 非聚集索引需要额外的存储空间，不过这个一般问题都不是很大，但是需要注意的一个问题

### 聚集索引

聚集索引决定了表数据的物理存储顺序，也就是说表的物理存储是根据聚集索引结构进行顺序存储的，因此一个表只能有一个聚集索引。

- 所有数据都在叶子节点的页上，在叶子节点（数据页）之间有一个链指针，这是B+树的特点；
- 非叶子节点都是索引页，存储的就是聚集索引字段的值；
- 表的物理存储就是依据聚集索引的结构的，一个表只能有一个聚集索引；

聚集索引的所有的数据都存储在叶子节点上，数据查询的复杂度都是一样的（树的深度），按照聚集索引列查找数据效率是非常高的。上面说了，聚集索引决定了表的物理存储结构，那如果没有创建聚集索引，会如何呢？——表内的所有页都无序存放，是一个无序的堆结构。堆数据的查询就会造成表扫描，性能是非常低的。

**因此聚集索引的的重要性不言而喻，一般来说，大多会对主键建立聚集索引，大多数普通情况这么做也可以。但实际应用应该尊从一个原则就是“频繁使用的、排序的字段上创建聚集索引”**

### 非聚集索引

除了聚集索引以外的其他索引，都称之为非聚集索引，非聚集索引一般都是为了优化特定的查询效率而创建的。非聚集索引也是B树（B+树和B-树）的结构，与聚集索引的存储结构唯一不一样的，就是非聚集索引中不存储真正的数据行，因为在聚集索引中已经存放了所有数据，非聚集索引只包含一个指向数据行的指针即可。

- 非聚集索引的创建会单独创建索引文件来存储索引结构，会占用一定存储空间，就是用空间换时间；
- 非聚集索引的目的很单纯：提高特定条件的查询效率，一个表有可能根据多种查询需求创建多个非聚集索引；

数据查询SQL简单来看，分为两个部分：`SELECT ***` 和 `Where ***`，因此索引的创建也是根据这两部分来决定的。根据这两点，有两种主要的索引形式：复合索引和覆盖索引，在实际使用中，根据具体情况可能都会用到，只要能提高查询效率就是好索引。

1.覆盖索引：就是在索引中包含的数据列（非索引列，SELECT需要的列），这样在使用该索引查询数据时就不会再进行键查找（也叫书签查找）了。

2.复合索引：主要针对Where中有多个条件的情况，索引包含多个数据列。在使用复合索引时，应注意多个索引键的顺序问题，这个是会影响查询效率的，一般的原则是唯一性高的放前面，还有就是SQl语句中Where条件的顺序应该和索引顺序一致。

## 索引碎片

前面说过了，索引在使用一段时间后（主要是新增、修改、删除数据，如果该页已经存储满了，就要进行页的拆分，频繁的拆分，会产生较多的索引碎片）会产生索引碎片，这就造成了索引页在磁盘上存储的不连续。会造成磁盘的访问使用的是随机的i/o，而不是顺序的i/o读取，这样访问索引页会变得更慢。如果碎片过多，数据库是可会能不使用该索引的（她嫌弃你太慢了，数据库会选择一个更优的执行计划）。

解决这个问题主要是两种方法：

**第一种是预防**：设置页的填充因子

意思就是在页上设置一段空白区域，在新增数据的时候，可以使用这段空白区域，可以一定的避免页的拆分，从而减少索引碎片的产生。

填充因子就是用来描述这种页中填充数据的一个比例，一般默认是100%填充的。如果我们修改填充因子为80%，那么页在存储数据时，就会剩余20%的剩余空间，这样在下次插入的时候就不会拆分页了。 那么是不是我们可以把填充因子设置低一点，留更多的剩余空间，不是很好嘛？当然也不好，填充因子设置的低，会需要分配更多的存储空间，叶子节点的深度会增加，这样是会影响查询效率的，因此，这是要根据实际情况而定的。

那么一般我们是怎么设置填充因子的呢，主要根据表的读写比例而定的。如果读的多，填充因子可以设置高一点，如100%，读写各一半，可以80~90%；修改多可以设置50~70%。

**第二种是索引修复**：定期对索引进行检查、维护，写一段SQL检查索引的碎片比例，如果碎片过多，进行碎片修复或重建，定期执行即可。

## 索引使用总结

- 创建索引的的字段尽量小，最好是数值，比如整形int等；
- 对于频繁修改的字段，尽量不要创建索引，维护索引的成本很高，而且更容易产生索引碎片；
- 定期的索引维护，如索引碎片的修复等；
- 不要建立或维护不必要的重复索引，会增加修改数据（新增、修改、删除数据）的成本；
- 使用唯一性高的字段创建索引，切不可在性别这样的低唯一性的字段上创建索引；
- 在SQL语句中，尽量不要在Where条件中使用函数、运算符或表达式计算，会造成索引无法正常使用；
- 应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描；
- 应尽量避免在 where 子句中使用!=或<>操作符，否则将导致引擎放弃使用索引而进行全表扫描；