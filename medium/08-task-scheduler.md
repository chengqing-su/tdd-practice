# Task Scheduler System

## Problem Description

Build a task scheduling system that can execute tasks at specified times, handle recurring tasks, manage dependencies, and provide monitoring capabilities.

## Requirements

### Task Class
- `Task(taskId, name, command, scheduleTime, priority)` - Create schedulable task
- `execute()` - Execute the task
- `getStatus()` - Get current task status (PENDING, RUNNING, COMPLETED, FAILED)
- `getDuration()` - Get execution time
- `getResult()` - Get execution result or error
- `retry()` - Retry failed task

### Schedule Class
- `Schedule(scheduleType, parameters)` - Define when task should run
- `getNextExecutionTime(currentTime)` - Calculate next run time
- `isTimeToRun(currentTime)` - Check if task should run now
- Schedule types: ONCE, RECURRING, CRON_EXPRESSION

### TaskScheduler Class
- `scheduleTask(task, schedule)` - Schedule task for execution
- `cancelTask(taskId)` - Cancel scheduled task
- `pauseTask(taskId)` - Pause task execution
- `resumeTask(taskId)` - Resume paused task
- `executeNow(taskId)` - Execute task immediately
- `getScheduledTasks()` - Get all scheduled tasks
- `getTaskHistory(taskId)` - Get task execution history

### Dependency System
- `TaskDependency(taskId, dependsOnTaskId, dependencyType)` - Task dependency
- `addDependency(taskId, dependsOnTaskId)` - Add task dependency
- `removeDependency(taskId, dependsOnTaskId)` - Remove dependency
- `canExecute(taskId)` - Check if all dependencies are satisfied
- Dependency types: SUCCESS, COMPLETION, FAILURE

### Monitoring and Alerts
- `TaskMonitor` - Monitor task execution and health
- `getTaskMetrics(taskId)` - Get task performance metrics
- `setAlert(condition, action)` - Set up alerts for task conditions
- `getSystemHealth()` - Get overall scheduler health
- `generateReport(timeRange)` - Generate execution reports

## Features

### Scheduling Options
- One-time task execution at specific time
- Recurring tasks (hourly, daily, weekly, monthly)
- Cron expression support for complex schedules
- Relative scheduling (after X minutes/hours)
- Calendar-based scheduling (specific dates)

### Task Management
- Task prioritization
- Task queuing and throttling
- Retry mechanisms with backoff strategies
- Task timeout handling
- Task cancellation and cleanup

### Advanced Features
- Task dependency chains
- Conditional task execution
- Task grouping and batching
- Resource allocation and limits
- Distributed execution support

## Examples

```python
scheduler = TaskScheduler()

# One-time task
task1 = Task("backup-db", "Database Backup", "/bin/backup.sh", Priority.HIGH)
once_schedule = Schedule(ScheduleType.ONCE, {"executeAt": datetime(2024, 1, 15, 2, 0)})
scheduler.schedule_task(task1, once_schedule)

# Recurring task (daily at 3 AM)
task2 = Task("daily-report", "Generate Daily Report", "/bin/report.py", Priority.MEDIUM)
daily_schedule = Schedule(ScheduleType.RECURRING,
                         {"interval": "daily", "time": "03:00"})
scheduler.schedule_task(task2, daily_schedule)

# Cron expression task (every Monday at 9 AM)
task3 = Task("weekly-cleanup", "Weekly Cleanup", "/bin/cleanup.sh", Priority.LOW)
cron_schedule = Schedule(ScheduleType.CRON_EXPRESSION, {"expression": "0 9 * * 1"})
scheduler.schedule_task(task3, cron_schedule)

# Task with dependency
task4 = Task("send-report", "Email Report", "/bin/email.py", Priority.MEDIUM)
dependency = TaskDependency(task4.id, task2.id, DependencyType.SUCCESS)
scheduler.add_dependency(dependency)
scheduler.schedule_task(task4, daily_schedule)

# Monitor execution
monitor = TaskMonitor(scheduler)
metrics = monitor.get_task_metrics(task1.id)
print(f"Success rate: {metrics.success_rate}%")
print(f"Average duration: {metrics.average_duration}s")
```

## Suggested Test Cases

### Basic Task Scheduling
1. Schedule one-time tasks
2. Schedule recurring tasks
3. Cancel scheduled tasks
4. Execute tasks immediately
5. Handle task priorities

