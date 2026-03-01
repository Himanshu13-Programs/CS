# 📘 C/C++ Chapter 3 — Functions, Recursion & Scope
---

## 1. Functions — Deep Dive

### 🔧 Function Basics
```c
// Declaration (prototype) — tells compiler signature before definition
int factorial(int n);   // return_type name(params)

// Definition — actual code
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

// Call
int result = factorial(5);  // 120
```

### 🔧 Parameter Passing — 3 Ways in C/C++

#### Pass by Value (C and C++)
```c
void increment(int x) {
    x++;          // modifies local copy only
}
int a = 5;
increment(a);
printf("%d\n", a);  // 5 — unchanged
```

#### Pass by Pointer (C and C++)
```c
void increment(int *x) {
    (*x)++;       // modifies original via address
}
int a = 5;
increment(&a);
printf("%d\n", a);  // 6 — changed!
```

#### Pass by Reference (C++ only)
```cpp
void increment(int &x) {  // & in parameter = reference
    x++;                   // directly modifies original
}
int a = 5;
increment(a);              // no & needed at call site
cout << a;  // 6 — changed!

// Reference vs pointer:
// Reference: must be initialized, can't be NULL, can't be reseated
// Pointer: can be NULL, can point to different things
```

### 🔧 Return Types
```c
void func() { return; }          // returns nothing
int func() { return 42; }        // returns int
int* func() { ... return ptr; }  // returns pointer (be careful!)
int& func() { ... return ref; }  // returns reference (C++ — be careful!)

// Returning multiple values in C — use pointer parameters or struct:
typedef struct { int q, r; } DivResult;
DivResult divmod(int a, int b) {
    return (DivResult){a/b, a%b};
}
```

### 🔧 Variable-Length Arguments (varargs)
```c
#include <stdarg.h>
int sum(int count, ...) {
    va_list args;
    va_start(args, count);
    int total = 0;
    for (int i = 0; i < count; i++) {
        total += va_arg(args, int);
    }
    va_end(args);
    return total;
}
printf("%d\n", sum(3, 10, 20, 30));  // 60
// printf itself uses varargs!
```

---

## 2. Scope, Linkage & Storage Classes

### 🔧 Scope
```c
int global = 100;   // file scope — visible everywhere in file

void func() {
    int local = 10;   // block scope — only inside func
    {
        int inner = 20;    // block scope — only inside {}
        printf("%d\n", local);  // OK
        printf("%d\n", global); // OK
    }
    // inner not accessible here
    printf("%d\n", local);   // OK
}

// Shadowing:
int x = 5;    // global x
void shadow() {
    int x = 10;   // local x SHADOWS global x
    printf("%d\n", x);   // 10 (local)
    printf("%d\n", ::x); // 5  (global, C++ only with ::)
}
```

### 🔧 Storage Classes

#### auto (default for local variables)
```c
void func() {
    auto int x = 5;   // redundant — all locals are auto by default
    int y = 5;        // same as above
    // Allocated on STACK, destroyed when function returns
}
```

#### static
```c
// 1. static local variable: persists between function calls
void counter() {
    static int count = 0;  // initialized ONCE, persists after return
    count++;
    printf("%d\n", count);
}
counter();  // 1
counter();  // 2
counter();  // 3

// 2. static global variable: file scope only (not visible in other files)
static int filePrivate = 10;  // internal linkage

// 3. static function: only callable from this file
static void helper() { ... }  // internal linkage
```

#### register (hint to compiler, largely obsolete)
```c
register int i;   // hint: store in CPU register for speed
// Compiler may ignore this. Can't take address of register variable.
// Mostly ignored in modern compilers.
```

#### extern
```c
// Declare a variable defined in ANOTHER file:
// file1.c: int globalVar = 42;
// file2.c: extern int globalVar;  // declaration, not definition
//          printf("%d\n", globalVar);  // works!
```

### 🔧 static Variable Lifetime Demo
```c
#include <stdio.h>
int getCount() {
    static int count = 0;  // initialized to 0 ONCE at program start
    return ++count;
}
int main() {
    printf("%d\n", getCount());  // 1
    printf("%d\n", getCount());  // 2
    printf("%d\n", getCount());  // 3
    return 0;
}
```

