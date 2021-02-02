```
Eureka 核心概念，整体上可以分为两个主体：Eureka Server 和 Eureka Client。
```

# Eureka服务状态

 Eureka 的工作流程：

1、Eureka Server 启动成功，等待服务端注册。在启动过程中如果配置了集群，集群之间定时通过 Replicate 同步注册表，每个 Eureka Server 都存在独立完整的服务注册表信息

2、Eureka Client 启动时根据配置的 Eureka Server 地址去注册中心注册服务

3、Eureka Client 会每 30s 向 Eureka Server 发送一次心跳请求，证明客户端服务正常

4、当 Eureka Server 90s 内没有收到 Eureka Client 的心跳，注册中心则认为该节点失效，会注销该实例

5、单位时间内 Eureka Server 统计到有大量的 Eureka Client 没有上送心跳，则认为可能为网络异常，进入自我保护机制，不再剔除没有上送心跳的客户端

6、当 Eureka Client 心跳请求恢复正常之后，Eureka Server 自动退出自我保护模式

7、Eureka Client 定时全量或者增量从注册中心获取服务注册表，并且将获取到的信息缓存到本地

8、服务调用时，Eureka Client 会先从本地缓存找寻调取的服务。如果获取不到，先从注册中心刷新注册表，再同步到本地缓存

9、Eureka Client 获取到目标服务器信息，发起服务调用

10、Eureka Client 程序关闭时向 Eureka Server 发送取消请求，Eureka Server 将实例从注册表中删除

这就是Eurka基本工作流程



# Eureka命令

> eureka信息查看

```
get: {ip:port}/eureka/status
1
```

> 注册到eureka的服务信息查看

```
get: {ip:port}/eureka/apps
1
```

> 注册到eureka的具体的服务查看

```
get: {ip:port}/eureka/apps/{appname}/{id}
对应eureka源码的：InstanceResource.getInstanceInfo
```

> 服务续约

```
put：{ip:port}/eureka/apps/{appname}/{id}?lastDirtyTimestamp={}&status=up
对应eureka源码的：InstanceResource.renewLease
```

> 更改服务状态

```
put：{ip:port}/eureka/apps/{appname}/{id}/status?lastDirtyTimestamp={}&value={UP/DOWN}
对应eureka源码的：InstanceResource.statusUpdate
```

> 删除状态更新

```
delete：{ip:port}/eureka/apps/{appname}/{id}/status?lastDirtyTimestamp={}&value={UP/DOWN}
对应eureka源码的：InstanceResource.deleteStatusUpdate
```

> 删除服务

```
delete: {ip:port}/eureka/apps/{appname}/{id}
对应eureka源码的：InstanceResource.cancelLease
```