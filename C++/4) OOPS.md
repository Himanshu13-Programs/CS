# 📘 C/C++ Chapter 4 — OOP, Templates, Exceptions & File I/O 

---

## 1. Classes & Objects

### 🔧 Class Basics
```cpp
class Rectangle {
private:             // accessible only within class (default for class)
    double width;
    double height;

public:              // accessible from anywhere
    // Constructor
    Rectangle(double w, double h) : width(w), height(h) {
        cout << "Rectangle created\n";
    }

    // Member functions
    double area() const {    // const: doesn't modify object
        return width * height;
    }

    double perimeter() const {
        return 2 * (width + height);
    }

    // Getter/Setter
    double getWidth() const { return width; }
    void setWidth(double w) {
        if (w > 0) width = w;
    }

protected:           // accessible within class and derived classes
    void helper() { }
};

// Create objects:
Rectangle r1(3.0, 4.0);         // stack object
Rectangle *r2 = new Rectangle(5.0, 6.0);  // heap object

cout << r1.area();               // 12
cout << r2->area();              // 30 (use -> for pointers)
delete r2;
```

### 🔧 struct vs class
```cpp
struct MyStruct {
    int x;      // PUBLIC by default
    int y;
};

class MyClass {
    int x;      // PRIVATE by default
    int y;
};

// Only difference: default access specifier
// struct: public by default
// class:  private by default
// In C++, structs can have constructors, methods, inheritance — same as class!
```

### 🔧 this Pointer
```cpp
class Counter {
    int count;
public:
    Counter(int count) {
        this->count = count;  // this->count = member, count = parameter
    }

    Counter& increment() {
        count++;
        return *this;   // return reference to current object (for chaining)
    }
};

Counter c(0);
c.increment().increment().increment();  // method chaining!
// count = 3
```

### 🔧 Static Members
```cpp
class MyClass {
    static int objectCount;   // shared by ALL instances (one copy)
    int id;
public:
    MyClass() {
        objectCount++;
        id = objectCount;
    }
    static int getCount() { return objectCount; }  // no 'this' pointer
};
int MyClass::objectCount = 0;   // must define outside class!

MyClass a, b, c;
cout << MyClass::getCount();  // 3 (use class name, not object)
cout << a.getCount();         // 3 (also valid but bad style)
```

---

## 2. Constructors & Destructors

### 🔧 Types of Constructors

#### Default Constructor
```cpp
class Foo {
public:
    int x;
    Foo() {             // no parameters
        x = 0;
        cout << "Default constructor\n";
    }
};
Foo f1;               // calls default constructor
Foo f2 = Foo();       // same
```

#### Parameterized Constructor
```cpp
class Point {
    int x, y;
public:
    Point(int x, int y) {
        this->x = x; this->y = y;
        cout << "Point(" << x << "," << y << ") created\n";
    }
};
Point p(3, 4);
```

#### Copy Constructor
```cpp
class Point {
    int x, y;
public:
    Point(int x, int y) : x(x), y(y) {}

    // Copy constructor: called when copying an object
    Point(const Point &other) {
        x = other.x;
        y = other.y;
        cout << "Copy constructor called\n";
    }
};

Point p1(1, 2);
Point p2 = p1;      // copy constructor called!
Point p3(p1);       // copy constructor called!
Point p4;
p4 = p1;            // assignment operator (NOT copy constructor)
```

#### Member Initializer List
```cpp
class Rectangle {
    const double width;    // const member MUST use initializer list
    double &height;        // reference MUST use initializer list
    int x;
public:
    // Initializer list: more efficient (direct initialization)
    Rectangle(double w, double &h, int x)
        : width(w), height(h), x(x) {
        // body runs after all members initialized
    }
};
// Order of initialization = ORDER OF DECLARATION in class (NOT list order!)
```

