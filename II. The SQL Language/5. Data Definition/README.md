## 5.11. Table Partitioning

表分区

PostgreSQL supports basic table partitioning. This section describes why and how to implement partitioning as part of your database design.

PostgreSQL 支持基本的表分区。本节介绍在数据库设计中实现分区的原因和实践。

### 5.11.1. Overview

概述

Partitioning refers to splitting what is logically one large table into smaller physical pieces. Partitioning can provide several benefits:

分区是指将逻辑上的一个大表分割成更小的物理部分。分区可以提供多种好处：

• Query performance can be improved dramatically in certain situations, particularly when most of the heavily accessed rows of the table are in a single partition or a small number of partitions. Partitioning effectively substitutes for the upper tree levels of indexes, making it more likely that the heavily-used parts of the indexes fit in memory.

在某些情况下，查询性能可以得到显著提高，特别是当表中大多数访问量很大的行位于单个分区或少量分区中时。分区有效地替代了索引的上层树，使得索引中频繁使用的部分更有可能适合内存。

• When queries or updates access a large percentage of a single partition, performance can be improved by using a sequential scan of that partition instead of using an index, which would require random-access reads scattered across the whole table.

当查询或更新访问单个分区的很大一部分时，可以通过对该分区进行顺序扫描而不是使用索引来提高性能，因为索引需要在整个表中分散进行随机访问读取。

• Bulk loads and deletes can be accomplished by adding or removing partitions, if the usage pattern is accounted for in the partitioning design. Dropping an individual partition using DROP TABLE, or doing ALTER TABLE DETACH PARTITION, is far faster than a bulk operation. These commands also entirely avoid the VACUUM overhead caused by a bulk DELETE.

如果在分区设计中考虑了使用模式，则可以通过添加或删除分区来完成批量加载和删除。使用 DROP TABLE 删除单个分区或执行 ALTER TABLE DETACH PARTITION 比批量操作要快得多。这些命令还完全避免了批量 DELETE 造成的 VACUUM 开销。

• Seldom-used data can be migrated to cheaper and slower storage media.

很少使用的数据可以迁移到更便宜且速度较慢的存储介质。

These benefits will normally be worthwhile only when a table would otherwise be very large. The exact point at which a table will benefit from partitioning depends on the application, although a rule of thumb is that the size of the table should exceed the physical memory of the database server.

这些好处通常只有在表非常大的情况下才值得使用。表从分区中获益的确切时间取决于应用程序，尽管经验法则是表的大小应该超过数据库服务器的物理内存。

PostgreSQL offers built-in support for the following forms of partitioning:

PostgreSQL提供了对以下形式的分区的内置支持:

**Range Partitioning**

范围分区

The table is partitioned into “ranges” defined by a key column or set of columns, with no overlap between the ranges of values assigned to different partitions. For example, one might partition by date ranges, or by ranges of identifiers for particular business objects. Each range's bounds are understood as being inclusive at the lower end and exclusive at the upper end. For example, if one partition's range is from 1 to 10, and the next one's range is from 10 to 20, then value 10 belongs to the second partition not the first.

表被划分为由一个键列或一组列定义的「范围」，分配给不同分区的值范围之间没有重叠。例如，可以按日期范围或特定业务对象的标识符范围进行分区。每个范围的界限被理解为**包括下端而不包括上端**。例如，如果一个分区的范围是从 1 到 10，下一个分区的范围是从 10 到 20，则值 10 属于第二个分区而不是第一个分区。

**List Partitioning**

列表分区

The table is partitioned by explicitly listing which key value(s) appear in each partition.

通过显式列出每个分区中出现的键值来对表进行分区。

**Hash Partitioning**

哈希分区

The table is partitioned by specifying a modulus and a remainder for each partition. Each partition will hold the rows for which the hash value of the partition key divided by the specified modulus will produce the specified remainder.

通过为每个分区指定模数和余数来对表进行分区。每个分区将保存分区键的哈希值除以指定模数将产生指定余数的行。

If your application needs to use other forms of partitioning not listed above, alternative methods such as inheritance and UNION ALL views can be used instead. Such methods offer flexibility but do not have some of the performance benefits of built-in declarative partitioning.

如果应用程序需要使用上面未列出的其他形式的分区，则可以使用继承和 UNION ALL 视图等替代方法。此类方法提供了灵活性，但不具备内置声明性分区的一些性能优势。

### 5.11.2. Declarative Partitioning

声明式分区

PostgreSQL allows you to declare that a table is divided into partitions. The table that is divided is referred to as a partitioned table. The declaration includes the partitioning method as described above, plus a list of columns or expressions to be used as the partition key.

PostgreSQL 允许声明表被划分为多个分区。被划分的表称为分区表。该声明包括如上所述的分区方法，以及用作分区键的列或表达式的列表。

