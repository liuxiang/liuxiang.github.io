title: mysql sql json解析支持
date: 2017-10-17 00:00:00
tags: [mysql json]

---
[TOC]

---
# MySQL 5.7.9原生支持
示例
```
mysql> SELECT c, JSON_EXTRACT(c, "$.id"), g
     > FROM jemp
     > WHERE JSON_EXTRACT(c, "$.id") > 1
     > ORDER BY JSON_EXTRACT(c, "$.name");
+-------------------------------+-----------+------+
| c | c->"$.id" | g |
+-------------------------------+-----------+------+
| {"id": "3", "name": "Barney"} | "3" | 3 |
| {"id": "4", "name": "Betty"} | "4" | 4 |
| {"id": "2", "name": "Wilma"} | "2" | 2 |
+-------------------------------+-----------+------+
3 rows in set (0.00 sec)

mysql> SELECT c, c->"$.id", g
     > FROM jemp
     > WHERE c->"$.id" > 1
     > ORDER BY c->"$.name";
+-------------------------------+-----------+------+
| c | c->"$.id" | g |
+-------------------------------+-----------+------+
| {"id": "3", "name": "Barney"} | "3" | 3 |
| {"id": "4", "name": "Betty"} | "4" | 4 |
| {"id": "2", "name": "Wilma"} | "2" | 2 |
+-------------------------------+-----------+------+
3 rows in set (0.00 sec)
```

`MySQL 在SQL中解析JSON字段`
https://my.oschina.net/Rayn/blog/599357

`MySQL 5.7 使用原生JSON类型的例子`
http://www.jianshu.com/p/455d3d4922e1

- doc
https://dev.mysql.com/doc/refman/5.7/en/json-search-functions.html#operator_json-column-path
https://dev.mysql.com/doc/refman/5.7/en/json.html

---
# 自定义json解析函数
`mysql 常用自定义函数解析 - George_sz - 博客园`
http://www.cnblogs.com/qiaoyihang/p/6250684.html

---
# sql中字段数据转JSON
```
To obtain nested objects, use this function as an argument for json_object
select json_object(
                f.last_update
            , json_members(
                    'film'
                , json_object(
                        f.film_id
                    , f.title
                    , f.last_update
                    )
                , 'category'
                , json_object(
                        c.category_id
                    , c.name
                    , c.last_update
                    )
                )
            ) as film_category
from film_category fc 
inner join film f
on fc.film_id = f.film_id
inner join category c
on fc.category_id = c.category_id
where f.film_id =1;

yields a string representing the following JSON object (indentation added for readability):
{
    last_update:"2006-02-15 05:03:42"
, film:{
        "film_id":1
    , "title":"ACADEMY DINOSAUR"
    , "last_update":"2006-02-15 05:03:42"
    }
, category:{
        "category_id":6
    , "name":"Documentary"
    , "last_update":"2006-02-15 04:46:27"
    }
}
```

---
**参考**
`新版本MySQL安装后没有找到json的相关函数, 官方到底有没有做实现? - SegmentFault`
https://segmentfault.com/q/1010000004416463

`A UDF library of functions to map relational data to the JSON format.`
https://github.com/mysqludf/lib_mysqludf_json
