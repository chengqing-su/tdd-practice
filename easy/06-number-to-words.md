# Number to Words Converter

## Problem Description

Create a function that converts numbers (0-999) into their English word representation.

## Examples

```javascript
numberToWords(0) → "zero"
numberToWords(5) → "five"
numberToWords(23) → "twenty three"
numberToWords(101) → "one hundred one"
numberToWords(999) → "nine hundred ninety nine"
numberToWords(520) → "five hundred twenty"
```

## Requirements

- Handle numbers from 0 to 999
- Use proper English number naming conventions
- Handle special cases like teens (11, 12, 13, etc.)
- No leading/trailing spaces in output

## Number Patterns

- **0-19**: zero, one, two, ..., nineteen (unique words)
- **20-90**: twenty, thirty, forty, fifty, sixty, seventy, eighty, ninety
- **100s**: one hundred, two hundred, ..., nine hundred

## Suggested Test Cases

1. Single digits: 0, 1, 5, 9
2. Teens: 10, 11, 15, 19
3. Tens: 20, 30, 50, 90
4. Two digits: 21, 34, 67, 89
5. Hundreds: 100, 200, 500, 900
6. Hundreds with tens: 123, 456, 789
7. Hundreds with single digits: 101, 205, 807
8. Edge cases: 10, 20, 100

## TDD Steps

1. **Red**: Test single digit (e.g., 5 → "five")
2. **Green**: Implement basic single digit conversion
3. **Red**: Test zero case (0 → "zero")
4. **Green**: Handle zero specifically
5. **Red**: Test teens (e.g., 15 → "fifteen")
6. **Green**: Implement teens handling (10-19)
7. **Red**: Test tens (e.g., 30 → "thirty")
8. **Green**: Implement tens conversion (20, 30, ..., 90)
9. **Red**: Test two-digit numbers (e.g., 23 → "twenty three")
10. **Green**: Combine tens and ones
11. **Red**: Test hundreds (e.g., 100 → "one hundred")
12. **Green**: Implement hundreds conversion
13. **Red**: Test complex numbers (e.g., 123 → "one hundred twenty three")
14. **Green**: Combine hundreds, tens, and ones
15. **Refactor**: Clean up and optimize

## Learning Goals

- String manipulation and concatenation
- Array/list usage for lookups
- Complex conditional logic
- Modular function design
- Pattern recognition in number systems

## Extension Ideas

Once you complete the basic version:
1. Extend to support numbers up to 9,999 (add thousands)
2. Add support for negative numbers
3. Handle decimal numbers
4. Add different language support

---

## 问题描述

创建一个函数，将数字（0-999）转换为对应的英文单词表示。

## 示例

```javascript
numberToWords(0) → "zero"
numberToWords(5) → "five"
numberToWords(23) → "twenty three"
numberToWords(101) → "one hundred one"
numberToWords(999) → "nine hundred ninety nine"
numberToWords(520) → "five hundred twenty"
```

## 需求

- 处理0到999的数字
- 使用正确的英文数字命名约定
- 处理特殊情况如teens（11, 12, 13等）
- 输出中没有前导/尾随空格

## 数字模式

- **0-19**: zero, one, two, ..., nineteen（唯一单词）
- **20-90**: twenty, thirty, forty, fifty, sixty, seventy, eighty, ninety
- **100s**: one hundred, two hundred, ..., nine hundred

## 建议测试用例

1. 个位数：0, 1, 5, 9
2. 十几：10, 11, 15, 19
3. 整十：20, 30, 50, 90
4. 两位数：21, 34, 67, 89
5. 整百：100, 200, 500, 900
6. 百位加十位：123, 456, 789
7. 百位加个位：101, 205, 807
8. 边界情况：10, 20, 100

## TDD 步骤

1. **红灯**: 测试个位数（如5 → "five"）
2. **绿灯**: 实现基本个位数转换
3. **红灯**: 测试零的情况（0 → "zero"）
4. **绿灯**: 特别处理零
5. **红灯**: 测试十几（如15 → "fifteen"）
6. **绿灯**: 实现十几的处理（10-19）
7. **红灯**: 测试整十（如30 → "thirty"）
8. **绿灯**: 实现整十转换（20, 30, ..., 90）
9. **红灯**: 测试两位数（如23 → "twenty three"）
10. **绿灯**: 组合十位和个位
11. **红灯**: 测试整百（如100 → "one hundred"）
12. **绿灯**: 实现百位转换
13. **红灯**: 测试复杂数字（如123 → "one hundred twenty three"）
14. **绿灯**: 组合百位、十位和个位
15. **重构**: 清理和优化

## 学习目标

- 字符串操作和连接
- 数组/列表用于查找
- 复杂条件逻辑
- 模块化函数设计
- 数字系统中的模式识别

## 扩展想法

完成基本版本后：
1. 扩展支持到9,999的数字（添加千位）
2. 添加负数支持
3. 处理小数
4. 添加不同语言支持