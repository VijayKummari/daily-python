# Day 3 – HTTP & FastAPI Basics

**Placement Preparation Series | Backend Python**

## Overview

This assignment covers HTTP fundamentals, REST API methods, FastAPI development, Pydantic validation, response models, exception handling, and file uploads.

The objective is to understand how backend APIs work and implement a basic REST API using Python and FastAPI.

---

# Task A – HTTP Methods and Status Codes

HTTP methods define the type of operation a client wants to perform on a server.

| HTTP Method | Purpose               | Real-World Example   | Status Code    |
| ----------- | --------------------- | -------------------- | -------------- |
| GET         | Retrieve data         | Fetch all products   | 200 OK         |
| POST        | Create data           | Register a new user  | 201 Created    |
| PUT         | Replace existing data | Replace user profile | 200 OK         |
| PATCH       | Partially update data | Update username      | 200 OK         |
| DELETE      | Remove data           | Delete a product     | 204 No Content |

### Important Concepts

* GET is used to retrieve resources.
* POST is used to create resources or process submissions.
* PUT replaces an existing resource.
* PATCH updates specific fields of a resource.
* DELETE removes a resource.

**Idempotency:** An operation is idempotent if repeating the same request produces the same intended effect.

GET, PUT, and DELETE are idempotent. POST is not guaranteed to be idempotent.

---

# Task B – Build a FastAPI Application

### 1. Project Setup

Install the required dependencies:

```bash
pip install fastapi "uvicorn[standard]" pydantic python-multipart
```

### 2. Project Structure

```text
Day-3/
│
├── main.py
├── requirements.txt
└── README.md
```

### 3. Create requirements.txt

```text
fastapi
uvicorn[standard]
pydantic
python-multipart
```

### 4. Implement main.py

The following application implements:

* GET /items/{item_id}
* GET /items?limit=...
* POST /items
* Pydantic validation
* Response models
* HTTP exception handling
* File upload functionality

```python
from fastapi import FastAPI, HTTPException, Query, UploadFile, File
from pydantic import BaseModel, Field
from typing import List

app = FastAPI(
    title="Day 3 - HTTP & FastAPI",
    description="Placement Preparation Assignment",
    version="1.0.0"
)

# In-memory database
items = []

# Request Model
class ItemCreate(BaseModel):
    name: str = Field(min_length=1, max_length=100)
    price: float = Field(gt=0)

# Response Model
class ItemResponse(BaseModel):
    id: int
    name: str
    price: float


# 1. GET - Retrieve all items
@app.get("/items", response_model=List[ItemResponse])
def get_items(limit: int = Query(default=10, ge=1, le=100)):

    return items[:limit]


# 2. GET - Retrieve an item by ID
@app.get("/items/{item_id}", response_model=ItemResponse)
def get_item(item_id: int):

    for item in items:
        if item["id"] == item_id:
            return item

    raise HTTPException(
        status_code=404,
        detail="Item not found"
    )


# 3. POST - Create a new item
@app.post(
    "/items",
    response_model=ItemResponse,
    status_code=201
)
def create_item(item: ItemCreate):

    new_item = {
        "id": len(items) + 1,
        "name": item.name.strip(),
        "price": item.price
    }

    items.append(new_item)

    return new_item


# 4. File Upload
@app.post("/upload")
async def upload_file(file: UploadFile = File(...)):

    allowed_types = [
        "image/jpeg",
        "image/png",
        "application/pdf"
    ]

    if file.content_type not in allowed_types:
        raise HTTPException(
            status_code=400,
            detail="File type not allowed"
        )

    content = await file.read()

    max_size = 5 * 1024 * 1024

    if len(content) > max_size:
        raise HTTPException(
            status_code=413,
            detail="File size exceeds 5 MB"
        )

    return {
        "filename": file.filename,
        "content_type": file.content_type,
        "size": len(content),
        "message": "File validated successfully"
    }


# Root Endpoint
@app.get("/")
def home():

    return {
        "message": "Day 3 FastAPI Assignment is running"
    }
```

### 5. Run the Application

```bash
uvicorn main:app --reload
```

Open the following URL in your browser:

```text
http://127.0.0.1:8000/docs
```

FastAPI provides interactive Swagger documentation where all endpoints can be tested.

**Note:** This implementation uses an in-memory list for demonstration. Data will reset whenever the application restarts.

---

# Task C – Pydantic Data Validation

Pydantic is used to validate incoming request data.

In this assignment, the following model is implemented:

```python
class ItemCreate(BaseModel):
    name: str = Field(min_length=1, max_length=100)
    price: float = Field(gt=0)
```

### Validation Rules

