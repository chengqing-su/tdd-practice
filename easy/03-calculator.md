# Simple Calculator

## Problem Description

Create a simple calculator class with methods for basic arithmetic operations: addition, subtraction, multiplication, and division. The calculator should handle edge cases like division by zero.

## Examples

```
calc = Calculator()
calc.add(2, 3) → 5
calc.subtract(10, 4) → 6
calc.multiply(3, 7) → 21
calc.divide(15, 3) → 5
calc.divide(10, 0) → throws error or returns special value
```

## Suggested Test Cases

1. Addition: positive numbers, negative numbers, zero
2. Subtraction: positive result, negative result, subtracting zero
3. Multiplication: positive numbers, negative numbers, zero, one
4. Division: normal division, division by zero, dividing zero
5. Floating point operations
6. Chain operations (if implementing)

## TDD Steps

1. **Red**: Write test for `add(2, 3) == 5`
2. **Green**: Implement simple addition
3. **Refactor**: Clean up if needed
4. **Repeat**: Add more addition tests, then other operations
5. **Red**: Test division by zero
6. **Green**: Handle division by zero appropriately
7. **Refactor**: Ensure error handling is consistent

## Learning Goals

- Class design
- Method implementation
- Error handling
- Edge case testing
- Basic arithmetic operations

---

## 问题描述

创建一个简单的计算器类，包含基本算术运算方法：加法、减法、乘法和除法。计算器应处理边界情况，如除零操作。

## 示例

```
calc = Calculator()
calc.add(2, 3) → 5
calc.subtract(10, 4) → 6
calc.multiply(3, 7) → 21
calc.divide(15, 3) → 5
calc.divide(10, 0) → 抛出错误或返回特殊值
```

## 建议测试用例

1. 加法：正数、负数、零
2. 减法：正数结果、负数结果、减零
3. 乘法：正数、负数、零、一
4. 除法：正常除法、除零、零被除
5. 浮点数运算
6. 连续运算（如果实现的话）

## TDD 步骤

1. **红灯**: 为 `add(2, 3) == 5` 编写测试
2. **绿灯**: 实现简单加法
3. **重构**: 必要时清理代码
4. **重复**: 添加更多加法测试，然后其他运算
5. **红灯**: 测试除零情况
6. **绿灯**: 适当处理除零操作
7. **重构**: 确保错误处理的一致性

## 学习目标

- 类设计
- 方法实现
- 错误处理
- 边界情况测试
- 基本算术运算