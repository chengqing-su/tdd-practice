# Workflow Engine

## Problem Description

Build a flexible workflow engine that can execute complex business processes with parallel tasks, conditional branching, loops, error handling, and state persistence.

## Requirements

### WorkflowDefinition Class
- `WorkflowDefinition(id, name, startNodeId)` - Define workflow structure
- `addNode(node)` - Add workflow node (task, condition, etc.)
- `addTransition(fromNodeId, toNodeId, condition)` - Define transitions
- `validate()` - Ensure workflow is well-formed
- `getStartNode()` - Get workflow entry point
- `getNode(nodeId)` - Get specific node

### Node Types

#### TaskNode
- `TaskNode(id, name, taskHandler)` - Executable task
- `execute(context)` - Run task with workflow context
- `getInputs()` - Get required input parameters
- `getOutputs()` - Get output parameters

#### ConditionNode
- `ConditionNode(id, name, condition)` - Conditional branching
- `evaluate(context)` - Evaluate condition
- `getTrueTransition()` - Get path when condition is true
- `getFalseTransition()` - Get path when condition is false

#### ParallelNode
- `ParallelNode(id, name, branches)` - Fork execution
- `addBranch(branchNodes)` - Add parallel execution branch
- `getJoinStrategy()` - ALL, ANY, or FIRST completion

#### LoopNode
- `LoopNode(id, name, condition, body)` - Iterative execution
- `getLoopCondition()` - Condition to continue looping
- `getLoopBody()` - Nodes to execute in loop

### WorkflowEngine Class
- `WorkflowEngine(stateStore, taskRegistry)` - Initialize engine
- `startWorkflow(definitionId, inputs)` - Start new workflow instance
- `continueWorkflow(instanceId)` - Resume workflow execution
- `pauseWorkflow(instanceId)` - Pause workflow execution
- `cancelWorkflow(instanceId)` - Cancel workflow execution
- `getWorkflowStatus(instanceId)` - Get current workflow state

### WorkflowInstance Class
- `WorkflowInstance(id, definitionId, status, context)` - Running workflow
- `getCurrentNode()` - Get currently executing node
- `getContext()` - Get workflow variables and data
- `getExecutionHistory()` - Get audit trail of executed nodes
- `isCompleted()` - Check if workflow finished
- `getResult()` - Get workflow output

### Execution Context
- `ExecutionContext(variables, workflowInstance)` - Runtime context
- `setVariable(name, value)` - Set workflow variable
- `getVariable(name)` - Get workflow variable
- `hasVariable(name)` - Check if variable exists

### State Persistence
- `WorkflowStateStore` - Persist workflow instances
- `save(instance)` - Save workflow state
- `load(instanceId)` - Load workflow state
- `delete(instanceId)` - Remove workflow instance

### Task Registry
- `TaskRegistry` - Register and resolve task handlers
- `register(taskName, handler)` - Register task implementation
- `getHandler(taskName)` - Get task handler
- `unregister(taskName)` - Remove task registration

## Examples

```python
# Define workflow
workflow = WorkflowDefinition("order-process", "Order Processing", "start")

# Add nodes
start_node = TaskNode("start", "Validate Order", ValidateOrderTask())
payment_node = TaskNode("payment", "Process Payment", PaymentTask())
inventory_check = ConditionNode("inventory", "Check Stock",
                               lambda ctx: ctx.get_variable("stock") > 0)
fulfill_node = TaskNode("fulfill", "Fulfill Order", FulfillmentTask())
notify_node = TaskNode("notify", "Send Notification", NotificationTask())

workflow.add_node(start_node)
workflow.add_node(payment_node)
workflow.add_node(inventory_check)
workflow.add_node(fulfill_node)
workflow.add_node(notify_node)

# Define transitions
workflow.add_transition("start", "payment", None)
workflow.add_transition("payment", "inventory", None)
workflow.add_transition("inventory", "fulfill", lambda ctx: True)
workflow.add_transition("inventory", "notify", lambda ctx: False)
workflow.add_transition("fulfill", "notify", None)

# Execute workflow
engine = WorkflowEngine(FileStateStore(), TaskRegistry())
instance_id = engine.start_workflow("order-process",
                                   {"order_id": "12345", "amount": 100.0})

# Check status
status = engine.get_workflow_status(instance_id)
print(f"Workflow status: {status}")

# Continue execution
engine.continue_workflow(instance_id)
```

## Suggested Test Cases

### WorkflowDefinition Tests
1. Create workflow with nodes and transitions
2. Validate workflow structure (no orphaned nodes)
3. Detect cycles in workflow
4. Ensure start node exists
5. Validate transition conditions

### Node Execution Tests
1. TaskNode executes and updates context
2. ConditionNode evaluates correctly
3. ParallelNode forks execution
4. LoopNode iterates while condition is true
5. Node error handling and recovery

