# Buran for C Programmers

## Introduction

If you know C, you understand the machine. You manage memory, manipulate pointers, and control every byte. You write code that runs fast because you tell the computer exactly what to do.

Buran is the opposite. No pointers, no memory management, no low-level control. Everything is a pattern, and all computation is pattern transformation. You describe *what* you want, not *how* to compute it.

This is a significant paradigm shift. This document will help you understand Buran by showing correspondences with C constructs, while being honest about what you gain and what you give up.

---

## The Paradigm Shift

C is imperative: step-by-step instructions for the machine.

Buran is declarative: descriptions of pattern transformations.

**C:**

```c
int factorial(int n) {
    if (n == 0) return 1;
    return n * factorial(n - 1);
}
```

**Buran:**

```
factorial {
    [0] ↦ [1]
    [𝑛] ↦ [𝑛 × factorial(𝑛 − 1)]
}
```

Same algorithm, different worldview. C tells the computer how to compute. Buran declares what patterns become.

---

## No Manual Memory Management

C gives you complete control over memory:

**C:**

```c
// Allocate
int *arr = malloc(10 * sizeof(int));
if (arr == NULL) {
    // Handle error
}

// Use
for (int i = 0; i < 10; i++) {
    arr[i] = i * 2;
}

// Free
free(arr);
arr = NULL;  // Avoid dangling pointer
```

**Buran:**

```
# Just create data — runtime handles memory
[list: 0, 2, 4, 6, 8, 10, 12, 14, 16, 18] ↦ arr

# Or generate it
map({ 𝑖 ↦ 𝑖 × 2 }, range([0], [9])) ↦ arr
```

No `malloc`, no `free`, no memory leaks, no dangling pointers, no buffer overflows. The runtime manages everything.

---

## No Pointers

C uses pointers for everything — indirection, arrays, strings, data structures:

**C:**

```c
int x = 42;
int *p = &x;        // Pointer to x
*p = 100;           // Modify through pointer

int arr[5] = {1, 2, 3, 4, 5};
int *ptr = arr;     // Array decays to pointer
int third = *(ptr + 2);  // Pointer arithmetic

void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}
```

**Buran:**

```
# No pointers — values are immutable
[42] ↦ x
# x cannot be "modified through a pointer"

# Lists are values, not pointer-based
[list: 1, 2, 3, 4, 5] ↦ arr
[list: _, _, third, ⟨_⟩] ↦ arr    # Destructure to get third

# No swap — return new values instead
swap {
    𝑎, 𝑏 ↦ [𝑏, 𝑎]
}
```

Everything is a value. No addresses, no dereferencing, no pointer arithmetic.

---

## Basic Syntax Correspondences

### Variables

**C:**

```c
int x = 42;
char *name = "hello";
double pi = 3.14159;
```

**Buran:**

```
[42] ↦ x
["hello"] ↦ name
𝛑 ↦ pi    # Built-in constant
```

All values in square brackets. No type declarations — structure determines type.

### Functions

**C:**

```c
int add(int a, int b) {
    return a + b;
}

char* greet(char *name) {
    // Would need malloc, sprintf, etc.
}
```

**Buran:**

```
add {
    𝑎, 𝑏 ↦ [𝑎 + 𝑏]
}

greet {
    name ↦ [string: "Hello, ", name, "!"]
}
```

No return type, no parameter types, no `return` statement.

### Function Calls

**C:**

```c
int sum = add(2, 3);
```

**Buran:**

```
add([2], [3]) ↦ sum
```

Literal arguments need brackets. Variables don't: `add(x, y)`.

---

## Structs → Constructors

**C:**

```c
struct Point {
    double x;
    double y;
};

struct Point p = {3.0, 4.0};
double dist = sqrt(p.x * p.x + p.y * p.y);

// With typedef
typedef struct {
    unsigned char r, g, b;
} Color;

Color red = {255, 0, 0};
```

**Buran:**

