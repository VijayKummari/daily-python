# Day 2 - Python Problem Solving & Core Data Structures 🐍

Hi everyone! 👋

This repository contains my notes, coding practice, interview questions, and assignments from **Day 2 of the Placement Preparation Series**.

In this session, I focused on Python lists, dictionaries, sets, problem-solving techniques, and time and space complexity. I also practiced some coding problems and learned how to handle edge cases and debug common mistakes.

I'm sharing these notes here for revision and practice.

## 📚 Topics Covered

* Python Lists and their operations
* Dictionaries and Sets
* Frequency Counting and Duplicate Detection
* Two-Pointer Technique
* Sliding-Window Technique
* Time and Space Complexity
* Coding Problems and Debugging
* Interview Questions and Answers
* Scenario-Based Questions
* Day 2 Assignments

---

## 1. Python Lists

### Q1. What is the difference between `append()` and `extend()`?

**Answer:**

`append()` adds one object to the end of a list, whereas `extend()` adds each element from an iterable to the list.

```python
numbers = [1, 2]

numbers.append([3, 4])
print(numbers)
# [1, 2, [3, 4]]

numbers = [1, 2]
numbers.extend([3, 4])
print(numbers)
# [1, 2, 3, 4]
```

### Q2. What is the difference between `sort()` and `sorted()`?

**Answer:**

* `sort()` sorts a list in place and returns `None`.
* `sorted()` returns a new sorted list without changing the original iterable.

```python
numbers = [3, 1, 2]

numbers.sort()
print(numbers)
# [1, 2, 3]

numbers = [3, 1, 2]
result = sorted(numbers)

print(result)
# [1, 2, 3]

print(numbers)
# [3, 1, 2]
```

### Q3. What is the difference between `remove()` and `pop()`?

**Answer:**

* `remove(value)` removes the first occurrence of a specified value.
* `pop(index)` removes and returns the element at the specified index. Without an index, it removes and returns the last element.

```python
numbers = [10, 20, 30, 20]

numbers.remove(20)
print(numbers)
# [10, 30, 20]

value = numbers.pop()
print(value)
# 20

print(numbers)
# [10, 30]
```

### Q4. What is a shallow copy in Python?

**Answer:**

A shallow copy creates a new outer object, but nested objects are still shared between the original and copied objects.

```python
original = [[1, 2], [3, 4]]
copied = original[:]

copied[0][0] = 100

print(original)
# [[100, 2], [3, 4]]
```

Here, changing a nested list through the copy also affects the original because both refer to the same nested object.

### Q5. What happens if we assign the result of `list.sort()` to a variable?

**Answer:**

The variable receives `None` because `sort()` modifies the list in place and does not return the sorted list.

```python
scores = [71, 95, 84]

result = scores.sort()

print(result)
# None

print(scores)
# [71, 84, 95]
```

If we need a new sorted list, we should use `sorted(scores)`.

---

## 2. Dictionaries and Sets

### Q6. Why do we use a set for duplicate detection?

**Answer:**

A set stores unique hashable elements and supports average-case O(1) membership checking. It helps us detect duplicates without repeatedly searching through a list.

```python
numbers = [10, 20, 30, 20]
seen = set()

for number in numbers:
    if number in seen:
        print("Duplicate found:", number)
        break

    seen.add(number)

# Output:
# Duplicate found: 20
```

### Q7. What is the difference between a list, a set, and a dictionary?

**Answer:**

| Feature    | List                | Set                                 | Dictionary                |
| ---------- | ------------------- | ----------------------------------- | ------------------------- |
| Stores     | Elements            | Unique elements                     | Key-value pairs           |
| Duplicates | Allowed             | Duplicate elements are not retained | Keys must be unique       |
| Order      | Preserves order     | No guaranteed iteration order       | Preserves insertion order |
| Membership | O(n)                | O(1) average                        | O(1) average for keys     |
| Common use | Ordered collections | Uniqueness and membership           | Lookup and counting       |

**Example:**

```python
numbers = [1, 2, 2, 3]
unique_numbers = {1, 2, 3}
student = {"name": "Kiran", "age": 22}
```

### Q8. What does `dict.get(key, default)` do?

**Answer:**

The `get()` method returns the value associated with a key. If the key does not exist, it returns the specified default value instead of raising a `KeyError`.

```python
student = {"name": "Kiran"}

print(student.get("name", "Unknown"))
# Kiran

print(student.get("age", 0))
# 0
```

### Q9. Can we use a list as a dictionary key?