### 🔧 Destructor
```cpp
class Foo {
public:
    ~Foo() {
        cout << "Destructor called\n";
    }
};

// Destructor called automatically when:
// - Stack object goes out of scope
// - delete is called on heap object
// - Program ends (for global/static objects)

// RULE: Constructor and Destructor ORDER
// Objects constructed FIRST → destroyed LAST (LIFO — like a stack)
{
    Foo a;   // constructed 1st
    Foo b;   // constructed 2nd
}            // b destroyed 1st, a destroyed 2nd
```

### 🔧 Constructor/Destructor Call Order — THE EXAM PATTERN
```cpp
class Base {
public:
    Base()  { cout << "Base constructor\n"; }
    ~Base() { cout << "Base destructor\n"; }
};

class Derived : public Base {
public:
    Derived()  { cout << "Derived constructor\n"; }
    ~Derived() { cout << "Derived destructor\n"; }
};

int main() {
    Derived d;
    return 0;
}
```
**Output:**
```
Base constructor       ← BASE always constructed FIRST
Derived constructor    ← DERIVED constructed AFTER base
Derived destructor     ← DERIVED destroyed FIRST
Base destructor        ← BASE destroyed LAST

RULE: Constructor order = Base → Derived → Derived's members
      Destructor order = REVERSE (members → Derived → Base)
```

---

## 3. Inheritance

### 🔧 Types of Inheritance
```cpp
class A { };

class B : public A { };     // public inheritance
class B : protected A { };  // protected inheritance
class B : private A { };    // private inheritance (default for class)

// Access changes in inheritance:
//              public     protected    private
// public:      public     protected    private
// protected:   protected  protected    private
// private:     private    private      private
```

### 🔧 Access Specifiers Through Inheritance
```cpp
class Base {
public:    int pub = 1;
protected: int prot = 2;
private:   int priv = 3;
};

class PublicDerived : public Base {
    void func() {
        cout << pub;    // OK: public → public
        cout << prot;   // OK: protected → protected
        // cout << priv;   // ERROR: private never inherited
    }
};

class ProtectedDerived : protected Base {
    void func() {
        cout << pub;    // OK: public → protected (demoted)
        cout << prot;   // OK: protected → protected
    }
};
```

### 🔧 Multiple Inheritance
```cpp
class A { public: void show() { cout << "A\n"; } };
class B { public: void show() { cout << "B\n"; } };

class C : public A, public B {
    // AMBIGUITY: C has two show() functions!
    void func() {
        A::show();   // explicitly specify which: A
        B::show();   // explicitly specify which: B
    }
};

C obj;
// obj.show();      // ERROR: ambiguous
obj.A::show();      // OK: explicitly call A's version
```

### 🔧 Diamond Problem & Virtual Inheritance
```cpp
class A { public: int x = 0; };
class B : public A { };
class C : public A { };
class D : public B, public C { };
// D has TWO copies of A! d.x is AMBIGUOUS

// Fix: Virtual inheritance
class B : virtual public A { };
class C : virtual public A { };
class D : public B, public C { };
// Now D has only ONE copy of A. d.x works!
```

### 🔧 Constructor Order with Inheritance & Members
```cpp
class Member {
public:
    Member(string s) { cout << "Member " << s << " created\n"; }
    ~Member()        { cout << "Member destroyed\n"; }
};

class Base {
    Member m;  // member object
public:
    Base() : m("base") { cout << "Base created\n"; }
    ~Base()             { cout << "Base destroyed\n"; }
};

class Derived : public Base {
    Member dm;  // member object
public:
    Derived() : dm("derived") { cout << "Derived created\n"; }
    ~Derived()                { cout << "Derived destroyed\n"; }
};

int main() { Derived d; }
```
**Output:**
```
Member base created      ← Base's member initialized first
Base created             ← Base constructor body
Member derived created   ← Derived's member initialized
Derived created          ← Derived constructor body
Derived destroyed        ← Derived destructor body first
Member destroyed         ← Derived's member destroyed
Base destroyed           ← Base destructor body
Member destroyed         ← Base's member destroyed

ORDER: Base members → Base body → Derived members → Derived body
REVERSE for destruction.
```

---

## 4. Polymorphism & Virtual Functions

