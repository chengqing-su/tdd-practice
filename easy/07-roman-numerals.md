# Roman Numerals Converter

## Problem Description

Create functions to convert between Arabic numbers and Roman numerals for numbers 1-3999.

## Roman Numeral Rules

### Basic Symbols
- I = 1
- V = 5
- X = 10
- L = 50
- C = 100
- D = 500
- M = 1000

### Rules
1. **Addition**: When a smaller symbol follows a larger one, add them (VI = 6)
2. **Subtraction**: When a smaller symbol precedes a larger one, subtract it (IV = 4)
3. **Repetition**: A symbol can be repeated up to 3 times (III = 3, XXX = 30)
4. **Subtraction pairs**: Only I, X, and C can be subtracted, and only from the next two higher symbols
   - I can be subtracted from V (4) and X (9)
   - X can be subtracted from L (40) and C (90)
   - C can be subtracted from D (400) and M (900)

## Examples

### toRoman(number)
```javascript
toRoman(1) → "I"
toRoman(4) → "IV"
toRoman(9) → "IX"
toRoman(27) → "XXVII"
toRoman(48) → "XLVIII"
toRoman(1994) → "MCMXCIV"
toRoman(3999) → "MMMCMXCIX"
```

### fromRoman(roman)
```javascript
fromRoman("I") → 1
fromRoman("IV") → 4
fromRoman("IX") → 9
fromRoman("XXVII") → 27
fromRoman("XLVIII") → 48
fromRoman("MCMXCIV") → 1994
fromRoman("MMMCMXCIX") → 3999
```

## Suggested Test Cases

### toRoman Tests
1. Single digits: 1→"I", 2→"II", 3→"III", 4→"IV", 5→"V", 9→"IX"
2. Tens: 10→"X", 20→"XX", 40→"XL", 50→"L", 90→"XC"
3. Hundreds: 100→"C", 400→"CD", 500→"D", 900→"CM"
4. Thousands: 1000→"M", 2000→"MM", 3000→"MMM"
5. Complex numbers: 1994→"MCMXCIV", 2023→"MMXXIII"
6. Edge cases: 3999 (maximum), boundary values

### fromRoman Tests
1. Basic symbols: "I"→1, "V"→5, "X"→10, "L"→50, "C"→100, "D"→500, "M"→1000
2. Subtraction cases: "IV"→4, "IX"→9, "XL"→40, "XC"→90, "CD"→400, "CM"→900
3. Complex numerals: "MCMXCIV"→1994, "MMXXIII"→2023
4. Invalid inputs: empty string, invalid characters, invalid combinations

## TDD Steps

### For toRoman:
1. **Red**: Test toRoman(1) → "I"
2. **Green**: Return "I" for input 1
3. **Red**: Test toRoman(2) → "II"
4. **Green**: Handle numbers 1-3 with repeated "I"
5. **Red**: Test toRoman(4) → "IV"
6. **Green**: Handle subtraction case for 4
7. **Red**: Test toRoman(5) → "V"
8. **Green**: Handle 5
9. **Red**: Test toRoman(9) → "IX"
10. **Green**: Handle subtraction case for 9
11. **Continue**: Add tens, hundreds, thousands
12. **Refactor**: Optimize using mapping arrays

### For fromRoman:
1. **Red**: Test fromRoman("I") → 1
2. **Green**: Handle single "I"
3. **Red**: Test fromRoman("II") → 2
4. **Green**: Handle repeated symbols
5. **Red**: Test fromRoman("IV") → 4
6. **Green**: Handle subtraction cases
7. **Continue**: Build up complexity
8. **Refactor**: Clean up parsing logic

## Learning Goals

- String processing and pattern matching
- Number base conversion concepts
- Historical numbering systems
- Algorithm design for parsing
- Input validation and error handling
- Bidirectional conversion logic

## Advanced Challenges

1. **Validation**: Add strict Roman numeral validation (reject invalid combinations)
2. **Alternative notations**: Handle alternative historical forms
3. **Performance**: Optimize for large numbers or batch conversions
4. **Error handling**: Provide meaningful error messages for invalid inputs

