## 🔥 Dangling Pointers

Consider:

```c
struct Node* head = malloc(sizeof(struct Node));
head->data = 10;
head->next = NULL;

free(head);
```

👉 What is head pointing to?

Answer:

It still contains the same memory address.
But that memory is no longer valid.

This is called a dangling pointer.

If you now do:

```c
printf("%d", head->data);
```

You get undefined behavior:

- Might print 10
- Might crash
- Might corrupt memory

---

✅ Correct Pattern

After freeing:

```c
free(head);
head = NULL;
```

Now it’s safe.

---

## 🔥 Double Free

```c
free(head);
free(head);
```

What happens?

💣 Undefined behavior again.

Most modern systems detect it and crash.

Why?

Because the memory allocator marks that block as freed.
Freeing it again corrupts the heap.

## 🔥 The Classic Linked List Bug
Look at this:
```c
struct Node* temp = head;
free(head);
head = head->next;
```

🚨 What’s wrong?

freed head first.

Then accessed:
```c
head->next
```
But `head` is already freed.

That is use-after-free.

## ✅ Correct Order

Always:

1. Save next
2. Free current
3. Move forward

Like this:
```c
struct Node* temp = head;
head = head->next;
free(temp);
```

Order matters when it's Linked list:
```c
while (current != NULL) {
    next = current->next;
    free(current);
    current = next;
}
```

## 🔥 Memory Leak Scenario

Consider:
```C
head = head->next;
```
Without freeing old head.

What happened?

You lost the reference to the first node.

That memory is now unreachable.

That is a memory leak.

-----
Every `malloc` must have exactly one free.   
If you draw memory like this:
```c
head → [A] → [B] → [C]
```
You must:

* Not lose arrows
* Not follow freed arrows
* Not free same box twice

Question:

Here’s a subtle one:
```c
void delete_node(struct Node* node) {
    free(node);
    node = NULL;
}
```

After calling this:  
```c
delete_node(head);
```
Is `head` now **NULL**?

<details>
  <summary style="color:#93f022">Click to reveal the answer to your question</summary>

<br>

Step-by-step:
Assume:
```
head → [Node A]
```

When calling the function:
```
node = head
```
But remember:

👉 node is a copy of the pointer.

So inside memory we now have:
```
head → [Node A]
node → [Node A]
```
Two pointers.   
Same address.   
Different variables.

**Inside the function**
```c
free(node);
```
Now:
```
head → (freed memory)
node → (freed memory)
```
Memory is gone.

Then:
```c
node = NULL;
```
Now:
```
node = NULL
head → (still pointing to freed memory)
```

Only the local copy changed.

When the function ends:

* node disappears
* head is still dangling

🚨 So now `head` is a **dangling pointer**.


### The Correct Way

If you want the caller’s pointer to become NULL:
```c
void delete_node(struct Node** node) {
    free(*node);
    *node = NULL;
}
```
Call it like:
```c
delete_node(&head);
```
Now:

* `*node` refers to head
* Setting `*node = NULL` changes the original pointer


* If a function must modify:

    | What you want to modify | What to pass    |
    | ----------------------- | --------------- |
    | The data inside node    | `struct Node*`  |
    | The pointer itself      | `struct Node**` |


</details>

------------------------
------------------------

Question:    
Now think about this one:    
```c
void delete_all(struct Node* head) {
    while (head != NULL) {
        struct Node* temp = head;
        head = head->next;
        free(temp);
    }
}
```
After calling:   
```c
delete_all(head);
```
1. Is the memory freed?
2. Is head in main now NULL?
3. Is it safe or dangerous?

<details>
  <summary style="color:#93f022">Click to reveal the answer to your question</summary>

<br>

1. Yes. All nodes are freed correctly.   
    The loop properly:

    1. Saves head in temp
    2. Moves head forward
    3. Frees the old node

    And no — we do NOT need **temp = NULL**;

    Why?

    Because temp is a local variable that disappears after the function ends. Setting it to NULL would change nothing meaningful.

2. No.   
    Because `head` was passed by value.  
    Inside the function:     
    ```c
    head = head->next;
    ```
    Only modifies the local copy.   
    The original `head` in `main` is untouched.

3. No, It's not safe.   
    Memory is freed  
    But the caller’s `head` still holds the old address  
    So after the function returns:   
    ```
    main's head → (freed memory)
    ```
    That means:  
    👉 head is now a dangling pointer.

    if we check:
    ```
    if(head == NULL)
    ```

    it will NOT be NULL.

    if you do:   
    ```c
    head->data
    ```
    💣 Undefined behavior.

</details>


-------------------
-------------------
<br>
<br>


### Proper Safe Version
```c
void delete_all(struct Node** head) {
    struct Node* current = *head;

    while (current != NULL) {
        struct Node* temp = current;
        current = current->next;
        free(temp);
    }

    *head = NULL;
}

delete_all(&head); // Call it
```
Now:

* Memory freed
* Caller’s pointer becomes NULL
* No dangling pointer
* Fully safe

--------------------
--------------------

You now understand:

* Pointer copy vs pointer reference
* Dangling pointers
* Use-after-free
* Proper deletion order
* Why ** is necessary
* Why typedef hiding pointers is dangerous

Next level question:

What happens in this situation?
```c
struct Node* a = malloc(sizeof(struct Node));
struct Node* b = a;

free(a);

b->data = 10;
```

Is this different from the earlier dangling case? Or exactly the same?

<details>
  <summary style="color:#93f022">Click to reveal the answer to your question</summary>

<br>

Step-by-step:
```c
struct Node* a = malloc(sizeof(struct Node));
```
Memory allocated on heap.

Example:     
```
a → 0x1000
```
```c
struct Node* b = a;
```
Now:
```
a → 0x1000
b → 0x1000
```
Two pointers.   
One memory block.

Important:

There is still only **one allocation**.

```
free(a);
```
Now:

* The memory at `0x1000` is returned to the allocator.
* That memory is invalid.

But the pointer values do NOT change.

So now:  
```
a → 0x1000 (dangling)
b → 0x1000 (dangling)
```
Both are dangling.

```
b->data = 10;
```
🚨 This is use-after-free.

Undefined behavior.

Could:

* Seem to work
* Crash
* Corrupt heap
* Overwrite another allocation
* Create future random bugs


</details>