The partitioned table itself is a “virtual” table having no storage of its own. Instead, the storage belongs to partitions, which are otherwise-ordinary tables associated with the partitioned table. Each partition stores a subset of the data as defined by its partition bounds. All rows inserted into a partitioned table will be routed to the appropriate one of the partitions based on the values of the partition key column(s). Updating the partition key of a row will cause it to be moved into a different partition if it no longer satisfies the partition bounds of its original partition.

分区表本身是一个「虚拟」表，没有自己的存储空间。相反，存储属于分区，分区是与分区表关联的普通表。每个分区存储由其分区边界定义的数据子集。插入到分区表中的所有行都将根据分区键列的值路由到相应的分区。如果行不再满足其原始分区的分区界限，更新该行的分区键将导致该行被移动到不同的分区中。

Partitions may themselves be defined as partitioned tables, resulting in sub-partitioning. Although all partitions must have the same columns as their partitioned parent, partitions may have their own indexes, constraints and default values, distinct from those of other partitions. See CREATE TABLE for more details on creating partitioned tables and partitions.

分区本身可以定义为分区表，从而产生子分区。尽管所有分区必须具有与其分区父分区相同的列，但分区可能有自己的索引、约束和默认值，与其他分区的索引、约束和默认值不同。有关创建分区表和分区的更多详细信息，请参阅 CREATE TABLE。

It is not possible to turn a regular table into a partitioned table or vice versa. However, it is possible to add an existing regular or partitioned table as a partition of a partitioned table, or remove a partition from a partitioned table turning it into a standalone table; this can simplify and speed up many main- tenance processes. See ALTER TABLE to learn more about the ATTACH PARTITION and DETACH PARTITION sub-commands.

无法将常规表转换为分区表，反之亦然。但是，可以添加现有的常规表或分区表作为分区表的分区，或者从分区表中删除分区，将其变成独立表；这可以简化并加速许多维护过程。请参阅 ALTER TABLE 了解有关 ATTACH PARTITION 和 DETACH PARTITION 子命令的更多信息。

Partitions can also be foreign tables, although considerable care is needed because it is then the user's responsibility that the contents of the foreign table satisfy the partitioning rule. There are some other restrictions as well. See CREATE FOREIGN TABLE for more information.

分区也可以是外部表，尽管需要相当小心，因为外部表的内容满足分区规则是用户的责任。还有一些其他限制。有关详细信息，请参阅创建外部表。

#### 5.11.2.1. Example

案例

Suppose we are constructing a database for a large ice cream company. The company measures peak temperatures every day as well as ice cream sales in each region. Conceptually, we want a table like:

假设我们正在为一家大型冰淇淋公司构建数据库。该公司每天测量峰值气温以及每个地区的冰淇淋销量。从概念上讲，我们想要一个像这样的表。

```sql
CREATE TABLE measurement (
    city_id int not null,
    logdate date not null,
    peaktemp int,
    unitsales int
);
```

We know that most queries will access just the last week's, month's or quarter's data, since the main use of this table will be to prepare online reports for management. To reduce the amount of old data that needs to be stored, we decide to keep only the most recent 3 years worth of data. At the beginning of each month we will remove the oldest month's data. In this situation we can use partitioning to help us meet all of our different requirements for the measurements table.

我们知道大多数查询只会访问上周、一个月或一个季度的数据，因为该表的主要用途是准备在线报告以供管理。为了减少需要存储的旧数据量，我们决定仅保留最近 3 年的数据。在每个月初，我们都会删除最旧月份的数据。在这种情况下，我们可以使用分区来帮助我们满足对测量表的所有不同要求。

To use declarative partitioning in this case, use the following steps:

要在这种情况下使用声明性分区，可使用以下步骤：

1. Create the measurement table as a partitioned table by specifying the PARTITION BY clause, which includes the partitioning method (RANGE in this case) and the list of column(s) to use as the partition key.

通过指定 PARTITION BY 子句将测量表创建为分区表，其中包括分区方法（在本例中为 RANGE）和用作分区键的列列表。

```sql
CREATE TABLE measurement (
    city_id int not null,
    logdate date not null,
    peaktemp int,
    unitsales int
) PARTITION BY RANGE (logdate);
```

2. Create partitions. Each partition's definition must specify bounds that correspond to the partitioning method and partition key of the parent. Note that specifying bounds such that the new partition's values would overlap with those in one or more existing partitions will cause an error.

创建分区。每个分区的定义必须指定与父分区的分区方法和分区键相对应的边界。请注意，指定边界以使新分区的值与一个或多个现有分区中的值重叠将导致错误。

