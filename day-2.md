# Day 2 – Python Problem Solving & Core Data Structures

**Placement Preparation Series | Python Developer**

## Overview

This assignment focuses on solving Python coding problems using appropriate data structures and algorithms.

The main objective is to write efficient, readable, and reusable Python code while understanding time complexity, space complexity, edge cases, and debugging techniques.

### Topics Covered

* Python Problem-Solving Approach
* Lists, Sets, and Dictionaries
* Duplicate Removal
* Two-Pointer Technique
* Sliding Window Algorithm
* Debugging and Edge Cases
* Time and Space Complexity

---

# Task A – Remove Duplicates While Preserving Order

### Problem Statement

Write a Python function that removes duplicate elements from a list while preserving the original order of their first occurrence.

**Condition:** Do not use a one-line shortcut.

### Approach

I used a combination of a set and a list to solve this problem.

* A set named `seen` keeps track of elements that have already appeared.
* A list named `result` stores unique elements in their original order.
* I iterate through the input list and check whether each element already exists in the set.
* If it is not present, I add it to both the set and the result list.

### Python Code

```python
def unique_in_order(items):

    seen = set()
    result = []

    for item in items:

        if item not in seen:
            seen.add(item)
            result.append(item)

    return result


# Test Cases

print(unique_in_order([3, 1, 3, 2, 1]))
print(unique_in_order([]))
print(unique_in_order([5]))
print(unique_in_order([2, 2, 2]))
print(unique_in_order([10, 20, 30, 10, 40, 20]))
print(unique_in_order(["apple", "banana", "apple", "orange"]))
```

### Expected Output

```text
[3, 1, 2]
[]
[5]
[2]
[10, 20, 30, 40]
['apple', 'banana', 'orange']
```

### Complexity Analysis

* **Time Complexity:** O(n) expected, because each element is visited once and set membership is O(1) on average.
* **Space Complexity:** O(n), because the set and result list store unique elements.

**Conclusion:** Using a set for membership checking and a list for maintaining order makes this solution efficient and readable.

Note: This implementation assumes all elements are hashable.

---

# Task B – Maximum Sum of a Contiguous Window

### Problem Statement

Write a Python function that returns the maximum sum of any contiguous subarray of size `k`.

The function should also validate invalid values of `k`.

### Approach – Sliding Window Technique

Instead of calculating the sum of every possible window separately, I used the sliding window technique.

Steps:

1. Check whether `k` is valid.
2. Calculate the sum of the first `k` elements.
3. Store this sum as the initial maximum.
4. Move the window one position at a time.
5. Subtract the element leaving the window.
6. Add the new element entering the window.
7. Update the maximum sum whenever a larger sum is found.

### Python Code

```python
def max_sum_window(numbers, k):

    # Validate window size
    if not isinstance(k, int) or isinstance(k, bool):
        return None

    if k <= 0 or k > len(numbers):
        return None

    # Calculate the first window sum
    window_sum = sum(numbers[:k])

    max_sum = window_sum

    # Slide the window
    for right in range(k, len(numbers)):

        window_sum += numbers[right] - numbers[right - k]

        max_sum = max(max_sum, window_sum)

    return max_sum


# Test Cases

print(max_sum_window([2, 1, 5, 1, 3, 2], 3))
print(max_sum_window([1, 2, 3, 4, 5], 2))
print(max_sum_window([5, 5, 5, 5], 2))
print(max_sum_window([-2, -1, -5, -3], 2))
print(max_sum_window([10], 1))
print(max_sum_window([], 1))
print(max_sum_window([1, 2, 3], 0))
print(max_sum_window([1, 2, 3], 5))
```

### Expected Output

```text
9
9
10
-3
10
None
None
None
```

### Example Explanation

Input:

```python
numbers = [2, 1, 5, 1, 3, 2]
k = 3
```

Possible windows:

| Window    | Sum |
| --------- | --: |
| [2, 1, 5] |   8 |
| [1, 5, 1] |   7 |
| [5, 1, 3] |   9 |
| [1, 3, 2] |   6 |

Maximum sum = **9**

### Complexity Analysis

* **Time Complexity:** O(n), because the initial window is calculated once and the remaining elements are processed in a single traversal.
* **Space Complexity:** O(1) auxiliary space, excluding the input.

### Edge Cases Tested

* Empty input list.
* Window size equal to 1.
* Window size equal to the list length.
* Window size greater than the list length.
* Window size equal to zero.
* Negative numbers.
* Repeated elements.

**Conclusion:** Sliding Window avoids recalculating the sum of every window and improves the solution from O(n × k) to O(n).

---

# Task C – Debugging Challenge

### Problem Statement

Consider the following code:

```python
numbers = [4, 8, 12]
target = 7

for number in numbers:

    if number == target:
        print("Found")

    else:
        print("Not found")
```

### Problem with the Original Code

The problem is that `"Not found"` is printed every time an element does not match the target.

For example, the output is:

```text
Not found
Not found
Not found
```

However, the expected behavior is to print `"Not found"` only once after checking the entire list.

### Corrected Code – Using for/else

```python
numbers = [4, 8, 12]
target = 7

for number in numbers:

    if number == target:
        print("Found")
        break

else:
    print("Not found")
```

### Expected Output

```text
Not found
```

### Another Test Case

```python
numbers = [4, 8, 12]
target = 8

for number in numbers:

    if number == target:
        print("Found")
        break

else:
    print("Not found")
```

Expected output:

```text
Found
```

### Explanation

Python's `for/else` statement executes the `else` block only when the loop completes without encountering a `break`.

* If the target is found, the loop breaks and prints `"Found"`.
* If the target is not found, the loop completes and prints `"Not found"` exactly once.

**Conclusion:** Using `for/else` avoids unnecessary repeated messages and makes the searching logic cleaner.

---

# Task D – Comparison of Python Data Structures

Python provides different data structures for storing and managing data.

The three commonly used data structures are List, Set, and Dictionary.

### 1. List

A list is an ordered and mutable collection that allows duplicate elements.

Example:

```python
students = ["Kiran", "Rahul", "Arjun", "Kiran"]

print(students)
print(students[0])
```

Output:

```text
['Kiran', 'Rahul', 'Arjun', 'Kiran']
Kiran
```

**Characteristics:**

* Maintains insertion order.
* Allows duplicate values.
* Supports indexing and slicing.
* Mutable.
* Membership checking takes O(n) time in the general case.

**Backend Example:**

Storing a list of products returned by an API.

```python
products = [
    {"id": 1, "name": "Laptop"},
    {"id": 2, "name": "Mobile"},
    {"id": 3, "name": "Keyboard"}
]
```

### 2. Set

A set is a collection of unique, hashable elements.

Example:

```python
user_ids = {101, 102, 103, 101}

print(user_ids)
print(102 in user_ids)
```

Output:

```text
{101, 102, 103}
True
```

**Characteristics:**

* Does not preserve insertion order as a guaranteed contract.
* Does not allow duplicate elements.
* Does not support indexing.
* Mutable.
* Membership checking takes O(1) time on average.

**Backend Example:**

Checking whether a user has already registered.

```python
registered_users = {101, 102, 103}

new_user = 102

if new_user in registered_users:
    print("User already registered")
else:
    registered_users.add(new_user)
    print("Registration successful")
```

Output:

```text
User already registered
```

### 3. Dictionary

A dictionary stores information in key-value pairs.

Example:

```python
student = {
    "id": 101,
    "name": "Kiran",
    "course": "Python"
}

print(student["name"])
print(student.get("course"))
```

Output:

```text
Kiran
Python
```

**Characteristics:**

* Maintains insertion order of keys.
* Stores data in key-value pairs.
* Keys must be unique and hashable.
* Values can contain duplicates.
* Membership checking by key takes O(1) time on average.

