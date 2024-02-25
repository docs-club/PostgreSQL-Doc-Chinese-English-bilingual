# Part II. The SQL Language

This part describes the use of the SQL language in PostgreSQL. We start with describing the general syntax of
SQL, then explain how to create the structures to hold data, how to populate the database, and how to query it. The middle part lists the available data types and functions for use in SQL commands. The rest treats several aspects that are important for tuning a database for optimal performance.

本部分介绍 PostgreSQL 中 SQL 语言的使用。我们首先描述 SQL 的一般语法，然后解释如何创建保存数据的结构、如何填充数据库以及如何查询数据库。中间部分列出了 SQL 命令中可用的数据类型和函数。其余部分讨论对于调整数据库以获得最佳性能很重要的几个方面。

The information in this part is arranged so that a novice user can follow it start to end to gain a full understanding of the topics without having to refer forward too many times. The chapters are intended to be self-contained, so that advanced users can read the chapters individually as they choose. The information in this part is presented in a narrative fashion in topical units. Readers looking for a complete description of a particular command should see Part VI.

这部分中的信息经过精心安排，以便新手用户可以从头到尾地遵循它，从而充分理解主题，而无需向前参考太多次。这些章节是独立的，以便高级用户可以根据自己的选择单独阅读这些章节。这部分的信息以主题单元的叙述方式呈现。寻找特定命令的完整描述的读者应该参见第六部分。

Readers of this part should know how to connect to a PostgreSQL database and issue SQL commands. Readers
that are unfamiliar with these issues are encouraged to read Part I first. SQL commands are typically entered using the PostgreSQL interactive terminal psql, but other programs that have similar functionality can be used as well.

这部分的读者应该知道如何连接到 PostgreSQL 数据库并发出 SQL 命令。鼓励不熟悉这些问题的读者首先阅读第一部分。 SQL 命令通常使用 PostgreSQL 交互式终端 psql 输入，但也可以使用具有类似功能的其他程序。

## Table of Contents

- [5. Data Definition](II.%20The%20SQL%20Language/5.%20Data%20Definition)
  - [5.11. Table Partitioning](II.%20The%20SQL%20Language/5.%20Data%20Definition#511-table-partitioning)