### 🧠 Static vs Dynamic Binding
```cpp
// Static (compile-time) binding: function call resolved at compile time
// Dynamic (runtime) binding: function call resolved at runtime via vtable

class Animal {
public:
    void speak() { cout << "Animal speaks\n"; }        // static binding
    virtual void vspeak() { cout << "Animal speaks\n"; } // dynamic binding
};

class Dog : public Animal {
public:
    void speak() { cout << "Dog barks\n"; }
    void vspeak() override { cout << "Dog barks\n"; }
};

Animal *a = new Dog();  // base class pointer to derived object!
a->speak();    // "Animal speaks" — STATIC binding (compile-time type is Animal*)
a->vspeak();   // "Dog barks"    — DYNAMIC binding (runtime type is Dog*)

// WITHOUT virtual: base pointer always calls base function
// WITH virtual:    base pointer calls ACTUAL object's function
```

### 🔧 vtable (Virtual Function Table)
```cpp
// Every class with virtual functions has a vtable:
// - Array of pointers to virtual functions
// - One vtable per CLASS (shared by all instances)
// - Each object has a vptr (pointer to its class's vtable)

// Cost of virtual: one extra pointer per object (vptr)
//                 one extra indirection per virtual call
// But enables runtime polymorphism!
```

### 🔧 override and final (C++11)
```cpp
class Base {
    virtual void func() { }
    virtual void func2() { }
};

class Derived : public Base {
    void func() override { }     // explicit: this overrides Base::func
    // void func2() overide { }  // typo caught by compiler (not override)

    virtual void func() final { }  // no further overriding allowed
};

class Further : public Derived {
    // void func() override { }  // ERROR: func is final in Derived
};
```

### 🔧 Virtual Destructor — CRITICAL
```cpp
class Base {
public:
    ~Base() { cout << "Base destroyed\n"; }  // NON-virtual!
};

class Derived : public Base {
public:
    int *data;
    Derived() { data = new int[100]; }
    ~Derived() { delete[] data; cout << "Derived destroyed\n"; }
};

Base *b = new Derived();
delete b;   // PROBLEM! Only Base::~Base called — data LEAKED!
            // Derived destructor NEVER called → memory leak!

// FIX: always make destructor virtual in base classes intended for inheritance:
class Base {
public:
    virtual ~Base() { cout << "Base destroyed\n"; }  // virtual!
};
// Now: delete b → Derived::~Derived() → Base::~Base() ✓
```

---

## 5. Abstract Classes & Interfaces

### 🔧 Pure Virtual Functions
```cpp
class Shape {
public:
    // Pure virtual = 0: MUST be overridden in derived class
    virtual double area() = 0;
    virtual double perimeter() = 0;
    virtual void draw() = 0;

    // Can have non-pure virtual (provides default):
    virtual void describe() {
        cout << "I am a shape with area " << area() << "\n";
    }

    virtual ~Shape() { }  // always virtual destructor!
};

// Shape is ABSTRACT — cannot instantiate:
// Shape s;  // ERROR: has pure virtual functions

class Circle : public Shape {
    double radius;
public:
    Circle(double r) : radius(r) {}
    double area() override { return 3.14159 * radius * radius; }
    double perimeter() override { return 2 * 3.14159 * radius; }
    void draw() override { cout << "Drawing circle\n"; }
};

Shape *s = new Circle(5.0);  // base pointer to derived object ✓
s->area();       // calls Circle::area()
s->describe();   // calls Shape::describe() (which calls Circle::area())
delete s;        // calls Circle destructor then Shape destructor
```

### 🔧 Interface Pattern in C++
```cpp
// C++ has no interface keyword — use abstract class with all pure virtuals
class Drawable {
public:
    virtual void draw() = 0;
    virtual ~Drawable() {}
};

class Resizable {
public:
    virtual void resize(double factor) = 0;
    virtual ~Resizable() {}
};

// "Implement" multiple interfaces:
class Square : public Drawable, public Resizable {
    double side;
public:
    Square(double s) : side(s) {}
    void draw() override { cout << "Drawing square\n"; }
    void resize(double f) override { side *= f; }
};
```

