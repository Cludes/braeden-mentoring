---
layout: page
title: "Python Basics: Your First Real Programming Language"
week_number: "7"
release_date: "2026-06-02"
phase_label: "Phase 2"
phase_name: "Python, Linux & Git"
phase_color: "#7209b7"
permalink: /week-07/
---

> Week 7. This week: Python.
> 
> You have been writing JavaScript in a browser. Python runs in the terminal and on servers. Nearly every cybersecurity tool is written in Python. You will use it constantly.
> 
> To run Python: open a terminal and type python3 script.py. If Python is not installed, run: sudo apt install python3.
> 
> Python uses indentation (spaces) instead of curly braces to define code blocks. This is the biggest source of beginner errors -- be consistent with 4 spaces.

---

# Variables, Types & Functions

## Variables and Types

Python is dynamically typed. The type is attached to the value, not the variable name.

```python
name = "alice"          # str
age = 30                # int
score = 98.5            # float
active = True           # bool
nothing = None          # NoneType
```

**Type conversion** (explicit only -- Python will not silently coerce):

```python
int("42")       # 42
str(42)         # "42"
float("3.14")   # 3.14
bool(0)         # False  (0, "", [], {}, None are all falsy)
bool("hello")   # True
```

**Common mistake:** treating a string number as a number.

```python
user_input = input("Enter age: ")  # always returns str
age = int(user_input)              # convert before arithmetic
```

---

## Functions

A function takes inputs, does work, returns output. Keep functions short (< 20 lines is a good target) and name them with verbs.

```python
def greet(name: str) -> str:
    return f"Hello, {name}"

print(greet("alice"))  # Hello, alice
```

**Default arguments:**

```python
def connect(host: str, port: int = 443) -> str:
    return f"{host}:{port}"

connect("example.com")        # example.com:443
connect("example.com", 8080)  # example.com:8080
```

**Multiple return values** (returns a tuple):

```python
def divide(a: int, b: int) -> tuple[float, str]:
    if b == 0:
        return 0.0, "division by zero"
    return a / b, "ok"

result, status = divide(10, 2)
```

**Keyword arguments** -- useful when a function has many parameters:

```python
def create_user(name: str, role: str = "viewer", active: bool = True):
    pass

create_user("alice", active=False, role="admin")
```

---

## Scope

Variables defined inside a function are local to that function.

```python
x = 10          # global

def add_five():
    x = 99      # local -- does NOT change the global x
    return x

add_five()      # 99
print(x)        # 10
```

Use `return` to pass data out. Avoid `global` -- it makes code hard to follow.

---

## Type Hints

Type hints do not enforce types at runtime but improve readability and catch bugs with tools like `mypy`.

```python
def hash_password(password: str, rounds: int = 12) -> str:
    ...
```

---

## Checklist

- [ ] Can name variables descriptively (not `x`, `tmp`, `data`)
- [ ] Can convert between str, int, float, bool
- [ ] Can write a function with parameters and a return value
- [ ] Understand the difference between local and global scope
- [ ] Know that `None` is not 0 or `False` -- it means "no value"

## Next

[Data Structures](02_data-structures.md)

---

## Activity: Caesar Cipher

1. Write a Python script that encodes a message using a Caesar cipher (shift each letter by N positions in the alphabet)
1. The script should have two functions: encode(message, shift) and decode(message, shift)
1. decode() should work by calling encode() with a negative shift, not by writing separate logic
1. Non-letter characters (spaces, punctuation) should be passed through unchanged
1. Test it: encode('Hello World', 3) should return 'Khoor Zruog'
1. Print both the encoded and decoded versions to the terminal
1. You know you are done when encode and decode are inverses of each other: decode(encode(msg, 3), 3) == msg

---

## Check-in Questions

Write your answers down before your next session.

1. What is the difference between int and str in Python? Why does int('42') work but int('hello') crash?
2. What does a function return if it has no return statement?
3. Why does Python use indentation instead of curly braces, and what error do you get if your indentation is inconsistent?
4. What is the difference between a positional argument and a default argument in a function?
