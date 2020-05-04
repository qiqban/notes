## 安装注册中心 zookeeper

```bash
docker volume create zookeeper

docker run -d -v zookeeper:/data -p 2181:2181 --restart always --name zookeeper zookeeper:latest
```

## 安装dubbo-admin

账号 root/root

## 提供者注册配置

- 应用名
  dubbo.application.name
- 注册中心
  dubbo.registry.protocol=zookeeper
  dubbo.registry.address=ip:port或者zookeeper//ip:port
- 协议
  dubbo.protocol.name=dubbo
  dubbo.protocol.port=20880
- 监控中心
  dubbo.monitor.protocol=registry
- 暴露接口
  ```xml
  <dubbo:service interface="" ref="" />
  ```
  或者
  @EnableDubbo   -- SpringBoot启动类
  @com.alibaba.dubbo.config.annotation.Service  -- Service类

## 消费者注册配置

- 应用名
  dubbo.application.name
- 注册中心
  dubbo.registry.protocol=zookeeper
  dubbo.registry.address=ip:port或者zookeeper//ip:port
- 监控中心
  dubbo.monitor.protocol=registry
- 调用接口
  ```xml
  <dubbo:reference interface="" id="" />
  ```
  或者
  @EnableDubbo   -- SpringBoot启动类
  @Reference  -- Service类注入

## 配置方式

- -D 比如：-Ddubbo.protocol.port:20880
- xml 比如：<dubbo:protocol port="20880" />
- properties 比如：dubbo.protocol.port=20880
- -D > xml > properties

## 配置

### 启动时检查

```properties
dubbo.reference.check=true
dubbo.consumer.check=true
dubbo.registry.check=true
```

### 超时

```properties
dubbo.reference.timeout=1000
```

### 重试次数

```properties
dubbo.reference.retries=3
```
多个提供方会自动更换，总重试次数为3

### 多版本

```properties
dubbo.service.version=1.0.0
dubbo.reference.version=1.0.0
```

### 本地存根

即在消费者本地实现接口让消费者远程调用前先调这个接口，可以用来做参数校验

```properties
dubbo.reference.stub=com.alibba.ServiceImpl
```

### 负载均衡

- Random LoadBalance 基于权重的随机负载均衡 默认
- RoundRobin LoadBalance 基于权重的轮询负载均衡
- LeastActive LoadBalance 最少活跃数负载均衡  即找最快的
- ConsistentHash LoadBalance 一致性Hash负载均衡

```properties
dubbo.reference.loadbalance=roundrobin
```

## 配置优先级

- 一级：方法 > 接口 > 全局
- 二级：消费者 > 提供者

## SpringBoot整合dubbo的3种方式

- 注解和application.properties配置
- xml配置， SpringBoot启动类注解@ImportResource(locations="classpath:provider.xml")
- 配置类

## 高可用

- zk或者注册中心挂掉了，消费者可以根据本地缓存记录的提供者ip端口进行直接调用
- dubbo直连: @Reference(url="ip:port")

## 服务降级

- 强制返回空 控制台的屏蔽
- 失败返回空 控制台的容错

## 容错模式

- Faillover 重试reties次数，适用于读操作
- Failfast 一次失败则报错，适用于非幂等操作
- Failsafe 失败直接忽略，适用于日志等操作
- Failback 失败定时重发，适用于消息通知操作
- Forking 并行调用forks个服务，一个成功则成功，适用于实时性要求高的读操作
- Broadcast 广播调用所有服务，一个报错则失败，适用于更新缓存等操作

```properties
dubbo.reference.cluster=failsafe
```

整合Hystrix
- spring-cloud-starter-netflix-hystrix
- SpringBoot启动类 @EnableHystrix
- 方法 @HystrixCommand(fallbackMethod="handler")

## 原理

### 流程

消费方调用存根 -> 存根封装调用信息 -> 序列化调用信息 -> 获取提供方地址 -> 发送给提供方存根
提供方存根接收到调用信息 -> 反序列化调用信息 -> 解析调用信息 -> 调用方法
方法返回结果 -> 提供方存根封装结果信息 -> 序列化结果信息 -> 返回给消费方存根
消费方存根接收到结果信息 -> 反序列化调用信息 -> 解析结果信息 -> 返回给消费方

### 基础

- NIO
  - channel
  - buffer
  - selector
  - selector轮流查看channel的进度，完成一个阶段则进行下一阶段，channel则读取或写入buffer
- Netty
  - bootstrap
  - Boss
  - Worker
  - bootstrap开启监听，有数据则交给Boss，Boss读取完则交给Worker，Worker处理完再回给Boss

### 框架设计

![dubbo-framework](_media/dubbo-framework.jpg)

- service
  - 用户认为的rpc，知道接口和实现就行了
- rpc
  - config 解析加载配置
  - proxy 代理对象来处理rpc
  - registry 服务注册与发现
  - cluster 路由和负载均衡
  - monitor 监控
  - protocol rpc调用核心
- remoting
  - exchange 数据交换，即架起client与server的通信管道
  - transport 数据传输，主要用netty实现
  - serialize 序列化

### 解析并加载配置

- DubboBeanDefinitionParser.parse

### 服务暴露

- ServiceBean.afterPropertiesSet setter所有配置信息

### 服务引用

### 服务调用