---

## 6. Operator Overloading

### 🔧 Basics
```cpp
class Complex {
    double real, imag;
public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}

    // Overload + as member function:
    Complex operator+(const Complex &other) const {
        return Complex(real + other.real, imag + other.imag);
    }

    // Overload << as friend (needs access to private members):
    friend ostream& operator<<(ostream &out, const Complex &c) {
        out << c.real << " + " << c.imag << "i";
        return out;  // return stream for chaining: cout << c1 << c2
    }

    // Overload == :
    bool operator==(const Complex &other) const {
        return (real == other.real) && (imag == other.imag);
    }

    // Overload prefix ++:
    Complex& operator++() {
        real++; return *this;
    }

    // Overload postfix ++ (int is dummy parameter to distinguish):
    Complex operator++(int) {
        Complex temp = *this;
        real++;
        return temp;  // return old value
    }
};

Complex c1(1, 2), c2(3, 4);
Complex c3 = c1 + c2;    // calls operator+
cout << c3 << "\n";      // calls operator<< : prints "4 + 6i"
```

### 🔧 Rules for Operator Overloading
```
CAN overload:  + - * / % ^ & | ~ ! = < > += -= *= /= ...
CANNOT overload: :: (scope), .* (member pointer), . (member access), ?: (ternary), sizeof

Cannot change:
  - Operator precedence
  - Operator arity (unary stays unary, binary stays binary)
  - Operators for built-in types (can't change int + int)

Member vs non-member:
  Member:     left operand = calling object (a.op(b))
  Non-member (friend): both operands explicit (op(a,b))
  << and >> MUST be non-member (left side is ostream/istream, not your class)
```

---

## 7. Templates

### 🔧 Function Templates
```cpp
// Generic function — works with any type T
template <typename T>
T maximum(T a, T b) {
    return (a > b) ? a : b;
}

// Compiler generates specific version for each type used:
cout << maximum(3, 5);        // T=int → maximum<int>
cout << maximum(3.14, 2.71);  // T=double → maximum<double>
cout << maximum('a', 'z');    // T=char → maximum<char>

// Multiple template parameters:
template <typename T, typename U>
void display(T first, U second) {
    cout << first << " " << second << "\n";
}
display(1, 3.14);     // T=int, U=double
display("hello", 42); // T=const char*, U=int

// Template specialization (specific behavior for one type):
template <>
const char* maximum(const char* a, const char* b) {
    return (strcmp(a, b) > 0) ? a : b;  // special case for strings
}
```

### 🔧 Class Templates
```cpp
template <typename T>
class Stack {
    T data[100];
    int top = -1;
public:
    void push(T val) { data[++top] = val; }
    T pop() { return data[top--]; }
    T peek() const { return data[top]; }
    bool empty() const { return top == -1; }
    int size() const { return top + 1; }
};

Stack<int> intStack;
intStack.push(1);
intStack.push(2);
cout << intStack.pop();  // 2 (LIFO)

Stack<string> strStack;
strStack.push("hello");
strStack.push("world");
cout << strStack.peek();  // "world"

// Template with non-type parameter:
template <typename T, int SIZE>
class FixedArray {
    T arr[SIZE];
public:
    T& operator[](int i) { return arr[i]; }
};
FixedArray<int, 5> fa;   // SIZE=5 baked in at compile time
```

### 🔧 Template Output Prediction
```cpp
#include <iostream>
using namespace std;

template <typename T>
T add(T a, T b) {
    cout << "Template called\n";
    return a + b;
}

int add(int a, int b) {
    cout << "Non-template called\n";
    return a + b;
}

int main() {
    cout << add(1, 2) << "\n";       // (A)
    cout << add(1.0, 2.0) << "\n";   // (B)
    cout << add<int>(1, 2) << "\n";  // (C)
}
```
**Output:**
```
(A) add(1,2): exact match for non-template (int,int) → Non-template preferred
    Non-template called
    3

(B) add(1.0,2.0): no non-template for double → template generates add<double>
    Template called
    3

(C) add<int>(1,2): explicitly requesting template version
    Template called
    3
```

