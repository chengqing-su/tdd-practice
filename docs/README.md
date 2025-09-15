# TDD Documentation

This directory contains comprehensive guides and resources for mastering Test-Driven Development (TDD) principles and practices.

[中文版本](#tdd-文档)

## 📚 Available Resources

### Core TDD Guides

- **[TDD Guide](tdd-guide.md)** - Complete beginner's guide to Test-Driven Development
  - Understanding the Red-Green-Refactor cycle
  - Writing your first TDD tests
  - Common patterns and best practices
  - Practical examples and exercises

### Task Management for TDD

- **[TDD Task Splitting Guide](tdd-task-splitting-guide.md)** - Comprehensive guide to breaking down development tasks to align with TDD principles
  - Core principles of task splitting
  - Behavior-driven decomposition strategies
  - Layer-by-layer breakdown techniques
  - Practical examples and anti-patterns
  - Task sizing guidelines and assessment

- **[Task Breakdown Templates](task-breakdown-templates.md)** - Ready-to-use templates and checklists for TDD-friendly task decomposition
  - Quick reference templates for common patterns
  - CRUD operations template
  - REST API endpoint template
  - Business logic implementation template
  - Domain-specific templates (e-commerce, user management, etc.)
  - Task assessment and prioritization frameworks

## 🎯 How to Use These Resources

### For Beginners
1. Start with the **[TDD Guide](tdd-guide.md)** to understand fundamental concepts
2. Practice with [easy problems](../easy/) from the main repository
3. Use **[Task Breakdown Templates](task-breakdown-templates.md)** to split larger problems into TDD-friendly chunks

### For Intermediate Developers
1. Review **[TDD Task Splitting Guide](tdd-task-splitting-guide.md)** for advanced decomposition strategies
2. Apply templates from **[Task Breakdown Templates](task-breakdown-templates.md)** to real projects
3. Practice with [medium](../medium/) and [Spring Boot beginner](../spring-boot/beginner/) problems

### For Advanced Practitioners
1. Use the advanced techniques in **[TDD Task Splitting Guide](tdd-task-splitting-guide.md)**
2. Adapt templates for your specific domain needs
3. Challenge yourself with [hard problems](../hard/) and [Spring Boot advanced](../spring-boot/advanced/) scenarios

## 🛠️ Quick Reference

### Task Splitting Checklist
Before starting any task, verify:
- [ ] Task focuses on single behavior or outcome
- [ ] Can write meaningful test that will fail initially
- [ ] Can implement in 30 minutes or less
- [ ] Task is independent of other pending tasks
- [ ] Task provides some user or business value
- [ ] Task description is clear and unambiguous

### Common Templates
- **CRUD Entity**: Create → Validate → Save → Retrieve → Update → Delete
- **REST API**: Accept Request → Parse → Validate → Process → Return Response → Handle Errors
- **Business Process**: Validate Preconditions → Execute Steps → Handle Failures → Confirm Success

## 📖 Learning Path

### Phase 1: Foundation
1. Read [TDD Guide](tdd-guide.md)
2. Complete 3-5 [easy problems](../easy/)
3. Practice task splitting with simple algorithms

### Phase 2: Application
1. Study [TDD Task Splitting Guide](tdd-task-splitting-guide.md)
2. Complete [medium problems](../medium/) using proper task breakdown
3. Try [Spring Boot beginner problems](../spring-boot/beginner/)

### Phase 3: Mastery
1. Apply advanced splitting techniques to real projects
2. Create custom templates for your domain
3. Tackle [hard problems](../hard/) and [Spring Boot advanced scenarios](../spring-boot/advanced/)

## 🤝 Contributing

If you'd like to contribute to these documentation resources:
1. Follow the existing format and structure
2. Provide practical examples
3. Include both English and Chinese versions
4. Test your examples with real TDD practice

---

# TDD 文档

这个目录包含掌握测试驱动开发（TDD）原则和实践的综合指南和资源。

## 📚 可用资源

### 核心 TDD 指南

- **[TDD 指南](tdd-guide.md)** - 测试驱动开发的完整初学者指南
  - 理解红绿重构循环
  - 编写你的第一个TDD测试
  - 常见模式和最佳实践
  - 实际例子和练习

### TDD 任务管理

- **[TDD 任务拆分指南](tdd-task-splitting-guide.md)** - 将开发任务分解以符合TDD原则的综合指南
  - 任务拆分的核心原则
  - 行为驱动分解策略
  - 逐层分解技术
  - 实际例子和反模式
  - 任务大小指南和评估

- **[任务分解模板](task-breakdown-templates.md)** - 用于TDD友好任务分解的即用模板和清单
  - 常见模式的快速参考模板
  - CRUD操作模板
  - REST API端点模板
  - 业务逻辑实现模板
  - 领域特定模板（电商、用户管理等）
  - 任务评估和优先级框架

## 🎯 如何使用这些资源

### 初学者
1. 从**[TDD指南](tdd-guide.md)**开始理解基本概念
2. 从主仓库的[简单问题](../easy/)开始练习
3. 使用**[任务分解模板](task-breakdown-templates.md)**将大问题拆分为TDD友好的块

### 中级开发者
1. 查看**[TDD任务拆分指南](tdd-task-splitting-guide.md)**学习高级分解策略
2. 将**[任务分解模板](task-breakdown-templates.md)**的模板应用到实际项目
3. 练习[中等](../medium/)和[Spring Boot初级](../spring-boot/beginner/)问题

### 高级实践者
1. 使用**[TDD任务拆分指南](tdd-task-splitting-guide.md)**中的高级技术
2. 为你的特定领域需求调整模板
3. 挑战[困难问题](../hard/)和[Spring Boot高级](../spring-boot/advanced/)场景

## 📖 学习路径

### 第一阶段：基础
1. 阅读[TDD指南](tdd-guide.md)
2. 完成3-5个[简单问题](../easy/)
3. 练习简单算法的任务拆分

### 第二阶段：应用
1. 学习[TDD任务拆分指南](tdd-task-splitting-guide.md)
2. 使用适当的任务分解完成[中等问题](../medium/)
3. 尝试[Spring Boot初级问题](../spring-boot/beginner/)

### 第三阶段：精通
1. 将高级拆分技术应用到实际项目
2. 为你的领域创建自定义模板
3. 解决[困难问题](../hard/)和[Spring Boot高级场景](../spring-boot/advanced/)

## 🤝 贡献

如果你想为这些文档资源做贡献：
1. 遵循现有格式和结构
2. 提供实际例子
3. 包含英文和中文版本
4. 用真实的TDD实践测试你的例子