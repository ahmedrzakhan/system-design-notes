# FastAPI Interview Questions & Answers

## 1. What is FastAPI and why use it?

**Q: Explain FastAPI**

A: FastAPI is modern, fast web framework for building APIs with Python 3.6+. Built on top of Starlette and Pydantic.

**Key features**:

- Fast (high performance, comparable to Go/Node.js)
- Easy to learn and use
- Built-in data validation (Pydantic)
- Automatic interactive API documentation (Swagger UI, ReDoc)
- Type hints for better code quality
- Dependency injection system
- Async/await support
- CORS, authentication, sessions built-in

**Why use FastAPI**:

- 3x faster to code than Flask/Django
- Fewer bugs (type hints catch errors)
- Intuitive, minimal boilerplate
- Production-ready
- Great for microservices and APIs
- Excellent documentation

**vs Flask**: FastAPI has built-in validation, async support, auto-docs. Flask is more minimal.

**vs Django**: FastAPI is lighter weight, faster, better for APIs. Django has more batteries.

---

## 2. What is your first FastAPI app?

**Q: Create a simple FastAPI app**

A:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello, World!"}

@app.get("/items/{item_id}")
def read_item(item_id: int):
    return {"item_id": item_id}

# Run: uvicorn main:app --reload
```

**Folder structure**:

```
myapp/
    main.py
    requirements.txt
```

**requirements.txt**:

```
fastapi
uvicorn[standard]
```

**Run**:

```bash
pip install -r requirements.txt
uvicorn main:app --reload
```

Visit: `http://localhost:8000/docs` — auto-generated Swagger UI

---

## 3. What are path parameters?

**Q: How do you use path parameters?**

A: Extract values from URL path.

```python
@app.get("/items/{item_id}")
def read_item(item_id: int):
    return {"item_id": item_id}

# GET /items/42
# Returns: {"item_id": 42}

@app.get("/users/{user_id}/items/{item_id}")
def read_user_item(user_id: int, item_id: int):
    return {"user_id": user_id, "item_id": item_id}

# GET /users/5/items/10
# Returns: {"user_id": 5, "item_id": 10}
```

**Type validation** — FastAPI validates type automatically:

```python
@app.get("/items/{item_id}")
def read_item(item_id: int):
    # If item_id is not int, FastAPI returns 422 error
    return {"item_id": item_id}

# GET /items/abc  # 422 Unprocessable Entity
```

**Enum path parameter**:

```python
from enum import Enum

class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"

@app.get("/models/{model_name}")
def get_model(model_name: ModelName):
    return {"model_name": model_name}

# GET /models/resnet
# Returns: {"model_name": "resnet"}
```

---

## 4. What are query parameters?

**Q: How do you use query parameters?**

A: Extract values from query string.

```python
@app.get("/items/")
def read_items(skip: int = 0, limit: int = 10):
    return {"skip": skip, "limit": limit}

# GET /items/?skip=20&limit=50
# Returns: {"skip": 20, "limit": 50}

# GET /items/
# Returns: {"skip": 0, "limit": 10}  (defaults)
```

**Optional query parameters**:

```python
from typing import Optional

@app.get("/items/")
def read_items(q: Optional[str] = None):
    if q:
        return {"q": q}
    return {"q": "not provided"}

# GET /items/?q=search
# Returns: {"q": "search"}

# GET /items/
# Returns: {"q": "not provided"}
```

**Multiple values**:

```python
from typing import List

@app.get("/items/")
def read_items(q: List[str] = []):
    return {"q": q}

# GET /items/?q=a&q=b&q=c
# Returns: {"q": ["a", "b", "c"]}
```

---

## 5. What is request body and Pydantic models?

**Q: How do you handle request bodies?**

A: Use Pydantic models for request validation.

```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None

@app.post("/items/")
def create_item(item: Item):
    return item

# POST /items/
# Body: {"name": "Laptop", "price": 1000.0}
# Returns: {"name": "Laptop", "description": null, "price": 1000.0, "tax": null}
```

**Request validation** — FastAPI validates automatically:

```python
# POST /items/
# Body: {"name": "Laptop"}  # Missing price
# Returns: 422 error

# Body: {"name": "Laptop", "price": "invalid"}
# Returns: 422 error (price must be float)
```

