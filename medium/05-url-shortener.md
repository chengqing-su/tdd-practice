# URL Shortener Service

## Problem Description

Build a URL shortening service (like bit.ly or tinyurl.com) that converts long URLs into short, shareable links and tracks usage analytics.

## Requirements

### ShortenedUrl Class
- `ShortenedUrl(originalUrl, shortCode, createdBy, expirationDate)` - Create URL mapping
- `isExpired()` - Check if URL has expired
- `incrementClickCount()` - Track when URL is accessed
- `getClickCount()` - Get total number of clicks
- `getCreationDate()` - Get when URL was created

### UrlShortener Class
- `shortenUrl(originalUrl, customAlias?, expirationDays?)` - Create short URL
- `expandUrl(shortCode)` - Get original URL from short code
- `deleteUrl(shortCode)` - Remove URL mapping
- `getUrlStats(shortCode)` - Get analytics for short URL
- `getUserUrls(userId)` - Get all URLs created by user
- `cleanupExpiredUrls()` - Remove expired URLs

### Analytics Class
- `recordClick(shortCode, userAgent, ipAddress, referrer)` - Track click
- `getClickStats(shortCode)` - Get detailed click statistics
- `getTopUrls(limit)` - Get most clicked URLs
- `getClicksByDate(shortCode, startDate, endDate)` - Time-based analytics

### URL Validation
- Validate URL format and accessibility
- Check for malicious URLs (basic blacklist)
- Prevent shortening of already shortened URLs
- Handle various URL formats (http, https, www, etc.)

## Features

### Short Code Generation
- Generate unique, URL-safe short codes
- Support custom aliases (if available)
- Avoid confusing characters (0/O, 1/l, etc.)
- Configurable code length

### Expiration Handling
- Optional expiration dates for URLs
- Automatic cleanup of expired URLs
- Grace period for recently expired URLs

### Analytics Tracking
- Click counting
- Geographic information (based on IP)
- Referrer tracking
- User agent analysis
- Time-based statistics

## Examples

```javascript
const shortener = new UrlShortener();

// Basic URL shortening
const shortUrl = shortener.shortenUrl("https://www.example.com/very/long/path/to/page");
console.log(shortUrl.shortCode); // "abc123"
console.log(shortUrl.fullShortUrl); // "https://short.ly/abc123"

// Custom alias
const customUrl = shortener.shortenUrl("https://example.com", "my-page");
console.log(customUrl.shortCode); // "my-page"

// With expiration
const tempUrl = shortener.shortenUrl("https://example.com", null, 7); // 7 days
console.log(tempUrl.expirationDate); // 7 days from now

// Expand URL
const originalUrl = shortener.expandUrl("abc123");
console.log(originalUrl); // "https://www.example.com/very/long/path/to/page"

// Analytics
const analytics = new Analytics();
analytics.recordClick("abc123", "Mozilla/5.0...", "192.168.1.1", "https://google.com");

const stats = analytics.getClickStats("abc123");
console.log(stats.totalClicks); // 1
console.log(stats.uniqueVisitors); // 1
console.log(stats.topReferrers); // ["https://google.com"]
```

## Suggested Test Cases

### URL Shortening
1. Shorten valid HTTP/HTTPS URLs
2. Handle URLs with query parameters and fragments
3. Generate unique short codes for same URL
4. Support custom aliases
5. Reject invalid URLs
6. Handle duplicate custom aliases

### URL Expansion
1. Expand existing short codes
2. Handle non-existent short codes
3. Handle expired URLs
4. Track clicks during expansion
5. Return appropriate error messages

### Short Code Generation
1. Generate URL-safe codes
2. Avoid character ambiguity
3. Handle collisions gracefully
4. Respect custom aliases
5. Generate codes of consistent length

### Expiration Management
1. Create URLs with expiration dates
2. Detect expired URLs
3. Cleanup expired URLs automatically
4. Handle edge cases around expiration time

### Analytics
1. Record click events accurately
2. Calculate unique visitors
3. Track referrer information
4. Generate time-based reports
5. Handle high-volume click tracking

### Validation
1. Validate URL formats
2. Check URL accessibility
3. Detect malicious URLs
4. Prevent recursive shortening

## TDD Steps

1. **Red**: Test basic URL shortening
2. **Green**: Implement simple URL to code mapping
3. **Red**: Test short code generation
4. **Green**: Implement unique code generator
5. **Red**: Test URL expansion
6. **Green**: Implement URL lookup and expansion
7. **Red**: Test URL validation
8. **Green**: Add URL format validation
9. **Red**: Test custom aliases
10. **Green**: Support custom short codes
11. **Red**: Test expiration functionality
12. **Green**: Implement expiration dates
13. **Red**: Test click tracking
14. **Green**: Implement basic analytics
15. **Red**: Test advanced analytics features
16. **Green**: Add detailed statistics
17. **Refactor**: Optimize storage and performance

## Learning Goals

- URL parsing and validation
- Unique ID generation algorithms
- Data persistence and retrieval
- Analytics and statistics calculation
- Rate limiting and abuse prevention
- Time-based data management
- User authentication and authorization
- Performance optimization for high-volume systems

## Advanced Features

1. **Rate Limiting**: Prevent abuse with request rate limits
2. **Bulk Operations**: Shorten multiple URLs at once
3. **QR Codes**: Generate QR codes for short URLs
4. **A/B Testing**: Support multiple versions of landing pages
5. **Geographic Restrictions**: Limit access by location
6. **Password Protection**: Secure URLs with passwords
7. **Preview Mode**: Show URL preview before redirect

