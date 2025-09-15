# File Upload API

Build a robust file upload and management API using FastAPI with comprehensive file validation, storage handling, and progress tracking. This problem focuses on file handling patterns and multipart request processing.

[中文版本](#文件上传api)

## Problem Description

Create a file upload API that provides:
- Secure file upload with validation
- Multiple file format support
- File size limits and validation
- File metadata extraction and storage
- File download and streaming
- Progress tracking for large uploads
- File organization with folders/categories

The API should handle various file types while ensuring security and providing detailed feedback about upload status.

## Requirements

### Functional Requirements
1. **File Upload** - Accept single and multiple file uploads
2. **File Validation** - Validate file types, sizes, and content
3. **File Storage** - Store files securely with unique identifiers
4. **File Metadata** - Extract and store file information
5. **File Download** - Serve files with proper headers
6. **File Management** - List, update, and delete files
7. **Folder Organization** - Organize files into folders/categories

### Security Requirements
- File type validation (whitelist approach)
- File size limits to prevent abuse
- Virus scanning integration (placeholder)
- Secure file naming to prevent path traversal
- Content-Type validation
- File content inspection

## API Specification

### Endpoints

| Method | Endpoint | Description | Content-Type | Status Code |
|--------|----------|-------------|--------------|-------------|
| POST | `/files/upload` | Upload single file | multipart/form-data | 201, 400 |
| POST | `/files/upload-multiple` | Upload multiple files | multipart/form-data | 201, 400 |
| GET | `/files/` | List files with pagination | application/json | 200 |
| GET | `/files/{file_id}` | Get file metadata | application/json | 200, 404 |
| GET | `/files/{file_id}/download` | Download file | file content | 200, 404 |
| PUT | `/files/{file_id}` | Update file metadata | application/json | 200, 404 |
| DELETE | `/files/{file_id}` | Delete file | application/json | 204, 404 |
| POST | `/folders/` | Create folder | application/json | 201, 400 |

### Data Models

**File Metadata:**
```json
{
  "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
  "filename": "document.pdf",
  "original_filename": "My Document.pdf",
  "content_type": "application/pdf",
  "size": 1024000,
  "folder_id": 1,
  "upload_date": "2023-01-01T12:00:00Z",
  "md5_hash": "5d41402abc4b2a76b9719d911017c592",
  "description": "Important document",
  "tags": ["work", "important"],
  "download_count": 5,
  "is_public": false
}
```

**Upload Progress:**
```json
{
  "upload_id": "upload_12345",
  "status": "uploading",
  "progress": 75.5,
  "uploaded_bytes": 756000,
  "total_bytes": 1024000,
  "estimated_time_remaining": 2.5
}
```

## Test Examples

### File Upload Tests
```python
from fastapi.testclient import TestClient
from main import app
import io
import pytest
from PIL import Image

client = TestClient(app)

def test_upload_valid_image():
    # Create a test image
    image = Image.new('RGB', (100, 100), color='red')
    image_bytes = io.BytesIO()
    image.save(image_bytes, format='PNG')
    image_bytes.seek(0)

    response = client.post(
        "/files/upload",
        files={"file": ("test.png", image_bytes, "image/png")},
        data={"description": "Test image"}
    )

    assert response.status_code == 201
    assert response.json()["filename"] == "test.png"
    assert response.json()["content_type"] == "image/png"
    assert "id" in response.json()

def test_upload_file_too_large():
    # Create a large file (simulate)
    large_content = b"x" * (11 * 1024 * 1024)  # 11MB, over limit

    response = client.post(
        "/files/upload",
        files={"file": ("large.txt", large_content, "text/plain")}
    )

    assert response.status_code == 400
    assert "size" in response.json()["detail"].lower()

def test_upload_invalid_file_type():
    # Try to upload an executable file
    response = client.post(
        "/files/upload",
        files={"file": ("malware.exe", b"fake exe content", "application/octet-stream")}
    )

    assert response.status_code == 400
    assert "file type" in response.json()["detail"].lower()

def test_upload_multiple_files():
    files = [
        ("files", ("file1.txt", b"content1", "text/plain")),
        ("files", ("file2.txt", b"content2", "text/plain"))
    ]

    response = client.post("/files/upload-multiple", files=files)

    assert response.status_code == 201
    assert len(response.json()["uploaded_files"]) == 2
    assert response.json()["success_count"] == 2
    assert response.json()["error_count"] == 0

def test_upload_mixed_valid_invalid_files():
    files = [
        ("files", ("valid.txt", b"valid content", "text/plain")),
        ("files", ("invalid.exe", b"exe content", "application/octet-stream"))
    ]

    response = client.post("/files/upload-multiple", files=files)

    assert response.status_code == 207  # Multi-status
    assert response.json()["success_count"] == 1
    assert response.json()["error_count"] == 1

def test_download_file():
    # First upload a file
    upload_response = client.post(
        "/files/upload",
        files={"file": ("download_test.txt", b"test content", "text/plain")}
    )
    file_id = upload_response.json()["id"]

    # Then download it
    download_response = client.get(f"/files/{file_id}/download")

    assert download_response.status_code == 200
    assert download_response.content == b"test content"
    assert "attachment" in download_response.headers.get("content-disposition", "")

def test_get_file_metadata():
    # Upload file first
    upload_response = client.post(
        "/files/upload",
        files={"file": ("metadata_test.txt", b"test content", "text/plain")},
        data={"description": "Test file for metadata"}
    )
    file_id = upload_response.json()["id"]

    # Get metadata
    response = client.get(f"/files/{file_id}")

    assert response.status_code == 200
    assert response.json()["filename"] == "metadata_test.txt"
    assert response.json()["description"] == "Test file for metadata"
    assert response.json()["size"] > 0

def test_delete_file():
    # Upload file first
    upload_response = client.post(
        "/files/upload",
        files={"file": ("delete_test.txt", b"content to delete", "text/plain")}
    )
    file_id = upload_response.json()["id"]

    # Delete file
    delete_response = client.delete(f"/files/{file_id}")
    assert delete_response.status_code == 204

    # Verify file is gone
    get_response = client.get(f"/files/{file_id}")
    assert get_response.status_code == 404

def test_update_file_metadata():
    # Upload file first
    upload_response = client.post(
        "/files/upload",
        files={"file": ("update_test.txt", b"content", "text/plain")}
    )
    file_id = upload_response.json()["id"]

    # Update metadata
    update_data = {
        "description": "Updated description",
        "tags": ["updated", "test"]
    }
    response = client.put(f"/files/{file_id}", json=update_data)

    assert response.status_code == 200
    assert response.json()["description"] == "Updated description"
    assert "updated" in response.json()["tags"]
```

## Implementation Guide

### Step 1: Dependencies and Configuration
```python
from fastapi import FastAPI, File, UploadFile, HTTPException, Form, Depends
from fastapi.responses import FileResponse, StreamingResponse
from pydantic import BaseModel, Field, validator
from typing import List, Optional, Dict, Any
import uuid
import os
import hashlib
import shutil
from pathlib import Path
import mimetypes
from datetime import datetime

app = FastAPI(title="File Upload API")

# Configuration
UPLOAD_DIR = Path("uploads")
UPLOAD_DIR.mkdir(exist_ok=True)
MAX_FILE_SIZE = 10 * 1024 * 1024  # 10MB
ALLOWED_EXTENSIONS = {
    'txt', 'pdf', 'png', 'jpg', 'jpeg', 'gif', 'doc', 'docx',
    'xls', 'xlsx', 'ppt', 'pptx', 'zip', 'rar'
}
ALLOWED_MIME_TYPES = {
    'text/plain', 'application/pdf', 'image/png', 'image/jpeg',
    'image/gif', 'application/msword', 'application/zip'
}

# In-memory storage
files_db = []
folders_db = [{"id": 1, "name": "Default", "created_at": datetime.now()}]
next_file_id = 1
next_folder_id = 2
```

### Step 2: Pydantic Models
```python
class FileMetadata(BaseModel):
    id: str
    filename: str
    original_filename: str
    content_type: str
    size: int
    folder_id: Optional[int] = 1
    upload_date: datetime
    md5_hash: str
    description: Optional[str] = None
    tags: List[str] = Field(default_factory=list)
    download_count: int = 0
    is_public: bool = False

class FileUpdate(BaseModel):
    description: Optional[str] = None
    tags: Optional[List[str]] = None
    folder_id: Optional[int] = None
    is_public: Optional[bool] = None

    @validator('tags')
    def validate_tags(cls, v):
        if v is not None:
            return [tag.strip().lower() for tag in v if tag.strip()]
        return v

class UploadResponse(BaseModel):
    success: bool
    file: Optional[FileMetadata] = None
    error: Optional[str] = None

class MultiUploadResponse(BaseModel):
    success_count: int
    error_count: int
    uploaded_files: List[FileMetadata]
    errors: List[Dict[str, str]]

class Folder(BaseModel):
    id: int
    name: str
    description: Optional[str] = None
    created_at: datetime

class FolderCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=50)
    description: Optional[str] = Field(None, max_length=200)

    @validator('name')
    def validate_folder_name(cls, v):
        # Sanitize folder name
        invalid_chars = ['/', '\\', ':', '*', '?', '"', '<', '>', '|']
        if any(char in v for char in invalid_chars):
            raise ValueError('Folder name contains invalid characters')
        return v.strip()
```

### Step 3: File Validation Functions
```python
def validate_file_type(filename: str, content_type: str) -> bool:
    """Validate file type based on extension and MIME type"""
    file_ext = filename.split('.')[-1].lower()
    return (
        file_ext in ALLOWED_EXTENSIONS and
        content_type in ALLOWED_MIME_TYPES
    )

def validate_file_size(file: UploadFile) -> bool:
    """Check if file size is within limits"""
    file.file.seek(0, 2)  # Seek to end
    size = file.file.tell()
    file.file.seek(0)  # Reset to beginning
    return size <= MAX_FILE_SIZE

def sanitize_filename(filename: str) -> str:
    """Sanitize filename to prevent path traversal"""
    # Remove path components
    filename = os.path.basename(filename)
    # Remove or replace dangerous characters
    filename = "".join(c for c in filename if c.isalnum() or c in '._-')
    # Ensure filename is not empty
    if not filename:
        filename = "unnamed_file"
    return filename

def calculate_md5(file_path: Path) -> str:
    """Calculate MD5 hash of file"""
    hash_md5 = hashlib.md5()
    with open(file_path, "rb") as f:
        for chunk in iter(lambda: f.read(4096), b""):
            hash_md5.update(chunk)
    return hash_md5.hexdigest()

async def save_upload_file(upload_file: UploadFile, destination: Path) -> None:
    """Save uploaded file to destination"""
    with open(destination, "wb") as buffer:
        shutil.copyfileobj(upload_file.file, buffer)

def get_file_info(file_path: Path) -> Dict[str, Any]:
    """Extract file information"""
    stat = file_path.stat()
    return {
        "size": stat.st_size,
        "created": datetime.fromtimestamp(stat.st_ctime),
        "modified": datetime.fromtimestamp(stat.st_mtime)
    }
```

### Step 4: File Upload Endpoints
```python
@app.post("/files/upload", response_model=FileMetadata, status_code=201)
async def upload_file(
    file: UploadFile = File(...),
    description: Optional[str] = Form(None),
    folder_id: Optional[int] = Form(1),
    tags: Optional[str] = Form(None)  # Comma-separated tags
):
    # Validate file type
    if not validate_file_type(file.filename, file.content_type):
        raise HTTPException(
            status_code=400,
            detail=f"File type not allowed. Allowed types: {', '.join(ALLOWED_EXTENSIONS)}"
        )

    # Validate file size
    if not validate_file_size(file):
        raise HTTPException(
            status_code=400,
            detail=f"File size exceeds maximum allowed size of {MAX_FILE_SIZE // (1024*1024)}MB"
        )

    # Validate folder exists
    if folder_id and not any(f["id"] == folder_id for f in folders_db):
        raise HTTPException(status_code=400, detail="Folder does not exist")

    # Generate unique filename
    file_id = str(uuid.uuid4())
    sanitized_filename = sanitize_filename(file.filename)
    file_extension = Path(sanitized_filename).suffix
    stored_filename = f"{file_id}{file_extension}"

    # Create folder path
    folder_path = UPLOAD_DIR / str(folder_id or 1)
    folder_path.mkdir(exist_ok=True)
    file_path = folder_path / stored_filename

    try:
        # Save file
        await save_upload_file(file, file_path)

        # Calculate hash
        md5_hash = calculate_md5(file_path)

        # Parse tags
        tag_list = []
        if tags:
            tag_list = [tag.strip().lower() for tag in tags.split(',') if tag.strip()]

        # Create metadata
        file_metadata = FileMetadata(
            id=file_id,
            filename=stored_filename,
            original_filename=file.filename,
            content_type=file.content_type,
            size=file_path.stat().st_size,
            folder_id=folder_id or 1,
            upload_date=datetime.now(),
            md5_hash=md5_hash,
            description=description,
            tags=tag_list
        )

        # Store in database
        files_db.append(file_metadata.dict())

        return file_metadata

    except Exception as e:
        # Clean up file if something went wrong
        if file_path.exists():
            file_path.unlink()
        raise HTTPException(status_code=500, detail=f"Failed to upload file: {str(e)}")

@app.post("/files/upload-multiple", response_model=MultiUploadResponse)
async def upload_multiple_files(
    files: List[UploadFile] = File(...),
    folder_id: Optional[int] = Form(1)
):
    uploaded_files = []
    errors = []

    for file in files:
        try:
            # Use the single upload logic
            result = await upload_file(file, folder_id=folder_id)
            uploaded_files.append(result)
        except HTTPException as e:
            errors.append({
                "filename": file.filename,
                "error": e.detail
            })
        except Exception as e:
            errors.append({
                "filename": file.filename,
                "error": f"Unexpected error: {str(e)}"
            })

    return MultiUploadResponse(
        success_count=len(uploaded_files),
        error_count=len(errors),
        uploaded_files=uploaded_files,
        errors=errors
    )

@app.get("/files/{file_id}/download")
async def download_file(file_id: str):
    # Find file in database
    file_data = None
    for file_info in files_db:
        if file_info["id"] == file_id:
            file_data = file_info
            break

    if not file_data:
        raise HTTPException(status_code=404, detail="File not found")

    # Construct file path
    folder_path = UPLOAD_DIR / str(file_data["folder_id"])
    file_path = folder_path / file_data["filename"]

    if not file_path.exists():
        raise HTTPException(status_code=404, detail="File not found on disk")

    # Increment download count
    file_data["download_count"] += 1

    # Return file
    return FileResponse(
        path=file_path,
        filename=file_data["original_filename"],
        media_type=file_data["content_type"]
    )

@app.get("/files/", response_model=List[FileMetadata])
def list_files(
    folder_id: Optional[int] = None,
    skip: int = 0,
    limit: int = 100
):
    filtered_files = files_db

    if folder_id is not None:
        filtered_files = [f for f in filtered_files if f["folder_id"] == folder_id]

    # Apply pagination
    return [FileMetadata(**f) for f in filtered_files[skip:skip + limit]]

@app.get("/files/{file_id}", response_model=FileMetadata)
def get_file_metadata(file_id: str):
    for file_data in files_db:
        if file_data["id"] == file_id:
            return FileMetadata(**file_data)

    raise HTTPException(status_code=404, detail="File not found")

@app.put("/files/{file_id}", response_model=FileMetadata)
def update_file_metadata(file_id: str, update_data: FileUpdate):
    file_index = None
    for i, file_data in enumerate(files_db):
        if file_data["id"] == file_id:
            file_index = i
            break

    if file_index is None:
        raise HTTPException(status_code=404, detail="File not found")

    # Update fields
    update_dict = update_data.dict(exclude_unset=True)
    files_db[file_index].update(update_dict)

    return FileMetadata(**files_db[file_index])

@app.delete("/files/{file_id}", status_code=204)
def delete_file(file_id: str):
    file_index = None
    file_data = None

    for i, file_info in enumerate(files_db):
        if file_info["id"] == file_id:
            file_index = i
            file_data = file_info
            break

    if file_index is None:
        raise HTTPException(status_code=404, detail="File not found")

    # Delete physical file
    folder_path = UPLOAD_DIR / str(file_data["folder_id"])
    file_path = folder_path / file_data["filename"]
    if file_path.exists():
        file_path.unlink()

    # Remove from database
    files_db.pop(file_index)
```

## Advanced Features

### File Streaming for Large Files
```python
@app.get("/files/{file_id}/stream")
async def stream_file(file_id: str, range: Optional[str] = None):
    """Stream file with support for range requests"""
    file_data = get_file_by_id(file_id)
    file_path = get_file_path(file_data)

    file_size = file_path.stat().st_size
    start = 0
    end = file_size - 1

    # Parse range header
    if range:
        range_match = re.match(r'bytes=(\d+)-(\d*)', range)
        if range_match:
            start = int(range_match.group(1))
            if range_match.group(2):
                end = int(range_match.group(2))

    def file_generator():
        with open(file_path, 'rb') as f:
            f.seek(start)
            remaining = end - start + 1
            while remaining:
                chunk_size = min(8192, remaining)
                chunk = f.read(chunk_size)
                if not chunk:
                    break
                remaining -= len(chunk)
                yield chunk

    headers = {
        'Content-Range': f'bytes {start}-{end}/{file_size}',
        'Accept-Ranges': 'bytes',
        'Content-Length': str(end - start + 1),
    }

    return StreamingResponse(
        file_generator(),
        status_code=206 if range else 200,
        headers=headers,
        media_type=file_data["content_type"]
    )
```

### File Content Validation
```python
def validate_image_content(file_path: Path) -> bool:
    """Validate that image file contains actual image data"""
    try:
        from PIL import Image
        with Image.open(file_path) as img:
            img.verify()
        return True
    except Exception:
        return False

def scan_for_malware(file_path: Path) -> bool:
    """Placeholder for malware scanning integration"""
    # In production, integrate with antivirus service
    # For now, just check file size and basic patterns
    if file_path.stat().st_size > MAX_FILE_SIZE:
        return False

    # Check for suspicious file headers
    with open(file_path, 'rb') as f:
        header = f.read(10)
        # Example: reject files that start with executable headers
        if header.startswith(b'MZ'):  # Windows executable
            return False

    return True
```

## Success Criteria

- [ ] Secure file upload with validation
- [ ] Multiple file format support
- [ ] File size limits enforced
- [ ] Proper file metadata handling
- [ ] File download functionality
- [ ] Comprehensive error handling
- [ ] All upload tests passing
- [ ] Security measures implemented

## Extensions

1. **Image Processing** - Thumbnail generation, image resizing
2. **Cloud Storage** - Integration with AWS S3, Google Cloud Storage
3. **Progress Tracking** - Real-time upload progress via WebSockets
4. **File Versioning** - Keep multiple versions of files
5. **Access Control** - File-level permissions and sharing

---

# 文件上传API

使用FastAPI构建强大的文件上传和管理API，具有全面的文件验证、存储处理和进度跟踪。这个问题专注于文件处理模式和多部分请求处理。

## 问题描述

创建一个文件上传API，提供：
- 带验证的安全文件上传
- 多种文件格式支持
- 文件大小限制和验证
- 文件元数据提取和存储
- 文件下载和流传输
- 大文件上传的进度跟踪
- 文件夹/类别组织

API应处理各种文件类型，同时确保安全并提供上传状态的详细反馈。

## 成功标准

- [ ] 带验证的安全文件上传
- [ ] 多种文件格式支持
- [ ] 强制文件大小限制
- [ ] 适当的文件元数据处理
- [ ] 文件下载功能
- [ ] 全面的错误处理
- [ ] 所有上传测试通过
- [ ] 实施安全措施