# Todo API with Advanced Validation

Build a comprehensive Todo management API with advanced Pydantic validation, custom validators, and sophisticated error handling. This problem focuses on input validation patterns and data integrity.

[中文版本](#带高级验证的todo-api)

## Problem Description

Create a Todo API that demonstrates advanced validation techniques including:
- Complex Pydantic models with custom validators
- Field-level and model-level validation
- Custom error messages and formatting
- Data transformation and sanitization
- Business rule validation

The API should handle various Todo operations while ensuring data quality and providing clear validation feedback.

## Requirements

### Functional Requirements
1. **Todo Management** - Create, read, update, delete todos
2. **Priority System** - Support high, medium, low priorities
3. **Category System** - Organize todos by categories
4. **Due Date Handling** - Set and validate due dates
5. **Status Tracking** - Track todo completion status
6. **Tag System** - Add tags to todos for organization
7. **Search and Filter** - Find todos by various criteria

### Validation Requirements
1. **Title Validation** - Required, length constraints, no profanity
2. **Description Validation** - Optional, length limits, formatting
3. **Priority Validation** - Must be valid enum value
4. **Due Date Validation** - Cannot be in the past, reasonable limits
5. **Category Validation** - Valid category names, existence check
6. **Tag Validation** - Format constraints, uniqueness
7. **Business Rules** - Complex validation across fields

## API Specification

### Endpoints

| Method | Endpoint | Description | Status Code |
|--------|----------|-------------|-------------|
| POST | `/todos/` | Create new todo | 201, 400 |
| GET | `/todos/` | List todos with filters | 200 |
| GET | `/todos/{todo_id}` | Get todo by ID | 200, 404 |
| PUT | `/todos/{todo_id}` | Update todo | 200, 400, 404 |
| DELETE | `/todos/{todo_id}` | Delete todo | 204, 404 |
| POST | `/categories/` | Create category | 201, 400 |
| GET | `/categories/` | List categories | 200 |

### Data Models

**Todo Model:**
```json
{
  "id": 1,
  "title": "Complete project documentation",
  "description": "Write comprehensive API documentation with examples",
  "priority": "high",
  "status": "pending",
  "category_id": 1,
  "tags": ["work", "documentation"],
  "due_date": "2023-12-31T23:59:59Z",
  "created_at": "2023-01-01T00:00:00Z",
  "updated_at": "2023-01-01T00:00:00Z",
  "completed_at": null
}
```

**Category Model:**
```json
{
  "id": 1,
  "name": "Work",
  "description": "Work-related tasks",
  "color": "#FF5733",
  "created_at": "2023-01-01T00:00:00Z"
}
```

## Test Examples

### Validation Tests
```python
from fastapi.testclient import TestClient
from main import app
import pytest
from datetime import datetime, timedelta

client = TestClient(app)

def test_create_todo_with_valid_data():
    todo_data = {
        "title": "Test Todo",
        "description": "A test todo item",
        "priority": "medium",
        "category_id": 1,
        "tags": ["test", "example"],
        "due_date": (datetime.now() + timedelta(days=7)).isoformat()
    }
    response = client.post("/todos/", json=todo_data)

    assert response.status_code == 201
    assert response.json()["title"] == "Test Todo"
    assert response.json()["priority"] == "medium"

def test_create_todo_with_empty_title():
    todo_data = {
        "title": "",
        "description": "Todo with empty title",
        "priority": "low"
    }
    response = client.post("/todos/", json=todo_data)

    assert response.status_code == 400
    assert "title" in response.json()["detail"][0]["loc"]
    assert "empty" in response.json()["detail"][0]["msg"].lower()

def test_create_todo_with_long_title():
    todo_data = {
        "title": "x" * 201,  # Too long
        "priority": "high"
    }
    response = client.post("/todos/", json=todo_data)

    assert response.status_code == 400
    assert "title" in response.json()["detail"][0]["loc"]
    assert "length" in response.json()["detail"][0]["msg"].lower()

def test_create_todo_with_invalid_priority():
    todo_data = {
        "title": "Test Todo",
        "priority": "super_urgent"  # Invalid priority
    }
    response = client.post("/todos/", json=todo_data)

    assert response.status_code == 400
    assert "priority" in response.json()["detail"][0]["loc"]

def test_create_todo_with_past_due_date():
    todo_data = {
        "title": "Test Todo",
        "due_date": (datetime.now() - timedelta(days=1)).isoformat()
    }
    response = client.post("/todos/", json=todo_data)

    assert response.status_code == 400
    assert "due_date" in response.json()["detail"][0]["loc"]
    assert "past" in response.json()["detail"][0]["msg"].lower()

def test_create_todo_with_duplicate_tags():
    todo_data = {
        "title": "Test Todo",
        "tags": ["work", "important", "work"]  # Duplicate tag
    }
    response = client.post("/todos/", json=todo_data)

    assert response.status_code == 201
    # Should automatically remove duplicates
    assert response.json()["tags"] == ["work", "important"]

def test_create_todo_with_invalid_category():
    todo_data = {
        "title": "Test Todo",
        "category_id": 999  # Non-existent category
    }
    response = client.post("/todos/", json=todo_data)

    assert response.status_code == 400
    assert "category" in response.json()["detail"].lower()

def test_update_completed_todo_due_date():
    # First create and complete a todo
    todo_data = {"title": "Test Todo", "status": "completed"}
    create_response = client.post("/todos/", json=todo_data)
    todo_id = create_response.json()["id"]

    # Try to update due date of completed todo
    update_data = {
        "due_date": (datetime.now() + timedelta(days=7)).isoformat()
    }
    response = client.put(f"/todos/{todo_id}", json=update_data)

    assert response.status_code == 400
    assert "completed" in response.json()["detail"].lower()
```

## Implementation Guide

### Step 1: Advanced Pydantic Models
```python
from pydantic import BaseModel, validator, Field, root_validator
from typing import List, Optional
from datetime import datetime
from enum import Enum
import re

class Priority(str, Enum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"

class Status(str, Enum):
    PENDING = "pending"
    IN_PROGRESS = "in_progress"
    COMPLETED = "completed"
    CANCELLED = "cancelled"

class TodoBase(BaseModel):
    title: str = Field(..., min_length=1, max_length=200)
    description: Optional[str] = Field(None, max_length=1000)
    priority: Priority = Priority.MEDIUM
    category_id: Optional[int] = None
    tags: List[str] = Field(default_factory=list)
    due_date: Optional[datetime] = None

    @validator('title')
    def validate_title(cls, v):
        if not v or v.isspace():
            raise ValueError('Title cannot be empty or whitespace only')

        # Check for profanity (simple example)
        profanity_words = ['spam', 'bad_word']  # Add your list
        if any(word in v.lower() for word in profanity_words):
            raise ValueError('Title contains inappropriate content')

        return v.strip()

    @validator('description')
    def validate_description(cls, v):
        if v is not None:
            v = v.strip()
            if len(v) == 0:
                return None
        return v

    @validator('tags')
    def validate_tags(cls, v):
        if not v:
            return []

        # Remove duplicates while preserving order
        seen = set()
        unique_tags = []
        for tag in v:
            tag = tag.strip().lower()
            if tag and tag not in seen:
                # Validate tag format
                if not re.match(r'^[a-zA-Z0-9_-]+$', tag):
                    raise ValueError(f'Invalid tag format: {tag}')
                if len(tag) > 20:
                    raise ValueError(f'Tag too long: {tag}')
                seen.add(tag)
                unique_tags.append(tag)

        if len(unique_tags) > 10:
            raise ValueError('Maximum 10 tags allowed')

        return unique_tags

    @validator('due_date')
    def validate_due_date(cls, v):
        if v is not None:
            if v <= datetime.now():
                raise ValueError('Due date must be in the future')

            # Don't allow due dates too far in the future
            max_future = datetime.now().replace(year=datetime.now().year + 2)
            if v > max_future:
                raise ValueError('Due date cannot be more than 2 years in the future')

        return v

    @root_validator
    def validate_high_priority_due_date(cls, values):
        priority = values.get('priority')
        due_date = values.get('due_date')

        if priority == Priority.HIGH and due_date is None:
            raise ValueError('High priority todos must have a due date')

        return values

class TodoCreate(TodoBase):
    @validator('category_id')
    def validate_category_exists(cls, v):
        if v is not None:
            # Check if category exists in database
            if not category_exists(v):
                raise ValueError(f'Category with id {v} does not exist')
        return v

class TodoUpdate(BaseModel):
    title: Optional[str] = Field(None, min_length=1, max_length=200)
    description: Optional[str] = Field(None, max_length=1000)
    priority: Optional[Priority] = None
    status: Optional[Status] = None
    category_id: Optional[int] = None
    tags: Optional[List[str]] = None
    due_date: Optional[datetime] = None

    @validator('title')
    def validate_title(cls, v):
        if v is not None:
            return TodoBase.__fields__['title'].validator(v)
        return v

    @validator('tags')
    def validate_tags(cls, v):
        if v is not None:
            return TodoBase.__fields__['tags'].validator(v)
        return v

    @validator('due_date')
    def validate_due_date(cls, v):
        if v is not None:
            return TodoBase.__fields__['due_date'].validator(v)
        return v

    @root_validator
    def validate_completed_todo_restrictions(cls, values):
        # In update context, we need access to current todo status
        # This would be handled in the endpoint logic
        return values

class Todo(TodoBase):
    id: int
    status: Status = Status.PENDING
    created_at: datetime
    updated_at: datetime
    completed_at: Optional[datetime] = None

    class Config:
        from_attributes = True

class CategoryBase(BaseModel):
    name: str = Field(..., min_length=1, max_length=50)
    description: Optional[str] = Field(None, max_length=200)
    color: str = Field(..., regex=r'^#[0-9A-F]{6}$')

    @validator('name')
    def validate_name(cls, v):
        if not re.match(r'^[a-zA-Z0-9\s-_]+$', v):
            raise ValueError('Category name contains invalid characters')
        return v.strip()

class CategoryCreate(CategoryBase):
    @validator('name')
    def validate_unique_name(cls, v):
        if category_name_exists(v):
            raise ValueError('Category name already exists')
        return v

class Category(CategoryBase):
    id: int
    created_at: datetime

    class Config:
        from_attributes = True
```

### Step 2: Custom Validators and Business Logic
```python
def category_exists(category_id: int) -> bool:
    """Check if category exists in database"""
    return any(cat["id"] == category_id for cat in categories_db)

def category_name_exists(name: str) -> bool:
    """Check if category name already exists"""
    return any(cat["name"].lower() == name.lower() for cat in categories_db)

def can_update_completed_todo(todo: dict, update_data: dict) -> bool:
    """Check if completed todo can be updated"""
    if todo["status"] == "completed":
        # Only allow status changes for completed todos
        allowed_fields = {"status"}
        update_fields = set(k for k, v in update_data.items() if v is not None)
        if update_fields - allowed_fields:
            return False
    return True
```

### Step 3: Enhanced Error Handling
```python
from fastapi import FastAPI, HTTPException, Request
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
from pydantic import ValidationError

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    """Custom validation error handler"""
    errors = []
    for error in exc.errors():
        field = " -> ".join(str(loc) for loc in error["loc"][1:])  # Skip 'body'
        message = error["msg"]
        errors.append({
            "field": field,
            "message": message,
            "invalid_value": error.get("input")
        })

    return JSONResponse(
        status_code=400,
        content={
            "detail": "Validation failed",
            "errors": errors
        }
    )

@app.exception_handler(ValueError)
async def value_error_handler(request: Request, exc: ValueError):
    """Handle business rule validation errors"""
    return JSONResponse(
        status_code=400,
        content={"detail": str(exc)}
    )
```

### Step 4: API Endpoints with Validation
```python
@app.post("/todos/", response_model=Todo, status_code=201)
def create_todo(todo: TodoCreate):
    global next_todo_id

    # Additional business validation
    if todo.category_id and not category_exists(todo.category_id):
        raise HTTPException(
            status_code=400,
            detail="Specified category does not exist"
        )

    todo_dict = {
        "id": next_todo_id,
        **todo.dict(),
        "status": Status.PENDING,
        "created_at": datetime.now(),
        "updated_at": datetime.now(),
        "completed_at": None
    }

    todos_db.append(todo_dict)
    next_todo_id += 1

    return Todo(**todo_dict)

@app.put("/todos/{todo_id}", response_model=Todo)
def update_todo(todo_id: int, todo_update: TodoUpdate):
    # Find todo
    todo_index = None
    for i, todo in enumerate(todos_db):
        if todo["id"] == todo_id:
            todo_index = i
            break

    if todo_index is None:
        raise HTTPException(status_code=404, detail="Todo not found")

    current_todo = todos_db[todo_index]

    # Business rule validation
    if not can_update_completed_todo(current_todo, todo_update.dict()):
        raise HTTPException(
            status_code=400,
            detail="Cannot modify completed todo except status"
        )

    # Update fields
    update_data = todo_update.dict(exclude_unset=True)
    for field, value in update_data.items():
        if value is not None:
            current_todo[field] = value

    current_todo["updated_at"] = datetime.now()

    # Set completed_at if status changed to completed
    if update_data.get("status") == Status.COMPLETED:
        current_todo["completed_at"] = datetime.now()
    elif update_data.get("status") and update_data["status"] != Status.COMPLETED:
        current_todo["completed_at"] = None

    return Todo(**current_todo)
```

## Validation Patterns

### Custom Field Validators
```python
@validator('email')
def validate_email_format(cls, v):
    if v and not re.match(r'^[^@]+@[^@]+\.[^@]+$', v):
        raise ValueError('Invalid email format')
    return v

@validator('priority', pre=True)
def validate_priority_case_insensitive(cls, v):
    if isinstance(v, str):
        return v.lower()
    return v
```

### Cross-Field Validation
```python
@root_validator
def validate_date_range(cls, values):
    start_date = values.get('start_date')
    due_date = values.get('due_date')

    if start_date and due_date and start_date >= due_date:
        raise ValueError('Start date must be before due date')

    return values
```

### Conditional Validation
```python
@root_validator
def validate_conditional_fields(cls, values):
    priority = values.get('priority')
    assignee = values.get('assignee')

    if priority == Priority.HIGH and not assignee:
        raise ValueError('High priority todos must have an assignee')

    return values
```

## Success Criteria

- [ ] Comprehensive input validation with Pydantic
- [ ] Custom validators for business rules
- [ ] Clear, actionable error messages
- [ ] Cross-field validation working
- [ ] Data sanitization and transformation
- [ ] All validation tests passing
- [ ] Performance optimization for validators
- [ ] Proper error response formatting

## Extensions

1. **Async Validators** - Database lookups in validators
2. **Validation Groups** - Different validation for different contexts
3. **Custom Error Codes** - Structured error identification
4. **Validation Caching** - Cache expensive validation results
5. **Internationalization** - Multi-language error messages

---

# 带高级验证的Todo API

构建具有高级Pydantic验证、自定义验证器和复杂错误处理的综合Todo管理API。这个问题专注于输入验证模式和数据完整性。

## 问题描述

创建一个Todo API，演示高级验证技术，包括：
- 带自定义验证器的复杂Pydantic模型
- 字段级和模型级验证
- 自定义错误消息和格式化
- 数据转换和清理
- 业务规则验证

API应处理各种Todo操作，同时确保数据质量并提供清晰的验证反馈。

## 成功标准

- [ ] 使用Pydantic进行综合输入验证
- [ ] 业务规则的自定义验证器
- [ ] 清晰、可操作的错误消息
- [ ] 跨字段验证工作
- [ ] 数据清理和转换
- [ ] 所有验证测试通过
- [ ] 验证器的性能优化
- [ ] 适当的错误响应格式化