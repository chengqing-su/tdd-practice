# Chat System

## Problem Description

Build a real-time chat system that supports multiple chat rooms, private messaging, user presence, message history, and notifications.

## Requirements

### User Class
- `User(userId, username, email, status)` - Create user account
- `setStatus(status)` - Set online/offline/away status
- `getLastSeen()` - Get last activity timestamp
- `blockUser(userId)` - Block another user
- `unblockUser(userId)` - Unblock user

### Message Class
- `Message(messageId, sender, content, timestamp, messageType)` - Create message
- `edit(newContent)` - Edit message content
- `delete()` - Mark message as deleted
- `addReaction(userId, emoji)` - Add emoji reaction
- `removeReaction(userId, emoji)` - Remove emoji reaction
- Message types: TEXT, IMAGE, FILE, SYSTEM

### ChatRoom Class
- `ChatRoom(roomId, name, description, roomType, creator)` - Create chat room
- `addMember(userId, role?)` - Add user to room
- `removeMember(userId)` - Remove user from room
- `sendMessage(message)` - Send message to room
- `getMessages(limit?, before?)` - Get message history
- `setPermissions(userId, permissions)` - Set user permissions
- Room types: PUBLIC, PRIVATE, DIRECT_MESSAGE

### ChatService Class
- `createRoom(name, type, creator)` - Create new chat room
- `joinRoom(userId, roomId)` - Join existing room
- `leaveRoom(userId, roomId)` - Leave room
- `sendMessage(userId, roomId, content, type?)` - Send message
- `getMessageHistory(roomId, pagination)` - Get room messages
- `searchMessages(query, roomId?, filters?)` - Search messages
- `getUserRooms(userId)` - Get user's chat rooms

### Notification System
- `NotificationService` - Handle push notifications
- `subscribe(userId, notificationType)` - Subscribe to notifications
- `sendNotification(userId, notification)` - Send notification
- `markAsRead(userId, notificationId)` - Mark notification read
- Notification types: MESSAGE, MENTION, ROOM_INVITE

### Presence System
- `PresenceService` - Track user online status
- `updatePresence(userId, status)` - Update user status
- `getUserPresence(userId)` - Get user's current status
- `getOnlineUsers(roomId?)` - Get online users in room
- Status types: ONLINE, AWAY, BUSY, OFFLINE

## Features

### Real-time Messaging
- Instant message delivery
- Message delivery confirmation
- Read receipts
- Typing indicators
- Message reactions and replies

### Room Management
- Create public and private rooms
- Room member management
- Admin and moderator roles
- Room settings and permissions
- Room search and discovery

### Message Features
- Text formatting (bold, italic, mentions)
- File and image sharing
- Message editing and deletion
- Message search and filtering
- Message threading (replies)

### User Features
- User profiles and avatars
- User blocking and reporting
- Custom status messages
- Do not disturb mode
- Message encryption (optional)

## Examples

```javascript
const chatService = new ChatService();
const user1 = new User("u1", "alice", "alice@example.com", UserStatus.ONLINE);
const user2 = new User("u2", "bob", "bob@example.com", UserStatus.ONLINE);

// Create chat room
const room = chatService.createRoom("General Discussion", RoomType.PUBLIC, user1.getId());

// Join room
chatService.joinRoom(user2.getId(), room.getId());

// Send messages
const message1 = chatService.sendMessage(user1.getId(), room.getId(), "Hello everyone!");
const message2 = chatService.sendMessage(user2.getId(), room.getId(), "Hi Alice!");

// Add reaction
message1.addReaction(user2.getId(), "👍");

// Get message history
const messages = chatService.getMessageHistory(room.getId(),
    { limit: 50, before: new Date() });

// Private messaging
const dmRoom = chatService.createDirectMessage(user1.getId(), user2.getId());
chatService.sendMessage(user1.getId(), dmRoom.getId(), "Hey Bob, how are you?");

// Presence updates
const presence = new PresenceService();
presence.updatePresence(user1.getId(), UserStatus.AWAY);

// Notifications
const notifications = new NotificationService();
notifications.subscribe(user2.getId(), NotificationType.MENTION);
```

## Suggested Test Cases

### User Management
1. Create users with different statuses
2. Update user presence status
3. Handle user blocking functionality
4. Track last seen timestamps
5. Manage user profiles

### Message Operations
1. Send messages to chat rooms
2. Edit and delete messages
3. Add and remove reactions
4. Handle different message types
5. Message validation and sanitization

