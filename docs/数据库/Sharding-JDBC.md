# Sharding-JDBC

## 原理

  - 解析
    - 词法分析 tokens
    - 语法分析 语法树
  - 路由
    - 分片键符号不同
      - 单片路由 等号 确定一个节点
      - 多片路由 in
      - 范围路由 between
    - 不带分片键
      - 广播路由，全库表路由
    - 根据分片键进行路由
      - 标准路由 不包括关联查询或仅限绑定表关联查询（实际对逻辑表来说不是关联）
      - 直接路由？
      - 笛卡尔积路由 主表与子表不知道怎么组合，所以都组合一遍
  - 改写
    - 改表名 逻辑表改为真实表
    - 补充列 为归并准备 group by order by
  - 执行
    - 内存限制模式 多线程，可并行 适合OLAP
    - 连接限制模式 可单连接串行控制事务 适合OLTP
  - 归并
    - 场景
      - 遍历 select
      - 排序
      - 分组
      - 分页
      - 聚合
    - 种类
      - 内存归并 消耗内存
      - 流式归并 比如排序，每次比较所有队列首，处理最大的并修改下表，再重复执行
      - 装饰者归并 比如聚合函数sum 在之前和的基础上再求和

## 如何解决分库分表问题

  - 分布式事务
    - 弱事务
      - 支持单库事务
      - 支持逻辑异常导致跨库回滚
      - 不支持断网、宕机导致的跨库回滚
    - 柔性事务
      - 针对断网、宕机有重试机制
      - 会记录事务情况
      - 修改操作的SQL要求幂等
        - 插入语句id要求固定，不能是数据库自动增加
        - 修改不能是自增，比如x=x+1
        - 删除不要求
  - 跨节点关联查询
    - 未知
  - 跨节点分页排序
    - 归并可以解决
  - 主键避重和唯一索引问题
    - 主键避重采用雪花算法
    - 唯一索引未知
  - 公共表
    - 每个库都有一个相同表
    - 广播式全修改一遍