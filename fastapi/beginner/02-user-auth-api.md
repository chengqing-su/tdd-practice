# User Authentication API

Build a secure user authentication system using FastAPI with JWT tokens, password hashing, and protected routes. This problem introduces authentication patterns and security best practices.

[中文版本](#用户认证api)

## Problem Description

Create a user authentication API that provides:
- User registration with secure password storage
- User login with JWT token generation
- Protected routes that require authentication
- Token validation and refresh mechanisms
- User profile management

The system should implement modern security practices including password hashing, JWT tokens, and proper error handling.

## Requirements

### Functional Requirements
1. **User Registration** - Create new user accounts with validation
2. **User Login** - Authenticate users and return JWT tokens
3. **Protected Routes** - Endpoints that require valid authentication
4. **Token Validation** - Verify JWT tokens on protected endpoints
5. **User Profile** - Allow users to view and update their profiles
6. **Password Security** - Hash passwords before storage
7. **Error Handling** - Secure error messages that don't leak information

### Security Requirements
- Password hashing using bcrypt or similar
- JWT tokens with expiration
- Secure token validation
- Protection against common attacks (timing attacks, etc.)
- Input validation and sanitization
- Proper error messages without information leakage

## API Specification

### Endpoints

| Method | Endpoint | Description | Authentication | Status Code |
|--------|----------|-------------|----------------|-------------|
| POST | `/auth/register` | Register new user | No | 201, 400 |
| POST | `/auth/login` | User login | No | 200, 401 |
| GET | `/auth/me` | Get current user profile | Yes | 200, 401 |
| PUT | `/auth/me` | Update user profile | Yes | 200, 401 |
| POST | `/auth/refresh` | Refresh JWT token | Yes | 200, 401 |

### Data Models

**User Registration:**
```json
{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "SecurePassword123!",
  "full_name": "John Doe"
}
```

**Login Request:**
```json
{
  "username": "john_doe",
  "password": "SecurePassword123!"
}
```

**Login Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in": 3600
}
```

**User Profile:**
```json
{
  "id": 1,
  "username": "john_doe",
  "email": "john@example.com",
  "full_name": "John Doe",
  "is_active": true,
  "created_at": "2023-01-01T00:00:00Z"
}
```

## Test Examples

### Authentication Tests
```python
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_register_user():
    user_data = {
        "username": "testuser",
        "email": "test@example.com",
        "password": "TestPassword123!",
        "full_name": "Test User"
    }
    response = client.post("/auth/register", json=user_data)

    assert response.status_code == 201
    assert response.json()["username"] == "testuser"
    assert "password" not in response.json()  # Password should not be returned

def test_register_duplicate_username():
    user_data = {
        "username": "duplicate",
        "email": "user1@example.com",
        "password": "Password123!",
        "full_name": "User One"
    }
    # First registration
    client.post("/auth/register", json=user_data)

    # Second registration with same username
    user_data["email"] = "user2@example.com"
    response = client.post("/auth/register", json=user_data)

    assert response.status_code == 400
    assert "username" in response.json()["detail"].lower()

def test_login_valid_credentials():
    # First register a user
    user_data = {
        "username": "logintest",
        "email": "login@example.com",
        "password": "LoginPassword123!",
        "full_name": "Login Test"
    }
    client.post("/auth/register", json=user_data)

    # Then login
    login_data = {
        "username": "logintest",
        "password": "LoginPassword123!"
    }
    response = client.post("/auth/login", json=login_data)

    assert response.status_code == 200
    assert "access_token" in response.json()
    assert response.json()["token_type"] == "bearer"

def test_login_invalid_credentials():
    login_data = {
        "username": "nonexistent",
        "password": "wrongpassword"
    }
    response = client.post("/auth/login", json=login_data)

    assert response.status_code == 401
    assert "invalid" in response.json()["detail"].lower()

def test_protected_route_without_token():
    response = client.get("/auth/me")
    assert response.status_code == 401

