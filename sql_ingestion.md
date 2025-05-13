# SQL Injection Prevention Guide

## Scenario
Imagine you have a simple database with a users table, and you want to allow users to log in using their username.

You take input from a user, e.g., username in a login form, and use that input to query the database for a matching user.

### Database Table: users
| id | username |
|----|----------|
| 1  | john     |
| 2  | admin    |
| 3  | jane     |

Now let's explore two methods:

## 1. Unsafe Method: Vulnerable to SQL Injection
### Code (Unsafe Method)
```python
username = input("Enter username: ")
query = f"SELECT * FROM users WHERE username = '{username}'"  # **Vulnerable code**
print(f"Executing query: {query}")

# In a real scenario, this query would be executed against the database (e.g., with sqlite3 or other DB connectors)
```

### What Happens with Unsafe Method?
Let's imagine the user enters ' OR '1'='1 in the input field.

```python
username = "' OR '1'='1"
```

So, the query that gets generated would be:

```sql
SELECT * FROM users WHERE username = '' OR '1'='1'
```

### Why This is Dangerous:
The OR '1'='1' is a valid SQL statement that always evaluates to true.

This means the query will return all rows in the users table because the condition 1=1 is always true.

The attacker could now bypass login or steal data by manipulating the query.

### Query Output:
```sql
SELECT * FROM users WHERE username = '' OR '1'='1'
```
This will return all users, even though the attacker was just trying to login as one user.

## 2. Safe Method: Using SQLAlchemy (Prevents SQL Injection)
### Code (Safe Method with SQLAlchemy)
```python
from sqlalchemy.orm import sessionmaker
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base

# Database setup
DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(DATABASE_URL, connect_args={"check_same_thread": False})
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

# Define User model
class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    username = Column(String, index=True)

# Initialize session
db = SessionLocal()

# Query with SQLAlchemy
username = input("Enter username: ")
user = db.query(User).filter(User.username == username).first()  # **Safe code using SQLAlchemy**

if user:
    print(f"Welcome {user.username}!")
else:
    print("Invalid username")
```

### How SQLAlchemy Protects Against Injection:
Parameterized Queries: SQLAlchemy does not directly concatenate the input into the query string. Instead, it generates the SQL query in a way that treats the input as data, not as executable SQL code.

The SQLAlchemy query that gets executed looks like:

```sql
SELECT * FROM users WHERE username = :username
```

Here, the :username is a placeholder, and SQLAlchemy safely binds the value of username as a parameter.

### What Happens When User Inputs ' OR '1'='1?
Let's assume the user input is:

```python
username = "' OR '1'='1"
```

SQLAlchemy will not directly insert this into the SQL string. Instead, it will escape this input and safely bind it to the query, resulting in something like:

```sql
SELECT * FROM users WHERE username = :username
```

Where :username is treated as a parameter, and the value of username (in this case, ' OR '1'='1) is passed safely to the database.

The final SQL query executed looks like:

```sql
SELECT * FROM users WHERE username = '' OR '1'='1'
```

But here's the key difference:

- The ' OR '1'='1 part is NOT considered as executable SQL.
- It's treated as data, not part of the SQL command.

### Result:
- No injection occurs.
- The query will return no users (if no user has ' OR '1'='1 as their username).
- It prevents bypassing or retrieving extra data.

### How Does SQLAlchemy Actually Prevent SQL Injection?
- SQLAlchemy treats the input as data and not as part of the SQL query.
- It binds parameters to the query, so user input is not executed as SQL code.
- No matter what the user enters, the query is safe because the database engine knows that user input is just data and not executable code.

### Example of SQLAlchemy Internals:
Internally, SQLAlchemy might execute something like:

```sql
SELECT * FROM users WHERE username = ?
```

Where ? is a placeholder, and the value of username is safely inserted. The database treats the input as data, not part of the query structure.

## Summary of Differences
| Feature | Unsafe Method | Safe Method (SQLAlchemy) |
|---------|---------------|--------------------------|
| Input Handling | Directly concatenates user input into SQL string. | Uses parameterized queries, separating input from SQL. |
| SQL Injection Risk | High: Attackers can inject SQL via user input. | Low: User input is safely bound and escaped. |
| Example Malicious Input | ' OR '1'='1 | Treated as a string, not executable SQL. |
| Result | Returns all users or bypasses authentication. | Returns the correct result without data exposure. |

## Key Takeaway
Unsafe queries are vulnerable to SQL injection because they directly embed user input into the SQL string, allowing attackers to alter the query.

SQLAlchemy solves this problem by parameterizing queries, ensuring user input is treated as data and not executable code. It automatically escapes and binds the parameters safely, which prevents SQL injection.