### WorkflowEngine Tests
1. Start workflow creates instance
2. Execute workflow step by step
3. Handle task failures and retries
4. Pause and resume workflow execution
5. Cancel running workflows
6. Persist workflow state between runs

### Parallel Execution Tests
1. Fork multiple parallel branches
2. Wait for all branches (AND join)
3. Continue when any branch completes (OR join)
4. Handle errors in parallel branches
5. Timeout on long-running parallel tasks

### Loop Execution Tests
1. Execute loop body multiple times
2. Exit loop when condition becomes false
3. Handle infinite loops with max iterations
4. Update context variables in loop
5. Break loop on errors

### State Persistence Tests
1. Save workflow state to storage
2. Load and resume from saved state
3. Handle storage failures
4. Concurrent access to same workflow
5. State consistency across restarts

### Error Handling Tests
1. Task execution failures
2. Timeout handling
3. Retry mechanisms
4. Compensation actions
5. Dead letter handling

## TDD Steps

1. **Red**: Test WorkflowDefinition creation
2. **Green**: Implement basic workflow structure
3. **Red**: Test adding nodes and transitions
4. **Green**: Implement workflow builder
5. **Red**: Test TaskNode execution
6. **Green**: Implement task execution
7. **Red**: Test workflow context management
8. **Green**: Implement execution context
9. **Refactor**: Clean up workflow core
10. **Red**: Test ConditionNode evaluation
11. **Green**: Implement conditional branching
12. **Red**: Test ParallelNode execution
13. **Green**: Implement parallel execution
14. **Red**: Test workflow state persistence
15. **Green**: Implement state store
16. **Red**: Test error handling and recovery
17. **Green**: Implement error mechanisms
18. **Red**: Test loop execution
19. **Green**: Implement loop nodes
20. **Refactor**: Optimize entire engine

## Learning Goals

- State machine design
- Complex control flow implementation
- Persistence and serialization
- Concurrent execution patterns
- Error handling strategies
- Plugin architecture (task registry)
- Event-driven programming
- Domain-specific languages (DSL)
- Business process modeling
- System recovery and resilience

## Advanced Features

- **BPMN Support**: Standard workflow notation
- **Compensation**: Undo completed tasks on failure
- **Timeouts**: Automatic task timeouts
- **Escalation**: Automatic error escalation
- **Sub-workflows**: Embed workflows within workflows
- **Dynamic Routing**: Runtime decision on next steps
- **Monitoring**: Real-time workflow execution metrics
- **Versioning**: Support multiple workflow versions

---

## 问题描述

构建一个灵活的工作流引擎，能够执行复杂的业务流程，包括并行任务、条件分支、循环、错误处理和状态持久化。

## 需求

### WorkflowDefinition 类
- `WorkflowDefinition(id, name, startNodeId)` - 定义工作流结构
- `addNode(node)` - 添加工作流节点（任务、条件等）
- `addTransition(fromNodeId, toNodeId, condition)` - 定义转换
- `validate()` - 确保工作流格式正确
- `getStartNode()` - 获取工作流入口点
- `getNode(nodeId)` - 获取特定节点

### 节点类型

#### TaskNode
- `TaskNode(id, name, taskHandler)` - 可执行任务
- `execute(context)` - 用工作流上下文运行任务
- `getInputs()` - 获取必需的输入参数
- `getOutputs()` - 获取输出参数

#### ConditionNode
- `ConditionNode(id, name, condition)` - 条件分支
- `evaluate(context)` - 评估条件
- `getTrueTransition()` - 获取条件为真时的路径
- `getFalseTransition()` - 获取条件为假时的路径

#### ParallelNode
- `ParallelNode(id, name, branches)` - 分叉执行
- `addBranch(branchNodes)` - 添加并行执行分支
- `getJoinStrategy()` - 全部、任意或首个完成

#### LoopNode
- `LoopNode(id, name, condition, body)` - 迭代执行
- `getLoopCondition()` - 继续循环的条件
- `getLoopBody()` - 循环中执行的节点

### WorkflowEngine 类
- `WorkflowEngine(stateStore, taskRegistry)` - 初始化引擎
- `startWorkflow(definitionId, inputs)` - 启动新工作流实例
- `continueWorkflow(instanceId)` - 恢复工作流执行
- `pauseWorkflow(instanceId)` - 暂停工作流执行
- `cancelWorkflow(instanceId)` - 取消工作流执行
- `getWorkflowStatus(instanceId)` - 获取当前工作流状态

### WorkflowInstance 类
- `WorkflowInstance(id, definitionId, status, context)` - 运行中的工作流
- `getCurrentNode()` - 获取当前执行节点
- `getContext()` - 获取工作流变量和数据
- `getExecutionHistory()` - 获取已执行节点的审计跟踪
- `isCompleted()` - 检查工作流是否完成
- `getResult()` - 获取工作流输出

### 执行上下文
- `ExecutionContext(variables, workflowInstance)` - 运行时上下文
- `setVariable(name, value)` - 设置工作流变量
- `getVariable(name)` - 获取工作流变量
- `hasVariable(name)` - 检查变量是否存在

