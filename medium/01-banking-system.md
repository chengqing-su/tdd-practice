# Banking System

## Problem Description

Design a simple banking system with the following features:
- Create bank accounts with initial balance
- Deposit and withdraw money
- Transfer money between accounts
- Track transaction history
- Handle insufficient funds and other error conditions

## Requirements

### Account Class
- `Account(accountId, initialBalance)` - Create account with ID and initial balance
- `deposit(amount)` - Add money to account
- `withdraw(amount)` - Remove money from account (check sufficient funds)
- `getBalance()` - Return current balance
- `getTransactionHistory()` - Return list of all transactions

### Bank Class
- `createAccount(accountId, initialBalance)` - Create and store new account
- `transfer(fromAccountId, toAccountId, amount)` - Transfer between accounts
- `getAccount(accountId)` - Retrieve account by ID

### Transaction Tracking
Each transaction should record:
- Transaction type (deposit, withdrawal, transfer)
- Amount
- Timestamp
- From/to account (for transfers)

## Examples

```javascript
bank = new Bank()
account1 = bank.createAccount("ACC001", 1000)
account2 = bank.createAccount("ACC002", 500)

account1.deposit(200)  // Balance: 1200
account1.withdraw(100) // Balance: 1100
bank.transfer("ACC001", "ACC002", 300) // ACC001: 800, ACC002: 800

account1.withdraw(1000) // Should throw InsufficientFundsError
```

## Suggested Test Cases

### Account Tests
1. Create account with initial balance
2. Deposit positive amount increases balance
3. Withdraw with sufficient funds decreases balance
4. Withdraw with insufficient funds throws error
5. Withdraw negative amount throws error
6. Transaction history tracks all operations

### Bank Tests
1. Create multiple accounts
2. Retrieve account by ID
3. Transfer between existing accounts
4. Transfer with insufficient funds fails
5. Transfer to non-existent account fails
6. Transfer records transactions in both accounts

## TDD Steps

1. **Red**: Test account creation with initial balance
2. **Green**: Implement Account constructor
3. **Red**: Test deposit functionality
4. **Green**: Implement deposit method
5. **Red**: Test withdrawal with sufficient funds
6. **Green**: Implement basic withdraw method
7. **Red**: Test withdrawal with insufficient funds
8. **Green**: Add error checking to withdraw
9. **Refactor**: Clean up Account class
10. **Repeat**: Continue with Bank class and transfers

## Learning Goals

- Object-oriented design
- Error handling and custom exceptions
- Data validation
- State management
- Inter-object communication
- Complex business logic testing

---

## 问题描述

设计一个简单的银行系统，包含以下功能：
- 创建带有初始余额的银行账户
- 存款和取款
- 账户间转账
- 跟踪交易历史
- 处理余额不足和其他错误情况

## 需求

### Account 类
- `Account(accountId, initialBalance)` - 用ID和初始余额创建账户
- `deposit(amount)` - 向账户存钱
- `withdraw(amount)` - 从账户取钱（检查余额是否充足）
- `getBalance()` - 返回当前余额
- `getTransactionHistory()` - 返回所有交易记录列表

### Bank 类
- `createAccount(accountId, initialBalance)` - 创建并存储新账户
- `transfer(fromAccountId, toAccountId, amount)` - 账户间转账
- `getAccount(accountId)` - 通过ID检索账户

### 交易跟踪
每个交易应记录：
- 交易类型（存款、取款、转账）
- 金额
- 时间戳
- 源/目标账户（转账时）

## 示例

```javascript
bank = new Bank()
account1 = bank.createAccount("ACC001", 1000)
account2 = bank.createAccount("ACC002", 500)

account1.deposit(200)  // 余额: 1200
account1.withdraw(100) // 余额: 1100
bank.transfer("ACC001", "ACC002", 300) // ACC001: 800, ACC002: 800

account1.withdraw(1000) // 应抛出 InsufficientFundsError
```

## 建议测试用例

### Account 测试
1. 用初始余额创建账户
2. 存入正数增加余额
3. 余额充足时取款减少余额
4. 余额不足时取款抛出错误
5. 取负数时抛出错误
6. 交易历史跟踪所有操作

### Bank 测试
1. 创建多个账户
2. 通过ID检索账户
3. 现有账户间转账
4. 余额不足时转账失败
5. 转账到不存在的账户失败
6. 转账在两个账户中都记录交易

## TDD 步骤

1. **红灯**: 测试用初始余额创建账户
2. **绿灯**: 实现 Account 构造函数
3. **红灯**: 测试存款功能
4. **绿灯**: 实现存款方法
5. **红灯**: 测试余额充足时的取款
6. **绿灯**: 实现基本取款方法
7. **红灯**: 测试余额不足时的取款
8. **绿灯**: 为取款添加错误检查
9. **重构**: 清理 Account 类
10. **重复**: 继续 Bank 类和转账功能

## 学习目标

- 面向对象设计
- 错误处理和自定义异常
- 数据验证
- 状态管理
- 对象间通信
- 复杂业务逻辑测试