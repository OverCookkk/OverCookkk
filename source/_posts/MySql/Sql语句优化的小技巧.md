---
title: Sql语句优化的小技巧
tags: [MySql]      #添加的标签
categories: MySql
description: 
#cover: 
---

## 避免使用select*



## 用union all代替union



## 小表驱动达标



## 批量操作



## 多用limit

有时候，我们需要查询某些数据中的第一条，比如：查询某个用户下的第一个订单，想看看他第一次的首单时间。

```mysql
select id, create_date 
from order 
where user_id=123 
order by create_date asc 
limit 1;
```

使用`limit 1`，只返回该用户下单时间最小的那一条数据即可。