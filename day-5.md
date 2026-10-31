# Day 5 – Advanced Backend Engineering

### Python Backend Placement Preparation Series

**Tech Stack:** Python · FastAPI · SQLAlchemy · Pydantic · Pytest · WebSockets · Celery · Redis

---

## 📌 Overview

This assignment focuses on building a maintainable backend application using FastAPI while applying concepts commonly used in real-world backend development.

The main objective is to develop a **Notes API** that supports CRUD operations, searching, pagination, API-key authentication, rate limiting, WebSocket communication, background processing, and automated testing.

Rather than treating each concept separately, this project combines them into a single backend application.

## 🎯 Learning Objectives

* Build REST APIs using FastAPI.
* Implement CRUD operations with proper request and response models.
* Add search, filtering, pagination, and sorting.
* Implement API-key authentication using dependencies.
* Understand and implement rate limiting.
* Establish real-time communication using WebSockets.
* Understand asynchronous background processing with Celery and Redis.
* Write automated tests using Pytest and FastAPI TestClient.
* Follow a modular backend architecture.
* Handle errors, validation, and security concerns.

---

# 1. Project Structure

The application follows a modular structure to separate API routes, business logic, database operations, authentication, and background processing.

```text
day-5/
│
├── app/
│   ├── __init__.py
│   ├── main.py
│   │
│   ├── core/
│   │   ├── config.py
│   │   └── security.py
│   │
│   ├── database/
│   │   ├── connection.py
│   │   └── models.py
│   │
│   ├── schemas/
│   │   └── note.py
│   │
│   ├── routers/
│   │   ├── notes.py
│   │   └── websocket.py
│   │
│   ├── services/
│   │   └── note_service.py
│   │
│   └── workers/
│       ├── celery_app.py
│       └── tasks.py
│
├── tests/
│   ├── conftest.py
│   ├── test_notes.py
│   ├── test_security.py
│   └── test_websocket.py
│
├── .env.example
├── .gitignore
├── requirements.txt
└── README.md
```

### Why this structure?

| Component   | Responsibility                  |
| ----------- | ------------------------------- |
| `main.py`   | Application entry point         |
| `core/`     | Configuration and security      |
| `database/` | Database connection and models  |
| `schemas/`  | Request and response validation |
| `routers/`  | API endpoint definitions        |
| `services/` | Business logic                  |
| `workers/`  | Background task processing      |
| `tests/`    | Automated testing               |

Separating these responsibilities makes the application easier to maintain, test, and extend.

---

# 2. Installation and Setup

### Step 1: Create a virtual environment

```bash
python -m venv venv
```

Activate it:

**Windows**

```bash
venv\Scripts\activate
```

**Linux / macOS**

```bash
source venv/bin/activate
```

### Step 2: Install dependencies

Create `requirements.txt`:

```txt
fastapi
uvicorn[standard]
sqlalchemy
pydantic
pydantic-settings
python-dotenv
pytest
httpx
websockets
celery
redis
```

Install:

```bash
pip install -r requirements.txt
```

### Step 3: Environment variables

Create `.env`:

```env
DATABASE_URL=sqlite:///./notes.db
API_KEY=change-this-development-key
REDIS_URL=redis://localhost:6379/0
```

Never commit real credentials or production secrets to GitHub.

### Step 4: Run the application

```bash
uvicorn app.main:app --reload
```

Open the interactive API documentation:

```text
http://127.0.0.1:8000/docs
```

---

# 3. Database Configuration

### `app/database/connection.py`

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, declarative_base
from app.core.config import settings

connect_args = {}

if settings.DATABASE_URL.startswith("sqlite"):
    connect_args = {"check_same_thread": False}

engine = create_engine(
    settings.DATABASE_URL,
    connect_args=connect_args
)

SessionLocal = sessionmaker(
    autocommit=False,
    autoflush=False,
    bind=engine
)

Base = declarative_base()


def get_db():
    db = SessionLocal()

    try:
        yield db
    finally:
        db.close()