### 🔧 Global vs Local vs Static Summary
| | Global | Local (auto) | Static Local | Static Global |
|---|---|---|---|---|
| **Scope** | File/all | Block | Block | File only |
| **Lifetime** | Program | Function call | Program | Program |
| **Init** | 0 (default) | Garbage | 0 (default) | 0 (default) |
| **Linkage** | External | None | None | Internal |

---

## 3. Inline Functions

### 🔧 C++ inline
```cpp
// Compiler replaces function call with function body (like macro but type-safe)
inline int square(int x) {
    return x * x;
}
// square(5) → compiler may replace with: 5 * 5 (no function call overhead)

// inline is a HINT, not a command — compiler may ignore it
// Good for: small, frequently called functions
// Bad for: large functions (code bloat)

// Difference from macro:
// inline: type checking, single evaluation of args, debuggable
// macro: no type check, multiple evaluation, text substitution
```

---

## 4. Recursion — Patterns & Tracing

### 🧠 How Recursion Works
```
Every recursive function needs:
1. BASE CASE: condition that stops recursion (returns directly)
2. RECURSIVE CASE: call itself with SMALLER input (moving toward base case)

Each call adds a STACK FRAME. Too deep → STACK OVERFLOW.
```

### 🔧 Pattern 1 — Linear Recursion (Single recursive call)

#### Factorial
```c
int factorial(int n) {
    if (n <= 1) return 1;       // base case
    return n * factorial(n-1);  // recursive case
}
// factorial(4):
// 4 * factorial(3)
//   3 * factorial(2)
//     2 * factorial(1)
//       1  ← base case
//     2 * 1 = 2
//   3 * 2 = 6
// 4 * 6 = 24
```

#### Power
```c
int power(int base, int exp) {
    if (exp == 0) return 1;
    return base * power(base, exp-1);
}
// power(2,3) = 2 * power(2,2) = 2 * 2 * power(2,1) = 2*2*2*power(2,0)
//            = 2*2*2*1 = 8
```

#### Sum of digits
```c
int digitSum(int n) {
    if (n < 10) return n;
    return (n % 10) + digitSum(n / 10);
}
// digitSum(123) = 3 + digitSum(12) = 3 + 2 + digitSum(1) = 3+2+1 = 6
```

### 🔧 Pattern 2 — Tail Recursion
```c
// Tail recursion: recursive call is the LAST thing (no pending operations)
// Can be optimized by compiler (tail call optimization) — doesn't grow stack

int factHelper(int n, int acc) {
    if (n <= 1) return acc;
    return factHelper(n-1, n * acc);  // tail recursive!
}
int factorial(int n) { return factHelper(n, 1); }

// Compare to non-tail: return n * factorial(n-1)
// Non-tail: must multiply AFTER recursive call returns → stack frame needed
// Tail: nothing to do after call → can reuse stack frame
```

### 🔧 Pattern 3 — Binary Recursion (Two recursive calls)

#### Fibonacci
```c
int fib(int n) {
    if (n <= 1) return n;   // fib(0)=0, fib(1)=1
    return fib(n-1) + fib(n-2);
}
// fib(5):
//          fib(5)
//         /      \
//      fib(4)   fib(3)
//      /    \   /   \
//   fib(3) fib(2) fib(2) fib(1)
//   ...
// TIME COMPLEXITY: O(2^n) — exponential! Very slow.
// fib(0..10): 0,1,1,2,3,5,8,13,21,34,55
```

#### Tower of Hanoi
```c
void hanoi(int n, char src, char dest, char aux) {
    if (n == 1) {
        printf("Move disk 1 from %c to %c\n", src, dest);
        return;
    }
    hanoi(n-1, src, aux, dest);   // move n-1 disks from src to aux
    printf("Move disk %d from %c to %c\n", n, src, dest);
    hanoi(n-1, aux, dest, src);   // move n-1 disks from aux to dest
}
// hanoi(3, 'A', 'C', 'B'):
// Moves = 2^n - 1 = 7 for n=3
```

