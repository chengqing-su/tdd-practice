# Password Validator

## Problem Description

Create a password validation system that checks if passwords meet specified security criteria.

## Password Requirements

A valid password must:
1. Be at least 8 characters long
2. Contain at least one uppercase letter (A-Z)
3. Contain at least one lowercase letter (a-z)
4. Contain at least one digit (0-9)
5. Contain at least one special character (!@#$%^&*)
6. Not contain common weak patterns

## Requirements

### PasswordValidator Class
- `validate(password)` - Returns validation result object
- `isValid(password)` - Returns boolean
- `getFailedCriteria(password)` - Returns list of failed criteria
- `getSuggestions(password)` - Returns improvement suggestions

### Validation Result
```javascript
{
  isValid: boolean,
  failedCriteria: string[],
  suggestions: string[],
  strength: "weak" | "medium" | "strong"
}
```

## Examples

```javascript
validator = new PasswordValidator()

validator.validate("Pass123!")
→ {
    isValid: true,
    failedCriteria: [],
    suggestions: [],
    strength: "strong"
  }

validator.validate("password")
→ {
    isValid: false,
    failedCriteria: ["no_uppercase", "no_digit", "no_special_char"],
    suggestions: ["Add uppercase letters", "Add numbers", "Add special characters"],
    strength: "weak"
  }

validator.validate("Pass1")
→ {
    isValid: false,
    failedCriteria: ["too_short", "no_special_char"],
    suggestions: ["Use at least 8 characters", "Add special characters"],
    strength: "weak"
  }
```

## Suggested Test Cases

### Basic Validation
1. Valid passwords: "Password123!", "MySecure@Pass1", "StrongP@ssw0rd"
2. Too short: "Pass1!", "123", "abC!"
3. No uppercase: "password123!", "mypass@123"
4. No lowercase: "PASSWORD123!", "MYPASS@123"
5. No digits: "Password!", "MyPassw@rd"
6. No special chars: "Password123", "MyPassword123"

### Advanced Validation
1. Common weak patterns: "Password123", "qwerty123", "abc123"
2. Sequential patterns: "123456789", "abcdefgh"
3. Repeated characters: "aaaaaa", "111111"
4. Empty/null passwords
5. Very long passwords (test performance)

### Edge Cases
1. Unicode characters
2. Whitespace handling
3. Case sensitivity
4. Special character edge cases

## TDD Steps

1. **Red**: Test basic valid password
2. **Green**: Implement basic structure returning true
3. **Red**: Test minimum length requirement
4. **Green**: Add length validation
5. **Red**: Test uppercase requirement
6. **Green**: Add uppercase checking
7. **Red**: Test lowercase requirement
8. **Green**: Add lowercase checking
9. **Red**: Test digit requirement
10. **Green**: Add digit checking
11. **Red**: Test special character requirement
12. **Green**: Add special character checking
13. **Red**: Test validation result object
14. **Green**: Return detailed validation results
15. **Red**: Test suggestions generation
16. **Green**: Implement suggestion system
17. **Refactor**: Clean up and optimize

## Learning Goals

- String analysis and pattern matching
- Regular expressions
- Object-oriented design
- Comprehensive input validation
- User feedback systems
- Security concepts

## Extension Ideas

1. **Configurable Rules**: Allow custom password requirements
2. **Strength Scoring**: Implement password strength scoring algorithm
3. **Common Password Detection**: Check against dictionary of common passwords
4. **Historical Password Tracking**: Prevent password reuse
5. **Multi-language Support**: Support international characters

---

## 问题描述

创建一个密码验证系统，检查密码是否符合指定的安全标准。

## 密码要求

有效密码必须：
1. 至少8个字符长
2. 包含至少一个大写字母（A-Z）
3. 包含至少一个小写字母（a-z）
4. 包含至少一个数字（0-9）
5. 包含至少一个特殊字符（!@#$%^&*）
6. 不包含常见的弱密码模式

## 需求

### PasswordValidator 类
- `validate(password)` - 返回验证结果对象
- `isValid(password)` - 返回布尔值
- `getFailedCriteria(password)` - 返回失败标准列表
- `getSuggestions(password)` - 返回改进建议

### 验证结果
```javascript
{
  isValid: boolean,
  failedCriteria: string[],
  suggestions: string[],
  strength: "weak" | "medium" | "strong"
}
```

## 示例

```javascript
validator = new PasswordValidator()

validator.validate("Pass123!")
→ {
    isValid: true,
    failedCriteria: [],
    suggestions: [],
    strength: "strong"
  }

validator.validate("password")
→ {
    isValid: false,
    failedCriteria: ["no_uppercase", "no_digit", "no_special_char"],
    suggestions: ["添加大写字母", "添加数字", "添加特殊字符"],
    strength: "weak"
  }

validator.validate("Pass1")
→ {
    isValid: false,
    failedCriteria: ["too_short", "no_special_char"],
    suggestions: ["使用至少8个字符", "添加特殊字符"],
    strength: "weak"
  }
```

## 建议测试用例

### 基本验证
1. 有效密码："Password123!", "MySecure@Pass1", "StrongP@ssw0rd"
2. 太短："Pass1!", "123", "abC!"
3. 无大写字母："password123!", "mypass@123"
4. 无小写字母："PASSWORD123!", "MYPASS@123"
5. 无数字："Password!", "MyPassw@rd"
6. 无特殊字符："Password123", "MyPassword123"

### 高级验证
1. 常见弱密码模式："Password123", "qwerty123", "abc123"
2. 连续模式："123456789", "abcdefgh"
3. 重复字符："aaaaaa", "111111"
4. 空/空密码
5. 很长的密码（测试性能）

### 边界情况
1. Unicode字符
2. 空格处理
3. 大小写敏感性
4. 特殊字符边界情况

## TDD 步骤

1. **红灯**: 测试基本有效密码
2. **绿灯**: 实现返回true的基本结构
3. **红灯**: 测试最小长度要求
4. **绿灯**: 添加长度验证
5. **红灯**: 测试大写字母要求
6. **绿灯**: 添加大写字母检查
7. **红灯**: 测试小写字母要求
8. **绿灯**: 添加小写字母检查
9. **红灯**: 测试数字要求
10. **绿灯**: 添加数字检查
11. **红灯**: 测试特殊字符要求
12. **绿灯**: 添加特殊字符检查
13. **红灯**: 测试验证结果对象
14. **绿灯**: 返回详细验证结果
15. **红灯**: 测试建议生成
16. **绿灯**: 实现建议系统
17. **重构**: 清理和优化

## 学习目标

- 字符串分析和模式匹配
- 正则表达式
- 面向对象设计
- 全面输入验证
- 用户反馈系统
- 安全概念

## 扩展想法

1. **可配置规则**: 允许自定义密码要求
2. **强度评分**: 实现密码强度评分算法
3. **常见密码检测**: 对照常见密码字典检查
4. **历史密码跟踪**: 防止密码重用
5. **多语言支持**: 支持国际字符