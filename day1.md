# Day 1 – Python Fundamentals & Problem Solving

**Placement Preparation Series | Python Developer**

## Overview

This assignment focuses on Python fundamentals, built-in data structures, functions, problem-solving techniques, and debugging.

The objective is to understand Python concepts practically and apply them to solve common programming problems.

### Topics Covered

* Python Fundamentals
* Variables and Data Types
* Mutable vs Immutable Objects
* Conditions and Loops
* Strings and String Operations
* Lists, Tuples, Sets, and Dictionaries
* Functions and Scope
* Time and Space Complexity
* Dictionary-Based Problem Solving
* Debugging Techniques

---

# Task A – First Non-Repeating Character

### Problem Statement

Write a Python function `first_non_repeating(text)` that returns the first character that appears only once in a given string.

### Requirements

* Ignore spaces.
* Preserve case sensitivity.
* Use a dictionary for frequency counting.
* Do not use `count()` inside a loop.
* Return a meaningful message if no unique character exists.
* Include at least five test cases.

### Approach

I used a dictionary to solve this problem efficiently.

The solution follows two passes:

**Pass 1 – Frequency Counting**

* Create an empty dictionary.
* Iterate through the input string.
* Ignore spaces.
* Count the frequency of each character using `dict.get()`.

**Pass 2 – Find the First Unique Character**

* Iterate through the original string again.
* Skip spaces.
* Check whether the character's frequency is equal to 1.
* Return the first matching character.
* If no unique character exists, return a meaningful message.

### Python Code

```python
def first_non_repeating(text):

    # Dictionary to store character frequencies
    freq = {}

    # First pass: Count character occurrences
    for char in text:

        if char != " ":
            freq[char] = freq.get(char, 0) + 1

    # Second pass: Find first non-repeating character
    for char in text:

        if char != " " and freq[char] == 1:
            return char

    return "No non-repeating character found"


# Test Cases

print(first_non_repeating("programming"))
print(first_non_repeating("aabbcdd"))
print(first_non_repeating("aabbcc"))
print(first_non_repeating("hello world"))
print(first_non_repeating("Python"))
print(first_non_repeating(""))
print(first_non_repeating("   "))
print(first_non_repeating("AaBbC"))
```

### Expected Output

```text
p
c
No non-repeating character found
h
P
No non-repeating character found
No non-repeating character found
A
```

### Additional Explanation

Consider the input:

```python
text = "aabbcdd"
```

Character frequencies:

| Character | Frequency |
| --------- | --------: |
| a         |         2 |
| b         |         2 |
| c         |         1 |
| d         |         2 |

The first character with a frequency of 1 is `c`.

Therefore, the output is:

```text
c
```

### Complexity Analysis

* **Time Complexity:** O(n), because the string is traversed twice.
* **Space Complexity:** O(k), where k represents the number of distinct non-space characters stored in the dictionary.

**Conclusion:** Using a dictionary avoids repeatedly scanning the string and provides an efficient solution.

---

# Task B – Python Concepts and Data Structures

## 1. Comparison of List, Tuple, Set, and Dictionary

Python provides four commonly used built-in data structures.

### List

A list is an ordered and mutable collection that allows duplicate elements.

Example:

```python
students = ["Anil", "Rahul", "Kiran", "Anil"]

students.append("Arjun")

print(students)
```

Output:

```text
['Anil', 'Rahul', 'Kiran', 'Anil', 'Arjun']
```

**Characteristics:**

* Maintains insertion order.
* Allows duplicate values.
* Supports indexing and slicing.
* Mutable.
* Useful for storing collections of records.

### Tuple

A tuple is an ordered and immutable collection.

Example:

```python
coordinates = (10, 20)

print(coordinates[0])
print(coordinates[1])
```

Output:

```text
10
20
```

**Characteristics:**

* Maintains insertion order.
* Allows duplicate values.
* Supports indexing.
* Cannot be modified after creation.
* Useful for storing fixed data.

### Set

A set is a collection of unique and hashable elements.

Example:

```python
student_ids = {101, 102, 103, 101}

print(student_ids)
```

Output:

```text
{101, 102, 103}
```

**Characteristics:**

* Stores unique elements.
* Does not guarantee positional order.
* Does not support indexing.
* Supports fast membership checking on average.
* Useful for removing duplicates.

### Dictionary

A dictionary stores information in key-value pairs.

Example:

