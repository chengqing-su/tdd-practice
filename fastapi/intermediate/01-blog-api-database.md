# Blog API with Database

Build a comprehensive blog API using FastAPI with SQLAlchemy ORM, featuring complex entity relationships, database testing, and advanced query patterns. This problem introduces database integration and ORM best practices.

[中文版本](#带数据库的博客api)

## Problem Description

Create a full-featured blog API that supports:
- User management with authentication
- Blog post creation and management
- Comment system with threading
- Tag and category organization
- Search and filtering capabilities
- Database relationships and constraints

The API should demonstrate proper database design, ORM usage, and comprehensive testing strategies.

## Requirements

### Functional Requirements
1. **User Management** - Registration, authentication, user profiles
2. **Post Management** - Create, edit, delete, publish blog posts
3. **Comment System** - Nested comments with threading support
4. **Category System** - Organize posts by categories
5. **Tagging System** - Tag posts for better organization
6. **Search Functionality** - Full-text search across posts
7. **Pagination** - Efficient pagination for large datasets

### Technical Requirements
- SQLAlchemy ORM with PostgreSQL/SQLite
- Database migrations with Alembic
- Comprehensive database testing
- Query optimization and N+1 prevention
- Database transactions and rollbacks
- Proper foreign key relationships

## Database Schema

### Entity Relationships
```
Users (1) ←→ (N) Posts
Users (1) ←→ (N) Comments
Posts (1) ←→ (N) Comments
Posts (N) ←→ (N) Tags (many-to-many)
Posts (N) ←→ (1) Category
Comments (1) ←→ (N) Comments (self-referential)
```

### Database Models
```python
# User Model
class User:
    id: int (PK)
    username: str (unique)
    email: str (unique)
    password_hash: str
    full_name: str
    bio: str
    avatar_url: str
    is_active: bool
    is_verified: bool
    created_at: datetime
    updated_at: datetime

# Post Model
class Post:
    id: int (PK)
    title: str
    content: text
    excerpt: str
    slug: str (unique)
    status: enum (draft, published, archived)
    author_id: int (FK → User)
    category_id: int (FK → Category)
    view_count: int
    like_count: int
    published_at: datetime
    created_at: datetime
    updated_at: datetime

# Comment Model
class Comment:
    id: int (PK)
    content: text
    author_id: int (FK → User)
    post_id: int (FK → Post)
    parent_id: int (FK → Comment, nullable)
    is_approved: bool
    created_at: datetime
    updated_at: datetime

# Category Model
class Category:
    id: int (PK)
    name: str (unique)
    description: str
    slug: str (unique)
    created_at: datetime

# Tag Model
class Tag:
    id: int (PK)
    name: str (unique)
    description: str
    created_at: datetime

# PostTag Association
class PostTag:
    post_id: int (FK → Post)
    tag_id: int (FK → Tag)
```

## API Specification

### Endpoints

| Method | Endpoint | Description | Authentication |
|--------|----------|-------------|----------------|
| POST | `/auth/register` | User registration | No |
| POST | `/auth/login` | User login | No |
| GET | `/users/me` | Get current user | Yes |
| PUT | `/users/me` | Update profile | Yes |
| POST | `/posts/` | Create post | Yes |
| GET | `/posts/` | List posts with filters | No |
| GET | `/posts/{slug}` | Get post by slug | No |
| PUT | `/posts/{id}` | Update post | Yes (owner) |
| DELETE | `/posts/{id}` | Delete post | Yes (owner) |
| POST | `/posts/{id}/comments` | Add comment | Yes |
| GET | `/posts/{id}/comments` | Get comments | No |
| PUT | `/comments/{id}` | Update comment | Yes (owner) |
| DELETE | `/comments/{id}` | Delete comment | Yes (owner) |
| GET | `/categories/` | List categories | No |
| POST | `/categories/` | Create category | Yes (admin) |
| GET | `/tags/` | List tags | No |
| POST | `/tags/` | Create tag | Yes |

## Test Examples

### Database Testing Setup
```python
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from fastapi.testclient import TestClient
from app.database import Base, get_db
from app.main import app
from app.models import User, Post, Comment, Category, Tag

# Test database configuration
SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(
    SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False}
)
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

@pytest.fixture
def db_session():
    """Create a fresh database for each test"""
    Base.metadata.create_all(bind=engine)
    db = TestingSessionLocal()
    try:
        yield db
    finally:
        db.close()
        Base.metadata.drop_all(bind=engine)

@pytest.fixture
def client(db_session):
    """Create test client with database dependency override"""
    def override_get_db():
        try:
            yield db_session
        finally:
            pass

    app.dependency_overrides[get_db] = override_get_db
    yield TestClient(app)
    app.dependency_overrides.clear()

@pytest.fixture
def test_user(db_session):
    """Create a test user"""
    user = User(
        username="testuser",
        email="test@example.com",
        password_hash="hashed_password",
        full_name="Test User"
    )
    db_session.add(user)
    db_session.commit()
    db_session.refresh(user)
    return user

@pytest.fixture
def test_category(db_session):
    """Create a test category"""
    category = Category(
        name="Technology",
        description="Tech-related posts",
        slug="technology"
    )
    db_session.add(category)
    db_session.commit()
    db_session.refresh(category)
    return category
```

### Model Tests
```python
def test_create_user(db_session):
    user = User(
        username="newuser",
        email="newuser@example.com",
        password_hash="hashed_password",
        full_name="New User"
    )
    db_session.add(user)
    db_session.commit()

    assert user.id is not None
    assert user.username == "newuser"
    assert user.created_at is not None

def test_user_posts_relationship(db_session, test_user, test_category):
    post = Post(
        title="Test Post",
        content="This is a test post",
        slug="test-post",
        author_id=test_user.id,
        category_id=test_category.id,
        status="published"
    )
    db_session.add(post)
    db_session.commit()

    # Test relationship
    assert len(test_user.posts) == 1
    assert test_user.posts[0].title == "Test Post"
    assert post.author.username == "testuser"

def test_comment_threading(db_session, test_user, test_category):
    # Create post
    post = Post(
        title="Test Post",
        content="Content",
        slug="test-post",
        author_id=test_user.id,
        category_id=test_category.id
    )
    db_session.add(post)
    db_session.commit()

    # Create parent comment
    parent_comment = Comment(
        content="Parent comment",
        author_id=test_user.id,
        post_id=post.id
    )
    db_session.add(parent_comment)
    db_session.commit()

    # Create child comment
    child_comment = Comment(
        content="Child comment",
        author_id=test_user.id,
        post_id=post.id,
        parent_id=parent_comment.id
    )
    db_session.add(child_comment)
    db_session.commit()

    # Test threading
    assert len(parent_comment.replies) == 1
    assert child_comment.parent == parent_comment
```

### API Tests
```python
def test_create_post(client, test_user, test_category):
    # Login first to get token
    login_response = client.post("/auth/login", json={
        "username": "testuser",
        "password": "testpassword"
    })
    token = login_response.json()["access_token"]
    headers = {"Authorization": f"Bearer {token}"}

    post_data = {
        "title": "New Blog Post",
        "content": "This is the content of the new blog post",
        "category_id": test_category.id,
        "tags": ["python", "fastapi"],
        "status": "published"
    }

    response = client.post("/posts/", json=post_data, headers=headers)

    assert response.status_code == 201
    assert response.json()["title"] == "New Blog Post"
    assert response.json()["slug"] == "new-blog-post"
    assert "python" in [tag["name"] for tag in response.json()["tags"]]

def test_get_posts_with_pagination(client, test_user, test_category):
    # Create multiple posts
    for i in range(25):
        post = Post(
            title=f"Post {i}",
            content=f"Content {i}",
            slug=f"post-{i}",
            author_id=test_user.id,
            category_id=test_category.id,
            status="published"
        )
        db_session.add(post)
    db_session.commit()

    # Test pagination
    response = client.get("/posts/?page=1&size=10")
    assert response.status_code == 200
    data = response.json()

    assert len(data["items"]) == 10
    assert data["total"] == 25
    assert data["page"] == 1
    assert data["pages"] == 3

def test_search_posts(client, test_user, test_category):
    # Create posts with different content
    posts_data = [
        {"title": "Python Tutorial", "content": "Learn Python programming"},
        {"title": "FastAPI Guide", "content": "Building APIs with FastAPI"},
        {"title": "Database Design", "content": "Designing efficient databases"}
    ]

    for post_data in posts_data:
        post = Post(
            **post_data,
            slug=post_data["title"].lower().replace(" ", "-"),
            author_id=test_user.id,
            category_id=test_category.id,
            status="published"
        )
        db_session.add(post)
    db_session.commit()

    # Search for Python-related posts
    response = client.get("/posts/search?q=python")
    assert response.status_code == 200

    results = response.json()["items"]
    assert len(results) == 2  # Should find "Python Tutorial" and "Learn Python programming"

def test_add_comment_to_post(client, test_user, test_category):
    # Create a post
    post = Post(
        title="Test Post",
        content="Test content",
        slug="test-post",
        author_id=test_user.id,
        category_id=test_category.id,
        status="published"
    )
    db_session.add(post)
    db_session.commit()

    # Login and add comment
    login_response = client.post("/auth/login", json={
        "username": "testuser",
        "password": "testpassword"
    })
    token = login_response.json()["access_token"]
    headers = {"Authorization": f"Bearer {token}"}

    comment_data = {"content": "This is a great post!"}
    response = client.post(f"/posts/{post.id}/comments", json=comment_data, headers=headers)

    assert response.status_code == 201
    assert response.json()["content"] == "This is a great post!"
    assert response.json()["author"]["username"] == "testuser"

def test_nested_comments(client, test_user, test_category):
    # Create post and parent comment (setup from previous test)
    # ... setup code ...

    # Add reply to comment
    reply_data = {
        "content": "I agree with your comment!",
        "parent_id": parent_comment.id
    }
    response = client.post(f"/posts/{post.id}/comments", json=reply_data, headers=headers)

    assert response.status_code == 201
    assert response.json()["parent_id"] == parent_comment.id

    # Get comments and verify threading
    comments_response = client.get(f"/posts/{post.id}/comments")
    comments = comments_response.json()

    parent = next(c for c in comments if c["id"] == parent_comment.id)
    assert len(parent["replies"]) == 1
```

## Implementation Guide

### Step 1: Database Models
```python
from sqlalchemy import Column, Integer, String, Text, DateTime, Boolean, ForeignKey, Table
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship
from sqlalchemy.sql import func
from datetime import datetime

Base = declarative_base()

# Association table for many-to-many relationship
post_tags = Table(
    'post_tags',
    Base.metadata,
    Column('post_id', Integer, ForeignKey('posts.id'), primary_key=True),
    Column('tag_id', Integer, ForeignKey('tags.id'), primary_key=True)
)

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    username = Column(String(50), unique=True, index=True, nullable=False)
    email = Column(String(100), unique=True, index=True, nullable=False)
    password_hash = Column(String(128), nullable=False)
    full_name = Column(String(100))
    bio = Column(Text)
    avatar_url = Column(String(255))
    is_active = Column(Boolean, default=True)
    is_verified = Column(Boolean, default=False)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())

    # Relationships
    posts = relationship("Post", back_populates="author", cascade="all, delete-orphan")
    comments = relationship("Comment", back_populates="author", cascade="all, delete-orphan")

class Category(Base):
    __tablename__ = "categories"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(50), unique=True, nullable=False)
    description = Column(Text)
    slug = Column(String(60), unique=True, nullable=False, index=True)
    created_at = Column(DateTime(timezone=True), server_default=func.now())

    # Relationships
    posts = relationship("Post", back_populates="category")

class Tag(Base):
    __tablename__ = "tags"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(30), unique=True, nullable=False, index=True)
    description = Column(Text)
    created_at = Column(DateTime(timezone=True), server_default=func.now())

    # Relationships
    posts = relationship("Post", secondary=post_tags, back_populates="tags")

class Post(Base):
    __tablename__ = "posts"

    id = Column(Integer, primary_key=True, index=True)
    title = Column(String(200), nullable=False)
    content = Column(Text, nullable=False)
    excerpt = Column(String(500))
    slug = Column(String(220), unique=True, nullable=False, index=True)
    status = Column(String(20), default="draft")  # draft, published, archived
    view_count = Column(Integer, default=0)
    like_count = Column(Integer, default=0)
    published_at = Column(DateTime(timezone=True))
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())

    # Foreign Keys
    author_id = Column(Integer, ForeignKey("users.id"), nullable=False)
    category_id = Column(Integer, ForeignKey("categories.id"))

    # Relationships
    author = relationship("User", back_populates="posts")
    category = relationship("Category", back_populates="posts")
    comments = relationship("Comment", back_populates="post", cascade="all, delete-orphan")
    tags = relationship("Tag", secondary=post_tags, back_populates="posts")

class Comment(Base):
    __tablename__ = "comments"

    id = Column(Integer, primary_key=True, index=True)
    content = Column(Text, nullable=False)
    is_approved = Column(Boolean, default=True)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())

    # Foreign Keys
    author_id = Column(Integer, ForeignKey("users.id"), nullable=False)
    post_id = Column(Integer, ForeignKey("posts.id"), nullable=False)
    parent_id = Column(Integer, ForeignKey("comments.id"))

    # Relationships
    author = relationship("User", back_populates="comments")
    post = relationship("Post", back_populates="comments")
    parent = relationship("Comment", remote_side=[id], back_populates="replies")
    replies = relationship("Comment", back_populates="parent", cascade="all, delete-orphan")
```

### Step 2: Database Configuration
```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from sqlalchemy.ext.declarative import declarative_base
import os

SQLALCHEMY_DATABASE_URL = os.getenv(
    "DATABASE_URL",
    "postgresql://user:password@localhost/blog_db"
)

engine = create_engine(SQLALCHEMY_DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### Step 3: Repository Pattern
```python
from sqlalchemy.orm import Session, joinedload
from sqlalchemy import and_, or_, func
from typing import List, Optional
from app.models import Post, User, Comment, Tag, Category

class PostRepository:
    def __init__(self, db: Session):
        self.db = db

    def create(self, post_data: dict, author_id: int) -> Post:
        post = Post(**post_data, author_id=author_id)
        self.db.add(post)
        self.db.commit()
        self.db.refresh(post)
        return post

    def get_by_id(self, post_id: int) -> Optional[Post]:
        return self.db.query(Post).options(
            joinedload(Post.author),
            joinedload(Post.category),
            joinedload(Post.tags),
            joinedload(Post.comments).joinedload(Comment.author)
        ).filter(Post.id == post_id).first()

    def get_by_slug(self, slug: str) -> Optional[Post]:
        return self.db.query(Post).options(
            joinedload(Post.author),
            joinedload(Post.category),
            joinedload(Post.tags)
        ).filter(Post.slug == slug).first()

    def get_published_posts(
        self,
        page: int = 1,
        size: int = 10,
        category_id: Optional[int] = None,
        tag_name: Optional[str] = None
    ) -> tuple[List[Post], int]:
        query = self.db.query(Post).filter(Post.status == "published")

        if category_id:
            query = query.filter(Post.category_id == category_id)

        if tag_name:
            query = query.join(Post.tags).filter(Tag.name == tag_name)

        total = query.count()
        posts = query.options(
            joinedload(Post.author),
            joinedload(Post.category),
            joinedload(Post.tags)
        ).offset((page - 1) * size).limit(size).all()

        return posts, total

    def search_posts(self, query: str, page: int = 1, size: int = 10) -> tuple[List[Post], int]:
        search_filter = or_(
            Post.title.contains(query),
            Post.content.contains(query),
            Post.excerpt.contains(query)
        )

        posts_query = self.db.query(Post).filter(
            and_(Post.status == "published", search_filter)
        )

        total = posts_query.count()
        posts = posts_query.options(
            joinedload(Post.author),
            joinedload(Post.category)
        ).offset((page - 1) * size).limit(size).all()

        return posts, total

    def update(self, post_id: int, update_data: dict) -> Optional[Post]:
        post = self.db.query(Post).filter(Post.id == post_id).first()
        if post:
            for key, value in update_data.items():
                setattr(post, key, value)
            self.db.commit()
            self.db.refresh(post)
        return post

    def delete(self, post_id: int) -> bool:
        post = self.db.query(Post).filter(Post.id == post_id).first()
        if post:
            self.db.delete(post)
            self.db.commit()
            return True
        return False
```

### Step 4: Service Layer
```python
from typing import List, Optional
from sqlalchemy.orm import Session
from app.repositories import PostRepository, UserRepository
from app.schemas import PostCreate, PostUpdate, PostResponse
from app.utils import generate_slug

class PostService:
    def __init__(self, db: Session):
        self.db = db
        self.post_repo = PostRepository(db)
        self.user_repo = UserRepository(db)

    def create_post(self, post_data: PostCreate, author_id: int) -> PostResponse:
        # Generate slug from title
        slug = generate_slug(post_data.title)

        # Ensure slug is unique
        counter = 1
        original_slug = slug
        while self.post_repo.get_by_slug(slug):
            slug = f"{original_slug}-{counter}"
            counter += 1

        # Handle tags
        tag_ids = []
        if post_data.tags:
            tag_ids = self._get_or_create_tags(post_data.tags)

        post_dict = post_data.dict(exclude={"tags"})
        post_dict["slug"] = slug

        if post_data.status == "published" and not post_dict.get("published_at"):
            post_dict["published_at"] = datetime.utcnow()

        post = self.post_repo.create(post_dict, author_id)

        # Associate tags
        if tag_ids:
            self._associate_tags(post.id, tag_ids)

        return PostResponse.from_orm(post)

    def get_posts(
        self,
        page: int = 1,
        size: int = 10,
        category_id: Optional[int] = None,
        tag_name: Optional[str] = None
    ) -> dict:
        posts, total = self.post_repo.get_published_posts(
            page=page, size=size, category_id=category_id, tag_name=tag_name
        )

        return {
            "items": [PostResponse.from_orm(post) for post in posts],
            "total": total,
            "page": page,
            "size": size,
            "pages": (total + size - 1) // size
        }

    def search_posts(self, query: str, page: int = 1, size: int = 10) -> dict:
        posts, total = self.post_repo.search_posts(query, page, size)

        return {
            "items": [PostResponse.from_orm(post) for post in posts],
            "total": total,
            "page": page,
            "size": size,
            "pages": (total + size - 1) // size,
            "query": query
        }

    def _get_or_create_tags(self, tag_names: List[str]) -> List[int]:
        tag_ids = []
        for tag_name in tag_names:
            tag = self.db.query(Tag).filter(Tag.name == tag_name.lower()).first()
            if not tag:
                tag = Tag(name=tag_name.lower())
                self.db.add(tag)
                self.db.commit()
                self.db.refresh(tag)
            tag_ids.append(tag.id)
        return tag_ids

    def _associate_tags(self, post_id: int, tag_ids: List[int]):
        post = self.db.query(Post).filter(Post.id == post_id).first()
        if post:
            tags = self.db.query(Tag).filter(Tag.id.in_(tag_ids)).all()
            post.tags = tags
            self.db.commit()
```

## Query Optimization

### N+1 Problem Prevention
```python
# Bad: N+1 queries
posts = db.query(Post).all()
for post in posts:
    print(post.author.username)  # Triggers N additional queries

# Good: Eager loading
posts = db.query(Post).options(joinedload(Post.author)).all()
for post in posts:
    print(post.author.username)  # No additional queries
```

### Efficient Pagination
```python
def get_posts_efficient(db: Session, page: int, size: int):
    # Use offset and limit for pagination
    offset = (page - 1) * size

    # Get total count efficiently
    total = db.query(func.count(Post.id)).filter(Post.status == "published").scalar()

    # Get posts with eager loading
    posts = (
        db.query(Post)
        .options(joinedload(Post.author), joinedload(Post.category))
        .filter(Post.status == "published")
        .order_by(Post.created_at.desc())
        .offset(offset)
        .limit(size)
        .all()
    )

    return posts, total
```

## Success Criteria

- [ ] Complete database schema with relationships
- [ ] SQLAlchemy ORM integration working
- [ ] Database migrations with Alembic
- [ ] Comprehensive database testing
- [ ] Query optimization implemented
- [ ] Repository pattern for data access
- [ ] All CRUD operations working
- [ ] Search and pagination functional

## Extensions

1. **Full-Text Search** - PostgreSQL full-text search or Elasticsearch
2. **Caching Layer** - Redis caching for frequently accessed data
3. **Image Handling** - Post image uploads and processing
4. **Email Notifications** - Comment notifications via email
5. **Social Features** - Like/dislike posts, follow users

---

# 带数据库的博客API

使用FastAPI和SQLAlchemy ORM构建综合博客API，具有复杂实体关系、数据库测试和高级查询模式。这个问题介绍了数据库集成和ORM最佳实践。

## 问题描述

创建一个功能齐全的博客API，支持：
- 带认证的用户管理
- 博客文章创建和管理
- 带线程的评论系统
- 标签和分类组织
- 搜索和过滤功能
- 数据库关系和约束

API应演示适当的数据库设计、ORM使用和综合测试策略。

## 成功标准

- [ ] 带关系的完整数据库模式
- [ ] SQLAlchemy ORM集成工作
- [ ] 使用Alembic的数据库迁移
- [ ] 全面的数据库测试
- [ ] 实施查询优化
- [ ] 数据访问的仓库模式
- [ ] 所有CRUD操作工作
- [ ] 搜索和分页功能