### 🔧 Pattern 4 — Mutual Recursion
```c
int isEven(int n);
int isOdd(int n);

int isEven(int n) {
    if (n == 0) return 1;
    return isOdd(n - 1);
}
int isOdd(int n) {
    if (n == 0) return 0;
    return isEven(n - 1);
}
// isEven(4) → isOdd(3) → isEven(2) → isOdd(1) → isEven(0) → 1
```

### 🔧 Recursion Trace — Step by Step Method for Exams
```
For recursive output questions:
1. Draw the call tree
2. Identify if output is BEFORE or AFTER recursive call
   - BEFORE call: printed during descent (top-down)
   - AFTER call: printed during ascent (bottom-up)

void func(int n) {
    if (n == 0) return;
    printf("%d ", n);    // ← BEFORE recursive call
    func(n - 1);
    printf("%d ", n);    // ← AFTER recursive call
}
func(3):
  print 3           (before)
  call func(2):
    print 2         (before)
    call func(1):
      print 1       (before)
      call func(0): return
      print 1       (after)
    print 2         (after)
  print 3           (after)

Output: 3 2 1 1 2 3
```

---

## 5. C++ Function Features

### 🔧 Default Arguments
```cpp
void greet(string name, string greeting = "Hello") {
    cout << greeting << ", " << name << "!\n";
}
greet("Rahul");            // Hello, Rahul!
greet("Priya", "Hi");      // Hi, Priya!
greet("Bob", "Namaste");   // Namaste, Bob!

// RULES for default args:
// 1. Defaults must be at the END (right side) of parameter list
// 2. Once a param has default, all following params must too
void bad(int a = 1, int b) { }     // ERROR: non-default after default
void good(int a, int b = 1) { }    // OK
void good2(int a = 0, int b = 1) { } // OK
```

### 🔧 Function Overloading
```cpp
// Same name, different parameter types or count
int add(int a, int b) { return a + b; }
double add(double a, double b) { return a + b; }
int add(int a, int b, int c) { return a + b + c; }

add(1, 2);        // calls int version
add(1.0, 2.0);    // calls double version
add(1, 2, 3);     // calls 3-parameter version

// CANNOT overload by return type alone:
int func(int x) { return x; }
double func(int x) { return x; }  // ERROR: ambiguous

// How compiler resolves overloads (simplified):
// 1. Exact match
// 2. Promotion (char→int, float→double)
// 3. Standard conversion (int→double)
// 4. User-defined conversion
// 5. Ellipsis (...)
```

### 🔧 References in C++
```cpp
int x = 10;
int &ref = x;    // ref IS x (alias, same memory location)
ref = 20;        // x is now 20
cout << x;       // 20

// References MUST be initialized:
int &r;          // ERROR: reference must be initialized
int &r = x;      // OK

// References CANNOT be reseated (changed to refer to different variable):
int y = 30;
ref = y;         // does NOT make ref refer to y
                 // copies y's VALUE into x! x = 30 now

// Const reference (can bind to temporaries):
const int &cr = 42;    // OK: 42 is a temporary, bound to const ref
const int &cr2 = x + 1; // OK: expression result bound to const ref
// cr = 5;              // ERROR: const ref

// Reference as function parameter (pass by reference):
void swap(int &a, int &b) {
    int temp = a; a = b; b = temp;
}
int a = 1, b = 2;
swap(a, b);   // a=2, b=1 — no & at call site
```

### 🔧 Lambda Functions (C++11)
```cpp
// [capture](params) -> return_type { body }
auto square = [](int x) { return x * x; };
cout << square(5);  // 25

// Capture variables from surrounding scope:
int offset = 10;
auto addOffset = [offset](int x) { return x + offset; };  // capture by value
auto addRef = [&offset](int x) { return x + offset; };     // capture by reference

// [=] capture all by value
// [&] capture all by reference
auto f = [=](int x) { return x + offset; };  // captures everything by value

// Common use with STL algorithms:
vector<int> v = {5, 2, 8, 1, 9};
sort(v.begin(), v.end(), [](int a, int b) { return a > b; });
// sort descending: 9 8 5 2 1
```

---

## 6. Output Prediction Drill

