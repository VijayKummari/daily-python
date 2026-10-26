# Day 1 — Python Fundamentals & Problem Solving

**Placement Preparation Series | Python Developer Interview Notes**

## Overview

This document covers my Day 1 learning of Python fundamentals, data structures, functions, problem-solving techniques, interview questions, and coding practice.

My approach is to **Learn → Attempt → Debug → Understand → Explain → Improve**.

For every problem, I try to understand the requirements, choose the appropriate data structure, write clean code, test different cases, and explain the time and space complexity.

---

# Part 1: Python Fundamentals

## 1. What Is Python?

Python is a high-level, general-purpose programming language known for its simple and readable syntax. It is widely used in backend development, automation, data analysis, scripting, and artificial intelligence.

## 2. Common Built-in Data Types

* **Numeric:** `int`, `float`, `complex`
* **Text:** `str`
* **Boolean:** `bool`
* **Collections:** `list`, `tuple`, `set`, `dict`
* **Absence of a value:** `None`

## 3. Dynamic Typing

Python determines an object's type at runtime. We don't need to declare the data type of a variable explicitly.

```python
value = 10
value = "Ten"

print(value)
# Output: Ten
```

## 4. Mutable vs. Immutable Objects

Mutable objects can be modified after creation. Examples include lists, dictionaries, and sets.

Immutable objects cannot be changed in place. Examples include integers, floats, booleans, strings, and tuples.

```python
items = [1, 2]
items.append(3)

print(items)
# Output: [1, 2, 3]

name = "Anil"
name = name + " K"

print(name)
# Output: Anil K
```

A tuple is immutable at the container level, but it can contain mutable objects.

## 5. Variables, Type Conversion, and Operators

Python variables should have meaningful names, such as `total_price`. Python identifiers are case-sensitive.

Examples of type conversion:

```python
int("12")
str(25)
float("3.5")
```

Invalid conversions can raise a `ValueError`.

Common operators include:

* Arithmetic: `+`, `-`, `*`, `/`, `//`, `%`, `**`
* Comparison: `==`, `!=`, `<`, `>`
* Logical: `and`, `or`, `not`
* Membership: `in`, `not in`
* Identity: `is`, `is not`
* Assignment: `=`, `+=`

Remember that `==` compares values, while `is` checks object identity.

## 6. Conditions and Loops

We use `if`, `elif`, and `else` to make decisions.

```python
score = 76

if score >= 90:
    grade = "A"
elif score >= 70:
    grade = "B"
else:
    grade = "C"

print(grade)
# Output: B
```

A `for` loop iterates over an iterable, while a `while` loop runs as long as its condition remains true.

```python
count = 3

while count > 0:
    print(count)
    count -= 1
```

Loop control statements:

* `break`: Exits the nearest loop.
* `continue`: Skips the current iteration.
* `pass`: Acts as a placeholder and does nothing.

## 7. Truthy and Falsy Values

Common falsy values include `False`, `None`, `0`, `0.0`, empty strings, empty lists, empty tuples, empty sets, and empty dictionaries.

For example:

```python
items = []

if items:
    print("Items available")
else:
    print("No items available")
```

Output:

```text
No items available
```

## 8. Strings and Common Operations

Strings are ordered sequences of Unicode characters and are immutable.

```python
word = "python"

print(word[0])     # p
print(word[-1])    # n
print(word[1:4])   # yth
print(word[::-1])  # nohtyp
```

Useful string methods:

* `strip()`: Removes whitespace from the beginning and end.
* `lower()`: Converts characters to lowercase.
* `upper()`: Converts characters to uppercase.
* `split()`: Splits a string into a list.
* `join()`: Combines strings.
* `replace()`: Replaces a substring.
* `startswith()`: Checks whether a string begins with a particular prefix.

String methods generally return new strings rather than modifying the original string.

---

# Part 2: Python Data Structures

| Feature    | List                | Tuple                   | Set                           | Dictionary                |
| ---------- | ------------------- | ----------------------- | ----------------------------- | ------------------------- |
| Ordered    | Yes                 | Yes                     | No positional order guarantee | Preserves insertion order |
| Duplicates | Allowed             | Allowed                 | Not allowed                   | Keys must be unique       |
| Mutable    | Yes                 | No, shallowly immutable | Yes                           | Yes                       |
| Main use   | Ordered collections | Fixed groups of values  | Unique values                 | Key-value storage         |

