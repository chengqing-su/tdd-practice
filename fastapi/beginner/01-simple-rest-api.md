# Simple REST API

Build a basic REST API for managing books using FastAPI with Test-Driven Development. This problem introduces core FastAPI concepts including routing, request/response models, and basic CRUD operations.

[中文版本](#简单rest-api)

## Problem Description

Create a simple book management API that allows users to:
- Create new books
- Retrieve all books
- Retrieve a single book by ID
- Update existing books
- Delete books

The API should use in-memory storage (Python lists/dictionaries) and follow RESTful conventions.

## Requirements

### Functional Requirements
1. **Create Book** - Add new books with title, author, and publication year
2. **List Books** - Retrieve all books with pagination support
3. **Get Book** - Retrieve a specific book by its ID
4. **Update Book** - Modify existing book information
5. **Delete Book** - Remove books from the collection
6. **Validation** - Ensure all required fields are provided
7. **Error Handling** - Return appropriate HTTP status codes and error messages

### Technical Requirements
- Use FastAPI framework
- Implement Pydantic models for request/response validation
- Return proper HTTP status codes (200, 201, 404, 400, etc.)
- Include automatic API documentation
- Use in-memory storage (no database required)
- Follow RESTful API conventions

## API Specification

### Endpoints

| Method | Endpoint | Description | Status Code |
|--------|----------|-------------|-------------|
| POST | `/books/` | Create a new book | 201 |
| GET | `/books/` | Get all books | 200 |
| GET | `/books/{book_id}` | Get book by ID | 200, 404 |
| PUT | `/books/{book_id}` | Update book | 200, 404 |
| DELETE | `/books/{book_id}` | Delete book | 204, 404 |

### Data Models

**Book Model:**
```json
{
  "id": 1,
  "title": "The Great Gatsby",
  "author": "F. Scott Fitzgerald",
  "publication_year": 1925,
  "isbn": "978-0-7432-7356-5"
}
```

**Book Creation Request:**
```json
{
  "title": "The Great Gatsby",
  "author": "F. Scott Fitzgerald",
  "publication_year": 1925,
  "isbn": "978-0-7432-7356-5"
}
```

## Test Examples

### Test Structure
```python
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_create_book():
    book_data = {
        "title": "Test Book",
        "author": "Test Author",
        "publication_year": 2023,
        "isbn": "978-0-123456-78-9"
    }
    response = client.post("/books/", json=book_data)

    assert response.status_code == 201
    assert response.json()["title"] == "Test Book"
    assert "id" in response.json()

def test_get_all_books():
    response = client.get("/books/")
    assert response.status_code == 200
    assert isinstance(response.json(), list)

def test_get_book_by_id():
    # First create a book
    book_data = {"title": "Test", "author": "Author", "publication_year": 2023}
    create_response = client.post("/books/", json=book_data)
    book_id = create_response.json()["id"]

    # Then retrieve it
    response = client.get(f"/books/{book_id}")
    assert response.status_code == 200
    assert response.json()["id"] == book_id

def test_get_nonexistent_book():
    response = client.get("/books/999")
    assert response.status_code == 404
    assert "not found" in response.json()["detail"].lower()
```

## Implementation Guide

### Step 1: Project Setup
Create the basic FastAPI application structure:

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import List, Optional

app = FastAPI(title="Book Management API", version="1.0.0")

# In-memory storage
books_db = []
next_id = 1
```

### Step 2: Define Pydantic Models
```python
class BookBase(BaseModel):
    title: str
    author: str
    publication_year: int
    isbn: Optional[str] = None

class BookCreate(BookBase):
    pass

class Book(BookBase):
    id: int

    class Config:
        from_attributes = True
```

### Step 3: Implement CRUD Operations
Start with the simplest endpoint and work your way up:

1. **Create Book** - POST `/books/`
2. **Get All Books** - GET `/books/`
3. **Get Book by ID** - GET `/books/{book_id}`
4. **Update Book** - PUT `/books/{book_id}`
5. **Delete Book** - DELETE `/books/{book_id}`

### Step 4: Add Error Handling
```python
def get_book_by_id(book_id: int):
    for book in books_db:
        if book["id"] == book_id:
            return book
    raise HTTPException(status_code=404, detail="Book not found")
```

## TDD Process

### Red Phase Example
```python
def test_create_book_returns_book_with_id():
    book_data = {
        "title": "1984",
        "author": "George Orwell",
        "publication_year": 1949
    }
    response = client.post("/books/", json=book_data)

    assert response.status_code == 201
    response_data = response.json()
    assert response_data["title"] == "1984"
    assert response_data["author"] == "George Orwell"
    assert "id" in response_data
    assert isinstance(response_data["id"], int)
```

### Green Phase Implementation
```python
@app.post("/books/", response_model=Book, status_code=201)
def create_book(book: BookCreate):
    global next_id
    book_dict = book.dict()
    book_dict["id"] = next_id
    books_db.append(book_dict)
    next_id += 1
    return book_dict
```

## Validation Requirements

### Input Validation
- `title`: Required, non-empty string, max 200 characters
- `author`: Required, non-empty string, max 100 characters
- `publication_year`: Required, integer between 1000 and current year
- `isbn`: Optional, valid ISBN format if provided

### Business Rules
- No duplicate ISBNs allowed
- Publication year cannot be in the future
- Title and author combination should be unique

## Advanced Features (Optional)

### Pagination
```python
@app.get("/books/", response_model=List[Book])
def get_books(skip: int = 0, limit: int = 100):
    return books_db[skip : skip + limit]
```

### Search and Filtering
```python
@app.get("/books/search/", response_model=List[Book])
def search_books(q: Optional[str] = None, author: Optional[str] = None):
    filtered_books = books_db

    if q:
        filtered_books = [book for book in filtered_books
                         if q.lower() in book["title"].lower()]

    if author:
        filtered_books = [book for book in filtered_books
                         if author.lower() in book["author"].lower()]

    return filtered_books
```

## Testing Strategy

### Unit Tests
- Test each endpoint independently
- Test validation rules
- Test error conditions
- Test edge cases

### Integration Tests
- Test complete workflows (create → read → update → delete)
- Test data persistence across requests
- Test error propagation

### Test Data Management
```python
import pytest

@pytest.fixture(autouse=True)
def clear_books():
    """Clear books database before each test"""
    global books_db, next_id
    books_db.clear()
    next_id = 1
```

## Common Challenges

### Challenge 1: ID Management
Managing unique IDs in memory without a database.

**Solution**: Use a global counter that increments with each new book.

### Challenge 2: Data Persistence
Data doesn't persist between test runs.

**Solution**: Use fixtures to set up and tear down test data for each test.

### Challenge 3: Validation Errors
Handling and testing various validation scenarios.

**Solution**: Create comprehensive test cases for each validation rule.

## Success Criteria

- [ ] All CRUD operations work correctly
- [ ] Proper HTTP status codes returned
- [ ] Request/response validation with Pydantic
- [ ] Comprehensive test coverage (>90%)
- [ ] Error handling for edge cases
- [ ] Clean, readable code structure
- [ ] Automatic API documentation works
- [ ] All tests pass consistently

## Extensions

Once basic functionality works:
1. Add book categories/genres
2. Implement soft delete functionality
3. Add created/updated timestamps
4. Implement book rating system
5. Add bulk operations (create/update multiple books)

---

# 简单REST API

使用FastAPI和测试驱动开发构建图书管理的基本REST API。这个问题介绍了核心FastAPI概念，包括路由、请求/响应模型和基本CRUD操作。

## 问题描述

创建一个简单的图书管理API，允许用户：
- 创建新图书
- 检索所有图书
- 根据ID检索单本图书
- 更新现有图书
- 删除图书

API应使用内存存储（Python列表/字典）并遵循RESTful约定。

## 需求

### 功能需求
1. **创建图书** - 添加带有标题、作者和出版年份的新图书
2. **列出图书** - 检索所有图书并支持分页
3. **获取图书** - 根据ID检索特定图书
4. **更新图书** - 修改现有图书信息
5. **删除图书** - 从集合中删除图书
6. **验证** - 确保提供所有必需字段
7. **错误处理** - 返回适当的HTTP状态码和错误消息

### 技术需求
- 使用FastAPI框架
- 实现Pydantic模型进行请求/响应验证
- 返回正确的HTTP状态码（200、201、404、400等）
- 包含自动API文档
- 使用内存存储（不需要数据库）
- 遵循RESTful API约定

## API规范

### 端点

| 方法 | 端点 | 描述 | 状态码 |
|------|------|------|--------|
| POST | `/books/` | 创建新图书 | 201 |
| GET | `/books/` | 获取所有图书 | 200 |
| GET | `/books/{book_id}` | 根据ID获取图书 | 200, 404 |
| PUT | `/books/{book_id}` | 更新图书 | 200, 404 |
| DELETE | `/books/{book_id}` | 删除图书 | 204, 404 |

## TDD过程

### 红灯阶段示例
```python
def test_create_book_returns_book_with_id():
    book_data = {
        "title": "1984",
        "author": "George Orwell",
        "publication_year": 1949
    }
    response = client.post("/books/", json=book_data)

    assert response.status_code == 201
    response_data = response.json()
    assert response_data["title"] == "1984"
    assert response_data["author"] == "George Orwell"
    assert "id" in response_data
    assert isinstance(response_data["id"], int)
```

## 成功标准

- [ ] 所有CRUD操作正常工作
- [ ] 返回正确的HTTP状态码
- [ ] 使用Pydantic进行请求/响应验证
- [ ] 全面的测试覆盖率（>90%）
- [ ] 边界情况的错误处理
- [ ] 清洁、可读的代码结构
- [ ] 自动API文档工作正常
- [ ] 所有测试始终通过