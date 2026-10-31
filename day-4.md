# Day 4 – Async, Middleware & Modular FastAPI

**Placement Preparation Series | Backend Python**

## Overview

This assignment focuses on asynchronous programming in Python and building scalable, modular APIs using FastAPI.

The objective is to understand how asynchronous operations work, execute independent tasks concurrently, implement middleware, manage dependencies, organize API routes, and handle application startup and shutdown.

### Topics Covered

* Synchronous vs Asynchronous Programming
* Coroutines and Event Loop
* async and await
* asyncio.gather()
* Blocking vs Non-Blocking Operations
* Async FastAPI Endpoints
* BackgroundTasks
* Application Lifespan
* Custom Middleware
* CORS
* Dependency Injection
* Sub-dependencies
* APIRouter
* Modular Project Architecture

---

# Task A – Async Programming: Sequential vs Concurrent Execution

## Problem Statement

Write three coroutines that simulate independent I/O operations.

Compare sequential execution with concurrent execution using `asyncio.gather()` and explain the difference in execution time.

## Approach

I created three asynchronous functions representing independent operations:

* Fetching user profile.
* Fetching notifications.
* Fetching recommendations.

Each operation takes approximately one second.

First, I execute them sequentially using `await`.

Then, I use `asyncio.gather()` to execute all three operations concurrently.

## Python Code

Create a file named `async_demo.py`.

```python
import asyncio
import time


async def fetch_data(name, delay):

    print(f"Starting {name}")

    await asyncio.sleep(delay)

    print(f"Completed {name}")

    return f"{name} data received"


# Sequential execution
async def sequential_execution():

    start = time.perf_counter()

    profile = await fetch_data("Profile", 1)
    notifications = await fetch_data("Notifications", 1)
    recommendations = await fetch_data("Recommendations", 1)

    results = [profile, notifications, recommendations]

    end = time.perf_counter()

    print("\nSequential Results:")
    print(results)

    print("Execution Time:", round(end - start, 2), "seconds")


# Concurrent execution
async def concurrent_execution():

    start = time.perf_counter()

    results = await asyncio.gather(
        fetch_data("Profile", 1),
        fetch_data("Notifications", 1),
        fetch_data("Recommendations", 1)
    )

    end = time.perf_counter()

    print("\nConcurrent Results:")
    print(results)

    print("Execution Time:", round(end - start, 2), "seconds")


async def main():

    print("Sequential Execution")
    await sequential_execution()

    print("\n-------------------------\n")

    print("Concurrent Execution")
    await concurrent_execution()


asyncio.run(main())
```

## Expected Output

The exact order of concurrent start/completion messages may vary.

```text
Sequential Execution

Starting Profile
Completed Profile
Starting Notifications
Completed Notifications
Starting Recommendations
Completed Recommendations

Sequential Results:
['Profile data received',
 'Notifications data received',
 'Recommendations data received']

Execution Time: approximately 3 seconds

-------------------------

Concurrent Execution

Starting Profile
Starting Notifications
Starting Recommendations

Completed Profile
Completed Notifications
Completed Recommendations

Concurrent Results:
['Profile data received',
 'Notifications data received',
 'Recommendations data received']

Execution Time: approximately 1 second
```

## Explanation

In sequential execution, each operation waits for the previous operation to finish.

Therefore:

1 + 1 + 1 = approximately 3 seconds.

In concurrent execution, `asyncio.gather()` allows independent coroutines to overlap while waiting for I/O.

Therefore, the total execution time is approximately the duration of the slowest operation.

## Complexity / Performance

| Execution  | Approximate Time |
| ---------- | ---------------: |
| Sequential |        3 seconds |
| Concurrent |         1 second |

**Conclusion:** Asynchronous programming improves efficiency when multiple independent I/O-bound operations need to execute without waiting for one another.

---

# Task B – Async FastAPI Endpoint

