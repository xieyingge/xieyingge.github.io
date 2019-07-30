---
layout:     post
title:      redis sentinel 
subtitle:   java jedis 连接测试代码😅
date:       2019-07-30
author:     BY
header-img: img/post-bg-ios10.jpg
catalog: true
tags:
    - java
    - redis
---


### 1、redis主从配置启动
redis-cli -p 6379 info replication 查看主从关系，以及是否连接
### 2、redis-sentinel哨兵启动
redis-cli -p 26379 info sentinel 连接多少个redis以及sentinel
### 3、运行代码测试
代码运行过程中shutdown  redis的master节点，然后一段时间后sentinel会failover出来一个新的master，代码继续运行

例如

```java
public class SentinelPoolFailOverTest {
    public static void main(String[] args) {
        String masterName = "mymaster";
        Set<String> sentinels = new HashSet<>();
        sentinels.add("192.168.0.30:26379");
        sentinels.add("192.168.0.30:26380");
        sentinels.add("192.168.0.30:26381");
        JedisSentinelPool sentinelPool = new JedisSentinelPool(masterName, sentinels);
        int i = 0;
        while (true) {
            i++;
            Jedis jedis = null;
            try {
                jedis = sentinelPool.getResource();
                String key = "key" + i;
                String value = "value" + i;
                jedis.set(key, value);
                if (i % 100 == 0) {
                    System.out.println(jedis.get(key));
                }
                TimeUnit.MILLISECONDS.sleep(10);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                if (jedis != null) {
                    jedis.close();
                }
            }
        }
    }
}
```

