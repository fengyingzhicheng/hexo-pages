---
title: Redis PipeLine使用实例
date: 2020-11-24 20:32:24
updated: 2020-11-24 20:32:24
categories: 
           - 项目优化
tags: 
- design
- redis
- excel
---


# 背景需求

在制作数据中台BI项目时候遇到的如下项目，需要将excel数据绑定到一份数据集（TableSelectInfo表），界面有上传excel、追加上传和解析的Excel字段信息以及当前数据集的分页预览数据，确定保存产生绑定关系(数据集到源数据）。对应着更新数据集时可以进行excel追加，如果取消则不会插入，确定才会插入到库中。

# 设计思路

本身考虑到数据并非上传后就绑定到数据集中，上传的excel和追加上传应该存储到临时的mongodb集合中，每次上传后MongoDB集合名插入到Redis的List中确定后异步线程去将其他集合的数据插入到第一个集合。对应的数据集状态分为 操作中 数据读取中 和已完成

`TableSelectedInfo`

| id | status     | mongodbCollectionName |
| -- | ---------- | --------------------- |
| 1  | 操作中     |                       |
| 2  | 数据读取中 |                       |
| 3  | 已完成     |                       |

`Redis`

key: tableSelected_Id

value: `List` collectionName

# 注意事项

**Redis缓存的续费策略：**

1. 如果Redis不设置续费可能会导致追加的数据丢失
2. 默认设置当前Redis key为30s
3. 使用定时任务扫描TableSelectedInfo中的状态不为已完成的数据集 进行redis key的批量续期 20s

**多人同时操作**

前端在进入更新数据集的界面之前需要调取接口获取数据集的状态，如果不为已经完成。说明有其他人正在操作

# Expire批量续期操作

开发过程中发现redis的key续期都是单独key的 ，项目使用List存储的方式+在使用状态的数据集的key进行批量续费。目前针对批量操作主要可以使用lua脚本和pipeline 方案。项目采用了pipeline方式

批量续费策略

```java
final String[] keys = {"key1", "key2", "key3", "key4"};
final Long timeout = 30L;
final TimeUnit unit = TimeUnit.DAYS;
final long rawTimeout = TimeoutUtils.toMillis(timeout, unit);
stringRedisTemplate.executePipelined(new RedisCallback<Object>() {
    @Override
    public Object doInRedis(RedisConnection connection) throws DataAccessException {
        for (String key : keys) {
            byte[] rawKey = RedisTemplateSerializerUtil.serializeKey(stringRedisTemplate, key);
            connection.pExpire(rawKey, rawTimeout);
        }
        return null;
    }
});
```

参考自[https://blog.csdn.net/mhxy199288/article/details/52911721](https://blog.csdn.net/mhxy199288/article/details/52911721)