# Library Management System

## Problem Description

Design a library management system that handles books, members, borrowing, and returning with due dates, fines, and reservations.

## Requirements

### Book Class
- `Book(isbn, title, author, publishYear, copies)` - Create book with metadata
- `isAvailable()` - Check if any copies are available
- `getAvailableCopies()` - Get number of available copies
- `borrowCopy()` - Mark a copy as borrowed
- `returnCopy()` - Mark a copy as returned

### Member Class
- `Member(memberId, name, email, membershipType)` - Create library member
- `canBorrow()` - Check if member can borrow more books
- `getBorrowedBooks()` - Get list of currently borrowed books
- `getOverdueBooks()` - Get list of overdue books
- `calculateFines()` - Calculate total outstanding fines

### Loan Class
- `Loan(book, member, borrowDate, dueDate)` - Create loan record
- `isOverdue()` - Check if loan is past due date
- `getDaysOverdue()` - Calculate days overdue
- `calculateFine()` - Calculate fine amount for overdue loan
- `markReturned()` - Mark loan as returned

### Library Class
- `addBook(book)` - Add book to library catalog
- `addMember(member)` - Register new member
- `borrowBook(memberId, isbn)` - Process book borrowing
- `returnBook(memberId, isbn)` - Process book return
- `reserveBook(memberId, isbn)` - Reserve unavailable book
- `getOverdueLoans()` - Get all overdue loans
- `getMemberHistory(memberId)` - Get member's borrowing history

### Business Rules
- **Loan Period**: 14 days for regular members, 21 days for premium members
- **Borrowing Limits**: 3 books for regular, 5 books for premium members
- **Fines**: $0.50 per day overdue
- **Reservations**: Members can reserve books that are currently borrowed

## Examples

```python
# Setup library
library = Library()
book1 = Book("978-0-123456-47-2", "The Great Gatsby", "F. Scott Fitzgerald", 1925, 3)
member1 = Member("M001", "John Doe", "john@email.com", MembershipType.REGULAR)

library.add_book(book1)
library.add_member(member1)

# Borrow book
loan = library.borrow_book("M001", "978-0-123456-47-2")
print(loan.due_date)  # 14 days from today for regular member

# Check availability
print(book1.get_available_copies())  # 2 (was 3, now 1 borrowed)

# Return book after due date
library.return_book("M001", "978-0-123456-47-2")
fine = member1.calculate_fines()  # Calculate overdue fines

# Reservation system
library.reserve_book("M002", "978-0-123456-47-2")  # Reserve if not available
```

## Suggested Test Cases

### Book Management
1. Create book with multiple copies
2. Check availability when all copies borrowed
3. Borrow and return copies
4. Handle borrowing when no copies available

### Member Management
1. Create members with different membership types
2. Check borrowing limits
3. Track borrowed books
4. Calculate member fines

### Loan Processing
1. Create loan with correct due date based on membership
2. Check overdue status
3. Calculate overdue days and fines
4. Mark loans as returned

### Library Operations
1. Process successful book borrowing
2. Handle borrowing when book unavailable
3. Handle borrowing when member at limit
4. Process book returns and update availability
5. Handle returns of overdue books
6. Reservation system functionality

### Business Logic
1. Different loan periods for membership types
2. Borrowing limit enforcement
3. Fine calculation accuracy
4. Overdue loan identification

## TDD Steps

1. **Red**: Test Book creation with copies
2. **Green**: Implement Book class with basic properties
3. **Red**: Test book availability checking
4. **Green**: Implement availability logic
5. **Red**: Test Member creation and limits
6. **Green**: Implement Member class
7. **Red**: Test Loan creation with due dates
8. **Green**: Implement Loan class with date logic
9. **Red**: Test Library book borrowing
10. **Green**: Implement borrowing process
11. **Red**: Test overdue calculations
12. **Green**: Implement overdue and fine logic
13. **Red**: Test return processing
14. **Green**: Implement return functionality
15. **Red**: Test reservation system
16. **Green**: Implement reservation logic
17. **Refactor**: Optimize and clean up all classes

## Learning Goals

- Complex domain modeling
- Date and time handling
- Business rule implementation
- State management across multiple objects
- Financial calculations
- Member/user management systems
- Inventory tracking

## Extension Ideas

1. **Renewal System**: Allow members to renew loans
2. **Hold Queue**: Priority queue for book reservations
3. **Late Fees**: Different fine structures
4. **Membership Tiers**: More complex membership benefits
5. **Digital Books**: Handle e-books with different rules
6. **Multi-branch Support**: Books across multiple library locations

---

## 问题描述

设计一个图书馆管理系统，处理图书、会员、借阅和归还，包括到期日期、罚款和预约功能。

## 需求

