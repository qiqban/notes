# RocketMQ

## 集群架构

![RocketMQ架构](_media/rmq-basic-arc.png)

- NameServer之间没有同步关系
- Broker自行注册到所有NameServer上
- 多个Broker具有一个BrokerName, 其中BrokerId=0是BrokerMaster, 否则是BrokerSlave
- BrokerMaster最好用于生产，BrokerSlave最好用于消费
- 每个BrokerMaster可以提供多个Topic服务

### 疑问

- 是否多个BrokerMaster对应一个Topic
- 如果是，那么生产者都要生产，还是BrokerMaster会自行同步

## 集群模式

### 单BrokerMaster
  
用于开发、测试环境

### 多BrokerMaster
  
- 解决单节点问题
- 但是单节点被消费时宕机，期间剩余未被消费的消息不可订阅

### 异步多BrokerMaster多BrokerSlave

- master落地数据后异步传给slave时响应生产者
- 数据在多个slave上有备份，不怕master节点故障了
- 异步传给slave期间有丢失数据的可能性

### 同步多BrokerMaster多BrokerSlave

- master落地数据后同步传给slave，完成后响应生产者
- 数据在多个slave上有备份，不怕master节点故障了
- RT略高、性能比异步低大约10%
- 有master宕机slave不能选举master的问题？

## 集群搭建

### 环境准备

- ip配置
- 防火墙配置
- RokectMQ的bin目录环境变量配置，方便使用命令
- 创建消息存储目录

### 准备Broker配置文件

- brokerRole
  - ASYNC-MASTER
  - SYNC-MASTER
  - SLAVE
- flushDiskType
  - ASYNC-FLUSH
  - SYNC-FLUSH

### 调整JVM内存参数

- runbroker.sh
  JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn=128m"
- runserver.sh
  JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn=128m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"

### 启动服务

- 启动NameServer

```bash
nohup sh mqnamesrv &
```

- 启动Broker

```bash
nohup sh mqbroker -c <配置文件路径> &
```