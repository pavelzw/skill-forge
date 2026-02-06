---
name: sqlalchemy
description: Python SQL toolkit and Object Relational Mapper (ORM). Use when working with databases in Python, defining models, building queries, managing sessions, or interacting with SQL databases using Python objects.
license: MIT
---

# SQLAlchemy Skill

SQLAlchemy is the Python SQL toolkit and Object Relational Mapper that provides the full power and flexibility of SQL. It consists of two main components: **Core** (SQL Expression Language) and **ORM** (Object Relational Mapper).

## When to Use This Skill

Use SQLAlchemy when:
- Working with relational databases in Python
- Defining database models as Python classes
- Building SQL queries programmatically
- Managing database transactions and sessions
- Mapping Python objects to database tables
- Need database-agnostic code that works across PostgreSQL, MySQL, SQLite, etc.

## Installation

```bash
pip install sqlalchemy

# For async support
pip install sqlalchemy[asyncio]
```

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    SQLAlchemy ORM                           │
│  (Declarative Mapping, Session, Relationships, Unit of Work)│
├─────────────────────────────────────────────────────────────┤
│                   SQLAlchemy Core                           │
│     (SQL Expression Language, Engine, Connection Pool)      │
├─────────────────────────────────────────────────────────────┤
│                        DBAPI                                │
│         (psycopg2, pymysql, sqlite3, etc.)                  │
└─────────────────────────────────────────────────────────────┘
```

## Engine and Connection

The Engine is the starting point for SQLAlchemy applications:

```python
from sqlalchemy import create_engine

# SQLite (in-memory)
engine = create_engine("sqlite://", echo=True)

# SQLite (file-based)
engine = create_engine("sqlite:///mydatabase.db")

# PostgreSQL
engine = create_engine("postgresql+psycopg2://user:password@localhost/dbname")

# MySQL
engine = create_engine("mysql+pymysql://user:password@localhost/dbname")

# Connection pool settings
engine = create_engine(
    "postgresql+psycopg2://user:password@localhost/dbname",
    pool_size=5,           # Number of connections to keep open
    max_overflow=10,       # Additional connections allowed
    pool_timeout=30,       # Seconds to wait for connection
    pool_recycle=1800,     # Recycle connections after N seconds
)
```

### Using Connections Directly (Core)

```python
from sqlalchemy import text

with engine.connect() as conn:
    result = conn.execute(text("SELECT * FROM users WHERE id = :id"), {"id": 1})
    for row in result:
        print(row)

    # For write operations, commit explicitly
    conn.execute(text("INSERT INTO users (name) VALUES (:name)"), {"name": "Alice"})
    conn.commit()
```

## ORM Declarative Mapping


```python
from datetime import datetime
from typing import List, Optional

from sqlalchemy import ForeignKey, String, func
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship


class Base(DeclarativeBase):
    pass


class User(Base):
    __tablename__ = "user_account"

    # Primary key with auto-increment
    id: Mapped[int] = mapped_column(primary_key=True)

    # Required string column with max length
    name: Mapped[str] = mapped_column(String(30))

    # Optional column (nullable)
    fullname: Mapped[Optional[str]]

    # Column with default value
    created_at: Mapped[datetime] = mapped_column(default=func.now())

    # One-to-many relationship
    addresses: Mapped[List["Address"]] = relationship(
        back_populates="user",
        cascade="all, delete-orphan"
    )

    def __repr__(self) -> str:
        return f"User(id={self.id!r}, name={self.name!r})"


class Address(Base):
    __tablename__ = "address"

    id: Mapped[int] = mapped_column(primary_key=True)
    email_address: Mapped[str]
    user_id: Mapped[int] = mapped_column(ForeignKey("user_account.id"))

    # Many-to-one relationship (back reference)
    user: Mapped["User"] = relationship(back_populates="addresses")

    def __repr__(self) -> str:
        return f"Address(id={self.id!r}, email_address={self.email_address!r})"
```

### Type Annotation Guide

| Python Type | SQL Type | Nullable |
|-------------|----------|----------|
| `Mapped[int]` | INTEGER | NOT NULL |
| `Mapped[Optional[int]]` | INTEGER | NULL |
| `Mapped[str]` | VARCHAR | NOT NULL |
| `Mapped[Optional[str]]` | VARCHAR | NULL |
| `Mapped[bool]` | BOOLEAN | NOT NULL |
| `Mapped[datetime]` | DATETIME | NOT NULL |
| `Mapped[float]` | FLOAT | NOT NULL |
| `Mapped[bytes]` | BLOB/BYTEA | NOT NULL |

### Creating Tables

```python
# Create all tables defined in Base.metadata
Base.metadata.create_all(engine)

