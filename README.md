# Implementation of Queue Using Stack

A **queue** is a linear data structure that follows the **First In First Out (FIFO)** principle, where the element inserted first is removed first.  
A **stack**, in contrast, follows the **Last In First Out (LIFO)** principle.

Even though both structures work differently, a queue can be implemented using stacks by using the **reversing nature of stacks**.

---

## Concept

The most common method of implementing a queue using stacks is by using **two stacks**:

- **Stack S1** → used for **enqueue (insertion)**
- **Stack S2** → used for **dequeue (removal)**

### Key Idea
When elements are moved from one stack to another, their order gets **reversed**.  
This reversal helps in achieving **FIFO behavior** using **LIFO stacks**.

---

## Working Mechanism

### Enqueue Operation
- The new element is pushed directly into **Stack S1**
- This operation is simple and takes **O(1)** time

### Dequeue Operation
- If **Stack S2** is not empty, pop the top element from **S2**
- If **Stack S2** is empty:
  - Move all elements from **S1** to **S2**
  - Then pop the top element from **S2**
- This ensures that the **oldest inserted element** is removed first

---

## Algorithms

### Enqueue Algorithm
```text
Push element into Stack S1

Dequeue Algorithm

If S2 is not empty:
    Pop from S2
Else if both stacks are empty:
    Queue underflow
Else:
    Move all elements from S1 to S2
    Pop from S2


---

Example

If elements 10, 20, 30 are enqueued:

Stack S1 stores: 10, 20, 30

During dequeue, elements are transferred to S2 as: 30, 20, 10

The element 10 is removed first, which confirms FIFO behavior



---

Time and Space Complexity

Enqueue: O(1)

Dequeue: O(n) in the worst case, O(1) amortized

Space Complexity: O(n)



---

Conclusion

Implementing a queue using stacks is a classic example of data structure transformation.
By using two stacks and reversing the element order, FIFO behavior is successfully achieved using LIFO structures.

This approach demonstrates strong conceptual understanding and logical problem-solving skills.