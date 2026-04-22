# Unions in C

You already know a struct:

```c
struct Point {
    int x;
    int y;
};
```

Members `x` and `y` each get **their own memory**.

A union looks similar:

```c
union Data {
    int   i;
    float f;
};
```

But something is completely different.


## The Core Rule

> In a struct, every member gets its own memory.
> In a union, **all members share the same memory.**

```c
struct S {
    int   i;   /* 4 bytes */
    float f;   /* 4 bytes */
};
/* sizeof(struct S) = 8 */

union U {
    int   i;   /* 4 bytes */
    float f;   /* 4 bytes */
};
/* sizeof(union U) = 4 */
```

The union is only as large as its **biggest member**.

---

## Memory Picture

```
struct S layout:
0x1000   i  (4 bytes)
0x1004   f  (4 bytes)

union U layout:
0x1000   i  ←──┐
0x1000   f  ←──┘  same address
```

Both `i` and `f` start at `0x1000`.

Writing to `i` overwrites `f`. Writing to `f` overwrites `i`.

**Only one member holds valid data at a time.**

---

## 🔥 Mini Exercise 1

```c
union Sensor {
    uint8_t  raw;
    int8_t   calibrated;
};
```

Questions:

1. What is `sizeof(union Sensor)`?
2. What address does `raw` start at if union starts at `0x2000`?
3. What address does `calibrated` start at?

<details>
  <summary style="color:#93f022">Click to reveal the answers</summary>

<br>

1. **1 byte** — `unit8_t` and `int8_t` both are 1-byte, so `sizeof(union Sensor)= 1-byte`.
2. `raw` starts at `0x2000`.
3. `calibrated` also starts at `0x2000`.

Both members occupy the exact same byte.
If you write `0xFF` into `raw`, reading `calibrated` gives `-1`.
They are the same memory, interpreted differently.

</details>

---

## 🔥 Mini Exercise 2

```c
union Test {
    uint32_t  word;
    uint8_t   bytes[4];
};
```

Questions:

1. What is `sizeof(union Test)`?
2. If you write `word = 0x12345678`, what is `bytes[0]` on a little-endian machine?
3. What is `bytes[3]`?

<details>
  <summary style="color:#93f022">Click to reveal the answers</summary>

<br>

1. **4 bytes** — largest member is `uint32_t`.

2. On a **little-endian** machine, the least significant byte is stored first:

```
Address    Value
0x1000     0x78   ← bytes[0]
0x1001     0x56   ← bytes[1]
0x1002     0x34   ← bytes[2]
0x1003     0x12   ← bytes[3]
```

So `bytes[0] = 0x78`.

3. `bytes[3] = 0x12`.


This is one of the most common uses of unions in embedded systems.

You receive 4 raw bytes over UART.
You want to read them as a single `uint32_t`.

The union does it without any casting or pointer tricks.

</details>

---

## 🧠 Union Size Rule

Union size = size of its **largest member**, rounded up to alignment of largest member.

```c
union Example {
    uint8_t  a;    /* 1 byte  */
    uint16_t b;    /* 2 bytes */
    uint32_t c;    /* 4 bytes */
};
/* sizeof = 4 */
```

---

## 🔥 Advanced Thinking Question 1

```c
union Example {
    uint8_t  a;
    uint16_t b;
    uint32_t c;
};
```

If `union Example u` starts at `0x1000`:

1. What address does `a` occupy?
2. What address does `b` occupy?
3. You write `u.c = 0xAABBCCDD`. What is `u.a`?
4. What is `u.b`?

<details>
  <summary style="color:#93f022">Click to reveal the answers</summary>

<br>

All three members start at the **same address**: `0x1000`.

Memory after `u.c = 0xAABBCCDD` on a little-endian machine:

```
0x1000   0xDD   ← u.a reads this
0x1001   0xCC
0x1002   0xBB
0x1003   0xAA
```

1. `a` is at `0x1000`
2. `b` is at `0x1000`
3. `u.a = 0xDD` (lowest byte)
4. `u.b = 0xCCDD` (lowest two bytes)

### ⚠ Important Warning

Reading a union member you did not **most recently write** is called
**type-punning**.

In C, this is actually defined behaviour (C99 and later explicitly allow it).
In C++, it is undefined behaviour.

This is one reason the union trick is used in **C firmware**, not C++.

</details>


## 🔥 Advanced Thinking Question 2

```c
typedef union
{
    uint8_t value[2]; 

    struct
    {
        uint8_t _0_Index;
        uint8_t _1_Index;
    };

} U8x2_t;
```
```c
typedef union
{
    uint16_t value; 

    struct
    {
        uint8_t low;
        uint8_t high;
    };

} U16_t;
```