## Problem Statement

Build an asynchronous FastAPI endpoint using an async-compatible operation or a safe demonstration coroutine.

Explain why blocking operations should be avoided.

## Approach

I created an asynchronous endpoint using `async def`.

For demonstration, I used `asyncio.sleep()` to simulate an I/O wait.

Unlike `time.sleep()`, `asyncio.sleep()` does not block the event loop.

## Python Code

```python
import asyncio

from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def home():

    return {
        "message": "FastAPI application is running"
    }


@app.get("/async-status")
async def async_status():

    await asyncio.sleep(2)

    return {
        "status": "completed",
        "message": "Async operation executed successfully"
    }
```

## Test the Endpoint

Start the application:

```bash
uvicorn app.main:app --reload
```

Request:

```http
GET /async-status
```

Expected Response:

```json
{
    "status": "completed",
    "message": "Async operation executed successfully"
}
```

## Blocking vs Non-Blocking

### Incorrect Approach

```python
import time

async def handler():

    time.sleep(2)

    return {"message": "Completed"}
```

Problem:

`time.sleep()` blocks the current thread, preventing the event loop from scheduling other tasks on that thread.

### Correct Approach

```python
import asyncio

async def handler():

    await asyncio.sleep(2)

    return {"message": "Completed"}
```

Here, `await` suspends the current coroutine while allowing other asynchronous tasks to run.

### Comparison

| Operation          | Behavior                                 |
| ------------------ | ---------------------------------------- |
| `time.sleep()`     | Blocking                                 |
| `asyncio.sleep()`  | Non-blocking                             |
| `async def`        | Defines a coroutine function             |
| `await`            | Suspends the current coroutine           |
| `asyncio.gather()` | Runs independent awaitables concurrently |

**Conclusion:** Using asynchronous operations helps FastAPI handle concurrent I/O efficiently. Simply adding `async` does not make blocking code non-blocking.

---

# Task C – Custom Middleware

## Problem Statement

Implement timing middleware in FastAPI and add an `X-Process-Time` header to every response.

Test the middleware and explain its importance.

## What is Middleware?

Middleware is a layer that executes during the request and response lifecycle.

It allows us to perform common operations before and after a request reaches the route handler.

Common use cases:

* Request logging.
* Execution time measurement.
* Request ID generation.
* Security-related processing.
* CORS configuration.

## Implementation

```python
from time import perf_counter
from uuid import uuid4

from fastapi import FastAPI, Request

app = FastAPI()


@app.middleware("http")
async def timing_middleware(request: Request, call_next):

    start_time = perf_counter()

    request_id = str(uuid4())

    response = await call_next(request)

    process_time = perf_counter() - start_time

    response.headers["X-Process-Time"] = (
        f"{process_time:.6f}"
    )

    response.headers["X-Request-ID"] = request_id

    return response


@app.get("/")
def home():

    return {
        "message": "Middleware is working"
    }


@app.get("/users")
def get_users():

    return {
        "users": ["Anil", "Rahul", "Kiran"]
    }
```

## Testing Middleware

Start the server:

```bash
uvicorn main:app --reload
```

Send a GET request:

```http
GET http://127.0.0.1:8000/users
```

Expected Response:

```json
{
    "users": [
        "Anil",
        "Rahul",
        "Kiran"
    ]
}
```

Response Headers will include:

```text
X-Process-Time: 0.001234
X-Request-ID: generated-unique-id
```

The execution time will vary depending on the system.

## Why is Middleware Useful?

1. Measures API execution time.
2. Helps identify slow endpoints.
3. Generates request IDs for tracking requests.
4. Supports centralized request processing.
5. Reduces repeated code across endpoints.

**Security Note:** Never log passwords, authorization tokens, or session cookies.

**Conclusion:** Middleware provides a centralized way to handle cross-cutting concerns across multiple API endpoints.

---

# Task D – Dependency Injection and Sub-dependencies