### Chat Room Management
1. Create different types of rooms
2. Add and remove room members
3. Set room permissions and roles
4. Handle room visibility settings
5. Room member limits and validation

### Message History
1. Retrieve message history with pagination
2. Search messages by content and filters
3. Sort messages by timestamp
4. Handle large message volumes
5. Message persistence and storage

### Real-time Features
1. Message broadcasting to room members
2. Presence status broadcasting
3. Typing indicator management
4. Connection handling and reconnection
5. Message delivery confirmation

### Notification System
1. Send notifications for mentions
2. Handle notification preferences
3. Mark notifications as read
4. Batch notification delivery
5. Push notification integration

### Security and Moderation
1. User permission validation
2. Message content filtering
3. Rate limiting for message sending
4. User reporting and blocking
5. Admin moderation tools

## TDD Steps

1. **Red**: Test user creation and basic properties
2. **Green**: Implement basic User class
3. **Red**: Test message creation and content
4. **Green**: Implement Message class
5. **Red**: Test chat room creation
6. **Green**: Implement ChatRoom class
7. **Red**: Test sending messages to rooms
8. **Green**: Implement message sending
9. **Red**: Test joining and leaving rooms
10. **Green**: Implement room membership
11. **Red**: Test message history retrieval
12. **Green**: Implement message persistence
13. **Red**: Test user presence tracking
14. **Green**: Implement presence service
15. **Red**: Test notification system
16. **Green**: Implement notifications
17. **Red**: Test message reactions and editing
18. **Green**: Implement message features
19. **Refactor**: Optimize performance and scalability

## Learning Goals

- Real-time communication patterns
- WebSocket or similar protocols
- Message queue systems
- User state management
- Scalable architecture design
- Data persistence strategies
- Push notification systems
- Security and authentication

## Advanced Features

1. **Voice/Video Calls**: WebRTC integration
2. **Message Encryption**: End-to-end encryption
3. **Bot Integration**: Chatbot API support
4. **File Sharing**: Advanced file upload/download
5. **Screen Sharing**: Share screen in chat
6. **Message Translation**: Multi-language support
7. **Chat Analytics**: Usage statistics and insights

---

## 问题描述

构建实时聊天系统，支持多个聊天室、私信、用户在线状态、消息历史和通知。

## 需求

### User 类
- `User(userId, username, email, status)` - 创建用户账户
- `setStatus(status)` - 设置在线/离线/离开状态
- `getLastSeen()` - 获取最后活动时间戳
- `blockUser(userId)` - 屏蔽其他用户
- `unblockUser(userId)` - 取消屏蔽用户

### Message 类
- `Message(messageId, sender, content, timestamp, messageType)` - 创建消息
- `edit(newContent)` - 编辑消息内容
- `delete()` - 标记消息为已删除
- `addReaction(userId, emoji)` - 添加表情反应
- `removeReaction(userId, emoji)` - 移除表情反应
- 消息类型：文本、图片、文件、系统

### ChatRoom 类
- `ChatRoom(roomId, name, description, roomType, creator)` - 创建聊天室
- `addMember(userId, role?)` - 添加用户到房间
- `removeMember(userId)` - 从房间移除用户
- `sendMessage(message)` - 发送消息到房间
- `getMessages(limit?, before?)` - 获取消息历史
- `setPermissions(userId, permissions)` - 设置用户权限
- 房间类型：公开、私人、直接消息

### ChatService 类
- `createRoom(name, type, creator)` - 创建新聊天室
- `joinRoom(userId, roomId)` - 加入现有房间
- `leaveRoom(userId, roomId)` - 离开房间
- `sendMessage(userId, roomId, content, type?)` - 发送消息
- `getMessageHistory(roomId, pagination)` - 获取房间消息
- `searchMessages(query, roomId?, filters?)` - 搜索消息
- `getUserRooms(userId)` - 获取用户的聊天室

### 通知系统
- `NotificationService` - 处理推送通知
- `subscribe(userId, notificationType)` - 订阅通知
- `sendNotification(userId, notification)` - 发送通知
- `markAsRead(userId, notificationId)` - 标记通知为已读
- 通知类型：消息、提及、房间邀请

### 在线状态系统
- `PresenceService` - 跟踪用户在线状态
- `updatePresence(userId, status)` - 更新用户状态
- `getUserPresence(userId)` - 获取用户当前状态
- `getOnlineUsers(roomId?)` - 获取房间在线用户
- 状态类型：在线、离开、忙碌、离线

## 功能