**Answer:**

No. Lists are mutable and unhashable, so they cannot be used directly as dictionary keys or set elements.

Strings, integers, and tuples containing only hashable elements are common examples of valid keys.

```python
# Invalid:
# data = {[1, 2]: "numbers"}

# Valid:
data = {(1, 2): "numbers"}

print(data[(1, 2)])
# numbers
```

### Q10. Why can repeated list membership checks be expensive?

**Answer:**

Searching an unsorted list takes O(n) time in the worst case. If we perform membership checks repeatedly while processing n elements, the overall time can become O(n²).

Using a set can reduce the expected total processing time to O(n), at the cost of additional memory.

---

## 3. Two-Pointer Technique

### Q11. What is the two-pointer technique?

**Answer:**

The two-pointer technique uses two positions to process a sequence efficiently.

Depending on the problem, the pointers can start at opposite ends or move through the sequence in different ways.

It is useful for problems such as palindrome checking and finding a pair sum in a sorted list.

### Problem 1: Check Whether a String Is a Palindrome

**Question:** Write a Python function to check whether a string reads the same forward and backward, ignoring spaces and letter case.

**Solution:**

```python
def is_palindrome(text):
    cleaned = text.lower().replace(" ", "")

    left = 0
    right = len(cleaned) - 1

    while left < right:
        if cleaned[left] != cleaned[right]:
            return False

        left += 1
        right -= 1

    return True


print(is_palindrome("Never odd or even"))
# True

print(is_palindrome("Python"))
# False
```

**Explanation:**

1. Convert the string to lowercase.
2. Remove spaces.
3. Place one pointer at the beginning and another at the end.
4. Compare the characters.
5. If they differ, return `False`.
6. Otherwise, move both pointers toward the center.

**Complexity:**

* Time: O(n)
* Auxiliary space: O(n) because a cleaned string is created.

### Problem 2: Find a Pair With a Target Sum

**Question:** Given a sorted list and a target value, determine whether two elements add up to the target.

**Example:**

Input: `[1, 3, 4, 6, 8]`

Target: `10`

Output: `True`

**Solution:**

```python
def has_pair_sum_sorted(numbers, target):
    left = 0
    right = len(numbers) - 1

    while left < right:
        total = numbers[left] + numbers[right]

        if total == target:
            return True

        if total < target:
            left += 1
        else:
            right -= 1

    return False


print(has_pair_sum_sorted([1, 3, 4, 6, 8], 10))
# True
```

**Explanation:**

* If the sum equals the target, return `True`.
* If the sum is smaller, move the left pointer forward.
* If the sum is greater, move the right pointer backward.
* If no pair is found, return `False`.

**Complexity:**

* Time: O(n)
* Auxiliary space: O(1)

**Note:** This approach requires sorted input.

---

## 4. Sliding-Window Technique

### Q12. What is the sliding-window technique?

**Answer:**

The sliding-window technique processes a contiguous section of an array or string.

Instead of recalculating the entire section every time it moves, we update the current result by removing the outgoing element and adding the incoming element.

This can improve an O(n × k) solution to O(n) for fixed-size window-sum problems.

### Problem 3: Find the Maximum Sum of a Fixed-Size Window

**Question:** Given a list of numbers and a window size `k`, find the maximum sum of any `k` consecutive elements.

**Example:**

Input: `[2, 1, 5, 1, 3, 2]`

Window size: `3`

Possible window sums:

* `[2, 1, 5]` → 8
* `[1, 5, 1]` → 7
* `[5, 1, 3]` → 9
* `[1, 3, 2]` → 6

Output: `9`

**Solution:**

```python
def max_sum_window(numbers, k):
    if k <= 0 or k > len(numbers):
        return None

    window_sum = sum(numbers[:k])
    best = window_sum

    for right in range(k, len(numbers)):
        window_sum += numbers[right] - numbers[right - k]
        best = max(best, window_sum)

    return best


print(max_sum_window([2, 1, 5, 1, 3, 2], 3))
# 9

print(max_sum_window([1, 2, 3], 1))
# 3

print(max_sum_window([1, 2, 3], 4))
# None
```

**Explanation:**

1. Validate the window size.
2. Calculate the sum of the first window.
3. Subtract the element leaving the window.
4. Add the new element entering the window.
5. Keep track of the maximum sum.

**Complexity:**

* Time: O(n)
* Auxiliary space: O(1), excluding the temporary slice used to calculate the first window.

---

## 5. Time and Space Complexity