## 1. List

A list is an ordered, mutable collection.

```python
a = [1, 2]
a.append([3, 4])

print(a)
# [1, 2, [3, 4]]

b = [1, 2]
b.extend([3, 4])

print(b)
# [1, 2, 3, 4]
```

`append()` adds one object, whereas `extend()` adds each element from an iterable.

## 2. Tuple

A tuple is an ordered, immutable collection. It is useful for fixed values such as coordinates.

```python
point = (10, 20)

print(point[0])
# Output: 10
```

A tuple can be used as a dictionary key only when all its contents are hashable.

## 3. Set

A set stores unique hashable elements. It is useful for removing duplicates and checking membership.

```python
student_ids = {101, 102, 101}

print(student_ids)
# Contains 101 and 102
```

Sets do not support positional indexing.

## 4. Dictionary

A dictionary stores data as key-value pairs.

```python
student = {
    "name": "Ravi",
    "email": "r@example.com"
}

student["phone"] = "9999999999"

print(student.get("grade", "Not assigned"))
```

Dictionaries are useful for student profiles, configuration, lookup tables, and frequency counting.

---

# Part 3: Functions, Scope, and Complexity

## 1. Functions

A function is a reusable block of code that performs a particular task.

```python
def add(a, b=0):
    return a + b

print(add(4, 5))
# Output: 9

print(add(4))
# Output: 4
```

Parameters are defined in the function, while arguments are the values passed when calling it.

## 2. `*args` and `**kwargs`

`*args` collects extra positional arguments into a tuple, while `**kwargs` collects extra keyword arguments into a dictionary.

```python
def describe(*args, **kwargs):
    print(args)
    print(kwargs)

describe("Python", "SQL", level="beginner")
```

## 3. LEGB Scope Rule

Python searches for variables in this order:

1. Local
2. Enclosing
3. Global
4. Built-in

## 4. Mutable Default Argument Trap

Mutable default arguments can cause unexpected behavior because the same default object is reused across function calls.

A safer approach is to use `None`.

```python
def add_item(item, items=None):
    if items is None:
        items = []

    items.append(item)
    return items
```

## 5. Big O Notation

Big O describes how an algorithm's time or memory requirements grow with the input size.

* `O(1)`: Constant time
* `O(log n)`: Logarithmic time
* `O(n)`: Linear time
* `O(n log n)`: Common in efficient sorting algorithms
* `O(n²)`: Often occurs with nested scans

For example, counting characters using a dictionary takes expected `O(n)` time and `O(k)` additional space, where `k` is the number of distinct characters.

---

# Part 4: Python Interview Questions and Answers

## Section A: Fundamentals

### Q1. Is Python dynamically typed?

Yes. Python determines the type of an object at runtime, and a variable name can refer to objects of different types at different times.

### Q2. Is Python strongly typed?

Yes, Python is generally considered strongly typed. It does not automatically combine incompatible types in many operations. For example, `'2' + 2` raises a `TypeError`.

### Q3. What is the difference between `=` and `==`?

The `=` operator assigns a value to a variable, while `==` compares two values for equality.

### Q4. What is the difference between `==` and `is`?

`==` checks whether two values are equal. `is` checks whether two references point to the same object.

### Q5. Why are strings immutable?

Strings cannot be modified in place after creation. Operations such as `replace()` return a new string instead of changing the original one.

### Q6. What does the `//` operator do?

It performs floor division, rounding the result down toward negative infinity.

### Q7. What does the `%` operator do?

It returns the remainder after division.

### Q8. What does `range(5)` produce?

It represents the integers from `0` to `4`. The ending value is excluded.

### Q9. What is the purpose of `enumerate()`?

It provides both the index and the value when iterating over a sequence.

### Q10. What is the purpose of `zip()`?

It combines elements from multiple iterables and stops when the shortest iterable is exhausted by default.

### Q11. What is `None` in Python?