### 🎯 Question 1 — Static Variable
```c
#include <stdio.h>
void func() {
    static int x = 0;
    x += 5;
    printf("%d\n", x);
}
int main() {
    func();
    func();
    func();
    return 0;
}
```
**Predict then check:**
```
static int x = 0: initialized ONCE. Persists across calls.

Call 1: x = 0+5 = 5  → prints 5
Call 2: x = 5+5 = 10 → prints 10
Call 3: x = 10+5 = 15 → prints 15

Output:
5
10
15
```

---

### 🎯 Question 2 — Recursion Trace
```c
#include <stdio.h>
void func(int n) {
    if (n == 0) return;
    printf("%d ", n);
    func(n - 1);
    printf("%d ", n);
}
int main() {
    func(4);
    printf("\n");
    return 0;
}
```
**Predict then check:**
```
func(4): print 4, call func(3), print 4
  func(3): print 3, call func(2), print 3
    func(2): print 2, call func(1), print 2
      func(1): print 1, call func(0), print 1
        func(0): return (base case)
      ← back: print 1
    ← back: print 2
  ← back: print 3
← back: print 4

Output: 4 3 2 1 1 2 3 4
```

---

### 🎯 Question 3 — Recursion with Return
```c
#include <stdio.h>
int func(int n) {
    if (n <= 0) return 0;
    return n + func(n - 2);
}
int main() {
    printf("%d\n", func(6));
    printf("%d\n", func(5));
    return 0;
}
```
**Predict then check:**
```
func(6) = 6 + func(4) = 6 + 4 + func(2) = 6+4+2+func(0)
        = 6+4+2+0 = 12

func(5) = 5 + func(3) = 5+3+func(1) = 5+3+1+func(-1)
        = 5+3+1+0 = 9
        (func(-1): n≤0 → return 0)

Output:
12
9
```

---

### 🎯 Question 4 — Default Arguments
```cpp
#include <iostream>
using namespace std;
int func(int a, int b = 10, int c = 20) {
    return a + b + c;
}
int main() {
    cout << func(1) << "\n";
    cout << func(1, 2) << "\n";
    cout << func(1, 2, 3) << "\n";
    return 0;
}
```
**Predict then check:**
```
func(1):      a=1, b=10, c=20 → 31
func(1,2):    a=1, b=2,  c=20 → 23
func(1,2,3):  a=1, b=2,  c=3  → 6

Output:
31
23
6
```

---

### 🎯 Question 5 — Overloading
```cpp
#include <iostream>
using namespace std;
void show(int x) { cout << "int: " << x << "\n"; }
void show(double x) { cout << "double: " << x << "\n"; }
void show(char x) { cout << "char: " << x << "\n"; }

int main() {
    show(5);
    show(5.0);
    show('A');
    show(5.5f);
    return 0;
}
```
**Predict then check:**
```
show(5):   5 is int literal → calls show(int)    → int: 5
show(5.0): 5.0 is double literal → show(double)   → double: 5
show('A'): 'A' is char → show(char)               → char: A
show(5.5f): 5.5f is float → no exact float overload
           → promoted to double → show(double)     → double: 5.5

Output:
int: 5
double: 5
char: A
double: 5.5
```

---

### 🎯 Question 6 — Reference Trap
```cpp
#include <iostream>
using namespace std;
void func(int &x, int y) {
    x = x + y;
    y = y + x;
    cout << x << " " << y << "\n";
}
int main() {
    int a = 5, b = 10;
    func(a, b);
    cout << a << " " << b << "\n";
    return 0;
}
```
**Predict then check:**
```
x is reference to a, y is copy of b.
x = x + y = 5 + 10 = 15  → a is now 15
y = y + x = 10 + 15 = 25  → local y only

Inside func: x=15, y=25 → prints 15 25

Back in main: a=15 (changed via ref), b=10 (unchanged)
  → prints 15 10

Output:
15 25
15 10
```

---