1. Are these two identical? Explain why?
2. What are the outputs of:
```c
U8x2_t u8x2;
u8x2.value[0] = 0x12;
u8x2.value[1] = 0x34;
printf("_0_Index: 0x%x, _1_Index: 0x%x \r\n", u8x2._0_Index, u8x2._1_Index);


U16_t u16;
u16.value = 0x1234;
printf("low: 0x%x, high: 0x%x \r\n", u16.low, u16.high);
```

<details>
  <summary style="color:#93f022">Click to reveal the answers</summary>

<br>

1. Are these two identical?

    No, they look similar but behave differently.    
    * `U8x2_t`: 
        * This is just two separate bytes.
        * `value[0]` maps directly to **_0_Index**
        * `value[1]` maps directly to **_1_Index**
        * There is no interpretation issue ➡ it's purely byte-based.

    * `U16_t`:
        * This stores a `16-bit` unsigned integer
        * Then reinterpret that memory as two bytes
        * The order of **low** and **high**

2. Output of the code

    * `U8x2_t`:
         
        ```c
        u8x2.value[0] = 0x12;
        u8x2.value[1] = 0x34;
        ```
        Memory layout:   
        ```c
        [0x12][0x34]
        ```
        So Output:
        ```c
        _0_Index: 0x12, _1_Index: 0x34
        ```



    * `U16_t`:

        ```c
        u16.value = 0x1234;
        ```

        Memory:
        ```c
        [0x34][0x12]
        ```
        So output:
        ```c
        low: 0x34, high: 0x12
        ```

    * **Extra knowledge** (Below is a byte-level and bit-level view of how data is stored.):    
        * Case 1: `U8x2_t`:
            ```c
            u8x2.value[0] = 0x12;
            u8x2.value[1] = 0x34;
            ```
            Byte representation
            ```
            Address →     +0       +1
                        ------    ------
                         0x12      0x34
            ```
            Bit representation
            ```
            0x12 = 0001 0010
            0x34 = 0011 0100

            Memory:
            +0 → 0001 0010
            +1 → 0011 0100
            ```

        * Case 2: `U16_t` :
            ```c
            u16.value = 0x1234;
            ```
            16-bit value:
            ```
            0x1234 = 0001 0010 0011 0100
                         ↑ high    ↑ low (logical view)
            ```
            Byte representation
            ```
            Address →   +0        +1
                       ------    ------
                        0x34      0x12
            ```
            Bit representation
            ```
            +0 → 0011 0100   (low byte)
            +1 → 0001 0010   (high byte)
            ```
        

**Note: `Big-Endian` System both are identical. (rare today)**

A big-endian system stores the most significant byte (MSB) of a multi-byte data value at the lowest memory address, effectively ordering bytes from "biggest" to "smallest". For example, the 32-bit hexadecimal value 0x12345678 is stored sequentially as 12 34 56 78 starting from the lowest address.

</details>

<br>
<br>
<br>

---

## Real-World Pattern: Tagged Union

In embedded systems, a union is often combined with a field that
tells you **which member is currently valid**. This is called a
**tagged union** (or discriminated union).

```c
typedef enum {
    SENSOR_TEMP     = 0,
    SENSOR_PRESSURE = 1,
    SENSOR_VOLTAGE  = 2
} SensorType_t;

typedef struct {
    SensorType_t type;      /* tag — tells us which field is valid */
    union {
        int8_t   temp_c;
        uint32_t pressure_pa;
        uint16_t voltage_mv;
    } value;
} SensorReading_t;
```

Usage:

```c
SensorReading_t r;
r.type          = SENSOR_VOLTAGE;
r.value.voltage_mv = 3300;

/* later, reading it safely */
if (r.type == SENSOR_VOLTAGE) {
    process_voltage(r.value.voltage_mv);
}
```

### 🧠 Why This Matters

Without the tag you cannot safely know which field to read.
The tag makes the intent explicit and prevents reading stale data.

This pattern appears everywhere in real firmware:
CAN bus frames, telemetry packets, sensor drivers.

---

## 🚀 Now You Are Ready

You have seen:

* A union shares memory — only one member valid at a time.
* Size = largest member.
* All members start at the same address.
* Unions are used for type-punning, byte access, and tagged variants.

Now we will apply this directly to a real embedded protocol parser,
where a union overlays a raw byte buffer with a typed packet struct —
eliminating all manual pointer arithmetic.

---