### Schedule Types
1. One-time execution at specific time
2. Recurring intervals (hourly, daily, weekly)
3. Cron expression parsing and execution
4. Relative time scheduling
5. Invalid schedule handling

### Task Execution
1. Execute tasks at scheduled times
2. Handle task failures and retries
3. Manage task timeouts
4. Track task execution duration
5. Capture task output and errors

### Dependency Management
1. Create task dependencies
2. Execute tasks in dependency order
3. Handle dependency failures
4. Skip tasks when dependencies fail
5. Circular dependency detection

### Monitoring and Metrics
1. Track task execution statistics
2. Generate performance reports
3. Set up alerts for task failures
4. Monitor scheduler health
5. Historical data analysis

### Advanced Features
1. Task queuing with priority
2. Resource limit enforcement
3. Concurrent task execution
4. Task retry with exponential backoff
5. System recovery after failures

## TDD Steps

1. **Red**: Test basic task creation
2. **Green**: Implement Task class
3. **Red**: Test schedule time calculations
4. **Green**: Implement Schedule class
5. **Red**: Test task scheduling
6. **Green**: Implement TaskScheduler basic functionality
7. **Red**: Test one-time task execution
8. **Green**: Add time-based execution
9. **Red**: Test recurring task scheduling
10. **Green**: Implement recurring schedules
11. **Red**: Test task dependencies
12. **Green**: Implement dependency system
13. **Red**: Test cron expression scheduling
14. **Green**: Add cron expression support
15. **Red**: Test task monitoring
16. **Green**: Implement monitoring and metrics
17. **Red**: Test failure handling and retries
18. **Green**: Add error handling and retry logic
19. **Refactor**: Optimize performance and reliability

## Learning Goals

- Time-based programming and scheduling
- Cron expression parsing and evaluation
- Task queue implementation
- Dependency graph management
- System monitoring and metrics
- Error handling and retry strategies
- Concurrent programming
- Resource management

## Advanced Concepts

1. **Distributed Scheduling**: Multi-node task execution
2. **Dynamic Scheduling**: Runtime schedule modifications
3. **Resource Pools**: Manage execution resources
4. **Workflow Engine Integration**: Complex task workflows
5. **Event-Driven Tasks**: Trigger tasks based on events
6. **Machine Learning**: Predictive task scheduling
7. **Container Integration**: Execute tasks in containers

---

## 问题描述

构建任务调度系统，可以在指定时间执行任务、处理重复任务、管理依赖关系并提供监控能力。

## 需求

### Task 类
- `Task(taskId, name, command, scheduleTime, priority)` - 创建可调度任务
- `execute()` - 执行任务
- `getStatus()` - 获取当前任务状态（待处理、运行中、已完成、失败）
- `getDuration()` - 获取执行时间
- `getResult()` - 获取执行结果或错误
- `retry()` - 重试失败任务

### Schedule 类
- `Schedule(scheduleType, parameters)` - 定义任务何时运行
- `getNextExecutionTime(currentTime)` - 计算下次运行时间
- `isTimeToRun(currentTime)` - 检查任务现在是否应该运行
- 调度类型：一次、重复、CRON表达式

### TaskScheduler 类
- `scheduleTask(task, schedule)` - 调度任务执行
- `cancelTask(taskId)` - 取消调度任务
- `pauseTask(taskId)` - 暂停任务执行
- `resumeTask(taskId)` - 恢复暂停任务
- `executeNow(taskId)` - 立即执行任务
- `getScheduledTasks()` - 获取所有调度任务
- `getTaskHistory(taskId)` - 获取任务执行历史

### 依赖系统
- `TaskDependency(taskId, dependsOnTaskId, dependencyType)` - 任务依赖
- `addDependency(taskId, dependsOnTaskId)` - 添加任务依赖
- `removeDependency(taskId, dependsOnTaskId)` - 移除依赖
- `canExecute(taskId)` - 检查所有依赖是否满足
- 依赖类型：成功、完成、失败

### 监控和告警
- `TaskMonitor` - 监控任务执行和健康
- `getTaskMetrics(taskId)` - 获取任务性能指标
- `setAlert(condition, action)` - 为任务条件设置告警
- `getSystemHealth()` - 获取调度器整体健康状况
- `generateReport(timeRange)` - 生成执行报告

## 功能

### 调度选项
- 特定时间的一次性任务执行
- 重复任务（每小时、每天、每周、每月）
- 复杂调度的Cron表达式支持
- 相对调度（X分钟/小时后）
- 基于日历的调度（特定日期）

