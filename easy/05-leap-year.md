# Leap Year Calculator

## Problem Description

Write a function that determines if a given year is a leap year according to the Gregorian calendar rules.

## Leap Year Rules

A year is a leap year if:
1. It is divisible by 4 AND
2. If it is divisible by 100, then it must also be divisible by 400

## Examples

```python
isLeapYear(2000) → True   # Divisible by 400
isLeapYear(1900) → False  # Divisible by 100 but not by 400
isLeapYear(2004) → True   # Divisible by 4 but not by 100
isLeapYear(2001) → False  # Not divisible by 4
isLeapYear(2024) → True   # Divisible by 4 but not by 100
```

## Suggested Test Cases

1. Regular leap years: 2004, 2008, 2012, 2016, 2020, 2024
2. Century years that are leap years: 1600, 2000, 2400
3. Century years that are not leap years: 1700, 1800, 1900, 2100, 2200, 2300
4. Regular non-leap years: 2001, 2002, 2003, 2005, 2006, 2007
5. Edge cases: year 1, year 0 (if handling), negative years (if applicable)

## TDD Steps

1. **Red**: Test a simple leap year (e.g., 2004)
2. **Green**: Implement basic divisible by 4 check
3. **Red**: Test a non-leap year (e.g., 2001)
4. **Green**: Add logic for non-divisible by 4
5. **Red**: Test century non-leap year (e.g., 1900)
6. **Green**: Add divisible by 100 but not 400 logic
7. **Red**: Test century leap year (e.g., 2000)
8. **Green**: Complete the 400-year rule
9. **Refactor**: Simplify and optimize the logic

## Learning Goals

- Boolean logic and conditions
- Mathematical operations (modulo)
- Complex conditional statements
- Historical calendar rules
- Edge case testing

## Alternative Implementation Challenges

Once you have a working solution, try implementing it in different ways:
1. Single boolean expression
2. Multiple if-else statements
3. Using a lookup table for test cases
4. Functional programming approach

---

## 问题描述

编写一个函数，根据格里高利历规则判断给定年份是否为闰年。

## 闰年规则

一年是闰年如果：
1. 能被4整除 且
2. 如果能被100整除，那么也必须能被400整除

## 示例

```python
isLeapYear(2000) → True   # 能被400整除
isLeapYear(1900) → False  # 能被100整除但不能被400整除
isLeapYear(2004) → True   # 能被4整除但不能被100整除
isLeapYear(2001) → False  # 不能被4整除
isLeapYear(2024) → True   # 能被4整除但不能被100整除
```

## 建议测试用例

1. 普通闰年：2004, 2008, 2012, 2016, 2020, 2024
2. 是闰年的世纪年：1600, 2000, 2400
3. 不是闰年的世纪年：1700, 1800, 1900, 2100, 2200, 2300
4. 普通非闰年：2001, 2002, 2003, 2005, 2006, 2007
5. 边界情况：第1年，第0年（如果处理），负年（如果适用）

## TDD 步骤

1. **红灯**: 测试简单闰年（如2004）
2. **绿灯**: 实现基本的被4整除检查
3. **红灯**: 测试非闰年（如2001）
4. **绿灯**: 添加不被4整除的逻辑
5. **红灯**: 测试世纪非闰年（如1900）
6. **绿灯**: 添加被100整除但不被400整除的逻辑
7. **红灯**: 测试世纪闰年（如2000）
8. **绿灯**: 完成400年规则
9. **重构**: 简化和优化逻辑

## 学习目标

- 布尔逻辑和条件
- 数学运算（取模）
- 复杂条件语句
- 历史日历规则
- 边界情况测试

## 替代实现挑战

一旦你有了可工作的解决方案，尝试用不同方式实现：
1. 单个布尔表达式
2. 多个if-else语句
3. 为测试用例使用查找表
4. 函数式编程方法