---

## 8. Exception Handling

### 🔧 try-catch-throw
```cpp
#include <stdexcept>

// throw: signal an exception
// try:   define code that might throw
// catch: handle the exception

int divide(int a, int b) {
    if (b == 0)
        throw runtime_error("Division by zero!");
    return a / b;
}

int main() {
    try {
        cout << divide(10, 2) << "\n";   // 5
        cout << divide(10, 0) << "\n";   // throws!
        cout << "This never prints\n";   // skipped after throw
    }
    catch (runtime_error &e) {
        cout << "Caught: " << e.what() << "\n";
    }
    catch (...) {
        cout << "Caught unknown exception\n";
    }
    cout << "After try-catch\n";  // still executes
    return 0;
}
```
**Output:**
```
5
Caught: Division by zero!
After try-catch
```

### 🔧 Exception Hierarchy
```cpp
// Standard exceptions (from <stdexcept>):
exception                    // base class
├── runtime_error
│   ├── overflow_error
│   ├── underflow_error
│   └── range_error
├── logic_error
│   ├── invalid_argument
│   ├── domain_error
│   ├── length_error
│   └── out_of_range
└── bad_alloc              // thrown by new when allocation fails
```

### 🔧 Custom Exception
```cpp
class MyException : public exception {
    string message;
public:
    MyException(string msg) : message(msg) {}
    const char* what() const noexcept override {
        return message.c_str();
    }
};

try {
    throw MyException("Custom error occurred");
}
catch (MyException &e) {
    cout << e.what();  // "Custom error occurred"
}
```

### 🔧 Stack Unwinding & Destructors
```cpp
class Resource {
public:
    Resource() { cout << "Resource acquired\n"; }
    ~Resource() { cout << "Resource released\n"; }
};

void func() {
    Resource r;        // constructor called
    throw runtime_error("Error!");
    // r's destructor called during stack unwinding!
}

try { func(); }
catch (...) { cout << "Exception caught\n"; }
```
**Output:**
```
Resource acquired
Resource released    ← destructor called BEFORE catch!
Exception caught
```

### 🔧 noexcept
```cpp
void safeFunc() noexcept { }         // promises not to throw
void mayThrow() noexcept(false) { }  // may throw

// If noexcept function throws → std::terminate() called (program aborts)
// Move constructors should be noexcept for STL performance
```

### 🔧 Exception Catch Order — Critical
```cpp
try {
    throw runtime_error("test");
}
catch (exception &e) { cout << "base caught\n"; }    // catches first!
catch (runtime_error &e) { cout << "derived\n"; }    // NEVER reached!

// RULE: catch blocks checked TOP TO BOTTOM
// Catch DERIVED EXCEPTIONS BEFORE BASE EXCEPTIONS
// Correct order:
catch (runtime_error &e) { ... }  // derived first
catch (exception &e) { ... }      // base second
catch (...) { ... }               // catch-all LAST
```

---

## 9. File I/O & Streams

### 🔧 Stream Hierarchy
```
ios
├── istream → ifstream (read from file)
├── ostream → ofstream (write to file)
└── iostream → fstream (read and write)
```

### 🔧 Writing to Files
```cpp
#include <fstream>

ofstream outFile("output.txt");         // creates/overwrites
ofstream appendFile("output.txt", ios::app);  // append mode

if (!outFile.is_open()) {
    cerr << "Failed to open file\n";
    return 1;
}

outFile << "Hello, World!\n";
outFile << 42 << " " << 3.14 << "\n";
outFile.close();  // good practice, auto-closed at end of scope
```