```

### `app/core/config.py`

```python
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    DATABASE_URL: str = "sqlite:///./notes.db"
    API_KEY: str
    REDIS_URL: str = "redis://localhost:6379/0"

    model_config = SettingsConfigDict(
        env_file=".env",
        extra="ignore"
    )


settings = Settings()
```

### Explanation

* SQLAlchemy provides the ORM layer.
* `create_engine()` establishes the database engine.
* `SessionLocal` creates database sessions.
* `get_db()` provides a session to API endpoints.
* The `finally` block ensures that database sessions are closed.

SQLite is used here to simplify local development. PostgreSQL can be configured through the database URL for deployment.

---

# 4. Database Model

### `app/database/models.py`

```python
from sqlalchemy import Column, Integer, String, Text
from app.database.connection import Base


class Note(Base):
    __tablename__ = "notes"

    id = Column(Integer, primary_key=True, index=True)

    title = Column(
        String(120),
        nullable=False
    )

    body = Column(
        Text,
        default=""
    )

    status = Column(
        String(20),
        default="draft",
        nullable=False
    )
```

The Note model represents the database table.

Each note contains:

* `id`: Unique identifier.
* `title`: Note title.
* `body`: Note content.
* `status`: Current note status.

---

# 5. Pydantic Request and Response Models

### `app/schemas/note.py`

```python
from pydantic import BaseModel, Field, ConfigDict
from typing import Literal


class NoteCreate(BaseModel):
    title: str = Field(
        min_length=1,
        max_length=120
    )

    body: str = Field(
        default="",
        max_length=10000
    )

    status: Literal["draft", "published"] = "draft"


class NoteUpdate(BaseModel):
    title: str | None = Field(
        default=None,
        min_length=1,
        max_length=120
    )

    body: str | None = Field(
        default=None,
        max_length=10000
    )

    status: Literal["draft", "published"] | None = None


class NoteOut(BaseModel):
    id: int
    title: str
    body: str
    status: str

    model_config = ConfigDict(from_attributes=True)
```

### Why use Pydantic?

Pydantic validates incoming data before it reaches the business logic.

For example:

* Empty titles are rejected.
* Titles longer than 120 characters are rejected.
* Invalid status values are rejected.
* Response models provide a consistent API contract.

---

# 6. API-Key Authentication

### `app/core/security.py`

```python
import secrets

from fastapi import Security, HTTPException, status
from fastapi.security import APIKeyHeader

from app.core.config import settings


api_key_header = APIKeyHeader(
    name="X-API-Key",
    auto_error=False
)


def verify_api_key(
    api_key: str | None = Security(api_key_header)
):
    if not api_key or not secrets.compare_digest(
        api_key,
        settings.API_KEY
    ):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid or missing API key"
        )

    return api_key
```

### How it works

1. The client sends an API key in the `X-API-Key` header.
2. FastAPI extracts the key using `APIKeyHeader`.
3. The dependency verifies the supplied key.
4. Invalid or missing credentials result in HTTP 401.
5. Valid credentials allow the request to continue.

Example request header:

```http
X-API-Key: your-development-api-key
```

**Security considerations:**

For production, use high-entropy API keys, store securely hashed credentials where practical, support key rotation and revocation, and associate keys with users, scopes, and quotas.

The single-key implementation above is a development example, not a complete production credential-management system.

---

# 7. Notes CRUD API

### `app/services/note_service.py`

```python
from sqlalchemy.orm import Session
from sqlalchemy import or_

from app.database.models import Note
from app.schemas.note import NoteCreate, NoteUpdate


def create_note(db: Session, data: NoteCreate):
    note = Note(**data.model_dump())

    db.add(note)
    db.commit()
    db.refresh(note)

    return note


def get_note(db: Session, note_id: int):
    return db.query(Note).filter(
        Note.id == note_id
    ).first()


def update_note(
    db: Session,
    note: Note,
    data: NoteUpdate
):
    update_data = data.model_dump(exclude_unset=True)

    for field, value in update_data.items():
        setattr(note, field, value)

    db.commit()
    db.refresh(note)

    return note


def delete_note(db: Session, note: Note):
    db.delete(note)
    db.commit()


