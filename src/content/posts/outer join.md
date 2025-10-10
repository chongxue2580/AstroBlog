---
title: outer join
published: 2025-10-11
tags: [mysql]
category: mysql
description: 三种join连接
draft: false
---



### jion 连接分为三种，内连接，外链接，交叉连接

#### 1:inner jion
默认，所以可以省略inner关键字
#### 2:outer join
1. left outer join,左外连接，结果表中出了匹配行外，还包括左表有而右表中不匹配的行，对于这样的行，右表选择列置为null
2. right outer join,右外连接，结果表中除了匹配行外，还包括右表有而左表中不匹配的行，对于这样的行，左表选择列置为null
3. 自然连接，分为natural left out join和natural right outer join，语义定义与inner join相同
#### 3.cross join,交叉连接，实际上就是将两个表进行笛卡尔积运算，结果表的行数等于两行表的列数

#### 总结
1. A inner join B : 取交集
2. A left join B：取A全部，B没有对应的值，则为null
3. A rigth join B：取并集，彼此没有对应的值为null
上述4种对应的条件，在on后填写



