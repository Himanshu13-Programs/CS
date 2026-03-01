# 📘 C/C++ Chapter 1 — C Basics: Macros, ASCII, Data Types & Operators
---

## 📌 Table of Contents
1. [C Program Structure](#1-c-program-structure)
2. [Data Types & Sizes](#2-data-types--sizes)
3. [ASCII Values — Complete Table](#3-ascii-values--complete-table)
4. [Macros & Preprocessor Directives](#4-macros--preprocessor-directives)
5. [Operators — Deep Dive](#5-operators--deep-dive)
6. [Type Conversion & Casting](#6-type-conversion--casting)
7. [Input / Output](#7-input--output)
8. [Control Flow](#8-control-flow)
9. [Arrays & Strings in C](#9-arrays--strings-in-c)
10. [Output Prediction Drill](#10-output-prediction-drill)
11. [MCQ Traps & Exam Q&A](#11-mcq-traps--exam-qa)

---

## 1. C Program Structure

```c
// Preprocessor directives (processed BEFORE compilation)
#include <stdio.h>      // standard I/O
#include <stdlib.h>     // malloc, free, exit
#include <string.h>     // strlen, strcpy, strcmp
#include <math.h>       // sqrt, pow, floor, ceil
#include <limits.h>     // INT_MAX, INT_MIN, CHAR_MAX...
#include <float.h>      // FLT_MAX, DBL_MAX...
#include <ctype.h>      // isalpha, isdigit, toupper, tolower

// Global variable (exists throughout program lifetime)
int globalVar = 10;

// Function declaration (prototype)
int add(int a, int b);

// Main function — entry point
int main() {
    // Local variable
    int x = 5;
    printf("%d\n", add(x, globalVar));  // prints 15
    return 0;  // 0 = success, non-zero = error
}

// Function definition
int add(int a, int b) {
    return a + b;
}
```

### 🔧 Compilation Stages
```
Source code (.c)
      ↓  Preprocessor  → expands macros, includes headers, removes comments
Preprocessed code (.i)
      ↓  Compiler      → converts to assembly
Assembly code (.s)
      ↓  Assembler     → converts to machine code
Object code (.o)
      ↓  Linker        → combines with libraries
Executable (.exe / a.out)
```

---

## 2. Data Types & Sizes

### 🔧 Primitive Types (typical 64-bit system)

| Type | Size | Range | Format |
|---|---|---|---|
| `char` | 1 byte | -128 to 127 (signed) | `%c`, `%d` |
| `unsigned char` | 1 byte | 0 to 255 | `%u` |
| `short` | 2 bytes | -32,768 to 32,767 | `%hd` |
| `unsigned short` | 2 bytes | 0 to 65,535 | `%hu` |
| `int` | 4 bytes | -2,147,483,648 to 2,147,483,647 | `%d` |
| `unsigned int` | 4 bytes | 0 to 4,294,967,295 | `%u` |
| `long` | 4 or 8 bytes | platform-dependent | `%ld` |
| `long long` | 8 bytes | -9.2×10¹⁸ to 9.2×10¹⁸ | `%lld` |
| `float` | 4 bytes | ~3.4×10³⁸ (6-7 decimal digits) | `%f` |
| `double` | 8 bytes | ~1.7×10³⁰⁸ (15-16 digits) | `%lf` |
| `long double` | 12 or 16 bytes | even larger | `%Lf` |

### 🔧 sizeof Operator
```c
#include <stdio.h>
int main() {
    printf("%lu\n", sizeof(char));        // 1
    printf("%lu\n", sizeof(int));         // 4
    printf("%lu\n", sizeof(long long));   // 8
    printf("%lu\n", sizeof(float));       // 4
    printf("%lu\n", sizeof(double));      // 8
    printf("%lu\n", sizeof(int*));        // 8 (pointer, 64-bit system)

    int arr[10];
    printf("%lu\n", sizeof(arr));         // 40 (10 * 4)
    printf("%lu\n", sizeof(arr)/sizeof(arr[0]));  // 10 (array length trick)
    return 0;
}
```

### 🔧 Integer Overflow — Classic Trap
```c
#include <stdio.h>
int main() {
    int x = 2147483647;   // INT_MAX
    printf("%d\n", x + 1);  // -2147483648 (wraps around to INT_MIN!)

    unsigned int y = 0;
    printf("%u\n", y - 1);  // 4294967295 (wraps to UINT_MAX!)

    char c = 127;
    printf("%d\n", c + 1);  // 128 — but stored as char: -128!
    // Actually: c+1 promotes to int = 128, printed as 128
    // BUT: char c2 = c + 1; printf("%d", c2); → -128 (overflow in char)
    return 0;
}
```

---

## 3. ASCII Values — Complete Table

### 🔧 Key ASCII Values to Memorize

```
'A' = 65    'a' = 97    '0' = 48
'B' = 66    'b' = 98    '1' = 49
...         ...         ...
'Z' = 90    'z' = 122   '9' = 57

Differences:
  'a' - 'A' = 32       (lowercase = uppercase + 32)
  '0' to '9': 48 to 57
  'A' to 'Z': 65 to 90
  'a' to 'z': 97 to 122

Special characters:
  '\0' = 0   (null terminator)
  '\n' = 10  (newline)
  '\t' = 9   (tab)
  ' '  = 32  (space)
  '!'  = 33
  '"'  = 34
  '#'  = 35
  '+'  = 43
  '-'  = 45
  '/'  = 47
  ':'  = 58
  ';'  = 59
  '='  = 61
  '?'  = 63
  '@'  = 64
  '['  = 91
  '\\' = 92
  ']'  = 93
  '_'  = 95
```

### 🔧 ASCII Operations — Exam Staples
```c
#include <stdio.h>
int main() {
    char c = 'A';
    printf("%c\n", c + 32);   // 'a' (uppercase to lowercase)
    printf("%c\n", c + 1);    // 'B'
    printf("%d\n", c);        // 65
    printf("%c\n", 65);       // A

    // Convert digit char to int:
    char digit = '7';
    int num = digit - '0';    // 7 (subtract ASCII of '0' = 48)
    printf("%d\n", num);      // 7

    // Convert int to digit char:
    int n = 5;
    char ch = n + '0';        // '5' (add 48)
    printf("%c\n", ch);       // 5

    // Check if uppercase:
    char x = 'G';
    if (x >= 'A' && x <= 'Z') printf("Uppercase\n");

    // Toggle case:
    printf("%c\n", 'A' ^ 32); // 'a' (XOR with 32 flips bit 5)
    printf("%c\n", 'a' ^ 32); // 'A'

    return 0;
}
```

### 🔧 ctype.h Functions
```c
#include <ctype.h>
isalpha(c)   // is letter (a-z or A-Z)?
isdigit(c)   // is digit (0-9)?
isalnum(c)   // is letter or digit?
isspace(c)   // is whitespace (' ', '\t', '\n')?
isupper(c)   // is uppercase?
islower(c)   // is lowercase?
toupper(c)   // convert to uppercase
tolower(c)   // convert to lowercase

// All return non-zero (true) or 0 (false)
// toupper/tolower return the converted char value
```

---

## 4. Macros & Preprocessor Directives

### 🧠 What are Macros?
```
Macros are processed by the PREPROCESSOR before compilation.
Text substitution — no type checking, no function overhead.
```

### 🔧 #define — Object-like Macros
```c
#define PI 3.14159
#define MAX_SIZE 100
#define NEWLINE '\n'

// Usage:
double area = PI * r * r;   // preprocessor replaces PI with 3.14159
int arr[MAX_SIZE];           // becomes int arr[100];

// NO semicolon at end of #define (it becomes part of the substitution)
#define PI 3.14;   // WRONG — PI becomes "3.14;" and area = 3.14; * r * r; = ERROR
```

### 🔧 #define — Function-like Macros
```c
#define SQUARE(x)    ((x) * (x))
#define MAX(a, b)    ((a) > (b) ? (a) : (b))
#define MIN(a, b)    ((a) < (b) ? (a) : (b))
#define ABS(x)       ((x) < 0 ? -(x) : (x))
#define SWAP(a, b)   { int t = a; a = b; b = t; }

// WHY double parentheses?
// Without: #define SQUARE(x) x * x
// SQUARE(2+3) → 2+3 * 2+3 = 2+6+3 = 11 (WRONG! should be 25)
// With:    #define SQUARE(x) ((x)*(x))
// SQUARE(2+3) → ((2+3)*(2+3)) = 5*5 = 25 ✓

// Macro side effect trap:
int a = 5;
int b = SQUARE(a++);
// Expands to: ((a++) * (a++)) — a incremented TWICE!
// Undefined behavior! Result unpredictable.
// This NEVER happens with a real function.
```

### 🔧 #define — Predefined Macros
```c
__FILE__     // current filename (string)
__LINE__     // current line number (int)
__DATE__     // compilation date (string): "Jan  1 2026"
__TIME__     // compilation time (string): "12:30:45"
__func__     // current function name (C99)

printf("Error in %s at line %d\n", __FILE__, __LINE__);
```

### 🔧 #ifdef, #ifndef, #endif — Conditional Compilation
```c
// Include guard (prevent double inclusion):
#ifndef MYHEADER_H
#define MYHEADER_H
    // header content here
#endif

// Conditional code:
#define DEBUG 1

#ifdef DEBUG
    printf("Debug: x = %d\n", x);  // only compiled if DEBUG defined
#endif

#ifndef NDEBUG
    // runs if NDEBUG is NOT defined (debug mode)
#endif

// Compile with: gcc -DDEBUG file.c  (defines DEBUG from command line)
```

### 🔧 #undef
```c
#define LIMIT 100
// ... use LIMIT ...
#undef LIMIT     // undefine it
// #define LIMIT 200  // can redefine now
```

### 🔧 ## (Token Pasting) and # (Stringification)
```c
// # converts argument to string literal:
#define STRINGIFY(x) #x
printf("%s\n", STRINGIFY(hello));  // prints: hello
printf("%s\n", STRINGIFY(3+4));    // prints: 3+4

// ## concatenates tokens:
#define CONCAT(a, b) a##b
int xy = 10;
printf("%d\n", CONCAT(x, y));  // becomes: printf("%d\n", xy); → 10

#define VAR(n) var##n
int var1 = 100, var2 = 200;
printf("%d %d\n", VAR(1), VAR(2));  // 100 200
```

### 🔧 Macros vs Functions
| Feature | Macro | Function |
|---|---|---|
| **Type checking** | ❌ None | ✅ Yes |
| **Overhead** | None (inline substitution) | Stack frame, parameter passing |
| **Side effects** | Dangerous (args evaluated multiple times) | Safe (args evaluated once) |
| **Debugging** | Hard (no function name) | Easy |
| **Recursion** | ❌ Not possible | ✅ Yes |
| **Return value** | Expression result | Explicit return |

---

## 5. Operators — Deep Dive

### 🔧 Arithmetic Operators
```c
int a = 17, b = 5;
printf("%d\n", a + b);   // 22
printf("%d\n", a - b);   // 12
printf("%d\n", a * b);   // 85
printf("%d\n", a / b);   // 3   (integer division, truncates toward zero)
printf("%d\n", a % b);   // 2   (modulo)

// Integer division truncates toward zero (NOT floor):
printf("%d\n", -7 / 2);  // -3  (NOT -4!)
printf("%d\n", 7 / -2);  // -3
printf("%d\n", -7 % 2);  // -1  (sign of result = sign of dividend)
printf("%d\n", 7 % -2);  // 1
```

### 🔧 Increment / Decrement — The Most Tested Operators
```c
int a = 5;

// Pre-increment: increment FIRST, then use
printf("%d\n", ++a);  // 6 (a becomes 6, then printed)
printf("%d\n", a);    // 6

int b = 5;
// Post-increment: use FIRST, then increment
printf("%d\n", b++);  // 5 (5 printed, then b becomes 6)
printf("%d\n", b);    // 6

// In expressions:
int x = 5, y;
y = x++ + x++;
// Undefined behavior in C! Don't do this.
// But common exam question: y = x++ + ++x with ONE variable
// Post-increment returns OLD value, pre-increment returns NEW value

int c = 5;
int d = c++ + ++c;
// Step 1: c++ evaluates to 5, c becomes 6
// Step 2: ++c increments c to 7, evaluates to 7
// d = 5 + 7 = 12, c = 7
// NOTE: This is technically undefined behavior in C, but
// many exams test it with expected answers.
```

### 🔧 Bitwise Operators — Must Know for CoreTex
```c
int a = 12;   // binary: 1100
int b = 10;   // binary: 1010

printf("%d\n", a & b);   // AND:  1000 = 8
printf("%d\n", a | b);   // OR:   1110 = 14
printf("%d\n", a ^ b);   // XOR:  0110 = 6
printf("%d\n", ~a);      // NOT:  ...10011 = -13 (two's complement)
printf("%d\n", a << 1);  // LEFT SHIFT:  11000 = 24 (multiply by 2)
printf("%d\n", a >> 1);  // RIGHT SHIFT: 0110 = 6  (divide by 2)

// Useful bit tricks:
// Check if nth bit is set:
int n = 3;
if (a & (1 << n)) printf("bit %d is set\n", n);

// Set nth bit:
a = a | (1 << n);

// Clear nth bit:
a = a & ~(1 << n);

// Toggle nth bit:
a = a ^ (1 << n);

// Check if number is power of 2:
if (a && !(a & (a-1))) printf("power of 2\n");

// Count set bits (Brian Kernighan):
int count = 0;
while (a) { a &= (a-1); count++; }
```

### 🔧 Relational & Logical Operators
```c
// Relational: return 1 (true) or 0 (false)
5 > 3    // 1
5 < 3    // 0
5 == 5   // 1
5 != 3   // 1
5 >= 5   // 1
5 <= 4   // 0

// Logical:
int x = 1, y = 0;
x && y   // 0 (AND: both must be true)
x || y   // 1 (OR: at least one true)
!x       // 0 (NOT)

// SHORT-CIRCUIT EVALUATION (critical exam concept):
// &&: if left side is false → right side NOT evaluated
// ||: if left side is true  → right side NOT evaluated

int a = 0;
if (a != 0 && (10/a > 2))  // safe! 10/a never evaluated (a==0)
    printf("ok\n");

int b = 0;
if (b++ || b++)  // b=0: first b++ evaluates to 0 (false), b becomes 1
                 //       second b++ evaluates to 1 (true), b becomes 2
                 // condition: 0 || 1 = true
// After: b = 2
```

### 🔧 Assignment Operators
```c
int x = 10;
x += 5;   // x = x + 5 = 15
x -= 3;   // x = x - 3 = 12
x *= 2;   // x = x * 2 = 24
x /= 4;   // x = x / 4 = 6
x %= 4;   // x = x % 4 = 2
x &= 3;   // x = x & 3
x |= 3;   // x = x | 3
x ^= 3;   // x = x ^ 3
x <<= 1;  // x = x << 1
x >>= 1;  // x = x >> 1
```

### 🔧 Ternary Operator
```c
int x = 10, y = 20;
int max = (x > y) ? x : y;   // 20

// Nested ternary:
int a = 5, b = 10, c = 7;
int mid = (a > b) ? ((a < c) ? a : c) : ((b < c) ? b : c);
// Readable version: find middle value of 3 numbers

// printf with ternary:
printf("%s\n", (x % 2 == 0) ? "even" : "odd");
```

### 🔧 Comma Operator
```c
// Evaluates both expressions, returns the value of the RIGHTMOST
int x = (3, 5, 7);   // x = 7
printf("%d\n", x);   // 7

// Used in for loops:
for (int i = 0, j = 10; i < j; i++, j--) {
    printf("%d %d\n", i, j);
}
```

### 🔧 Operator Precedence (High to Low)
```
()  []  ->  .               (postfix, left to right)
++  --  +  -  !  ~  *  &  sizeof  (prefix, right to left)
*  /  %                     (multiplicative)
+  -                        (additive)
<<  >>                      (shift)
<  <=  >  >=               (relational)
==  !=                      (equality)
&                           (bitwise AND)
^                           (bitwise XOR)
|                           (bitwise OR)
&&                          (logical AND)
||                          (logical OR)
?:                          (ternary)
=  +=  -=  *=  /=  etc.    (assignment, right to left)
,                           (comma)

MEMORIZE: "Please Excuse My Dear Aunt Sally" won't work here.
Just remember: arithmetic > relational > bitwise > logical > ternary > assignment
```

---

## 6. Type Conversion & Casting

### 🔧 Implicit Conversion (Automatic)
```c
// Promotion hierarchy (smaller → larger automatically):
// char → short → int → long → long long → float → double → long double

int i = 10;
double d = i;          // int automatically promoted to double: d = 10.0

double x = 3.7;
int y = x;             // double truncated to int: y = 3 (NOT rounded)

char c = 'A';
int n = c;             // char promoted to int: n = 65

// In expressions — both operands promoted to larger type:
int a = 5;
double b = 2.0;
double result = a / b;  // a promoted to double: 5.0/2.0 = 2.5
int bad = a / 2;        // INTEGER division: 5/2 = 2 (NOT 2.5!)
```

### 🔧 Explicit Casting
```c
int a = 5, b = 2;
double result = (double)a / b;      // 2.5 (cast a to double first)
double bad = (double)(a / b);       // 2.0 (integer div first, THEN cast!)

// TRAP: Order of cast and division matters!
printf("%.1f\n", (double)(5/2));    // 2.0 (int div first)
printf("%.1f\n", (double)5/2);      // 2.5 (cast first, then div)
printf("%.1f\n", 5/(double)2);      // 2.5 (cast 2, then div)

// char ↔ int:
char c = (char)65;    // 'A'
int n = (int)'Z';     // 90

// Truncation vs rounding:
int x = (int)3.9;     // 3 (truncates, does NOT round)
int y = (int)-3.9;    // -3 (truncates toward zero)
int z = (int)(-3.9 - 0.5);  // manual floor for negative: -4
```

### 🔧 Integer Promotion Rules
```c
char a = 200;        // stored as -56 (signed char: 200 - 256 = -56)
unsigned char b = 200; // stored as 200

printf("%d\n", a);   // -56
printf("%d\n", b);   // 200

// In arithmetic: char/short promoted to int
char x = 100, y = 100;
char z = x + y;    // x+y = 200, cast back to char = -56 (overflow!)
int w = x + y;     // 200 (safe in int)
```

---

## 7. Input / Output

### 🔧 printf Format Specifiers
```c
%d    // int
%u    // unsigned int
%f    // float/double (default 6 decimal places)
%e    // scientific notation: 3.14e+00
%g    // shorter of %f and %e
%c    // char
%s    // string (char*)
%p    // pointer address
%x    // hexadecimal (lowercase)
%X    // hexadecimal (uppercase)
%o    // octal
%lld  // long long
%lf   // double in scanf (same as %f in printf)
%lu   // unsigned long
%%    // literal %

// Width and precision:
printf("%10d\n", 42);      //         42 (right-aligned, width 10)
printf("%-10d|\n", 42);    // 42        | (left-aligned)
printf("%010d\n", 42);     // 0000000042 (zero-padded)
printf("%.2f\n", 3.14159); // 3.14 (2 decimal places)
printf("%8.3f\n", 3.14);   //    3.140 (width 8, 3 decimals)
printf("%.5s\n", "Hello World"); // Hello (first 5 chars)
```

### 🔧 scanf Traps
```c
int n;
scanf("%d", &n);    // & required! (pass address, not value)

char name[50];
scanf("%s", name);  // NO & for arrays (array = pointer)
                    // reads until whitespace — no spaces!

scanf(" %c", &c);   // SPACE before %c skips whitespace/newline
                    // Without space: reads '\n' left in buffer!

// Reading a full line:
char line[100];
fgets(line, sizeof(line), stdin);  // safer than gets()
// gets() is BANNED — no bounds checking, buffer overflow!

// scanf return value:
int ret = scanf("%d %d", &a, &b);
// returns number of items successfully read (0, 1, or 2 here)
// returns EOF on end of file
```

### 🔧 printf Return Value
```c
// printf returns number of characters printed (rarely used)
int n = printf("Hello\n");  // n = 6 (5 chars + newline)
```

---

## 8. Control Flow

### 🔧 if-else Chains
```c
// Dangling else problem:
int x = 5, y = 10;
if (x > 3)
    if (y > 8)
        printf("A\n");
else    // this else belongs to INNER if (y > 8), NOT outer if (x > 3)!
    printf("B\n");

// Output: A (since x>3 and y>8, inner if true, else skipped)
// If y = 5: output would be B (x>3 true, y>8 false → else executes)

// Fix with braces:
if (x > 3) {
    if (y > 8)
        printf("A\n");
} else {
    printf("B\n");
}
```

### 🔧 switch Statement
```c
int x = 2;
switch (x) {
    case 1:
        printf("one\n");
        break;
    case 2:
        printf("two\n");   // prints this
        // NO BREAK → FALL THROUGH!
    case 3:
        printf("three\n"); // ALSO prints this (fall-through!)
        break;
    default:
        printf("other\n");
}
// Output: two\nthree\n  (fall-through!)

// switch only works with: int, char, short, long, enum
// Does NOT work with: float, double, string
```

### 🔧 Loops — break and continue
```c
// break: exit loop immediately
// continue: skip rest of current iteration, go to next

for (int i = 0; i < 5; i++) {
    if (i == 3) break;
    printf("%d ", i);
}
// Output: 0 1 2

for (int i = 0; i < 5; i++) {
    if (i == 3) continue;
    printf("%d ", i);
}
// Output: 0 1 2 4

// Nested loop — break only exits INNERMOST loop:
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (j == 1) break;   // only exits inner loop
        printf("%d%d ", i, j);
    }
}
// Output: 00 10 20
```

### 🔧 do-while vs while
```c
// do-while: always executes body AT LEAST ONCE
int x = 10;
do {
    printf("%d\n", x);  // prints 10 even though condition is false
    x++;
} while (x < 5);
// Output: 10

// while: may never execute
while (x < 5) {         // x=11, condition false, never enters
    printf("%d\n", x);
}
// No output
```

### 🔧 goto (Rare but Tested)
```c
int i = 0;
loop:
    if (i < 3) {
        printf("%d\n", i);
        i++;
        goto loop;
    }
// Output: 0\n1\n2
// goto jumps to label. Avoid in practice but know for MCQs.
```

---

## 9. Arrays & Strings in C

### 🔧 Arrays
```c
// Declaration and initialization:
int arr[5] = {1, 2, 3, 4, 5};
int arr2[5] = {1, 2};        // {1, 2, 0, 0, 0} — rest initialized to 0
int arr3[] = {1, 2, 3};      // size inferred: 3 elements
int arr4[5] = {0};            // all zeros
int arr5[5];                  // UNINITIALIZED — garbage values!

// Array name = pointer to first element:
printf("%p\n", arr);          // address of arr[0]
printf("%p\n", &arr[0]);      // same address

// Array index out of bounds — NO ERROR in C (undefined behavior):
int a[3] = {1, 2, 3};
printf("%d\n", a[5]);         // undefined behavior! may print garbage

// 2D Arrays:
int matrix[3][4];                          // 3 rows, 4 columns
int m[2][3] = {{1,2,3},{4,5,6}};
printf("%d\n", m[1][2]);                   // 6

// Memory layout: row-major order
// m[0][0] m[0][1] m[0][2] m[1][0] m[1][1] m[1][2]
//    1       2       3       4       5       6
```

### 🔧 Strings in C
```c
// String = char array ending with '\0' (null terminator)
char str[] = "Hello";     // {'H','e','l','l','o','\0'} — 6 chars, NOT 5!
char str2[6] = "Hello";   // same
char str3[] = {'H','e','l','l','o','\0'};  // explicit

printf("%zu\n", sizeof(str));    // 6 (includes '\0')
printf("%zu\n", strlen(str));    // 5 (does NOT count '\0')

// String functions (string.h):
strlen(s)              // length (not counting '\0')
strcpy(dest, src)      // copy src into dest (dest must be large enough)
strncpy(dest, src, n)  // copy at most n chars
strcat(dest, src)      // append src to dest
strncat(dest, src, n)  // append at most n chars
strcmp(s1, s2)         // 0 if equal, <0 if s1<s2, >0 if s1>s2
strncmp(s1, s2, n)     // compare first n chars
strchr(s, c)           // find first occurrence of char c in s (returns pointer or NULL)
strstr(s1, s2)         // find first occurrence of s2 in s1

// Common trap:
char s1[] = "abc";
char s2[] = "abc";
if (s1 == s2)          // WRONG! Compares ADDRESSES, not contents!
    printf("equal\n");
if (strcmp(s1, s2) == 0)  // CORRECT
    printf("equal\n");
```

---

## 10. Output Prediction Drill

### 🎯 Question 1
```c
#include <stdio.h>
#define DOUBLE(x) x + x

int main() {
    int a = 5;
    printf("%d\n", 10 * DOUBLE(a));
    return 0;
}
```
**Predict output before reading answer:**
```
Macro expands: 10 * DOUBLE(a) → 10 * a + a → 10 * 5 + 5 = 50 + 5 = 55

Output: 55

Fix: #define DOUBLE(x) ((x) + (x)) → 10 * ((5)+(5)) = 10*10 = 100
```

---

### 🎯 Question 2
```c
#include <stdio.h>
int main() {
    char c = 'A';
    while (c <= 'E') {
        printf("%c ", c++);
    }
    printf("\n%d\n", c);
    return 0;
}
```
**Predict output:**
```
Loop: c starts at 'A'(65)
  c='A'(65) ≤ 'E'(69): print 'A', c++ makes c='B'
  c='B'(66) ≤ 'E'(69): print 'B', c++ → 'C'
  c='C'(67) ≤ 'E'(69): print 'C', c++ → 'D'
  c='D'(68) ≤ 'E'(69): print 'D', c++ → 'E'
  c='E'(69) ≤ 'E'(69): print 'E', c++ → 'F'(70)
  c='F'(70) > 'E'(69): exit loop

Output:
A B C D E
70
```

---

### 🎯 Question 3
```c
#include <stdio.h>
int main() {
    int x = 5;
    printf("%d %d %d\n", x++, ++x, x++);
    return 0;
}
```
**Predict output:**
```
IMPORTANT: In function call arguments, order of evaluation is UNSPECIFIED in C.
Different compilers may give different results.
This is UNDEFINED BEHAVIOR — but many exams expect a specific answer.

Common exam expected behavior (right-to-left evaluation on many compilers):
  x++ (rightmost): returns 5, x becomes 6
  ++x (middle):    x becomes 7, returns 7
  x++ (leftmost):  returns 7, x becomes 8

Output (compiler-dependent): 7 7 5

But on other compilers with left-to-right: different answer.
KEY EXAM ANSWER: "Undefined behavior" or compiler-specific.
If exam gives specific answer: likely 7 7 5 or 5 7 6.
```

---

### 🎯 Question 4
```c
#include <stdio.h>
int main() {
    int a = 10, b = 20;
    printf("%d\n", (a, b));      // comma operator
    printf("%d\n", a > b ? a : b);  // ternary
    a = b = 5;
    printf("%d %d\n", a, b);
    return 0;
}
```
**Predict output:**
```
(a, b): comma operator — evaluates a (10), then b (20), returns 20.
  → prints 20

a > b: 10 > 20 is false → returns b = 20.
  → prints 20

a = b = 5: right-to-left assignment. b=5, then a=b=5. Both 5.
  → prints 5 5

Output:
20
20
5 5
```

---

### 🎯 Question 5
```c
#include <stdio.h>
#define MAX(a,b) ((a)>(b)?(a):(b))

int main() {
    int i = 10, j = 12;
    int k = MAX(i++, j++);
    printf("%d %d %d\n", i, j, k);
    return 0;
}
```
**Predict output:**
```
Macro expands: ((i++)>(j++)?(i++):(j++))

Step 1: Compare (i++) > (j++): i=10, j=12 evaluated.
        10 > 12 is false. i becomes 11, j becomes 13.
Step 2: Condition false → evaluate (j++): returns 13, j becomes 14.

i=11, j=14, k=13

Output: 11 14 13
```

---

### 🎯 Question 6
```c
#include <stdio.h>
int main() {
    int x = 0;
    if (x = 5)           // assignment, NOT comparison!
        printf("true\n");
    else
        printf("false\n");

    if (x == 5)
        printf("equal\n");

    return 0;
}
```
**Predict output:**
```
if (x = 5): assigns 5 to x, evaluates to 5 (non-zero = true)
  → prints "true"

x is now 5. x == 5 is true.
  → prints "equal"

Output:
true
equal
```

---

### 🎯 Question 7 — ASCII Trick
```c
#include <stdio.h>
int main() {
    char s[] = "Hello";
    for (int i = 0; s[i] != '\0'; i++) {
        if (s[i] >= 'a' && s[i] <= 'z')
            s[i] -= 32;
    }
    printf("%s\n", s);
    return 0;
}
```
**Predict output:**
```
'H'=72: not lowercase, unchanged → H
'e'=101: lowercase → 101-32=69='E'
'l'=108: lowercase → 108-32=76='L'
'l'=108: lowercase → 76='L'
'o'=111: lowercase → 111-32=79='O'

Output: HELLO
```

---

### 🎯 Question 8 — switch Fall-Through
```c
#include <stdio.h>
int main() {
    int x = 1;
    switch (x) {
        case 1: printf("one ");
        case 2: printf("two ");
        case 3: printf("three ");
                break;
        case 4: printf("four ");
        default: printf("default ");
    }
    printf("\n");
    return 0;
}
```
**Predict output:**
```
x=1 → matches case 1.
No break → FALLS THROUGH to case 2.
No break → FALLS THROUGH to case 3.
break → exits switch.

Output: one two three
```

---

### 🎯 Question 9 — Bitwise
```c
#include <stdio.h>
int main() {
    int a = 5;      // 0101
    int b = 3;      // 0011

    printf("%d\n", a & b);   // AND
    printf("%d\n", a | b);   // OR
    printf("%d\n", a ^ b);   // XOR
    printf("%d\n", ~a);      // NOT
    printf("%d\n", a << 1);  // LEFT SHIFT
    printf("%d\n", a >> 1);  // RIGHT SHIFT
    return 0;
}
```
**Predict output:**
```
a & b = 0101 & 0011 = 0001 = 1
a | b = 0101 | 0011 = 0111 = 7
a ^ b = 0101 ^ 0011 = 0110 = 6
~a    = ~0101 = ...11111010 = -6 (two's complement: -(5+1) = -6)
a<<1  = 1010 = 10
a>>1  = 0010 = 2

Output:
1
7
6
-6
10
2
```

---

### 🎯 Question 10 — Type Conversion
```c
#include <stdio.h>
int main() {
    printf("%d\n", 5/2);         // integer division
    printf("%.1f\n", 5/2);       // still integer division before cast!
    printf("%.1f\n", 5.0/2);     // float division
    printf("%.1f\n", (float)5/2);// explicit cast
    printf("%d\n", (int)3.9);    // truncation
    printf("%d\n", (int)-3.9);   // truncation toward zero
    return 0;
}
```
**Predict output:**
```
5/2 → int division → 2. printf %d → 2
5/2 → 2 (int), then 2 passed as double to %f → 2.0. But wait...
  Actually: 5/2 = 2 (int), then %f reads it as double... 
  This is undefined behavior (passing int where double expected without cast)
  Most compilers: 2.0 but technically UB. Exam answer: 2.0

5.0/2 → double division → 2.5 → 2.5
(float)5/2 → 5.0f/2 → 2.5 → 2.5
(int)3.9 → 3 (truncate)
(int)-3.9 → -3 (truncate toward zero)

Output:
2
2.0
2.5
2.5
3
-3
```

---

## 11. MCQ Traps & Exam Q&A

### ⚠️ THE TRAPS

---

**TRAP 1: `=` vs `==` in conditions**
```c
if (x = 0)  // assigns 0 to x, evaluates to 0 (FALSE) — always false!
if (x == 0) // comparison — tests if x equals 0
```

---

**TRAP 2: Macro without parentheses**
```c
#define SQ(x) x*x
SQ(2+3) → 2+3*2+3 = 11 (WRONG, should be 25)
Always: #define SQ(x) ((x)*(x))
```

---

**TRAP 3: sizeof on array vs pointer**
```c
int arr[5] = {1,2,3,4,5};
int *p = arr;
sizeof(arr) = 20  (5 * 4 bytes)
sizeof(p)   = 8   (pointer size, 64-bit)
// sizeof on array name = total bytes
// sizeof on pointer = pointer size (always 4 or 8)
```

---

**TRAP 4: Integer division truncation direction**
```c
-7 / 2 = -3   (NOT -4! Truncates toward zero)
7 / -2 = -3   (same)
-7 % 2 = -1   (sign of result = sign of DIVIDEND)
```

---

**TRAP 5: String comparison with ==**
```c
char s1[] = "hello";
char s2[] = "hello";
s1 == s2   // FALSE (compares addresses, not content!)
strcmp(s1, s2) == 0  // TRUE (correct way)
```

---

**TRAP 6: sizeof("hello")**
```c
sizeof("hello") = 6  // includes '\0' null terminator
strlen("hello") = 5  // does NOT include '\0'
```

---

**TRAP 7: Post vs pre-increment return value**
```c
int x = 5;
int y = x++;  // y = 5 (old value), then x = 6
int z = ++x;  // x = 7 first, then z = 7
```

---

**TRAP 8: `~` (bitwise NOT) on positive int**
```c
~5 = -6   (NOT -5!)
~n = -(n+1)  always
```

---

### ⚡ Quick Fire Q&A

| Question | Answer |
|---|---|
| ASCII of 'A'? | 65 |
| ASCII of 'a'? | 97 |
| ASCII of '0'? | 48 |
| 'a' - 'A' = ? | 32 |
| sizeof(int) on 64-bit? | 4 bytes |
| sizeof(pointer) on 64-bit? | 8 bytes |
| sizeof("hi") = ? | 3 (includes '\0') |
| strlen("hi") = ? | 2 |
| -7 / 2 in C = ? | -3 (truncate toward zero) |
| ~0 in C = ? | -1 |
| ~5 in C = ? | -6 |
| 5 & 3 = ? | 1 |
| 5 \| 3 = ? | 7 |
| 5 ^ 3 = ? | 6 |
| 5 << 1 = ? | 10 |
| 5 >> 1 = ? | 2 |
| (int)3.9 = ? | 3 |
| (int)-3.9 = ? | -3 |
| Macro #define X 5; — what's wrong? | Semicolon becomes part of substitution |
| Which is faster: macro or function? | Macro (no function call overhead) |
| Can switch work with float? | No — only integer types and char |
| Which else does dangling else attach to? | The nearest (inner) if |

---
