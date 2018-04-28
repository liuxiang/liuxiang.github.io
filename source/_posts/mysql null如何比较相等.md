title: mysql null如何比较相等
date: 2017-12-21 00:00:00
categories: mysql
tags: [mysql]

---
# 表关联中null值无法用"="号匹配.
```
drop table if exists tab;create table tab as select * from (
    select 1 id,'刘一' name,'男' sex,18 age,SYSDATE() time
    union select 2,'陈二','女',28,date_sub(SYSDATE(),interval 1 day)
    union select 3,'张三','女',28,date_sub(SYSDATE(),interval 2 day)
    union select 4,'李四','女',30,date_sub(SYSDATE(),interval 3 day)
    union select 5,'王五','女',30,date_sub(SYSDATE(),interval 4 day)
    union select 6,'赵六','女',30,date_sub(SYSDATE(),interval 5 day)
    union select 7,null,'女',30,date_sub(SYSDATE(),interval 6 day)
) temp;

select * from tab;
```

## 无法用"="号匹配.
```
-- 无法用"="号匹配.
select * from tab a,tab b where a.name = b.name;-- 不支持null值匹配
select * from tab a,tab b where a.name <=> b.name;-- 支持null值匹配
```
```
-- 证明
select null = null,null is null,null <=> null;-- <null> 1 1
select STRCMP('a','a'),STRCMP('a','b'),STRCMP('b','a'),STRCMP(null,null);-- 0 -1 1 <null>
```

---
**参考**
`如何利用MySQL数据库判断NULL结果为1`
https://jingyan.baidu.com/article/f71d6037a6ae2a1ab641d113.html