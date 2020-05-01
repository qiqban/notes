## 包

org.springframework.cloud:spring-cloud-starter-netflix-eureka-server

## 集群搭建

##　注册服务

## Actuator完善

## Discovery信息

## 页面红字提示的解释

- Eureka开启自我保护模式
- 默认情况服务定时向注册中心发送心跳，如果90秒中丢失大量心跳，注册中心会注销该服务
- 丢失心跳可能因为网络卡顿等原因暂时接收不到，于注册中心开启保护模式以保留所有注册的服务
- 目的是不错杀注册的服务，以使服务更健壮

## 关闭自我保护模式
eureka.server.enable-self-preservation=false
eureka.server.eviction-interval-timer-in-ms=2000
eureka.instance.lease-renewal-interval-in-seconds=1
eureka.instance.lease-expiration-duration-in-seconds=2

## 2.0停止了，1.0继续维护