```python
student = {
    "id": 101,
    "name": "Anil",
    "course": "Python"
}

print(student["name"])
```

Output:

```text
Anil
```

**Characteristics:**

* Maintains insertion order of keys.
* Stores data as key-value pairs.
* Keys must be unique and hashable.
* Values can be duplicated.
* Useful for structured information and quick lookups.

## 2. Comparison Table

| Feature      | List         | Tuple      | Set                 | Dictionary     |
| ------------ | ------------ | ---------- | ------------------- | -------------- |
| Ordered      | Yes          | Yes        | No guaranteed order | Yes            |
| Duplicates   | Yes          | Yes        | No                  | Keys: No       |
| Mutable      | Yes          | No         | Yes                 | Yes            |
| Indexing     | Yes          | Yes        | No                  | Key-based      |
| Membership   | O(n)         | O(n)       | O(1) average        | O(1) average   |
| Main Purpose | Ordered data | Fixed data | Unique values       | Key-value data |

## 3. Scenario 1 – Store Unique Student IDs

**Problem:** We receive thousands of student IDs and need to identify duplicate registrations.

**Solution:** Use a set.

```python
student_ids = [101, 102, 103, 101, 104, 102]

seen = set()
duplicates = set()

for student_id in student_ids:

    if student_id in seen:
        duplicates.add(student_id)

    else:
        seen.add(student_id)

print("Unique IDs:", seen)
print("Duplicate IDs:", duplicates)
```

Expected output:

```text
Unique IDs: {101, 102, 103, 104}
Duplicate IDs: {101, 102}
```

**Why use a set?**

* Automatically stores unique elements.
* Provides average O(1) membership checking.
* Avoids repeatedly searching through a list.

**Backend Example:** Preventing duplicate student registrations in an admission system.

## 4. Scenario 2 – Store Student Profile Information

**Problem:** We need to store a student's name, email, and phone number.

**Solution:** Use a dictionary.

```python
student = {
    "id": 101,
    "name": "Anil",
    "email": "anil@example.com",
    "phone": "9876543210"
}

print("Student Name:", student["name"])
print("Email:", student["email"])

# Adding a new field
student["course"] = "Computer Science"

print(student)
```

Expected output:

```text
Student Name: Anil
Email: anil@example.com
```

**Why use a dictionary?**

* Stores information using meaningful keys.
* Makes accessing individual fields easy.
* Supports updating and adding information.

**Backend Example:** Storing and retrieving user profile information in a student management API.

### Conclusion

I would use:

* List when order and modification are important.
* Tuple when storing fixed collections.
* Set when uniqueness is required.
* Dictionary when information needs to be accessed using keys.

Choosing the correct data structure improves code readability and performance.

---

# Task C – Debugging Challenge

### Problem Statement

Consider the following code:

```python
students = ["Ravi", "Anil", "Kiran", "Suresh"]

for i in range(len(students)):

    if students[i] == "Kiran":
        print("Student found:", students[i])

    else:
        print("Student not found")
```

### Problem with the Original Code

The `else` block belongs to the `if` statement inside the loop.

Therefore, whenever an element does not match `"Kiran"`, the program prints `"Student not found"`.

Even if Kiran exists later in the list, the program prints incorrect messages for earlier elements.

### Incorrect Output

```text
Student not found
Student not found
Student found: Kiran
Student not found
```

### Corrected Code – Using for/else

```python
students = ["Ravi", "Anil", "Kiran", "Suresh"]

for student in students:

    if student == "Kiran":
        print("Student found:", student)
        break

else:
    print("Student not found")
```

### Expected Output – When Student Exists

```text
Student found: Kiran
```

### Test Case – When Student Does Not Exist

```python
students = ["Ravi", "Anil", "Suresh"]

for student in students:

    if student == "Kiran":
        print("Student found:", student)
        break

else:
    print("Student not found")
```

Expected output:

```text
Student not found
```

### Explanation

Python's `for/else` statement works as follows:

* The `for` loop iterates through the collection.
* If the required student is found, `break` terminates the loop.
* The `else` block executes only when the loop finishes without encountering `break`.

This ensures that the failure message is printed exactly once.

**Conclusion:** Using `for/else` makes the searching logic cleaner and avoids unnecessary repeated messages.

---

# Task D – Reflection

## 1. My Approach

I started by understanding the problem requirements and identifying the appropriate Python concepts and data structures.

