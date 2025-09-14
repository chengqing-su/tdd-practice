# Shopping Cart System

## Problem Description

Build an e-commerce shopping cart system that handles products, quantities, discounts, taxes, and checkout calculations.

## Requirements

### Product Class
- `Product(id, name, price, category, inStock)` - Create product
- `getPrice()` - Get product price
- `isAvailable()` - Check if product is in stock

### CartItem Class
- `CartItem(product, quantity)` - Create cart item with product and quantity
- `getTotalPrice()` - Calculate total price for this item
- `updateQuantity(newQuantity)` - Update item quantity

### ShoppingCart Class
- `addItem(product, quantity)` - Add product to cart
- `removeItem(productId)` - Remove product from cart
- `updateItemQuantity(productId, quantity)` - Update quantity of existing item
- `getItemCount()` - Get total number of different items
- `getTotalQuantity()` - Get total quantity of all items
- `getSubtotal()` - Get subtotal before discounts and tax
- `applyDiscount(discountPercentage)` - Apply percentage discount
- `calculateTax(taxRate)` - Calculate tax on current total
- `getTotal()` - Get final total with discounts and tax
- `clear()` - Empty the cart

### Checkout Class
- `Checkout(cart)` - Initialize with shopping cart
- `calculateShipping(address)` - Calculate shipping cost based on location
- `processPayment(paymentMethod, amount)` - Process payment
- `generateInvoice()` - Generate invoice with all details

## Examples

```java
Product laptop = new Product("P001", "Gaming Laptop", 1200.00, "Electronics", true);
Product mouse = new Product("P002", "Wireless Mouse", 25.00, "Electronics", true);

ShoppingCart cart = new ShoppingCart();
cart.addItem(laptop, 1);
cart.addItem(mouse, 2);

System.out.println(cart.getSubtotal()); // 1250.00
cart.applyDiscount(10); // 10% discount
System.out.println(cart.getTotal()); // 1125.00

cart.calculateTax(8.5); // 8.5% tax
System.out.println(cart.getTotal()); // 1220.625

Checkout checkout = new Checkout(cart);
checkout.calculateShipping("USA"); // Additional shipping cost
Invoice invoice = checkout.generateInvoice();
```

## Suggested Test Cases

### Product Tests
1. Create product with all properties
2. Check price and availability
3. Handle out-of-stock products

### CartItem Tests
1. Create cart item with product and quantity
2. Calculate total price for multiple quantities
3. Update quantity and verify price changes
4. Handle zero and negative quantities

### ShoppingCart Tests
1. Add single and multiple items
2. Remove items from cart
3. Update item quantities
4. Calculate subtotal correctly
5. Apply percentage discounts
6. Calculate tax on discounted total
7. Handle adding same product multiple times
8. Clear cart functionality

### Checkout Tests
1. Initialize checkout with cart
2. Calculate shipping for different locations
3. Process payment with sufficient/insufficient funds
4. Generate complete invoice with all calculations

### Edge Cases
1. Empty cart calculations
2. Adding out-of-stock products
3. Invalid discount percentages
4. Negative tax rates
5. Payment processing failures

## TDD Steps

1. **Red**: Test Product creation
2. **Green**: Implement Product class
3. **Red**: Test CartItem total price calculation
4. **Green**: Implement CartItem class
5. **Red**: Test adding items to cart
6. **Green**: Implement basic ShoppingCart add functionality
7. **Red**: Test subtotal calculation
8. **Green**: Implement subtotal calculation
9. **Refactor**: Clean up cart calculations
10. **Red**: Test discount application
11. **Green**: Implement discount logic
12. **Red**: Test tax calculation
13. **Green**: Implement tax logic
14. **Repeat**: Continue with checkout functionality

## Learning Goals

- Complex business logic implementation
- Money/decimal precision handling
- State management across multiple objects
- Validation and error handling
- Mathematical calculations in code
- Integration testing between components