### Book 类
- `Book(isbn, title, author, publishYear, copies)` - 创建带元数据的图书
- `isAvailable()` - 检查是否有副本可用
- `getAvailableCopies()` - 获取可用副本数量
- `borrowCopy()` - 标记副本为已借出
- `returnCopy()` - 标记副本为已归还

### Member 类
- `Member(memberId, name, email, membershipType)` - 创建图书馆会员
- `canBorrow()` - 检查会员是否可以借更多书
- `getBorrowedBooks()` - 获取当前借阅图书列表
- `getOverdueBooks()` - 获取逾期图书列表
- `calculateFines()` - 计算总未缴罚款

### Loan 类
- `Loan(book, member, borrowDate, dueDate)` - 创建借阅记录
- `isOverdue()` - 检查借阅是否过期
- `getDaysOverdue()` - 计算逾期天数
- `calculateFine()` - 计算逾期借阅的罚款金额
- `markReturned()` - 标记借阅为已归还

### Library 类
- `addBook(book)` - 添加图书到图书馆目录
- `addMember(member)` - 注册新会员
- `borrowBook(memberId, isbn)` - 处理图书借阅
- `returnBook(memberId, isbn)` - 处理图书归还
- `reserveBook(memberId, isbn)` - 预约不可用的图书
- `getOverdueLoans()` - 获取所有逾期借阅
- `getMemberHistory(memberId)` - 获取会员借阅历史

### 业务规则
- **借阅期限**: 普通会员14天，高级会员21天
- **借阅限制**: 普通会员3本，高级会员5本
- **罚款**: 每天逾期0.50美元
- **预约**: 会员可以预约当前已借出的图书

## 示例

```python
# 设置图书馆
library = Library()
book1 = Book("978-0-123456-47-2", "了不起的盖茨比", "F. Scott Fitzgerald", 1925, 3)
member1 = Member("M001", "张三", "zhang@email.com", MembershipType.REGULAR)

library.add_book(book1)
library.add_member(member1)

# 借书
loan = library.borrow_book("M001", "978-0-123456-47-2")
print(loan.due_date)  # 普通会员从今天算起14天

# 检查可用性
print(book1.get_available_copies())  # 2（原来3本，现在借出1本）

# 过期后还书
library.return_book("M001", "978-0-123456-47-2")
fine = member1.calculate_fines()  # 计算逾期罚款

# 预约系统
library.reserve_book("M002", "978-0-123456-47-2")  # 不可用时预约
```

## 建议测试用例

### 图书管理
1. 创建有多个副本的图书
2. 所有副本借出时检查可用性
3. 借出和归还副本
4. 处理无副本可用时的借阅

### 会员管理
1. 创建不同会员类型的会员
2. 检查借阅限制
3. 跟踪借阅图书
4. 计算会员罚款

### 借阅处理
1. 根据会员类型创建正确到期日的借阅
2. 检查逾期状态
3. 计算逾期天数和罚款
4. 标记借阅为已归还

### 图书馆操作
1. 处理成功的图书借阅
2. 处理图书不可用时的借阅
3. 处理会员达到限制时的借阅
4. 处理图书归还和更新可用性
5. 处理逾期图书的归还
6. 预约系统功能

### 业务逻辑
1. 不同会员类型的借阅期限
2. 借阅限制执行
3. 罚款计算准确性
4. 逾期借阅识别

## TDD 步骤

1. **红灯**: 测试创建有副本的图书
2. **绿灯**: 实现带基本属性的图书类
3. **红灯**: 测试图书可用性检查
4. **绿灯**: 实现可用性逻辑
5. **红灯**: 测试会员创建和限制
6. **绿灯**: 实现会员类
7. **红灯**: 测试带到期日的借阅创建
8. **绿灯**: 实现带日期逻辑的借阅类
9. **红灯**: 测试图书馆图书借阅
10. **绿灯**: 实现借阅过程
11. **红灯**: 测试逾期计算
12. **绿灯**: 实现逾期和罚款逻辑
13. **红灯**: 测试归还处理
14. **绿灯**: 实现归还功能
15. **红灯**: 测试预约系统
16. **绿灯**: 实现预约逻辑
17. **重构**: 优化和清理所有类

## 学习目标

- 复杂领域建模
- 日期和时间处理
- 业务规则实现
- 多对象间状态管理
- 财务计算
- 会员/用户管理系统
- 库存跟踪

## 扩展想法

1. **续借系统**: 允许会员续借
2. **预约队列**: 图书预约优先队列
3. **滞纳金**: 不同罚款结构
4. **会员等级**: 更复杂的会员权益
5. **电子书**: 处理不同规则的电子书
6. **多分馆支持**: 跨多个图书馆位置的图书