# Distributed Cache System

## Problem Description

Design and implement a distributed cache system that can handle multiple cache nodes, data partitioning, replication, and failure recovery. The system should provide high availability and consistency guarantees.

## Requirements

### CacheNode Class
- `CacheNode(nodeId, capacity)` - Create cache node with ID and capacity
- `put(key, value, ttl)` - Store key-value pair with time-to-live
- `get(key)` - Retrieve value by key
- `delete(key)` - Remove key-value pair
- `size()` - Get number of stored items
- `isHealthy()` - Check node health status
- `getStats()` - Get performance statistics

### ConsistentHash Class
- `ConsistentHash(nodes, virtualNodes)` - Initialize with nodes and virtual nodes
- `addNode(node)` - Add node to the ring
- `removeNode(node)` - Remove node from the ring
- `getNode(key)` - Get primary node for key
- `getReplicationNodes(key, replicas)` - Get nodes for replication

### DistributedCache Class
- `DistributedCache(nodes, replicationFactor, consistencyLevel)` - Initialize cache
- `put(key, value, ttl)` - Store with replication
- `get(key)` - Retrieve with read repair
- `delete(key)` - Remove from all replicas
- `getNodeForKey(key)` - Get responsible node
- `handleNodeFailure(nodeId)` - Handle node failure
- `addNode(node)` - Add new node and rebalance
- `removeNode(nodeId)` - Remove node and migrate data

### Replication Strategies
- **SYNC**: Synchronous replication (wait for all replicas)
- **ASYNC**: Asynchronous replication (best effort)
- **QUORUM**: Wait for majority of replicas

### Consistency Levels
- **ONE**: Read/write from one node
- **QUORUM**: Read/write from majority
- **ALL**: Read/write from all replicas

### Health Monitoring
- `HealthMonitor(cache)` - Monitor node health
- `checkNodeHealth(nodeId)` - Ping node health
- `handleUnhealthyNode(nodeId)` - Mark node as unhealthy
- `scheduleHealthChecks()` - Periodic health checks

## Examples

```java
// Initialize distributed cache
List<CacheNode> nodes = Arrays.asList(
    new CacheNode("node1", 1000),
    new CacheNode("node2", 1000),
    new CacheNode("node3", 1000)
);

DistributedCache cache = new DistributedCache(nodes, 2, ConsistencyLevel.QUORUM);

// Store data
cache.put("user:123", userData, 3600); // 1 hour TTL

// Retrieve data
User user = cache.get("user:123");

// Handle node failure
cache.handleNodeFailure("node2");

// Add new node
CacheNode newNode = new CacheNode("node4", 1000);
cache.addNode(newNode); // Triggers rebalancing

// Get statistics
CacheStats stats = cache.getStats();
System.out.println("Hit ratio: " + stats.getHitRatio());
```

## Suggested Test Cases

### CacheNode Tests
1. Store and retrieve key-value pairs
2. Handle TTL expiration
3. Capacity management and eviction
4. Node health status
5. Performance statistics tracking

### ConsistentHash Tests
1. Hash key distribution across nodes
2. Add node and verify rebalancing
3. Remove node and verify data migration
4. Virtual nodes improve distribution
5. Same key always maps to same nodes

### DistributedCache Tests
1. Store data across multiple nodes
2. Retrieve data with read repair
3. Delete data from all replicas
4. Handle node failures gracefully
5. Replication factor compliance
6. Consistency level enforcement

### Replication Tests
1. Synchronous replication waits for all
2. Asynchronous replication doesn't block
3. Quorum replication waits for majority
4. Handle partial replication failures
5. Read repair fixes inconsistencies

### Failure Recovery Tests
1. Node failure detection
2. Automatic failover
3. Data reconstruction from replicas
4. Network partition handling
5. Split-brain prevention

### Performance Tests
1. Concurrent read/write operations
2. Load balancing across nodes
3. Cache hit/miss ratios
4. Response time measurements
5. Throughput under load

## TDD Steps

1. **Red**: Test CacheNode basic put/get
2. **Green**: Implement basic cache node
3. **Red**: Test TTL expiration
4. **Green**: Implement TTL handling
5. **Refactor**: Optimize cache node performance
6. **Red**: Test ConsistentHash key distribution
7. **Green**: Implement consistent hashing
8. **Red**: Test node addition/removal
9. **Green**: Implement ring rebalancing
10. **Red**: Test DistributedCache replication
11. **Green**: Implement basic replication
12. **Red**: Test consistency levels
13. **Green**: Implement consistency guarantees
14. **Red**: Test failure handling
15. **Green**: Implement failure recovery
16. **Red**: Test performance under load
17. **Green**: Optimize for performance
18. **Refactor**: Clean up entire system

## Learning Goals

- Distributed systems concepts
- Consistent hashing algorithms
- Replication strategies
- Failure detection and recovery
- Consensus algorithms
- Performance optimization
- Concurrent programming
- Network programming
- Monitoring and metrics
- System design patterns

## Advanced Considerations

- **CAP Theorem**: Choose between Consistency, Availability, and Partition tolerance
- **Vector Clocks**: Track causality in distributed updates
- **Gossip Protocol**: Efficient information dissemination
- **Read Repair**: Fix inconsistencies during reads
- **Anti-entropy**: Periodic synchronization between replicas
- **Bloom Filters**: Memory-efficient existence checking