For the first non-repeating character problem, I used a dictionary to count character frequencies and a second traversal to identify the first unique character.

For the data structure comparison, I studied the differences between lists, tuples, sets, and dictionaries and implemented practical examples.

For the debugging challenge, I identified the incorrect placement of the `else` block and corrected the logic using Python's `for/else` statement.

## 2. What I Learned

Through this assignment, I improved my understanding of:

* Python built-in data types.
* Mutable and immutable objects.
* Variables, operators, conditions, and loops.
* String manipulation and common methods.
* Lists, tuples, sets, and dictionaries.
* Functions and return statements.
* Dictionary-based frequency counting.
* Time and space complexity.
* Debugging using `break` and `for/else`.

## 3. What I Found Difficult

Initially, understanding how dictionary frequency counting works and how to identify the first non-repeating character was challenging.

I also needed to understand why the `else` block in a loop can behave differently from an ordinary `if/else` statement.

Working through the examples helped me understand these concepts more clearly.

## 4. Complexity Analysis

| Problem                       | Time Complexity | Space Complexity       |
| ----------------------------- | --------------- | ---------------------- |
| First non-repeating character | O(n)            | O(k)                   |
| Duplicate student IDs         | O(n) expected   | O(n)                   |
| Student profile lookup        | O(1) average    | O(n) for stored fields |
| Student search                | O(n)            | O(1)                   |

## 5. What I Would Improve

* Practise more dictionary-based coding problems.
* Improve my understanding of Python functions and scope.
* Solve problems without immediately looking at solutions.
* Practise explaining my approach before writing code.
* Test more edge cases before considering a solution complete.

---

# Additional Interview Preparation

### Important Python Concepts

| Concept        | Explanation                                             |
| -------------- | ------------------------------------------------------- |
| Dynamic Typing | Variable names can refer to objects of different types. |
| Mutable        | Objects that can be modified in place.                  |
| Immutable      | Objects that cannot be modified in place.               |
| `==`           | Checks value equality.                                  |
| `is`           | Checks object identity.                                 |
| `break`        | Terminates the nearest loop.                            |
| `continue`     | Skips the current iteration.                            |
| `pass`         | Placeholder statement that does nothing.                |
| `*args`        | Collects extra positional arguments into a tuple.       |
| `**kwargs`     | Collects extra keyword arguments into a dictionary.     |
| LEGB           | Local, Enclosing, Global, Built-in scope resolution.    |
| PEP 8          | Python coding style guidelines.                         |

### Important Interview Questions

**Q1. What is the difference between `==` and `is`?**

Answer: `==` compares values, whereas `is` checks whether two references point to the same object.

**Q2. Why are strings immutable?**

Answer: String contents cannot be modified in place. Operations such as `replace()` return a new string.

**Q3. What is the difference between `append()` and `extend()`?**

Answer: `append()` adds one object to a list, whereas `extend()` adds each element from an iterable.

**Q4. What is the difference between `sort()` and `sorted()`?**

Answer: `sort()` modifies a list in place and returns None. `sorted()` returns a new sorted list.

**Q5. What is `None` in Python?**

Answer: `None` represents the absence of a value. It should be checked using `is None`.

**Q6. What is a mutable default argument problem?**

Answer: Default arguments are created once when a function is defined. Using a mutable default such as `[]` can cause values to be shared between function calls.

**Q7. What is the LEGB rule?**

Answer: LEGB stands for Local, Enclosing, Global, and Built-in. It defines the order Python follows when resolving variable names.

**Q8. What is a virtual environment?**

Answer: A virtual environment isolates project dependencies so different projects can use different package versions.

---

# Technologies and Concepts Used

* Python 3
* Lists
* Tuples
* Sets
* Dictionaries
* Functions
* String Manipulation
* Loops and Conditions
* Dictionary Frequency Counting
* Time and Space Complexity

---

# Conclusion

This assignment helped me strengthen my Python fundamentals and understand how to approach common programming problems.

I implemented a first non-repeating character solution using dictionary frequency counting, compared Python's built-in data structures, and corrected a student-search debugging problem.

The most important thing I learned is that writing a correct solution is only one part of problem solving. Choosing the right data structure, understanding complexity, testing edge cases, and explaining the approach are equally important.

---

**Placement Preparation Series | Day 1**

*Python Fundamentals & Problem Solving*

### Assignment Submission