def search_notes(
    db: Session,
    search: str | None,
    status: str | None,
    limit: int,
    offset: int,
    sort_by: str,
    order: str
):
    query = db.query(Note)

    if search:
        pattern = f"%{search}%"

        query = query.filter(
            or_(
                Note.title.ilike(pattern),
                Note.body.ilike(pattern)
            )
        )

    if status:
        query = query.filter(Note.status == status)

    allowed_fields = {
        "id": Note.id,
        "title": Note.title,
        "status": Note.status
    }

    sort_column = allowed_fields[sort_by]

    if order == "desc":
        query = query.order_by(sort_column.desc())
    else:
        query = query.order_by(sort_column.asc())

    return query.offset(offset).limit(limit).all()
```

### Important implementation points

* Business logic is separated from the route handlers.
* SQLAlchemy query filters parameterise values.
* Sorting uses an allowlist rather than directly accepting arbitrary SQL identifiers.
* Pagination is applied through `offset()` and `limit()`.
* Search checks both title and body.

---

# 8. Notes Router

### `app/routers/notes.py`

```python
from fastapi import (
    APIRouter,
    Depends,
    HTTPException,
    Query,
    status
)

from sqlalchemy.orm import Session

from app.database.connection import get_db
from app.database.models import Note
from app.schemas.note import NoteCreate, NoteUpdate, NoteOut
from app.services import note_service
from app.core.security import verify_api_key


router = APIRouter(
    prefix="/v1/notes",
    tags=["Notes"],
    dependencies=[Depends(verify_api_key)]
)


@router.post(
    "",
    response_model=NoteOut,
    status_code=status.HTTP_201_CREATED
)
def create_note(
    data: NoteCreate,
    db: Session = Depends(get_db)
):
    return note_service.create_note(db, data)


@router.get("", response_model=list[NoteOut])
def list_notes(
    search: str | None = None,
    note_status: str | None = Query(
        default=None,
        alias="status"
    ),
    limit: int = Query(default=20, ge=1, le=100),
    offset: int = Query(default=0, ge=0),
    sort_by: str = Query(
        default="id",
        pattern="^(id|title|status)$"
    ),
    order: str = Query(
        default="asc",
        pattern="^(asc|desc)$"
    ),
    db: Session = Depends(get_db)
):
    return note_service.search_notes(
        db=db,
        search=search,
        status=note_status,
        limit=limit,
        offset=offset,
        sort_by=sort_by,
        order=order
    )


@router.get("/{note_id}", response_model=NoteOut)
def get_note(
    note_id: int,
    db: Session = Depends(get_db)
):
    note = note_service.get_note(db, note_id)

    if not note:
        raise HTTPException(
            status_code=404,
            detail="Note not found"
        )

    return note


@router.patch("/{note_id}", response_model=NoteOut)
def update_note(
    note_id: int,
    data: NoteUpdate,
    db: Session = Depends(get_db)
):
    note = note_service.get_note(db, note_id)

    if not note:
        raise HTTPException(
            status_code=404,
            detail="Note not found"
        )

    return note_service.update_note(db, note, data)


@router.delete(
    "/{note_id}",
    status_code=status.HTTP_204_NO_CONTENT
)
def delete_note(
    note_id: int,
    db: Session = Depends(get_db)
):
    note = note_service.get_note(db, note_id)

    if not note:
        raise HTTPException(
            status_code=404,
            detail="Note not found"
        )

    note_service.delete_note(db, note)