### 🔧 Reading from Files
```cpp
ifstream inFile("input.txt");

if (!inFile) {  // same as !inFile.is_open()
    cerr << "File not found\n";
}

// Read word by word:
string word;
while (inFile >> word) {
    cout << word << "\n";
}

// Read line by line:
string line;
while (getline(inFile, line)) {
    cout << line << "\n";
}

// Read character by character:
char c;
while (inFile.get(c)) {
    cout << c;
}

inFile.close();
```

### 🔧 File Open Modes
```cpp
ios::in      // read (default for ifstream)
ios::out     // write (default for ofstream, overwrites!)
ios::app     // append (write at end)
ios::trunc   // truncate on open (default with out)
ios::binary  // binary mode (no newline conversion)
ios::ate     // seek to end after open

// Combinations:
fstream f("data.bin", ios::in | ios::out | ios::binary);
```

### 🔧 String Streams (istringstream, ostringstream)
```cpp
#include <sstream>

// ostringstream: build strings efficiently
ostringstream oss;
oss << "Name: " << "Rahul" << ", Age: " << 20;
string result = oss.str();
cout << result;  // "Name: Rahul, Age: 20"

// istringstream: parse strings
string data = "42 3.14 hello";
istringstream iss(data);
int n; double d; string s;
iss >> n >> d >> s;
cout << n << " " << d << " " << s;  // 42 3.14 hello

// Convert string to int:
string numStr = "123";
istringstream(numStr) >> n;   // n = 123
// Or use stoi (C++11): int n = stoi(numStr);

// Convert int to string:
ostringstream os;
os << 42;
string str = os.str();         // "42"
// Or use to_string (C++11): string str = to_string(42);
```

### 🔧 Stream State & Error Handling
```cpp
ifstream f("test.txt");

f.good()   // true if no errors
f.eof()    // true if end of file reached
f.fail()   // true if operation failed (bad format etc.)
f.bad()    // true if serious error (hardware failure)

// Clear error state:
f.clear();
f.seekg(0);  // seek back to beginning

// Check after read:
int x;
if (!(f >> x)) {
    cout << "Read failed\n";
}
```

---

## 10. Output Prediction Drill

### 🎯 Question 1 — Constructor/Destructor Order
```cpp
#include <iostream>
using namespace std;
class A {
public:
    A() { cout << "A()\n"; }
    ~A() { cout << "~A()\n"; }
};
class B : public A {
public:
    B() { cout << "B()\n"; }
    ~B() { cout << "~B()\n"; }
};
class C : public B {
public:
    C() { cout << "C()\n"; }
    ~C() { cout << "~C()\n"; }
};
int main() {
    C obj;
    return 0;
}
```
**Predict then check:**
```
Construction: deepest base first → A → B → C
Destruction: reverse → C → B → A

Output:
A()
B()
C()
~C()
~B()
~A()
```

---

### 🎯 Question 2 — Virtual Function
```cpp
#include <iostream>
using namespace std;
class Base {
public:
    virtual void show() { cout << "Base\n"; }
    void print() { cout << "Base print\n"; }
};
class Derived : public Base {
public:
    void show() override { cout << "Derived\n"; }
    void print() { cout << "Derived print\n"; }
};
int main() {
    Base *b = new Derived();
    b->show();    // (1)
    b->print();   // (2)
    Derived *d = new Derived();
    d->show();    // (3)
    d->print();   // (4)
    delete b; delete d;
    return 0;
}
```
**Predict then check:**
```
b is Base* pointing to Derived object.
(1) show() is VIRTUAL → dynamic binding → Derived::show() → "Derived"
(2) print() is NOT virtual → static binding → Base::print() → "Base print"

d is Derived* → direct Derived object.
(3) show() → Derived::show() → "Derived"
(4) print() → Derived::print() → "Derived print"

Output:
Derived
Base print
Derived
Derived print
```

---

