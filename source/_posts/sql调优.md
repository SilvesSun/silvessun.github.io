---
title: sql调优
date: 2018-07-03 14:31:06
tags:
- SQL优化
categories:
- 数据库
---

## 在update的过程中, 通过sum子表更新head表某个值

最近在开发的工程中, 遇到一个问题. 最开始建表时未将需要在主表展示的部分数据从从表更新到主表, 然后现在因为业务需要将从表的qty求和后根据对应的外键更新到主表.

两张表分别为 order_header, order_detail. 其中相关的列为 order_header(id, total_qty), order_detail(qty, header_id). order_header表有17406条记录, detail表有 511732 条记录

第一映像感觉很简单. 只要根据外键做聚合就好, 所以写出下面语句

```sql
update kohls_order_header h set total_qty= (select sum(d.po_qty) from kohls_order_detail d where d.header_id=h.id)
```

然后发现执行速度特别慢, 本地测试超过30分钟. 说明语句存在问题. 执行explain查看结果

![2018-07-03-15-03-33](http://p3euxxfa8.bkt.clouddn.com/2018-07-03-15-03-33.png)

上面这条语句相当于针对每个head id, 都需要从子表中查找到对应的detail后然后sum, 耗时较多.

优化思路:

首先将detail中需要的数据先统计出来, 然后和head表关联. 优化后语句为:

```sql
UPDATE kohls_order_header h
set total_qty=detail_res.sum_qty 
from  (SELECT sum(d.po_qty) as sum_qty, d.header_id FROM kohls_order_detail d GROUP BY d.header_id) as detail_res
WHERE h.id=detail_res.header_id
```

执行耗时为1.7秒.

执行计划为:

![2018-07-03-15-11-49](http://p3euxxfa8.bkt.clouddn.com/2018-07-03-15-11-49.png)

可以说优化的不错了.