### 🎯 Question 7 — Fibonacci Trace
```c
#include <stdio.h>
int fib(int n) {
    printf("fib(%d) called\n", n);
    if (n <= 1) return n;
    return fib(n-1) + fib(n-2);
}
int main() {
    printf("Result: %d\n", fib(4));
    return 0;
}
```
**Predict then check:**
```
fib(4):
  fib(4) called
  → fib(3):
      fib(3) called
      → fib(2):
          fib(2) called
          → fib(1): fib(1) called → returns 1
          → fib(0): fib(0) called → returns 0
          returns 1
      → fib(1): fib(1) called → returns 1
      returns 2
  → fib(2):
      fib(2) called
      → fib(1): fib(1) called → returns 1
      → fib(0): fib(0) called → returns 0
      returns 1
  returns 3

Output:
fib(4) called
fib(3) called
fib(2) called
fib(1) called
fib(0) called
fib(1) called
fib(2) called
fib(1) called
fib(0) called
Result: 3
```

---

### 🎯 Question 8 — Scope & Shadowing
```c
#include <stdio.h>
int x = 1;
void func() {
    int x = 2;
    {
        int x = 3;
        printf("%d\n", x);  // (1)
    }
    printf("%d\n", x);      // (2)
}
int main() {
    printf("%d\n", x);      // (3)
    func();
    printf("%d\n", x);      // (4)
    return 0;
}
```
**Predict then check:**
```
(3) main sees global x = 1 → 1
func():
  (1) innermost block x = 3 → 3
  (2) func's local x = 2 → 2
(4) back in main, global x still = 1 → 1

Output:
1
3
2
1
```

---

## 7. MCQ Traps & Exam Q&A

### ⚠️ THE TRAPS

---

**TRAP 1: static local variable initialized only once**
```c
void f() { static int x = 5; x++; printf("%d\n", x); }
f(); f(); f();
// Output: 6 7 8  (NOT 6 6 6!)
// Initialization happens ONCE at program start, not every call
```

---

**TRAP 2: Default arguments — must be rightmost**
```cpp
void f(int a = 1, int b) { }  // ERROR
void f(int a, int b = 1) { }  // OK
```

---

**TRAP 3: Reference must be initialized**
```cpp
int &r;     // ERROR: must bind to something
int x = 5;
int &r = x; // OK
```

---

**TRAP 4: Overloading on return type alone is illegal**
```cpp
int f() { return 1; }
double f() { return 1.0; }  // ERROR: same parameter list
```

---

**TRAP 5: Recursive output order — before vs after call**
```c
// BEFORE call → top-down (counting down)
// AFTER call  → bottom-up (counting up, reversed)
// BOTH (before and after) → like a palindrome: n...1 1...n
```

---

**TRAP 6: Pass by value doesn't modify original**
```c
void f(int x) { x = 100; }
int a = 5;
f(a);
// a is STILL 5 — f only modified its local copy
```

---

**TRAP 7: Assigning to reference copies the value, doesn't reseat**
```cpp
int x = 5, y = 10;
int &r = x;
r = y;     // copies y's value into x! x = 10 now. r still references x.
           // Does NOT make r reference y!
```

---

### ⚡ Quick Fire Q&A

| Question | Answer |
|---|---|
| How many times is static local var initialized? | Once (at first call or program start) |
| C++ reference vs pointer key difference? | Ref: can't be NULL, can't be reseated. Ptr: can. |
| Can you overload on return type only? | No — must differ in parameter types/count |
| Default args must be at which end? | Right end (trailing) |
| Tail recursion benefit? | Compiler can optimize to not grow stack (TCO) |
| fib(10) with naive recursion is O(?)? | O(2^n) — exponential |
| factorial(0) = ? | 1 (base case) |
| factorial(1) = ? | 1 |
| Output before recursive call = ? | Top-down (descending order) |
| Output after recursive call = ? | Bottom-up (ascending order) |
| static global variable linkage? | Internal (visible only within file) |
| extern keyword purpose? | Declare variable defined in another file |
| register keyword does what? | Hints compiler to use CPU register (often ignored) |
| Can you take address of register variable? | No |
| Lambda syntax? | `[capture](params) { body }` |
| `[=]` capture means? | Capture all local variables by value |
| `[&]` capture means? | Capture all local variables by reference |
| hanoi(n) total moves = ? | 2^n - 1 |

---