## Problem Statement

Create a reusable pagination or current-user dependency, add a sub-dependency, and explain how authentication should be verified in production.

## What is Dependency Injection?

Dependency Injection is a mechanism where FastAPI automatically calls a required dependency and provides its result to a route function.

FastAPI uses `Depends()` for dependency injection.

Common use cases:

* Authentication.
* Authorization.
* Pagination.
* Database connections.
* Reusable validation.
* Shared application configuration.

## 1. Pagination Dependency

Create a file named `dependencies.py`.

```python
from typing import Annotated

from fastapi import Depends


def common_pagination(
    skip: int = 0,
    limit: int = 10
):

    return {
        "skip": max(skip, 0),
        "limit": min(max(limit, 1), 100)
    }


Pagination = Annotated[
    dict,
    Depends(common_pagination)
]
```

Use this dependency in an endpoint:

```python
from fastapi import FastAPI

from dependencies import Pagination

app = FastAPI()


@app.get("/products")
def get_products(paging: Pagination):

    return {
        "skip": paging["skip"],
        "limit": paging["limit"]
    }
```

Example Request:

```text
GET /products?skip=5&limit=20
```

Expected Response:

```json
{
    "skip": 5,
    "limit": 20
}
```

## 2. Sub-dependency and Authorization

A sub-dependency is a dependency that itself depends on another dependency.

Example:

```python
from fastapi import (
    FastAPI,
    Depends,
    HTTPException,
    Header
)

app = FastAPI()


# First dependency
def get_current_user(
    authorization: str | None = Header(default=None)
):

    if authorization != "Bearer demo-token":
        raise HTTPException(
            status_code=401,
            detail="Invalid or missing authentication"
        )

    return {
        "id": 101,
        "role": "admin"
    }


# Sub-dependency
def require_admin(
    user=Depends(get_current_user)
):

    if user["role"] != "admin":

        raise HTTPException(
            status_code=403,
            detail="Admin access required"
        )

    return user


# Protected endpoint
@app.get("/admin")
def admin_dashboard(
    user=Depends(require_admin)
):

    return {
        "message": "Welcome to admin dashboard",
        "user_id": user["id"]
    }
```

### Test Request

```http
GET /admin
Authorization: Bearer demo-token
```

Expected Response:

```json
{
    "message": "Welcome to admin dashboard",
    "user_id": 101
}
```

Without the token:

```json
{
    "detail": "Invalid or missing authentication"
}
```

Status Code: `401 Unauthorized`

### Dependency Flow

```text
Client Request
      |
      v
   /admin
      |
      v
 require_admin()
      |
      v
 get_current_user()
      |
      v
 Verify Credentials
      |
      v
 Check User Role
      |
      v
 Return Response
```

### Production Authentication Considerations

The above authentication example is only for demonstrating dependency injection.

In production:

* Verify JWT signatures and expiration.
* Use a trusted authentication mechanism such as OAuth2/JWT.
* Retrieve user roles from trusted server-side data.
* Never trust roles supplied directly by the client.
* Store secrets securely.
* Return 401 for invalid authentication and 403 for insufficient permissions.

**Conclusion:** Dependency injection helps create reusable, maintainable, and testable API logic.

---

# Task E – Modular Application Using APIRouter

## Problem Statement

Split API endpoints into at least two router modules and register them using prefixes and tags.

Verify the endpoints through FastAPI documentation.

## What is APIRouter?

`APIRouter` is used to organize related API endpoints into separate modules.

Benefits:

* Cleaner project structure.
* Better code organization.
* Reusable route modules.
* Easier maintenance.
* Better API documentation.

## Project Structure

```text
Day-4/
│
├── app/
│   │
│   ├── __init__.py
│   ├── main.py
│   ├── dependencies.py
│   │
│   └── routers/
│       ├── __init__.py
│       ├── users.py
│       └── products.py
│
├── async_demo.py
├── requirements.txt
└── README.md
```

