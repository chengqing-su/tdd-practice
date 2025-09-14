# Todo List Manager

## Problem Description

Create a todo list management system that allows users to create, manage, and organize tasks with different priorities, due dates, and categories.

## Requirements

### Task Class
- `Task(title, description, dueDate, priority, category)` - Create new task
- `markCompleted()` - Mark task as completed
- `markIncomplete()` - Mark task as incomplete
- `isOverdue()` - Check if task is past due date
- `updatePriority(newPriority)` - Change task priority

### TodoList Class
- `addTask(task)` - Add task to the list
- `removeTask(taskId)` - Remove task by ID
- `getTask(taskId)` - Get specific task by ID
- `getAllTasks()` - Get all tasks
- `getTasksByCategory(category)` - Filter tasks by category
- `getTasksByPriority(priority)` - Filter tasks by priority
- `getCompletedTasks()` - Get all completed tasks
- `getPendingTasks()` - Get all incomplete tasks
- `getOverdueTasks()` - Get all overdue tasks

### Task Properties
- **Priority**: HIGH, MEDIUM, LOW
- **Status**: COMPLETED, PENDING
- **Due Date**: Date object
- **Category**: String (work, personal, shopping, etc.)

## Examples

```python
todo_list = TodoList()
task1 = Task("Buy groceries", "Milk, bread, eggs", date(2024, 1, 15), Priority.HIGH, "shopping")
task2 = Task("Finish report", "Q4 sales report", date(2024, 1, 10), Priority.MEDIUM, "work")

todo_list.add_task(task1)
todo_list.add_task(task2)

# Filter tasks
work_tasks = todo_list.get_tasks_by_category("work")
high_priority = todo_list.get_tasks_by_priority(Priority.HIGH)

# Mark completed
task1.mark_completed()
completed = todo_list.get_completed_tasks()

# Check overdue
overdue_tasks = todo_list.get_overdue_tasks()
```

## Suggested Test Cases

### Task Tests
1. Create task with all properties
2. Mark task as completed/incomplete
3. Check overdue status for past due dates
4. Check not overdue for future dates
5. Update task priority
6. Task equality and comparison

### TodoList Tests
1. Add and remove tasks
2. Retrieve task by ID
3. Filter by category (case insensitive)
4. Filter by priority
5. Get completed vs pending tasks
6. Get overdue tasks
7. Handle empty lists gracefully
8. Handle invalid task IDs

### Integration Tests
1. Add multiple tasks, filter, and verify counts
2. Mark tasks complete and verify filtering
3. Update task properties and verify filters update

## TDD Steps

1. **Red**: Test Task creation with properties
2. **Green**: Implement Task constructor
3. **Red**: Test task completion status
4. **Green**: Implement completion methods
5. **Red**: Test overdue functionality
6. **Green**: Implement overdue logic
7. **Refactor**: Clean up Task class
8. **Red**: Test TodoList add/remove tasks
9. **Green**: Implement basic list operations
10. **Red**: Test filtering by category
11. **Green**: Implement category filtering
12. **Repeat**: Continue with other filtering methods

## Learning Goals

- Complex object relationships
- Enumeration/constants usage
- Date/time handling
- Filtering and searching algorithms
- State management across objects
- Test organization for complex systems

---

## 问题描述

创建一个待办事项管理系统，允许用户创建、管理和组织具有不同优先级、截止日期和分类的任务。

## 需求

### Task 类
- `Task(title, description, dueDate, priority, category)` - 创建新任务
- `markCompleted()` - 标记任务为已完成
- `markIncomplete()` - 标记任务为未完成
- `isOverdue()` - 检查任务是否已过期
- `updatePriority(newPriority)` - 更改任务优先级

### TodoList 类
- `addTask(task)` - 添加任务到列表
- `removeTask(taskId)` - 通过ID移除任务
- `getTask(taskId)` - 通过ID获取特定任务
- `getAllTasks()` - 获取所有任务
- `getTasksByCategory(category)` - 按分类筛选任务
- `getTasksByPriority(priority)` - 按优先级筛选任务
- `getCompletedTasks()` - 获取所有已完成任务
- `getPendingTasks()` - 获取所有未完成任务
- `getOverdueTasks()` - 获取所有逾期任务

### 任务属性
- **优先级**: 高、中、低
- **状态**: 已完成、待处理
- **截止日期**: 日期对象
- **分类**: 字符串（工作、个人、购物等）

## 示例

```python
todo_list = TodoList()
task1 = Task("买杂货", "牛奶、面包、鸡蛋", date(2024, 1, 15), Priority.HIGH, "购物")
task2 = Task("完成报告", "Q4销售报告", date(2024, 1, 10), Priority.MEDIUM, "工作")

todo_list.add_task(task1)
todo_list.add_task(task2)

# 筛选任务
work_tasks = todo_list.get_tasks_by_category("工作")
high_priority = todo_list.get_tasks_by_priority(Priority.HIGH)

# 标记完成
task1.mark_completed()
completed = todo_list.get_completed_tasks()

# 检查逾期
overdue_tasks = todo_list.get_overdue_tasks()
```

## 建议测试用例

### Task 测试
1. 创建包含所有属性的任务
2. 标记任务为已完成/未完成
3. 检查过期日期的逾期状态
4. 检查未来日期的非逾期状态
5. 更新任务优先级
6. 任务相等性和比较

### TodoList 测试
1. 添加和移除任务
2. 通过ID检索任务
3. 按分类筛选（不区分大小写）
4. 按优先级筛选
5. 获取已完成与待处理任务
6. 获取逾期任务
7. 优雅处理空列表
8. 处理无效任务ID

### 集成测试
1. 添加多个任务、筛选并验证计数
2. 标记任务完成并验证筛选
3. 更新任务属性并验证筛选更新

## TDD 步骤

1. **红灯**: 测试创建带属性的任务
2. **绿灯**: 实现任务构造函数
3. **红灯**: 测试任务完成状态
4. **绿灯**: 实现完成方法
5. **红灯**: 测试逾期功能
6. **绿灯**: 实现逾期逻辑
7. **重构**: 清理任务类
8. **红灯**: 测试TodoList添加/移除任务
9. **绿灯**: 实现基本列表操作
10. **红灯**: 测试按分类筛选
11. **绿灯**: 实现分类筛选
12. **重复**: 继续其他筛选方法

## 学习目标

- 复杂对象关系
- 枚举/常量使用
- 日期/时间处理
- 筛选和搜索算法
- 跨对象状态管理
- 复杂系统的测试组织