### 任务管理
- 任务优先级
- 任务队列和限流
- 带退避策略的重试机制
- 任务超时处理
- 任务取消和清理

### 高级功能
- 任务依赖链
- 条件任务执行
- 任务分组和批处理
- 资源分配和限制
- 分布式执行支持

## 示例

```python
scheduler = TaskScheduler()

# 一次性任务
task1 = Task("backup-db", "数据库备份", "/bin/backup.sh", Priority.HIGH)
once_schedule = Schedule(ScheduleType.ONCE, {"executeAt": datetime(2024, 1, 15, 2, 0)})
scheduler.schedule_task(task1, once_schedule)

# 重复任务（每天凌晨3点）
task2 = Task("daily-report", "生成日报", "/bin/report.py", Priority.MEDIUM)
daily_schedule = Schedule(ScheduleType.RECURRING,
                         {"interval": "daily", "time": "03:00"})
scheduler.schedule_task(task2, daily_schedule)

# Cron表达式任务（每周一上午9点）
task3 = Task("weekly-cleanup", "周清理", "/bin/cleanup.sh", Priority.LOW)
cron_schedule = Schedule(ScheduleType.CRON_EXPRESSION, {"expression": "0 9 * * 1"})
scheduler.schedule_task(task3, cron_schedule)

# 带依赖的任务
task4 = Task("send-report", "邮件报告", "/bin/email.py", Priority.MEDIUM)
dependency = TaskDependency(task4.id, task2.id, DependencyType.SUCCESS)
scheduler.add_dependency(dependency)
scheduler.schedule_task(task4, daily_schedule)

# 监控执行
monitor = TaskMonitor(scheduler)
metrics = monitor.get_task_metrics(task1.id)
print(f"成功率: {metrics.success_rate}%")
print(f"平均持续时间: {metrics.average_duration}s")
```

## 建议测试用例

### 基本任务调度
1. 调度一次性任务
2. 调度重复任务
3. 取消调度任务
4. 立即执行任务
5. 处理任务优先级

### 调度类型
1. 特定时间的一次性执行
2. 重复间隔（每小时、每天、每周）
3. Cron表达式解析和执行
4. 相对时间调度
5. 无效调度处理

### 任务执行
1. 在调度时间执行任务
2. 处理任务失败和重试
3. 管理任务超时
4. 跟踪任务执行持续时间
5. 捕获任务输出和错误

### 依赖管理
1. 创建任务依赖
2. 按依赖顺序执行任务
3. 处理依赖失败
4. 依赖失败时跳过任务
5. 循环依赖检测

### 监控和指标
1. 跟踪任务执行统计
2. 生成性能报告
3. 为任务失败设置告警
4. 监控调度器健康
5. 历史数据分析

### 高级功能
1. 带优先级的任务队列
2. 资源限制执行
3. 并发任务执行
4. 指数退避的任务重试
5. 故障后系统恢复

## TDD 步骤

1. **红灯**: 测试基本任务创建
2. **绿灯**: 实现任务类
3. **红灯**: 测试调度时间计算
4. **绿灯**: 实现调度类
5. **红灯**: 测试任务调度
6. **绿灯**: 实现任务调度器基本功能
7. **红灯**: 测试一次性任务执行
8. **绿灯**: 添加基于时间的执行
9. **红灯**: 测试重复任务调度
10. **绿灯**: 实现重复调度
11. **红灯**: 测试任务依赖
12. **绿灯**: 实现依赖系统
13. **红灯**: 测试cron表达式调度
14. **绿灯**: 添加cron表达式支持
15. **红灯**: 测试任务监控
16. **绿灯**: 实现监控和指标
17. **红灯**: 测试故障处理和重试
18. **绿灯**: 添加错误处理和重试逻辑
19. **重构**: 优化性能和可靠性

## 学习目标

- 基于时间的编程和调度
- Cron表达式解析和评估
- 任务队列实现
- 依赖图管理
- 系统监控和指标
- 错误处理和重试策略
- 并发编程
- 资源管理

## 高级概念

1. **分布式调度**: 多节点任务执行
2. **动态调度**: 运行时调度修改
3. **资源池**: 管理执行资源
4. **工作流引擎集成**: 复杂任务工作流
5. **事件驱动任务**: 基于事件触发任务
6. **机器学习**: 预测性任务调度
7. **容器集成**: 在容器中执行任务