| Field | Condition              |
| ----- | ---------------------- |
| name  | Required               |
| name  | Minimum length: 1      |
| name  | Maximum length: 100    |
| price | Required               |
| price | Must be greater than 0 |

### Test Case 1 – Valid Input

Request:

```json
{
    "name": "Laptop",
    "price": 55000
}
```

Expected response:

```json
{
    "id": 1,
    "name": "Laptop",
    "price": 55000
}
```

Status Code: `201 Created`

### Test Case 2 – Missing Field

Request:

```json
{
    "price": 55000
}
```

Expected result:

`422 Unprocessable Content`

Reason: The required name field is missing.

### Test Case 3 – Invalid Price

Request:

```json
{
    "name": "Laptop",
    "price": -500
}
```

Expected result:

`422 Unprocessable Content`

Reason: Price must be greater than zero.

### Test Case 4 – Empty Name

Request:

```json
{
    "name": "",
    "price": 500
}
```

Expected result:

`422 Unprocessable Content`

Reason: Name must contain at least one character.

---

# Task D – Response Models and Error Handling

### 1. Response Model

The following response model is implemented:

```python
class ItemResponse(BaseModel):
    id: int
    name: str
    price: float
```

It ensures that the API returns data in a consistent format.

### 2. Handling 404 Errors

When a requested item does not exist, the API raises an HTTP exception.

```python
@app.get("/items/{item_id}")
def get_item(item_id: int):

    for item in items:
        if item["id"] == item_id:
            return item

    raise HTTPException(
        status_code=404,
        detail="Item not found"
    )
```

Example:

Request:

```http
GET /items/100
```

Expected response:

```json
{
    "detail": "Item not found"
}
```

Status Code: `404 Not Found`

### 3. Why These Status Codes?

| Status Code | Reason                               |
| ----------- | ------------------------------------ |
| 200         | Successfully retrieved data          |
| 201         | Successfully created a new item      |
| 400         | Invalid file type                    |
| 404         | Requested item does not exist        |
| 413         | Uploaded file exceeds the size limit |
| 422         | Request validation failed            |

---

# Task E – File Upload and Security

A file upload endpoint is implemented using FastAPI's `UploadFile`.

```python
@app.post("/upload")
async def upload_file(file: UploadFile = File(...)):

    content = await file.read()

    return {
        "filename": file.filename,
        "content_type": file.content_type,
        "size": len(content)
    }
```

### Security Measures

Before deploying a file upload API to production, the following security measures should be implemented:

1. **File Size Restriction:** Prevent users from uploading excessively large files.

2. **File Type Validation:** Allow only supported file formats.

3. **Unique Filenames:** Generate server-side filenames instead of trusting uploaded filenames.

4. **Content Validation:** Validate actual file content rather than relying only on the MIME type provided by the client.

5. **Secure Storage:** Store uploaded files in a secure location and prevent unauthorized access.

6. **Error Handling:** Return appropriate HTTP status codes for invalid uploads.

---

# Task F – Project Reflection

### 1. Project Setup

Created a FastAPI application using Python and installed the required dependencies.

### 2. API Endpoints Implemented

| Endpoint           | Method | Functionality              |
| ------------------ | ------ | -------------------------- |
| `/`                | GET    | Check API status           |
| `/items`           | GET    | Retrieve all items         |
| `/items/{item_id}` | GET    | Retrieve item by ID        |
| `/items`           | POST   | Create a new item          |
| `/upload`          | POST   | Upload and validate a file |

### 3. Testing

The API can be tested using FastAPI Swagger UI.

Test cases include:

* Creating an item with valid data.
* Retrieving all items.
* Retrieving an item using its ID.
* Requesting a non-existing item.
* Submitting an empty name.
* Submitting a negative price.
* Uploading an unsupported file.
* Uploading a file larger than the allowed limit.

### 4. What I Learned

Through this assignment, I learned how HTTP methods work and how clients communicate with backend servers.

I also gained practical experience in:

* Building REST APIs using FastAPI.
* Handling path and query parameters.
* Validating request data using Pydantic.
* Defining response models.
* Handling exceptions using HTTPException.
* Implementing file upload endpoints.
* Understanding HTTP status codes and API security.

One important takeaway was understanding how FastAPI automatically validates incoming data and generates interactive API documentation.

---

# Conclusion

This assignment helped me understand the fundamentals of HTTP communication and practical API development using FastAPI.

I implemented basic CRUD-related API operations, request validation, response models, error handling, and file upload functionality.

**Technologies Used:**

* Python
* FastAPI
* Pydantic v2
* Uvicorn
* REST APIs
* HTTP

---

**Placement Preparation Series | Day 3**

*HTTP & FastAPI Basics*
