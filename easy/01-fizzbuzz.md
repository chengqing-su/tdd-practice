# FizzBuzz

## Problem Description

Write a function that takes a positive integer `n` and returns a list of strings from 1 to `n` with the following rules:
- For multiples of 3, return "Fizz"
- For multiples of 5, return "Buzz"
- For multiples of both 3 and 5, return "FizzBuzz"
- For all other numbers, return the number as a string

## Examples

```
fizzBuzz(5) → ["1", "2", "Fizz", "4", "Buzz"]
fizzBuzz(15) → ["1", "2", "Fizz", "4", "Buzz", "Fizz", "7", "8", "Fizz", "Buzz", "11", "Fizz", "13", "14", "FizzBuzz"]
```

## Suggested Test Cases

1. `fizzBuzz(1)` should return `["1"]`
2. `fizzBuzz(3)` should return `["1", "2", "Fizz"]`
3. `fizzBuzz(5)` should return `["1", "2", "Fizz", "4", "Buzz"]`
4. `fizzBuzz(15)` should return the complete sequence including "FizzBuzz"
5. Handle edge cases like `fizzBuzz(0)` appropriately

## TDD Steps

1. **Red**: Write a failing test for `fizzBuzz(1)`
2. **Green**: Make it pass with the simplest implementation
3. **Refactor**: Clean up if needed
4. **Repeat**: Add tests for multiples of 3, then 5, then both

## Learning Goals

- Basic TDD cycle (Red-Green-Refactor)
- Simple conditional logic
- Working with arrays/lists
- String conversion

---

## 问题描述

编写一个函数，接受正整数 `n`，返回从1到n的字符串列表，规则如下：
- 3的倍数返回 "Fizz"
- 5的倍数返回 "Buzz"
- 既是3又是5的倍数返回 "FizzBuzz"
- 其他数字返回数字的字符串形式

## 示例

```
fizzBuzz(5) → ["1", "2", "Fizz", "4", "Buzz"]
fizzBuzz(15) → ["1", "2", "Fizz", "4", "Buzz", "Fizz", "7", "8", "Fizz", "Buzz", "11", "Fizz", "13", "14", "FizzBuzz"]
```

## 建议测试用例

1. `fizzBuzz(1)` 应该返回 `["1"]`
2. `fizzBuzz(3)` 应该返回 `["1", "2", "Fizz"]`
3. `fizzBuzz(5)` 应该返回 `["1", "2", "Fizz", "4", "Buzz"]`
4. `fizzBuzz(15)` 应该返回包含 "FizzBuzz" 的完整序列
5. 适当处理边界情况如 `fizzBuzz(0)`

## TDD 步骤

1. **红灯**: 为 `fizzBuzz(1)` 编写失败测试
2. **绿灯**: 用最简单的实现让测试通过
3. **重构**: 必要时清理代码
4. **重复**: 添加3的倍数、5的倍数、两者倍数的测试

## 学习目标

- 基础TDD循环（红绿重构）
- 简单条件逻辑
- 数组/列表操作
- 字符串转换