# ////////////////////////////////////////////////////////
# 🚀 FASTAPI + SQLALCHEMY CRUD CHEATSHEET (.env READY)
# ////////////////////////////////////////////////////////

==========================================================
1️⃣ DATABASE CONNECTION (app/database.py)
==========================================================
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

# Example .env file:
# DATABASE_URL=postgresql+psycopg2://postgres:password@localhost:5432/screenly

==========================================================
2️⃣ ORM MODEL TEMPLATE (app/models/<model>.py)
==========================================================
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

==========================================================
3️⃣ SCHEMA TEMPLATE (app/schemas/<model>.py)
==========================================================
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

==========================================================
4️⃣ CRUD ROUTES TEMPLATE (app/routers/<model>.py)
==========================================================
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

==========================================================
5️⃣ KEY NOTES
==========================================================
- SQLAlchemy uses `db.query(Model)` for queries.
- `.dict()` → converts Pydantic schema to dictionary for unpacking.
- `.dict(exclude_unset=True)` → only updates fields provided in request.
- `orm_mode = True` → ensures ORM objects can be returned as responses.

==========================================================
🎯 HOW TO USE
==========================================================
- Replace <ModelName> → e.g., Employee, Movie, User.
- Replace <endpoint> → e.g., employees, movies, users.
- Replace <object> → lowercase singular (e.g., employee, movie).
