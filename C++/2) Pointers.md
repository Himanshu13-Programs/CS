# 📘 C/C++ Chapter 2 — Pointers & Memory Management 
---

## 📌 Table of Contents
1. [Pointer Basics](#1-pointer-basics)
2. [Pointer Arithmetic](#2-pointer-arithmetic)
3. [Pointers & Arrays](#3-pointers--arrays)
4. [Pointers to Pointers](#4-pointers-to-pointers)
5. [Pointers & Functions](#5-pointers--functions)
6. [Dynamic Memory — malloc/calloc/realloc/free](#6-dynamic-memory--malloccallocreallocfree)
7. [C++ new & delete](#7-c-new--delete)
8. [Common Memory Errors](#8-common-memory-errors)
9. [Function Pointers](#9-function-pointers)
10. [Output Prediction Drill](#10-output-prediction-drill)
11. [MCQ Traps & Exam Q&A](#11-mcq-traps--exam-qa)

---

## 1. Pointer Basics

### 🔧 Declarations & Operations
```c
int x = 10;
int *p;          // p is a pointer to int (uninitialized — DANGEROUS)
p = &x;          // & = address-of operator: p now holds address of x

printf("%d\n",  x);   // 10  (value of x)
printf("%p\n",  p);   // 0x... (address stored in p = address of x)
printf("%p\n", &x);   // same address
printf("%d\n", *p);   // 10  (* = dereference: value AT address p points to)
printf("%p\n", &p);   // address of p itself (different from p!)

*p = 20;             // change value at address p points to
printf("%d\n", x);   // 20 (x changed through pointer!)
```

### 🔧 The Three Uses of `*`
```c
int *p;    // DECLARATION: * means "p is a pointer to int"
*p = 5;    // DEREFERENCE: * means "value at address p"
int y = 2 * 3;  // MULTIPLICATION: just multiply
```

### 🔧 NULL Pointer
```c
int *p = NULL;   // p points to nothing (address 0)
// Always initialize pointers to NULL if not assigned yet
// Dereferencing NULL → segmentation fault (crashes)

if (p != NULL) {
    *p = 5;      // safe: only dereference if not NULL
}
// or simply: if (p) { ... }
```

### 🔧 const with Pointers — 4 Combinations
```c
int x = 10, y = 20;

// 1. Pointer to const (can't change value through pointer)
const int *p1 = &x;
*p1 = 5;     // ERROR: can't modify value through p1
p1 = &y;     // OK: can change where p1 points

// 2. Const pointer (can't change where pointer points)
int * const p2 = &x;
*p2 = 5;     // OK: can modify value
p2 = &y;     // ERROR: can't change where p2 points

// 3. Const pointer to const (can't change either)
const int * const p3 = &x;
*p3 = 5;     // ERROR
p3 = &y;     // ERROR

// 4. Non-const pointer to non-const (regular pointer)
int *p4 = &x;
*p4 = 5;     // OK
p4 = &y;     // OK

// TRICK: Read right-to-left from the variable name:
// const int *p  → p is a pointer(*) to const int → value is const
// int * const p → p is a const pointer(*) to int → pointer is const
```

---

## 2. Pointer Arithmetic

### 🔧 Rules
```c
int arr[] = {10, 20, 30, 40, 50};
int *p = arr;   // p points to arr[0]

// Pointer arithmetic moves by sizeof(type):
printf("%d\n", *p);      // 10 (arr[0])
printf("%d\n", *(p+1));  // 20 (arr[1]) — moves 4 bytes forward
printf("%d\n", *(p+2));  // 30 (arr[2])

p++;             // p now points to arr[1]
printf("%d\n", *p);      // 20

p += 2;          // p now points to arr[3]
printf("%d\n", *p);      // 40

// Pointer difference:
int *q = &arr[4];
printf("%ld\n", q - p);  // 1 (arr[4] - arr[3] = 1 element)
// Difference gives number of ELEMENTS, not bytes!

// Valid operations:
// ptr + n   (move forward n elements)
// ptr - n   (move backward n elements)
// ptr1 - ptr2  (distance in elements, if same array)
// ptr < ptr2   (compare positions, if same array)

// INVALID operations:
// ptr + ptr  (can't add two pointers)
// ptr * n    (can't multiply pointer)
```

### 🔧 Pointer Arithmetic with Different Types
```c
char  *cp = (char*)1000;
int   *ip = (int*)1000;
double *dp = (double*)1000;

cp++;   // cp = 1001 (char = 1 byte)
ip++;   // ip = 1004 (int  = 4 bytes)
dp++;   // dp = 1008 (double = 8 bytes)

// p++ moves by sizeof(*p) bytes
```

---

## 3. Pointers & Arrays

### 🔧 Array-Pointer Equivalence
```c
int arr[] = {10, 20, 30, 40, 50};
int *p = arr;   // array name decays to pointer to first element

// These are ALL equivalent:
arr[2]     == *(arr + 2)  == *(p + 2)  == p[2]   // value = 30
&arr[2]    == arr + 2     == p + 2               // address

// CRITICAL DIFFERENCE:
// arr is an ARRAY (fixed, can't be reassigned)
// p is a POINTER (can be moved)
arr = arr + 1;  // ERROR! arr is not a variable
p = p + 1;      // OK! p is a pointer variable

// sizeof difference:
sizeof(arr) = 20   // total array size (5 * 4)
sizeof(p)   = 8    // just the pointer (on 64-bit)
```

### 🔧 Array of Pointers vs Pointer to Array
```c
// Array of pointers — each element is a pointer:
int *ap[5];          // 5 pointers to int
ap[0] = &arr[0];

// Pointer to array — one pointer to entire array:
int (*pa)[5] = &arr; // pointer to array of 5 ints
printf("%d\n", (*pa)[2]);  // 30

// TRICK to read declarations (right-to-left from name):
// int *ap[5] → ap is an array[5] of pointers(*) to int
// int (*pa)[5] → pa is a pointer(*) to array[5] of int
```

### 🔧 Strings as Char Pointers
```c
// String literal: stored in read-only memory
char *str1 = "Hello";     // str1 points to read-only "Hello"
str1[0] = 'h';            // UNDEFINED BEHAVIOR (may crash)

// Character array: stored in writable stack memory
char str2[] = "Hello";    // copy of "Hello" on stack
str2[0] = 'h';            // OK! str2 is modifiable

// Key difference:
char *s = "Hello";     // pointer to string literal (read-only)
char a[] = "Hello";    // array containing copy (read-write)
```

### 🔧 2D Arrays and Pointers
```c
int matrix[3][4];

// matrix[i][j] is equivalent to:
// *(*(matrix + i) + j)
// *(matrix[i] + j)

int (*ptr)[4] = matrix;  // ptr is a pointer to array of 4 ints
printf("%d\n", ptr[1][2]);  // matrix[1][2]
```

---

## 4. Pointers to Pointers

### 🔧 Double Pointer
```c
int x = 10;
int *p = &x;     // p points to x
int **pp = &p;   // pp points to p

printf("%d\n",   x);    // 10
printf("%d\n",  *p);    // 10
printf("%d\n", **pp);   // 10

**pp = 20;              // change x through double pointer
printf("%d\n", x);      // 20

// Memory model:
// pp → [ p's address ] → [ x's address ] → [ 10 ]
//  pp        p                x

// Use case: modify a pointer inside a function
void allocate(int **ptr, int size) {
    *ptr = malloc(size * sizeof(int));  // modifies the pointer itself
}
int *arr = NULL;
allocate(&arr, 10);  // pass address of pointer
```

### 🔧 Array of Strings (char**)
```c
// Common pattern for multiple strings:
char *names[] = {"Alice", "Bob", "Charlie"};
// names is an array of char pointers

for (int i = 0; i < 3; i++) {
    printf("%s\n", names[i]);   // print each string
}

// main function's argv:
int main(int argc, char *argv[]) {  // argv is char** (or char*[])
    printf("%s\n", argv[0]);  // program name
    printf("%d\n", argc);     // number of arguments
}
```

---

## 5. Pointers & Functions

### 🔧 Pass by Value vs Pass by Pointer
```c
// Pass by VALUE — function gets a COPY, original unchanged:
void addTen_val(int x) {
    x += 10;    // modifies local copy only
}

// Pass by POINTER — function gets address, can modify original:
void addTen_ptr(int *x) {
    *x += 10;   // modifies value at address
}

int main() {
    int a = 5;
    addTen_val(a);
    printf("%d\n", a);    // 5 (unchanged)

    addTen_ptr(&a);
    printf("%d\n", a);    // 15 (changed!)
    return 0;
}
```

### 🔧 Returning Pointers — Common Trap
```c
// WRONG: returning pointer to local variable (dangling pointer!)
int* bad_func() {
    int x = 10;
    return &x;    // x is destroyed when function returns!
    // accessing *returned_ptr is UNDEFINED BEHAVIOR
}

// CORRECT: return pointer to static or dynamically allocated memory:
int* good_static() {
    static int x = 10;   // static: persists after function returns
    return &x;           // OK (but shared across calls)
}

int* good_dynamic() {
    int *p = malloc(sizeof(int));
    *p = 10;
    return p;    // caller must free() this!
}

// CORRECT: modify through pointer parameter (don't return pointer):
void good_param(int *result) {
    *result = 10;
}
```

### 🔧 Array as Function Parameter
```c
// All three are equivalent in function parameters:
void func1(int arr[]) { ... }
void func2(int arr[5]) { ... }  // size ignored!
void func3(int *arr) { ... }    // same as above

// Passing 2D array:
void func(int arr[][4], int rows) { ... }    // columns must be specified
void func(int (*arr)[4], int rows) { ... }   // same

// Cannot determine array size inside function from parameter alone:
void func(int *arr) {
    sizeof(arr);  // = 8 (pointer size, NOT array size!)
    // Must pass size separately
}
```

---

## 6. Dynamic Memory — malloc/calloc/realloc/free

### 🔧 malloc — Memory Allocation
```c
#include <stdlib.h>

// malloc(size_in_bytes): allocates uninitialized memory on HEAP
// Returns: void* pointer to block, or NULL if failed

int *p = (int*)malloc(5 * sizeof(int));  // allocate 5 ints
if (p == NULL) {
    printf("Allocation failed\n");
    exit(1);
}

// Always check for NULL before using!
// ALWAYS free when done:
free(p);
p = NULL;  // good practice: avoid dangling pointer
```

### 🔧 calloc — Initialized Allocation
```c
// calloc(count, size_each): allocates AND ZEROS the memory
int *p = (int*)calloc(5, sizeof(int));
// All 5 ints initialized to 0 (unlike malloc which has garbage)

// malloc equivalent of calloc:
int *q = (int*)malloc(5 * sizeof(int));
memset(q, 0, 5 * sizeof(int));   // zero out manually
```

### 🔧 realloc — Resize Allocation
```c
int *p = (int*)malloc(5 * sizeof(int));

// Grow to 10 ints:
p = (int*)realloc(p, 10 * sizeof(int));
// May return same pointer (if space available)
// Or new pointer (data copied to new location)
// Old pointer INVALID if new location returned!

// CORRECT pattern:
int *temp = (int*)realloc(p, 10 * sizeof(int));
if (temp == NULL) {
    // realloc failed, original p still valid
    free(p);
    return;
}
p = temp;  // safe to update p now
```

### 🔧 free
```c
int *p = malloc(sizeof(int));
free(p);     // releases memory back to OS/heap

// TRAPS:
free(p);
free(p);     // DOUBLE FREE — undefined behavior, may crash!

int x = 5;
int *q = &x;
free(q);     // WRONG — only free heap memory (malloc/calloc/realloc)

free(NULL);  // OK — freeing NULL is a no-op (safe)

// After free, always NULL the pointer:
p = NULL;   // prevents accidental use of dangling pointer
```

### 🔧 Stack vs Heap Memory
```
STACK:                          HEAP:
- Function local variables      - malloc/calloc/realloc
- Fixed size (usually ~1-8 MB)  - Large (limited by RAM)
- Automatic management          - Manual management (free!)
- LIFO allocation               - Fragmented over time
- Fast access                   - Slightly slower (pointer indirection)
- Destroyed when func returns   - Lives until free() called

Stack overflow: too deep recursion or too large local array.
Memory leak: malloc without free → heap fills up over time.
```

### 📝 Dynamic 2D Array
```c
// Method 1: Array of pointers (rows can be different sizes)
int **matrix = (int**)malloc(rows * sizeof(int*));
for (int i = 0; i < rows; i++) {
    matrix[i] = (int*)malloc(cols * sizeof(int));
}
matrix[1][2] = 42;

// Free:
for (int i = 0; i < rows; i++) free(matrix[i]);
free(matrix);

// Method 2: Single block (contiguous, faster access)
int *flat = (int*)malloc(rows * cols * sizeof(int));
flat[i * cols + j] = 42;  // access as flat[row * cols + col]
free(flat);
```

---

## 7. C++ new & delete

### 🔧 new and delete
```cpp
// new: allocate single object
int *p = new int;         // uninitialized
int *q = new int(10);     // initialized to 10
int *r = new int{10};     // C++11 uniform initialization

// delete: free single object
delete p;
delete q;
p = nullptr;  // C++11: use nullptr instead of NULL

// new[]: allocate array
int *arr = new int[10];         // array of 10 ints
int *arr2 = new int[10]{};      // zero-initialized
int *arr3 = new int[5]{1,2,3};  // {1,2,3,0,0}

// delete[]: free array (MUST match new[])
delete[] arr;
delete[] arr2;
// delete arr; ← WRONG! Must use delete[] for arrays
```

### 🔧 new vs malloc Key Differences
| Feature | malloc | new |
|---|---|---|
| **Language** | C and C++ | C++ only |
| **Type** | Returns void* (must cast) | Returns typed pointer (no cast) |
| **Constructor** | Not called | Called automatically |
| **Destructor** | Not called by free() | Called by delete |
| **Failure** | Returns NULL | Throws std::bad_alloc |
| **Resize** | realloc() | No direct equivalent |
| **Size arg** | You specify bytes | You specify type/count |

```cpp
// malloc requires manual cast and sizeof:
int *p = (int*)malloc(sizeof(int));

// new is cleaner:
int *q = new int;

// new calls constructors — critical for objects:
class Foo { Foo() { cout << "constructed\n"; } };
Foo *f1 = (Foo*)malloc(sizeof(Foo));  // NO constructor called!
Foo *f2 = new Foo;                    // constructor called ✓
```

---

## 8. Common Memory Errors

### 🔧 1. Memory Leak
```c
void leak() {
    int *p = malloc(100 * sizeof(int));
    // ... use p ...
    return;  // forgot to free! Memory lost forever until program ends.
}
// Fix: always free before function returns
```

### 🔧 2. Dangling Pointer
```c
int *p = malloc(sizeof(int));
*p = 10;
free(p);
printf("%d\n", *p);  // DANGLING: p freed but still used!
// Fix: p = NULL after free, check before use
```

### 🔧 3. Double Free
```c
int *p = malloc(sizeof(int));
free(p);
free(p);   // DOUBLE FREE — crashes or corrupts heap
// Fix: p = NULL after first free (free(NULL) is safe)
```

### 🔧 4. Buffer Overflow
```c
int arr[5];
arr[10] = 42;   // writes beyond array — overwrites other memory!
// Can corrupt stack, cause crashes, security vulnerability
```

### 🔧 5. Use Before Initialize
```c
int *p;          // uninitialized pointer (garbage address)
*p = 5;          // CRASH: writes to random memory location
// Fix: always initialize: int *p = NULL; or int *p = malloc(...);
```

### 🔧 6. Stack Overflow
```c
void infinite() {
    int arr[1000000];  // 4MB on stack!
    infinite();        // recursive with no base case
}
// Stack is typically 1-8 MB — easily overflowed
```

---

## 9. Function Pointers

### 🔧 Declaration and Use
```c
// int (*fp)(int, int)  means:
// fp is a pointer to function taking (int,int) returning int

int add(int a, int b) { return a + b; }
int mul(int a, int b) { return a * b; }

int (*fp)(int, int);    // declare function pointer
fp = add;               // assign (no & needed for functions)
printf("%d\n", fp(3,4)); // call through pointer: 7

fp = mul;
printf("%d\n", fp(3,4)); // 12

// Array of function pointers:
int (*ops[3])(int,int) = {add, mul, /* sub */};
printf("%d\n", ops[0](5,3));  // 8
printf("%d\n", ops[1](5,3));  // 15

// typedef for cleaner syntax:
typedef int (*BinaryOp)(int, int);
BinaryOp op = add;
printf("%d\n", op(2,3));  // 5
```

### 🔧 Callback Functions
```c
// Passing function as argument:
void applyToArr(int *arr, int n, int (*func)(int)) {
    for (int i = 0; i < n; i++) {
        arr[i] = func(arr[i]);
    }
}

int doubleIt(int x) { return x * 2; }
int square(int x) { return x * x; }

int arr[] = {1, 2, 3, 4, 5};
applyToArr(arr, 5, doubleIt);  // {2,4,6,8,10}
applyToArr(arr, 5, square);    // {4,16,36,64,100}

// qsort uses callback:
int compare(const void *a, const void *b) {
    return (*(int*)a - *(int*)b);
}
qsort(arr, 5, sizeof(int), compare);
```

---

## 10. Output Prediction Drill

### 🎯 Question 1 — Basic Pointer
```c
#include <stdio.h>
int main() {
    int x = 5, y = 10;
    int *p = &x;
    *p = 20;
    p = &y;
    *p += 5;
    printf("%d %d\n", x, y);
    return 0;
}
```
**Predict then check:**
```
*p = 20: changes x to 20. p still points to x.
p = &y:  p now points to y. x unchanged.
*p += 5: y = y + 5 = 15.

Output: 20 15
```

---

### 🎯 Question 2 — Pointer Arithmetic
```c
#include <stdio.h>
int main() {
    int arr[] = {10, 20, 30, 40, 50};
    int *p = arr + 2;
    printf("%d\n", *p);
    printf("%d\n", *(p-1));
    printf("%d\n", p[1]);
    printf("%d\n", *(p+2));
    p--;
    printf("%d\n", *p);
    return 0;
}
```
**Predict then check:**
```
p = arr + 2 → points to arr[2] = 30
*p = 30
*(p-1) = arr[1] = 20
p[1] = *(p+1) = arr[3] = 40
*(p+2) = arr[4] = 50
p-- → p now points to arr[1]
*p = 20

Output:
30
20
40
50
20
```

---

### 🎯 Question 3 — Double Pointer
```c
#include <stdio.h>
int main() {
    int a = 1, b = 2;
    int *p = &a;
    int **pp = &p;

    printf("%d\n", **pp);  // (1)
    *pp = &b;               // pp changes where p points
    printf("%d\n", *p);    // (2)
    printf("%d\n", **pp);  // (3)
    **pp = 99;
    printf("%d %d\n", a, b); // (4)
    return 0;
}
```
**Predict then check:**
```
**pp = *p = a = 1          → (1) prints 1

*pp = &b: changes p to point to b.
  p now points to b.

*p = b = 2                 → (2) prints 2
**pp = *p = b = 2          → (3) prints 2

**pp = 99: b = 99. a unchanged.
                            → (4) prints 1 99

Output:
1
2
2
1 99
```

---

### 🎯 Question 4 — malloc and Pointer
```c
#include <stdio.h>
#include <stdlib.h>
int main() {
    int *p = (int*)malloc(3 * sizeof(int));
    for (int i = 0; i < 3; i++)
        p[i] = (i+1) * 10;

    int *q = p;
    q[1] = 99;

    for (int i = 0; i < 3; i++)
        printf("%d ", p[i]);
    printf("\n");

    free(p);
    return 0;
}
```
**Predict then check:**
```
p[0]=10, p[1]=20, p[2]=30

q = p: q and p point to SAME memory.
q[1] = 99: changes p[1] too!

p: {10, 99, 30}

Output: 10 99 30
```

---

### 🎯 Question 5 — const Pointer Trap
```c
#include <stdio.h>
int main() {
    int x = 10, y = 20;
    const int *p = &x;    // pointer to const int
    p = &y;               // (A) is this OK?
    // *p = 30;           // (B) is this OK?
    printf("%d\n", *p);
    return 0;
}
```
**Predict then check:**
```
const int *p: the VALUE is const, but the POINTER can change.
(A) p = &y: OK! Pointer itself is not const. p now points to y.
(B) *p = 30: NOT OK! Can't modify value through const pointer. (Commented out = fine)

*p = y = 20

Output: 20
```

---

### 🎯 Question 6 — Function Modifying via Pointer
```c
#include <stdio.h>
void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}
void badSwap(int a, int b) {
    int temp = a;
    a = b;
    b = temp;
}
int main() {
    int x = 5, y = 10;
    badSwap(x, y);
    printf("%d %d\n", x, y);  // (1)
    swap(&x, &y);
    printf("%d %d\n", x, y);  // (2)
    return 0;
}
```
**Predict then check:**
```
badSwap(x,y): passes copies. Original x,y unchanged.
  (1): 5 10

swap(&x,&y): passes addresses. *a and *b are x and y.
  temp=5, *a=10, *b=5 → x=10, y=5
  (2): 10 5

Output:
5 10
10 5
```

---

### 🎯 Question 7 — Pointer to Pointer Swap
```c
#include <stdio.h>
void swapPtrs(int **a, int **b) {
    int *temp = *a;
    *a = *b;
    *b = temp;
}
int main() {
    int x = 100, y = 200;
    int *p = &x, *q = &y;
    swapPtrs(&p, &q);
    printf("%d %d\n", *p, *q);
    printf("%d %d\n", x, y);
    return 0;
}
```
**Predict then check:**
```
swapPtrs(&p, &q): swaps the POINTERS (not the values).
  After: p points to y, q points to x.
  x and y themselves are unchanged.

*p = y = 200, *q = x = 100
x = 100, y = 200 (unchanged)

Output:
200 100
100 200
```

---

### 🎯 Question 8 — sizeof Traps
```c
#include <stdio.h>
int func(int arr[]) {
    return sizeof(arr);
}
int main() {
    int arr[10];
    printf("%lu\n", sizeof(arr));      // (1)
    printf("%lu\n", sizeof(arr)/sizeof(arr[0])); // (2)
    printf("%d\n",  func(arr));        // (3)
    int *p = arr;
    printf("%lu\n", sizeof(p));        // (4)
    return 0;
}
```
**Predict then check:**
```
(1) sizeof(arr) where arr is array: 10 * 4 = 40
(2) 40 / 4 = 10 (array length)
(3) func receives arr as int* (pointer). sizeof(pointer) = 8 (64-bit).
(4) sizeof(p) where p is pointer: 8

Output:
40
10
8
8
```

---

## 11. MCQ Traps & Exam Q&A

### ⚠️ THE TRAPS

---

**TRAP 1: `p++` vs `(*p)++` vs `*p++`**
```c
int x = 5;
int *p = &x;

(*p)++    // increment VALUE: x becomes 6, p unchanged
*(p++)    // post-increment pointer: returns *p (=5), then p moves
*p++      // SAME as *(p++) due to precedence (++ binds tighter than *)
++(*p)    // pre-increment value: x becomes 6
*(++p)    // pre-increment pointer: p moves first, then dereference
```

---

**TRAP 2: Array name ≠ pointer (for sizeof)**
```c
int arr[5];
int *p = arr;
sizeof(arr) = 20  // array: total bytes
sizeof(p)   = 8   // pointer: always pointer size
sizeof(*p)  = 4   // int: one element size
```

---

**TRAP 3: Returning local variable address**
```c
int* bad() {
    int x = 5;
    return &x;   // x dies when function returns → dangling!
}
// Accessing returned pointer = undefined behavior
```

---

**TRAP 4: `malloc` doesn't zero memory**
```c
int *p = malloc(sizeof(int));
printf("%d\n", *p);  // GARBAGE (uninitialized)
// Use calloc for zero-initialized memory
```

---

**TRAP 5: `delete` vs `delete[]`**
```cpp
int *p = new int;
int *arr = new int[10];
delete p;       // correct for single object
delete[] arr;   // MUST use delete[] for arrays
delete arr;     // UNDEFINED BEHAVIOR (UB!)
```

---

**TRAP 6: Pointer comparison**
```c
char *s1 = "hello";
char *s2 = "hello";
if (s1 == s2)  // comparing addresses, NOT content!
// May be true (compiler may optimize to same literal) or false
// Always use strcmp for string content comparison
```

---

**TRAP 7: void pointer**
```c
void *vp;           // can hold any pointer type
int x = 5;
vp = &x;            // OK — any type can assign to void*
int *ip = vp;       // C: OK (implicit). C++: MUST cast explicitly
int *ip2 = (int*)vp;// Always safe
*vp = 10;           // ERROR: can't dereference void*
```

---

### ⚡ Quick Fire Q&A

| Question | Answer |
|---|---|
| `int *p = &x` — what does `&` do? | Address-of: gets memory address of x |
| `*p = 5` — what does `*` do here? | Dereference: write 5 at the address p holds |
| `int * const p` vs `const int *p`? | First: pointer is const. Second: value is const. |
| After `p++`, p moves by how much if `int*`? | 4 bytes (sizeof int) |
| Can you add two pointers? | No — only subtract (gives element count) |
| `malloc` returns what type? | `void*` (must cast in C++, optional in C) |
| Difference: `free(p)` vs `delete p`? | free for malloc, delete for new (C++) |
| After `free(p)`, is p still valid? | No — it's a dangling pointer (set to NULL!) |
| `calloc(5, 4)` allocates how many bytes? | 20, all zeroed |
| What does `realloc(p, 0)` do? | Equivalent to free(p) |
| Array parameter in function is really a? | Pointer (int arr[] = int *arr) |
| sizeof on pointer parameter gives? | Pointer size (8), NOT array size |
| How to prevent double-free bug? | Set pointer to NULL after free |
| What is a memory leak? | Heap-allocated memory never freed |
| `int (*fp)(int)` — what is fp? | Pointer to function taking int, returning int |
| `new int[5]` freed with? | `delete[]` (NOT `delete`) |
| Difference: NULL vs nullptr? | NULL is 0 (int macro), nullptr is C++11 typed null pointer |
| `*(arr + i)` is same as? | `arr[i]` |
| Can array name be reassigned? | No (not an lvalue) — only pointers can be moved |

---
