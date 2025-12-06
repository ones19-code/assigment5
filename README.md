# ssignment 5 — Docker, MariaDB, SQL & SQLAlchemy

This project demonstrates the use of Docker, MariaDB (SQL), and SQLAlchemy (Core & ORM) to perform basic database operations, including:

Running MariaDB inside Docker using docker-compose

Initializing the database using SQL scripts

Connecting to MariaDB using SQLAlchemy Engine

Defining database schemas using SQLAlchemy Core

Defining ORM models using SQLAlchemy Declarative Base

Creating a session factory for DB operations

Implementing CRUD functions:

Retrieve all users

Retrieve a user by username

Insert a new user

Update user information

📁 Project Structure
projet_sqlalchemy_docker/
│
├── docker-compose.yml          # MariaDB container configuration
├── init.sql                    # DB initialization script
│
├── src/
│   ├── database.py             # SQLAlchemy engine + session factory
│   ├── tables.py               # SQLAlchemy Core table definitions
│   ├── models.py               # SQLAlchemy ORM models
│   ├── crud.py                 # CRUD operations
│   └── main.py                 # Tests and demo script
│
└── README.md

🐳 Docker Setup

To start the MariaDB database inside Docker:

docker compose up -d


Check that the container is running:

docker ps


Expected output example:

CONTAINER ID   IMAGE            STATUS          PORTS
xxxxxxxxxxxx   mariadb:latest   Up 20 seconds   0.0.0.0:3306->3306/tcp

🛢️ Database Initialization (init.sql)

This file is executed automatically when the container is started.

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(120) NOT NULL,
    age INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO users (username, email, age)
VALUES
    ('alice', 'alice@example.com', 25),
    ('bob', 'bob@example.com', 30),
    ('charlie', 'charlie@example.com', 29);

🔌 SQLAlchemy Engine (database.py)
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

DATABASE_URL = "mariadb+pymysql://app_user:app_pass@localhost:3306/app_db"

engine = create_engine(
    DATABASE_URL,
    pool_size=10,
    max_overflow=20,
    echo=True,
    future=True,
)

SessionLocal = sessionmaker(bind=engine, autoflush=False, autocommit=False, future=True)

🧱 SQLAlchemy Core Table Definition (tables.py)
from sqlalchemy import Table, Column, Integer, String, TIMESTAMP, MetaData

metadata = MetaData()

users_table = Table(
    "users",
    metadata,
    Column("id", Integer, primary_key=True),
    Column("username", String(50), nullable=False, unique=True),
    Column("email", String(120), nullable=False),
    Column("age", Integer),
    Column("created_at", TIMESTAMP)
)

🧩 SQLAlchemy ORM Model (models.py)
from sqlalchemy.orm import declarative_base, Mapped, mapped_column
from sqlalchemy import Integer, String, TIMESTAMP

Base = declarative_base()

class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(Integer, primary_key=True)
    username: Mapped[str] = mapped_column(String(50), unique=True, nullable=False)
    email: Mapped[str] = mapped_column(String(120), nullable=False)
    age: Mapped[int | None] = mapped_column(Integer)
    created_at: Mapped[str | None] = mapped_column(TIMESTAMP)

    def __repr__(self):
        return f"User(id={self.id}, username={self.username})"

🔧 CRUD Operations (crud.py)
✔ Get all users
def get_all_users():
    stmt = select(User)
    return session.execute(stmt).scalars().all()

✔ Get user by username
def get_user_by_username(username: str):
    stmt = select(User).where(User.username == username)
    return session.execute(stmt).scalar_one_or_none()

✔ Insert new user (with error handling)
def insert_user(username: str, email: str, age: int):
    try:
        new_user = User(username=username, email=email, age=age)
        session.add(new_user)
        session.commit()
        session.refresh(new_user)
    except IntegrityError:
        session.rollback()

✔ Update user
def update_user(username: str, new_email=None, new_age=None):
    stmt = update(User).where(User.username == username)
    session.execute(stmt)
    session.commit()

▶️ Running the Project

Run the test file:

python src/main.py


Expected output:

All users...
User(id=1, username='alice')
...
Inserted user: User(id=4, username='david')
Updated user: User(id=2, username='bob')

📤 Pushing to GitHub
git add .
git commit -m "Assignment 5 completed"
git push -u origin main

✔ Assignment Completed

This project demonstrates:

Docker container orchestration

MariaDB SQL setup and initialization

SQLAlchemy Core + ORM

CRUD operations

Connection pooling and engine configuration
