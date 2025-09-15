# TDD Task Breakdown Templates

Ready-to-use templates and checklists for breaking down development tasks into TDD-friendly chunks.

[中文版本](#tdd-任务分解模板)

## 📋 Quick Reference Templates

### 🎯 CRUD Entity Template
For implementing basic entity operations:

```
□ Create [Entity] with required fields
□ Validate [Entity] required field constraints
□ Validate [Entity] field formats/patterns
□ Save [Entity] to repository
□ Retrieve [Entity] by ID
□ Handle [Entity] not found scenarios
□ Update [Entity] with new data
□ Validate [Entity] update permissions
□ Delete [Entity] by ID
□ Handle [Entity] deletion constraints
□ List [Entity] with pagination
□ Search [Entity] by criteria
```

### 🌐 REST API Endpoint Template
For each API endpoint:

```
□ Accept [HTTP Method] requests at [endpoint]
□ Parse request body/parameters correctly
□ Validate required parameters present
□ Validate parameter data types
□ Validate parameter formats/patterns
□ Validate business rules for input
□ Process request and return success response
□ Return appropriate HTTP status codes
□ Handle and return validation errors (400)
□ Handle and return authentication errors (401)
□ Handle and return authorization errors (403)
□ Handle and return not found errors (404)
□ Handle and return server errors (500)
□ Add proper response headers
□ Add response timing/performance logging
```

### 🏗️ Service Layer Template
For business logic implementation:

```
□ Validate method preconditions
□ Handle null/empty inputs gracefully
□ Implement happy path logic
□ Add input parameter validation
□ Add business rule validation
□ Handle repository/data access errors
□ Add transaction management
□ Implement error recovery logic
□ Add method-level logging
□ Add performance monitoring
□ Handle concurrent access scenarios
□ Add unit tests for each scenario
```

### 🔐 Authentication/Authorization Template
For security implementation:

```
□ User can register with valid credentials
□ System validates password strength
□ System checks username/email uniqueness
□ Password is hashed before storage
□ User can login with valid credentials
□ System rejects invalid login attempts
□ System locks account after failed attempts
□ User session is created on successful login
□ User can logout and session is invalidated
□ Protected resources require authentication
□ User permissions are checked for actions
□ JWT tokens are generated and validated
□ Token refresh mechanism works
□ Password reset functionality works
```

### 📊 Data Processing Template
For complex data transformations:

```
□ Load/read input data successfully
□ Validate input data format
□ Handle empty/null input data
□ Parse data into internal format
□ Apply business rules/transformations
□ Validate transformed data
□ Handle transformation errors
□ Save/output processed data
□ Log processing statistics
□ Handle large data sets efficiently
□ Add progress tracking for long operations
□ Implement rollback for failed processing
```

### 🔄 Integration Template
For external service integration:

```
□ Establish connection to external service
□ Handle connection timeouts
□ Handle authentication with external service
□ Send request with proper format
□ Parse successful response
□ Handle service unavailable scenarios
□ Handle malformed response scenarios
□ Implement retry logic for failures
□ Add circuit breaker for reliability
□ Cache responses when appropriate
□ Add comprehensive logging
□ Monitor integration health
```

## 🎨 Domain-Specific Templates

### 💰 E-Commerce Order Template

```
Shopping Cart Management:
□ Create empty shopping cart for user
□ Add product to cart with quantity
□ Update product quantity in cart
□ Remove product from cart
□ Calculate cart subtotal
□ Apply discount codes
□ Calculate shipping costs
□ Calculate taxes based on location
□ Calculate final total

Order Processing:
□ Validate cart contents before checkout
□ Check product availability
□ Reserve inventory for order items
□ Create order from cart contents
□ Process payment with payment gateway
□ Handle payment success
□ Handle payment failures
□ Send order confirmation
□ Release inventory on payment failure

Order Management:
□ Update order status
□ Cancel order if allowed
□ Process refunds
□ Generate order reports
□ Track order fulfillment
□ Handle order modifications
```

### 👥 User Management Template

```
Registration:
□ Validate email format
□ Check email uniqueness
□ Validate password strength
□ Hash password securely
□ Create user account
□ Send welcome email
□ Handle registration errors

Profile Management:
□ Display user profile
□ Update profile information
□ Validate profile updates
□ Handle profile image upload
□ Manage user preferences
□ Handle account deactivation

Authentication:
□ Implement login functionality
□ Handle password reset requests
□ Send password reset emails
□ Validate reset tokens
□ Update forgotten passwords
□ Implement two-factor authentication
□ Handle account lockouts
```

### 📝 Content Management Template

```
Content Creation:
□ Create new content item
□ Validate content data
□ Handle content media uploads
□ Generate content slug/URL
□ Set content metadata
□ Save draft content
□ Publish content
□ Schedule content publication

Content Management:
□ List content with pagination
□ Search content by criteria
□ Edit existing content
□ Version content changes
□ Delete content (soft delete)
□ Handle content permissions
□ Moderate user content
□ Handle content reports

Content Display:
□ Display published content
□ Handle content not found
□ Add content view tracking
□ Implement content caching
□ Handle content SEO metadata
□ Add social sharing features
```

## ⚡ Quick Task Assessment

### Task Size Evaluation Checklist

**Perfect Size (✅ Ready for TDD)**
- [ ] Can write a failing test in under 5 minutes
- [ ] Can implement in 15-45 minutes
- [ ] Tests single behavior/outcome
- [ ] Independent of other pending tasks
- [ ] Provides demonstrable value
- [ ] Clear success criteria

**Too Large (⚠️ Needs Further Breakdown)**
- [ ] Takes more than 1 hour to implement
- [ ] Requires multiple test assertions
- [ ] Involves multiple components/layers
- [ ] Can't explain in one sentence
- [ ] Has ambiguous success criteria
- [ ] Depends on incomplete work

**Too Small (🔬 Consider Combining)**
- [ ] Only a getter/setter method
- [ ] No meaningful behavior to test
- [ ] Trivial implementation
- [ ] No business/user value
- [ ] Just configuration/setup

## 🎯 Task Prioritization Framework

### MoSCoW Method for TDD Tasks
**Must Have (Critical Path)**
- Core functionality that blocks other features
- Basic happy path scenarios
- Essential validations

**Should Have (Important)**
- Error handling scenarios
- Edge cases
- Performance optimizations

**Could Have (Nice to Have)**
- Advanced features
- UI/UX enhancements
- Convenience methods

**Won't Have (This Iteration)**
- Future features
- Optimization without metrics
- Gold-plating features

### Value-Complexity Matrix
```
High Value, Low Complexity  → Do First
High Value, High Complexity → Break Down Further
Low Value, Low Complexity   → Do Later
Low Value, High Complexity  → Avoid/Postpone
```

## 📝 Task Writing Best Practices

### Good Task Naming Patterns
```
✅ User can [action] [object] [condition]
   "User can view order history when logged in"

✅ System [validates/processes/handles] [scenario]
   "System validates email format during registration"

✅ [Component] [returns/calculates/generates] [result]
   "Calculator returns correct result for division by zero"

✅ Handle [error scenario] gracefully
   "Handle network timeout during payment processing"
```

### Poor Task Naming Patterns
```
❌ Implement [technical concept]
   "Implement Repository pattern"

❌ Add [vague functionality]
   "Add error handling"

❌ Build [large feature]
   "Build user management system"

❌ Fix [broad problem]
   "Fix authentication issues"
```

## 🔄 Iterative Refinement Process

### Step 1: Initial Breakdown
1. Start with user story or feature requirement
2. List all major behaviors/outcomes needed
3. Group related behaviors together
4. Prioritize by dependency and value

### Step 2: Task Refinement
1. Review each task against size guidelines
2. Split tasks that are too large
3. Combine tasks that are too small
4. Ensure clear acceptance criteria

### Step 3: Dependency Mapping
1. Identify which tasks depend on others
2. Reorder to minimize dependencies
3. Consider parallel development opportunities
4. Plan integration points

### Step 4: Estimation and Planning
1. Estimate effort for each task
2. Identify risks and unknowns
3. Plan for learning spikes if needed
4. Buffer time for refactoring

## 🧪 Testing Strategy per Task Type

### Data Validation Tasks
```
Test Categories:
- Valid input acceptance
- Invalid input rejection
- Boundary value testing
- Format validation
- Business rule validation
```

### Business Logic Tasks
```
Test Categories:
- Happy path scenarios
- Edge cases and boundaries
- Error conditions
- State transitions
- Calculation accuracy
```

### Integration Tasks
```
Test Categories:
- Successful integration
- Service unavailable
- Timeout scenarios
- Authentication failures
- Data format mismatches
```

### UI/Controller Tasks
```
Test Categories:
- Request/response handling
- Status code accuracy
- Error message clarity
- Input validation
- Security checks
```

## 📚 Documentation Templates

### Task Definition Template
```
## Task: [Clear, Action-Oriented Title]

### Description
Brief description of what needs to be implemented.

### Acceptance Criteria
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

### Test Scenarios
- Happy path: [description]
- Edge case: [description]
- Error case: [description]

### Definition of Done
- [ ] Tests written and passing
- [ ] Code reviewed
- [ ] Documentation updated
- [ ] Functionality demonstrated

### Dependencies
- Requires: [other tasks]
- Enables: [future tasks]

### Estimates
- Development: [time]
- Testing: [time]
- Review: [time]
```

### Sprint Planning Template
```
## Sprint Goal
[What we want to achieve this sprint]

## User Stories
### Story 1: [Title]
**Priority**: High/Medium/Low
**Estimate**: [story points]
**Tasks**:
- [ ] Task 1
- [ ] Task 2
- [ ] Task 3

### Story 2: [Title]
[Repeat format]

## Definition of Ready
- [ ] Tasks are well-defined
- [ ] Acceptance criteria are clear
- [ ] Dependencies are identified
- [ ] Estimates are agreed upon

## Definition of Done
- [ ] All tasks completed
- [ ] Tests passing
- [ ] Code reviewed
- [ ] Documentation updated
- [ ] Deployed to staging
```

---

# TDD 任务分解模板

用于将开发任务分解为TDD友好块的即用模板和清单。

## 📋 快速参考模板

### 🎯 CRUD实体模板
用于实现基本实体操作：

```
□ 创建带必需字段的[实体]
□ 验证[实体]必需字段约束
□ 验证[实体]字段格式/模式
□ 将[实体]保存到仓库
□ 根据ID检索[实体]
□ 处理[实体]未找到场景
□ 用新数据更新[实体]
□ 验证[实体]更新权限
□ 根据ID删除[实体]
□ 处理[实体]删除约束
□ 用分页列出[实体]
□ 根据条件搜索[实体]
```

### 🌐 REST API端点模板
对于每个API端点：

```
□ 在[端点]接受[HTTP方法]请求
□ 正确解析请求体/参数
□ 验证必需参数存在
□ 验证参数数据类型
□ 验证参数格式/模式
□ 验证输入的业务规则
□ 处理请求并返回成功响应
□ 返回适当的HTTP状态码
□ 处理并返回验证错误(400)
□ 处理并返回认证错误(401)
□ 处理并返回授权错误(403)
□ 处理并返回未找到错误(404)
□ 处理并返回服务器错误(500)
□ 添加适当的响应头
□ 添加响应时间/性能日志
```

### 🏗️ 服务层模板
用于业务逻辑实现：

```
□ 验证方法前置条件
□ 优雅处理null/空输入
□ 实现快乐路径逻辑
□ 添加输入参数验证
□ 添加业务规则验证
□ 处理仓库/数据访问错误
□ 添加事务管理
□ 实现错误恢复逻辑
□ 添加方法级日志记录
□ 添加性能监控
□ 处理并发访问场景
□ 为每个场景添加单元测试
```

## 🎨 领域特定模板

### 💰 电商订单模板

```
购物车管理：
□ 为用户创建空购物车
□ 向购物车添加带数量的产品
□ 更新购物车中的产品数量
□ 从购物车移除产品
□ 计算购物车小计
□ 应用折扣码
□ 计算运费
□ 根据位置计算税费
□ 计算最终总额

订单处理：
□ 结账前验证购物车内容
□ 检查产品可用性
□ 为订单项目预留库存
□ 从购物车内容创建订单
□ 用支付网关处理付款
□ 处理付款成功
□ 处理付款失败
□ 发送订单确认
□ 付款失败时释放库存

订单管理：
□ 更新订单状态
□ 如允许则取消订单
□ 处理退款
□ 生成订单报告
□ 跟踪订单履行
□ 处理订单修改
```

## ⚡ 快速任务评估

### 任务大小评估清单

**完美大小（✅ 准备好TDD）**
- [ ] 可以在5分钟内写出失败测试
- [ ] 可以在15-45分钟内实现
- [ ] 测试单个行为/结果
- [ ] 独立于其他待定任务
- [ ] 提供可演示的价值
- [ ] 明确的成功标准

**太大（⚠️ 需要进一步分解）**
- [ ] 实现需要超过1小时
- [ ] 需要多个测试断言
- [ ] 涉及多个组件/层
- [ ] 不能用一句话解释
- [ ] 成功标准模糊
- [ ] 依赖未完成的工作

**太小（🔬 考虑组合）**
- [ ] 只是getter/setter方法
- [ ] 没有有意义的行为要测试
- [ ] 实现微不足道
- [ ] 没有业务/用户价值
- [ ] 只是配置/设置

## 🎯 任务优先级框架

### TDD任务的MoSCoW方法
**必须有（关键路径）**
- 阻塞其他功能的核心功能
- 基本快乐路径场景
- 基本验证

**应该有（重要）**
- 错误处理场景
- 边界情况
- 性能优化

**可以有（锦上添花）**
- 高级功能
- UI/UX增强
- 便利方法

**不会有（此迭代）**
- 未来功能
- 没有指标的优化
- 镀金功能