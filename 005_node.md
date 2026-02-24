
# 🚀 Node 

Let’s build mental clarity before coding.

Consider:

```c
struct Node {
    int value;
    struct Node *next;
};
```

Memory layout (32-bit system):

```
0x1000  value (4 bytes)
0x1004  next  (4 bytes)
```

Now suppose:

```
Node A at 0x1000
Node B at 0x2000
```

If:

```
A.next = &B
```

Memory looks like:

```
0x1000  A.value
0x1004  0x2000

0x2000  B.value
0x2004  NULL
```

That’s it.

That’s a linked list.

---

## 🔥 Mini Exercise

Suppose we execute:

```c
struct Node n1, n2;

n1.value = 10;
n2.value = 20;

n1.next = &n2;
n2.next = NULL;
```

Questions:

1. How many total bytes allocated?
2. Are these on stack or heap?
3. What happens if this function returns?

```
struct Node* create_list() {
    struct Node n1, n2;
    ...
    return &n1;
}
```

4. Why do linked lists usually use malloc instead?

<details>
  <summary style="color:#93f022">Click to reveal the answer to your question</summary>

<br>

1. (On 32-bit system:)

   ```
   int        → 4 bytes
   Node*      → 4 bytes
   struct Node → 8 bytes
   ```

   Allocates:

   ```
   struct Node n1, n2;
   8 + 8 = 16 bytes
   ```

2. Since no malloc() was used:

   ```
   n1 → stack
   n2 → stack
   ```

   They are automatic (local) variables.  
   in the `Stack`.

3. The stack frame is destroyed. 💥  
   So:
   - n1 and n2 memory becomes invalid
   - Returning &n1 gives a `dangling pointer`
   - Accessing it = undefined behavior

   This is a VERY important rule:  
   **Never return address of a local variable.**  
   **Because stack memory disappears after function exits.**

4. If we want:
   - List survives function return
   - Dynamic size
   - Nodes added at runtime

   We must allocate nodes in the heap:

   ```c
   struct Node *n1 = malloc(sizeof(struct Node));
   ```

   Heap memory remains valid until `free()`.

</details>

## 🧠 Critical Concept You Just Crossed

- Stack nodes → temporary
- Heap nodes → persistent

This is why real linked lists use dynamic memory.

<br>
<br>
<br>

# Linked List Properly

Let’s mentally build this:

```c
struct Node {
    int value;
    struct Node *next;
};
```

Now:

```c
struct Node *head = NULL;
```

Question:

1. Why is head a pointer and not a struct?  
   Why don’t we write:
   ```c
   struct Node head;
   ```
2. What would happen if head was a struct?
3. Which version allows an empty list?

<details>
  <summary style="color:#93f022">Click to reveal the answer to your question</summary>

<br>

1. A linked list is a dynamic chain of nodes that live in the heap.

   So:

   ```c
   struct Node *head = NULL;
   ```

   means:
   - `head` stores the address of the **first node**
   - If list is empty → head == NULL
   - If list has elements → head points to first heap node

   This allows:
   - Dynamic size
   - Reassigning head
   - Inserting at beginning
   - Deleting first node

2. struct Node head;  
   This creates a real node in stack memory.  
   Problems:  
   ❌ Cannot represent empty list cleanly  
   Because `head` always exists.

   You would need special flags like:

   ```
   head.value = -1;
   ```

   ❌ Head becomes a fixed embedded node  
   If you insert at beginning:  
   You must copy values instead of relinking pointers.  
   That breaks the elegance of linked list logic.

   ❌ Cannot safely return it  
   If declared inside a function:

   ```
   struct Node head;
   return &head;   // ❌ dangling pointer
   ```

   Invalid after function exits.

3. struct Node \*head = NULL;

   This is the cleanest representation of:
   - “There is no first node.”

   NULL literally means:
   - No memory. No node.

   **A pointer to the first dynamically allocated node.**

</details>

### 🚀 Now Let’s Go One Step Deeper

Consider this insertion:

```c
struct Node *new_node = malloc(sizeof(struct Node));
new_node->value = 5;
new_node->next = head;
head = new_node;
```

This inserts at the beginning.

### 🔥 Mini Exercise (Very Important)

Suppose initially: 　

```c
head = NULL;
```

We run the insertion code above twice.

Step-by-step:

1️⃣ After first insertion, what does memory look like?  
2️⃣ After second insertion, what does memory look like?  
3️⃣ What is the order of values in the list?

