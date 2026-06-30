# 🚀 FastAPI + SQLAlchemy ORM + SQLModel ORM

<p align="center">
  <!-- FastAPI -->
  <img src="https://fastapi.tiangolo.com/img/logo-margin/logo-teal.png" alt="FastAPI" width="120"/>
  
  <!-- SQLAlchemy -->
  <img src="https://www.sqlalchemy.org/img/sqla_logo.png" alt="SQLAlchemy" width="120"/>
  
 
  
  <!-- Uvicorn (community-hosted logo) -->
  <img src="https://raw.githubusercontent.com/encode/uvicorn/master/docs/uvicorn.png" alt="Uvicorn" width="120"/>
</p>


---

## 📖 Overview
This template provides a **production‑ready scaffold** for building APIs with **FastAPI** using either:
- **SQLAlchemy ORM** (classic, mature ORM for relational databases)
- **SQLModel ORM** (modern ORM built on top of SQLAlchemy + Pydantic)

It’s designed to help you quickly spin up CRUD endpoints with clean separation of concerns:
- `models/` → ORM models
- `schemas/` → Pydantic/SQLModel schemas
- `routes/` → API endpoints
- `database.py` → DB connection & session management

---

## 🛠️ Tech Stack
- **FastAPI** → High‑performance Python web framework
- **SQLAlchemy ORM** → Traditional ORM for relational databases
- **SQLModel ORM** → Hybrid ORM with Pydantic validation
- **Uvicorn** → Lightning‑fast ASGI server

---
.env
DATABASE_URL=postgresql+psycopg2://postgres:password@localhost:5432/mydb
SECRET_KEY=supersecretjwtkey
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

-----
uvicorn app.main:app --reload


