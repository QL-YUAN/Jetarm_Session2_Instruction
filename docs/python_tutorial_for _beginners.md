# Beginner Python Programming Tutorial (With Run Instructions and Expected Output)

---

## 1. Installing Python

1. Verify installation:

```bash id="install-check"
python --version
```

Expected output (version may differ):

```
Python 3.11.2
```

---

## 2. Your First Program

**Step 1:** Open a text editor (VS Code, Notepad, or gedit).
**Step 2:** Type the following:

```python id="hello-py"
print("Hello, world!")
```

**Step 3:** Save as `hello.py`.
**Step 4:** Run in terminal/command prompt:

```bash id="run-hello"
python hello.py
```

**Expected Output:**

```
Hello, world!
```

---

## 3. Variables and Data Types

**Code (`variables.py`):**

```python id="variables"
name = "Alice"
age = 25
height = 5.6
is_student = True

print(name, age, height, is_student)
```

**Run:**

```bash id="run-variables"
python variables.py
```

**Expected Output:**

```
Alice 25 5.6 True
```

---

## 4. Lists and Tuples

**Code (`lists_tuples.py`):**

```python id="lists-tuples"
# List (mutable)
fruits = ["apple", "banana", "cherry"]
fruits.append("orange")
fruits.remove("banana")
print("List:", fruits)

# Tuple (immutable)
colors = ("red", "green", "blue")
print("Tuple:", colors)
print("First color:", colors[0])
```

**Run:**

```bash id="run-lists-tuples"
python lists_tuples.py
```

**Expected Output:**

```
List: ['apple', 'cherry', 'orange']
Tuple: ('red', 'green', 'blue')
First color: red
```

---

## 5. Dictionaries

**Code (`dict_example.py`):**

```python id="dict-example"
student = {"name": "Alice", "age": 25, "is_student": True}

print("Name:", student["name"])
student["age"] = 26
student["grade"] = "A"
print("Updated student:", student)
```

**Run:**

```bash id="run-dict"
python dict_example.py
```

**Expected Output:**

```
Name: Alice
Updated student: {'name': 'Alice', 'age': 26, 'is_student': True, 'grade': 'A'}
```

---

## 6. If Statements

**Code (`if_example.py`):**

```python id="if-example"
age = 20

if age >= 18:
    print("You are an adult.")
else:
    print("You are a minor.")
```

**Run:**

```bash id="run-if"
python if_example.py
```

**Expected Output:**

```
You are an adult.
```

---

## 7. Loops

**Code (`loops.py`):**

```python id="loops"
# For loop
for i in range(3):
    print("For loop:", i)

# While loop
count = 0
while count < 3:
    print("While loop:", count)
    count += 1
```

**Run:**

```bash id="run-loops"
python loops.py
```

**Expected Output:**

```
For loop: 0
For loop: 1
For loop: 2
While loop: 0
While loop: 1
While loop: 2
```

---

## 8. Functions

**Code (`functions.py`):**

```python id="functions"
def greet(name):
    return f"Hello, {name}!"

print(greet("Alice"))
print(greet("Bob"))
```

**Run:**

```bash id="run-functions"
python functions.py
```

**Expected Output:**

```
Hello, Alice!
Hello, Bob!
```

---

## 9. File Read/Write

**Code (`file_io.py`):**

```python id="file-io"
# Write to file
with open("example.txt", "w") as f:
    f.write("Hello, world!\n")
    f.write("Python is fun!\n")

# Read from file
with open("example.txt", "r") as f:
    content = f.read()
    print("File content:\n", content)
```

**Run:**

```bash id="run-file"
python file_io.py
```

**Expected Output:**

```
File content:
 Hello, world!
Python is fun!
```

---

## 10. Classes

**Code (`classes.py`):**

```python id="classes"
class Student:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def greet(self):
        print(f"Hello, my name is {self.name} and I am {self.age} years old.")

student1 = Student("Alice", 25)
student2 = Student("Bob", 22)

student1.greet()
student2.greet()
```

**Run:**

```bash id="run-classes"
python classes.py
```

**Expected Output:**

```
Hello, my name is Alice and I am 25 years old.
Hello, my name is Bob and I am 22 years old.
```

---

## 11. Mini Project: Student Records (Classes + Dict + File)

**Code (`student_records.py`):**

```python id="student-records"
class Student:
    def __init__(self, name, age, grade):
        self.name = name
        self.age = age
        self.grade = grade

    def to_dict(self):
        return {"name": self.name, "age": self.age, "grade": self.grade}

students = [
    Student("Alice", 25, "A"),
    Student("Bob", 22, "B")
]

# Save to file
with open("students.txt", "w") as f:
    for s in students:
        f.write(str(s.to_dict()) + "\n")

# Read from file
print("Student Records:")
with open("students.txt", "r") as f:
    for line in f:
        print(line.strip())
```

**Run:**

```bash id="run-student-records"
python student_records.py
```

**Expected Output:**

```
Student Records:
{'name': 'Alice', 'age': 25, 'grade': 'A'}
{'name': 'Bob', 'age': 22, 'grade': 'B'}
```

