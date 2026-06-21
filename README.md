from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from sqlalchemy.orm import declarative_base

DATABASE_URL = "postgresql://postgres:123456@localhost/kontalo"

engine = create_engine(
    DATABASE_URL
)

SessionLocal = sessionmaker(
    autocommit=False,
    autoflush=False,
    bind=engine
)

Base = declarative_base().  from sqlalchemy import Column
from sqlalchemy import Integer
from sqlalchemy import String
from sqlalchemy import ForeignKey

from database import Base


class User(Base):

    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String)
    email = Column(String, unique=True, index=True)
    password_hash = Column(String)


class Company(Base):

    __tablename__ = "companies"

    id = Column(Integer, primary_key=True)
    name = Column(String)
    sector = Column(String)
    currency = Column(String)
    owner_id = Column(Integer, ForeignKey("users.id"))


class Invoice(Base):

    __tablename__ = "invoices"

    id = Column(Integer, primary_key=True)
    company_id = Column(Integer, ForeignKey("companies.id"))
    client = Column(String)
    amount = Column(Integer)
    status = Column(String)
    date = Column(String).  from passlib.context import CryptContext

pwd_context = CryptContext(
    schemes=["bcrypt"],
    deprecated="auto"
)

def hash_password(password: str):
    return pwd_context.hash(password)

def verify_password(password: str, hashed: str):
    return pwd_context.verify(password, hashed).  from pydantic import BaseModel


class UserRegister(BaseModel):
    name: str
    email: str
    password: str


class UserLogin(BaseModel):
    email: str
    password: str from fastapi import FastAPI
from fastapi import HTTPException

from database import Base
from database import engine
from database import SessionLocal

from models import User

from schemas import UserRegister
from schemas import UserLogin

from auth import hash_password
from auth import verify_password

app = FastAPI()

Base.metadata.create_all(bind=engine) @app.post("/register")
def register(user: UserRegister):

    db = SessionLocal()

    existing = db.query(User).filter(
        User.email == user.email
    ).first()

    if existing:
        raise HTTPException(
            status_code=400,
            detail="Email already exists"
        )

    new_user = User(
        name=user.name,
        email=user.email,
        password_hash=hash_password(user.password)
    )

    db.add(new_user)
    db.commit()

    return {"message": "User created"}  @app.post("/login")
def login(user: UserLogin):

    db = SessionLocal()

    db_user = db.query(User).filter(
        User.email == user.email
    ).first()

    if not db_user:
        raise HTTPException(
            status_code=401,
            detail="Invalid credentials"
        )

    if not verify_password(
        user.password,
        db_user.password_hash
    ):
        raise HTTPException(
            status_code=401,
            detail="Invalid credentials"
        )

    return {"message": "Login successful"}.   