<details>
  <summary style="color:#93f022">Click to reveal the answer to your question</summary>

<br>

`We assume 32-bit system (Node = 8 bytes).`

1.

```c
struct Node *head = NULL;
```

head → NULL  
No heap memory allocated yet.

- First Insertion

  ```c
  struct Node *new_node = malloc(sizeof(struct Node));
  new_node->value = 5;
  new_node->next = head;   // head is NULL
  head = new_node;
  ```

  Step-by-step
  1. `malloc()` allocates 8 bytes somewhere in heap
     Assume address = `0x2000`

  2. Set fields:

  ```c
  0x2000  value = 5
  0x2004  next  = NULL
  ```

  3. Update head:

  ```c
  head = 0x2000
  ```

* **Memory After First Insert**

  ```c
  STACK:
  head → 0x2000

  HEAP:
  0x2000  5
  0x2004  NULL
  ```

* **List:**
  ```c
  (5) → NULL
  ```

- **Second Insertion**

  Same code runs again.
  1. `malloc()` allocates another 8 bytes  
     Assume address = `0x3000`

  2. Set value:

  ```c
  0x3000  value = 5
  ```

  3. Set next:

  ```c
  new_node->next = head
  ```

  At this moment:

  ```c
  head = 0x2000
  ```

  so:

  ```c
  0x3004 = 0x2000
  ```

  4. Update head:

  ```c
  head = 0x3000
  ```

* **Memory After Second Insert**

  ```c
  STACK:
  head → 0x3000

  HEAP:
  0x3000  5
  0x3004  0x2000

  0x2000  5
  0x2004  NULL
  ```

* **Final List Order**

  ```
  (5) → (5) → NULL
  ```

  - The second inserted node is now the FIRST node.

* If we insert values in this order:
  ```
  1
  2
  3
  4
  ```
  the final list look like:
  ```
  (4)->(3)->(2)->(1)->NULL
  ```

</details>

---

```c
struct Node* head;
```

How do we print the list?

```c
struct Node *temp = head;

while (temp != NULL) {
    printf("%d\n", temp->value);
    temp = temp->next;
}
```

What Is Actually Happening?
This line:

```c
temp = temp->next;
```

means:

1. Read the address stored in temp->next
2. Move temp to that address
3. Repeat

You are literally walking through memory.

### 🔥 Mini Exercise

Suppose memory is:

```
head → 0x3000

0x3000: value=4, next=0x2800
0x2800: value=3, next=0x2500
0x2500: value=2, next=0x2100
0x2100: value=1, next=NULL
```

Question

1. Walk the loop mentally.  
   What are the values printed, in order?

2. If we want to delete the head node.  
   What must we do?

3. What happens if we free(head) immediately?
4. What happens to the rest of the list?
5. What correct steps should be taken?

<details>
  <summary style="color:#93f022">Click to reveal the answer to your question</summary>

<br>

1.

```
4
3
2
1
```

2.

```c

void delete_head(struct Node** head) {
    if (*head == NULL)
        return;

    struct Node* temp = *head;
    *head = (*head)->next;
    free(temp);
}

delete_head(&head); // This modifies the original pointer directly.
```

3. If we do:  
   `free(head)` without saving `head->next`, you lose the address of the second node forever.
   It's leads to **Memory Leeks**.

4. Lost of track leads to **Memory Leeks**.

5. We need to free all the memory allocation for the `Nodes`.

```c
void free_list(struct Node** head) {
    struct Node* current = *head;
    struct Node* tmp;

    while (current != NULL) {
        tmp = current->next;
        free(current);
        current = tmp;
    }

    *head = NULL;
}
```

</details>

**Note :**

- When we pass a pointer to a function. ex :- `void function(struct Node* head){}`.  
  **we are passing the pointer by value.**

- if you want to modify the pinter itself, we need: **pointer to pointer**. ex:-  
  `void function(struct Node** head){}`

Question

Consider this function:

```c
void insert_at_head(struct Node* head, int value);
```

Why is this signature wrong for inserting into a linked list?

What should it be instead?

<details>
  <summary style="color:#93f022">Click to reveal the answer to your question</summary>

<br>

Here:

- head is passed by value
- It is a copy of the pointer
- Changing it only changes the local copy

**Correct Version (Pointer to Pointer)**

```c
void insert_at_head(struct Node** head, int value)
```

Now you pass the address of the pointer.  
to modify the original head pointer.  
✔ This is the correct pattern.

</details>