Partitions thus created are in every way normal PostgreSQL tables (or, possibly, foreign tables). It is possible to specify a tablespace and storage parameters for each partition separately.

这样创建的分区在任何方面都是普通的 PostgreSQL 表（或者可能是外部表）。可以为每个分区单独指定表空间和存储参数。

For our example, each partition should hold one month's worth of data, to match the requirement of deleting one month's data at a time. So the commands might look like:

对于我们的示例，每个分区应保存一个月的数据，以满足一次删除一个月数据的要求。所以命令可能看起来像：

```sql
CREATE TABLE measurement_y2006m02 PARTITION OF measurement
    FOR VALUES FROM ('2006-02-01') TO ('2006-03-01');

CREATE TABLE measurement_y2006m03 PARTITION OF measurement
    FOR VALUES FROM ('2006-03-01') TO ('2006-04-01');

...

CREATE TABLE measurement_y2007m11 PARTITION OF measurement
    FOR VALUES FROM ('2007-11-01') TO ('2007-12-01');

CREATE TABLE measurement_y2007m12 PARTITION OF measurement
    FOR VALUES FROM ('2007-12-01') TO ('2008-01-01')
    TABLESPACE fasttablespace;

CREATE TABLE measurement_y2008m01 PARTITION OF measurement
    FOR VALUES FROM ('2008-01-01') TO ('2008-02-01')
    WITH (parallel_workers = 4)
    TABLESPACE fasttablespace;
```

(Recall that adjacent partitions can share a bound value, since range upper bounds are treated as exclusive bounds.)

回想一下，相邻分区可以共享一个边界值，因为范围上限被视为独占边界。

If you wish to implement sub-partitioning, again specify the PARTITION BY clause in the commands used to create individual partitions, for example:

如果希望实现子分区，请再次在用于创建单独分区的命令中指定 PARTITION BY 子句，例如：

```sql
CREATE TABLE measurement_y2006m02 PARTITION OF measurement
    FOR VALUES FROM ('2006-02-01') TO ('2006-03-01')
    PARTITION BY RANGE (peaktemp);
```

After creating partitions of `measurement_y2006m02`, any data inserted into `measurement` that is mapped to `measurement_y2006m02` (or data that is directly inserted into `measure-ment_y2006m02`, which is allowed provided its partition constraint is satisfied) will be further redirected to one of its partitions based on the `peaktemp` column. The partition key specified may overlap with the parent's partition key, although care should be taken when specifying the bounds of a sub-partition such that the set of data it accepts constitutes a subset of what the partition's own bounds allow; the system does not try to check whether that's really the case.

创建 measurement_y2006m02 分区后，插入到 measurement 中映射到 measurement_y2006m02 的任何数据（或直接插入到 measurement_y2006m02 的数据，只要满足其分区约束就允许）将被进一步重定向到基于 peaktemp 列的分区。指定的分区键可能与父分区的分区键重叠，但在指定子分区的边界时应小心，使其接受的数据集构成分区自身边界允许的子集；系统不会尝试检查情况是否确实如此。

Inserting data into the parent table that does not map to one of the existing partitions will cause an error; an appropriate partition must be added manually.

在父表中插入没有映射到现有分区的数据将导致错误；必须手动添加适当的分区。

It is not necessary to manually create table constraints describing the partition boundary conditions for partitions. Such constraints will be created automatically.

无需手动创建描述分区的分区边界条件的表约束。此类约束将自动创建。

3. Create an index on the key column(s), as well as any other indexes you might want, on the partitioned table. (The key index is not strictly necessary, but in most scenarios it is helpful.) This automatically creates a matching index on each partition, and any partitions you create or attach later will also have such an index. An index or unique constraint declared on a partitioned table is “virtual” in the same way that the partitioned table is: the actual data is in child indexes on the individual partition tables.

在分区表的键列上创建索引，以及可能需要的任何其他索引。（键索引并不是严格必要的，但在大多数情况下它很有帮助）这会自动在每个分区上创建一个匹配的索引，并且稍后创建或附加的任何分区也将具有这样的索引。在分区表上声明的索引或唯一约束是「虚拟的」，就像分区表一样：实际数据位于各个分区表的子索引中。

```sql
CREATE INDEX ON measurement (logdate);
```

4. Ensure that the enable_partition_pruning configuration parameter is not disabled in `postgresql.conf`. If it is, queries will not be optimized as desired. In the above example we would be creating a new partition each month, so it might be wise to write a script that generates the required DDL automatically.

确保 `postgresql.conf` 中的 `enable_partition_pruning` 配置参数没有被禁用。如果禁用，查询将不会根据需要进行优化。在上面的示例中，我们每个月都会创建一个新分区，因此编写一个自动生成所需 DDL 的脚本可能是明智之举。
