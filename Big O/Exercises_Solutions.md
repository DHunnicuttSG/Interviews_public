Below are **practical, interview‑style Big O exercises** that reflect what you’ll actually see in coding interviews. Each problem is followed by a **clear, step‑by‑step solution** showing how interviewers expect you to reason.

***

# Big O Interview Exercises (with Solutions)

## Exercise 1: Simple Loop

### ❓ Problem

What is the **time complexity** of the following code?

```python
def print_items(arr):
    for item in arr:
        print(item)
```

### ✅ Solution

*   The loop runs **once per element**
*   If the array has `n` elements, it executes `n` times

**Time Complexity:**  
✅ **O(n)**

***

## Exercise 2: Nested Loops (Same Input)

### ❓ Problem

What is the time complexity?

```python
def print_pairs(arr):
    for i in arr:
        for j in arr:
            print(i, j)
```

### ✅ Solution

*   Outer loop runs `n` times
*   Inner loop runs `n` times for **each** outer iteration
*   Total operations: `n × n`

**Time Complexity:**  
✅ **O(n²)**

📌 *Classic interview example for quadratic time.*

***

## Exercise 3: Nested Loops (Different Inputs)

### ❓ Problem

What is the Big O?

```python
def print_pairs(a, b):
    for x in a:
        for y in b:
            print(x, y)
```

### ✅ Solution

*   Loop over `a` → `n` times
*   Loop over `b` → `m` times

**Time Complexity:**  
✅ **O(n × m)**

📌 Don’t assume both inputs are the same unless stated.

***

## Exercise 4: Sequential Loops

### ❓ Problem

Determine the time complexity:

```python
def process(arr):
    for x in arr:
        print(x)

    for y in arr:
        print(y)
```

### ✅ Solution

*   First loop: `n`
*   Second loop: `n`
*   Total: `n + n = 2n`
*   Drop constants

**Time Complexity:**  
✅ **O(n)**

***

## Exercise 5: Logarithmic Loop

### ❓ Problem

What is the time complexity?

```python
def halve(n):
    while n > 1:
        n //= 2
```

### ✅ Solution

*   Each iteration halves `n`
*   Number of times you can divide by 2 before hitting 1 is `log₂(n)`

**Time Complexity:**  
✅ **O(log n)**

📌 Division, not subtraction → logarithmic.

***

## Exercise 6: Binary Search

### ❓ Problem

What is the time complexity of binary search?

```python
def binary_search(arr, target):
    low, high = 0, len(arr) - 1
    while low <= high:
        mid = (low + high) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
```

### ✅ Solution

*   Each iteration cuts the remaining search space in half

**Best Case:** O(1)  
**Worst/Average Case:** ✅ **O(log n)**

***

## Exercise 7: Recursive Linear Function

### ❓ Problem

Find the time complexity:

```python
def countdown(n):
    if n == 0:
        return
    countdown(n - 1)
```

### ✅ Solution

*   One recursive call per function
*   Depth of recursion = `n`

**Time Complexity:**  
✅ **O(n)**  
**Space Complexity:**  
✅ **O(n)** (call stack)

***

## Exercise 8: Exponential Recursion

### ❓ Problem

Analyze the complexity:

```python
def fib(n):
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)
```

### ✅ Solution

*   Each function call makes **two more**
*   Recomputes the same values repeatedly

**Time Complexity:**  
✅ **O(2ⁿ)**  
**Space Complexity:**  
✅ **O(n)** (call stack)

📌 Interviewers *love* this example.

***

## Exercise 9: Sorting Inside a Loop (Red Flag)

### ❓ Problem

What’s the time complexity?

```python
def sort_each_time(arr):
    for i in range(len(arr)):
        arr.sort()
```

### ✅ Solution

*   Loop runs `n` times
*   `.sort()` is `O(n log n)`
*   Total work: `n × (n log n)`

**Time Complexity:**  
✅ **O(n² log n)**

🚨 *Major interview red flag.*

***

## Exercise 10: Hash Map Lookup

### ❓ Problem

Determine the time complexity:

```python
def contains(nums, target):
    lookup = set(nums)
    return target in lookup
```

### ✅ Solution

*   Building the set: `O(n)`
*   Lookup in set: `O(1)` (average case)

**Total Time Complexity:**  
✅ **O(n)**  
**Space Complexity:**  
✅ **O(n)**

📌 Trade space for time.

***

## Exercise 11: Best vs Worst Case

### ❓ Problem

What is the time complexity of linear search?

```python
def linear_search(arr, target):
    for i in arr:
        if i == target:
            return True
    return False
```

### ✅ Solution

| Case    | Complexity |
| ------- | ---------- |
| Best    | O(1)       |
| Average | O(n)       |
| Worst   | O(n)       |

📌 **Big O usually refers to the worst case → O(n)**

***

## Exercise 12: Space Complexity

### ❓ Problem

Analyze space complexity:

```python
def duplicate(arr):
    new_arr = []
    for x in arr:
        new_arr.append(x)
    return new_arr
```

### ✅ Solution

*   Stores a new array of size `n`

**Space Complexity:**  
✅ **O(n)**

***

## Interview Tip: How to Answer Verbally

When asked for Big O, say:

> “The loop runs *n* times and the inner operation is constant, so the overall time complexity is O(n).”

Clear, structured reasoning matters **more than speed**.

***
