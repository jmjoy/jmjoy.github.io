+++
date = "2017-01-10T00:35:17+08:00"
draft = false
title = "Sql语句：分组最后一条"
categories = ["mysql"]
+++

https://stackoverflow.com/questions/1313120/retrieving-the-last-record-in-each-group/1313293#1313293

别笑，这个问题困惑了我很久，我曾经用过两个子查询的语句去做，导致性能挺差的，也试过用一张中间表去保存最后的
ID，结果因为莫名其妙的ID没有同步好，导致了一个BUG，退款了给别人，心塞啊。今天试着google了一下，原来这么容易就可以解决这个问题了，果然是所有的子查询都可以写成JOIN的形式。

场景 ： 比如一个日志表，有个订单号的字段，现在要取各订单号的最后一条日志

```sql
SELECT m1.*
FROM messages m1 LEFT JOIN messages m2
 ON (m1.name = m2.name AND m1.id < m2.id)
WHERE m2.id IS NULL;
```

感谢解答了这个问题的大佬！