### 实时消息
- 即时消息传递
- 消息传递确认
- 已读回执
- 输入指示器
- 消息反应和回复

### 房间管理
- 创建公开和私人房间
- 房间成员管理
- 管理员和版主角色
- 房间设置和权限
- 房间搜索和发现

### 消息功能
- 文本格式化（粗体、斜体、提及）
- 文件和图片分享
- 消息编辑和删除
- 消息搜索和过滤
- 消息线程（回复）

### 用户功能
- 用户配置文件和头像
- 用户屏蔽和举报
- 自定义状态消息
- 免打扰模式
- 消息加密（可选）

## 示例

```javascript
const chatService = new ChatService();
const user1 = new User("u1", "alice", "alice@example.com", UserStatus.ONLINE);
const user2 = new User("u2", "bob", "bob@example.com", UserStatus.ONLINE);

// 创建聊天室
const room = chatService.createRoom("一般讨论", RoomType.PUBLIC, user1.getId());

// 加入房间
chatService.joinRoom(user2.getId(), room.getId());

// 发送消息
const message1 = chatService.sendMessage(user1.getId(), room.getId(), "大家好!");
const message2 = chatService.sendMessage(user2.getId(), room.getId(), "嗨 Alice!");

// 添加反应
message1.addReaction(user2.getId(), "👍");

// 获取消息历史
const messages = chatService.getMessageHistory(room.getId(),
    { limit: 50, before: new Date() });

// 私信
const dmRoom = chatService.createDirectMessage(user1.getId(), user2.getId());
chatService.sendMessage(user1.getId(), dmRoom.getId(), "嘿 Bob，你好吗？");

// 在线状态更新
const presence = new PresenceService();
presence.updatePresence(user1.getId(), UserStatus.AWAY);

// 通知
const notifications = new NotificationService();
notifications.subscribe(user2.getId(), NotificationType.MENTION);
```

## 建议测试用例

### 用户管理
1. 创建不同状态的用户
2. 更新用户在线状态
3. 处理用户屏蔽功能
4. 跟踪最后查看时间戳
5. 管理用户配置文件

### 消息操作
1. 发送消息到聊天室
2. 编辑和删除消息
3. 添加和移除反应
4. 处理不同消息类型
5. 消息验证和清理

### 聊天室管理
1. 创建不同类型的房间
2. 添加和移除房间成员
3. 设置房间权限和角色
4. 处理房间可见性设置
5. 房间成员限制和验证

### 消息历史
1. 使用分页检索消息历史
2. 按内容和过滤器搜索消息
3. 按时间戳排序消息
4. 处理大量消息
5. 消息持久化和存储

### 实时功能
1. 向房间成员广播消息
2. 在线状态广播
3. 输入指示器管理
4. 连接处理和重连
5. 消息传递确认

### 通知系统
1. 发送提及通知
2. 处理通知偏好
3. 标记通知为已读
4. 批量通知传递
5. 推送通知集成

### 安全和管理
1. 用户权限验证
2. 消息内容过滤
3. 消息发送速率限制
4. 用户举报和屏蔽
5. 管理员管理工具

## TDD 步骤

1. **红灯**: 测试用户创建和基本属性
2. **绿灯**: 实现基本用户类
3. **红灯**: 测试消息创建和内容
4. **绿灯**: 实现消息类
5. **红灯**: 测试聊天室创建
6. **绿灯**: 实现聊天室类
7. **红灯**: 测试向房间发送消息
8. **绿灯**: 实现消息发送
9. **红灯**: 测试加入和离开房间
10. **绿灯**: 实现房间成员管理
11. **红灯**: 测试消息历史检索
12. **绿灯**: 实现消息持久化
13. **红灯**: 测试用户在线状态跟踪
14. **绿灯**: 实现在线状态服务
15. **红灯**: 测试通知系统
16. **绿灯**: 实现通知
17. **红灯**: 测试消息反应和编辑
18. **绿灯**: 实现消息功能
19. **重构**: 优化性能和可扩展性

## 学习目标

- 实时通信模式
- WebSocket或类似协议
- 消息队列系统
- 用户状态管理
- 可扩展架构设计
- 数据持久化策略
- 推送通知系统
- 安全和认证

## 高级功能

1. **语音/视频通话**: WebRTC集成
2. **消息加密**: 端到端加密
3. **机器人集成**: 聊天机器人API支持
4. **文件分享**: 高级文件上传/下载
5. **屏幕分享**: 在聊天中分享屏幕
6. **消息翻译**: 多语言支持
7. **聊天分析**: 使用统计和洞察