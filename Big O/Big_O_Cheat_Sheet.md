Below is a **concise, high‑signal Big O cheat sheet** you can use for interviews, studying, or day‑to‑day engineering decisions.

***

# Big O Notation — Cheat Sheet

## What Big O Describes

**Big O notation** describes how an algorithm’s **time or space usage grows as input size (n) increases**, focusing on the **worst‑case** behavior and **ignoring constants and lower-order terms**.

***

## Common Time Complexities (Fast → Slow)

    O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ) < O(n!)

***

## Complexity Classes at a Glance

### **O(1) — Constant**

*   Time does not depend on input size
*   Example:

```python
x = arr[0]
```

***

### **O(log n) — Logarithmic**

*   Work is halved each step
*   Common in binary search, balanced trees

```python
while n > 1:
    n //= 2
```

***

### **O(n) — Linear**

*   One pass through data

```python
for x in arr:
    process(x)
```

***

### **O(n log n) — Linearithmic**

*   Divide-and-conquer
*   Optimal for comparison sorting

```python
merge_sort(arr)
```

***

### **O(n²) — Quadratic**

*   Nested loops over same data

```python
for i in arr:
    for j in arr:
        compare(i, j)
```

***

### **O(n³) — Cubic**

*   Triple nested loops
*   Very slow at scale

***

### **O(2ⁿ) — Exponential**

*   Each input doubles work

```python
fib(n) = fib(n-1) + fib(n-2)
```

***

### **O(n!) — Factorial**

*   Tries all permutations
*   Practically unusable beyond small n

***

## Best, Average, and Worst Case

| Case      | Meaning                  |
| --------- | ------------------------ |
| Best (Ω)  | Fastest possible         |
| Average   | Typical behavior         |
| Worst (O) | Guaranteed upper bound ✅ |

📌 **Big O usually means worst case**.

***

## Rules for Analyzing Big O

### 1. Drop Constants

    O(5n) → O(n)

### 2. Drop Lower-Order Terms

    O(n² + n) → O(n²)

### 3. Sequential Code → Add

    O(n) + O(n) = O(n)

### 4. Nested Loops → Multiply

    O(n) × O(n) = O(n²)

### 5. Different Inputs → Use Variables

    for a in A:
        for b in B:
            work()
    → O(a × b)

***

## Recursion Quick Reference

| Recursion Pattern   | Time       |
| ------------------- | ---------- |
| One recursive call  | O(n)       |
| Two recursive calls | O(2ⁿ)      |
| Divide in half      | O(log n)   |
| Divide + merge      | O(n log n) |

***

## Space Complexity Examples

```python
# O(1) space
def sum(arr):
    total = 0
    for x in arr:
        total += x

# O(n) space
def copy(arr):
    return arr[:]
```

***

## Data Structure Time Complexity (Average Case)

| Structure     | Lookup   | Insert   | Delete   |
| ------------- | -------- | -------- | -------- |
| Array         | O(n)     | O(n)     | O(n)     |
| Linked List   | O(n)     | O(1)     | O(1)     |
| Hash Map      | O(1)     | O(1)     | O(1)     |
| Balanced Tree | O(log n) | O(log n) | O(log n) |

***

## Common Pitfalls

*   ❌ Thinking O(1) always means “fast”
*   ❌ Ignoring hidden loops in library calls
*   ❌ Assuming hash maps are always O(1)
*   ❌ Optimizing Big O before correctness/readability

***

## Interview Red Flags 🚩

*   Nested loops with no justification
*   Sorting inside a loop
*   Exponential recursion without memoization
*   Using lists instead of sets/maps for lookups

***

## Mental Model

If input size **doubles**:

*   O(1): no change
*   O(log n): +1 step
*   O(n): 2× slower
*   O(n²): 4× slower
*   O(2ⁿ): catastrophic

***

## One‑Line Summary

> **Big O measures how algorithms scale—not how fast they run today.**