`None` represents the absence of a value. We generally use `is None` to check for it.

### Q12. What is a keyword argument?

A keyword argument is passed using the parameter name, such as `timeout=5`.

### Q13. What is a docstring?

A docstring is a string used to document the purpose of a module, function, or class.

### Q14. What is PEP 8?

PEP 8 is Python's style guide. It recommends conventions for writing readable and consistent code.

### Q15. Why do we use a virtual environment?

A virtual environment isolates project dependencies and helps prevent package-version conflicts between projects.

## Section B: Data Structures and Functions

### Q16. When would you choose a set?

I would use a set when I need unique values or efficient average-case membership checks. For example, I would use a set to track unique student IDs.

### Q17. Can a list be used as a dictionary key?

No. Lists are mutable and unhashable. A suitable immutable object, such as a tuple containing hashable values, can be used instead.

### Q18. Can a tuple contain a list?

Yes. However, the list inside the tuple can still be modified even though the tuple itself cannot be reassigned at the element level.

### Q19. What is the difference between `append()` and `extend()`?

`append()` adds one object to a list, whereas `extend()` adds the elements of another iterable.

### Q20. What is the difference between `remove()` and `pop()`?

`remove(value)` deletes the first matching value. `pop(index)` removes and returns an element at the specified index, defaulting to the last element.

### Q21. What does `dict.get()` do?

It retrieves a value by key. If the key does not exist, it returns a default value instead of raising a `KeyError`.

### Q22. What is a list comprehension?

A list comprehension is a concise way to create a list from an iterable, optionally applying a condition.

### Q23. What is a lambda function?

A lambda is a small anonymous function containing a single expression. It is useful for simple operations.

### Q24. What is `*args`?

It collects extra positional arguments into a tuple.

### Q25. What is `**kwargs`?

It collects extra keyword arguments into a dictionary.

### Q26. What is a default parameter?

A default parameter has a predefined value that is used when the caller does not provide an argument.

### Q27. What is a closure?

A closure is an inner function that retains access to variables from its enclosing scope, even after the outer function has finished executing.

### Q28. What does `return` do?

It ends a function call and sends a result back to the caller.

### Q29. What happens if a function has no `return` statement?

Python automatically returns `None`.

### Q30. Why should we avoid using `count()` inside a loop for character frequencies?

Repeated calls to `count()` can scan the same string multiple times, potentially resulting in `O(n²)` time complexity. A dictionary allows us to count characters in one pass using expected `O(n)` time.

## Section C: Advanced Python Questions

### Q31. Why is dictionary lookup usually efficient?

Python dictionaries use hashing to locate values by their keys. Lookup takes `O(1)` average-case time, although unusual cases can affect performance.

### Q32. What makes an object hashable?

A hashable object has a hash value that remains stable during its lifetime, with equality behavior compatible with hashing. Strings and integers are common examples.

### Q33. What is the difference between shallow copy and deep copy?

A shallow copy creates a new outer object but keeps references to nested objects. A deep copy recursively copies nested objects where supported.

### Q34. What is an exception?

An exception is an event that interrupts normal program execution. We can handle expected exceptions to provide a controlled response.

### Q35. Why do we use `try` and `except`?

We use them to handle expected runtime errors. For example, we can catch a `ValueError` when converting invalid input into an integer.

### Q36. What is the purpose of `finally`?

The `finally` block normally executes whether an exception occurs or not. It is often used for cleanup operations.

### Q37. What is the difference between a module and a package?

A module is a Python file containing code. A package organizes related modules under a common namespace.

### Q38. What is pip?

Pip is a package installer commonly used to install Python libraries and dependencies.

### Q39. Why do we use `requirements.txt`?

It records the dependencies required by a project so that another environment can install compatible packages.

### Q40. What is the LEGB rule?

LEGB stands for Local, Enclosing, Global, and Built-in. It describes the order Python follows when resolving names.

### Q41. Why can mutable default arguments cause problems?

The same default object is reused across function calls. If that object is modified, the changes may affect subsequent calls.

### Q42. What is the difference between `sorted()` and `list.sort()`?

`sorted()` returns a new sorted list, while `list.sort()` modifies the existing list in place and returns `None`.

