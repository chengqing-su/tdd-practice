# TDD Guide for Beginners

## What is Test-Driven Development?

Test-Driven Development (TDD) is a software development approach where you write tests before writing the actual code. It follows a simple cycle known as Red-Green-Refactor.

## The Red-Green-Refactor Cycle

### 🔴 Red Phase
1. Write a failing test that describes the desired behavior
2. Run the test to confirm it fails (important!)
3. The failure should be meaningful - test what you intend to build

### 🟢 Green Phase
1. Write the minimal code needed to make the test pass
2. Don't worry about perfect code - just make it work
3. Run the test to confirm it passes

### 🔵 Refactor Phase
1. Improve the code while keeping tests green
2. Remove duplication
3. Improve readability and design
4. Run tests frequently to ensure nothing breaks

## Core Principles

### 1. Write Only Enough Test Code to Fail
- Don't write comprehensive tests upfront
- Focus on one specific behavior at a time

### 2. Write Only Enough Production Code to Pass
- Resist the urge to write extra code "you might need later"
- Keep implementations simple and focused

### 3. Refactor Both Test and Production Code
- Clean up both test code and implementation
- Tests are first-class citizens in your codebase

## Benefits of TDD

### Design Benefits
- **Better API Design**: Writing tests first forces you to think about how your code will be used
- **Simpler Code**: TDD encourages minimal, focused implementations
- **Modular Architecture**: Testable code tends to be well-structured

### Quality Benefits
- **Fewer Bugs**: High test coverage catches issues early
- **Regression Protection**: Tests prevent breaking existing functionality
- **Living Documentation**: Tests serve as executable specifications

### Development Benefits
- **Faster Feedback**: Immediate feedback on code changes
- **Confidence to Refactor**: Comprehensive tests enable safe code improvements
- **Focus**: One failing test keeps you focused on current task

## Common Mistakes

### 1. Writing Too Many Tests at Once
- **Problem**: Writing multiple failing tests before implementing
- **Solution**: Write one failing test, make it pass, then write the next

### 2. Making Tests Too Complex
- **Problem**: Tests that are hard to read and maintain
- **Solution**: Keep tests simple, focused, and well-named

### 3. Skipping the Refactor Phase
- **Problem**: Accumulating technical debt
- **Solution**: Always clean up code after making tests pass

### 4. Testing Implementation Details
- **Problem**: Tests break when refactoring internal code
- **Solution**: Test behavior, not implementation

## Best Practices

### Test Naming
- Use descriptive names that explain what is being tested
- Include the expected behavior in the name
- Examples:
  - `should_return_fizz_when_number_is_divisible_by_three`
  - `should_throw_exception_when_account_has_insufficient_funds`

### Test Structure (Arrange-Act-Assert)
```python
def test_calculator_adds_two_numbers():
    # Arrange
    calculator = Calculator()

    # Act
    result = calculator.add(2, 3)

    # Assert
    assert result == 5
```

### Keep Tests Independent
- Each test should run independently of others
- Tests should not depend on the order of execution
- Clean up resources after each test

## Getting Started Tips

### 1. Start Small
- Begin with simple problems like FizzBuzz or Calculator
- Focus on mastering the Red-Green-Refactor cycle
- Don't try to learn TDD on complex projects initially

### 2. Use Your IDE Effectively
- Learn keyboard shortcuts for running tests
- Set up automatic test execution on file changes
- Use debugger to understand test failures

### 3. Practice Regularly
- TDD is a skill that improves with practice
- Try coding katas and simple exercises
- Join TDD practice groups or online communities

### 4. Read and Learn
- Study TDD books and articles
- Watch experienced developers do TDD
- Learn from code reviews and pair programming

## Recommended Reading

- **"Test Driven Development: By Example" by Kent Beck** - The definitive TDD guide
- **"Growing Object-Oriented Software, Guided by Tests" by Steve Freeman and Nat Pryce** - Advanced TDD techniques
- **"The Art of Unit Testing" by Roy Osherove** - Comprehensive testing guide

## Tools and Frameworks

### Testing Frameworks by Language
- **Java**: JUnit, TestNG, Mockito
- **Python**: pytest, unittest, mock
- **JavaScript**: Jest, Mocha, Jasmine
- **C#**: NUnit, MSTest, Moq
- **Ruby**: RSpec, Minitest

### IDE Plugins
- Test runners and coverage tools
- Automatic test execution
- TDD timers and cycle reminders

## Next Steps

1. Choose a simple problem from the [Easy Level](../easy/) section
2. Set up your development environment with a testing framework
3. Start with the Red-Green-Refactor cycle
4. Practice regularly with different types of problems
5. Gradually move to more complex scenarios

