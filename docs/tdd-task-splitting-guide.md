# TDD Task Splitting Guide

A comprehensive guide to breaking down development tasks to align with Test-Driven Development principles and maximize the effectiveness of the Red-Green-Refactor cycle.

[中文版本](#tdd-任务拆分指南)

## 🎯 Why Task Splitting Matters for TDD

TDD thrives on small, focused iterations. Large, complex tasks violate TDD principles by:
- Making it difficult to write simple, failing tests
- Requiring complex implementations that can't be built incrementally
- Leading to long Red phases that increase debugging difficulty
- Reducing the frequency of the rewarding Green phase

## 🔍 Core Principles of TDD Task Splitting

### 1. **One Behavior, One Test**
Each task should focus on implementing one specific behavior that can be validated by a single, focused test.

### 2. **Minimal Implementation Steps**
Tasks should be small enough that the simplest possible implementation can make the test pass.

### 3. **Independent and Testable**
Each task should be independently testable without requiring complex setup or multiple components.

### 4. **Value-Driven Progression**
Tasks should build upon each other, delivering incremental value while maintaining a working system.

## 📋 Task Splitting Strategies

### Strategy 1: **Behavior-Driven Decomposition**

Break down features by specific behaviors or use cases.

#### ❌ Too Large:
```
Task: Implement user authentication system
```

#### ✅ Well-Split:
```
1. User can register with valid email and password
2. System rejects registration with invalid email format
3. System rejects registration with weak password
4. User can login with valid credentials
5. System rejects login with invalid credentials
6. System locks account after 5 failed attempts
7. User can request password reset
```

### Strategy 2: **Data Flow Decomposition**

Split tasks based on data transformations and validations.

#### ❌ Too Large:
```
Task: Process payment for order
```

#### ✅ Well-Split:
```
1. Validate payment amount is positive
2. Validate credit card number format
3. Validate credit card expiry date
4. Calculate tax based on location
5. Apply discount codes if provided
6. Process payment with external gateway
7. Handle payment success response
8. Handle payment failure response
```

### Strategy 3: **Layer-by-Layer Decomposition**

Break down by architectural layers, but maintain end-to-end functionality.

#### ❌ Too Large:
```
Task: Build product catalog system
```

#### ✅ Well-Split:
```
1. Product entity can be created with basic properties
2. Product repository can save and retrieve products
3. Product service validates product data before saving
4. Product controller accepts POST requests for new products
5. Product controller returns proper HTTP status codes
6. Product controller handles validation errors gracefully
7. Products can be retrieved by category
8. Products can be searched by name
```

### Strategy 4: **Happy Path First, Then Edge Cases**

Start with the main success scenario, then add error handling.

#### ❌ Mixed Approach:
```
Task: User login with error handling and edge cases
```

#### ✅ Well-Split:
```
1. User can login successfully with valid credentials
2. System shows error message for invalid username
3. System shows error message for invalid password
4. System handles empty username input
5. System handles empty password input
6. System prevents SQL injection in login form
7. System logs failed login attempts
```

## 🏗️ Task Breakdown Templates

### Template 1: CRUD Operations

For each entity, split into these tasks:

```
1. Create [Entity] with required fields
2. Validate [Entity] required field constraints
3. Save [Entity] to repository
4. Retrieve [Entity] by ID
5. Handle [Entity] not found scenarios
6. Update [Entity] with new data
7. Validate [Entity] update permissions
8. Delete [Entity] by ID
9. Handle [Entity] deletion constraints
```

### Template 2: API Endpoints

For each endpoint, split into these tasks:

```
1. Accept [HTTP Method] requests at [endpoint]
2. Parse request body/parameters correctly
3. Validate input data format
4. Validate business rules for input
5. Process request and return success response
6. Return appropriate HTTP status codes
7. Handle and return validation errors
8. Handle and return server errors
9. Add proper response headers
```

### Template 3: Business Processes

For complex business logic, split into these tasks:

```
1. Validate preconditions for [process]
2. Execute step 1 of [process]
3. Execute step 2 of [process]
4. ... (continue for each step)
5. Handle failure at step [X]
6. Implement rollback for step [Y]
7. Generate success confirmation
8. Log process completion
```

## 💡 Practical Examples

### Example 1: Shopping Cart Feature

#### Original Large Task:
"Implement shopping cart functionality with add/remove items, quantity updates, and checkout"

#### TDD-Friendly Breakdown:
```
1. Create empty shopping cart for user
2. Add single item to empty cart
3. Add same item twice (should increase quantity)
4. Add different items to cart
5. Calculate total price for items in cart
6. Remove item from cart completely
7. Update item quantity in cart
8. Handle removing non-existent item
9. Clear entire cart
10. Validate item availability before adding
11. Apply discount code to cart total
12. Calculate tax based on delivery address
13. Process checkout with payment
```

### Example 2: Email Notification System

#### Original Large Task:
"Build email notification system for various user actions"

#### TDD-Friendly Breakdown:
```
1. Send welcome email on user registration
2. Validate email address format before sending
3. Handle email delivery failure gracefully
4. Send password reset email with secure token
5. Send order confirmation email with details
6. Send shipping notification with tracking info
7. Queue emails for bulk sending
8. Retry failed email deliveries
9. Track email delivery status
10. Prevent duplicate email sends
```

### Example 3: File Upload Feature

#### Original Large Task:
"Implement file upload with validation and processing"

#### TDD-Friendly Breakdown:
```
1. Accept file upload via HTTP POST
2. Validate file is not empty
3. Validate file size within limits
4. Validate file type is allowed
5. Generate unique filename for storage
6. Save file to designated directory
7. Store file metadata in database
8. Handle file upload failures
9. Generate download URL for uploaded file
10. Implement file deletion functionality
```

## ⚡ TDD Task Splitting Anti-Patterns

### Anti-Pattern 1: **The "Big Bang" Task**
```
❌ BAD: "Implement complete user management system"
✅ GOOD: Start with "User can be created with username and email"
```

### Anti-Pattern 2: **The "Infrastructure First" Task**
```
❌ BAD: "Set up database, caching, and logging infrastructure"
✅ GOOD: "User data can be persisted and retrieved"
```

### Anti-Pattern 3: **The "Perfect World" Task**
```
❌ BAD: "Implement feature with full error handling and edge cases"
✅ GOOD: Start with happy path, then add error cases one by one
```

### Anti-Pattern 4: **The "Technical Jargon" Task**
```
❌ BAD: "Implement Repository pattern with Unit of Work"
✅ GOOD: "Products can be saved and retrieved from storage"
```

### Anti-Pattern 5: **The "Everything Connected" Task**
```
❌ BAD: "Build product catalog with shopping cart integration"
✅ GOOD: Split into separate product and cart features
```

## 🎯 Task Sizing Guidelines

### Perfect Size Indicators:
- ✅ Can write a failing test in 2-5 minutes
- ✅ Can implement solution in 10-30 minutes
- ✅ Implementation is 5-50 lines of code
- ✅ Test describes single behavior clearly
- ✅ Can demo working functionality after task

### Too Large Indicators:
- ❌ Struggling to write a focused test
- ❌ Implementation takes over 1 hour
- ❌ Need multiple assertions in one test
- ❌ Can't explain task in one sentence
- ❌ Requires touching multiple files/components

### Too Small Indicators:
- ❌ Task is just a method signature
- ❌ No meaningful behavior to test
- ❌ Implementation is trivial (getters/setters)
- ❌ Doesn't provide user or business value

## 🔄 The TDD Task Cycle

### Phase 1: Task Planning
1. **Identify the Feature**: What needs to be built?
2. **List Behaviors**: What should it do?
3. **Prioritize**: Which behavior is most important?
4. **Split**: Break into testable chunks
5. **Validate**: Can each chunk be tested independently?

### Phase 2: Task Execution
1. **Red**: Write failing test for first task
2. **Green**: Implement minimal solution
3. **Refactor**: Clean up code
4. **Repeat**: Move to next task

### Phase 3: Task Review
1. **Assess**: Did task deliver expected value?
2. **Learn**: What would you split differently?
3. **Adjust**: Refine remaining tasks based on learnings

## 📝 Task Splitting Checklist

Before starting any task, verify:

- [ ] Task focuses on single behavior or outcome
- [ ] Can write meaningful test that will fail initially
- [ ] Can implement in 30 minutes or less
- [ ] Task is independent of other pending tasks
- [ ] Task provides some user or business value
- [ ] Task description is clear and unambiguous
- [ ] Success criteria are well-defined
- [ ] Edge cases are handled in separate tasks

## 🛠️ Tools and Techniques

### User Story Mapping
Break epics into stories, then stories into tasks:
```
Epic: User Account Management
  Story: User Registration
    Task 1: Validate email format
    Task 2: Check email uniqueness
    Task 3: Hash password securely
    Task 4: Save user to database
```

### Acceptance Criteria Decomposition
Each acceptance criterion becomes a task:
```
Story: User Login
Acceptance Criteria:
- User enters valid credentials → Task 1
- System authenticates user → Task 2
- User is redirected to dashboard → Task 3
- Invalid credentials show error → Task 4
```

### Behavior-Driven Development (BDD)
Use Given-When-Then to identify tasks:
```
Given a user with valid account
When they enter correct credentials
Then they should be logged in

→ Task: "User can login with valid credentials"
```

## 🔧 Task Management Best Practices

### 1. **Use Clear Naming Conventions**
- Start with action verbs: "Validate", "Create", "Handle"
- Include the actor: "User can...", "System should..."
- Be specific: "Login with email" vs. "Login functionality"

### 2. **Maintain Task Dependencies**
- Complete foundational tasks first
- Don't start tasks that depend on incomplete work
- Mark dependencies explicitly

### 3. **Track Progress Granularly**
- Update task status frequently
- Celebrate small wins
- Learn from task estimation misses

### 4. **Document Decisions**
- Record why tasks were split certain ways
- Note any assumptions made
- Share learnings with team

## 🎓 Advanced Task Splitting Techniques

### Outside-In Development
Start with user-facing behavior, work inward:
```
1. API returns 200 for valid request
2. Controller calls service correctly
3. Service validates business rules
4. Repository saves data properly
```

### Walking Skeleton
Build end-to-end flow first, then flesh out:
```
1. Request reaches controller (returns mock data)
2. Controller calls service (returns mock data)
3. Service calls repository (returns mock data)
4. Replace mocks with real implementations
```

### Spike and Stabilize
For uncertain areas:
```
1. Create learning spike (time-boxed research)
2. Identify tasks based on spike findings
3. Implement with proper TDD approach
```

---

# TDD 任务拆分指南

帮助开发者将开发任务分解以符合测试驱动开发原则并最大化红绿重构循环效果的综合指南。

## 🎯 为什么任务拆分对TDD很重要

TDD在小而专注的迭代中蓬勃发展。大而复杂的任务违反了TDD原则：
- 难以编写简单的失败测试
- 需要无法增量构建的复杂实现
- 导致长时间的红灯阶段，增加调试难度
- 减少了令人满意的绿灯阶段的频率

## 🔍 TDD任务拆分的核心原则

### 1. **一个行为，一个测试**
每个任务都应专注于实现一个可以通过单个专注测试验证的特定行为。

### 2. **最小实现步骤**
任务应该足够小，以便最简单的实现就能让测试通过。

### 3. **独立且可测试**
每个任务都应该可以独立测试，无需复杂设置或多个组件。

### 4. **价值驱动的进展**
任务应该相互构建，在维护工作系统的同时提供增量价值。

## 📋 任务拆分策略

### 策略1：**行为驱动分解**

根据特定行为或用例分解功能。

#### ❌ 太大：
```
任务：实现用户认证系统
```

#### ✅ 良好拆分：
```
1. 用户可以用有效邮箱和密码注册
2. 系统拒绝无效邮箱格式的注册
3. 系统拒绝弱密码的注册
4. 用户可以用有效凭据登录
5. 系统拒绝无效凭据的登录
6. 5次失败尝试后系统锁定账户
7. 用户可以请求密码重置
```

### 策略2：**数据流分解**

根据数据转换和验证拆分任务。

#### ❌ 太大：
```
任务：处理订单付款
```

#### ✅ 良好拆分：
```
1. 验证付款金额为正数
2. 验证信用卡号码格式
3. 验证信用卡到期日期
4. 根据位置计算税费
5. 如提供则应用折扣码
6. 用外部网关处理付款
7. 处理付款成功响应
8. 处理付款失败响应
```

## 💡 实际例子

### 例子1：购物车功能

#### 原始大任务：
"实现购物车功能，包括添加/删除商品、数量更新和结账"

#### TDD友好的分解：
```
1. 为用户创建空购物车
2. 向空购物车添加单个商品
3. 两次添加同一商品（应增加数量）
4. 向购物车添加不同商品
5. 计算购物车中商品的总价
6. 从购物车完全删除商品
7. 更新购物车中的商品数量
8. 处理删除不存在的商品
9. 清空整个购物车
10. 添加前验证商品可用性
11. 对购物车总价应用折扣码
12. 根据送货地址计算税费
13. 用付款处理结账
```

## ⚡ TDD任务拆分反模式

### 反模式1：**"大爆炸"任务**
```
❌ 不好：实现完整的用户管理系统
✅ 好：从"用户可以用用户名和邮箱创建"开始
```

### 反模式2：**"基础设施优先"任务**
```
❌ 不好：设置数据库、缓存和日志记录基础设施
✅ 好：用户数据可以被持久化和检索
```

## 🎯 任务大小指南

### 完美大小指标：
- ✅ 可以在2-5分钟内写出失败测试
- ✅ 可以在10-30分钟内实现解决方案
- ✅ 实现是5-50行代码
- ✅ 测试清晰描述单个行为
- ✅ 任务完成后可以演示工作功能

### 太大指标：
- ❌ 难以写出专注的测试
- ❌ 实现需要超过1小时
- ❌ 一个测试需要多个断言
- ❌ 不能用一句话解释任务
- ❌ 需要触及多个文件/组件

## 🛠️ 工具和技术

### 用户故事映射
将史诗分解为故事，然后故事分解为任务：
```
史诗：用户账户管理
  故事：用户注册
    任务1：验证邮箱格式
    任务2：检查邮箱唯一性
    任务3：安全地哈希密码
    任务4：将用户保存到数据库
```

## 🎓 高级任务拆分技术

### 由外向内开发
从面向用户的行为开始，向内工作：
```
1. API对有效请求返回200
2. 控制器正确调用服务
3. 服务验证业务规则
4. 仓库正确保存数据
```

### 行走骨架
首先构建端到端流程，然后充实：
```
1. 请求到达控制器（返回模拟数据）
2. 控制器调用服务（返回模拟数据）
3. 服务调用仓库（返回模拟数据）
4. 用真实实现替换模拟
```