### Q43. What is an iterator?

An iterator produces values one at a time using Python's iterator protocol. We commonly use `iter()` and `next()` when working with iterators.

### Q44. What would you do if you didn't know the answer to an interview question?

I would honestly explain what I know, reason through a small example, and describe how I would verify the answer instead of guessing blindly.

---

# Part 5: Scenario-Based Interview Questions

## Scenario 1: Detect Duplicate Student IDs

**Question:** You receive thousands of student IDs and need to identify duplicates. What would you use?

**Answer:**

I would use a set to track the IDs I have already seen. If an ID is already in the set, I would report it as a duplicate. Otherwise, I would add it to the set.

This approach takes expected `O(n)` time for `n` IDs.

## Scenario 2: Store Student Profile Information

**Question:** You need to store a student's name, email, and phone number. Which data structure would you choose?

**Answer:**

I would use a dictionary because student details naturally fit key-value pairs. For example, I can use `name`, `email`, and `phone` as keys. I would also validate required fields before using the data.

## Scenario 3: Find the First Unique Character

**Question:** Find the first non-repeating character in `aabbcdd`.

**Answer:**

I would count the frequency of every character using a dictionary. Then I would scan the original string and return the first character whose frequency is one.

The answer is `c`.

## Scenario 4: Fix a Search Program That Prints "Not Found" Repeatedly

**Question:** A program prints "not found" for every nonmatching list item. How would you fix it?

**Answer:**

The problem occurs when the failure message is placed inside the loop. I would move it outside the loop or use `for...else`. This ensures that the message appears only when the complete search finishes without finding a match.

## Scenario 5: Handle Blank Input in an API

**Question:** How would you handle an API request that contains blank input?

**Answer:**

I would first decide how the API should handle empty input and whether whitespace should be ignored. Then I would validate the input and return an appropriate response instead of processing invalid data.

## Scenario 6: Improve a Slow Duplicate-Checking Algorithm

**Question:** Your code uses nested loops to detect duplicates. How would you improve it?

**Answer:**

Nested loops can result in `O(n²)` time complexity. If I only need to check uniqueness or membership, I would use a set, which generally allows me to solve the problem in expected `O(n)` time.

## Scenario 7: Remove Duplicates While Preserving Order

**Question:** How would you remove duplicates without changing the original order?

**Answer:**

I would use a list to store the output and a set to track values already seen. Whenever I encounter a new value, I add it to both structures. This preserves the original order while avoiding duplicates.

## Scenario 8: A Function Changes the Caller's List

**Question:** Why can a function modify a list passed to it?

**Answer:**

Lists are mutable, so in-place changes made inside a function can be visible to the caller. If I want to avoid this, I can copy the list or return a new list instead.

---

# Part 6: Coding Challenges

## Problem 1: First Non-Repeating Character

### Requirements

* Ignore spaces.
* Preserve case sensitivity.
* Use a dictionary.
* Avoid `count()` inside a loop.
* Return a clear message when no unique character exists.

### Solution

```python
def first_non_repeating(text):
    text = text.replace(" ", "")
    freq = {}

    for char in text:
        freq[char] = freq.get(char, 0) + 1

    for char in text:
        if freq[char] == 1:
            return char

    return "No non-repeating character found"


print(first_non_repeating("programming"))
print(first_non_repeating("aabbcdd"))
print(first_non_repeating("aabbcc"))
print(first_non_repeating("x"))
print(first_non_repeating(""))
print(first_non_repeating("Aa"))
```

### Expected Output

```text
p
c
No non-repeating character found
x
No non-repeating character found
A
```

### Explanation

First, I remove spaces and create a dictionary to count the frequency of each character. Then I iterate through the string again and return the first character whose frequency is one.

If no such character exists, I return a message indicating that there is no non-repeating character.

### Complexity

* **Time:** `O(n)` expected.
* **Space:** `O(k)`, where `k` is the number of distinct characters.

## Problem 2: Debug the Student-Search Program

### Incorrect Code

```python
students = ["Ravi", "Anil", "Kiran", "Suresh"]

for i in range(len(students)):
    if students[i] == "Kiran":
        print("Student found:", students[i])
    else:
        print("Student not found")
```