# Drop all tables
Base.metadata.drop_all(engine)
```

## Session and CRUD Operations

### Session Basics

```python
from sqlalchemy.orm import Session, sessionmaker

# Option 1: Direct Session usage
with Session(engine) as session:
    # ... operations
    session.commit()

# Option 2: Using sessionmaker (recommended for applications)
SessionFactory = sessionmaker(bind=engine)

with SessionFactory() as session:
    # ... operations
    session.commit()

# Option 3: With explicit begin/commit/rollback
with Session(engine) as session:
    with session.begin():
        # Automatically commits on success, rolls back on exception
        session.add(some_object)
```

### Create (INSERT)

```python
with Session(engine) as session:
    # Create single object
    user = User(name="alice", fullname="Alice Smith")
    session.add(user)

    # Create with related objects
    user_with_addresses = User(
        name="bob",
        fullname="Bob Jones",
        addresses=[
            Address(email_address="bob@example.com"),
            Address(email_address="bob@work.com"),
        ]
    )
    session.add(user_with_addresses)

    # Add multiple objects
    session.add_all([
        User(name="carol"),
        User(name="dave"),
    ])

    session.commit()
```

### Read (SELECT)

```python
from sqlalchemy import select

with Session(engine) as session:
    # Get by primary key
    user = session.get(User, 1)

    # Select all
    stmt = select(User)
    users = session.scalars(stmt).all()

    # Select with filter
    stmt = select(User).where(User.name == "alice")
    alice = session.scalars(stmt).first()

    # Select with multiple conditions
    stmt = select(User).where(
        User.name.like("a%"),
        User.id > 5
    )

    # Select specific columns
    stmt = select(User.name, User.fullname)
    rows = session.execute(stmt).all()
    for name, fullname in rows:
        print(f"{name}: {fullname}")

    # Order by
    stmt = select(User).order_by(User.name.desc())

    # Limit and offset
    stmt = select(User).limit(10).offset(20)

    # Count
    from sqlalchemy import func
    stmt = select(func.count()).select_from(User)
    count = session.scalar(stmt)
```

### Update

```python
with Session(engine) as session:
    # Update via ORM (load then modify)
    user = session.get(User, 1)
    user.fullname = "Alice Johnson"
    session.commit()

    # Bulk update
    from sqlalchemy import update
    stmt = update(User).where(User.name == "alice").values(fullname="Alice Updated")
    session.execute(stmt)
    session.commit()
```

### Delete

```python
with Session(engine) as session:
    # Delete via ORM
    user = session.get(User, 1)
    session.delete(user)
    session.commit()

    # Bulk delete
    from sqlalchemy import delete
    stmt = delete(User).where(User.name == "alice")
    session.execute(stmt)
    session.commit()
```

## Relationships

### One-to-Many / Many-to-One

```python
class Parent(Base):
    __tablename__ = "parent"

    id: Mapped[int] = mapped_column(primary_key=True)
    children: Mapped[List["Child"]] = relationship(back_populates="parent")


class Child(Base):
    __tablename__ = "child"

    id: Mapped[int] = mapped_column(primary_key=True)
    parent_id: Mapped[int] = mapped_column(ForeignKey("parent.id"))
    parent: Mapped["Parent"] = relationship(back_populates="children")
```

### One-to-One

```python
class User(Base):
    __tablename__ = "user"

    id: Mapped[int] = mapped_column(primary_key=True)
    profile: Mapped["Profile"] = relationship(back_populates="user", uselist=False)


class Profile(Base):
    __tablename__ = "profile"

    id: Mapped[int] = mapped_column(primary_key=True)
    user_id: Mapped[int] = mapped_column(ForeignKey("user.id"), unique=True)
    user: Mapped["User"] = relationship(back_populates="profile")
```

### Many-to-Many

```python
from sqlalchemy import Column, Table

# Association table (no ORM class needed)
association_table = Table(
    "association",
    Base.metadata,
    Column("left_id", ForeignKey("left.id"), primary_key=True),
    Column("right_id", ForeignKey("right.id"), primary_key=True),
)


class Left(Base):
    __tablename__ = "left"

    id: Mapped[int] = mapped_column(primary_key=True)
    rights: Mapped[List["Right"]] = relationship(
        secondary=association_table,
        back_populates="lefts"
    )


class Right(Base):
    __tablename__ = "right"

    id: Mapped[int] = mapped_column(primary_key=True)
    lefts: Mapped[List["Left"]] = relationship(
        secondary=association_table,
        back_populates="rights"
    )
```

### Association Object (Many-to-Many with extra data)

```python
class Association(Base):
    __tablename__ = "association"

    left_id: Mapped[int] = mapped_column(ForeignKey("left.id"), primary_key=True)
    right_id: Mapped[int] = mapped_column(ForeignKey("right.id"), primary_key=True)
    extra_data: Mapped[Optional[str]]

    left: Mapped["Left"] = relationship(back_populates="right_associations")
    right: Mapped["Right"] = relationship(back_populates="left_associations")