## 1. Create users.py

Location: `app/routers/users.py`

```python
from fastapi import APIRouter

router = APIRouter(
    prefix="/users",
    tags=["Users"]
)


@router.get("/")
def get_users():

    return {
        "users": [
            {"id": 1, "name": "Anil"},
            {"id": 2, "name": "Rahul"},
            {"id": 3, "name": "Kiran"}
        ]
    }


@router.get("/{user_id}")
def get_user(user_id: int):

    return {
        "user_id": user_id,
        "name": "Anil"
    }
```

## 2. Create products.py

Location: `app/routers/products.py`

```python
from fastapi import APIRouter

router = APIRouter(
    prefix="/products",
    tags=["Products"]
)


@router.get("/")
def get_products():

    return {
        "products": [
            {"id": 1, "name": "Laptop"},
            {"id": 2, "name": "Mobile"},
            {"id": 3, "name": "Keyboard"}
        ]
    }


@router.get("/{product_id}")
def get_product(product_id: int):

    return {
        "product_id": product_id,
        "name": "Laptop"
    }
```

## 3. Register Routers in main.py

Location: `app/main.py`

```python
from fastapi import FastAPI

from app.routers import users, products

app = FastAPI(
    title="Day 4 Modular FastAPI",
    description="Async and Middleware Assignment",
    version="1.0.0"
)


app.include_router(
    users.router,
    prefix="/api/v1"
)

app.include_router(
    products.router,
    prefix="/api/v1"
)


@app.get("/")
def home():

    return {
        "message": "Modular FastAPI application is running"
    }
```

## 4. Final API Endpoints

| Method | Endpoint                        | Description            |
| ------ | ------------------------------- | ---------------------- |
| GET    | `/`                             | Application status     |
| GET    | `/api/v1/users/`                | Retrieve all users     |
| GET    | `/api/v1/users/{user_id}`       | Retrieve user by ID    |
| GET    | `/api/v1/products/`             | Retrieve all products  |
| GET    | `/api/v1/products/{product_id}` | Retrieve product by ID |

## 5. Verify API Documentation

Run:

```bash
uvicorn app.main:app --reload
```

Open:

```text
http://127.0.0.1:8000/docs
```

The Swagger documentation will display two separate groups:

* Users
* Products

**Conclusion:** APIRouter helps organize large applications into independent modules instead of keeping every endpoint inside one main.py file.

---

# Task F – Lifespan and Background Tasks

## Problem Statement

Demonstrate application startup and shutdown using lifespan.

Explain which operations are suitable for BackgroundTasks and which require a durable task queue.

## 1. Application Lifespan

FastAPI provides lifespan events to initialize and clean up shared resources.

Example:

```python
from contextlib import asynccontextmanager

from fastapi import FastAPI


@asynccontextmanager
async def lifespan(app: FastAPI):

    # Startup
    app.state.ready = True

    print("Application Started")

    yield

    # Shutdown
    app.state.ready = False

    print("Application Shutdown")


app = FastAPI(lifespan=lifespan)


@app.get("/health")
def health_check():

    return {
        "status": "ready" if app.state.ready else "not ready"
    }
```

### Expected Console Output

When the application starts:

```text
Application Started
```

When the application shuts down:

```text
Application Shutdown
```

### Why Use Lifespan?

* Initialize database connection pools.
* Create reusable HTTP clients.
* Load shared application resources.
* Close resources during shutdown.
* Manage application lifecycle cleanly.

## 2. BackgroundTasks

FastAPI provides `BackgroundTasks` for executing small tasks after sending a response.

Example:

```python
from fastapi import FastAPI, BackgroundTasks

app = FastAPI()


def write_audit_log(message: str):

    print("Audit Log:", message)


@app.post("/orders")
def create_order(background_tasks: BackgroundTasks):

    background_tasks.add_task(
        write_audit_log,
        "New order created"
    )

    return {
        "message": "Order accepted successfully"
    }
```