```

### API endpoint summary

| Method | Endpoint         | Purpose               | Expected status |
| ------ | ---------------- | --------------------- | --------------- |
| POST   | `/v1/notes`      | Create a note         | 201             |
| GET    | `/v1/notes`      | List and search notes | 200             |
| GET    | `/v1/notes/{id}` | Get a note            | 200             |
| PATCH  | `/v1/notes/{id}` | Update a note         | 200             |
| DELETE | `/v1/notes/{id}` | Delete a note         | 204             |

All these endpoints require a valid API key.

---

# 9. Search, Filtering, Pagination and Sorting

The list endpoint supports four important features.

### Search

```http
GET /v1/notes?search=python
```

Searches for matching text in the note title and body.

### Filtering

```http
GET /v1/notes?status=published
```

Returns only published notes.

### Pagination

```http
GET /v1/notes?limit=10&offset=0
```

* `limit`: Number of records returned.
* `offset`: Number of records skipped.

### Sorting

```http
GET /v1/notes?sort_by=title&order=asc
```

Sorts notes by title in ascending order.

### Why validate pagination?

Without validation, a client could send:

```http
GET /v1/notes?limit=-100&offset=-50
```

Using `ge=1`, `le=100`, and `ge=0` prevents invalid pagination values.

### Offset vs Cursor Pagination

| Offset Pagination                  | Cursor Pagination                       |
| ---------------------------------- | --------------------------------------- |
| Uses limit and offset              | Uses a cursor                           |
| Easy to implement                  | More complex                            |
| Deep offsets can be expensive      | Better for large feeds                  |
| Records can shift between requests | More stable with deterministic ordering |

For large and frequently changing datasets, cursor pagination using an indexed ordering such as `(created_at, id)` is often a better choice.

---

# 10. Rate Limiting

Rate limiting controls how frequently a client can access an API.

For example, a service may allow a maximum of 10 requests per minute for a particular client.

### Basic in-memory implementation

The following is a simple learning implementation. It is suitable only for a single-process demonstration.

```python
import time

from collections import defaultdict
from fastapi import HTTPException, Request


REQUEST_LIMIT = 10
TIME_WINDOW = 60

request_records = defaultdict(list)


async def rate_limit(request: Request):
    client_ip = request.client.host if request.client else "unknown"

    current_time = time.time()

    request_records[client_ip] = [
        timestamp
        for timestamp in request_records[client_ip]
        if current_time - timestamp < TIME_WINDOW
    ]

    if len(request_records[client_ip]) >= REQUEST_LIMIT:
        raise HTTPException(
            status_code=429,
            detail="Too many requests. Try again later."
        )

    request_records[client_ip].append(current_time)
```

To apply it to the notes router, add the dependency:

```python
dependencies=[
    Depends(verify_api_key),
    Depends(rate_limit)
]
```

### Rate-limiting strategies

| Strategy       | Description                             |
| -------------- | --------------------------------------- |
| Fixed Window   | Counts requests in fixed time intervals |
| Sliding Window | Counts requests over a rolling interval |
| Token Bucket   | Allows controlled bursts using tokens   |
| Leaky Bucket   | Smooths incoming request rates          |

### Production considerations

An in-memory dictionary is not reliable for multiple workers because every worker maintains separate state.

A production implementation should use shared storage such as Redis with atomic operations.

A rate-limited request should return:

```http
HTTP/1.1 429 Too Many Requests
```

A production API can also provide a `Retry-After` header.

---

# 11. WebSockets – Real-Time Communication

WebSockets provide two-way communication between a client and server over a persistent connection.

Unlike normal HTTP requests, the connection remains open until it is closed.

### HTTP vs WebSockets

| HTTP                                  | WebSockets                         |
| ------------------------------------- | ---------------------------------- |
| Request-response communication        | Bidirectional communication        |
| Connection generally used per request | Persistent connection              |
| Client typically initiates requests   | Either side can send messages      |
| Suitable for CRUD APIs                | Suitable for live updates and chat |

### `app/routers/websocket.py`

```python
from fastapi import APIRouter, WebSocket, WebSocketDisconnect


router = APIRouter(tags=["WebSockets"])


class ConnectionManager:

    def __init__(self):
        self.rooms = {}

    async def connect(self, room: str, websocket: WebSocket):
        await websocket.accept()

        if room not in self.rooms:
            self.rooms[room] = set()

        self.rooms[room].add(websocket)

    def disconnect(self, room: str, websocket: WebSocket):
        if room in self.rooms:
            self.rooms[room].discard(websocket)

            if not self.rooms[room]:
                del self.rooms[room]

    async def broadcast(self, room: str, message: str):
        for connection in list(self.rooms.get(room, set())):
            try:
                await connection.send_text(message)
            except Exception:
                self.disconnect(room, connection)


manager = ConnectionManager()


