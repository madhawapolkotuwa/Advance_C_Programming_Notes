
---

# 🚀 Pointer Arithmetic

Now we go deeper.

Consider:

```c
int arr[3] = {10, 20, 30};
int *p = arr;
```

Important:

`arr` decays to `&arr[0]`

So:

```c
p = address of arr[0]
```

If `arr` starts at `0x1000`, and `int` is 4 bytes:

```c
Address     Value
0x1000      10
0x1004      20
0x1008      30
```

Now:

```c
p + 1
```

Does NOT mean:

```c
0x1000 + 1
```

It means:

```c
0x1000 + sizeof(int)
```

So:

```c
0x1004
```

---

## 🧠 Massive Insight

Pointer arithmetic is scaled by the size of the pointed type.

If:

```c
char *p;
```

Then `p + 1` → next byte.

If:

```c
int *p;
```

Then `p + 1` → next 4 bytes (on most systems).

---

## 🔥 Mini Exercise 4 (Important)

Assume:

* `int` = 4 bytes
* `arr` starts at address `0x1000`

```c
int arr[5] = {1,2,3,4,5};
int *p = arr;
```

Questions:

1. What is the address of `arr[3]`?
2. What is the value of `*(p + 2)`?
3. What is the address stored in `(p + 4)`?
4. What is the difference between:

   ```c
   p + 1
   *p + 1
   ```

<details>
  <summary style="color:#93f022">Click to reveal the answers to the questions</summary>

<br>

1. &arr[3] ?
```c
&arr[0] ➞ 0x1000
&arr[1] ➞ 0x1004
&arr[2] ➞ 0x1008
&arr[3] ➞ 0x100C   
&arr[4] ➞ 0x1010  
```

2. *(p + 2) ?
```c
p ➞ 0x1000  
p + 2 ➞ 0x1000 + (2 * sizeof(int)) ➞ 0x1000 + 8 ➞ 0x1008  
*(p + 2) ➞ *(0x1008) ➞ (3rd element)arr[2] → 3
```

3. (p + 4) ?
```c
p + 4 = 0x1000 + (4 * 4) = 0x1010 (4th element address)
```

4. (p + 1) vs (*p + 1)
```c
p + 1 ➞ 0x1000 + (1 * 4) ➞ 0x1004 ➞ &arr[1]

*p + 1 (It does NOT modify memory) ➞ *(0x1000) + 1 ➞ arr[0] + 1  ➞ 1 + 1 = 2 
```

### 🧠 Big Insight

There is a massive difference between:
```c
*p + 1      // just calculates
```

and
```c
*p = *p + 1;  // modifies memory 
*p += 1;     
(*p)++;
```
* modifies memory just like this (p + 1) that because it store back to the pointer.
* `One missing = changes everything.`

```c
x = *p + 1  // just calculates
x* = *p + 1 // modifies memory   
```

</details>
