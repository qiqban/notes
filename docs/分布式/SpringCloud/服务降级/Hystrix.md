- 服务降级 fallback
  - 什么情况使用
    - 运行异常
    - 超时
    - 熔断
    - 线程池、内存打满
- 服务熔断
  - 失败达到指定程度则断开
  - 后成功达到指定程度则恢复
```java
@HystrixProperty(name = "circuitBreaker.enabled", value = "true"), // 开启熔断
@HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"), // 请求次数
@HystrixProperty(name = "circuitBreaker.sleepwindowInMilliseconds", value = "10000"), // 时间范围
@HystrixProperty(name = "circuitBreaker.erorThresholdPercentage", value = "60"), // 失败率
```
- 服务限流