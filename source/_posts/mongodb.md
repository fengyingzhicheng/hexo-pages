---
title: MongoDB聚合操作
date: 2020-12-24 15:40:24
updated: 2020-12-24 15:40:24
categories: 
           - NoSQL
tags:
           - mongodb
---

* `$project`：修改输入文档的结构。可以用来重命名、增加或删除域，也可以用于创建计算结果以及嵌套文档。
* `$match`：用于过滤数据，只输出符合条件的文档。$match使用MongoDB的标准查询操作。
* `$limit`：用来限制MongoDB聚合管道返回的文档数。
* `$skip`：在聚合管道中跳过指定数量的文档，并返回余下的文档。
* `$unwind`：将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。
* `$group`：将集合中的文档分组，可用于统计结果。
* `$sort`：将输入文档排序后输出。
* `$geoNear`：输出接近某一地理位置的有序文档。
* `$bucket`: 分组（分桶）计算。
* `$facet` : 多次分组计算。
* `$out`: 将结果集输出，必须是Pipline最后一个Stage。

`$unwind`

```java
db.inventory2.aggregate( [ { $unwind : "$sizes" } ] )
```

![image.png](image-20220317202753-zj3qxuy.png)

![image.png](image-20220317202728-pwzjz52.png)

`$bucket`

```java
db.artwork.aggregate( [
   {
     $bucket: {
       groupBy: "$price",
       boundaries: [ 0, 200, 400 ],
       default: "Other",
       output: {
         "count": { $sum: 1 },
         "titles" : { $push: "$title" }
       }
     }
   }
 ] )
```

![image.png](image-20220317202810-6aahpm4.png)

![image.png](image-20220317202838-b1rddzj.png)

![image.png](image-20220317202903-zxbzuw2.png)