**Nested models**:

```python
class Address(BaseModel):
    street: str
    city: str

class Person(BaseModel):
    name: str
    address: Address

@app.post("/people/")
def create_person(person: Person):
    return person

# POST /people/
# Body: {
#   "name": "John",
#   "address": {
#     "street": "123 Main St",
#     "city": "NYC"
#   }
# }
```

**Model example in docs**:

```python
class Item(BaseModel):
    name: str
    price: float

    class Config:
        schema_extra = {
            "example": {
                "name": "Laptop",
                "price": 1000.0
            }
        }
```

---

## 6. What are HTTP methods?

**Q: Different HTTP operations**

A: FastAPI supports all HTTP methods.

```python
@app.post("/items/")
def create_item(item: Item):
    return {"created": item}

@app.get("/items/{item_id}")
def read_item(item_id: int):
    return {"item_id": item_id}

@app.put("/items/{item_id}")
def update_item(item_id: int, item: Item):
    return {"item_id": item_id, "item": item}

@app.patch("/items/{item_id}")
def partial_update_item(item_id: int, item: dict):
    return {"item_id": item_id, "item": item}

@app.delete("/items/{item_id}")
def delete_item(item_id: int):
    return {"deleted": item_id}

@app.head("/items/{item_id}")
def head_item(item_id: int):
    pass

@app.options("/items/{item_id}")
def options_item(item_id: int):
    pass
```

**RESTful design**:

- POST — create
- GET — read
- PUT/PATCH — update
- DELETE — delete

---

## 7. What is response model?

**Q: How do you format responses?**

A: Use response_model to validate and document responses.

```python
from pydantic import BaseModel

class ItemResponse(BaseModel):
    id: int
    name: str
    price: float

class ItemRequest(BaseModel):
    name: str
    price: float

@app.post("/items/", response_model=ItemResponse)
def create_item(item: ItemRequest):
    return {"id": 1, "name": item.name, "price": item.price}
```

**Exclude fields**:

```python
@app.get("/items/{item_id}", response_model=Item, response_model_exclude={"tax"})
def read_item(item_id: int):
    return {"id": item_id, "name": "Item", "price": 100, "tax": 10}

# Returns: {"id": item_id, "name": "Item", "price": 100}
```

**Include/exclude by condition**:

```python
@app.get("/items/{item_id}", response_model_exclude_unset=True)
def read_item(item_id: int):
    return {"id": item_id, "name": "Item"}

# Excludes fields not explicitly set
```

**List response**:

```python
@app.get("/items/", response_model=List[Item])
def read_items():
    return [{"id": 1, "name": "Item1"}, {"id": 2, "name": "Item2"}]
```

---

## 8. What is status codes?

**Q: How do you set HTTP status codes?**

A: Specify status_code parameter or return Response object.

```python
from fastapi import status

@app.post("/items/", status_code=status.HTTP_201_CREATED)
def create_item(item: Item):
    return item

# Returns 201 Created instead of default 200 OK

@app.get("/items/{item_id}", status_code=status.HTTP_200_OK)
def read_item(item_id: int):
    return {"item_id": item_id}

@app.delete("/items/{item_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_item(item_id: int):
    pass

# 204 No Content
```

**Dynamic status codes**:

```python
from fastapi import Response

@app.post("/items/")
def create_item(item: Item, response: Response):
    response.status_code = status.HTTP_201_CREATED
    return item
```

**Common status codes**:

- 200 — OK
- 201 — Created
- 204 — No Content
- 400 — Bad Request
- 401 — Unauthorized
- 403 — Forbidden
- 404 — Not Found
- 422 — Unprocessable Entity (validation error)
- 500 — Internal Server Error

---

## 9. What is error handling?

**Q: How do you handle errors?**

A: Raise HTTPException for API errors.

```python
from fastapi import HTTPException

@app.get("/items/{item_id}")
def read_item(item_id: int):
    if item_id < 0:
        raise HTTPException(
            status_code=400,
            detail="Item ID must be positive"
        )
    if item_id > 1000:
        raise HTTPException(
            status_code=404,
            detail="Item not found"
        )
    return {"item_id": item_id}
```