### Expected Response

```json
{
    "message": "Order accepted successfully"
}
```

Console output after the response:

```text
Audit Log: New order created
```

### BackgroundTasks vs Durable Queue

| Feature        | BackgroundTasks       | Durable Task Queue              |
| -------------- | --------------------- | ------------------------------- |
| Execution      | In-process            | Separate worker                 |
| Setup          | Simple                | Requires queue infrastructure   |
| Server restart | Task may be lost      | Can persist pending jobs        |
| Retries        | Not built-in          | Can be implemented              |
| Best for       | Small follow-up tasks | Critical and long-running tasks |

### Suitable BackgroundTasks Operations

* Simple audit logging.
* Small notification operations.
* Non-critical follow-up processing.

### Operations Requiring a Durable Queue

* Invoice generation.
* Important email delivery.
* Payment processing workflows.
* Large file processing.
* Long-running background jobs.

**Conclusion:** BackgroundTasks is useful for lightweight in-process work. Critical jobs should use a durable queue with persistence, retries, monitoring, and appropriate idempotency.

---

# Task G – Project Reflection

## 1. Project Setup

### Requirements

* Python 3
* FastAPI
* Uvicorn

### Install Dependencies

Create `requirements.txt`:

```text
fastapi
uvicorn[standard]
```

Install:

```bash
pip install -r requirements.txt
```

### Run Async Demonstration

```bash
python async_demo.py
```

### Run FastAPI Application

From the `Day-4` root directory:

```bash
uvicorn app.main:app --reload
```

### Open API Documentation

```text
http://127.0.0.1:8000/docs
```

## 2. Implemented Features

| Feature               | Implementation     |
| --------------------- | ------------------ |
| Async Programming     | async_demo.py      |
| Concurrent Execution  | asyncio.gather()   |
| Async Endpoint        | /async-status      |
| Timing Middleware     | X-Process-Time     |
| Request Tracking      | X-Request-ID       |
| Dependency Injection  | Depends()          |
| Sub-dependency        | require_admin()    |
| Authentication Demo   | get_current_user() |
| Modular Routing       | APIRouter          |
| Application Lifecycle | lifespan           |
| Background Processing | BackgroundTasks    |

## 3. Testing and Edge Cases

The following tests should be performed before submission.

### Async Programming

* Run all three coroutines sequentially.
* Run all three using asyncio.gather().
* Compare execution times.
* Test one coroutine that raises an exception.
* Test timeout behavior.

### Async FastAPI

* Access `/async-status`.
* Verify that the endpoint returns a successful response.
* Confirm that non-blocking operations are used.

### Middleware

* Access different endpoints.
* Verify `X-Process-Time` in response headers.
* Verify `X-Request-ID` is generated for each request.

### Dependencies

* Test pagination with valid values.
* Test negative skip values.
* Test limit values greater than 100.
* Access `/admin` with a valid demo token.
* Access `/admin` without authentication.

### APIRouter

* Test `/api/v1/users/`.
* Test `/api/v1/users/1`.
* Test `/api/v1/products/`.
* Test `/api/v1/products/1`.
* Verify router grouping in Swagger UI.

### Lifespan and BackgroundTasks

* Start the application and verify startup logs.
* Stop the application and verify shutdown logs.
* Submit an order and verify the audit task executes.

## 4. Architecture Explanation

The project follows a modular architecture.

Instead of keeping all endpoints in one file, I separated user and product routes into different router modules.

The main application is responsible for registering the routers and configuring the application.

Dependencies are reusable and can be shared across different endpoints.

The lifespan function manages application startup and shutdown.

Middleware handles common request and response operations.

This structure makes the application easier to maintain and extend as the project grows.

## 5. What I Learned

Through this assignment, I learned:

