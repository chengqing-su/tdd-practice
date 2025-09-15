# File Storage System

## Problem Description

Design a file storage system that handles file upload, download, versioning, sharing, and organization with metadata tracking and access control.

## Requirements

### File Class
- `File(filename, content, owner, size, mimeType)` - Create file object
- `getMetadata()` - Get file metadata (size, type, created date, etc.)
- `updateContent(newContent)` - Update file content and increment version
- `getVersion()` - Get current version number
- `getSize()` - Get file size in bytes
- `calculateChecksum()` - Generate content hash for integrity

### Folder Class
- `Folder(name, owner, parentFolder?)` - Create folder/directory
- `addFile(file)` - Add file to folder
- `addSubfolder(folder)` - Add subfolder
- `removeFile(filename)` - Remove file from folder
- `listContents()` - List all files and subfolders
- `getPath()` - Get full path from root
- `getTotalSize()` - Calculate total size including subfolders

### FileStorage Class
- `uploadFile(filename, content, folderId?, metadata?)` - Upload new file
- `downloadFile(fileId)` - Download file content
- `deleteFile(fileId)` - Delete file and all versions
- `moveFile(fileId, newFolderId)` - Move file between folders
- `copyFile(fileId, newFolderId)` - Copy file to another location
- `searchFiles(query, filters?)` - Search files by name, content, or metadata

### Permission System
- `Permission(fileId, userId, permissionType)` - File access permission
- `shareFile(fileId, userId, permission)` - Grant user access to file
- `revokeAccess(fileId, userId)` - Remove user access
- `checkPermission(userId, fileId, action)` - Verify user can perform action
- Permission types: READ, WRITE, DELETE, SHARE

### Version Control
- `FileVersion(file, versionNumber, content, timestamp)` - File version
- `getFileHistory(fileId)` - Get all versions of file
- `revertToVersion(fileId, versionNumber)` - Restore previous version
- `compareVersions(fileId, version1, version2)` - Compare file versions
- `createBranch(fileId, branchName)` - Create named branch

## Features

### File Operations
- Upload with automatic metadata extraction
- Download with integrity verification
- Duplicate detection and deduplication
- File compression for storage optimization
- Batch operations for multiple files

### Organization
- Hierarchical folder structure
- File tagging system
- Smart folders based on criteria
- Favorites and recent files
- Trash/recycle bin functionality

### Search and Discovery
- Full-text search in file contents
- Metadata-based filtering
- Tag-based organization
- File type and size filters
- Date range searches

## Examples

```java
FileStorage storage = new FileStorage();
User user1 = new User("user1", "John Doe");
User user2 = new User("user2", "Jane Smith");

// Create folder structure
Folder documentsFolder = storage.createFolder("Documents", user1.getId());
Folder projectFolder = storage.createFolder("Project", user1.getId(), documentsFolder.getId());

// Upload file
File document = storage.uploadFile("report.pdf", pdfContent, projectFolder.getId(),
                                  Map.of("category", "business", "priority", "high"));

// Share file with another user
storage.shareFile(document.getId(), user2.getId(), Permission.READ);

// Update file (creates new version)
storage.updateFile(document.getId(), updatedContent);
System.out.println(document.getVersion()); // 2

// Search files
List<File> results = storage.searchFiles("quarterly report",
                                         SearchFilter.builder()
                                           .fileType("pdf")
                                           .owner(user1.getId())
                                           .dateRange(lastMonth, now)
                                           .build());

// File history
List<FileVersion> history = storage.getFileHistory(document.getId());
storage.revertToVersion(document.getId(), 1); // Revert to first version
```

## Suggested Test Cases

### File Management
1. Upload files with various content types
2. Download files and verify content integrity
3. Delete files and handle references
4. Move files between folders
5. Copy files with proper versioning

### Folder Operations
1. Create nested folder structures
2. Calculate folder sizes recursively
3. Move folders with all contents
4. Handle folder name conflicts
5. Generate correct folder paths

### Version Control
1. Create new versions on content updates
2. Maintain version history
3. Revert to previous versions
4. Compare content between versions
5. Handle concurrent updates

