
---

# Build a Correct Mental Model of Memory


## 🔹 What Is a Variable in C?

When you write:

```c
int x = 10;
```

Three things happen:

1. Memory is reserved (usually on stack)
2. That memory has an **address**
3. Value `10` is stored at that address

Think of memory as:

```
Address      Value
0x1000       10
```

And `x` is just a **label** for address `0x1000`.

---

## 🔹 Important Truth #1

> A variable name is NOT the memory.
> It is a label for a memory location.

This is where pointer clarity begins.

---

## 🔹 Stack vs Heap (Very Important)

### Stack

* Automatic memory
* Local variables
* Fast
* Destroyed automatically

```c
void func() {
    int a = 5;  // stack
}
```

`a` dies when function ends.

---

### Heap

* Dynamic memory
* Manual control
* Slower
* You must free it

```c
int *p = malloc(sizeof(int));
```

This allocates memory somewhere in heap.

---

## 🔹 Important Truth #2

Stack and Heap are just different regions of memory.

Pointers don’t care where memory is.

---

# 🔥 Mini Exercise 1 (Think, don’t compile yet)

Consider:

```c
int main() {
    int x = 5;
    int y = x;
    y = 10;
    return 0;
}
```

Questions:

1. Does `x` change?
2. Why?
3. How many memory locations are used?
4. Are `x` and `y` related after assignment?



<details>
  <summary style="color:#93f022">Click to reveal the answer to your question</summary>

<br>

1.    
    No.

2.   
    Because `y = x` copies the value stored at `x`’s address, not the address itself.

3.    
    Two distinct stack locations.

4.     
    No. After the copy, they are completely independent.

-----

</details>