# DevOps Fundamentals Interview Questions

## DevOps Concepts
1. What is DevOps?
2. What are the core principles of DevOps?
3. What DevOps methodologies do you follow while implementing a project?
4. Explain the complete CI/CD lifecycle from scratch.
5. What are the stages of a software development lifecycle (SDLC)?
6. How does DevOps improve software delivery?
7. What is Infrastructure as Code (IaC)?
8. What is GitOps?
9. What is Shift-Left testing?
10. What is DevSecOps?

## Collaboration & Process
11. How do development and operations teams collaborate in DevOps?
12. What challenges have you faced while implementing DevOps practices?
13. How do you measure DevOps success?
14. What KPIs do you track for CI/CD pipelines?
15. How do you handle change management in production?

## Release Management
16. What is Continuous Integration?
17. What is Continuous Delivery?
18. What is Continuous Deployment?
19. Difference between Continuous Delivery and Continuous Deployment?
20. What deployment strategies have you used?

## Scaling
21. What is the key difference between Horizontal Scaling and Vertical Scaling?

> Git-specific questions have moved to [Git/git.md](../Git/git.md).

## Coding Practice

General scripting/algorithm exercises that come up alongside DevOps interviews.

### Exercise 1: Binary Search
**Objective:** Implement an O(log n) search over a sorted list.
**Task:** Write a function that takes a sorted list and a target value, and returns the index of the target (or -1 if not found) using binary search.

**Solution (Python):**
```python
from typing import List, Optional

def binary_search(arr: List[int], lb: int, ub: int, target: int) -> Optional[int]:
    while lb <= ub:
        mid = lb + (ub - lb) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            lb = mid + 1
        else:
            ub = mid - 1
    return -1
```

### Exercise 2: Merge Sort
**Objective:** Implement an O(n log n) divide-and-conquer sort.
**Task:** Write a function that sorts a list using merge sort.

**Solution (Python):**
```python
from typing import List

def merge_sort(arr: List[int]) -> List[int]:
    if len(arr) <= 1:
        return arr
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    return merge(left, right)

def merge(left: List[int], right: List[int]) -> List[int]:
    merged = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            merged.append(left[i])
            i += 1
        else:
            merged.append(right[j])
            j += 1
    merged.extend(left[i:])
    merged.extend(right[j:])
    return merged
```

### Exercise 3: Reimplement `grep -A -B` in Python
**Objective:** Practice text processing — a very common DevOps take-home task (log parsing).
**Task:** Implement the equivalent of `grep error -A 2 -B 2 some_file` in Python: find lines matching a pattern and print them along with N lines of context before and after each match.

**Solution (Python):**
```python
def grep_with_context(filename: str, pattern: str, before: int = 2, after: int = 2) -> None:
    with open(filename) as f:
        lines = f.readlines()

    match_indexes = [i for i, line in enumerate(lines) if pattern in line]

    for idx in match_indexes:
        start = max(0, idx - before)
        end = min(len(lines), idx + after + 1)
        print("".join(lines[start:end]))
        print("--")

if __name__ == "__main__":
    grep_with_context("some_file", "error")
```