---

## 问题描述

构建URL缩短服务（类似bit.ly或tinyurl.com），将长URL转换为短的可分享链接并跟踪使用分析。

## 需求

### ShortenedUrl 类
- `ShortenedUrl(originalUrl, shortCode, createdBy, expirationDate)` - 创建URL映射
- `isExpired()` - 检查URL是否已过期
- `incrementClickCount()` - 跟踪URL访问时
- `getClickCount()` - 获取总点击次数
- `getCreationDate()` - 获取URL创建时间

### UrlShortener 类
- `shortenUrl(originalUrl, customAlias?, expirationDays?)` - 创建短URL
- `expandUrl(shortCode)` - 从短代码获取原始URL
- `deleteUrl(shortCode)` - 删除URL映射
- `getUrlStats(shortCode)` - 获取短URL的分析
- `getUserUrls(userId)` - 获取用户创建的所有URL
- `cleanupExpiredUrls()` - 清理过期URL

### Analytics 类
- `recordClick(shortCode, userAgent, ipAddress, referrer)` - 跟踪点击
- `getClickStats(shortCode)` - 获取详细点击统计
- `getTopUrls(limit)` - 获取点击最多的URL
- `getClicksByDate(shortCode, startDate, endDate)` - 基于时间的分析

### URL 验证
- 验证URL格式和可访问性
- 检查恶意URL（基本黑名单）
- 防止缩短已经缩短的URL
- 处理各种URL格式（http, https, www等）

## 功能

### 短代码生成
- 生成唯一的URL安全短代码
- 支持自定义别名（如果可用）
- 避免令人困惑的字符（0/O, 1/l等）
- 可配置代码长度

### 过期处理
- URL的可选过期日期
- 自动清理过期URL
- 最近过期URL的宽限期

### 分析跟踪
- 点击计数
- 地理信息（基于IP）
- 来源跟踪
- 用户代理分析
- 基于时间的统计

## 示例

```javascript
const shortener = new UrlShortener();

// 基本URL缩短
const shortUrl = shortener.shortenUrl("https://www.example.com/very/long/path/to/page");
console.log(shortUrl.shortCode); // "abc123"
console.log(shortUrl.fullShortUrl); // "https://short.ly/abc123"

// 自定义别名
const customUrl = shortener.shortenUrl("https://example.com", "my-page");
console.log(customUrl.shortCode); // "my-page"

// 带过期时间
const tempUrl = shortener.shortenUrl("https://example.com", null, 7); // 7天
console.log(tempUrl.expirationDate); // 从现在算起7天

// 展开URL
const originalUrl = shortener.expandUrl("abc123");
console.log(originalUrl); // "https://www.example.com/very/long/path/to/page"

// 分析
const analytics = new Analytics();
analytics.recordClick("abc123", "Mozilla/5.0...", "192.168.1.1", "https://google.com");

const stats = analytics.getClickStats("abc123");
console.log(stats.totalClicks); // 1
console.log(stats.uniqueVisitors); // 1
console.log(stats.topReferrers); // ["https://google.com"]
```

## 建议测试用例

### URL缩短
1. 缩短有效的HTTP/HTTPS URL
2. 处理带查询参数和片段的URL
3. 为相同URL生成唯一短代码
4. 支持自定义别名
5. 拒绝无效URL
6. 处理重复的自定义别名

### URL展开
1. 展开现有短代码
2. 处理不存在的短代码
3. 处理过期URL
4. 展开期间跟踪点击
5. 返回适当的错误消息

### 短代码生成
1. 生成URL安全代码
2. 避免字符歧义
3. 优雅处理冲突
4. 尊重自定义别名
5. 生成一致长度的代码

### 过期管理
1. 创建带过期日期的URL
2. 检测过期URL
3. 自动清理过期URL
4. 处理过期时间边界情况

### 分析
1. 准确记录点击事件
2. 计算独特访问者
3. 跟踪来源信息
4. 生成基于时间的报告
5. 处理高容量点击跟踪

### 验证
1. 验证URL格式
2. 检查URL可访问性
3. 检测恶意URL
4. 防止递归缩短

## TDD 步骤

1. **红灯**: 测试基本URL缩短
2. **绿灯**: 实现简单URL到代码映射
3. **红灯**: 测试短代码生成
4. **绿灯**: 实现唯一代码生成器
5. **红灯**: 测试URL展开
6. **绿灯**: 实现URL查找和展开
7. **红灯**: 测试URL验证
8. **绿灯**: 添加URL格式验证
9. **红灯**: 测试自定义别名
10. **绿灯**: 支持自定义短代码
11. **红灯**: 测试过期功能
12. **绿灯**: 实现过期日期
13. **红灯**: 测试点击跟踪
14. **绿灯**: 实现基本分析
15. **红灯**: 测试高级分析功能
16. **绿灯**: 添加详细统计
17. **重构**: 优化存储和性能

## 学习目标

- URL解析和验证
- 唯一ID生成算法
- 数据持久化和检索
- 分析和统计计算
- 速率限制和滥用防护
- 基于时间的数据管理
- 用户认证和授权
- 高容量系统性能优化

## 高级功能

1. **速率限制**: 用请求速率限制防止滥用
2. **批量操作**: 一次缩短多个URL
3. **二维码**: 为短URL生成二维码
4. **A/B测试**: 支持登录页面的多个版本
5. **地理限制**: 按位置限制访问
6. **密码保护**: 用密码保护URL
7. **预览模式**: 重定向前显示URL预览