```
# Just create the pattern — no declaration needed
[point: 3.0, 4.0] ↦ p

distance {
    [point: 𝑥, 𝑦] ↦ [√(𝑥² + 𝑦²)]
}

distance(p) ↦ dist    # [5.0]

# Colors the same way
[rgb: 255, 0, 0] ↦ red
```

No struct declaration, no typedef. Patterns are self-describing.

### Defining Types with Identity

If you need type metadata (like C's type system):

**Buran:**

```
⟨[point: _, _]⟩ ↤ [identity:
    "type": "Point",
    "fields": ["x", "y"]
]

⟨[rgb: _, _, _]⟩ ↤ [identity:
    "type": "Color",
    "description": "RGB color 0-255"
]
```

---

## Unions → Multiple Constructors

**C:**

```c
typedef enum { CIRCLE, RECTANGLE } ShapeType;

typedef struct {
    ShapeType type;
    union {
        struct { double radius; } circle;
        struct { double width, height; } rectangle;
    };
} Shape;

double area(Shape s) {
    switch (s.type) {
        case CIRCLE:
            return M_PI * s.circle.radius * s.circle.radius;
        case RECTANGLE:
            return s.rectangle.width * s.rectangle.height;
    }
}
```

**Buran:**

```
# No union — different patterns for different cases
area {
    [circle: 𝑟] ↦ [𝛑 × 𝑟²]
    [rectangle: 𝑤, ℎ] ↦ [𝑤 × ℎ]
}

# Usage
area([circle: 5.0]) ↦ result      # [78.54...]
area([rectangle: 3.0, 4.0]) ↦ result  # [12.0]
```

Pattern matching replaces tagged unions and switch statements.

---

## Function Pointers → Pattern Blocks

**C:**

```c
int add(int a, int b) { return a + b; }
int mul(int a, int b) { return a * b; }

int apply(int (*f)(int, int), int x, int y) {
    return f(x, y);
}

int result = apply(add, 2, 3);  // 5
int result2 = apply(mul, 2, 3); // 6

// Array of function pointers
int (*ops[])(int, int) = {add, mul};
```

**Buran:**

```
add { 𝑎, 𝑏 ↦ [𝑎 + 𝑏] }
mul { 𝑎, 𝑏 ↦ [𝑎 × 𝑏] }

apply {
    f, 𝑥, 𝑦 ↦ f(𝑥, 𝑦)
}

apply(add, [2], [3]) ↦ result    # [5]
apply(mul, [2], [3]) ↦ result2   # [6]

# Anonymous functions (like function pointers to lambdas)
apply({ 𝑎, 𝑏 ↦ 𝑎 − 𝑏 }, [5], [3]) ↦ result3  # [2]
```

Functions are first-class values. Pattern blocks `{ ... }` are anonymous functions.

---

## Arrays → Lists

**C:**

```c
int arr[5] = {1, 2, 3, 4, 5};
int first = arr[0];
int len = sizeof(arr) / sizeof(arr[0]);

// Dynamic array
int *darr = malloc(n * sizeof(int));
// ... use it ...
free(darr);

// Multidimensional
int matrix[3][3] = {{1,2,3}, {4,5,6}, {7,8,9}};
```

**Buran:**

```
[list: 1, 2, 3, 4, 5] ↦ arr

# Destructuring
[list: first, ⟨_⟩] ↦ arr

length(arr) ↦ len

# All lists are dynamic (runtime managed)
# No malloc/free needed

# Nested lists
[list:
    [list: 1, 2, 3],
    [list: 4, 5, 6],
    [list: 7, 8, 9]
] ↦ matrix
```

---

## Strings

**C:**

```c
char *str = "hello";
int len = strlen(str);
char buffer[100];
strcpy(buffer, str);
strcat(buffer, " world");
// buffer = "hello world"

// Manual memory for dynamic strings
char *result = malloc(len1 + len2 + 1);
strcpy(result, str1);
strcat(result, str2);
// Don't forget to free!
```

**Buran:**

```
["hello"] ↦ str
length(str) ↦ len

# Concatenation — no buffer management
[string: str, " world"] ↦ result    # ["hello world"]

# Multiple strings
[string: s1, s2, s3] ↦ combined
```

No null terminators, no buffer overflows, no manual allocation.

---

## Error Handling

**C:**

```c
FILE *f = fopen("data.txt", "r");
if (f == NULL) {
    perror("Error opening file");
    return -1;
}

// Or with errno
int result = some_operation();
if (result == -1) {
    fprintf(stderr, "Error: %s\n", strerror(errno));
}
```

**Buran:**

```
[File: "data.txt"] ↦ {
    [error: msg] ↦ msg ↦ [stderr]
    content ↦ process(content)
}

# Errors are patterns, not return codes
some-operation() ↦ {
    [error: e] ↦ handle-error(e)
    result ↦ use-result(result)
}
```

No error codes, no errno, no NULL checks. Pattern matching handles all cases.

---

## NULL → Empty Pattern

**C:**

```c
int *p = NULL;
if (p != NULL) {
    use(*p);
}

char *find(char *haystack, char needle) {
    // Returns NULL if not found
    for (int i = 0; haystack[i]; i++) {
        if (haystack[i] == needle) {
            return &haystack[i];
        }
    }
    return NULL;
}
```

**Buran:**

```
# Empty pattern represents absence
[] ↦ nothing

# Pattern matching handles both cases
find(text, char) ↦ {
    [] ↦ handle-not-found()
    result ↦ use-result(result)
}

# Function that might not find something
find-char {
    [""], _ ↦ []    # Not found
    [string: c, ⟨_⟩], c ↦ [0]    # Found at position 0
    [string: _, ⟨rest⟩], c ↦
        find-char(rest, c) ↦ {
            [] ↦ []
            [pos] ↦ [pos + 1]
        }
}
```

No NULL pointer dereferences. The empty pattern `[]` is explicit.

---

## Preprocessor → None

**C:**

```c
#define MAX(a, b) ((a) > (b) ? (a) : (b))
#define ARRAY_SIZE(arr) (sizeof(arr) / sizeof(arr[0]))
#define DEBUG 1

#ifdef DEBUG
    printf("Debug: x = %d\n", x);
#endif

#include "myheader.h"
```

**Buran:**

```
# No preprocessor — use functions
max {
    𝑎, 𝑏 | 𝑎 > 𝑏 ↦ 𝑎
    𝑎, 𝑏 ↦ 𝑏
}

# Length is a function
length(arr) ↦ size

# Conditional compilation not applicable
# Include via file connection
[Include: "mymodule.buran"]
```

No macros, no `#define`, no `#ifdef`. Functions and pattern matching handle everything.

---

## Control Flow

**C:**

```c
// If-else
if (x > 0) {
    result = 1;
} else if (x < 0) {
    result = -1;
} else {
    result = 0;
}

// Switch
switch (op) {
    case '+': result = a + b; break;
    case '-': result = a - b; break;
    case '*': result = a * b; break;
    default: result = 0;
}

// For loop
for (int i = 0; i < n; i++) {
    process(i);
}

// While loop
while (condition) {
    // ...
}
```

**Buran:**

```
# Pattern matching replaces if/switch
sign {
    [𝑥 | 𝑥 > 0] ↦ [1]
    [𝑥 | 𝑥 < 0] ↦ [−1]
    [0] ↦ [0]
}

calculate {
    ["+"], 𝑎, 𝑏 ↦ [𝑎 + 𝑏]
    ["-"], 𝑎, 𝑏 ↦ [𝑎 − 𝑏]
    ["*"], 𝑎, 𝑏 ↦ [𝑎 × 𝑏]
    _, _, _ ↦ [0]
}

# Recursion replaces loops
process-range {
    [𝑛], _ | 𝑛 < 0 ↦ []
    [𝑛], f ↦ f([𝑛]) ↦ _ ↦ process-range([𝑛 − 1], f)
}

# Or use map
map({ 𝑖 ↦ process([𝑖]) }, range([0], [𝑛 − 1]))
```

No `if`, no `switch`, no `for`, no `while`. Pattern matching and recursion.

---

## File I/O

**C:**

```c
FILE *f = fopen("data.txt", "r");
if (f == NULL) {
    perror("fopen");
    return 1;
}

char buffer[1024];
while (fgets(buffer, sizeof(buffer), f)) {
    process(buffer);
}

fclose(f);

// Write
FILE *out = fopen("output.txt", "w");
fprintf(out, "Hello, %s!\n", name);
fclose(out);
```

**Buran:**

```
# Read entire file
[File: "data.txt"] ↦ content

# Write
[string: "Hello, ", name, "!"] ↦ [File: "output.txt"]

# With format parsing
[File: "config.json" :: json] ↦ config
data ↦ [File: "output.json" :: json]
```

No file handles, no fopen/fclose, no buffer management.

---

## Mathematical Notation

C uses ASCII and library functions. Buran uses mathematical symbols:

| C           | Buran | Meaning               |
| ----------- | ----- | --------------------- |
| `*`         | `×`   | Multiplication        |
| `/`         | `÷`   | Division              |
| `pow(x, 2)` | `𝑥²`  | Square                |
| `sqrt(x)`   | `√𝑥`  | Square root           |
| `cbrt(x)`   | `∛𝑥`  | Cube root             |
| `M_PI`      | `𝛑`   | Pi                    |
| `M_E`       | `𝐞`   | Euler's number        |
| `<=`        | `≤`   | Less than or equal    |
| `>=`        | `≥`   | Greater than or equal |
| `!=`        | `≠`   | Not equal             |
| `&&`        | `∧`   | Logical AND           |
| `\|\|`      | `∨`   | Logical OR            |
| `!`         | `¬`   | Logical NOT           |
| `1` (true)  | `[✔]` | Boolean true          |
| `0` (false) | `[✘]` | Boolean false         |
| `NULL`      | `[]`  | Empty/null            |

---

## No Undefined Behavior

C has extensive undefined behavior:

**C:**

```c
int x;              // Uninitialized — UB to read
int *p = NULL;
*p = 5;             // NULL dereference — UB

int a[5];
a[10] = 1;          // Buffer overflow — UB

int i = INT_MAX;
i++;                // Signed overflow — UB

int arr[3] = {1, 2, 3};
int *p = &arr[2];
p++;                // Past-the-end pointer — UB to dereference
```

**Buran:**

```
# No uninitialized values — all bindings require values
[42] ↦ x

# No pointers — no null dereference
# No arrays — no buffer overflow
# No machine integers — no overflow
# Everything is well-defined
```

Buran has no undefined behavior. Every program has defined semantics.

---

## Comparison Table

| Concept          | C                     | Buran                    |
| ---------------- | --------------------- | ------------------------ |
| Function         | `int f(int x) { }`    | `f { pattern ↦ result }` |
| Variable         | `int x = 1;`          | `[1] ↦ x`                |
| Pointer          | `int *p = &x;`        | Not needed               |
| malloc/free      | `malloc()`, `free()`  | Automatic                |
| Struct           | `struct S { };`       | `[constructor: ...]`     |
| Union            | `union { };`          | Different constructors   |
| Array            | `int arr[5];`         | `[list: ...]`            |
| String           | `char *s = "...";`    | `["..."]`                |
| NULL             | `NULL`                | `[]`                     |
| Function pointer | `int (*f)(int)`       | First-class functions    |
| If/else          | `if () { } else { }`  | Pattern matching         |
| Switch           | `switch () { case: }` | Pattern matching         |
| For loop         | `for (;;)`            | Recursion or map         |
| While            | `while ()`            | Tail recursion           |
| #define          | `#define X 1`         | `[1] ↦ X`                |
| #include         | `#include "x.h"`      | `[Include: "x.buran"]`   |
| Error            | `errno`, return codes | `[error: ...]` patterns  |
| printf           | `printf("%d", x)`     | `x ↦ [stdout]`           |

---

## Example: Linked List

**C:**

```c
typedef struct Node {
    int value;
    struct Node *next;
} Node;

Node* create_node(int value) {
    Node *n = malloc(sizeof(Node));
    n->value = value;
    n->next = NULL;
    return n;
}

void free_list(Node *head) {
    while (head) {
        Node *next = head->next;
        free(head);
        head = next;
    }
}

int length(Node *head) {
    int count = 0;
    while (head) {
        count++;
        head = head->next;
    }
    return count;
}

Node* reverse(Node *head) {
    Node *prev = NULL;
    while (head) {
        Node *next = head->next;
        head->next = prev;
        prev = head;
        head = next;
    }
    return prev;
}
```

**Buran:**

```
# Lists are built-in — no manual implementation
[list: 1, 2, 3, 4, 5] ↦ list

length {
    [list: ] ↦ [0]
    [list: _, ⟨rest⟩] ↦ [1 + length(rest)]
}

reverse {
    [list: ] ↦ [list: ]
    [list: head, ⟨rest⟩] ↦ [list: ⟨reverse(rest)⟩, head]
}

# No malloc, no free, no memory leaks
```

---

## Example: Binary Tree

**C:**

```c
typedef struct Tree {
    int value;
    struct Tree *left;
    struct Tree *right;
} Tree;

Tree* create_leaf(int value) {
    Tree *t = malloc(sizeof(Tree));
    t->value = value;
    t->left = NULL;
    t->right = NULL;
    return t;
}

int tree_sum(Tree *t) {
    if (t == NULL) return 0;
    return t->value + tree_sum(t->left) + tree_sum(t->right);
}

void free_tree(Tree *t) {
    if (t == NULL) return;
    free_tree(t->left);
    free_tree(t->right);
    free(t);
}
```

**Buran:**

```
sum {
    [] ↦ [0]
    [leaf: 𝑛] ↦ 𝑛
    [node: left, 𝑣, right] ↦ [sum(left) + 𝑣 + sum(right)]
}

# Create a tree
[node:
    [leaf: 1],
    5,
    [node: [leaf: 3], 7, [leaf: 4]]
] ↦ tree

sum(tree) ↦ result    # [20]

# No manual memory management
```

---

## What Buran Trades Away

Coming from C, you should know what Buran doesn't provide:

1. **Performance control** — No cache optimization, no SIMD, no inline assembly
2. **Memory control** — No allocation control, no memory layout
3. **Direct hardware access** — No pointers, no memory-mapped I/O
4. **Bit manipulation** — No bit fields, no bitwise struct packing
5. **Deterministic performance** — GC pauses possible
6. **Systems programming** — Can't write kernels, drivers, embedded code
7. **Minimal runtime** — Requires runtime support

## What Buran Offers Instead

1. **No memory bugs** — No leaks, no dangling pointers, no buffer overflows
2. **No undefined behavior** — Every program is well-defined
3. **Simpler code** — Linked list in 4 lines, not 40
4. **Mathematical notation** — Write `√𝑥` not `sqrt(x)`
5. **Pattern matching** — Built into every function
6. **AI-friendly syntax** — Consistent structure for code generation

---

## When to Use Which

**Use C when:**

- Writing operating systems, drivers, or embedded code
- Maximum performance is critical
- You need direct hardware control
- Working with existing C codebases
- Memory layout matters

**Use Buran when:**

- Correctness matters more than performance
- You want to avoid memory bugs entirely
- The domain is data transformation
- AI systems are generating code
- Mathematical notation aids readability

---

## Philosophical Difference

C asks: *"What bytes should the machine manipulate, and how?"*

Buran asks: *"What pattern does this pattern become?"*

C is a thin layer over assembly — you're telling the machine what to do. Buran is a mathematical language — you're describing transformations.

They solve different problems. C gives you complete control for systems programming. Buran gives you safety and simplicity for application logic.

If you're comfortable with C's power and responsibility, you might find Buran limiting. But if you've ever spent hours debugging memory corruption, you might appreciate Buran's approach: make those bugs impossible by design.

---

*Buran is in development. Specification and reference implementation expected early 2026.*

© 2026 Danslav Slavenskoj
