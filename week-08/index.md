---
layout: page
title: "Python Data Structures: Lists, Dicts, and Sets"
week_number: "8"
release_date: "2026-06-09"
phase_label: "Phase 2"
phase_name: "Python, Linux & Git"
phase_color: "#7209b7"
permalink: /week-08/
---

> Week 8. This week: data structures in Python.
> 
> Lists, dictionaries, and sets are the building blocks of almost every Python program. In cybersecurity you will use them to store IP addresses, parse log files, track seen values, and build lookup tables.
> 
> After this week you will have the tools to write tools that process real data.

---

# Data Structures

Pick the right container for the job. Wrong choice = slow code or unnecessary complexity.

## List

Ordered, mutable, allows duplicates. Use when order matters or you need to iterate.

```python
ports = [80, 443, 8080]
ports.append(22)
ports.remove(8080)
ports[0]         # 80  (zero-indexed)
ports[-1]        # 22  (last item)
ports[1:3]       # [443, 22]  (slice)

for port in ports:
    print(port)
```

**Common operations:**

```python
sorted([3, 1, 2])          # [1, 2, 3]  -- new list
[3, 1, 2].sort()           # sorts in place, returns None
len(ports)                 # count
port in ports              # membership check (O(n))
[x * 2 for x in ports]    # list comprehension
```

---

## Dict

Key-value pairs, unordered (insertion order preserved in Python 3.7+). Use when you need fast lookup by name.

```python
user = {
    "name": "alice",
    "role": "admin",
    "active": True,
}

user["name"]               # "alice"
user.get("email")          # None  (safe -- no KeyError)
user.get("email", "n/a")   # "n/a"  (default)
user["email"] = "a@b.com"  # add / update
del user["active"]         # remove

for key, value in user.items():
    print(f"{key}: {value}")
```

**Membership check is O(1)** -- prefer dict/set over list for lookup:

```python
allowed_roles = {"admin", "moderator", "editor"}
"admin" in allowed_roles   # True, O(1)
```

---

## Set

Unordered, no duplicates, O(1) membership test. Use for deduplication and set operations.

```python
seen_ips = set()
seen_ips.add("10.0.0.1")
seen_ips.add("10.0.0.1")   # duplicate -- ignored
len(seen_ips)               # 1

a = {1, 2, 3}
b = {2, 3, 4}
a & b    # {2, 3}   intersection
a | b    # {1, 2, 3, 4}  union
a - b    # {1}  difference
```

---

## Tuple

Ordered, immutable. Use for data that should not change (coordinates, config values, return values).

```python
point = (10, 20)
x, y = point               # unpacking

HOST, PORT = "localhost", 8080
```

---

## When to Use What

| Need | Use |
|------|-----|
| Ordered collection, may change | `list` |
| Fast key-based lookup | `dict` |
| Membership test, deduplication | `set` |
| Fixed grouping of values | `tuple` |
| Large numeric data | `array` or `numpy` |

---

## Checklist

- [ ] Can create, index, slice, and iterate a list
- [ ] Can get, set, and delete dict keys safely (using `.get()`)
- [ ] Know when a set is faster than a list for membership checks
- [ ] Can unpack a tuple
- [ ] Can write a list comprehension

## Next

[Control Flow & Error Handling](03_control-flow-errors.md)

---

## Activity: Password Strength Analyzer

1. Write a Python function analyze_password(password) that returns a score from 0-100 and a list of feedback messages
1. Score components: +20 if length >= 8, +10 more if length >= 12, +20 if contains uppercase, +20 if contains lowercase, +20 if contains a digit, +10 if contains a special character (!@#$%^&*)
1. For each missing component, add a specific feedback message: 'Add uppercase letters', 'Add a number', etc
1. Print the score and each feedback message
1. Test with: 'password', 'Password1', 'P@ssw0rd!', 'xK9#mQ2!vL'
1. You know you are done when each test password gets a sensible score and the feedback is accurate

---

## Check-in Questions

Write your answers down before your next session.

1. What is the difference between a list and a set? When would you use a set instead of a list?
2. How do you check if a key exists in a dictionary without getting a KeyError?
3. What does list.sort() do differently from sorted(list)?
4. What is a list comprehension? Write one that creates a list of squares from 1 to 10.
