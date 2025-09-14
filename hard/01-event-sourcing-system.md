# Event Sourcing System

## Problem Description

Build an event sourcing system that captures all changes to application state as a sequence of events. Instead of storing current state, the system stores events and reconstructs state by replaying events.

## Requirements

### Event Class
- `Event(eventType, aggregateId, eventData, timestamp, version)` - Create immutable event
- `getEventType()` - Get type of event (e.g., "UserCreated", "EmailChanged")
- `getAggregateId()` - Get ID of the aggregate this event affects
- `getEventData()` - Get event payload data
- `getTimestamp()` - Get when event occurred
- `getVersion()` - Get event version for optimistic locking

### EventStore Interface
- `appendEvent(event)` - Store new event
- `getEvents(aggregateId)` - Get all events for specific aggregate
- `getEventsAfterVersion(aggregateId, version)` - Get events after specific version
- `getAllEvents()` - Get all events in chronological order
- `getEventsByType(eventType)` - Filter events by type

### Aggregate Base Class
- `loadFromHistory(events)` - Rebuild aggregate state from events
- `getUncommittedEvents()` - Get events not yet persisted
- `markEventsAsCommitted()` - Clear uncommitted events
- `getVersion()` - Get current aggregate version
- `applyEvent(event)` - Apply single event to aggregate

### User Aggregate Example
- `createUser(id, email, name)` - Create new user (generates UserCreated event)
- `changeEmail(newEmail)` - Change email (generates EmailChanged event)
- `deactivateUser()` - Deactivate user (generates UserDeactivated event)

### EventBus
- `publish(event)` - Publish event to subscribers
- `subscribe(eventType, handler)` - Subscribe handler to event type
- `unsubscribe(eventType, handler)` - Unsubscribe handler

### Repository Pattern
- `save(aggregate)` - Save aggregate by storing its uncommitted events
- `getById(id)` - Load aggregate by replaying its events

## Examples

```python
# Create events
user_created = Event("UserCreated", "user-123",
                    {"email": "john@example.com", "name": "John Doe"},
                    datetime.now(), 1)

email_changed = Event("EmailChanged", "user-123",
                     {"oldEmail": "john@example.com", "newEmail": "john.doe@example.com"},
                     datetime.now(), 2)

# Store events
event_store = InMemoryEventStore()
event_store.append_event(user_created)
event_store.append_event(email_changed)

# Rebuild aggregate from events
user_events = event_store.get_events("user-123")
user = User()
user.load_from_history(user_events)

print(user.email)  # john.doe@example.com
print(user.version)  # 2

# Repository usage
repository = UserRepository(event_store)
user = repository.get_by_id("user-123")
user.change_email("new@example.com")
repository.save(user)  # Stores EmailChanged event
```

## Suggested Test Cases

### Event Tests
1. Create immutable event with all properties
2. Event equality and hash code
3. Event serialization/deserialization
4. Invalid event data handling

### EventStore Tests
1. Append single and multiple events
2. Retrieve events by aggregate ID
3. Retrieve events after specific version
4. Filter events by type
5. Handle concurrent event appending
6. Optimistic locking with version conflicts

### Aggregate Tests
1. Load aggregate from empty event history
2. Load aggregate from multiple events
3. Apply events in correct order
4. Track uncommitted events
5. Version management
6. Event application order matters

### User Aggregate Tests
1. Create user generates correct event
2. Change email generates correct event
3. Deactivate user generates correct event
4. State reconstruction from events
5. Business rule validation

### Repository Tests
1. Save aggregate stores uncommitted events
2. Load aggregate rebuilds from events
3. Handle non-existent aggregates
4. Optimistic concurrency control

### EventBus Tests
1. Publish events to subscribers
2. Multiple handlers for same event type
3. Unsubscribe handlers
4. Handle exceptions in event handlers

## TDD Steps

1. **Red**: Test Event creation with immutability
2. **Green**: Implement immutable Event class
3. **Red**: Test EventStore append functionality
4. **Green**: Implement basic EventStore
5. **Red**: Test event retrieval by aggregate ID
6. **Green**: Implement event retrieval
7. **Refactor**: Optimize EventStore implementation
8. **Red**: Test Aggregate loading from events
9. **Green**: Implement Aggregate base class
10. **Red**: Test User aggregate creation
11. **Green**: Implement User aggregate with events
12. **Red**: Test Repository save/load
13. **Green**: Implement Repository pattern
14. **Red**: Test EventBus publish/subscribe
15. **Green**: Implement EventBus
16. **Refactor**: Clean up and optimize entire system

## Learning Goals

- Event-driven architecture
- CQRS (Command Query Responsibility Segregation)
- Immutable data structures
- Optimistic concurrency control
- Domain-driven design
- Repository pattern
- Observer pattern (EventBus)
- Time-based testing
- Complex system integration testing

---

## 问题描述