### Corrected Code

```python
students = ["Ravi", "Anil", "Kiran", "Suresh"]

for student in students:
    if student == "Kiran":
        print("Student found:", student)
        break
else:
    print("Student not found")
```

### Explanation

In the original code, the `else` belongs to the `if` statement inside the loop. Therefore, it prints "Student not found" every time an individual student doesn't match.

In the corrected version, the `else` belongs to the `for` loop. It runs only if the loop finishes without executing `break`.

## Problem 3: Character Frequency Counter

```python
def character_counts(text):
    counts = {}

    for char in text:
        if char != " ":
            counts[char] = counts.get(char, 0) + 1

    return counts


print(character_counts("banana"))
```

Output:

```text
{'b': 1, 'a': 3, 'n': 2}
```

### Explanation

I use a dictionary to store each character and its frequency. Whenever a character appears, I increase its count. If it hasn't appeared before, I start its count at one. Spaces are ignored as required.

---

# Part 7: Additional Coding Questions to Practise

1. **First non-repeating character:** Use dictionary frequency counting and a second pass.
2. **Character or word frequency:** Use a dictionary counter and clarify whether spaces and case matter.
3. **Remove duplicates while preserving order:** Use a set to track seen values and a list for output.
4. **Find the second-largest distinct number:** Track the largest and second-largest values or sort the distinct values.
5. **Check whether a string is a palindrome:** Normalize the input if required and compare it with its reverse or use two pointers.
6. **Two Sum:** Use a dictionary to store previously seen values and find the required complement.
7. **Flatten a nested list:** Clarify the required nesting depth; arbitrary nesting may require recursion.
8. **Debug a search loop:** Use a found flag or `for...else` to display the correct result.

---

# Part 8: Day 1 Assignment

## Task A — Coding

Implement `first_non_repeating(text)` using a dictionary.

Requirements:

* Ignore spaces.
* Keep uppercase and lowercase characters distinct.
* Include at least five test cases.
* Handle empty input and cases where every character repeats.

## Task B — Concepts

Compare lists, tuples, sets, and dictionaries.

Explain which data structure you would choose for:

1. Storing unique student IDs.
2. Storing student profile information.

## Task C — Debugging

Fix the student-search loop so that it prints exactly one correct message whether or not `Kiran` exists.

Explain why the original output was incorrect.

## Task D — Reflection

### My Approach

I started by understanding the requirements and choosing the appropriate data structures. For the first non-repeating character problem, I used a dictionary to count character frequencies and then checked the original string to find the first unique character.

I also tested different inputs to verify that the solution worked correctly.

### What I Learned

I learned how to use dictionaries for frequency counting, when to choose lists, tuples, sets, and dictionaries, and how to debug loops using `break` and `for...else`.

I also understood the importance of testing edge cases and explaining time and space complexity.

### What I Found Difficult

Initially, understanding why the original student-search program printed "Student not found" multiple times was a little confusing. After reviewing the loop structure, I understood how the placement of the `else` block affected the output.

### What I Would Improve After Mentor Feedback

I would improve the readability of my code, practise more problem-solving questions, and test additional edge cases before submitting my solutions. I would also focus on explaining my approach clearly instead of relying only on memorized definitions.

---

# Part 9: Final Revision Checklist

* [ ] I can explain the main Python data types.
* [ ] I understand mutable and immutable objects.
* [ ] I can explain the difference between `==` and `is`.
* [ ] I know when to use lists, tuples, sets, and dictionaries.
* [ ] I can write functions and explain `*args` and `**kwargs`.
* [ ] I understand the LEGB rule and mutable default arguments.
* [ ] I can explain basic Big O complexity.
* [ ] I can answer Python fundamentals interview questions.
* [ ] I can explain practical scenarios involving Python data structures.
* [ ] I have tested my coding solutions with different inputs.
* [ ] I can explain my approach, time complexity, and space complexity.
* [ ] I can explain my solutions in my own words.

**Key takeaway:** My goal is not just to write code that works, but to understand why it works, test it properly, and explain my approach clearly during an interview.