### 状态持久化
- `WorkflowStateStore` - 持久化工作流实例
- `save(instance)` - 保存工作流状态
- `load(instanceId)` - 加载工作流状态
- `delete(instanceId)` - 删除工作流实例

### 任务注册表
- `TaskRegistry` - 注册和解析任务处理器
- `register(taskName, handler)` - 注册任务实现
- `getHandler(taskName)` - 获取任务处理器
- `unregister(taskName)` - 移除任务注册

## 示例

```python
# 定义工作流
workflow = WorkflowDefinition("order-process", "订单处理", "start")

# 添加节点
start_node = TaskNode("start", "验证订单", ValidateOrderTask())
payment_node = TaskNode("payment", "处理付款", PaymentTask())
inventory_check = ConditionNode("inventory", "检查库存",
                               lambda ctx: ctx.get_variable("stock") > 0)
fulfill_node = TaskNode("fulfill", "履行订单", FulfillmentTask())
notify_node = TaskNode("notify", "发送通知", NotificationTask())

workflow.add_node(start_node)
workflow.add_node(payment_node)
workflow.add_node(inventory_check)
workflow.add_node(fulfill_node)
workflow.add_node(notify_node)

# 定义转换
workflow.add_transition("start", "payment", None)
workflow.add_transition("payment", "inventory", None)
workflow.add_transition("inventory", "fulfill", lambda ctx: True)
workflow.add_transition("inventory", "notify", lambda ctx: False)
workflow.add_transition("fulfill", "notify", None)

# 执行工作流
engine = WorkflowEngine(FileStateStore(), TaskRegistry())
instance_id = engine.start_workflow("order-process",
                                   {"order_id": "12345", "amount": 100.0})

# 检查状态
status = engine.get_workflow_status(instance_id)
print(f"工作流状态: {status}")

# 继续执行
engine.continue_workflow(instance_id)
```

## 建议测试用例

### WorkflowDefinition 测试
1. 创建带节点和转换的工作流
2. 验证工作流结构（无孤立节点）
3. 检测工作流中的循环
4. 确保存在开始节点
5. 验证转换条件

### 节点执行测试
1. TaskNode执行并更新上下文
2. ConditionNode正确评估
3. ParallelNode分叉执行
4. LoopNode在条件为真时迭代
5. 节点错误处理和恢复

### WorkflowEngine 测试
1. 启动工作流创建实例
2. 逐步执行工作流
3. 处理任务失败和重试
4. 暂停和恢复工作流执行
5. 取消运行中的工作流
6. 在运行间持久化工作流状态

### 并行执行测试
1. 分叉多个并行分支
2. 等待所有分支（AND连接）
3. 任一分支完成时继续（OR连接）
4. 处理并行分支中的错误
5. 长时间运行并行任务的超时

### 循环执行测试
1. 多次执行循环体
2. 条件变为假时退出循环
3. 用最大迭代处理无限循环
4. 在循环中更新上下文变量
5. 错误时中断循环

### 状态持久化测试
1. 将工作流状态保存到存储
2. 从保存状态加载和恢复
3. 处理存储失败
4. 同一工作流的并发访问
5. 重启间的状态一致性

### 错误处理测试
1. 任务执行失败
2. 超时处理
3. 重试机制
4. 补偿动作
5. 死信处理

## TDD 步骤

1. **红灯**: 测试WorkflowDefinition创建
2. **绿灯**: 实现基本工作流结构
3. **红灯**: 测试添加节点和转换
4. **绿灯**: 实现工作流构建器
5. **红灯**: 测试TaskNode执行
6. **绿灯**: 实现任务执行
7. **红灯**: 测试工作流上下文管理
8. **绿灯**: 实现执行上下文
9. **重构**: 清理工作流核心
10. **红灯**: 测试ConditionNode评估
11. **绿灯**: 实现条件分支
12. **红灯**: 测试ParallelNode执行
13. **绿灯**: 实现并行执行
14. **红灯**: 测试工作流状态持久化
15. **绿灯**: 实现状态存储
16. **红灯**: 测试错误处理和恢复
17. **绿灯**: 实现错误机制
18. **红灯**: 测试循环执行
19. **绿灯**: 实现循环节点
20. **重构**: 优化整个引擎

## 学习目标

- 状态机设计
- 复杂控制流实现
- 持久化和序列化
- 并发执行模式
- 错误处理策略
- 插件架构（任务注册表）
- 事件驱动编程
- 领域特定语言（DSL）
- 业务流程建模
- 系统恢复和弹性

## 高级功能

- **BPMN支持**: 标准工作流记号
- **补偿**: 失败时撤销已完成任务
- **超时**: 自动任务超时
- **升级**: 自动错误升级
- **子工作流**: 在工作流中嵌入工作流
- **动态路由**: 运行时决定下一步
- **监控**: 实时工作流执行指标
- **版本管理**: 支持多个工作流版本