@router.websocket("/ws/notes/{room}")
async def notes_websocket(
    websocket: WebSocket,
    room: str
):
    await manager.connect(room, websocket)

    try:
        while True:
            message = await websocket.receive_text()

            await manager.broadcast(
                room,
                message
            )

    except WebSocketDisconnect:
        manager.disconnect(room, websocket)
```

### How it works

1. The client connects to a WebSocket room.
2. The server accepts the connection.
3. The connection is added to the room.
4. Incoming messages are broadcast to the room.
5. When a client disconnects, its connection is removed.

### Example connection

```javascript
const socket = new WebSocket(
    "ws://127.0.0.1:8000/ws/notes/general"
);

socket.onopen = () => {
    console.log("Connected");
    socket.send("New note created");
};

socket.onmessage = (event) => {
    console.log("Received:", event.data);
};

socket.onclose = () => {
    console.log("Disconnected");
};
```

### Important security and scaling considerations

* Authenticate WebSocket connections.
* Authorise access to each room.
* Validate message formats and sizes.
* Prevent unauthorised room broadcasting.
* Clean up disconnected connections.
* Use Redis Pub/Sub or another shared messaging system when multiple application workers need to communicate.

The in-memory connection manager above is a single-process demonstration, not a distributed WebSocket solution.

---

# 12. Background Processing with Celery and Redis

Some operations take too much time to execute inside an HTTP request.

Examples include:

* Sending emails.
* Generating reports.
* Processing uploaded files.
* Creating note summaries.
* Scheduled notifications.

Instead of making the client wait, we can move the work to a background worker.

### Architecture

```text
Client
  |
  v
FastAPI Application
  |
  v
Celery Task Queue
  |
  v
Redis Broker
  |
  v
Celery Worker
  |
  v
Task Processing
```

### `app/workers/celery_app.py`

```python
from celery import Celery

from app.core.config import settings


celery_app = Celery(
    "notes_worker",
    broker=settings.REDIS_URL,
    backend=settings.REDIS_URL
)
```

### `app/workers/tasks.py`

```python
from app.workers.celery_app import celery_app


@celery_app.task(
    bind=True,
    max_retries=3
)
def generate_note_summary(self, note_id: int):

    try:
        # Replace this demonstration with actual
        # database retrieval and summary generation.

        return {
            "note_id": note_id,
            "status": "completed",
            "summary": "Summary generation completed."
        }

    except Exception as exc:
        raise self.retry(
            exc=exc,
            countdown=5
        )
```

### Why use Celery?

Celery allows long-running work to execute independently of the API request.

The API can return immediately while the worker processes the task.

### Important concepts

| Concept        | Explanation                                                |
| -------------- | ---------------------------------------------------------- |
| Broker         | Transports task messages                                   |
| Worker         | Executes background tasks                                  |
| Result Backend | Stores task state or results                               |
| Celery Beat    | Schedules periodic tasks                                   |
| Retry          | Re-executes eligible failed tasks                          |
| Idempotency    | Prevents repeated execution from causing duplicate effects |

A task should be designed to handle retries safely. For example, sending an email or creating a payment should not produce duplicate business effects if a task executes more than once.

---

# 13. Task Submission and Status Tracking

A background task should not keep the HTTP request open until processing finishes.

Instead, return HTTP 202 Accepted with a task ID.

Example task submission endpoint:

```python
from fastapi import APIRouter, Depends, HTTPException

from celery.result import AsyncResult

from app.workers.tasks import generate_note_summary
from app.workers.celery_app import celery_app
from app.core.security import verify_api_key


router = APIRouter(
    prefix="/v1/tasks",
    tags=["Background Tasks"],
    dependencies=[Depends(verify_api_key)]
)


@router.post("/notes/{note_id}/summary", status_code=202)
def create_summary(note_id: int):

    task = generate_note_summary.delay(note_id)

    return {
        "task_id": task.id,
        "status": "queued"
    }


@router.get("/{task_id}")
def get_task_status(task_id: str):

    result = AsyncResult(
        task_id,
        app=celery_app
    )

    return {
        "task_id": task_id,
        "state": result.state,
        "ready": result.ready()
    }