Remember: TDD is a journey, not a destination. The more you practice, the more natural it becomes!

---

# TDD 初学者指南

## 什么是测试驱动开发？

测试驱动开发（TDD）是一种软件开发方法，在编写实际代码之前先编写测试。它遵循一个简单的循环，被称为红绿重构。

## 红绿重构循环

### 🔴 红灯阶段
1. 编写描述期望行为的失败测试
2. 运行测试确认它失败（重要！）
3. 失败应该是有意义的 - 测试你打算构建的东西

### 🟢 绿灯阶段
1. 编写使测试通过所需的最少代码
2. 不要担心完美的代码 - 只要让它工作
3. 运行测试确认它通过

### 🔵 重构阶段
1. 在保持测试绿色的同时改进代码
2. 消除重复
3. 提高可读性和设计质量
4. 频繁运行测试确保没有破坏任何东西

## 核心原则

### 1. 只编写足够失败的测试代码
- 不要预先编写全面的测试
- 一次专注于一个特定行为

### 2. 只编写足够通过的生产代码
- 抵制编写"以后可能需要"的额外代码的冲动
- 保持实现简单和专注

### 3. 重构测试和生产代码
- 清理测试代码和实现
- 测试是代码库中的一等公民

## TDD 的好处

### 设计好处
- **更好的API设计**: 先写测试迫使你思考代码将如何被使用
- **更简单的代码**: TDD鼓励最小化、专注的实现
- **模块化架构**: 可测试的代码往往结构良好

### 质量好处
- **更少的错误**: 高测试覆盖率早期捕获问题
- **回归保护**: 测试防止破坏现有功能
- **活文档**: 测试作为可执行规范

### 开发好处
- **更快反馈**: 代码更改的即时反馈
- **重构信心**: 全面的测试使安全的代码改进成为可能
- **专注**: 一个失败的测试让你专注于当前任务

## 常见错误

### 1. 一次写太多测试
- **问题**: 在实现之前编写多个失败测试
- **解决方案**: 写一个失败测试，让它通过，然后写下一个

### 2. 让测试过于复杂
- **问题**: 难以阅读和维护的测试
- **解决方案**: 保持测试简单、专注、命名良好

### 3. 跳过重构阶段
- **问题**: 积累技术债务
- **解决方案**: 在让测试通过后总是清理代码

### 4. 测试实现细节
- **问题**: 重构内部代码时测试中断
- **解决方案**: 测试行为，不是实现

## 最佳实践

### 测试命名
- 使用描述性名称解释正在测试什么
- 在名称中包含期望行为
- 示例:
  - `should_return_fizz_when_number_is_divisible_by_three`
  - `should_throw_exception_when_account_has_insufficient_funds`

### 测试结构（安排-执行-断言）
```python
def test_calculator_adds_two_numbers():
    # 安排
    calculator = Calculator()

    # 执行
    result = calculator.add(2, 3)

    # 断言
    assert result == 5
```

### 保持测试独立
- 每个测试应该独立于其他测试运行
- 测试不应依赖执行顺序
- 每个测试后清理资源

## 入门技巧

### 1. 从小开始
- 从像FizzBuzz或计算器这样的简单问题开始
- 专注于掌握红绿重构循环
- 不要在复杂项目上初始学习TDD

### 2. 有效使用IDE
- 学习运行测试的键盘快捷键
- 设置文件更改时自动测试执行
- 使用调试器理解测试失败

### 3. 定期练习
- TDD是一个通过练习改进的技能
- 尝试编码kata和简单练习
- 加入TDD练习小组或在线社区

### 4. 阅读和学习
- 学习TDD书籍和文章
- 观看有经验的开发者做TDD
- 从代码评审和结对编程中学习

## 推荐阅读

- **《测试驱动开发：实例教程》Kent Beck著** - 权威TDD指南
- **《实例化需求》Gojko Adzic著** - 高级TDD技术
- **《单元测试的艺术》Roy Osherove著** - 全面的测试指南

## 工具和框架

### 按语言的测试框架
- **Java**: JUnit, TestNG, Mockito
- **Python**: pytest, unittest, mock
- **JavaScript**: Jest, Mocha, Jasmine
- **C#**: NUnit, MSTest, Moq
- **Ruby**: RSpec, Minitest

### IDE插件
- 测试运行器和覆盖率工具
- 自动测试执行
- TDD计时器和循环提醒

## 下一步

1. 从[简单级别](../easy/)部分选择一个简单问题
2. 用测试框架设置你的开发环境
3. 从红绿重构循环开始
4. 用不同类型的问题定期练习
5. 逐步转向更复杂的场景

记住：TDD是一个旅程，不是目的地。你练习得越多，它就变得越自然！