### 🎯 Question 3 — Virtual Destructor
```cpp
#include <iostream>
using namespace std;
class Base {
public:
    Base()  { cout << "Base ctor\n"; }
    ~Base() { cout << "Base dtor\n"; }  // NOT virtual
};
class Derived : public Base {
public:
    Derived()  { cout << "Derived ctor\n"; }
    ~Derived() { cout << "Derived dtor\n"; }
};
int main() {
    Base *b = new Derived();
    delete b;
    return 0;
}
```
**Predict then check:**
```
Constructors: Base ctor, then Derived ctor
delete b: b is Base* → calls Base::~Base() ONLY (non-virtual!)
          Derived destructor NEVER called! (memory leak / UB)

Output:
Base ctor
Derived ctor
Base dtor

← Derived dtor NOT printed! (UB/leak)
```

---

### 🎯 Question 4 — Exception Flow
```cpp
#include <iostream>
using namespace std;
void func(int x) {
    cout << "Before throw\n";
    if (x < 0) throw x;
    cout << "After throw\n";
}
int main() {
    try {
        func(5);
        func(-1);
        func(3);
    }
    catch (int e) {
        cout << "Caught: " << e << "\n";
    }
    cout << "After try-catch\n";
    return 0;
}
```
**Predict then check:**
```
func(5):  x=5 ≥ 0, no throw.
  Before throw
  After throw

func(-1): x=-1 < 0, throws -1.
  Before throw
  (jumps to catch, skips "After throw" and func(3))

catch(int e): e=-1
  Caught: -1

After try-catch

Output:
Before throw
After throw
Before throw
Caught: -1
After try-catch
```

---

### 🎯 Question 5 — Template Deduction
```cpp
#include <iostream>
using namespace std;
template <typename T>
void print(T x) {
    cout << "T: " << x << "\n";
}
template <typename T>
void print(T* x) {
    cout << "T*: " << *x << "\n";
}
int main() {
    int n = 42;
    print(n);
    print(&n);
    print<int>(n);
    print<int*>(&n);
    return 0;
}
```
**Predict then check:**
```
print(n): n is int → T=int → "T: 42"
print(&n): &n is int* → matches T* specialization → T=int, x=&n → "T*: 42"
print<int>(n): explicitly T=int, not pointer → "T: 42"
print<int*>(&n): T=int*, x is int** → "T*: 42" (dereferences to int*)
  Wait: if T=int*, then T* = int**. &n is int*. int* ≠ int**.
  So print<int*>(&n) calls void print(T x) with T=int*, x=&n.
  cout << "T: " << x → prints address!

Output:
T: 42
T*: 42
T: 42
T: [some address]
```

---

### 🎯 Question 6 — this pointer & static
```cpp
#include <iostream>
using namespace std;
class Counter {
    static int total;
    int id;
public:
    Counter() { id = ++total; cout << "Created #" << id << "\n"; }
    ~Counter()              { cout << "Destroyed #" << id << "\n"; }
    static int getTotal()   { return total; }
};
int Counter::total = 0;

int main() {
    Counter a;
    {
        Counter b;
        Counter c;
        cout << "Total: " << Counter::getTotal() << "\n";
    }  // b and c destroyed here
    cout << "Total: " << Counter::getTotal() << "\n";
    return 0;
}  // a destroyed here
```
**Predict then check:**
```
Counter a: total=1, id=1 → "Created #1"
Counter b: total=2, id=2 → "Created #2"
Counter c: total=3, id=3 → "Created #3"
Total: 3

Block ends: c destroyed first (LIFO), then b:
"Destroyed #3"
"Destroyed #2"
Note: static total is NOT decremented (we didn't code that)

Total: 3 (static not changed)

main ends: a destroyed:
"Destroyed #1"

Output:
Created #1
Created #2
Created #3
Total: 3
Destroyed #3
Destroyed #2
Total: 3
Destroyed #1
```

---

## 11. MCQ Traps & Exam Q&A

### ⚠️ THE TRAPS

---

**TRAP 1: Virtual destructor requirement**
```
If class has virtual functions AND is used polymorphically (base ptr to derived):
Base class destructor MUST be virtual.
Otherwise: only base destructor called on delete → derived destructor skipped → memory leak!
```

---

