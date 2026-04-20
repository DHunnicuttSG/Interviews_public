Below are **Big O “trick” questions interviewers love**—not because they’re obscure, but because they reveal whether you truly understand **how growth works** and can avoid common reasoning traps.

For each one, I’ll show:

*   ✅ The trick
*   ✅ The correct Big O
*   ✅ What interviewers are testing
*   ✅ How to answer out loud

***

# Big O Trick Questions Interviewers Love

***

## 1. Two Loops ≠ Nested Loops (The Classic Trap)

### ❓ Problem

```python
for i in range(n):
    do_a()

for j in range(n):
    do_b()
```

### ❌ Common Wrong Answer

`O(n²)`

### ✅ Correct Answer

**O(n)**

### 🧠 Why It’s a Trick

Candidates see *two loops* and panic.

### ✅ How to Say It

> “The loops are sequential, not nested. Each runs n times, so the total work is n + n = 2n, which simplifies to O(n).”

***

## 2. Hidden Logarithm

### ❓ Problem

```python
i = 1
while i < n:
    i *= 2
```

### ❌ Common Wrong Answer

`O(n)`

### ✅ Correct Answer

**O(log n)**

### 🧠 Why It’s a Trick

Loop condition *looks* linear → but growth is multiplicative.

### ✅ How to Say It

> “The variable doubles each iteration, so the loop runs log₂(n) times.”

📌 **Multiplication/division → logarithmic**

***

## 3. Dropping Dominant Terms

### ❓ Problem

```python
for i in range(n):
    for j in range(n):
        work()

for k in range(n):
    work()
```

### ❌ Common Wrong Answer

`O(n² + n)`

### ✅ Correct Answer

**O(n²)**

### 🧠 Why It’s a Trick

Candidates forget we drop smaller terms.

### ✅ How to Say It

> “The dominant term is n², so we drop the linear part. The complexity is O(n²).”

***

## 4. Not All Nested Loops Are O(n²)

### ❓ Problem

```python
for i in range(n):
    for j in range(i):
        work()
```

### ✅ Correct Answer

**O(n²)**

### 🧠 Why It’s a Trick

Inner loop doesn’t always run n times.

### ✅ Explanation

Total work:

    0 + 1 + 2 + ... + (n-1) = n(n-1)/2

Still quadratic.

### ✅ How to Say It

> “Even though the inner loop grows with i, the total number of operations still scales quadratically.”

***

## 5. Sorting Inside a Loop 🚨

### ❓ Problem

```python
for i in range(n):
    arr.sort()
```

### ❌ Common Wrong Answer

`O(n log n)`

### ✅ Correct Answer

**O(n² log n)**

### 🧠 Why It’s a Trick

Forgetting that sorting happens **n times**.

### ✅ How to Say It

> “The loop runs n times, and sorting takes n log n each time, so the total is O(n² log n).”

📌 Interviewers *love* this one.

***

## 6. Hash Maps Are Not Always O(1)

### ❓ Problem

```python
def contains(nums, target):
    return target in nums_set
```

### ✅ Correct Answer

**Average: O(1)**  
**Worst case: O(n)**

### 🧠 Why It’s a Trick

Candidates say “O(1)” without qualification.

### ✅ How to Say It

> “Hash lookups are O(1) on average, but in the worst case—due to collisions—they can degrade to O(n).”

💯 Saying *both* earns points.

***

## 7. Input Size ≠ Value Size

### ❓ Problem

```python
def print_digits(n):
    while n > 0:
        print(n % 10)
        n //= 10
```

### ❌ Common Wrong Answer

`O(n)`

### ✅ Correct Answer

**O(log n)**

### 🧠 Why It’s a Trick

Here, `n` is a **number**, not a collection.

### ✅ How to Say It

> “Each iteration removes one digit. The number of digits in n is log₁₀(n), so the time complexity is O(log n).”

📌 Input representation matters.

***

## 8. Recursive Fibonacci (Everyone’s Favorite)

### ❓ Problem

```python
def fib(n):
    if n <= 1:
        return n
    return fib(n-1) + fib(n-2)
```

### ✅ Correct Answer

**O(2ⁿ)**

### 🧠 Why It’s a Trick

Looks innocent. Is catastrophic.

### ✅ How to Say It

> “Each call branches into two more calls, forming an exponential recursion tree with massive redundant computation.”

Bonus: mention memoization reduces it to O(n).

***

## 9. Space Complexity Is the Real Question

### ❓ Problem

```python
def reverse(arr):
    return arr[::-1]
```

### ✅ Correct Answer

*   **Time:** O(n)
*   **Space:** O(n)

### 🧠 Why It’s a Trick

Candidates only analyze time.

### ✅ How to Say It

> “The operation copies the array, so it requires additional space proportional to n.”

📌 Interviewers love balanced answers.

***

## 10. “What’s the Complexity of This Function?” (No Code)

### ❓ Problem

> “What’s the time complexity of checking if a value exists in a sorted array?”

### ✅ Best Answer

> “If using binary search, O(log n). If using linear search, O(n). It depends on the algorithm.”

### 🧠 Why It’s a Trick

They want you to **ask clarifying questions** or state assumptions.

***

## The Meta‑Trick Interviewers Use

They are checking whether you:

✅ Understand **growth**, not syntax  
✅ Can **reason out loud**  
✅ Can justify tradeoffs  
✅ Know worst vs average case  
✅ Don’t panic under ambiguity

***

## Interview Power Move

End answers like this:

> “That’s the worst‑case time complexity. If assumptions change, the complexity could differ.”

This signals **senior‑level thinking**.

***