### Q13. What is time complexity?

**Answer:**

Time complexity describes how the amount of work performed by an algorithm grows as the input size increases.

Common examples:

| Complexity | Meaning           | Example                              |
| ---------- | ----------------- | ------------------------------------ |
| O(1)       | Constant time     | Accessing a list element by index    |
| O(n)       | Linear time       | Traversing a list once               |
| O(n²)      | Quadratic time    | Two nested loops over the same input |
| O(n log n) | Linearithmic time | Typical comparison-based sorting     |

### Q14. What is space complexity?

**Answer:**

Space complexity describes how memory requirements grow with the input size.

For example, removing duplicates while preserving order requires a set and a result list. Both may grow with the input size, so the auxiliary space complexity is O(n).

### Q15. What is the difference between time and auxiliary space complexity?

**Answer:**

Time complexity describes the growth in computational work, while auxiliary space complexity describes the additional memory used by the algorithm.

For example, a two-pointer pair-sum solution on a sorted list takes O(n) time and O(1) auxiliary space.

### Quick Complexity Reference

| Operation or Technique     | Typical Complexity           |
| -------------------------- | ---------------------------- |
| List indexing              | O(1)                         |
| Searching an unsorted list | O(n)                         |
| List membership            | O(n)                         |
| Set membership             | O(1) average                 |
| Dictionary key lookup      | O(1) average                 |
| Sorting a list             | O(n log n)                   |
| Single traversal           | O(n)                         |
| Nested full traversal      | O(n²)                        |
| Two pointers               | O(n) for common linear scans |
| Fixed-size sliding window  | O(n)                         |

---

## 6. Scenario-Based Interview Questions

### Scenario 1: Duplicate Registrations

**Question:**

A service receives thousands of registration IDs and must identify repeated IDs without comparing each ID against every previously received ID. What approach would you use?

**Answer:**

I would use a set called `seen` to store IDs that have already appeared.

For each new ID:

1. Check whether it exists in `seen`.
2. If it exists, flag the ID as a duplicate.
3. Otherwise, add it to the set.

This takes O(n) expected time and O(n) additional space.

If an ID appears for the third time, it will still be detected as a duplicate. Whether to report every repeated occurrence or each duplicated ID only once depends on the requirement.

### Scenario 2: Preserve First-Seen Order

**Question:**

A log contains event codes such as `A, B, A, C, B`. The output must contain each code once in the order it first appeared. Why is converting the list directly into a set not the right approach?

**Answer:**

A set removes duplicates, but its iteration order is not a contract for preserving the original sequence.

I would use both a set and a list:

* The set checks whether an element has appeared before.
* The list stores unique elements in their original first-seen order.

The expected time complexity is O(n), and the auxiliary space complexity is O(n).

### Scenario 3: Incorrect Sorting Result

**Question:**

A developer writes:

```python
scores = [71, 95, 84]
result = scores.sort()

print(result)
```

Why does the program print `None`?

**Answer:**

The `sort()` method changes the original list in place and returns `None`.

Two correct approaches are:

```python
scores = [71, 95, 84]

scores.sort()
print(scores)
# [71, 84, 95]
```

Or:

```python
scores = [71, 95, 84]

result = sorted(scores)

print(result)
# [71, 84, 95]

print(scores)
# [71, 95, 84]
```

The first approach modifies the original list. The second creates a new sorted list.

### Scenario 4: Maximum Usage Over Three Hours

**Question:**

A dashboard receives hourly usage values and must find the maximum total across any three consecutive hours. How can a sliding window help?

**Answer:**

First, I would calculate the sum of the first three values.

For every next position, I would subtract the value leaving the window and add the new value entering it. I would update the maximum after each movement.

This avoids recalculating every three-element sum from scratch.

The time complexity is O(n), and the auxiliary space complexity is O(1).

I would also validate that `k` is positive and no greater than the number of available elements.

### Scenario 5: Empty API Input

**Question:**

An API receives a list of item prices and calculates their average. What should happen if the list is empty?

**Answer:**

The API should validate the input before dividing by the number of elements.

The expected behavior should be defined in the API contract. For example, the endpoint could return a validation error for an empty list.

It should not silently return a misleading average or hide the problem with a broad exception handler.

### Scenario 6: Slow Membership Checks

**Question:**

A program checks each incoming username against a large list of previously processed usernames. Why might the program become slow, and how can we improve it?

**Answer:**

List membership can take O(n) time for each check. Repeating this operation for many usernames can lead to O(n²) total time.