**TRAP 2: Constructor/Destructor order**
```
Always: Base constructed FIRST, Derived LAST.
Always: Derived destroyed FIRST, Base LAST.
Members destroyed in REVERSE order of declaration.
```

---

**TRAP 3: Non-virtual function through base pointer**
```cpp
Base *b = new Derived();
b->nonVirtual();  // calls BASE version (static binding)
b->virtualFunc(); // calls DERIVED version (dynamic binding)
```

---

**TRAP 4: Exception catch order**
```
Derived exceptions BEFORE base exceptions.
catch(base) before catch(derived) → derived NEVER caught!
catch(...) MUST be last.
```

---

**TRAP 5: Assigning to reference in catch**
```cpp
catch (exception e) { ... }   // copies exception — may slice!
catch (exception &e) { ... }  // reference — no copy, no slicing ✓
```

---

**TRAP 6: Template vs non-template preference**
```
When exact match exists for non-template → non-template preferred.
Explicit template syntax (func<int>) forces template version.
```

---

**TRAP 7: operator<< must be non-member**
```cpp
// WRONG: member function would need:  myObj << cout;
// RIGHT: friend/non-member:           cout << myObj;
friend ostream& operator<<(ostream&, const MyClass&);
```

---

**TRAP 8: Abstract class instantiation**
```cpp
class Abstract { virtual void f() = 0; };
Abstract a;  // ERROR: cannot instantiate abstract class
Abstract *p = new Concrete();  // OK: pointer to abstract class is fine
```

---

### ⚡ Quick Fire Q&A

| Question | Answer |
|---|---|
| Constructor order with inheritance? | Base → Derived (most base first) |
| Destructor order with inheritance? | Derived → Base (reverse of construction) |
| Virtual function binding type? | Dynamic (runtime) binding |
| Non-virtual function binding type? | Static (compile-time) binding |
| When is virtual destructor needed? | When deleting derived via base pointer |
| Can abstract class be instantiated? | No — can only use pointers/references |
| Pure virtual syntax? | `virtual void f() = 0;` |
| Default access in class vs struct? | class=private, struct=public |
| `this` pointer type in `MyClass`? | `MyClass * const this` |
| Can static member function access `this`? | No — no object context |
| What is vtable? | Per-class table of virtual function pointers |
| Operator that CANNOT be overloaded? | `::`, `.*`, `.`, `?:`, `sizeof` |
| `operator<<` should be member or friend? | Friend (non-member) |
| `try` block exits immediately after throw? | Yes — remaining code skipped |
| Stack unwinding calls what? | Destructors of all local objects in scope |
| `noexcept` function throws — what happens? | `std::terminate()` called |
| Template specialization syntax? | `template <> void f<int>(int x) { ... }` |
| `ios::app` vs `ios::trunc`? | app=append to end, trunc=erase file |
| `getline` vs `>>` for strings? | getline reads whole line, >> stops at space |
| `ostringstream` main use? | Build strings efficiently, number-to-string conversion |

---

### 📋 OOP Quick Reference

```
ACCESS SPECIFIERS:
  public:    accessible everywhere
  protected: class + derived classes
  private:   class only (default in class)

INHERITANCE ACCESS CHANGES:
  public B:    public→public, protected→protected
  protected B: public→protected, protected→protected
  private B:   public→private, protected→private
  private never inherited.

POLYMORPHISM RULES:
  virtual = dynamic binding (runtime)
  no virtual = static binding (compile-time)
  Base* → Derived object: virtual calls Derived, non-virtual calls Base
  ALWAYS: virtual destructor if deleting polymorphically

CONSTRUCTOR ORDER: Base → Derived (most base first)
DESTRUCTOR ORDER:  Derived → Base (reverse)

EXCEPTION RULES:
  catch order: most derived first, base last, ... last
  reference catch: catch(Type &e) preferred over catch(Type e)
  stack unwinding: all local destructors called before catch

TEMPLATE RULES:
  non-template preferred over template for exact match
  explicit: func<Type>(arg) forces template
  class template member defined outside: template<T> RetType ClassName<T>::func()
```

---