class Left(Base):
    __tablename__ = "left"

    id: Mapped[int] = mapped_column(primary_key=True)
    right_associations: Mapped[List["Association"]] = relationship(back_populates="left")


class Right(Base):
    __tablename__ = "right"

    id: Mapped[int] = mapped_column(primary_key=True)
    left_associations: Mapped[List["Association"]] = relationship(back_populates="right")
```

## Loading Strategies

### Lazy Loading (Default)

```python
# Lazy loading - queries database when attribute is accessed
user = session.get(User, 1)
# SELECT ... FROM user WHERE id = 1

addresses = user.addresses  # N+1 query problem!
# SELECT ... FROM address WHERE user_id = 1
```

### Eager Loading with joinedload

```python
from sqlalchemy.orm import joinedload

# Load user and addresses in single query using JOIN
stmt = select(User).options(joinedload(User.addresses)).where(User.id == 1)
user = session.scalars(stmt).unique().first()
# SELECT ... FROM user LEFT OUTER JOIN address ON ...
```

### Eager Loading with selectinload (Recommended)

```python
from sqlalchemy.orm import selectinload

# Load users, then load all addresses with IN clause
stmt = select(User).options(selectinload(User.addresses))
users = session.scalars(stmt).all()
# SELECT ... FROM user
# SELECT ... FROM address WHERE user_id IN (1, 2, 3, ...)
```

### Raise on Lazy Load (Prevent N+1)

```python
from sqlalchemy.orm import raiseload

# Raise error if lazy loading is attempted
stmt = select(User).options(raiseload(User.addresses))
user = session.scalars(stmt).first()
user.addresses  # Raises InvalidRequestError
```

### Setting Default Loading Strategy

```python
class User(Base):
    __tablename__ = "user"

    id: Mapped[int] = mapped_column(primary_key=True)
    # Always eager load addresses
    addresses: Mapped[List["Address"]] = relationship(lazy="selection")
```

## Joins and Complex Queries

```python
from sqlalchemy import select, and_, or_, func

# Explicit JOIN
stmt = (
    select(User, Address)
    .join(Address, User.id == Address.user_id)
    .where(User.name == "alice")
)

# JOIN using relationship
stmt = (
    select(Address)
    .join(Address.user)
    .where(User.name == "alice")
)

# LEFT OUTER JOIN
stmt = select(User).outerjoin(User.addresses)

# Subquery
subq = select(func.count(Address.id)).where(Address.user_id == User.id).scalar_subquery()
stmt = select(User.name, subq.label("address_count"))

# GROUP BY and HAVING
stmt = (
    select(User.name, func.count(Address.id).label("count"))
    .join(User.addresses)
    .group_by(User.name)
    .having(func.count(Address.id) > 1)
)

# UNION
stmt1 = select(User.name).where(User.id < 5)
stmt2 = select(User.name).where(User.id > 10)
stmt = stmt1.union(stmt2)

# EXISTS
from sqlalchemy import exists
subq = select(Address).where(Address.user_id == User.id).exists()
stmt = select(User).where(subq)

# IN with subquery
subq = select(Address.user_id).where(Address.email_address.like("%@example.com"))
stmt = select(User).where(User.id.in_(subq))
```

## Column Operators

```python
# Comparison
User.name == "alice"
User.id != 5
User.id > 10
User.id >= 10
User.id < 10
User.id <= 10
User.id.between(5, 10)

# NULL checks
User.fullname.is_(None)
User.fullname.is_not(None)

# String operations
User.name.like("a%")       # SQL LIKE
User.name.ilike("a%")      # Case-insensitive LIKE
User.name.startswith("a")
User.name.endswith("z")
User.name.contains("bc")

# IN
User.id.in_([1, 2, 3])
User.id.not_in([1, 2, 3])

# Logical operators
and_(User.name == "alice", User.id > 5)
or_(User.name == "alice", User.name == "bob")
~(User.name == "alice")  # NOT
```

## Transactions

```python
# Automatic transaction management with context manager
with Session(engine) as session:
    with session.begin():
        session.add(User(name="alice"))
        session.add(User(name="bob"))
    # Commits automatically, rolls back on exception

# Manual transaction control
session = Session(engine)
try:
    session.add(User(name="alice"))
    session.commit()
except Exception:
    session.rollback()
    raise
finally:
    session.close()

# Nested transactions (savepoints)
with Session(engine) as session:
    session.add(User(name="alice"))

    with session.begin_nested():  # Creates SAVEPOINT
        session.add(User(name="bob"))
        # Can rollback just this savepoint

    session.commit()
```

## Async Support

```python
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine, async_sessionmaker