I would use a set to store previously processed usernames. Set membership takes O(1) time on average, so processing n usernames takes O(n) expected time, with O(n) additional memory.

---

## 7. Debugging Practice

### Problem: Incorrect "Not Found" Message

**Question:**

The following code prints `"Not found"` for every element that does not match the target.

```python
numbers = [4, 8, 12]
target = 7

for number in numbers:
    if number == target:
        print("Found")
    else:
        print("Not found")
```

Why is the output incorrect?

**Answer:**

The `else` block runs for every individual nonmatching element. The program does not wait until the entire list has been checked.

A better solution is to use a `for/else` loop.

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

**Output:**

```text
Not found
```

The `else` block runs only if the loop finishes without encountering a `break`.

If the target is found, the program prints `"Found"` and stops searching.

---

## 8. Additional Coding Practice

### Problem 4: Remove Duplicates While Preserving Order

**Question:** Write a function that removes duplicate elements without changing the order of their first occurrence.

```python
def unique_in_order(items):
    seen = set()
    result = []

    for item in items:
        if item not in seen:
            seen.add(item)
            result.append(item)

    return result


print(unique_in_order([]))
# []

print(unique_in_order([5]))
# [5]

print(unique_in_order([2, 2, 2]))
# [2]

print(unique_in_order([3, 1, 3, 2, 1]))
# [3, 1, 2]
```

**Explanation:**

The set helps identify previously encountered elements, while the list preserves their first-seen order.

**Complexity:**

* Time: O(n) expected
* Auxiliary space: O(n)

### Problem 5: Handle Invalid Window Sizes

**Question:** What should a maximum-window-sum function do when `k` is zero, negative, or larger than the input list?

**Answer:**

The function should validate `k` before calculating the initial window sum.

For this example, I return `None` for an invalid window size:

```python
def max_sum_window(numbers, k):
    if k <= 0 or k > len(numbers):
        return None

    window_sum = sum(numbers[:k])
    best = window_sum

    for right in range(k, len(numbers)):
        window_sum += numbers[right] - numbers[right - k]
        best = max(best, window_sum)

    return best
```

Returning `None` is one possible design choice. An API or application could instead raise a `ValueError`, depending on its requirements.

---

## 9. Day 2 Assignments

These are the assignments I need to practice and complete.

### Task A — Remove Duplicates

Write a function that removes duplicates while preserving first-seen order.

Requirements:

* Do not use a one-line shortcut.
* Include at least five test cases.
* Explain the approach and complexity.

### Task B — Maximum Window Sum

Write a function that returns the maximum sum of a contiguous window of size `k`.

Requirements:

* Validate invalid window sizes.
* Test empty input and negative numbers.
* Explain the sliding-window technique.
* State time and auxiliary space complexity.

### Task C — Debugging

Correct the loop that prints `"Not found"` for every nonmatching element.

Requirements:

* Use `for/else` or a suitable flag.
* Explain why the original code was incorrect.

### Task D — Compare Data Structures

Compare lists, sets, and dictionaries in terms of:

* Order
* Duplicate handling
* Membership checking
* Common use cases

Include one practical backend example for each.

### Task E — Reflection

Write a short reflection about:

* My problem-solving approach
* Test cases I tried
* Time and space complexity
* Difficulties I faced
* What I can improve after feedback

---

## 10. Testing Checklist

Before considering a solution complete, I should test different inputs.

* [ ] Empty list
* [ ] Single-element list
* [ ] List containing duplicates
* [ ] List containing only duplicates
* [ ] Negative numbers
* [ ] Already sorted input
* [ ] Target found
* [ ] Target not found
* [ ] Window size `k = 1`
* [ ] Window size equal to the list length
* [ ] Window size `k = 0`
* [ ] Negative window size
* [ ] Window size greater than the list length

---

## 11. What I Learned Today

One thing I understood from today's practice is that choosing the right approach can make a big difference in performance.

For example, using a set instead of repeatedly searching a list can make duplicate detection much faster. Similarly, the sliding-window technique avoids calculating the same elements repeatedly.

I also learned that testing edge cases and understanding the time and space complexity are important parts of solving coding problems.

I still need to practice more problems to become comfortable with these techniques.

## 👨‍💻 About Me

I'm a Computer Science graduate working on improving my Python programming and problem-solving skills as part of my placement preparation.

I'm using this repository to keep my learning notes, practice coding problems, and track my progress.

Thanks for checking out my repository! 😊

**Learning step by step and improving with practice. 🚀**
