# Word Count

## Problem Description

Create a function that analyzes a text string and returns various statistics about the words it contains.

## Requirements

Write a function `analyzeText(text)` that returns an object/dictionary with:
- `wordCount`: Total number of words
- `characterCount`: Total number of characters (excluding spaces)
- `characterCountWithSpaces`: Total number of characters (including spaces)
- `longestWord`: The longest word in the text
- `averageWordLength`: Average length of words (rounded to 2 decimal places)

## Examples

```javascript
analyzeText("Hello world")
→ {
    wordCount: 2,
    characterCount: 10,
    characterCountWithSpaces: 11,
    longestWord: "world",
    averageWordLength: 5.00
  }

analyzeText("The quick brown fox jumps")
→ {
    wordCount: 5,
    characterCount: 20,
    characterCountWithSpaces: 24,
    longestWord: "quick",
    averageWordLength: 4.00
  }

analyzeText("") → {
    wordCount: 0,
    characterCount: 0,
    characterCountWithSpaces: 0,
    longestWord: "",
    averageWordLength: 0
  }
```

## Suggested Test Cases

1. Empty string handling
2. Single word
3. Multiple words with spaces
4. Text with multiple spaces between words
5. Text with punctuation (decide how to handle)
6. Text with mixed case
7. Very long text
8. Text with numbers

## TDD Steps

1. **Red**: Test empty string returns all zeros
2. **Green**: Handle empty string case
3. **Red**: Test single word
4. **Green**: Implement single word analysis
5. **Red**: Test multiple words
6. **Green**: Implement word splitting and counting
7. **Red**: Test longest word detection
8. **Green**: Implement longest word logic
9. **Red**: Test average word length calculation
10. **Green**: Implement average calculation
11. **Refactor**: Clean up and optimize

## Learning Goals

- String manipulation and analysis
- Data structure creation (objects/dictionaries)
- Mathematical calculations (averages)
- Edge case handling (empty strings)
- Text processing algorithms

---

## 问题描述

创建一个函数，分析文本字符串并返回包含单词的各种统计信息。

## 需求

编写函数 `analyzeText(text)`，返回包含以下内容的对象/字典：
- `wordCount`: 单词总数
- `characterCount`: 字符总数（不包括空格）
- `characterCountWithSpaces`: 字符总数（包括空格）
- `longestWord`: 文本中最长的单词
- `averageWordLength`: 单词平均长度（四舍五入到2位小数）

## 示例

```javascript
analyzeText("Hello world")
→ {
    wordCount: 2,
    characterCount: 10,
    characterCountWithSpaces: 11,
    longestWord: "world",
    averageWordLength: 5.00
  }

analyzeText("The quick brown fox jumps")
→ {
    wordCount: 5,
    characterCount: 20,
    characterCountWithSpaces: 24,
    longestWord: "quick",
    averageWordLength: 4.00
  }

analyzeText("") → {
    wordCount: 0,
    characterCount: 0,
    characterCountWithSpaces: 0,
    longestWord: "",
    averageWordLength: 0
  }
```

## 建议测试用例

1. 空字符串处理
2. 单个单词
3. 有空格的多个单词
4. 单词间有多个空格的文本
5. 有标点符号的文本（决定如何处理）
6. 大小写混合的文本
7. 很长的文本
8. 包含数字的文本

## TDD 步骤

1. **红灯**: 测试空字符串返回全零
2. **绿灯**: 处理空字符串情况
3. **红灯**: 测试单个单词
4. **绿灯**: 实现单词分析
5. **红灯**: 测试多个单词
6. **绿灯**: 实现单词分割和计数
7. **红灯**: 测试最长单词检测
8. **绿灯**: 实现最长单词逻辑
9. **红灯**: 测试平均单词长度计算
10. **绿灯**: 实现平均值计算
11. **重构**: 清理和优化

## 学习目标

- 字符串操作和分析
- 数据结构创建（对象/字典）
- 数学计算（平均值）
- 边界情况处理（空字符串）
- 文本处理算法