---

## 问题描述

设计并实现一个分布式缓存系统，能够处理多个缓存节点、数据分区、复制和故障恢复。系统应提供高可用性和一致性保证。

## 需求

### CacheNode 类
- `CacheNode(nodeId, capacity)` - 创建带ID和容量的缓存节点
- `put(key, value, ttl)` - 存储带生存时间的键值对
- `get(key)` - 通过键检索值
- `delete(key)` - 删除键值对
- `size()` - 获取存储项目数量
- `isHealthy()` - 检查节点健康状态
- `getStats()` - 获取性能统计

### ConsistentHash 类
- `ConsistentHash(nodes, virtualNodes)` - 用节点和虚拟节点初始化
- `addNode(node)` - 添加节点到环中
- `removeNode(node)` - 从环中移除节点
- `getNode(key)` - 获取键的主节点
- `getReplicationNodes(key, replicas)` - 获取复制节点

### DistributedCache 类
- `DistributedCache(nodes, replicationFactor, consistencyLevel)` - 初始化缓存
- `put(key, value, ttl)` - 带复制的存储
- `get(key)` - 带读修复的检索
- `delete(key)` - 从所有副本中删除
- `getNodeForKey(key)` - 获取负责节点
- `handleNodeFailure(nodeId)` - 处理节点故障
- `addNode(node)` - 添加新节点并重新平衡
- `removeNode(nodeId)` - 移除节点并迁移数据

### 复制策略
- **SYNC**: 同步复制（等待所有副本）
- **ASYNC**: 异步复制（尽力而为）
- **QUORUM**: 等待大多数副本

### 一致性级别
- **ONE**: 从一个节点读/写
- **QUORUM**: 从大多数节点读/写
- **ALL**: 从所有副本读/写

### 健康监控
- `HealthMonitor(cache)` - 监控节点健康
- `checkNodeHealth(nodeId)` - 检查节点健康
- `handleUnhealthyNode(nodeId)` - 标记节点为不健康
- `scheduleHealthChecks()` - 定期健康检查

## 示例

```java
// 初始化分布式缓存
List<CacheNode> nodes = Arrays.asList(
    new CacheNode("node1", 1000),
    new CacheNode("node2", 1000),
    new CacheNode("node3", 1000)
);

DistributedCache cache = new DistributedCache(nodes, 2, ConsistencyLevel.QUORUM);

// 存储数据
cache.put("user:123", userData, 3600); // 1小时TTL

// 检索数据
User user = cache.get("user:123");

// 处理节点故障
cache.handleNodeFailure("node2");

// 添加新节点
CacheNode newNode = new CacheNode("node4", 1000);
cache.addNode(newNode); // 触发重新平衡

// 获取统计信息
CacheStats stats = cache.getStats();
System.out.println("命中率: " + stats.getHitRatio());
```

## 建议测试用例

### CacheNode 测试
1. 存储和检索键值对
2. 处理TTL过期
3. 容量管理和逐出
4. 节点健康状态
5. 性能统计跟踪

### ConsistentHash 测试
1. 键在节点间的哈希分布
2. 添加节点并验证重新平衡
3. 移除节点并验证数据迁移
4. 虚拟节点改善分布
5. 相同键总是映射到相同节点

### DistributedCache 测试
1. 跨多节点存储数据
2. 带读修复的检索数据
3. 从所有副本删除数据
4. 优雅处理节点故障
5. 复制因子合规性
6. 一致性级别强制执行

### 复制测试
1. 同步复制等待所有
2. 异步复制不阻塞
3. 法定人数复制等待大多数
4. 处理部分复制失败
5. 读修复修复不一致

### 故障恢复测试
1. 节点故障检测
2. 自动故障转移
3. 从副本重建数据
4. 网络分区处理
5. 脑裂预防

### 性能测试
1. 并发读/写操作
2. 跨节点负载平衡
3. 缓存命中/未命中率
4. 响应时间测量
5. 负载下的吞吐量

## TDD 步骤

1. **红灯**: 测试CacheNode基本put/get
2. **绿灯**: 实现基本缓存节点
3. **红灯**: 测试TTL过期
4. **绿灯**: 实现TTL处理
5. **重构**: 优化缓存节点性能
6. **红灯**: 测试ConsistentHash键分布
7. **绿灯**: 实现一致性哈希
8. **红灯**: 测试节点添加/移除
9. **绿灯**: 实现环重新平衡
10. **红灯**: 测试DistributedCache复制
11. **绿灯**: 实现基本复制
12. **红灯**: 测试一致性级别
13. **绿灯**: 实现一致性保证
14. **红灯**: 测试故障处理
15. **绿灯**: 实现故障恢复
16. **红灯**: 测试负载下性能
17. **绿灯**: 性能优化
18. **重构**: 清理整个系统

## 学习目标

- 分布式系统概念
- 一致性哈希算法
- 复制策略
- 故障检测和恢复
- 共识算法
- 性能优化
- 并发编程
- 网络编程
- 监控和指标
- 系统设计模式

## 高级考虑

- **CAP定理**: 在一致性、可用性和分区容忍性之间选择
- **向量时钟**: 跟踪分布式更新中的因果关系
- **Gossip协议**: 高效的信息传播
- **读修复**: 在读取期间修复不一致
- **反熵**: 副本间定期同步
- **布隆过滤器**: 内存高效的存在性检查