```

**Production note:** Before returning task status or results, authenticate the caller and verify task ownership. Do not expose internal exception details or arbitrary task results.

### Starting Redis

If Redis is installed locally:

```bash
redis-server
```

### Starting the Celery worker

```bash
celery -A app.workers.celery_app.celery_app worker --loglevel=info
```

The exact worker command may need adjustment depending on the operating system and Celery installation.

---

# 14. Main Application

### `app/main.py`

```python
from fastapi import FastAPI

from app.database.connection import Base, engine
from app.routers import notes, websocket
from app.routers import tasks


Base.metadata.create_all(bind=engine)


app = FastAPI(
    title="Advanced Notes API",
    description="Notes API with authentication, search, pagination, WebSockets and background tasks",
    version="1.0.0"
)


app.include_router(notes.router)
app.include_router(websocket.router)
app.include_router(tasks.router)


@app.get("/")
def root():
    return {
        "message": "Advanced Notes API is running"
    }


@app.get("/health")
def health():
    return {
        "status": "healthy"
    }
```

The main application registers the routers and creates the database tables for this development example.

For production, use database migrations such as Alembic rather than relying on `create_all()`.

---

# 15. Automated Testing with Pytest

Testing is important because a successful API response does not necessarily mean that the application behaves correctly in every situation.

We need to test:

* Valid input.
* Invalid input.
* Missing records.
* Authentication failures.
* Pagination boundaries.
* Duplicate requests.
* WebSocket communication.
* Background task failures.

### `tests/conftest.py`

```python
import pytest

from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from sqlalchemy.pool import StaticPool

from app.main import app
from app.database.connection import Base, get_db
from app.core.security import verify_api_key


SQLALCHEMY_DATABASE_URL = "sqlite://"

engine = create_engine(
    SQLALCHEMY_DATABASE_URL,
    connect_args={"check_same_thread": False},
    poolclass=StaticPool
)

TestingSessionLocal = sessionmaker(
    autocommit=False,
    autoflush=False,
    bind=engine
)


def override_get_db():
    db = TestingSessionLocal()

    try:
        yield db
    finally:
        db.close()


def override_api_key():
    return "test-api-key"


@pytest.fixture
def client():

    Base.metadata.create_all(bind=engine)

    app.dependency_overrides[get_db] = override_get_db
    app.dependency_overrides[verify_api_key] = override_api_key

    with TestClient(app) as test_client:
        yield test_client

    app.dependency_overrides.clear()
    Base.metadata.drop_all(bind=engine)
```

### `tests/test_notes.py`

```python
def test_create_note(client):

    response = client.post(
        "/v1/notes",
        json={
            "title": "Python Backend",
            "body": "Learning FastAPI",
            "status": "draft"
        }
    )

    assert response.status_code == 201

    data = response.json()

    assert data["title"] == "Python Backend"
    assert data["status"] == "draft"


def test_get_note(client):

    create_response = client.post(
        "/v1/notes",
        json={
            "title": "Test Note",
            "body": "Testing GET endpoint"
        }
    )

    note_id = create_response.json()["id"]

    response = client.get(
        f"/v1/notes/{note_id}"
    )

    assert response.status_code == 200
    assert response.json()["id"] == note_id


def test_missing_note(client):

    response = client.get("/v1/notes/99999")

    assert response.status_code == 404


def test_invalid_title(client):

    response = client.post(
        "/v1/notes",
        json={
            "title": "",
            "body": "Invalid note"
        }
    )

    assert response.status_code == 422


def test_pagination(client):

    for number in range(5):
        client.post(
            "/v1/notes",
            json={
                "title": f"Note {number}",
                "body": "Pagination test"
            }
        )

    response = client.get(
        "/v1/notes?limit=2&offset=0"
    )

    assert response.status_code == 200
    assert len(response.json()) == 2


def test_search(client):

    client.post(
        "/v1/notes",
        json={
            "title": "Python Learning",
            "body": "FastAPI project"
        }
    )

    response = client.get(
        "/v1/notes?search=Python"
    )

    assert response.status_code == 200
    assert len(response.json()) >= 1


