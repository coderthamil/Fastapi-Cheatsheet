# 🚀 FastAPI + SQLAlchemy CRUD Cheatsheet (.env Ready)

<p align="center">
  <img src="https://fastapi.tiangolo.com/img/logo-margin/logo-teal.png" alt="FastAPI" width="120"/>
  <img src="https://www.sqlalchemy.org/img/sqla_logo.png" alt="SQLAlchemy" width="120"/>
  <img src="https://raw.githubusercontent.com/encode/uvicorn/master/docs/uvicorn.png" alt="Uvicorn" width="120"/>
</p>

<p align="center">
  <b>Production-ready CRUD scaffold for FastAPI with SQLAlchemy ORM</b>
</p>

---

## ✨ Features
- ⚡ **FastAPI** → High-performance Python web framework
- 🗄️ **SQLAlchemy ORM** → Mature ORM for relational databases
- 🔑 **Environment-based configuration (.env)**
- 📦 **Reusable CRUD routes**
- 🔄 **Alembic migrations ready**

---

## 📂 Project Structure
```📁 app
├── 📁 models        # SQLAlchemy models
├── 📁 schemas       # Pydantic schemas
├── 📁 routers       # API endpoints (CRUD)
├── 📁 database.py   # DB connection/session
├── main.py          # FastAPI entrypoint
└── .env             # Environment variables
```
---

## ⚡ Database Connection (`app/database.py`)
```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, declarative_base
import os
from dotenv import load_dotenv

load_dotenv()

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://user:password@localhost/screenly")

engine = create_engine(DATABASE_URL, echo=True)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()


🗄️ ORM Model Template (app/models/<model>.py)
import uuid
from uuid import UUID
from sqlalchemy import Column, String, Integer
from sqlalchemy.dialects.postgresql import UUID as pgUUID
from app.database import Base

class <ModelName>(Base):
    __tablename__ = "<endpoint>"

    id = Column(pgUUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    field1 = Column(String, nullable=False)
    field2 = Column(Integer, nullable=True)
    field3 = Column(String, nullable=True)
```

📦 Schema Template (app/schemas/<model>.py)
```python
from uuid import UUID
from pydantic import BaseModel
from typing import Optional

class <ModelName>Create(BaseModel):
    field1: str
    field2: Optional[int] = None
    field3: Optional[str] = None

class <ModelName>Read(BaseModel):
    id: UUID
    field1: str
    field2: Optional[int] = None
    field3: Optional[str] = None

    class Config:
        orm_mode = True

class <ModelName>Update(BaseModel):
    field1: Optional[str] = None
    field2: Optional[int] = None
    field3: Optional[str] = None
```
⚡ CRUD Routes Template (app/routers/<model>.py)
```
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from sqlalchemy.exc import IntegrityError
from uuid import UUID
from app.database import get_db
from app.models.<model> import <ModelName>
from app.schemas.<model> import <ModelName>Create, <ModelName>Read, <ModelName>Update

router = APIRouter()

# 🔹 CREATE
@router.post("/<endpoint>", response_model=<ModelName>Read)
def create_<object>(payload: <ModelName>Create, db: Session = Depends(get_db)):
    new_obj = <ModelName>(**payload.dict())
    db.add(new_obj)
    try:
        db.commit()
        db.refresh(new_obj)
    except IntegrityError:
        db.rollback()
        raise HTTPException(status_code=400, detail="Integrity error")
    return new_obj

# 🔹 READ BY ID
@router.get("/<endpoint>/{id}", response_model=<ModelName>Read)
def get_<object>(id: UUID, db: Session = Depends(get_db)):
    obj = db.query(<ModelName>).filter(<ModelName>.id == id).first()
    if not obj:
        raise HTTPException(status_code=404, detail="<object> not found")
    return obj

# 🔹 READ ALL
@router.get("/<endpoint>", response_model=list[<ModelName>Read])
def get_all_<objects>(db: Session = Depends(get_db)):
    return db.query(<ModelName>).all()

# 🔹 UPDATE
@router.put("/<endpoint>/{id}", response_model=<ModelName>Read)
def update_<object>(id: UUID, payload: <ModelName>Update, db: Session = Depends(get_db)):
    obj = db.query(<ModelName>).filter(<ModelName>.id == id).first()
    if not obj:
        raise HTTPException(status_code=404, detail="<object> not found")
    for key, value in payload.dict(exclude_unset=True).items():
        setattr(obj, key, value)
    db.commit()
    db.refresh(obj)
    return obj

# 🔹 DELETE
@router.delete("/<endpoint>/{id}")
def delete_<object>(id: UUID, db: Session = Depends(get_db)):
    obj = db.query(<ModelName>).filter(<ModelName>.id == id).first()
    if not obj:
        raise HTTPException(status_code=404, detail="<object> not found")
    db.delete(obj)
    db.commit()
    return {"detail": "<object> deleted successfully"}
   ``` 
==================================================================
📝 Key Notes
🗄️ SQLAlchemy uses db.query(Model) for queries.

⚙️ .dict() → converts Pydantic schema to dictionary for unpacking.

🔄 .dict(exclude_unset=True) → only updates fields provided in request.

✅ orm_mode = True → ensures ORM objects can be returned as responses.

🎯 How to Use
Replace <ModelName> → e.g., Employee, Movie, User.

Replace <endpoint> → e.g., employees, movies, users.

Replace <object> → lowercase singular (e.g., employee, movie).

---
This enhanced `README.md` now has a **premium design**: logos, badges, clear sections, and professional formatting.  

👉 Would you like me to also prepare a **side‑by‑side comparison section** showing how the same CRUD looks in **SQLAlchemy vs SQLModel** inside this README?