def test_protected_route_with_valid_token():
    # Register and login to get token
    user_data = {
        "username": "protectedtest",
        "email": "protected@example.com",
        "password": "ProtectedPassword123!",
        "full_name": "Protected Test"
    }
    client.post("/auth/register", json=user_data)

    login_response = client.post("/auth/login", json={
        "username": "protectedtest",
        "password": "ProtectedPassword123!"
    })
    token = login_response.json()["access_token"]

    # Access protected route
    headers = {"Authorization": f"Bearer {token}"}
    response = client.get("/auth/me", headers=headers)

    assert response.status_code == 200
    assert response.json()["username"] == "protectedtest"

def test_protected_route_with_invalid_token():
    headers = {"Authorization": "Bearer invalid_token"}
    response = client.get("/auth/me", headers=headers)
    assert response.status_code == 401
```

## Implementation Guide

### Step 1: Dependencies Setup
```python
from fastapi import FastAPI, HTTPException, Depends, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from pydantic import BaseModel, EmailStr
from passlib.context import CryptContext
from jose import JWTError, jwt
from datetime import datetime, timedelta
from typing import Optional

app = FastAPI(title="User Authentication API")

# Security configuration
SECRET_KEY = "your-secret-key-here"  # In production, use environment variable
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
security = HTTPBearer()

# In-memory storage
users_db = []
next_user_id = 1
```

### Step 2: Pydantic Models
```python
class UserBase(BaseModel):
    username: str
    email: EmailStr
    full_name: str

class UserCreate(UserBase):
    password: str

class User(UserBase):
    id: int
    is_active: bool = True
    created_at: datetime

class UserInDB(User):
    hashed_password: str

class Token(BaseModel):
    access_token: str
    token_type: str
    expires_in: int

class LoginRequest(BaseModel):
    username: str
    password: str
```

### Step 3: Security Functions
```python
def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    return pwd_context.hash(password)

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

def get_user_by_username(username: str) -> Optional[UserInDB]:
    for user in users_db:
        if user["username"] == username:
            return UserInDB(**user)
    return None

def authenticate_user(username: str, password: str) -> Optional[UserInDB]:
    user = get_user_by_username(username)
    if not user:
        return None
    if not verify_password(password, user.hashed_password):
        return None
    return user

async def get_current_user(credentials: HTTPAuthorizationCredentials = Depends(security)) -> User:
    try:
        payload = jwt.decode(credentials.credentials, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Could not validate credentials"
            )
    except JWTError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Could not validate credentials"
        )

    user = get_user_by_username(username)
    if user is None:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Could not validate credentials"
        )
    return User(**user.dict())
```

### Step 4: Authentication Endpoints
```python
@app.post("/auth/register", response_model=User, status_code=201)
def register_user(user: UserCreate):
    global next_user_id

    # Check if username already exists
    if get_user_by_username(user.username):
        raise HTTPException(
            status_code=400,
            detail="Username already registered"
        )

    # Check if email already exists
    for existing_user in users_db:
        if existing_user["email"] == user.email:
            raise HTTPException(
                status_code=400,
                detail="Email already registered"
            )

    # Create new user
    hashed_password = get_password_hash(user.password)
    user_dict = {
        "id": next_user_id,
        "username": user.username,
        "email": user.email,
        "full_name": user.full_name,
        "hashed_password": hashed_password,
        "is_active": True,
        "created_at": datetime.utcnow()
    }
    users_db.append(user_dict)
    next_user_id += 1

    return User(**user_dict)

@app.post("/auth/login", response_model=Token)
def login_user(login_request: LoginRequest):
    user = authenticate_user(login_request.username, login_request.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid username or password"
        )

    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user.username}, expires_delta=access_token_expires
    )

    return {
        "access_token": access_token,
        "token_type": "bearer",
        "expires_in": ACCESS_TOKEN_EXPIRE_MINUTES * 60
    }

@app.get("/auth/me", response_model=User)
def get_current_user_profile(current_user: User = Depends(get_current_user)):
    return current_user

@app.put("/auth/me", response_model=User)
def update_user_profile(
    user_update: UserBase,
    current_user: User = Depends(get_current_user)
):
    # Find and update user in database
    for i, user in enumerate(users_db):
        if user["id"] == current_user.id:
            users_db[i].update({
                "username": user_update.username,
                "email": user_update.email,
                "full_name": user_update.full_name
            })
            return User(**users_db[i])

    raise HTTPException(status_code=404, detail="User not found")
```

## Validation Requirements

### Password Validation
```python
import re

