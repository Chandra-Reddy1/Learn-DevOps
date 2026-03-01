# 🐍 Python — Zero to Hero Complete Guide
> **Beginner → Intermediate → Advanced · Real-Time Scenarios · Production Patterns**

---

## 📌 Table of Contents

1. [What is Python & Why Learn It?](#1-what-is-python--why-learn-it)
2. [Setup & First Program](#2-setup--first-program)
3. [Variables & Data Types](#3-variables--data-types)
4. [Strings — Deep Dive](#4-strings--deep-dive)
5. [Numbers & Math](#5-numbers--math)
6. [Booleans & Comparisons](#6-booleans--comparisons)
7. [Control Flow — if / elif / else](#7-control-flow--if--elif--else)
8. [Loops — for & while](#8-loops--for--while)
9. [Functions](#9-functions)
10. [Lists](#10-lists)
11. [Tuples](#11-tuples)
12. [Dictionaries](#12-dictionaries)
13. [Sets](#13-sets)
14. [File Handling](#14-file-handling)
15. [Error Handling — try / except](#15-error-handling--try--except)
16. [Object-Oriented Programming (OOP)](#16-object-oriented-programming-oop)
17. [Modules & Packages](#17-modules--packages)
18. [List Comprehensions & Generators](#18-list-comprehensions--generators)
19. [Decorators](#19-decorators)
20. [Lambda, Map, Filter, Reduce](#20-lambda-map-filter-reduce)
21. [Regular Expressions](#21-regular-expressions)
22. [Working with APIs (requests)](#22-working-with-apis-requests)
23. [Working with JSON & CSV](#23-working-with-json--csv)
24. [Database with SQLite](#24-database-with-sqlite)
25. [Virtual Environments & pip](#25-virtual-environments--pip)
26. [Real-World Project — Build a REST API](#26-real-world-project--build-a-rest-api)
27. [Python Best Practices & Tips](#27-python-best-practices--tips)
28. [Learning Roadmap & Next Steps](#28-learning-roadmap--next-steps)

---

## 1. What is Python & Why Learn It?

Python is a **high-level, interpreted, general-purpose** programming language created by Guido van Rossum in 1991. Its design philosophy emphasizes code readability — it looks almost like plain English.

### Why Python?

| Reason | Detail |
|---|---|
| **Easy to learn** | Minimal syntax, no semicolons, reads like English |
| **Versatile** | Web dev, data science, AI/ML, automation, scripting, DevOps |
| **Huge ecosystem** | 400,000+ packages on PyPI (pip) |
| **High demand** | Top 3 most-used language worldwide, great salaries |
| **Community** | Massive community, tons of free resources |

### Where is Python used in real life?

```
🌐 Web Development     → Django (Instagram), Flask (Pinterest), FastAPI
📊 Data Science        → Pandas, NumPy, Matplotlib (used at Netflix, Spotify)
🤖 AI & Machine Learning → TensorFlow, PyTorch (used at Google, OpenAI)
⚙️  Automation / DevOps  → Ansible, scripts, AWS Lambda functions
🔬 Scientific Research  → NASA, CERN use Python for data analysis
💰 Finance             → Algorithmic trading, risk modeling (JPMorgan, Goldman)
```

---

## 2. Setup & First Program

### Installing Python

```bash
# Check if Python is already installed
python3 --version

# macOS (using Homebrew)
brew install python3

# Ubuntu/Debian Linux
sudo apt update && sudo apt install python3 python3-pip

# Windows → Download from https://python.org → check "Add to PATH" ✅
```

### Your First Program

```python
# hello.py
print("Hello, World!")
print("I am learning Python!")
```

**Run it:**
```bash
python3 hello.py
# Output:
# Hello, World!
# I am learning Python!
```

### 🟠 Real-World Scenario — Automated Morning Report
```python
# morning_report.py
# Used by operations teams to print a daily status summary

import datetime

today = datetime.date.today()
print(f"=== Daily Report: {today} ===")
print("Server Status: ✅ All systems operational")
print("Deployments today: 3")
print("Alerts: 0 critical, 2 warnings")
print("Report generated automatically by Python.")
```

---

## 3. Variables & Data Types

A **variable** is a named container that holds a value. Python figures out the type automatically (dynamic typing).

### Basic Data Types

```python
# Integer — whole numbers
age = 25
server_count = 100

# Float — decimal numbers
price = 19.99
cpu_usage = 73.5

# String — text (single or double quotes)
name = "Alice"
city = 'Hyderabad'

# Boolean — True or False (capital T/F)
is_active = True
is_admin = False

# NoneType — absence of a value (like null in other languages)
result = None
```

### Checking Types

```python
print(type(age))          # <class 'int'>
print(type(price))        # <class 'float'>
print(type(name))         # <class 'str'>
print(type(is_active))    # <class 'bool'>
print(type(result))       # <class 'NoneType'>
```

### Multiple Assignment

```python
x, y, z = 1, 2, 3
a = b = c = 0           # all three get 0
first, *rest = [1, 2, 3, 4]  # first=1, rest=[2,3,4]
```

### Type Conversion

```python
age_str = "25"
age_int = int(age_str)       # "25" → 25
price_str = str(19.99)       # 19.99 → "19.99"
height = float("5.9")        # "5.9" → 5.9
is_valid = bool(1)           # 1 → True, 0 → False
```

### 🟠 Real-World Scenario — User Registration System
```python
# Collecting and validating user data types
username = "john_doe"
email = "john@example.com"
age = 28
is_verified = False
balance = 0.00
referral_code = None        # User hasn't added one yet

print(f"New user: {username}")
print(f"Email: {email}")
print(f"Age: {age}")
print(f"Verified: {is_verified}")
print(f"Balance: ${balance}")
print(f"Referral: {referral_code if referral_code else 'None provided'}")
```

---

## 4. Strings — Deep Dive

Strings are sequences of characters. Python has extremely powerful string tools.

### Creating & Accessing Strings

```python
message = "Hello, Python!"

# Indexing (starts at 0)
print(message[0])    # H
print(message[-1])   # ! (negative = from the end)

# Slicing [start:end:step]
print(message[0:5])  # Hello
print(message[7:])   # Python!
print(message[:5])   # Hello
print(message[::2])  # Hlo yhn  (every 2nd character)
print(message[::-1]) # !nohtyP ,olleH  (reversed!)
```

### Common String Methods

```python
text = "  Hello World  "

print(text.strip())          # "Hello World"      (remove whitespace)
print(text.lower())          # "  hello world  "
print(text.upper())          # "  HELLO WORLD  "
print(text.replace("World", "Python"))  # "  Hello Python  "
print("apple,banana,cherry".split(","))  # ['apple', 'banana', 'cherry']
print(",".join(["a", "b", "c"]))        # "a,b,c"
print("hello world".title())            # "Hello World"
print("hello".startswith("he"))         # True
print("hello".endswith("lo"))           # True
print("hello world".find("world"))      # 6 (index where found)
print("abcabc".count("a"))             # 2
print("hello".center(11, "-"))          # ---hello---
```

### f-Strings (Recommended — Python 3.6+)

```python
name = "Alice"
score = 98.5
rank = 1

# Old way (avoid)
print("Name: " + name + ", Score: " + str(score))

# f-string way (clean & fast)
print(f"Name: {name}, Score: {score}, Rank: {rank}")

# Expressions inside f-strings
print(f"Score rounded: {round(score)}")
print(f"Next rank: {rank + 1}")
print(f"Is passing: {score >= 60}")
print(f"{'PASS' if score >= 60 else 'FAIL'}")

# Number formatting
price = 1234567.89
print(f"Price: ${price:,.2f}")      # Price: $1,234,567.89
print(f"Percentage: {0.753:.1%}")   # Percentage: 75.3%
```

### Multi-line Strings

```python
email_body = """
Dear {name},

Thank you for signing up!
Your account is now active.

Best regards,
The Team
"""
print(email_body.format(name="Alice"))
```

### 🟠 Real-World Scenario — Log Parser
```python
# Parse a server log line
log_line = "2024-01-15 14:32:05 ERROR user_id=1042 action=login message=Invalid password"

parts = log_line.split(" ", 3)
date = parts[0]
time = parts[1]
level = parts[2]
details = parts[3]

# Extract specific fields
user_id = details.split("user_id=")[1].split(" ")[0]
action = details.split("action=")[1].split(" ")[0]

print(f"Date: {date}")
print(f"Time: {time}")
print(f"Level: {level}")
print(f"User ID: {user_id}")
print(f"Action: {action}")

# Output:
# Date: 2024-01-15
# Time: 14:32:05
# Level: ERROR
# User ID: 1042
# Action: login
```

---

## 5. Numbers & Math

```python
# Basic arithmetic
print(10 + 3)    # 13  addition
print(10 - 3)    # 7   subtraction
print(10 * 3)    # 30  multiplication
print(10 / 3)    # 3.3333  division (always float)
print(10 // 3)   # 3    floor division (no remainder)
print(10 % 3)    # 1    modulo (remainder)
print(10 ** 3)   # 1000 exponentiation (10 to the power of 3)

# Math module
import math

print(math.sqrt(144))     # 12.0
print(math.ceil(4.1))     # 5  (round up)
print(math.floor(4.9))    # 4  (round down)
print(math.pi)            # 3.14159...
print(math.abs(-5))       # 5  (absolute value → use built-in abs())
print(abs(-5))            # 5
print(round(3.14159, 2))  # 3.14

# Random numbers
import random

print(random.randint(1, 10))          # Random int between 1 and 10
print(random.random())                # Random float 0.0 to 1.0
print(random.choice(["a","b","c"]))   # Random item from list
items = [1, 2, 3, 4, 5]
random.shuffle(items)                 # Shuffle list in place
print(items)
```

### 🟠 Real-World Scenario — E-Commerce Discount Calculator
```python
import math

def calculate_order(items, discount_percent=0, tax_rate=0.18):
    subtotal = sum(item["price"] * item["qty"] for item in items)
    discount = subtotal * (discount_percent / 100)
    after_discount = subtotal - discount
    tax = after_discount * tax_rate
    total = after_discount + tax

    print(f"Subtotal:  ₹{subtotal:,.2f}")
    print(f"Discount:  -₹{discount:,.2f} ({discount_percent}% off)")
    print(f"Tax (GST): +₹{tax:,.2f} ({tax_rate*100:.0f}%)")
    print(f"Total:     ₹{total:,.2f}")
    return round(total, 2)

cart = [
    {"name": "Laptop", "price": 55000, "qty": 1},
    {"name": "Mouse",  "price": 1200,  "qty": 2},
    {"name": "Bag",    "price": 2500,  "qty": 1},
]

calculate_order(cart, discount_percent=10)
# Subtotal:  ₹59,900.00
# Discount:  -₹5,990.00 (10% off)
# Tax (GST): +₹9,703.80 (18%)
# Total:     ₹63,613.80
```

---

## 6. Booleans & Comparisons

```python
# Comparison operators → return True or False
print(5 > 3)     # True
print(5 < 3)     # False
print(5 >= 5)    # True
print(5 <= 4)    # False
print(5 == 5)    # True   (equality, note double ==)
print(5 != 3)    # True   (not equal)

# Logical operators
print(True and True)    # True  (both must be True)
print(True and False)   # False
print(True or False)    # True  (at least one must be True)
print(False or False)   # False
print(not True)         # False (flip it)
print(not False)        # True

# Membership operators
fruits = ["apple", "banana", "cherry"]
print("apple" in fruits)       # True
print("mango" not in fruits)   # True

# Identity operators
x = [1, 2, 3]
y = x           # y points to SAME list
z = [1, 2, 3]  # z is a DIFFERENT list with same values
print(x is y)   # True  (same object in memory)
print(x is z)   # False (different object)
print(x == z)   # True  (same values)

# Truthy and Falsy values
# Falsy: 0, 0.0, "", [], {}, (), None, False
# Truthy: everything else

print(bool(0))     # False
print(bool(""))    # False
print(bool([]))    # False
print(bool(None))  # False
print(bool(1))     # True
print(bool("hi"))  # True
print(bool([1]))   # True
```

---

## 7. Control Flow — if / elif / else

Control flow lets your program make decisions.

### Basic if/elif/else

```python
score = 85

if score >= 90:
    print("Grade: A")
elif score >= 80:
    print("Grade: B")
elif score >= 70:
    print("Grade: C")
elif score >= 60:
    print("Grade: D")
else:
    print("Grade: F")

# Output: Grade: B
```

### Ternary (one-liner if)

```python
age = 20
status = "Adult" if age >= 18 else "Minor"
print(status)  # Adult

# Nested ternary (use sparingly)
temp = 35
weather = "Hot" if temp > 30 else ("Warm" if temp > 20 else "Cold")
```

### 🟠 Real-World Scenario 1 — Bank Transaction Validator
```python
def process_transaction(balance, amount, transaction_type):
    if transaction_type == "withdraw":
        if amount <= 0:
            return "❌ Error: Amount must be positive"
        elif amount > balance:
            return f"❌ Insufficient funds. Balance: ₹{balance}, Requested: ₹{amount}"
        elif amount > 50000:
            return "⚠️  Large transaction requires manager approval"
        else:
            new_balance = balance - amount
            return f"✅ Withdrawn ₹{amount}. New balance: ₹{new_balance}"

    elif transaction_type == "deposit":
        if amount <= 0:
            return "❌ Error: Amount must be positive"
        elif amount > 200000:
            return "⚠️  Large deposit flagged for compliance review"
        else:
            new_balance = balance + amount
            return f"✅ Deposited ₹{amount}. New balance: ₹{new_balance}"
    else:
        return "❌ Unknown transaction type"

print(process_transaction(10000, 3000, "withdraw"))   # ✅ Withdrawn
print(process_transaction(10000, 15000, "withdraw"))  # ❌ Insufficient
print(process_transaction(10000, 5000, "deposit"))    # ✅ Deposited
```

### 🟠 Real-World Scenario 2 — Traffic Light System
```python
def traffic_light(color, is_emergency_vehicle=False):
    if is_emergency_vehicle:
        print("🚨 Emergency vehicle detected — all lights RED, clear path!")
        return

    if color == "green":
        print("🟢 GO — Proceed safely")
    elif color == "yellow":
        print("🟡 SLOW DOWN — Prepare to stop")
    elif color == "red":
        print("🔴 STOP — Wait for green")
    else:
        print("⚠️  Unknown signal — proceed with caution")

traffic_light("green")
traffic_light("red")
traffic_light("yellow", is_emergency_vehicle=True)
```

---

## 8. Loops — for & while

### for Loop

```python
# Loop over a range
for i in range(5):        # 0, 1, 2, 3, 4
    print(i)

for i in range(1, 6):     # 1, 2, 3, 4, 5
    print(i)

for i in range(0, 10, 2): # 0, 2, 4, 6, 8 (step 2)
    print(i)

# Loop over a list
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(fruit)

# Loop with index using enumerate
for index, fruit in enumerate(fruits):
    print(f"{index}: {fruit}")
# 0: apple
# 1: banana
# 2: cherry

# Loop over dictionary
person = {"name": "Alice", "age": 25, "city": "Delhi"}
for key, value in person.items():
    print(f"{key}: {value}")

# Nested loops
for row in range(3):
    for col in range(3):
        print(f"({row},{col})", end=" ")
    print()
```

### while Loop

```python
# Basic while
count = 0
while count < 5:
    print(count)
    count += 1   # ALWAYS increment to avoid infinite loop!

# while with break
number = 0
while True:            # Infinite loop
    number += 1
    if number == 5:
        break          # Exit the loop
    print(number)
# Prints: 1, 2, 3, 4

# continue — skip current iteration
for i in range(10):
    if i % 2 == 0:    # skip even numbers
        continue
    print(i)
# Prints: 1, 3, 5, 7, 9
```

### 🟠 Real-World Scenario 1 — ATM PIN System
```python
def atm_login():
    correct_pin = "1234"
    max_attempts = 3

    for attempt in range(1, max_attempts + 1):
        pin = input(f"Enter PIN (attempt {attempt}/{max_attempts}): ")

        if pin == correct_pin:
            print("✅ Login successful! Welcome.")
            return True
        else:
            remaining = max_attempts - attempt
            if remaining > 0:
                print(f"❌ Wrong PIN. {remaining} attempt(s) remaining.")
            else:
                print("🔒 Card blocked after 3 failed attempts. Contact your bank.")
                return False

atm_login()
```

### 🟠 Real-World Scenario 2 — Inventory Stock Alert
```python
inventory = {
    "Laptop":   {"stock": 5,  "reorder_level": 10},
    "Mouse":    {"stock": 45, "reorder_level": 20},
    "Keyboard": {"stock": 3,  "reorder_level": 15},
    "Monitor":  {"stock": 8,  "reorder_level": 10},
    "Headset":  {"stock": 0,  "reorder_level": 5},
}

print("=== Stock Alert Report ===")
out_of_stock = []
low_stock = []

for product, data in inventory.items():
    stock = data["stock"]
    reorder = data["reorder_level"]

    if stock == 0:
        out_of_stock.append(product)
        print(f"🔴 OUT OF STOCK: {product}")
    elif stock <= reorder:
        low_stock.append(product)
        print(f"🟡 LOW STOCK:    {product} ({stock} units, reorder at {reorder})")
    else:
        print(f"🟢 OK:           {product} ({stock} units)")

print(f"\nSummary: {len(out_of_stock)} out of stock, {len(low_stock)} low stock")
```

---

## 9. Functions

Functions are reusable blocks of code. **Write once, use many times.**

### Defining Functions

```python
# Basic function
def greet():
    print("Hello!")

greet()  # Call the function

# Function with parameters
def greet_user(name):
    print(f"Hello, {name}!")

greet_user("Alice")
greet_user("Bob")

# Function with return value
def add(a, b):
    return a + b

result = add(3, 5)
print(result)  # 8

# Default parameters
def greet(name, greeting="Hello"):
    print(f"{greeting}, {name}!")

greet("Alice")             # Hello, Alice!
greet("Bob", "Hi")         # Hi, Bob!
greet("Carol", greeting="Hey")  # Hey, Carol!

# Keyword arguments
def create_profile(name, age, city="Unknown"):
    return {"name": name, "age": age, "city": city}

p = create_profile(age=25, name="Alice", city="Mumbai")
print(p)

# *args — accept any number of positional arguments
def total(*numbers):
    return sum(numbers)

print(total(1, 2, 3))          # 6
print(total(10, 20, 30, 40))   # 100

# **kwargs — accept any number of keyword arguments
def show_info(**details):
    for key, value in details.items():
        print(f"{key}: {value}")

show_info(name="Alice", age=25, role="Engineer")
```

### 🟠 Real-World Scenario 1 — Email Validator
```python
import re

def validate_email(email):
    """
    Validates email format.
    Returns (is_valid, message)
    """
    if not email:
        return False, "Email cannot be empty"

    if "@" not in email:
        return False, "Email must contain @"

    parts = email.split("@")
    if len(parts) != 2:
        return False, "Email must have exactly one @"

    local, domain = parts

    if not local:
        return False, "Username part cannot be empty"

    if "." not in domain:
        return False, "Domain must have a dot (e.g., .com)"

    if len(email) > 254:
        return False, "Email too long"

    return True, "✅ Valid email"

# Test it
test_emails = [
    "alice@gmail.com",
    "john.doe@company.co.in",
    "notanemail",
    "missing@dotcom",
    "@nodomain.com",
    "",
]

for email in test_emails:
    valid, msg = validate_email(email)
    status = "✅" if valid else "❌"
    print(f"{status} '{email}' → {msg}")
```

### 🟠 Real-World Scenario 2 — Password Strength Checker
```python
def check_password_strength(password):
    """
    Returns strength score and feedback.
    Used in user registration forms.
    """
    score = 0
    feedback = []

    if len(password) >= 8:
        score += 1
    else:
        feedback.append("Use at least 8 characters")

    if len(password) >= 12:
        score += 1

    if any(c.isupper() for c in password):
        score += 1
    else:
        feedback.append("Add uppercase letters (A-Z)")

    if any(c.islower() for c in password):
        score += 1
    else:
        feedback.append("Add lowercase letters (a-z)")

    if any(c.isdigit() for c in password):
        score += 1
    else:
        feedback.append("Add numbers (0-9)")

    special_chars = "!@#$%^&*()_+-=[]{}|;:,.<>?"
    if any(c in special_chars for c in password):
        score += 1
    else:
        feedback.append("Add special characters (!@#$...)")

    if score <= 2:
        strength = "🔴 Weak"
    elif score <= 4:
        strength = "🟡 Moderate"
    elif score == 5:
        strength = "🟢 Strong"
    else:
        strength = "💪 Very Strong"

    return {"score": score, "strength": strength, "tips": feedback}

passwords = ["abc", "Password1", "P@ssw0rd!", "Tr0ub4dor&3#secure"]
for pwd in passwords:
    result = check_password_strength(pwd)
    print(f"'{pwd[:15]}...' → {result['strength']} ({result['score']}/6)")
    for tip in result["tips"]:
        print(f"  💡 {tip}")
```

---

## 10. Lists

Lists are **ordered, mutable** collections of items. Most used data structure in Python.

```python
# Creating lists
fruits = ["apple", "banana", "cherry"]
numbers = [1, 2, 3, 4, 5]
mixed = [1, "hello", True, 3.14, None]
empty = []

# Accessing items
print(fruits[0])    # apple (first)
print(fruits[-1])   # cherry (last)
print(fruits[1:3])  # ['banana', 'cherry'] (slice)

# Modifying lists
fruits.append("mango")          # Add to end
fruits.insert(1, "blueberry")   # Insert at index 1
fruits.extend(["kiwi", "plum"]) # Add multiple items
fruits.remove("banana")         # Remove first occurrence
popped = fruits.pop()           # Remove & return last item
popped2 = fruits.pop(0)         # Remove & return at index 0
del fruits[1]                   # Delete by index
fruits.clear()                  # Empty the list

# Useful list methods
nums = [3, 1, 4, 1, 5, 9, 2, 6]
print(sorted(nums))             # [1, 1, 2, 3, 4, 5, 6, 9] (new sorted list)
nums.sort()                     # Sort in place
nums.sort(reverse=True)        # Sort descending
nums.reverse()                  # Reverse in place
print(nums.count(1))            # Count occurrences of 1
print(nums.index(4))            # Find index of value 4
print(len(nums))                # Length of list
print(sum(nums))                # Sum of all numbers
print(min(nums))                # Minimum value
print(max(nums))                # Maximum value

# Checking membership
print("apple" in fruits)        # True/False

# Copy a list (important!)
original = [1, 2, 3]
bad_copy = original       # This is NOT a copy — same object!
good_copy = original.copy()   # Actual copy
also_good = original[:]       # Also a copy
```

### List Comprehension (Preview)

```python
squares = [x**2 for x in range(1, 6)]
print(squares)  # [1, 4, 9, 16, 25]

evens = [x for x in range(20) if x % 2 == 0]
print(evens)    # [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```

### 🟠 Real-World Scenario — Student Grade Report
```python
def generate_grade_report(students):
    """Process a list of student scores and generate report."""

    # Sort by score descending
    sorted_students = sorted(students, key=lambda s: s["score"], reverse=True)

    total_scores = [s["score"] for s in students]
    class_avg = sum(total_scores) / len(total_scores)
    top_score = max(total_scores)
    lowest_score = min(total_scores)

    print("=" * 45)
    print(f"{'RANK':<5} {'NAME':<20} {'SCORE':<8} {'GRADE'}")
    print("=" * 45)

    for rank, student in enumerate(sorted_students, 1):
        score = student["score"]
        if score >= 90:   grade = "A+"
        elif score >= 80: grade = "A"
        elif score >= 70: grade = "B"
        elif score >= 60: grade = "C"
        else:             grade = "F"

        marker = " 🏆" if rank == 1 else ""
        print(f"{rank:<5} {student['name']:<20} {score:<8} {grade}{marker}")

    print("=" * 45)
    print(f"Class Average: {class_avg:.1f} | Top: {top_score} | Low: {lowest_score}")
    passed = len([s for s in students if s["score"] >= 60])
    print(f"Pass rate: {passed}/{len(students)} ({passed/len(students)*100:.0f}%)")

students = [
    {"name": "Alice Kumar",    "score": 94},
    {"name": "Bob Singh",      "score": 78},
    {"name": "Carol Sharma",   "score": 88},
    {"name": "David Patel",    "score": 55},
    {"name": "Eva Reddy",      "score": 91},
    {"name": "Frank Nair",     "score": 67},
]

generate_grade_report(students)
```

---

## 11. Tuples

Tuples are **ordered, immutable** collections. Like lists, but cannot be changed after creation. Use when data should not change.

```python
# Creating tuples
point = (3, 5)
colors = ("red", "green", "blue")
single = (42,)         # IMPORTANT: comma needed for single-item tuple
empty_tuple = ()

# Accessing (same as list)
print(point[0])        # 3
print(colors[1])       # green
print(colors[-1])      # blue

# Tuple unpacking
x, y = point
print(f"x={x}, y={y}")

# Swap variables elegantly
a, b = 10, 20
a, b = b, a            # Python magic!
print(a, b)            # 20, 10

# Named tuples (more readable)
from collections import namedtuple

Employee = namedtuple("Employee", ["name", "department", "salary"])
emp = Employee("Alice", "Engineering", 75000)
print(emp.name)         # Alice
print(emp.salary)       # 75000
print(emp)              # Employee(name='Alice', department='Engineering', salary=75000)

# Tuples are hashable → can be dict keys or set members
location_data = {
    (28.6, 77.2): "New Delhi",
    (19.0, 72.8): "Mumbai",
    (13.0, 80.2): "Chennai",
}
print(location_data[(28.6, 77.2)])  # New Delhi
```

### When to use Tuple vs List?

| Situation | Use |
|---|---|
| Data that shouldn't change (coordinates, RGB color, DB record) | Tuple |
| Collection of items you'll add/remove/sort | List |
| Returning multiple values from a function | Tuple |
| Dictionary key that's a compound value | Tuple (lists can't be keys) |

---

## 12. Dictionaries

Dictionaries store **key-value pairs**. Like a real dictionary — look up a word (key) to find its meaning (value).

```python
# Creating a dictionary
person = {
    "name": "Alice",
    "age": 28,
    "city": "Bangalore",
    "skills": ["Python", "SQL", "Docker"]
}

# Accessing values
print(person["name"])              # Alice
print(person.get("age"))           # 28
print(person.get("salary", 0))     # 0 (default if key missing — no error!)

# Adding / Updating
person["email"] = "alice@gmail.com"  # Add new key
person["age"] = 29                   # Update existing key

# Deleting
del person["city"]
removed = person.pop("email")        # Remove & return value

# Useful methods
print(person.keys())    # dict_keys(['name', 'age', 'skills'])
print(person.values())  # dict_values(['Alice', 29, [...]])
print(person.items())   # dict_items([('name', 'Alice'), ...])

# Check if key exists
print("name" in person)    # True
print("salary" in person)  # False

# Iterate over dictionary
for key, value in person.items():
    print(f"{key}: {value}")

# Nested dictionaries
company = {
    "name": "TechCorp",
    "departments": {
        "engineering": {"head": "Alice", "size": 25},
        "marketing":   {"head": "Bob",   "size": 10},
    }
}
print(company["departments"]["engineering"]["head"])  # Alice

# Dictionary comprehension
squares = {x: x**2 for x in range(1, 6)}
print(squares)  # {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

# Merge two dicts (Python 3.9+)
defaults = {"theme": "light", "lang": "en", "notifications": True}
user_prefs = {"theme": "dark", "lang": "hi"}
merged = defaults | user_prefs    # user_prefs overrides defaults
print(merged)
# {'theme': 'dark', 'lang': 'hi', 'notifications': True}
```

### 🟠 Real-World Scenario — Phone Book / Contact Manager
```python
contacts = {}

def add_contact(name, phone, email=None):
    contacts[name.lower()] = {
        "name": name,
        "phone": phone,
        "email": email or "Not provided"
    }
    print(f"✅ Contact '{name}' added.")

def search_contact(name):
    result = contacts.get(name.lower())
    if result:
        print(f"\n📇 Contact Found:")
        print(f"   Name:  {result['name']}")
        print(f"   Phone: {result['phone']}")
        print(f"   Email: {result['email']}")
    else:
        print(f"❌ Contact '{name}' not found.")

def delete_contact(name):
    if name.lower() in contacts:
        contacts.pop(name.lower())
        print(f"🗑️  Contact '{name}' deleted.")
    else:
        print(f"❌ Contact '{name}' not found.")

def list_all():
    if not contacts:
        print("📭 No contacts found.")
        return
    print(f"\n📒 All Contacts ({len(contacts)} total):")
    for key, data in sorted(contacts.items()):
        print(f"  - {data['name']}: {data['phone']}")

# Using the contact manager
add_contact("Alice Kumar",  "+91-9876543210", "alice@gmail.com")
add_contact("Bob Singh",    "+91-9876500001")
add_contact("Carol Sharma", "+91-9999888877", "carol@work.com")

list_all()
search_contact("Alice Kumar")
delete_contact("Bob Singh")
list_all()
```

---

## 13. Sets

Sets store **unique, unordered** items. Perfect for removing duplicates and mathematical set operations.

```python
# Creating sets
fruits = {"apple", "banana", "cherry", "apple"}  # Duplicate removed!
print(fruits)  # {'banana', 'cherry', 'apple'} (unordered)

numbers = set([1, 2, 2, 3, 3, 3, 4])
print(numbers)  # {1, 2, 3, 4}

# Add / Remove
fruits.add("mango")
fruits.remove("banana")    # Error if not found
fruits.discard("kiwi")     # No error if not found

# Set operations (like math)
a = {1, 2, 3, 4, 5}
b = {4, 5, 6, 7, 8}

print(a | b)   # Union: {1, 2, 3, 4, 5, 6, 7, 8}
print(a & b)   # Intersection: {4, 5}
print(a - b)   # Difference: {1, 2, 3}
print(a ^ b)   # Symmetric difference: {1, 2, 3, 6, 7, 8}

print(a.issubset({1, 2, 3, 4, 5, 6}))     # True
print(a.issuperset({1, 2}))               # True
print(a.isdisjoint({6, 7, 8}))            # True (no common elements)
```

### 🟠 Real-World Scenario — User Permission System
```python
def check_access(user_permissions, required_permissions):
    """
    Checks if a user has all required permissions.
    Real scenario: API endpoint access control.
    """
    user_set = set(user_permissions)
    required_set = set(required_permissions)

    has_access = required_set.issubset(user_set)
    missing = required_set - user_set
    extra = user_set - required_set

    return {
        "access_granted": has_access,
        "missing_permissions": missing,
        "extra_permissions": extra
    }

# Admin trying to access report endpoint
admin_perms = ["read", "write", "delete", "export", "admin"]
report_requires = ["read", "export"]

result = check_access(admin_perms, report_requires)
print(f"Access: {'✅ Granted' if result['access_granted'] else '❌ Denied'}")

# Regular user trying to access admin endpoint
user_perms = ["read", "write"]
admin_requires = ["read", "write", "admin", "export"]

result = check_access(user_perms, admin_requires)
print(f"Access: {'✅ Granted' if result['access_granted'] else '❌ Denied'}")
print(f"Missing permissions: {result['missing_permissions']}")
```

---

## 14. File Handling

Reading and writing files is essential for any real application.

```python
# ─── Writing to a file ───
with open("output.txt", "w") as file:    # "w" = write (overwrites)
    file.write("Hello, file!\n")
    file.write("Second line.\n")

# ─── Appending to a file ───
with open("output.txt", "a") as file:    # "a" = append (adds to end)
    file.write("Third line added later.\n")

# ─── Reading a file ───
with open("output.txt", "r") as file:   # "r" = read
    content = file.read()               # Read entire file as string
    print(content)

# Read line by line (memory efficient for large files!)
with open("output.txt", "r") as file:
    for line in file:
        print(line.strip())             # strip() removes trailing \n

# Read all lines into a list
with open("output.txt", "r") as file:
    lines = file.readlines()
    print(lines)  # ['Hello, file!\n', 'Second line.\n', ...]
```

### File Modes Quick Reference

| Mode | Meaning |
|---|---|
| `"r"` | Read (default) — error if file doesn't exist |
| `"w"` | Write — creates file, **overwrites** if exists |
| `"a"` | Append — creates file, adds to end if exists |
| `"x"` | Exclusive create — error if file already exists |
| `"rb"` / `"wb"` | Read/write in **binary** mode (images, PDFs, etc.) |

### 🟠 Real-World Scenario — Application Log System
```python
import datetime
import os

LOG_FILE = "app.log"

def log(level, message, module="app"):
    """
    Append a timestamped log entry to app.log
    Used across all production applications.
    """
    timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    entry = f"[{timestamp}] [{level.upper():8}] [{module}] {message}\n"

    with open(LOG_FILE, "a") as f:
        f.write(entry)

    # Also print to console if ERROR or above
    if level.upper() in ("ERROR", "CRITICAL"):
        print(f"🔴 {entry.strip()}")

def read_logs(level_filter=None, last_n=None):
    """Read and display logs with optional filtering."""
    if not os.path.exists(LOG_FILE):
        print("No log file found.")
        return

    with open(LOG_FILE, "r") as f:
        lines = f.readlines()

    if level_filter:
        lines = [l for l in lines if f"[{level_filter.upper()}" in l]

    if last_n:
        lines = lines[-last_n:]

    for line in lines:
        print(line.strip())

# Usage
log("INFO",     "Application started",            module="main")
log("INFO",     "Database connected",              module="db")
log("WARNING",  "High memory usage: 87%",          module="monitor")
log("ERROR",    "Failed to send email to user 42", module="email")
log("INFO",     "User 1042 logged in",             module="auth")
log("CRITICAL", "Database connection lost!",       module="db")

print("\n--- Last 3 log entries ---")
read_logs(last_n=3)

print("\n--- ERROR logs only ---")
read_logs(level_filter="ERROR")
```

---

## 15. Error Handling — try / except

Errors will happen. Handle them gracefully instead of letting your program crash.

```python
# Basic try/except
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero!")

# Multiple exception types
try:
    number = int("abc")
except ValueError as e:
    print(f"Value error: {e}")
except TypeError as e:
    print(f"Type error: {e}")

# Catch any exception (use sparingly)
try:
    risky_operation()
except Exception as e:
    print(f"Something went wrong: {e}")

# else — runs if NO exception occurred
try:
    result = 10 / 2
except ZeroDivisionError:
    print("Error!")
else:
    print(f"Success: {result}")  # Runs only if no exception

# finally — ALWAYS runs (cleanup code)
try:
    file = open("data.txt", "r")
    data = file.read()
except FileNotFoundError:
    print("File not found.")
finally:
    print("This always runs — good for cleanup")

# Custom exceptions
class InsufficientFundsError(Exception):
    def __init__(self, balance, amount):
        self.balance = balance
        self.amount = amount
        super().__init__(f"Need ₹{amount} but only ₹{balance} available")

def withdraw(balance, amount):
    if amount > balance:
        raise InsufficientFundsError(balance, amount)
    return balance - amount

try:
    new_balance = withdraw(500, 1000)
except InsufficientFundsError as e:
    print(f"Transaction failed: {e}")
```

### Common Exception Types

| Exception | When it Occurs |
|---|---|
| `ValueError` | Wrong value type (int("abc")) |
| `TypeError` | Wrong type for operation ("a" + 1) |
| `KeyError` | Key not in dictionary |
| `IndexError` | Index out of list range |
| `FileNotFoundError` | File doesn't exist |
| `ZeroDivisionError` | Dividing by zero |
| `AttributeError` | Object has no such attribute |
| `ImportError` | Module not found |
| `PermissionError` | No permission to access file |

### 🟠 Real-World Scenario — Robust API Data Fetcher
```python
import json
import os

def load_config(filepath):
    """
    Safely load a JSON config file.
    Handles all common failure scenarios.
    """
    try:
        with open(filepath, "r") as f:
            config = json.load(f)
        print(f"✅ Config loaded from {filepath}")
        return config

    except FileNotFoundError:
        print(f"❌ Config file not found: {filepath}")
        print("   Using default configuration.")
        return {"debug": False, "db_url": "sqlite:///default.db"}

    except json.JSONDecodeError as e:
        print(f"❌ Invalid JSON in config file: {e}")
        print("   Please fix the syntax error.")
        return None

    except PermissionError:
        print(f"❌ Permission denied reading: {filepath}")
        return None

    except Exception as e:
        print(f"❌ Unexpected error loading config: {type(e).__name__}: {e}")
        return None

# Test it
config = load_config("config.json")      # File doesn't exist
config = load_config("/etc/passwd")      # Wrong format (not JSON)
```

---

## 16. Object-Oriented Programming (OOP)

OOP lets you model real-world things as **objects** that have properties (attributes) and behaviors (methods).

### The Four Pillars

1. **Encapsulation** — Bundle data + methods, hide internal details
2. **Inheritance** — Child class inherits from parent class
3. **Polymorphism** — Same method name, different behavior
4. **Abstraction** — Hide complex implementation, show simple interface

### Classes & Objects

```python
class Dog:
    # Class variable (shared by all instances)
    species = "Canis familiaris"

    # Constructor — runs when object is created
    def __init__(self, name, breed, age):
        # Instance variables (unique to each object)
        self.name = name
        self.breed = breed
        self.age = age
        self._energy = 100      # _ prefix = "protected" (convention)

    # Instance method
    def bark(self):
        return f"{self.name} says: Woof! 🐕"

    def describe(self):
        return f"{self.name} is a {self.age}-year-old {self.breed}"

    def fetch(self, item):
        if self._energy > 10:
            self._energy -= 20
            return f"{self.name} fetches the {item}! (Energy: {self._energy})"
        else:
            return f"{self.name} is too tired to fetch."

    # String representation
    def __str__(self):
        return f"Dog({self.name}, {self.breed})"

    def __repr__(self):
        return f"Dog(name={self.name!r}, breed={self.breed!r}, age={self.age})"

# Create objects (instances)
buddy = Dog("Buddy", "Golden Retriever", 3)
max   = Dog("Max", "German Shepherd", 5)

print(buddy.bark())
print(max.describe())
print(buddy.fetch("ball"))
print(buddy.fetch("stick"))
print(Dog.species)        # Class variable
print(buddy.species)      # Also accessible via instance
```

### Inheritance

```python
class Animal:
    def __init__(self, name, sound):
        self.name = name
        self.sound = sound

    def speak(self):
        return f"{self.name} says {self.sound}"

    def eat(self):
        return f"{self.name} is eating."

class Dog(Animal):
    def __init__(self, name):
        super().__init__(name, "Woof")   # Call parent constructor
        self.tricks = []

    def learn_trick(self, trick):
        self.tricks.append(trick)
        return f"{self.name} learned: {trick}!"

    def perform(self):
        if self.tricks:
            return f"{self.name} performs: {', '.join(self.tricks)}"
        return f"{self.name} doesn't know any tricks yet."

    # Override parent method
    def speak(self):
        return f"{self.name} barks loudly: WOOF WOOF! 🐕"

class Cat(Animal):
    def __init__(self, name, indoor=True):
        super().__init__(name, "Meow")
        self.indoor = indoor

    def speak(self):
        return f"{self.name} meows softly: meow... 🐱"

# Polymorphism in action
animals = [Dog("Buddy"), Cat("Whiskers"), Dog("Max")]
for animal in animals:
    print(animal.speak())   # Each calls its OWN speak() method
```

### 🟠 Real-World Scenario — Bank Account System
```python
class BankAccount:
    _total_accounts = 0   # Class variable

    def __init__(self, owner, account_number, initial_balance=0):
        self.owner = owner
        self.account_number = account_number
        self._balance = initial_balance
        self._transactions = []
        BankAccount._total_accounts += 1

    @property
    def balance(self):
        return self._balance

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("Deposit amount must be positive")
        self._balance += amount
        self._transactions.append({"type": "deposit", "amount": amount, "balance": self._balance})
        print(f"✅ Deposited ₹{amount:,}. New balance: ₹{self._balance:,}")

    def withdraw(self, amount):
        if amount <= 0:
            raise ValueError("Withdrawal amount must be positive")
        if amount > self._balance:
            raise ValueError(f"Insufficient funds. Balance: ₹{self._balance:,}")
        self._balance -= amount
        self._transactions.append({"type": "withdraw", "amount": amount, "balance": self._balance})
        print(f"✅ Withdrew ₹{amount:,}. New balance: ₹{self._balance:,}")

    def get_statement(self, last_n=5):
        print(f"\n{'='*45}")
        print(f"Account: {self.account_number} | Owner: {self.owner}")
        print(f"{'='*45}")
        for t in self._transactions[-last_n:]:
            symbol = "+" if t["type"] == "deposit" else "-"
            print(f"  {symbol}₹{t['amount']:>10,}  |  Balance: ₹{t['balance']:,}")
        print(f"{'='*45}")
        print(f"Current Balance: ₹{self._balance:,}")

    @classmethod
    def get_total_accounts(cls):
        return cls._total_accounts

    def __str__(self):
        return f"BankAccount({self.owner}, ****{self.account_number[-4:]})"


class SavingsAccount(BankAccount):
    def __init__(self, owner, account_number, initial_balance=0, interest_rate=0.04):
        super().__init__(owner, account_number, initial_balance)
        self.interest_rate = interest_rate

    def add_interest(self):
        interest = self._balance * self.interest_rate
        self._balance += interest
        print(f"💰 Interest added: ₹{interest:,.2f} @ {self.interest_rate*100}%")
        return interest


# Using the system
acc1 = BankAccount("Alice Kumar", "ACC001", initial_balance=10000)
acc2 = SavingsAccount("Bob Singh", "SAV002", initial_balance=50000, interest_rate=0.06)

acc1.deposit(5000)
acc1.withdraw(2000)
acc1.deposit(1000)

acc2.deposit(10000)
acc2.add_interest()
acc2.withdraw(5000)

acc1.get_statement()
acc2.get_statement()

print(f"\nTotal accounts opened: {BankAccount.get_total_accounts()}")
```

---

## 17. Modules & Packages

Modules let you organize code across multiple files and reuse code.

```python
# ─── Using built-in modules ───
import os                    # Operating system interface
import sys                   # Python interpreter
import math                  # Math functions
import datetime              # Dates and times
import random                # Random numbers
import json                  # JSON encode/decode
import re                    # Regular expressions
import time                  # Time operations
import pathlib               # File path handling

# Import specific items
from datetime import datetime, timedelta
from pathlib import Path
from os.path import exists, join

# Import with alias
import numpy as np           # Common alias
import pandas as pd          # Common alias
import matplotlib.pyplot as plt  # Common alias

# ─── os module — file system operations ───
print(os.getcwd())                # Current working directory
print(os.listdir("."))            # List files in current dir
os.makedirs("logs", exist_ok=True)  # Create directory
print(os.path.exists("logs"))     # Check if path exists
print(os.path.join("logs", "app.log"))  # Build paths safely
print(os.environ.get("HOME"))     # Read environment variable

# ─── datetime module ───
now = datetime.now()
print(now)                        # 2024-01-15 14:32:05.123456
print(now.strftime("%d %B %Y"))   # 15 January 2024
print(now.strftime("%Y-%m-%d"))   # 2024-01-15
tomorrow = now + timedelta(days=1)
next_week = now + timedelta(weeks=1)
birthday = datetime(1995, 6, 15)
age_days = (now - birthday).days
print(f"Days since birthday: {age_days}")
```

### Creating Your Own Module

```python
# math_utils.py  ← your own module file
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def percentage(part, total):
    if total == 0:
        return 0
    return (part / total) * 100

PI = 3.14159265358979
```

```python
# main.py  ← import and use your module
import math_utils

print(math_utils.add(5, 3))          # 8
print(math_utils.percentage(25, 200)) # 12.5

# Or import specific functions
from math_utils import add, percentage
print(add(10, 20))
```

---

## 18. List Comprehensions & Generators

These are powerful Python shortcuts for creating collections efficiently.

### List Comprehensions

```python
# Format: [expression for item in iterable if condition]

# Basic
squares = [x**2 for x in range(1, 11)]
# [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

# With condition (filter)
even_squares = [x**2 for x in range(1, 11) if x % 2 == 0]
# [4, 16, 36, 64, 100]

# String transformation
names = ["alice", "BOB", "carol", "DAVID"]
clean = [name.title() for name in names]
# ['Alice', 'Bob', 'Carol', 'David']

# Dict comprehension
word_lengths = {word: len(word) for word in ["Python", "Java", "Go", "Rust"]}
# {'Python': 6, 'Java': 4, 'Go': 2, 'Rust': 4}

# Set comprehension
unique_lengths = {len(word) for word in ["cat", "dog", "bird", "ant"]}
# {3, 4}

# Nested comprehension
matrix = [[1,2,3],[4,5,6],[7,8,9]]
flat = [num for row in matrix for num in row]
# [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### Generators

Generators are like list comprehensions but **lazy** — they produce items one at a time, using almost no memory.

```python
# Generator expression (like list comp but with ())
gen = (x**2 for x in range(1_000_000))  # Uses almost NO memory
print(next(gen))   # 0
print(next(gen))   # 1
print(next(gen))   # 4

# Generator function
def fibonacci(n):
    a, b = 0, 1
    count = 0
    while count < n:
        yield a          # yield pauses and returns a value
        a, b = b, a + b
        count += 1

for num in fibonacci(10):
    print(num, end=" ")
# 0 1 1 2 3 5 8 13 21 34
```

### 🟠 Real-World Scenario — Data Pipeline Processing
```python
import csv

# Simulated sales data processing pipeline
sales_data = [
    {"product": "Laptop", "qty": 5,  "price": 55000, "region": "North"},
    {"product": "Mouse",  "qty": 50, "price": 800,   "region": "South"},
    {"product": "Laptop", "qty": 3,  "price": 55000, "region": "East"},
    {"product": "Keyboard","qty":20, "price": 1500,  "region": "North"},
    {"product": "Monitor","qty": 8,  "price": 18000, "region": "West"},
    {"product": "Mouse",  "qty": 30, "price": 800,   "region": "North"},
]

# Using comprehensions as a data pipeline
total_revenue = sum(
    item["qty"] * item["price"] for item in sales_data
)

north_revenue = sum(
    item["qty"] * item["price"]
    for item in sales_data
    if item["region"] == "North"
)

product_totals = {
    product: sum(
        item["qty"] * item["price"]
        for item in sales_data
        if item["product"] == product
    )
    for product in {item["product"] for item in sales_data}
}

print(f"Total Revenue:  ₹{total_revenue:,}")
print(f"North Revenue:  ₹{north_revenue:,}")
print("\nRevenue by Product:")
for product, revenue in sorted(product_totals.items(), key=lambda x: x[1], reverse=True):
    print(f"  {product:12} → ₹{revenue:>10,}")
```

---

## 19. Decorators

Decorators let you **add behavior to functions** without changing their code. Used everywhere in real Python code (Flask routes, Django views, retry logic, logging, timing).

```python
# A decorator is a function that takes a function and returns a function
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Before the function runs")
        result = func(*args, **kwargs)
        print("After the function runs")
        return result
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

say_hello()
# Before the function runs
# Hello!
# After the function runs
```

### Practical Decorators

```python
import time
import functools

# ─── Timer decorator ───
def timer(func):
    @functools.wraps(func)       # Preserves original function name
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"⏱️  {func.__name__} took {end-start:.4f} seconds")
        return result
    return wrapper

@timer
def slow_operation():
    time.sleep(0.5)
    return "Done"

slow_operation()  # ⏱️  slow_operation took 0.5012 seconds

# ─── Retry decorator ───
def retry(max_attempts=3, delay=1):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt < max_attempts:
                        print(f"Attempt {attempt} failed: {e}. Retrying in {delay}s...")
                        time.sleep(delay)
                    else:
                        print(f"All {max_attempts} attempts failed.")
                        raise
        return wrapper
    return decorator

@retry(max_attempts=3, delay=2)
def call_external_api(url):
    import random
    if random.random() < 0.7:   # 70% chance of failure (simulating flaky API)
        raise ConnectionError("API timeout")
    return {"status": "ok", "data": "..."}
```

### 🟠 Real-World Scenario — Auth Decorator (Like Flask/Django)
```python
# Simulating how web frameworks protect routes with decorators

current_user = {"id": 1, "name": "Alice", "role": "user"}  # Simulated session

def require_auth(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        if not current_user:
            print("❌ 401 Unauthorized — Please log in first")
            return None
        return func(*args, **kwargs)
    return wrapper

def require_role(*roles):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            if current_user.get("role") not in roles:
                print(f"❌ 403 Forbidden — Requires role: {roles}")
                return None
            print(f"✅ Access granted to {current_user['name']}")
            return func(*args, **kwargs)
        return wrapper
    return decorator

# Protecting endpoints
@require_auth
@require_role("admin", "superuser")
def delete_user(user_id):
    print(f"🗑️  User {user_id} deleted")

@require_auth
@require_role("user", "admin", "superuser")
def view_profile(user_id):
    print(f"👤 Viewing profile of user {user_id}")

delete_user(42)       # ❌ Forbidden (current_user is "user" not "admin")
view_profile(42)      # ✅ Allowed (current_user "user" is in allowed roles)
```

---

## 20. Lambda, Map, Filter, Reduce

Compact tools for functional-style programming.

```python
# Lambda — anonymous one-line function
# lambda arguments: expression

double = lambda x: x * 2
add = lambda x, y: x + y
square = lambda x: x ** 2

print(double(5))    # 10
print(add(3, 4))    # 7

# ─── map() — apply function to every item ───
numbers = [1, 2, 3, 4, 5]
doubled = list(map(lambda x: x * 2, numbers))
print(doubled)  # [2, 4, 6, 8, 10]

names = ["alice", "bob", "carol"]
titled = list(map(str.title, names))
print(titled)  # ['Alice', 'Bob', 'Carol']

# ─── filter() — keep items where function returns True ───
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
evens = list(filter(lambda x: x % 2 == 0, numbers))
print(evens)  # [2, 4, 6, 8, 10]

words = ["apple", "a", "banana", "an", "cherry"]
long_words = list(filter(lambda w: len(w) > 3, words))
print(long_words)  # ['apple', 'banana', 'cherry']

# ─── reduce() — reduce list to single value ───
from functools import reduce

numbers = [1, 2, 3, 4, 5]
product = reduce(lambda x, y: x * y, numbers)
print(product)  # 120 (1*2*3*4*5)

# Sorting with lambda key
employees = [
    {"name": "Alice", "salary": 75000},
    {"name": "Bob",   "salary": 50000},
    {"name": "Carol", "salary": 90000},
]

by_salary = sorted(employees, key=lambda e: e["salary"], reverse=True)
for emp in by_salary:
    print(f"{emp['name']}: ₹{emp['salary']:,}")
```

---

## 21. Regular Expressions

Regex is a powerful mini-language for matching and extracting text patterns.

```python
import re

# ─── Basic patterns ───
# .  = any character
# *  = 0 or more of previous
# +  = 1 or more of previous
# ?  = 0 or 1 of previous
# \d = digit [0-9]
# \w = word character [a-zA-Z0-9_]
# \s = whitespace
# ^  = start of string
# $  = end of string
# [abc] = character class
# (group) = capture group

text = "My phone is +91-9876543210 and email is alice@gmail.com"

# ─── re.search() — find first match ───
phone_match = re.search(r"\+\d{2}-\d{10}", text)
if phone_match:
    print(f"Phone: {phone_match.group()}")    # +91-9876543210

# ─── re.findall() — find all matches ───
emails = re.findall(r"[\w.+-]+@[\w-]+\.[a-z]{2,}", text)
print(emails)   # ['alice@gmail.com']

# ─── re.sub() — find and replace ───
redacted = re.sub(r"\d{10}", "XXXXXXXXXX", text)
print(redacted)  # My phone is +91-XXXXXXXXXX...

# ─── re.match() — match at start of string ───
pattern = re.compile(r"^\d{4}-\d{2}-\d{2}$")   # Date format YYYY-MM-DD
print(bool(pattern.match("2024-01-15")))  # True
print(bool(pattern.match("15/01/2024")))  # False

# ─── Groups ───
log = "2024-01-15 14:32:05 ERROR Database connection failed"
match = re.search(r"(\d{4}-\d{2}-\d{2}) (\d{2}:\d{2}:\d{2}) (\w+) (.+)", log)
if match:
    date, time, level, message = match.groups()
    print(f"Date: {date}, Level: {level}")
```

### 🟠 Real-World Scenario — Form Input Validation
```python
import re

def validate_form(data):
    errors = {}

    # Validate phone (Indian format: +91 or 0 + 10 digits)
    phone = data.get("phone", "")
    if not re.match(r"^(\+91|0)?[6-9]\d{9}$", phone):
        errors["phone"] = "Invalid Indian phone number"

    # Validate email
    email = data.get("email", "")
    if not re.match(r"^[\w.+-]+@[\w-]+\.[a-z]{2,}$", email, re.IGNORECASE):
        errors["email"] = "Invalid email address"

    # Validate PIN code (India: 6 digits)
    pincode = data.get("pincode", "")
    if not re.match(r"^[1-9]\d{5}$", pincode):
        errors["pincode"] = "Invalid PIN code (must be 6 digits)"

    # Validate PAN number (India: AAAAA9999A format)
    pan = data.get("pan", "")
    if pan and not re.match(r"^[A-Z]{5}[0-9]{4}[A-Z]$", pan):
        errors["pan"] = "Invalid PAN number format"

    return errors

# Test validation
form = {
    "phone":   "+919876543210",
    "email":   "alice@gmail.com",
    "pincode": "500001",
    "pan":     "ABCDE1234F"
}

errors = validate_form(form)
if errors:
    for field, msg in errors.items():
        print(f"❌ {field}: {msg}")
else:
    print("✅ All fields valid!")
```

---

## 22. Working with APIs (requests)

The `requests` library lets your Python code talk to web services over HTTP.

```python
# Install first: pip install requests
import requests
import json

# ─── GET request — fetch data ───
response = requests.get("https://api.github.com/users/python")

print(response.status_code)     # 200 (OK), 404 (Not Found), etc.
print(response.headers)         # Response headers
data = response.json()          # Parse JSON response
print(data["name"])
print(data["public_repos"])

# ─── With parameters ───
params = {"q": "python", "sort": "stars", "per_page": 5}
response = requests.get(
    "https://api.github.com/search/repositories",
    params=params
)
results = response.json()["items"]
for repo in results:
    print(f"{repo['full_name']}: ⭐ {repo['stargazers_count']:,}")

# ─── POST request — send data ───
payload = {"title": "Test Post", "body": "Hello from Python!", "userId": 1}
response = requests.post(
    "https://jsonplaceholder.typicode.com/posts",
    json=payload,
    headers={"Content-Type": "application/json"}
)
print(response.status_code)     # 201 Created
print(response.json())

# ─── With authentication ───
headers = {"Authorization": "Bearer YOUR_API_TOKEN"}
response = requests.get("https://api.example.com/data", headers=headers)

# ─── Error handling ───
try:
    response = requests.get("https://api.example.com", timeout=10)
    response.raise_for_status()   # Raises exception for 4xx/5xx status
    data = response.json()
except requests.exceptions.Timeout:
    print("❌ Request timed out")
except requests.exceptions.ConnectionError:
    print("❌ Connection failed")
except requests.exceptions.HTTPError as e:
    print(f"❌ HTTP error: {e}")
```

---

## 23. Working with JSON & CSV

JSON and CSV are the two most common data exchange formats.

```python
import json
import csv

# ─── JSON ───
# Python dict → JSON string
data = {"name": "Alice", "age": 25, "skills": ["Python", "SQL"]}
json_string = json.dumps(data, indent=2)
print(json_string)

# JSON string → Python dict
parsed = json.loads(json_string)
print(parsed["name"])

# Write JSON to file
with open("data.json", "w") as f:
    json.dump(data, f, indent=2)

# Read JSON from file
with open("data.json", "r") as f:
    loaded = json.load(f)

# ─── CSV ───
# Write CSV
employees = [
    ["Name", "Department", "Salary"],
    ["Alice Kumar",  "Engineering", 75000],
    ["Bob Singh",    "Marketing",   55000],
    ["Carol Sharma", "Engineering", 85000],
]

with open("employees.csv", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerows(employees)

# Read CSV
with open("employees.csv", "r") as f:
    reader = csv.DictReader(f)   # Reads rows as dicts using header row
    for row in reader:
        print(f"{row['Name']}: ₹{int(row['Salary']):,}")
```

---

## 24. Database with SQLite

SQLite is a built-in Python database — no server needed. Perfect for small apps and learning SQL.

```python
import sqlite3

# Connect (creates file if doesn't exist)
conn = sqlite3.connect("company.db")
cursor = conn.cursor()

# Create table
cursor.execute("""
    CREATE TABLE IF NOT EXISTS employees (
        id          INTEGER PRIMARY KEY AUTOINCREMENT,
        name        TEXT NOT NULL,
        department  TEXT,
        salary      REAL,
        hire_date   TEXT
    )
""")

# Insert data (ALWAYS use ? placeholders — prevents SQL injection!)
employees = [
    ("Alice Kumar",  "Engineering", 75000, "2022-03-15"),
    ("Bob Singh",    "Marketing",   55000, "2021-07-01"),
    ("Carol Sharma", "Engineering", 85000, "2020-01-10"),
    ("David Patel",  "HR",          50000, "2023-06-20"),
]
cursor.executemany(
    "INSERT INTO employees (name, department, salary, hire_date) VALUES (?,?,?,?)",
    employees
)
conn.commit()

# Query data
cursor.execute("SELECT * FROM employees WHERE department = ?", ("Engineering",))
results = cursor.fetchall()
for row in results:
    print(row)

# Query as dict (more readable)
conn.row_factory = sqlite3.Row
cursor = conn.cursor()
cursor.execute("SELECT name, salary FROM employees ORDER BY salary DESC")
for row in cursor.fetchall():
    print(f"{row['name']}: ₹{row['salary']:,.0f}")

# Update
cursor.execute("UPDATE employees SET salary = ? WHERE name = ?", (80000, "Bob Singh"))
conn.commit()

# Delete
cursor.execute("DELETE FROM employees WHERE id = ?", (4,))
conn.commit()

# Aggregate queries
cursor.execute("SELECT department, COUNT(*), AVG(salary) FROM employees GROUP BY department")
for dept, count, avg_sal in cursor.fetchall():
    print(f"{dept}: {count} employees, avg salary ₹{avg_sal:,.0f}")

conn.close()
```

---

## 25. Virtual Environments & pip

Always use virtual environments to keep project dependencies isolated.

```bash
# ─── Create a virtual environment ───
python3 -m venv myenv

# Activate it
source myenv/bin/activate     # macOS/Linux
myenv\Scripts\activate        # Windows

# You'll see (myenv) in your terminal prompt now

# ─── Install packages ───
pip install requests
pip install flask sqlalchemy pandas
pip install numpy==1.24.0     # Specific version

# ─── See installed packages ───
pip list
pip show requests              # Detailed info about one package

# ─── Save requirements ───
pip freeze > requirements.txt  # Save all installed packages

# ─── Install from requirements.txt ───
pip install -r requirements.txt  # Reproduce exact environment

# ─── Deactivate ───
deactivate

# Common packages by use case:
# Web:        flask, fastapi, django, requests, httpx
# Data:       pandas, numpy, matplotlib, seaborn
# Database:   sqlalchemy, psycopg2, pymongo, redis
# AI/ML:      scikit-learn, tensorflow, pytorch, transformers
# DevOps:     boto3 (AWS), kubernetes, docker
# Testing:    pytest, unittest, coverage
# Utilities:  python-dotenv, pydantic, rich, click
```

---

## 26. Real-World Project — Build a REST API

Let's build a complete REST API for a Todo app using Flask.

```bash
pip install flask
```

```python
# app.py — Complete Todo REST API

from flask import Flask, request, jsonify
from datetime import datetime
import uuid

app = Flask(__name__)

# In-memory database (in production use SQLite/PostgreSQL)
todos = {}

def create_response(data=None, message="", status=200, error=None):
    """Standardized API response format."""
    response = {
        "status": "success" if not error else "error",
        "message": message,
        "timestamp": datetime.now().isoformat()
    }
    if data is not None:
        response["data"] = data
    if error:
        response["error"] = error
    return jsonify(response), status

# ─── GET all todos ───
@app.route("/api/todos", methods=["GET"])
def get_todos():
    status_filter = request.args.get("status")  # ?status=completed
    result = list(todos.values())

    if status_filter:
        result = [t for t in result if t["status"] == status_filter]

    # Sort by created_at
    result.sort(key=lambda x: x["created_at"], reverse=True)
    return create_response(data=result, message=f"Found {len(result)} todos")

# ─── GET single todo ───
@app.route("/api/todos/<todo_id>", methods=["GET"])
def get_todo(todo_id):
    todo = todos.get(todo_id)
    if not todo:
        return create_response(error="Todo not found", status=404)
    return create_response(data=todo)

# ─── CREATE a todo ───
@app.route("/api/todos", methods=["POST"])
def create_todo():
    body = request.get_json()
    if not body or not body.get("title"):
        return create_response(error="Title is required", status=400)

    todo_id = str(uuid.uuid4())[:8]
    todo = {
        "id": todo_id,
        "title": body["title"].strip(),
        "description": body.get("description", ""),
        "status": "pending",
        "priority": body.get("priority", "medium"),
        "created_at": datetime.now().isoformat(),
        "updated_at": datetime.now().isoformat()
    }
    todos[todo_id] = todo
    return create_response(data=todo, message="Todo created", status=201)

# ─── UPDATE a todo ───
@app.route("/api/todos/<todo_id>", methods=["PUT"])
def update_todo(todo_id):
    todo = todos.get(todo_id)
    if not todo:
        return create_response(error="Todo not found", status=404)

    body = request.get_json() or {}
    allowed_fields = {"title", "description", "status", "priority"}

    for field in allowed_fields:
        if field in body:
            todo[field] = body[field]

    todo["updated_at"] = datetime.now().isoformat()
    return create_response(data=todo, message="Todo updated")

# ─── DELETE a todo ───
@app.route("/api/todos/<todo_id>", methods=["DELETE"])
def delete_todo(todo_id):
    if todo_id not in todos:
        return create_response(error="Todo not found", status=404)
    deleted = todos.pop(todo_id)
    return create_response(data={"id": deleted["id"]}, message="Todo deleted")

# ─── Stats endpoint ───
@app.route("/api/todos/stats", methods=["GET"])
def get_stats():
    all_todos = list(todos.values())
    stats = {
        "total": len(all_todos),
        "pending": len([t for t in all_todos if t["status"] == "pending"]),
        "in_progress": len([t for t in all_todos if t["status"] == "in_progress"]),
        "completed": len([t for t in all_todos if t["status"] == "completed"]),
        "by_priority": {
            "high":   len([t for t in all_todos if t["priority"] == "high"]),
            "medium": len([t for t in all_todos if t["priority"] == "medium"]),
            "low":    len([t for t in all_todos if t["priority"] == "low"]),
        }
    }
    return create_response(data=stats)

if __name__ == "__main__":
    app.run(debug=True, port=5000)
```

**Test your API:**
```bash
# Start the server
python3 app.py

# In another terminal:

# Create a todo
curl -X POST http://localhost:5000/api/todos \
  -H "Content-Type: application/json" \
  -d '{"title": "Learn Python", "priority": "high"}'

# Get all todos
curl http://localhost:5000/api/todos

# Update a todo
curl -X PUT http://localhost:5000/api/todos/abc12345 \
  -H "Content-Type: application/json" \
  -d '{"status": "completed"}'
```

---

## 27. Python Best Practices & Tips

### Code Style (PEP 8)

```python
# ✅ GOOD
def calculate_monthly_salary(annual_salary, months=12):
    """Calculate monthly salary from annual salary."""
    return annual_salary / months

MAX_RETRY_COUNT = 3         # UPPER_CASE for constants
class UserAccount:          # PascalCase for classes
    user_name = "alice"     # snake_case for variables & functions

# ❌ BAD
def CalcMonthlySalary(AnnualSalary):
    return AnnualSalary/12

maxretrycount = 3
```

### Common Pythonic Patterns

```python
# ✅ Use enumerate instead of manual index
for i, item in enumerate(items):
    print(f"{i}: {item}")

# ✅ Use zip to pair lists
names = ["Alice", "Bob", "Carol"]
scores = [95, 87, 91]
for name, score in zip(names, scores):
    print(f"{name}: {score}")

# ✅ Use get() for dicts (avoids KeyError)
data = {"name": "Alice"}
age = data.get("age", "Unknown")   # Default if missing

# ✅ Use 'in' to check membership
if "admin" in user_roles:
    grant_access()

# ✅ Use ternary for simple if-else
label = "Pass" if score >= 60 else "Fail"

# ✅ Use with for file handling (auto-closes)
with open("file.txt") as f:
    content = f.read()

# ✅ Use list comprehension over loops for simple transforms
squares = [x**2 for x in range(10)]

# ✅ Unpack tuples clearly
lat, lng = (28.6, 77.2)

# ✅ Use f-strings (not + or .format for most cases)
print(f"Hello, {name}!")

# ✅ Use _ for unused variables
for _ in range(5):
    do_something()

first, *rest, last = [1, 2, 3, 4, 5]

# ✅ Use any() and all()
has_admin = any(r == "admin" for r in roles)
all_passed = all(s >= 60 for s in scores)
```

### Debugging Tips

```python
# print debugging (basic)
print(f"DEBUG: {variable = }")   # Python 3.8+ shows variable name too!

# Using breakpoint() (Python 3.7+)
def buggy_function(data):
    result = process(data)
    breakpoint()    # Drops into interactive debugger here
    return result

# Logging instead of print (production code)
import logging
logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)

logger.debug("Starting process")
logger.info("User logged in")
logger.warning("High memory usage")
logger.error("Database connection failed")
logger.critical("System crash!")
```

---

## 28. Learning Roadmap & Next Steps

### Your Python Journey

```
LEVEL 1 — BEGINNER (Weeks 1-4)
  ✅ Variables, Data Types, Strings
  ✅ Control Flow (if/elif/else)
  ✅ Loops (for, while)
  ✅ Functions
  ✅ Lists, Dicts, Tuples, Sets
  ✅ File Handling
  ✅ Error Handling

LEVEL 2 — INTERMEDIATE (Weeks 5-10)
  ✅ OOP (Classes, Inheritance, Polymorphism)
  ✅ Modules & Packages
  ✅ List Comprehensions & Generators
  ✅ Decorators
  ✅ Regular Expressions
  ✅ Working with APIs
  ✅ JSON & CSV handling
  ✅ SQLite / Databases

LEVEL 3 — ADVANCED (Weeks 11-20)
  ⬜ Context Managers
  ⬜ Threading & Multiprocessing
  ⬜ Async/Await (asyncio)
  ⬜ Type Hints & mypy
  ⬜ Testing with pytest
  ⬜ Design Patterns
  ⬜ Performance & Profiling
  ⬜ Memory Management

SPECIALIZATION PATHS (Month 4+):
  🌐 Web Dev    → Flask/FastAPI/Django
  📊 Data Sci   → Pandas/NumPy/Matplotlib
  🤖 ML/AI      → scikit-learn/TensorFlow/PyTorch
  ⚙️  DevOps     → Ansible/boto3/Docker SDK
  🔒 Security   → Penetration testing tools
```

### Top Learning Resources

| Resource | Type | Best For |
|---|---|---|
| [docs.python.org](https://docs.python.org) | Official Docs | Reference, latest features |
| [realpython.com](https://realpython.com) | Tutorials | Practical, project-based |
| [leetcode.com](https://leetcode.com) | Practice | Coding interviews |
| [kaggle.com](https://kaggle.com) | Notebooks | Data science projects |
| [github.com](https://github.com) | Projects | Reading real code |
| Python Crash Course (book) | Book | Complete beginners |
| Fluent Python (book) | Book | Intermediate → Advanced |

### Practice Projects by Level

```
BEGINNER PROJECTS:
  1. Calculator with all operations
  2. Guess the number game
  3. To-do list with file persistence
  4. Password generator
  5. Unit converter (km↔miles, °C↔°F)

INTERMEDIATE PROJECTS:
  1. Web scraper (BeautifulSoup + requests)
  2. Weather app using OpenWeatherMap API
  3. Student management system with SQLite
  4. File organizer (auto-sort downloads folder)
  5. WhatsApp/Email automation

ADVANCED PROJECTS:
  1. REST API with authentication (FastAPI)
  2. Real-time chat app (WebSockets)
  3. ML model — house price predictor
  4. Web dashboard with Streamlit
  5. Automated trading bot
  6. CLI tool published to PyPI
```

---

> 💡 **Golden Rule of Learning Python:**
> Read code → Write code → Break code → Fix code → Repeat.
> Build real projects as early as possible. Theory without practice fades in days.

---

*Python Zero to Hero Guide · Covers Python 3.8+ · Last updated 2026*
*Happy Coding! 🐍*