**Custom exception handlers**:

```python
from fastapi import Request
from fastapi.responses import JSONResponse

class CustomException(Exception):
    def __init__(self, name: str):
        self.name = name

@app.exception_handler(CustomException)
async def custom_exception_handler(request: Request, exc: CustomException):
    return JSONResponse(
        status_code=400,
        content={"detail": f"Custom error: {exc.name}"}
    )

@app.get("/items/")
def read_items():
    raise CustomException(name="test")
```

**Validation error details**:

```python
# FastAPI automatically returns validation errors with details
# POST /items/ with invalid data returns:
# {
#   "detail": [
#     {
#       "loc": ["body", "price"],
#       "msg": "value is not a valid integer",
#       "type": "type_error.integer"
#     }
#   ]
# }
```

---

## 10. What is async and await?

**Q: How do you use async in FastAPI?**

A: FastAPI has built-in async support.

```python
# Async endpoint — non-blocking
@app.get("/items/")
async def read_items():
    await some_async_operation()
    return {"items": []}

async def some_async_operation():
    # Sleep doesn't block other requests
    import asyncio
    await asyncio.sleep(1)
    return

# Regular endpoint — blocking
@app.get("/users/")
def read_users():
    # Blocking I/O
    time.sleep(1)
    return {"users": []}
```

**Benefits of async**:

- Handle more concurrent requests
- Non-blocking I/O operations
- Better performance

**Async database query**:

```python
import databases

database = databases.Database("postgresql://user:password@localhost/dbname")

@app.on_event("startup")
async def startup():
    await database.connect()

@app.on_event("shutdown")
async def shutdown():
    await database.disconnect()

@app.get("/items/")
async def read_items():
    query = "SELECT * FROM items"
    items = await database.fetch_all(query)
    return items
```

**When to use async**:

- Database queries
- External API calls
- File I/O
- Long-running operations

**When not needed**:

- CPU-intensive operations (use threads/processes)
- Simple operations

---

## 11. What are dependencies?

**Q: Explain FastAPI dependency injection**

A: Reusable logic via dependencies.

```python
from fastapi import Depends

# Simple dependency
def get_query(q: Optional[str] = None):
    return q

@app.get("/items/")
def read_items(q: str = Depends(get_query)):
    return {"q": q}
```

**Database dependency**:

```python
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/items/")
def read_items(db: Session = Depends(get_db)):
    return db.query(Item).all()
```

**Nested dependencies**:

```python
def get_user(db: Session = Depends(get_db)):
    user = db.query(User).first()
    return user

@app.get("/items/")
def read_items(user: User = Depends(get_user)):
    return {"user": user}
```

**Security dependency**:

```python
from fastapi.security import HTTPBasic, HTTPBasicCredentials

security = HTTPBasic()

@app.get("/secure/")
def secure_endpoint(credentials: HTTPBasicCredentials = Depends(security)):
    return {"username": credentials.username}
```

**Benefits**:

- Code reuse
- Easy to test
- Dependency graph handled by FastAPI
- Dependency caching within request

---

## 12. What is authentication?

**Q: How do you implement authentication?**

A: Multiple ways to authenticate.

**OAuth2 with Password Flow**:

```python
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from passlib.context import CryptContext
from datetime import datetime, timedelta
from jose import JWTError, jwt

pwd_context = CryptContext(schemes=["bcrypt"])
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

@app.post("/token")
def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user = authenticate_user(form_data.username, form_data.password)
    if not user:
        raise HTTPException(status_code=401)

    token = create_access_token(data={"sub": user.username})
    return {"access_token": token, "token_type": "bearer"}

@app.get("/users/me")
def read_current_user(token: str = Depends(oauth2_scheme)):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        username = payload.get("sub")
    except JWTError:
        raise HTTPException(status_code=401)

    user = get_user(username)
    return user
```

**API Key authentication**:

```python
from fastapi.security import APIKeyHeader

api_key_header = APIKeyHeader(name="X-API-Key")

@app.get("/secure/")
def secure_endpoint(api_key: str = Depends(api_key_header)):
    if api_key != "valid-key":
        raise HTTPException(status_code=403)
    return {"message": "Authorized"}
```

---

## 13. What is CORS?

