# 分布式

## session 分布式处理

第一种：粘性session。粘性Session是指将用户锁定到某一个服务器上（通过NG）。

第二种：服务器session复制。

第三种：session共享机制。使用分布式缓存方案比如memcached、Redis，但是要求Memcached或Redis必须是集群。

第四种：session持久化到数据库。

## 谈谈业务中使用分布式的场景

## 分布式锁的场景

## 分布是锁的实现方案

## 分布式事务

## 集群与负载均衡的算法与实现

## 说说分库与分表设计

## 分库与分表带来的分布式困境与应对之策