### Permission System
1. Grant and revoke file access
2. Check permissions before operations
3. Inherit folder permissions
4. Handle permission conflicts
5. Admin override capabilities

### Search Functionality
1. Search by filename patterns
2. Full-text content search
3. Metadata and tag filtering
4. Date range queries
5. Complex search combinations

### Storage Optimization
1. Detect and handle duplicate files
2. Compress files for storage
3. Clean up orphaned files
4. Archive old versions
5. Calculate storage usage

## TDD Steps

1. **Red**: Test file creation and basic properties
2. **Green**: Implement basic File class
3. **Red**: Test file content updates and versioning
4. **Green**: Add version control to File
5. **Red**: Test folder creation and hierarchy
6. **Green**: Implement Folder class
7. **Red**: Test file upload to storage
8. **Green**: Implement basic FileStorage upload
9. **Red**: Test file download and integrity
10. **Green**: Add download with checksum verification
11. **Red**: Test permission system
12. **Green**: Implement access control
13. **Red**: Test search functionality
14. **Green**: Add file search capabilities
15. **Red**: Test folder operations
16. **Green**: Implement folder management
17. **Red**: Test advanced versioning
18. **Green**: Add version comparison and history
19. **Refactor**: Optimize storage and performance

## Learning Goals

- File system design principles
- Binary data handling
- Version control concepts
- Permission and access control systems
- Search and indexing algorithms
- Data integrity and checksums
- Hierarchical data structures
- Metadata management

## Advanced Features

1. **Synchronization**: Multi-device file sync
2. **Collaboration**: Real-time collaborative editing
3. **Backup**: Automated backup strategies
4. **Encryption**: File encryption at rest and in transit
5. **Quotas**: Storage limits and usage tracking
6. **Audit Trail**: Complete activity logging
7. **Integration**: External storage provider support

---

## 问题描述

设计一个文件存储系统，处理文件上传、下载、版本控制、共享和组织，包括元数据跟踪和访问控制。

## 需求

### File 类
- `File(filename, content, owner, size, mimeType)` - 创建文件对象
- `getMetadata()` - 获取文件元数据（大小、类型、创建日期等）
- `updateContent(newContent)` - 更新文件内容并递增版本
- `getVersion()` - 获取当前版本号
- `getSize()` - 获取文件大小（字节）
- `calculateChecksum()` - 生成内容哈希以确保完整性

### Folder 类
- `Folder(name, owner, parentFolder?)` - 创建文件夹/目录
- `addFile(file)` - 添加文件到文件夹
- `addSubfolder(folder)` - 添加子文件夹
- `removeFile(filename)` - 从文件夹中移除文件
- `listContents()` - 列出所有文件和子文件夹
- `getPath()` - 获取从根目录的完整路径
- `getTotalSize()` - 计算包括子文件夹的总大小

### FileStorage 类
- `uploadFile(filename, content, folderId?, metadata?)` - 上传新文件
- `downloadFile(fileId)` - 下载文件内容
- `deleteFile(fileId)` - 删除文件和所有版本
- `moveFile(fileId, newFolderId)` - 在文件夹间移动文件
- `copyFile(fileId, newFolderId)` - 复制文件到另一位置
- `searchFiles(query, filters?)` - 按名称、内容或元数据搜索文件

### 权限系统
- `Permission(fileId, userId, permissionType)` - 文件访问权限
- `shareFile(fileId, userId, permission)` - 授予用户文件访问权
- `revokeAccess(fileId, userId)` - 撤销用户访问权
- `checkPermission(userId, fileId, action)` - 验证用户可以执行操作
- 权限类型：读取、写入、删除、共享

### 版本控制
- `FileVersion(file, versionNumber, content, timestamp)` - 文件版本
- `getFileHistory(fileId)` - 获取文件的所有版本
- `revertToVersion(fileId, versionNumber)` - 恢复到之前版本
- `compareVersions(fileId, version1, version2)` - 比较文件版本
- `createBranch(fileId, branchName)` - 创建命名分支

## 功能

### 文件操作
- 自动元数据提取的上传
- 完整性验证的下载
- 重复检测和去重
- 存储优化的文件压缩
- 多文件的批量操作

