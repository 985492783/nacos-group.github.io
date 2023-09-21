---
title: DistributedLock
keywords: DistributedLock
description: DistributedLock
---

# 分布式锁

> nacos lock分布式锁从3.0版本开始支持

## 参数

|参数名|默认值| 启止版本     | 说明     |
|-----|-----|----------------|--------|
|nacos.lock.default_expire_time | 30000 | 3.0.0 ~ latest | 默认超时时间 |
|nacos.lock.max_expire_time | 1800000 | 3.0.0 ~ latest | 最大超时时间 |

## 分布式锁场景

分布式定时任务，同时只有一个客户端能获取分布式锁，获取锁失败的客户端不进行任何操作，获取到锁的客户端进行业务操作，在超时时间内完成操作并且释放该锁。

## 接口说明

### 分布式锁实例

通过spi插件化设计成可拓展的分布式锁，例如锁重入，服务端心跳探测等等功能将会在插件中实现不同的分布式锁处理方案。
> `LockInstance`结构  
> Nacos通过 lockType和key两个参数组成的`LockKey` 在集群中确认唯一的锁实例

| 属性名 | 属性类型 | 描述                            |
|:--------|:-------|:------------------------------|
| key | string | 锁名，同一业务场景的锁名称应该一致             |
| expireTimestamp | Long | 锁自动超时时间，默认为30s                |
| params | Map<String, ? extends Serializable>   | 额外参数，用于插件设计时传入特殊参数进行处理        |
| lockType | String | 锁类型，用于获取锁真实处理方案，默认为NACOS_LOCK |

### 锁的获取与释放

#### 分布式锁获取

##### 描述

用于服务获取分布式锁

```java
public Boolean lock(LockInstance instance)throws NacosException;
```

请求参数

| 参数名 | 参数类型   | 描述                      |
|:----|:-------|:------------------------|
| instance | LockInstance | 锁实例，用于传输锁的唯一标识，超时时间以及参数 |

返回值

| 参数类型    | 描述   |
|:--------|:-----|
| Boolean | 是否成功 |

异常说明

获取锁超时或网络异常，抛出 NacosException 异常。

#### 分布式锁释放

##### 描述

用于服务释放分布式锁

```java
public Boolean unLock(LockInstance instance)throws NacosException;
```

请求参数

| 参数名 | 参数类型   | 描述                      |
|:----|:-------|:------------------------|
| instance | LockInstance | 锁实例，用于传输锁的唯一标识，超时时间以及参数 |

返回值

| 参数类型    | 描述   |
|:--------|:-----|
| Boolean | 是否成功 |

异常说明

释放锁超时或网络异常，抛出 NacosException 异常。

## 使用方式

### 依赖

> 引入nacos-client  
> 详情见[Java的SDK](./sdk.md)

Maven 坐标

```
<dependency>
    <groupId>com.alibaba.nacos</groupId>
    <artifactId>nacos-client</artifactId>
    <version>${latest.version}</version>
</dependency>
```

### Java SDK

在java程序中使用nacos分布式锁  
请求示例

```java
public class Main {
    
    public static void main(String[] args) throws NacosException {
        Properties properties = new Properties();
        properties.setProperty("serverAddr", "localhost");
        //properties.setProperty("username", "nacos");
        //properties.setProperty("password", "nacos");
        LockService lockService = NacosFactory.createLockService(properties);
        NLock nLock = NLockFactory.getLock("key");
        
        Boolean lock = lockService.lock(nLock);
        if (Boolean.TRUE.equals(lock)) {
            try {
                //do something
                System.out.println("do something");
            } finally {
                lockService.unLock(nLock);
            }
        }
    }
}
```
