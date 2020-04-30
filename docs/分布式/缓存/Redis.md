# 基础

## 简介

- Redis : REmote DIctionary Service
- author : Salvatore Sanfilippo

## 基本使用

- 启动服务 redis-server
- 连接 redis-cli
- 帮助 help
- 普通CRUB
  - set key value
  - get key
  - incr key
  - del key
- 批量CRUB
  - mset key value key value ...
  - mget key key ...
- 事务
  multi ... exec

# 复杂数据类型

## hash表

```bash
hset hashkey key value
hget hashkey key
hmset hashkey key value key value ...
hmget hashkey key key ...
hkeys hashkey
hvals hashkey
```

## list表
```bash
l(r)push listkey value value ...
l(r)range listkey 0  -1
l(r)rem listkey 0 value
l(r)pop listkey
rpoplpush listkey listkey
```

## 阻塞列表
## 有序集合
## 范围
## 并集
	
# 到期

# 命名空间

# 其他命令

# 高级应用

- 接口
- 服务器信息
- redis配置
- AOF = Append Only File
- 主从复制
- 数据存储
- 集群
- Bloom过滤
- SETBIT/GETBIT


# 与其他数据库合作

- 多持久并发服务
- 数据填充
- 关系存储
- 服务