**Q: How do you enable CORS?**

A: Cross-Origin Resource Sharing for frontend requests.

```python
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# Allow specific origins
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Allow all origins (development only!)
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=False,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

**Options**:

- `allow_origins` — list of allowed origins
- `allow_methods` — allowed HTTP methods
- `allow_headers` — allowed headers
- `allow_credentials` — allow cookies/auth headers
- `max_age` — cache time in seconds

---

## 14. What are middleware?

**Q: How do you use middleware?**

A: Middleware processes requests/responses.

```python
from starlette.middleware.base import BaseHTTPMiddleware
from time import time

class TimingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        start = time()
        response = await call_next(request)
        duration = time() - start
        response.headers["X-Process-Time"] = str(duration)
        return response

app.add_middleware(TimingMiddleware)
```

**Built-in middleware**:

```python
from fastapi.middleware.trustedhost import TrustedHostMiddleware
from fastapi.middleware.gzip import GZIPMiddleware

app.add_middleware(GZIPMiddleware, minimum_size=1000)
app.add_middleware(TrustedHostMiddleware, allowed_hosts=["example.com"])
```

---

## 15. What is logging?

**Q: How do you log in FastAPI?**

A:

```python
import logging

logger = logging.getLogger(__name__)

@app.get("/items/")
def read_items():
    logger.info("Reading items")
    logger.debug("Debug message")
    logger.error("Error occurred")
    return {"items": []}
```

**Logging configuration**:

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s"
)

logger = logging.getLogger(__name__)
```

---

## 16. What is testing?

**Q: How do you test FastAPI?**

A: Use TestClient.

```python
from fastapi.testclient import TestClient

client = TestClient(app)

def test_read_items():
    response = client.get("/items/")
    assert response.status_code == 200
    assert response.json() == {"items": []}

def test_create_item():
    response = client.post("/items/", json={"name": "Item", "price": 100})
    assert response.status_code == 200
    assert response.json()["name"] == "Item"

def test_item_not_found():
    response = client.get("/items/999")
    assert response.status_code == 404
```

**Run tests**:

```bash
pytest
pytest -v  # Verbose
pytest tests/test_main.py  # Specific file
```

---

## 17. What is database integration?

**Q: How do you use databases?**

A: Use SQLAlchemy ORM.

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import declarative_base, sessionmaker
from sqlalchemy import Column, Integer, String

DATABASE_URL = "sqlite:///./test.db"

engine = create_engine(DATABASE_URL, connect_args={"check_same_thread": False})
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

class Item(Base):
    __tablename__ = "items"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, index=True)
    price = Column(float)

Base.metadata.create_all(bind=engine)

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.post("/items/")
def create_item(item: ItemSchema, db: Session = Depends(get_db)):
    db_item = Item(**item.dict())
    db.add(db_item)
    db.commit()
    db.refresh(db_item)
    return db_item

@app.get("/items/")
def read_items(db: Session = Depends(get_db)):
    return db.query(Item).all()
```

---

## 18. What is background tasks?

**Q: How do you run background tasks?**

A: Use BackgroundTasks.

```python
from fastapi import BackgroundTasks

def write_log(message: str):
    import time
    time.sleep(1)
    with open("log.txt", "a") as log:
        log.write(message + "\n")