def test_delete_note(client):

    created = client.post(
        "/v1/notes",
        json={
            "title": "Delete Me",
            "body": "Testing delete"
        }
    )

    note_id = created.json()["id"]

    response = client.delete(
        f"/v1/notes/{note_id}"
    )

    assert response.status_code == 204
```

### `tests/test_security.py`

```python
from fastapi.testclient import TestClient

from app.main import app


def test_missing_api_key():

    client = TestClient(app)

    response = client.get("/v1/notes")

    assert response.status_code == 401
```

### `tests/test_websocket.py`

```python
def test_websocket_connection(client):

    with client.websocket_connect(
        "/ws/notes/general"
    ) as websocket:

        websocket.send_text("Hello Backend")

        response = websocket.receive_text()

        assert response == "Hello Backend"
```

### Run tests

```bash
pytest -q
```

### Run tests with coverage

Install:

```bash
pip install pytest-cov
```

Execute:

```bash
pytest --cov=app --cov-report=term-missing
```

Coverage identifies which code was executed. It does not guarantee that the application is correct or secure.

---

# 16. HTTP Status Codes

Understanding HTTP status codes is an important part of backend development.

| Status Code | Meaning               | Example                                |
| ----------- | --------------------- | -------------------------------------- |
| 200         | OK                    | Successful GET request                 |
| 201         | Created               | Note created                           |
| 202         | Accepted              | Background task queued                 |
| 204         | No Content            | Note deleted                           |
| 400         | Bad Request           | Invalid request under the API contract |
| 401         | Unauthorized          | Missing or invalid API key             |
| 403         | Forbidden             | Authenticated user lacks permission    |
| 404         | Not Found             | Note does not exist                    |
| 409         | Conflict              | Duplicate or conflicting resource      |
| 422         | Unprocessable Entity  | Validation failure                     |
| 429         | Too Many Requests     | Rate limit exceeded                    |
| 500         | Internal Server Error | Unexpected server error                |

### Important interview question

**What is the difference between 401 and 403?**

* 401 means the request does not have valid authentication credentials.
* 403 means the user is authenticated but does not have permission to perform the requested action.

---

# 17. Important Backend Concepts

### 1. What is API versioning?

API versioning allows developers to introduce breaking changes without immediately affecting existing clients.

Example:

```text
/v1/notes
/v2/notes
```

### 2. What is dependency injection in FastAPI?

Dependency injection is a mechanism for providing reusable resources or logic to endpoints.

Examples:

* Database sessions.
* API-key verification.
* Current-user information.
* Permission checks.

### 3. What is a race condition?

A race condition occurs when multiple operations execute concurrently and the final result depends on their timing or execution order.

### 4. What is idempotency?

An operation is idempotent when repeating it produces the same intended result as performing it once.

This is especially important for background jobs and client retries.

### 5. Why use Redis?

Redis provides fast shared data storage and can be used for:

* Celery message brokering.
* Rate-limit counters.
* Caching.
* Pub/Sub communication.

### 6. Why use a service layer?

A service layer separates business logic from HTTP route handling.

It makes the application easier to test, reuse, and maintain.

### 7. Why use WebSockets?

WebSockets are useful when the server needs to send real-time updates without waiting for a new HTTP request.

### 8. What is the difference between synchronous and asynchronous execution?

Synchronous execution waits for an operation to complete before continuing.

Asynchronous execution allows an application to perform other work while waiting for supported non-blocking operations.

### 9. Why use Celery?

Celery executes time-consuming or scheduled tasks outside the normal HTTP request-response cycle.

### 10. What is a database connection pool?

A connection pool manages reusable database connections instead of creating a new connection for every request.

---

# 18. Assignment Test Checklist

The following checklist covers the practical requirements of this assignment.

| No. | Requirement            | Test Scenario                             |
| --- | ---------------------- | ----------------------------------------- |
| 1   | Create note            | Valid title and body                      |
| 2   | Read note              | Existing note ID                          |
| 3   | Update note            | Update only the title                     |
| 4   | Delete note            | Delete an existing note                   |
| 5   | Search                 | Search by title or body                   |
| 6   | Filter                 | Filter by draft/published status          |
| 7   | Pagination             | Test limit and offset                     |
| 8   | Sorting                | Sort by title or status                   |
| 9   | API-key authentication | Missing, invalid, and valid keys          |
| 10  | Rate limiting          | Send requests beyond the configured limit |
| 11  | WebSockets             | Connect, send, broadcast, disconnect      |
| 12  | Background tasks       | Enqueue and check task status             |
| 13  | Error handling         | Missing note and invalid input            |
| 14  | Automated tests        | Run Pytest                                |

### Additional production-level test cases

* Repeated client requests should not create unintended duplicate notes.
* Rate limits should remain consistent across multiple workers.
* Disconnected WebSocket clients should be removed.
* WebSocket clients must not receive messages from unauthorised rooms.
* External services should be mocked in isolated unit tests.
* Background task retries should not duplicate business effects.
* Database and Redis failures should be handled safely.

---

# 19. Common Backend Failure Scenarios

| Problem                                | Possible Solution                                                                |
| -------------------------------------- | -------------------------------------------------------------------------------- |
| Duplicate notes after client retry     | Implement idempotency keys and database uniqueness constraints where appropriate |
| Different rate limits across workers   | Use shared atomic Redis state                                                    |
| Disconnected WebSockets remain tracked | Handle disconnect events and remove connections                                  |
| Tests call real email services         | Mock external dependencies                                                       |
| Background task fails temporarily      | Implement bounded retries and backoff                                            |
| Redis broker unavailable               | Return an appropriate error instead of claiming the task was queued              |
| User accesses another user's task      | Verify task ownership                                                            |
| Deep pagination becomes slow           | Consider cursor pagination and indexing                                          |

---

# 20. How to Run the Project

### Start the FastAPI application

```bash
uvicorn app.main:app --reload
```

### Start Redis

```bash
redis-server
```

### Start the Celery worker

```bash
celery -A app.workers.celery_app.celery_app worker --loglevel=info
```

### Run automated tests

```bash
pytest -q
```

### Check API documentation

```text
http://127.0.0.1:8000/docs
```

### Example API request

Create a note:

```bash
curl -X POST "http://127.0.0.1:8000/v1/notes" \
-H "Content-Type: application/json" \
-H "X-API-Key: your-development-api-key" \
-d "{\"title\":\"Backend Engineering\",\"body\":\"Learning FastAPI\",\"status\":\"draft\"}"
```

List notes:

```bash
curl "http://127.0.0.1:8000/v1/notes?limit=10&offset=0" \
-H "X-API-Key: your-development-api-key"
```

---

# 21. Reflection

### What did I learn?

Through this assignment, I explored how different backend engineering concepts work together in a single application.

I learned how to:

* Design REST APIs using FastAPI.
* Separate API routes and business logic.
* Validate request and response data using Pydantic.
* Perform database operations using SQLAlchemy.
* Implement API-key authentication.
* Apply searching, filtering, pagination, and sorting.
* Understand rate-limiting strategies.
* Establish real-time communication using WebSockets.
* Understand asynchronous task processing using Celery and Redis.
* Write automated tests using Pytest and TestClient.

### What was challenging?

Integrating multiple backend concepts into one application was the most challenging part.

Understanding the difference between normal HTTP communication, WebSocket connections, and background task execution helped me understand how backend systems handle different types of workloads.

### What would I improve next?

I would extend this application with:

* PostgreSQL integration.
* Alembic database migrations.
* User-based authentication and authorisation.
* Redis-based distributed rate limiting.
* Distributed WebSocket communication.
* Improved task monitoring and retry handling.
* Docker-based deployment.
* Structured logging and monitoring.

---

# 22. Final Conclusion

This assignment helped me understand that backend development involves much more than creating API endpoints.

A reliable backend application also needs proper validation, authentication, database management, error handling, testing, background processing, and communication between different components.

By combining these concepts into a Notes API, I gained a better understanding of how real-world backend applications are structured and maintained.

**My approach: Understand → Design → Implement → Test → Debug → Improve.**

---