---

## 问题描述

构建一个电子商务购物车系统，处理商品、数量、折扣、税费和结账计算。

## 需求

### Product 类
- `Product(id, name, price, category, inStock)` - 创建商品
- `getPrice()` - 获取商品价格
- `isAvailable()` - 检查商品是否有库存

### CartItem 类
- `CartItem(product, quantity)` - 创建带商品和数量的购物车项目
- `getTotalPrice()` - 计算此项目的总价
- `updateQuantity(newQuantity)` - 更新项目数量

### ShoppingCart 类
- `addItem(product, quantity)` - 添加商品到购物车
- `removeItem(productId)` - 从购物车移除商品
- `updateItemQuantity(productId, quantity)` - 更新现有项目的数量
- `getItemCount()` - 获取不同商品的总数
- `getTotalQuantity()` - 获取所有商品的总数量
- `getSubtotal()` - 获取折扣和税费前的小计
- `applyDiscount(discountPercentage)` - 应用百分比折扣
- `calculateTax(taxRate)` - 计算当前总额的税费
- `getTotal()` - 获取含折扣和税费的最终总额
- `clear()` - 清空购物车

### Checkout 类
- `Checkout(cart)` - 用购物车初始化
- `calculateShipping(address)` - 根据地址计算运费
- `processPayment(paymentMethod, amount)` - 处理付款
- `generateInvoice()` - 生成包含所有详情的发票

## 示例

```java
Product laptop = new Product("P001", "游戏笔记本", 1200.00, "电子产品", true);
Product mouse = new Product("P002", "无线鼠标", 25.00, "电子产品", true);

ShoppingCart cart = new ShoppingCart();
cart.addItem(laptop, 1);
cart.addItem(mouse, 2);

System.out.println(cart.getSubtotal()); // 1250.00
cart.applyDiscount(10); // 10% 折扣
System.out.println(cart.getTotal()); // 1125.00

cart.calculateTax(8.5); // 8.5% 税费
System.out.println(cart.getTotal()); // 1220.625

Checkout checkout = new Checkout(cart);
checkout.calculateShipping("中国"); // 额外运费
Invoice invoice = checkout.generateInvoice();
```

## 建议测试用例

### Product 测试
1. 创建带所有属性的商品
2. 检查价格和可用性
3. 处理缺货商品

### CartItem 测试
1. 创建带商品和数量的购物车项目
2. 计算多个数量的总价
3. 更新数量并验证价格变化
4. 处理零和负数量

### ShoppingCart 测试
1. 添加单个和多个商品
2. 从购物车移除商品
3. 更新商品数量
4. 正确计算小计
5. 应用百分比折扣
6. 计算折扣总额的税费
7. 处理多次添加相同商品
8. 清空购物车功能

### Checkout 测试
1. 用购物车初始化结账
2. 计算不同地址的运费
3. 处理余额充足/不足的付款
4. 生成包含所有计算的完整发票

### 边界情况
1. 空购物车计算
2. 添加缺货商品
3. 无效折扣百分比
4. 负税率
5. 付款处理失败

## TDD 步骤

1. **红灯**: 测试商品创建
2. **绿灯**: 实现商品类
3. **红灯**: 测试购物车项目总价计算
4. **绿灯**: 实现购物车项目类
5. **红灯**: 测试添加商品到购物车
6. **绿灯**: 实现基本购物车添加功能
7. **红灯**: 测试小计计算
8. **绿灯**: 实现小计计算
9. **重构**: 清理购物车计算
10. **红灯**: 测试折扣应用
11. **绿灯**: 实现折扣逻辑
12. **红灯**: 测试税费计算
13. **绿灯**: 实现税费逻辑
14. **重复**: 继续结账功能

## 学习目标

- 复杂业务逻辑实现
- 金钱/小数精度处理
- 多对象间状态管理
- 验证和错误处理
- 代码中的数学计算
- 组件间集成测试