**Backend Example:**

Storing user information returned by an API.

```python
user = {
    "id": 101,
    "username": "kiran",
    "email": "kiran@example.com",
    "role": "developer"
}

print(user["username"])
```

Output:

```text
kiran
```

### Comparison Table

| Feature      | List            | Set                   | Dictionary            |
| ------------ | --------------- | --------------------- | --------------------- |
| Ordered      | Yes             | No guaranteed order   | Yes                   |
| Duplicates   | Allowed         | Not allowed           | Keys must be unique   |
| Indexing     | Supported       | Not supported         | Access using keys     |
| Mutable      | Yes             | Yes                   | Yes                   |
| Membership   | O(n)            | O(1) average          | O(1) average for keys |
| Main Purpose | Store sequences | Store unique elements | Store key-value data  |

### Conclusion

I would choose a list when order and duplicates matter, a set when uniqueness and fast membership checking are important, and a dictionary when data needs to be stored and accessed using keys.

Choosing the correct data structure helps improve application performance and code readability.

---

# Task E – Reflection

## 1. My Approach

While solving the problems, I first understood the requirements and identified the expected inputs and outputs.

Then I selected the appropriate data structures and algorithms based on the problem.

For duplicate removal, I used a set and a list.

For finding the maximum sum of a fixed-size window, I used the Sliding Window technique.

For the debugging challenge, I used Python's `for/else` statement to correct the searching logic.

## 2. Test Cases

I tested the solutions with:

* Empty lists.
* Single-element lists.
* Duplicate values.
* Negative numbers.
* Repeated elements.
* Invalid window sizes.
* Targets that exist and do not exist.

## 3. Time and Space Complexity

| Problem            | Time Complexity | Space Complexity |
| ------------------ | --------------- | ---------------- |
| Remove duplicates  | O(n) expected   | O(n)             |
| Maximum window sum | O(n)            | O(1) auxiliary   |
| Linear search      | O(n)            | O(1)             |

## 4. What I Found Difficult

Initially, understanding how the Sliding Window technique updates the sum without recalculating the entire window was challenging.

Understanding the behavior of Python's `for/else` statement was another interesting part of the debugging exercise.

## 5. What I Learned

Through this assignment, I improved my understanding of:

* Selecting appropriate Python data structures.
* Using sets for efficient duplicate detection.
* Maintaining insertion order while removing duplicates.
* Implementing the Sliding Window algorithm.
* Debugging loops using `for/else`.
* Identifying edge cases before writing solutions.
* Analyzing time and space complexity.

One important takeaway is that choosing the right algorithm and data structure can significantly improve the performance of a program.

---

# Key Takeaways

| Concept          | What I Learned                                     |
| ---------------- | -------------------------------------------------- |
| List             | Ordered collection that allows duplicates          |
| Set              | Stores unique elements                             |
| Dictionary       | Stores key-value pairs                             |
| Two Pointers     | Processes sequences using two positions            |
| Sliding Window   | Efficiently processes contiguous ranges            |
| for/else         | Executes else when the loop finishes without break |
| Time Complexity  | Measures how execution time grows                  |
| Space Complexity | Measures additional memory usage                   |

---

# Technologies Used

* Python 3
* Python Lists
* Python Sets
* Python Dictionaries
* Two-Pointer Technique
* Sliding Window Algorithm

---

# Conclusion

This assignment helped me strengthen my Python problem-solving skills and understand how different data structures and algorithms can be used to solve real-world problems.

I implemented duplicate removal, maximum window sum calculation, and debugging solutions while analyzing their time and space complexities.

The main lesson I learned is that writing code is not just about getting the correct output. It is also about choosing an efficient approach, handling edge cases, and writing clean and understandable solutions.

---

**Placement Preparation Series | Day 2**

*Python Problem Solving & Core Data Structures*