def validate_password(password: str) -> bool:
    """
    Password must contain:
    - At least 8 characters
    - At least one uppercase letter
    - At least one lowercase letter
    - At least one digit
    - At least one special character
    """
    if len(password) < 8:
        return False

    if not re.search(r"[A-Z]", password):
        return False

    if not re.search(r"[a-z]", password):
        return False

    if not re.search(r"\d", password):
        return False

    if not re.search(r"[!@#$%^&*(),.?\":{}|<>]", password):
        return False

    return True
```

### Username Validation
- 3-20 characters long
- Alphanumeric characters and underscores only
- Must start with a letter
- Case-insensitive uniqueness

## Security Considerations

### Password Security
- Use bcrypt for password hashing
- Never store plain text passwords
- Implement password strength requirements
- Consider password history to prevent reuse

### JWT Token Security
- Use strong secret keys
- Set appropriate expiration times
- Include token refresh mechanism
- Consider token blacklisting for logout

### API Security
- Implement rate limiting
- Use HTTPS in production
- Validate all inputs
- Don't leak information in error messages

## TDD Process

### Red Phase - Write Failing Tests
1. Test user registration with valid data
2. Test duplicate username/email handling
3. Test password validation rules
4. Test login with valid credentials
5. Test login with invalid credentials
6. Test protected route access
7. Test token validation

### Green Phase - Minimal Implementation
1. Create basic models and endpoints
2. Implement password hashing
3. Add JWT token generation
4. Implement authentication dependency
5. Add validation logic

### Refactor Phase - Improve Security
1. Add comprehensive input validation
2. Improve error handling
3. Add security headers
4. Optimize password hashing
5. Add logging for security events

## Common Challenges

### Challenge 1: Password Security
Ensuring passwords are properly hashed and verified.

**Solution**: Use established libraries like passlib with bcrypt.

### Challenge 2: JWT Token Management
Handling token creation, validation, and expiration.

**Solution**: Use python-jose library with proper error handling.

### Challenge 3: Test Authentication
Testing protected endpoints requires token management.

**Solution**: Create helper functions to authenticate test users.

## Success Criteria

- [ ] Secure user registration with password hashing
- [ ] JWT-based authentication working correctly
- [ ] Protected routes require valid authentication
- [ ] Comprehensive input validation
- [ ] Secure error handling (no information leakage)
- [ ] All security tests passing
- [ ] Password strength requirements enforced
- [ ] Token expiration and refresh working

## Extensions

1. **Password Reset** - Email-based password reset flow
2. **Two-Factor Authentication** - TOTP or SMS verification
3. **Social Login** - OAuth integration with Google/GitHub
4. **Role-Based Access** - Different user roles and permissions
5. **Session Management** - Track active sessions and logout

---

# 用户认证API

使用FastAPI构建安全的用户认证系统，包含JWT令牌、密码哈希和受保护路由。这个问题介绍了认证模式和安全最佳实践。

## 问题描述

创建一个用户认证API，提供：
- 安全密码存储的用户注册
- JWT令牌生成的用户登录
- 需要认证的受保护路由
- 令牌验证和刷新机制
- 用户档案管理

系统应实现现代安全实践，包括密码哈希、JWT令牌和适当的错误处理。

## 需求

### 功能需求
1. **用户注册** - 创建带验证的新用户账户
2. **用户登录** - 认证用户并返回JWT令牌
3. **受保护路由** - 需要有效认证的端点
4. **令牌验证** - 在受保护端点上验证JWT令牌
5. **用户档案** - 允许用户查看和更新档案
6. **密码安全** - 存储前哈希密码
7. **错误处理** - 不泄露信息的安全错误消息

### 安全需求
- 使用bcrypt或类似工具进行密码哈希
- 带过期时间的JWT令牌
- 安全令牌验证
- 防护常见攻击（时序攻击等）
- 输入验证和清理
- 不泄露信息的适当错误消息

## 成功标准

- [ ] 安全的用户注册与密码哈希
- [ ] 基于JWT的认证正常工作
- [ ] 受保护路由需要有效认证
- [ ] 全面的输入验证
- [ ] 安全的错误处理（无信息泄露）
- [ ] 所有安全测试通过
- [ ] 强制密码强度要求
- [ ] 令牌过期和刷新正常工作