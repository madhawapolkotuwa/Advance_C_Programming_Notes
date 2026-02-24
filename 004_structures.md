
----
# Structures

Consider:

```c
struct Point {
    int x;
    int y;
};
```

This is NOT memory yet.

It is a blueprint.

When you do:

```c
struct Point p1;
```

Now memory is allocated.

If `int` = 4 bytes:

```
Address     Value
0x2000      p1.x
0x2004      p1.y
```

* Struct members are stored sequentially in memory.
* Struct alignment = alignment of `largest member` (4 bytes)

---

# 🔥 Mini Exercise 5

Given:

```c
struct Test {
    char a;
    int b;
};
```

Assume:

* `char` = 1 byte
* `int` = 4 bytes
* System aligns `int` on 4-byte boundary

Answer:

1. What is the size of `struct Test`?
2. Why?
3. What addresses would `a` and `b` likely have if struct starts at `0x1000`?

This question introduces:

> Structure padding & alignment
<details>
  <summary style="color:#93f022">Click to reveal the answer to your question</summary>

<br>

1. Total Size = `8 bytes`

2. 

Assume:
* char = 1 byte
* int = 4 bytes
* `int` must be aligned to `4-byte` boundary.

**Memory Layout Step-by-Step**   
Struct starts at `0x1000`

Step 1 — `char a`
```
0x1000   a  (1 byte)
```
Now next free address is:
```
0x1001
```

**⚠ Problem**

`int` b must start at an address `divisible by 4`. like:
```
0x1000
0x1004
0x1008
...
```
* But `0x1001` is NOT aligned.

**So the compiler inserts padding.**    
Padding bytes are added:
```
0x1001   padding
0x1002   padding
0x1003   padding
```
Now:
```
0x1004   b (4 bytes)
```

**Final Memory Layout**:
```
0x1000   a
0x1001   padding
0x1002   padding
0x1003   padding
0x1004   b
0x1005
0x1006
0x1007
```

3. a -> `0x1000` and b -> `0x1004`

</details>


## 🧠 Critical Insight

Structure size is determined by:

1. Member sizes
2. Alignment rules
3. Padding inserted by compiler

---

## 🔥 Why Alignment Exists

CPUs read memory in chunks.

Reading an `int` from an unaligned address can:

* Be slower
* Cause extra memory cycles
* Crash on some architectures (ARM can fault)

In embedded systems, this knowledge is essential.

---

## 🔥 Mini Exercise 6 (Very Important)

Now consider:

```c
struct Test2 {
    int b;
    char a;
};
```

Same assumptions:

* `char` = 1 byte
* `int` = 4 bytes
* 4-byte alignment

Questions:

1. What is the size of `struct Test2`?
2. Why?
3. Where is padding inserted?


<details>
  <summary style="color:#93f022">Click to reveal the answer to your question</summary>

<br>

1. 8-bytes
2.   
Assumptions:

* int = 4 bytes
* char = 1 byte
* 4-byte alignment
* Struct alignment = alignment of largest member (4 bytes)

**Step 1 — Place b**
Struct starts at `0x1000`
```
0x1000  b
0x1001
0x1002
0x1003
```

**Step 2 — Place a**
Next free address = `0x1004`
```
0x1004  a
```

Now used bytes:
```
0x1000 → 0x1004
```
* Total used so far = 5 bytes

**🚨 But We Are Not Done**  
The struct itself must be aligned to 4 bytes.    
Why?
Because if you create an array:
```
struct Test2 arr[2];
```
* Each element must start at a properly aligned address.
* If struct size were 5 bytes:
```
arr[0] → 0x1000
arr[1] → 0x1005  ❌ not divisible by 4
```
* That would misalign int b inside arr[1].
* The compiler prevents this.

**So What Happens?**

Padding is added at the end.
```
0x1005  padding
0x1006  padding
0x1007  padding
```

**Final Size = 8 bytes**    
Even though logically it contains only 5 bytes of data.


</details>


## 🧠 Major Rule

Struct size is rounded up to a multiple of its largest member alignment.

In this case:
Largest member = `int` (4 bytes)

So struct size must be multiple of 4.

5 → round up → 8

---

## 🔥 Powerful Insight

Both of these:

```c
struct Test {
    char a;
    int b;
};
```

and

```c
struct Test2 {
    int b;
    char a;
};
```

Have size = **8 bytes** (on most systems).

But the padding location differs.

---

## 🚀 Advanced Thinking Question

Which struct is more memory efficient in large arrays?

```c
struct A {
    char a;
    int b;
    char c;
};
```

Answer:

1. What is the size of `struct A`?
2. Why?
3. Where is padding inserted?

<details>
  <summary style="color:#93f022">Click to reveal the answer to your question</summary>

<br>

1. 12-bytes
2. largest member of struct (int) 4-bytes, elements numbers 3,  
    Total bytes = 4 * 3 = 12 bytes
3. 
```
0x1000 a 
0x1001 padding 
0x1002 padding 
0x1003 padding 
0x1004 b 
0x1005 
0x1006 
0x1007 
0x1008 c 
0x1009 padding 
0x100A padding 
0x100B padding
```
</details>

-------------------------------------
-------------------------------------

## Let's Understand real situation

> Assume typical embedded / 32-bit system:

* `uint8_t` = 1 byte
* `uint16_t` = 2 bytes
* Alignment of uint16_t = 2 bytes
* Struct alignment = largest member alignment (2 bytes)

```c
struct Test {
    uint16_t channel;
    uint8_t  command;
    uint16_t data[10];
};
```

**Step 1 — Place channel**  
Struct starts at `0x1000`:
```
0x1000  channel (2 bytes)
0x1001
```
Next free address:
```
0x1002
```
**Step 2 — Place command***
```
0x1002  command (1 byte)
```
Next free address:
```
0x1003
```
**⚠ Alignment Check**  
Next member:
```
uint16_t data[10];
```
Since `uint16_t` requires 2-byte alignment,
it must start at an even address.

But `0x1003` is odd.

So padding is inserted.
```
0x1003  padding (1 byte)
```
Now next address:
```
0x1004  ← aligned
```

**Step 3 — Place data[10]**
Each element:
```
2 bytes
```
Total:
```
10 × 2 = 20 bytes
```

So: 
```
0x1004  data[0]
0x1005
0x1006  data[1]
0x1007
...
...
0x1016  data[9]
0x1017  
```
20 bytes total from `0x1004` to `0x1017`

**Final Memory Layout**
```
0x1000  channel (2)
0x1002  command (1)
0x1003  padding (1)
0x1004  data[0]
...
0x1017  data[9]
```

Total Size

Let’s calculate:
channel → 2,
command → 1,
padding → 1,
data → 20
```
2 + 1 + 1 + 20 = 24 bytes
```
**Final Struct Size = 24 bytes**    

Now check alignment rule:

* Largest member alignment = **2 bytes**
* `24` is divisible by `2` → no trailing padding needed.
* So final size stays `24`.

### Important Insight About Arrays in Structs
`uint16_t data[10];` is not a pointer.

**It is:**
`10` contiguous `uint16_t` elements embedded directly inside the struct.

So the struct literally contains 20 bytes for that array.

If you declare:
```c
struct Test t;
```
**All 24 bytes are allocated in one contiguous block.**

### 🔥 Critical Thinking Question

**What is the difference between:**
```
uint16_t data[10];
```
and
```
uint16_t *data;
```

inside a struct?

This is a HUGE difference.

Answer carefully:
1. Memory size difference?
2. Where is actual array stored?
3. Which one requires `malloc`?
3. Which one is safer in embedded systems?


<details>
  <summary style="color:#93f022">Click to reveal the answer to your question</summary>

<br>

1.  `uint16_t data[10] → 20 bytes`  
    `uint16_t *data → 4 bytes (on 32-bit)`  
    `uint16_t *data → 8 bytes (on 64-bit)`

2. `uint16_t data[10];`   
    The 20 bytes are physically embedded inside the struct.  
    If struct starts at `0x1000`, the array is inside that memory block.

    for:    
    `uint16_t *data;`    

    The struct only contains:
    ```
    0x1000 → some address (like 0x5000)
    ```
    * The actual array lives somewhere else (stack, heap, global, etc.).

3. `uint16_t *data;` requires malloc if you want dynamic storage.

    Example:
    ```
    struct Test t;
    t.data = malloc(10 * sizeof(uint16_t));
    ```
    Without malloc, data is just an uninitialized pointer.

4. `uint16_t data[10];`
    Why?

    * No fragmentation
    * No heap failure
    * No memory leak risk
    * Deterministic memory usage
    * No `free()` mistakes

`This is why embedded firmware often avoids dynamic memory completely.`

</details>


## Now Let’s Level You Up

There is something deeper here.

Consider this struct:
```c
struct Node {
    int value;
    struct Node *next;
};
```
Notice something important:

Why can we NOT write
```c
struct Node {
    int value;
    struct Node next;   // ❌ illegal
};
```
But we CAN write:
```c
struct Node *next;
```

This question unlocks:

👉 Linked lists  
👉 Recursive types   
👉 Node structures   
👉 Real pointer mastery  

Take your time.

Questions:

1. Why `struct Node next;` is illegal
2. Why `struct Node *next;` is valid
3. What would happen if the first one were allowed


<details>
  <summary style="color:#93f022">Click to reveal the answer to your question</summary>

<br>

1. It creates an infinite size problem.
2. `value → 4 bytes`    
    `next → pointer (4 bytes on 32-bit, 8 bytes on 64-bit)`

    The compiler knows pointer size.    
    It does NOT need to know the size of the pointed-to object.      
    A pointer is just an address.
    ```
    sizeof(struct Node) = 4 + 4 = 8 bytes (on 32-bit)
    ```
    Perfectly valid.

3. If this were legal:
    ```
    struct Node {
        int value;
        struct Node next;
    };
    ```
    Then each Node would contain:
    * an int
    * another full Node

    But that second Node contains:
    * an int
    * another full Node

    Which contains:
    * an int
    * another full Node

    And so on…

    **This never ends.**

### 🔥 The Real Problem

The compiler must know the **exact size** of a struct at compile time.

But here:
```c
sizeof(struct Node) = sizeof(int) + sizeof(struct Node)
```
* That equation is impossible.
* It creates infinite recursion.
* So the compiler rejects it.

### 🧠 Critical Insight

A struct cannot contain itself.
* But it CAN contain a pointer to itself.

* This is called:
    Self-referential structure

* And this is how linked lists are born.

> Self-referential structure

</details>