### 组织
- 层次化文件夹结构
- 文件标签系统
- 基于条件的智能文件夹
- 收藏和最近文件
- 垃圾箱/回收站功能

### 搜索和发现
- 文件内容的全文搜索
- 基于元数据的过滤
- 基于标签的组织
- 文件类型和大小过滤器
- 日期范围搜索

## 示例

```java
FileStorage storage = new FileStorage();
User user1 = new User("user1", "张三");
User user2 = new User("user2", "李四");

// 创建文件夹结构
Folder documentsFolder = storage.createFolder("文档", user1.getId());
Folder projectFolder = storage.createFolder("项目", user1.getId(), documentsFolder.getId());

// 上传文件
File document = storage.uploadFile("报告.pdf", pdfContent, projectFolder.getId(),
                                  Map.of("分类", "业务", "优先级", "高"));

// 与另一用户共享文件
storage.shareFile(document.getId(), user2.getId(), Permission.READ);

// 更新文件（创建新版本）
storage.updateFile(document.getId(), updatedContent);
System.out.println(document.getVersion()); // 2

// 搜索文件
List<File> results = storage.searchFiles("季度报告",
                                         SearchFilter.builder()
                                           .fileType("pdf")
                                           .owner(user1.getId())
                                           .dateRange(lastMonth, now)
                                           .build());

// 文件历史
List<FileVersion> history = storage.getFileHistory(document.getId());
storage.revertToVersion(document.getId(), 1); // 恢复到第一版本
```

## 建议测试用例

### 文件管理
1. 上传各种内容类型的文件
2. 下载文件并验证内容完整性
3. 删除文件并处理引用
4. 在文件夹间移动文件
5. 正确版本控制的文件复制

### 文件夹操作
1. 创建嵌套文件夹结构
2. 递归计算文件夹大小
3. 移动包含所有内容的文件夹
4. 处理文件夹名称冲突
5. 生成正确的文件夹路径

### 版本控制
1. 内容更新时创建新版本
2. 维护版本历史
3. 恢复到之前版本
4. 比较版本间内容
5. 处理并发更新

### 权限系统
1. 授予和撤销文件访问权
2. 操作前检查权限
3. 继承文件夹权限
4. 处理权限冲突
5. 管理员覆盖能力

### 搜索功能
1. 按文件名模式搜索
2. 全文内容搜索
3. 元数据和标签过滤
4. 日期范围查询
5. 复杂搜索组合

### 存储优化
1. 检测和处理重复文件
2. 压缩文件存储
3. 清理孤立文件
4. 归档旧版本
5. 计算存储使用量

## TDD 步骤

1. **红灯**: 测试文件创建和基本属性
2. **绿灯**: 实现基本文件类
3. **红灯**: 测试文件内容更新和版本控制
4. **绿灯**: 为文件添加版本控制
5. **红灯**: 测试文件夹创建和层次结构
6. **绿灯**: 实现文件夹类
7. **红灯**: 测试文件上传到存储
8. **绿灯**: 实现基本文件存储上传
9. **红灯**: 测试文件下载和完整性
10. **绿灯**: 添加校验和验证的下载
11. **红灯**: 测试权限系统
12. **绿灯**: 实现访问控制
13. **红灯**: 测试搜索功能
14. **绿灯**: 添加文件搜索能力
15. **红灯**: 测试文件夹操作
16. **绿灯**: 实现文件夹管理
17. **红灯**: 测试高级版本控制
18. **绿灯**: 添加版本比较和历史
19. **重构**: 优化存储和性能

## 学习目标

- 文件系统设计原则
- 二进制数据处理
- 版本控制概念
- 权限和访问控制系统
- 搜索和索引算法
- 数据完整性和校验和
- 层次化数据结构
- 元数据管理

## 高级功能

1. **同步**: 多设备文件同步
2. **协作**: 实时协作编辑
3. **备份**: 自动备份策略
4. **加密**: 静态和传输中的文件加密
5. **配额**: 存储限制和使用跟踪
6. **审计跟踪**: 完整活动日志记录
7. **集成**: 外部存储提供商支持