构建一个事件溯源系统，将应用程序状态的所有更改捕获为事件序列。系统不存储当前状态，而是存储事件并通过重放事件来重建状态。

## 需求

### Event 类
- `Event(eventType, aggregateId, eventData, timestamp, version)` - 创建不可变事件
- `getEventType()` - 获取事件类型（如 "UserCreated", "EmailChanged"）
- `getAggregateId()` - 获取此事件影响的聚合ID
- `getEventData()` - 获取事件载荷数据
- `getTimestamp()` - 获取事件发生时间
- `getVersion()` - 获取事件版本用于乐观锁

### EventStore 接口
- `appendEvent(event)` - 存储新事件
- `getEvents(aggregateId)` - 获取特定聚合的所有事件
- `getEventsAfterVersion(aggregateId, version)` - 获取特定版本后的事件
- `getAllEvents()` - 按时间顺序获取所有事件
- `getEventsByType(eventType)` - 按类型筛选事件

### Aggregate 基类
- `loadFromHistory(events)` - 从事件重建聚合状态
- `getUncommittedEvents()` - 获取尚未持久化的事件
- `markEventsAsCommitted()` - 清除未提交事件
- `getVersion()` - 获取当前聚合版本
- `applyEvent(event)` - 对聚合应用单个事件

### User 聚合示例
- `createUser(id, email, name)` - 创建新用户（生成 UserCreated 事件）
- `changeEmail(newEmail)` - 更改邮箱（生成 EmailChanged 事件）
- `deactivateUser()` - 停用用户（生成 UserDeactivated 事件）

### EventBus
- `publish(event)` - 向订阅者发布事件
- `subscribe(eventType, handler)` - 为事件类型订阅处理器
- `unsubscribe(eventType, handler)` - 取消订阅处理器

### Repository 模式
- `save(aggregate)` - 通过存储未提交事件保存聚合
- `getById(id)` - 通过重放事件加载聚合

## 示例

```python
# 创建事件
user_created = Event("UserCreated", "user-123",
                    {"email": "john@example.com", "name": "John Doe"},
                    datetime.now(), 1)

email_changed = Event("EmailChanged", "user-123",
                     {"oldEmail": "john@example.com", "newEmail": "john.doe@example.com"},
                     datetime.now(), 2)

# 存储事件
event_store = InMemoryEventStore()
event_store.append_event(user_created)
event_store.append_event(email_changed)

# 从事件重建聚合
user_events = event_store.get_events("user-123")
user = User()
user.load_from_history(user_events)

print(user.email)  # john.doe@example.com
print(user.version)  # 2

# Repository 使用
repository = UserRepository(event_store)
user = repository.get_by_id("user-123")
user.change_email("new@example.com")
repository.save(user)  # 存储 EmailChanged 事件
```

## 建议测试用例

### Event 测试
1. 创建包含所有属性的不可变事件
2. 事件相等性和哈希码
3. 事件序列化/反序列化
4. 无效事件数据处理

### EventStore 测试
1. 追加单个和多个事件
2. 按聚合ID检索事件
3. 检索特定版本后的事件
4. 按类型筛选事件
5. 处理并发事件追加
6. 版本冲突的乐观锁

### Aggregate 测试
1. 从空事件历史加载聚合
2. 从多个事件加载聚合
3. 按正确顺序应用事件
4. 跟踪未提交事件
5. 版本管理
6. 事件应用顺序的重要性

### User Aggregate 测试
1. 创建用户生成正确事件
2. 更改邮箱生成正确事件
3. 停用用户生成正确事件
4. 从事件重建状态
5. 业务规则验证

### Repository 测试
1. 保存聚合存储未提交事件
2. 加载聚合从事件重建
3. 处理不存在的聚合
4. 乐观并发控制

### EventBus 测试
1. 向订阅者发布事件
2. 同一事件类型的多个处理器
3. 取消订阅处理器
4. 处理事件处理器中的异常

## TDD 步骤

1. **红灯**: 测试不可变事件创建
2. **绿灯**: 实现不可变事件类
3. **红灯**: 测试EventStore追加功能
4. **绿灯**: 实现基本EventStore
5. **红灯**: 测试按聚合ID检索事件
6. **绿灯**: 实现事件检索
7. **重构**: 优化EventStore实现
8. **红灯**: 测试聚合从事件加载
9. **绿灯**: 实现聚合基类
10. **红灯**: 测试User聚合创建
11. **绿灯**: 实现带事件的User聚合
12. **红灯**: 测试Repository保存/加载
13. **绿灯**: 实现Repository模式
14. **红灯**: 测试EventBus发布/订阅
15. **绿灯**: 实现EventBus
16. **重构**: 清理和优化整个系统

## 学习目标

- 事件驱动架构
- CQRS（命令查询职责分离）
- 不可变数据结构
- 乐观并发控制
- 领域驱动设计
- Repository模式
- 观察者模式（EventBus）
- 基于时间的测试
- 复杂系统集成测试