# Create async engine
engine = create_async_engine("postgresql+asyncpg://user:pass@localhost/db")

# Create async session factory
AsyncSessionFactory = async_sessionmaker(engine, expire_on_commit=False)

async def get_user(user_id: int) -> User | None:
    async with AsyncSessionFactory() as session:
        stmt = select(User).where(User.id == user_id)
        result = await session.execute(stmt)
        return result.scalar_one_or_none()

async def create_user(name: str) -> User:
    async with AsyncSessionFactory() as session:
        user = User(name=name)
        session.add(user)
        await session.commit()
        await session.refresh(user)
        return user

# Important: Use selectinload for eager loading in async
async def get_user_with_addresses(user_id: int) -> User | None:
    async with AsyncSessionFactory() as session:
        stmt = (
            select(User)
            .options(selectinload(User.addresses))
            .where(User.id == user_id)
        )
        result = await session.execute(stmt)
        return result.scalar_one_or_none()
```

## Common Patterns

### Repository Pattern

```python
from typing import Generic, TypeVar
from sqlalchemy import select
from sqlalchemy.orm import Session

T = TypeVar("T", bound=Base)


class Repository(Generic[T]):
    def __init__(self, session: Session, model: type[T]):
        self.session = session
        self.model = model

    def get(self, id: int) -> T | None:
        return self.session.get(self.model, id)

    def get_all(self) -> list[T]:
        return list(self.session.scalars(select(self.model)).all())

    def add(self, entity: T) -> T:
        self.session.add(entity)
        return entity

    def delete(self, entity: T) -> None:
        self.session.delete(entity)


# Usage
with Session(engine) as session:
    user_repo = Repository(session, User)
    user = user_repo.get(1)
    all_users = user_repo.get_all()
```

### Soft Delete

```python
from datetime import datetime


class SoftDeleteMixin:
    deleted_at: Mapped[Optional[datetime]] = mapped_column(default=None)

    @property
    def is_deleted(self) -> bool:
        return self.deleted_at is not None

    def soft_delete(self) -> None:
        self.deleted_at = datetime.utcnow()


class User(SoftDeleteMixin, Base):
    __tablename__ = "user"
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]


# Query only non-deleted
stmt = select(User).where(User.deleted_at.is_(None))
```

### Timestamp Mixin

```python
from datetime import datetime
from sqlalchemy import func


class TimestampMixin:
    created_at: Mapped[datetime] = mapped_column(default=func.now())
    updated_at: Mapped[datetime] = mapped_column(
        default=func.now(),
        onupdate=func.now()
    )


class User(TimestampMixin, Base):
    __tablename__ = "user"
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
```

### Database Reflection

```python
from sqlalchemy import MetaData, Table

# Reflect existing database schema
metadata = MetaData()
metadata.reflect(bind=engine)

# Access reflected table
users_table = metadata.tables["users"]

# Query reflected table
with engine.connect() as conn:
    result = conn.execute(users_table.select())
```

## Best Practices

1. **Always use context managers** for Session to ensure proper cleanup
2. **Prefer `selectinload`** over `joinedload` for collection relationships
3. **Use `raiseload`** during development to catch N+1 query problems
4. **Keep transactions short** - commit as soon as the logical unit of work is done
5. **Use `expire_on_commit=False`** in async contexts and when passing objects outside session scope
6. **Define `__repr__`** methods on models for easier debugging
7. **Use type annotations** with `Mapped` for better IDE support and type checking
8. **Index foreign keys** and columns used in WHERE clauses
9. **Use bulk operations** (`insert().values([...])`) for large datasets
10. **Handle sessions per-request** in web applications, not globally

## Common Column Types

```python
from sqlalchemy import (
    String, Text, Integer, BigInteger, SmallInteger,
    Float, Numeric, Boolean, Date, DateTime, Time,
    LargeBinary, JSON, Enum, UUID
)
from sqlalchemy.dialects.postgresql import ARRAY, JSONB

# Examples
name: Mapped[str] = mapped_column(String(100))
description: Mapped[str] = mapped_column(Text)
price: Mapped[float] = mapped_column(Numeric(10, 2))
data: Mapped[dict] = mapped_column(JSON)
tags: Mapped[list] = mapped_column(ARRAY(String))  # PostgreSQL only
```

## Constraints and Indexes

```python
from sqlalchemy import CheckConstraint, UniqueConstraint, Index


class Product(Base):
    __tablename__ = "product"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(unique=True)
    price: Mapped[float]
    category: Mapped[str]

    __table_args__ = (
        CheckConstraint("price > 0", name="positive_price"),
        UniqueConstraint("name", "category", name="unique_name_category"),
        Index("idx_category", "category"),
        Index("idx_name_price", "name", "price"),
    )
```
