# Palindrome Checker

## Problem Description

Write a function `isPalindrome(s)` that determines if a given string is a palindrome. A palindrome reads the same forwards and backwards, ignoring case, spaces, and punctuation.

## Examples

```
isPalindrome("racecar") → true
isPalindrome("A man a plan a canal Panama") → true
isPalindrome("race a car") → false
isPalindrome("hello") → false
isPalindrome("") → true
```

## Suggested Test Cases

1. Empty string should return `true`
2. Single character should return `true`
3. Simple palindromes: "aba", "racecar"
4. Case insensitive: "Aa", "RaceCar"
5. With spaces: "a man a plan a canal panama"
6. With punctuation: "A man, a plan, a canal: Panama!"
7. Non-palindromes: "hello", "world"

## TDD Steps

1. **Red**: Start with empty string test
2. **Green**: Return `true` for empty string
3. **Red**: Add single character test
4. **Green**: Handle single character
5. **Red**: Add simple palindrome test
6. **Green**: Implement basic palindrome check
7. **Refactor**: Clean up and optimize
8. **Repeat**: Add case insensitive, spaces, punctuation

## Learning Goals

- String manipulation
- Character comparison
- Handling edge cases
- Case insensitive comparison
- Regular expressions (for cleaning input)

---

## 问题描述

编写函数 `isPalindrome(s)`，判断给定字符串是否为回文。回文正读反读都一样，忽略大小写、空格和标点符号。

## 示例

```
isPalindrome("racecar") → true
isPalindrome("A man a plan a canal Panama") → true
isPalindrome("race a car") → false
isPalindrome("hello") → false
isPalindrome("") → true
```

## 建议测试用例

1. 空字符串应返回 `true`
2. 单个字符应返回 `true`
3. 简单回文："aba", "racecar"
4. 不区分大小写："Aa", "RaceCar"
5. 包含空格："a man a plan a canal panama"
6. 包含标点："A man, a plan, a canal: Panama!"
7. 非回文："hello", "world"

## TDD 步骤

1. **红灯**: 从空字符串测试开始
2. **绿灯**: 为空字符串返回 `true`
3. **红灯**: 添加单字符测试
4. **绿灯**: 处理单字符情况
5. **红灯**: 添加简单回文测试
6. **绿灯**: 实现基本回文检查
7. **重构**: 清理和优化代码
8. **重复**: 添加不区分大小写、空格、标点的处理

## 学习目标

- 字符串操作
- 字符比较
- 边界情况处理
- 不区分大小写比较
- 正则表达式（用于清理输入）