---

## 问题描述

创建函数在阿拉伯数字和罗马数字之间转换（1-3999）。

## 罗马数字规则

### 基本符号
- I = 1
- V = 5
- X = 10
- L = 50
- C = 100
- D = 500
- M = 1000

### 规则
1. **加法**: 当小符号跟在大符号后面时，相加（VI = 6）
2. **减法**: 当小符号在大符号前面时，相减（IV = 4）
3. **重复**: 符号最多可重复3次（III = 3, XXX = 30）
4. **减法组合**: 只有I、X和C可以被减，且只能从接下来两个更大的符号中减
   - I可以从V（4）和X（9）中减
   - X可以从L（40）和C（90）中减
   - C可以从D（400）和M（900）中减

## 示例

### toRoman(number)
```javascript
toRoman(1) → "I"
toRoman(4) → "IV"
toRoman(9) → "IX"
toRoman(27) → "XXVII"
toRoman(48) → "XLVIII"
toRoman(1994) → "MCMXCIV"
toRoman(3999) → "MMMCMXCIX"
```

### fromRoman(roman)
```javascript
fromRoman("I") → 1
fromRoman("IV") → 4
fromRoman("IX") → 9
fromRoman("XXVII") → 27
fromRoman("XLVIII") → 48
fromRoman("MCMXCIV") → 1994
fromRoman("MMMCMXCIX") → 3999
```

## 建议测试用例

### toRoman 测试
1. 个位数：1→"I", 2→"II", 3→"III", 4→"IV", 5→"V", 9→"IX"
2. 十位：10→"X", 20→"XX", 40→"XL", 50→"L", 90→"XC"
3. 百位：100→"C", 400→"CD", 500→"D", 900→"CM"
4. 千位：1000→"M", 2000→"MM", 3000→"MMM"
5. 复杂数字：1994→"MCMXCIV", 2023→"MMXXIII"
6. 边界情况：3999（最大值），边界值

### fromRoman 测试
1. 基本符号："I"→1, "V"→5, "X"→10, "L"→50, "C"→100, "D"→500, "M"→1000
2. 减法情况："IV"→4, "IX"→9, "XL"→40, "XC"→90, "CD"→400, "CM"→900
3. 复杂数字："MCMXCIV"→1994, "MMXXIII"→2023
4. 无效输入：空字符串，无效字符，无效组合

## TDD 步骤

### 对于 toRoman:
1. **红灯**: 测试 toRoman(1) → "I"
2. **绿灯**: 对输入1返回"I"
3. **红灯**: 测试 toRoman(2) → "II"
4. **绿灯**: 用重复"I"处理数字1-3
5. **红灯**: 测试 toRoman(4) → "IV"
6. **绿灯**: 处理4的减法情况
7. **红灯**: 测试 toRoman(5) → "V"
8. **绿灯**: 处理5
9. **红灯**: 测试 toRoman(9) → "IX"
10. **绿灯**: 处理9的减法情况
11. **继续**: 添加十位、百位、千位
12. **重构**: 使用映射数组优化

### 对于 fromRoman:
1. **红灯**: 测试 fromRoman("I") → 1
2. **绿灯**: 处理单个"I"
3. **红灯**: 测试 fromRoman("II") → 2
4. **绿灯**: 处理重复符号
5. **红灯**: 测试 fromRoman("IV") → 4
6. **绿灯**: 处理减法情况
7. **继续**: 构建复杂性
8. **重构**: 清理解析逻辑

## 学习目标

- 字符串处理和模式匹配
- 数字进制转换概念
- 历史数字系统
- 解析算法设计
- 输入验证和错误处理
- 双向转换逻辑

## 高级挑战

1. **验证**: 添加严格的罗马数字验证（拒绝无效组合）
2. **替代记法**: 处理替代历史形式
3. **性能**: 针对大数字或批量转换优化
4. **错误处理**: 为无效输入提供有意义的错误信息