@app.post("/send-notification/")
def send_notification(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(write_log, f"Notification sent to {email}")
    return {"message": "Notification sent in background"}

# Multiple tasks
@app.post("/tasks/")
def add_tasks(background_tasks: BackgroundTasks):
    background_tasks.add_task(task1)
    background_tasks.add_task(task2, arg1, arg2)
    return {"message": "Tasks added"}
```

---

## 19. What is file uploads?

**Q: How do you handle file uploads?**

A:

```python
from fastapi import File, UploadFile

@app.post("/uploadfile/")
async def upload_file(file: UploadFile = File(...)):
    contents = await file.read()
    return {"filename": file.filename}

# Multiple files
@app.post("/uploadfiles/")
async def upload_files(files: List[UploadFile] = File(...)):
    return [{"filename": file.filename} for file in files]

# Save file
@app.post("/upload/")
async def upload_file(file: UploadFile = File(...)):
    with open(f"uploads/{file.filename}", "wb") as f:
        contents = await file.read()
        f.write(contents)
    return {"filename": file.filename}
```

---

## 20. What is request and response?

**Q: Work with Request/Response objects**

A:

```python
from fastapi import Request, Response

@app.get("/items/")
async def read_items(request: Request):
    return {
        "url": request.url,
        "method": request.method,
        "headers": dict(request.headers),
    }

@app.get("/custom-response/")
def custom_response():
    return Response(
        content="Custom response",
        status_code=200,
        media_type="text/plain"
    )

@app.get("/json-response/")
def json_response():
    from fastapi.responses import JSONResponse
    return JSONResponse({"message": "Hello"})

@app.get("/file-response/")
def file_response():
    from fastapi.responses import FileResponse
    return FileResponse("file.pdf")
```

---

## 21. What is validation?

**Q: Pydantic validation**

A:

```python
from pydantic import BaseModel, Field, validator

class Item(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    price: float = Field(..., gt=0)
    tax: Optional[float] = Field(None, ge=0)

    @validator('name')
    def name_must_be_alphanumeric(cls, v):
        if not v.isalnum():
            raise ValueError('Name must be alphanumeric')
        return v

@app.post("/items/")
def create_item(item: Item):
    return item
```

**Field options**:

- `min_length` / `max_length` — string length
- `gt` / `ge` — greater than / greater than or equal
- `lt` / `le` — less than / less than or equal
- `regex` — regex pattern
- `description` — field description for docs

---

## 22. What is pagination?

**Q: How do you implement pagination?**

A:

```python
from typing import List

@app.get("/items/")
def read_items(skip: int = 0, limit: int = 10):
    # Simulate database query
    items = [{"id": i, "name": f"Item {i}"} for i in range(100)]
    return items[skip: skip + limit]

# GET /items/?skip=0&limit=10
# GET /items/?skip=10&limit=10
```

**With database**:

```python
@app.get("/items/")
def read_items(skip: int = 0, limit: int = 10, db: Session = Depends(get_db)):
    items = db.query(Item).offset(skip).limit(limit).all()
    return items
```

---

## 23. What is filtering and sorting?

**Q: Filter and sort results**

A:

```python
@app.get("/items/")
def read_items(
    skip: int = 0,
    limit: int = 10,
    sort_by: str = "name",
    ascending: bool = True,
    min_price: Optional[float] = None,
    max_price: Optional[float] = None,
    db: Session = Depends(get_db)
):
    query = db.query(Item)

    # Filter
    if min_price:
        query = query.filter(Item.price >= min_price)
    if max_price:
        query = query.filter(Item.price <= max_price)

    # Sort
    if sort_by == "name":
        query = query.order_by(Item.name if ascending else Item.name.desc())
    elif sort_by == "price":
        query = query.order_by(Item.price if ascending else Item.price.desc())

    # Paginate
    items = query.offset(skip).limit(limit).all()
    return items

# GET /items/?min_price=10&max_price=100&sort_by=price&ascending=false
```

---

## 24. What are environment variables?

**Q: How do you use environment variables?**

A:

```python
from pydantic import BaseSettings
import os

class Settings(BaseSettings):
    app_name: str = "FastAPI App"
    debug: bool = False
    database_url: str = "sqlite:///./test.db"
    secret_key: str = "secret"

    class Config:
        env_file = ".env"

settings = Settings()

@app.get("/")
def read_root():
    return {"app_name": settings.app_name}
```

**.env file**:

```
APP_NAME="My API"
DEBUG=true
DATABASE_URL="postgresql://user:pass@localhost/db"
SECRET_KEY="my-secret"
```

---

## 25. What is automatic documentation?

**Q: FastAPI auto-generates docs**

A: FastAPI generates interactive documentation automatically.

**Swagger UI**:

```
http://localhost:8000/docs
```

**ReDoc**:

```
http://localhost:8000/redoc
```

**Customize docs**:

```python
app = FastAPI(
    title="My API",
    description="This is my API",
    version="1.0.0",
    docs_url="/api/docs",
    redoc_url="/api/redoc",
    openapi_url="/api/openapi.json"
)
```

**Docstring in endpoint**:

```python
@app.get("/items/{item_id}")
def read_item(item_id: int):
    """
    Get an item by ID.

    - **item_id**: The ID of the item
    """
    return {"item_id": item_id}
```

---

## 26. What are OpenAPI and JSON Schema?

**Q: OpenAPI specification**

A: FastAPI uses OpenAPI to generate docs.

```python
from fastapi.openapi.utils import get_openapi

def custom_openapi():
    if app.openapi_schema:
        return app.openapi_schema

    openapi_schema = get_openapi(
        title="My API",
        version="1.0.0",
        description="My awesome API",
        routes=app.routes,
    )
    app.openapi_schema = openapi_schema
    return app.openapi_schema

app.openapi = custom_openapi
```

---

## 27. What is API versioning?

**Q: How do you version your API?**

A:

```python
# URL path versioning
@app.get("/v1/items/")
def read_items_v1():
    return {"version": "1"}

@app.get("/v2/items/")
def read_items_v2():
    return {"version": "2"}

# Header versioning
@app.get("/items/")
def read_items(api_version: str = Header(default="1")):
    if api_version == "1":
        return {"version": "1"}
    else:
        return {"version": "2"}

# Query parameter versioning
@app.get("/items/")
def read_items(v: int = 1):
    if v == 1:
        return {"version": "1"}
    else:
        return {"version": "2"}
```

---

## 28. What is rate limiting?

**Q: How do you implement rate limiting?**

A: Use slowapi library.

```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter

@app.get("/items/")
@limiter.limit("5/minute")
def read_items(request: Request):
    return {"items": []}
```

---

## 29. What is request ID tracking?

**Q: Track requests with IDs**

A:

```python
import uuid
from starlette.middleware.base import BaseHTTPMiddleware

class RequestIDMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        request_id = str(uuid.uuid4())
        request.state.request_id = request_id
        response = await call_next(request)
        response.headers["X-Request-ID"] = request_id
        return response

app.add_middleware(RequestIDMiddleware)

@app.get("/items/")
def read_items(request: Request):
    request_id = request.state.request_id
    return {"request_id": request_id}
```

---

## 30. What is production deployment?

**Q: How do you deploy FastAPI?**

A: Multiple ways to deploy.

**Using Gunicorn + Uvicorn**:

```bash
pip install gunicorn uvicorn

gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app
```

**Docker**:

```dockerfile
FROM python:3.9

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Docker compose**:

```yaml
version: "3"
services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql://user:pass@db:5432/dbname
    depends_on:
      - db
  db:
    image: postgres:13
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: dbname
```

**Cloud platforms**:

- Heroku: `git push heroku main`
- AWS: ECS, Lambda, Elastic Beanstalk
- Google Cloud: Cloud Run, App Engine
- Azure: App Service, Container Instances
- DigitalOcean: App Platform, Droplets

---

## 31. What are common best practices?

**Q: FastAPI best practices**

A:

1. **Use type hints** — everywhere
2. **Use dependency injection** — for DRY code
3. **Organize code** — by features/modules
4. **Use async** — for I/O operations
5. **Validate input** — with Pydantic
6. **Handle errors** — with HTTPException
7. **Use environment variables** — for config
8. **Write tests** — with TestClient
9. **Use middleware** — for cross-cutting concerns
10. **Document API** — with docstrings
11. **Use proper HTTP methods** — GET, POST, PUT, DELETE
12. **Return proper status codes** — 200, 201, 400, 404, 500

---

## 32. What is project structure?

**Q: How to organize FastAPI project**

A:

```
myapi/
    main.py
    requirements.txt
    .env
    .gitignore
    docker-compose.yml
    Dockerfile

    app/
        __init__.py
        main.py
        models/
            __init__.py
            database.py
            schemas.py
        routes/
            __init__.py
            items.py
            users.py
        dependencies.py
        config.py
        tests/
            __init__.py
            test_items.py
            test_users.py
```

**main.py**:

```python
from fastapi import FastAPI
from app.routes import items, users
from app.config import settings

app = FastAPI(title=settings.app_name)

app.include_router(items.router)
app.include_router(users.router)

@app.get("/")
def read_root():
    return {"message": "Hello"}
```

**routes/items.py**:

```python
from fastapi import APIRouter

router = APIRouter(prefix="/items", tags=["items"])

@router.get("/")
def read_items():
    return {"items": []}
```

---

## 33. What are common issues?

**Q: Common FastAPI issues**

A:

**1. Blocking operations**:

```python
# Bad — blocks event loop
@app.get("/items/")
async def read_items():
    time.sleep(5)  # Blocks!
    return {}

# Good — use async operation
@app.get("/items/")
async def read_items():
    await asyncio.sleep(5)
    return {}
```

**2. Database connection pooling**:

```python
# Bad — creates new connection per request
engine = create_engine(DATABASE_URL)

# Good — reuse connections
engine = create_engine(
    DATABASE_URL,
    pool_pre_ping=True,
    pool_size=10,
    max_overflow=20
)
```

**3. N+1 queries**:

```python
# Bad — multiple queries
users = db.query(User).all()
for user in users:
    print(user.posts)  # Query per user!

# Good — eager load
users = db.query(User).options(joinedload(User.posts)).all()
```

**4. Unvalidated input**:

```python
# Good — Pydantic validates
class Item(BaseModel):
    price: float = Field(..., gt=0)

@app.post("/items/")
def create_item(item: Item):
    return item
```

---

## 34. What is real-world example?

**Q: Build a complete API**

A:

```python
from fastapi import FastAPI, Depends, HTTPException, status
from pydantic import BaseModel, Field
from sqlalchemy import Column, Integer, String, Float, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session
from typing import List

DATABASE_URL = "sqlite:///./items.db"
engine = create_engine(DATABASE_URL, connect_args={"check_same_thread": False})
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

# Database model
class ItemDB(Base):
    __tablename__ = "items"
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, index=True)
    price = Column(Float)

Base.metadata.create_all(bind=engine)

# Pydantic models
class ItemCreate(BaseModel):
    name: str = Field(..., min_length=1)
    price: float = Field(..., gt=0)

class Item(ItemCreate):
    id: int
    class Config:
        from_attributes = True

# Dependency
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# API
app = FastAPI(title="Item API")

@app.post("/items/", response_model=Item, status_code=status.HTTP_201_CREATED)
def create_item(item: ItemCreate, db: Session = Depends(get_db)):
    db_item = ItemDB(**item.dict())
    db.add(db_item)
    db.commit()
    db.refresh(db_item)
    return db_item

@app.get("/items/", response_model=List[Item])
def read_items(skip: int = 0, limit: int = 10, db: Session = Depends(get_db)):
    return db.query(ItemDB).offset(skip).limit(limit).all()

@app.get("/items/{item_id}", response_model=Item)
def read_item(item_id: int, db: Session = Depends(get_db)):
    item = db.query(ItemDB).filter(ItemDB.id == item_id).first()
    if not item:
        raise HTTPException(status_code=404, detail="Item not found")
    return item

@app.put("/items/{item_id}", response_model=Item)
def update_item(item_id: int, item: ItemCreate, db: Session = Depends(get_db)):
    db_item = db.query(ItemDB).filter(ItemDB.id == item_id).first()
    if not db_item:
        raise HTTPException(status_code=404, detail="Item not found")
    db_item.name = item.name
    db_item.price = item.price
    db.commit()
    db.refresh(db_item)
    return db_item

@app.delete("/items/{item_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_item(item_id: int, db: Session = Depends(get_db)):
    db_item = db.query(ItemDB).filter(ItemDB.id == item_id).first()
    if not db_item:
        raise HTTPException(status_code=404, detail="Item not found")
    db.delete(db_item)
    db.commit()
```

---

## FastAPI Interview Tips

1. **Type hints everywhere** — Core FastAPI feature
2. **Pydantic models** — Validation is key
3. **Async/await** — Know when to use
4. **Dependencies** — Powerful DI system
5. **Error handling** — HTTPException for APIs
6. **Testing** — TestClient for unit tests
7. **Database** — SQLAlchemy integration
8. **Authentication** — OAuth2 common pattern
9. **Documentation** — Automatic Swagger/ReDoc
10. **Deployment** — Know Docker, Gunicorn
11. **Real projects** — Talk about production experience
12. **Performance** — Async operations, pagination, caching
