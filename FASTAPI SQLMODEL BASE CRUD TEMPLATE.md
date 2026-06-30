# ////////////////////////////////////////////////////////
# 🚀 FASTAPI + SQLMODEL PREMIUM CRUD CHEATSHEET (.env READY)
# ////////////////////////////////////////////////////////

==========================================================
1️⃣ DATABASE CONNECTION (app/database/database.py)
==========================================================
from sqlmodel import SQLModel, create_engine, Session
import os
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

DATABASE_URL = os.getenv("DATABASE_URL")  # defined in .env
engine = create_engine(DATABASE_URL, echo=True)

def get_session():
    with Session(engine) as session:
        yield session

def init_db():
    SQLModel.metadata.create_all(engine)

# Example .env file:
# DATABASE_URL=postgresql+psycopg2://postgres:password@localhost:5432/screenly
# SECRET_KEY=supersecretjwtkey
# ALGORITHM=HS256
# ACCESS_TOKEN_EXPIRE_MINUTES=30

==========================================================
2️⃣ MODEL TEMPLATE (app/models/<model>.py)
==========================================================
import uuid
from uuid import UUID
from sqlmodel import SQLModel, Field
from typing import Optional
from datetime import datetime

class <ModelName>(SQLModel, table=True):
    id: UUID = Field(default_factory=uuid.uuid4, primary_key=True)
    field1: str
    field2: Optional[str] = None
    field3: Optional[int] = None
    created_at: datetime = Field(default_factory=datetime.utcnow)

==========================================================
3️⃣ SCHEMA TEMPLATE (app/schemas/<model>.py)
==========================================================
from uuid import UUID
from sqlmodel import SQLModel
from typing import Optional
from datetime import datetime

class <ModelName>Create(SQLModel):
    field1: str
    field2: Optional[str] = None
    field3: Optional[int] = None

class <ModelName>Read(SQLModel):
    id: UUID
    field1: str
    field2: Optional[str] = None
    field3: Optional[int] = None
    created_at: datetime

    class Config:
        orm_mode = True

class <ModelName>Update(SQLModel):
    field1: Optional[str] = None
    field2: Optional[str] = None
    field3: Optional[int] = None

==========================================================
4️⃣ CRUD ROUTES TEMPLATE (app/routers/<model>.py)
==========================================================
from fastapi import APIRouter, Depends, HTTPException
from sqlmodel import Session, select
from uuid import UUID
from app.database.database import get_session
from app.models.<model> import <ModelName>
from app.schemas.<model> import <ModelName>Create, <ModelName>Read, <ModelName>Update

router = APIRouter()

# 🔹 CREATE
@router.post("/<endpoint>", response_model=<ModelName>Read)
def create_<object>(data: <ModelName>Create, session: Session = Depends(get_session)):
    new_obj = <ModelName>.from_orm(data)
    session.add(new_obj)
    session.commit()
    session.refresh(new_obj)
    return new_obj

# 🔹 READ BY ID
@router.get("/<endpoint>/{id}", response_model=<ModelName>Read)
def get_<object>(id: UUID, session: Session = Depends(get_session)):
    obj = session.get(<ModelName>, id)
    if not obj:
        raise HTTPException(status_code=404, detail="<object> not found")
    return obj

# 🔹 READ ALL
@router.get("/<endpoint>", response_model=list[<ModelName>Read])
def get_all_<objects>(session: Session = Depends(get_session)):
    return session.exec(select(<ModelName>)).all()

# 🔹 UPDATE
@router.put("/<endpoint>/{id}", response_model=<ModelName>Read)
def update_<object>(id: UUID, data: <ModelName>Update, session: Session = Depends(get_session)):
    obj = session.get(<ModelName>, id)
    if not obj:
        raise HTTPException(status_code=404, detail="<object> not found")
    for key, value in data.dict(exclude_unset=True).items():
        setattr(obj, key, value)
    session.add(obj)
    session.commit()
    session.refresh(obj)
    return obj

# 🔹 DELETE
@router.delete("/<endpoint>/{id}")
def delete_<object>(id: UUID, session: Session = Depends(get_session)):
    obj = session.get(<ModelName>, id)
    if not obj:
        raise HTTPException(status_code=404, detail="<object> not found")
    session.delete(obj)
    session.commit()
    return {"detail": "<object> deleted successfully"}

==========================================================
5️⃣ KEY NOTES
==========================================================
- .env file keeps secrets safe (DB URL, JWT keys).
- .dict() → converts schema to dictionary for unpacking.
- .dict(exclude_unset=True) → only updates fields provided in request.
- from_orm() → converts schema into ORM model directly.
- orm_mode = True → ensures SQLModel objects can be returned as responses.

==========================================================
🎯 HOW TO USE
==========================================================
- Replace <ModelName> → e.g., Movie, User, Booking.
- Replace <endpoint> → e.g., movies, users, bookings.
- Replace <object> → lowercase singular (e.g., movie, user).