* How synchronous and asynchronous execution differ.
* How coroutines and the event loop work.
* How to execute independent tasks concurrently using asyncio.gather().
* Why blocking operations should be avoided in asynchronous routes.
* How to implement custom middleware.
* How to measure API execution time.
* How to use Depends() for dependency injection.
* How sub-dependencies work.
* How to organize endpoints using APIRouter.
* How to manage startup and shutdown using lifespan.
* How to use BackgroundTasks for lightweight background operations.

One important takeaway was understanding that asynchronous programming is particularly useful for I/O-bound operations, but it does not automatically make CPU-intensive operations faster.

---

# Interview Preparation – Important Questions

### Q1. What is a coroutine?

A coroutine is an asynchronous unit of work defined using `async def`. Calling it creates a coroutine object.

### Q2. What does await do?

`await` suspends the current coroutine until an awaitable operation completes, allowing other tasks to run.

### Q3. What is an event loop?

An event loop schedules and coordinates asynchronous tasks and I/O operations.

### Q4. What is the difference between asyncio.gather() and sequential awaits?

Sequential awaits generally wait for each operation one by one. `asyncio.gather()` allows independent awaitables to execute concurrently.

### Q5. Why is time.sleep() risky in asynchronous code?

It blocks the current thread and prevents the event loop from scheduling other work on that thread.

### Q6. Does async make CPU-heavy operations faster?

No. CPU-intensive operations may require multiprocessing, a process pool, or a separate worker.

### Q7. What is middleware?

Middleware is a layer that wraps request and response processing. It is useful for logging, timing, request IDs, and CORS.

### Q8. What is CORS?

Cross-Origin Resource Sharing is a browser-enforced policy that controls whether a frontend from one origin can access responses from another origin.

### Q9. What is Depends()?

`Depends()` is used by FastAPI to declare and inject dependencies into route functions.

### Q10. What is a sub-dependency?

A dependency that itself depends on another dependency.

### Q11. What is APIRouter?

APIRouter organizes related API endpoints into separate modules using prefixes and tags.

### Q12. When should BackgroundTasks be used?

For small, in-process tasks that do not require durable execution or automatic retries.

### Q13. What is lifespan?

Lifespan manages application startup and shutdown operations, such as initializing and closing shared resources.

### Q14. Why use timing middleware?

It helps measure request processing time and identify performance issues.

---

# Final Key Takeaways

| Concept          | Important Point                              |
| ---------------- | -------------------------------------------- |
| async def        | Defines a coroutine function                 |
| await            | Suspends the current coroutine               |
| Event Loop       | Schedules asynchronous work                  |
| asyncio.gather() | Executes independent awaitables concurrently |
| time.sleep()     | Blocking operation                           |
| asyncio.sleep()  | Non-blocking asynchronous wait               |
| Middleware       | Wraps request and response processing        |
| Depends()        | Provides dependency injection                |
| Sub-dependency   | Dependency inside another dependency         |
| APIRouter        | Organizes API endpoints                      |
| Lifespan         | Manages startup and shutdown                 |
| BackgroundTasks  | Executes small in-process tasks              |

---

# Conclusion

This assignment helped me understand asynchronous programming and how FastAPI handles concurrent operations and request processing.

I implemented asynchronous coroutines, compared sequential and concurrent execution, created an async endpoint, implemented custom middleware, and explored dependency injection and sub-dependencies.

I also organized the application using APIRouter and learned how to manage application resources through lifespan and execute lightweight background tasks.

The main takeaway is that building a maintainable backend application requires more than creating API endpoints. It also involves understanding concurrency, reusable components, application lifecycle, and proper project architecture.

---

**Technologies Used:**

* Python 3
* FastAPI
* Uvicorn
* asyncio
* Pydantic
* REST APIs

---

**Placement Preparation Series | Day 4**

*Async, Middleware & Modular FastAPI*


