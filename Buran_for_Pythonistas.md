# Buran for Python Programmers

## Introduction

If you know Python, you understand readable code. Buran shares that philosophy but approaches computation differently: everything is a pattern, and all computation is pattern transformation.

This document will help you transition from Python to Buran by showing correspondences between familiar constructs and new syntax.

---

## The Core Difference

Python is imperative: you tell the computer *how* to do things step by step.

Buran is declarative: you describe *what* patterns transform into other patterns.

**Python:**

```python
def factorial(n):
    if n == 0:
        return 1
    else:
        return n * factorial(n - 1)
```

**Buran:**

```
factorial {
    [0] ↦ [1]
    [𝑛] ↦ [𝑛 × factorial(𝑛 − 1)]
}
```

In Buran, there's no `if/else`, no `return`. You simply declare: "the pattern `[0]` transforms to `[1]`" and "the pattern `[𝑛]` transforms to `[𝑛 × factorial(𝑛 − 1)]`".

---

## Basic Syntax Correspondences

### Variables and Values

**Python:**

```python
x = 42
name = "hello"
items = [1, 2, 3]
```

**Buran:**

```
[42] ↦ 𝑥
["hello"] ↦ name
[list: 1, 2, 3] ↦ items
```

Key differences:

- Values are always in square brackets: `[42]`, `["hello"]`
- Arrow `↦` indicates data flow (like assignment, but bidirectional)
- Mathematical italic `𝑥` for math variables, ASCII `name` for regular variables

### Functions

**Python:**

```python
def greet(name):
    return f"Hello, {name}!"

def add(a, b):
    return a + b
```

**Buran:**

```
greet {
    name ↦ [string: "Hello, ", name, "!"]
}

add {
    𝑎, 𝑏 ↦ [𝑎 + 𝑏]
}
```

Buran functions are pattern matching blocks. Each clause matches a pattern and transforms it.

### Function Calls

**Python:**

```python
result = greet("World")
total = add(2, 3)
```

**Buran:**

```
greet("World") ↦ result
add([2], [3]) ↦ total
```

Note: Literal arguments need brackets: `add([2], [3])`. Variables don't: `add(𝑥, 𝑦)`.

---

## Pattern Matching vs if/elif/else

Python's conditionals become pattern matching in Buran:

**Python:**

```python
def classify(n):
    if n > 0:
        return "positive"
    elif n < 0:
        return "negative"
    else:
        return "zero"
```

**Buran:**

```
classify {
    [𝑛 | 𝑛 > 0] ↦ ["positive"]
    [𝑛 | 𝑛 < 0] ↦ ["negative"]
    [0] ↦ ["zero"]
}
```

The `|` introduces a **guard** — a condition that must be true for the pattern to match.

### Multiple Conditions

**Python:**

```python
def in_range(n):
    if n >= 0 and n <= 100:
        return True
    return False
```

**Buran:**

```
in-range {
    [𝑛 | 𝑛 ≥ 0 ∧ 𝑛 ≤ 100] ↦ [✔]
    _ ↦ [✘]
}
```

- `∧` is logical AND (like Python's `and`)
- `∨` is logical OR (like Python's `or`)
- `¬` is logical NOT (like Python's `not`)
- `[✔]` and `[✘]` are Buran's booleans (like `True` and `False`)
- `_` is a wildcard that matches anything (like Python's `_` in match statements)

---

## Lists and Sequences

**Python:**

```python
items = [1, 2, 3, 4, 5]
first = items[0]
rest = items[1:]
empty = []
```

**Buran:**

```
[list: 1, 2, 3, 4, 5] ↦ items
[list: first, ⟨rest⟩] ↦ items    # destructuring
[list: ] ↦ empty
```

The mirror brackets `⟨rest⟩` capture the remaining elements — similar to Python's `*rest` unpacking.

### List Operations

**Python:**

```python
def length(lst):
    if not lst:
        return 0
    return 1 + length(lst[1:])

def reverse(lst):
    if not lst:
        return []
    return reverse(lst[1:]) + [lst[0]]
```

**Buran:**

```
length {
    [list: ] ↦ [0]
    [list: _, ⟨rest⟩] ↦ [1 + length(rest)]
}

reverse {
    [list: ] ↦ [list: ]
    [list: first, ⟨rest⟩] ↦ [list: ⟨reverse(rest)⟩, first]
}
```

---

## map, filter, reduce

**Python:**

```python
doubled = list(map(lambda x: x * 2, numbers))
evens = list(filter(lambda x: x % 2 == 0, numbers))
total = reduce(lambda acc, x: acc + x, numbers, 0)
```

**Buran:**

```
map({ 𝑥 ↦ 𝑥 × 2 }, numbers) ↦ doubled
filter({ 𝑛 | 𝑛 mod 2 = 0 }, items) ↦ evens
fold({ acc, 𝑥 ↦ acc + 𝑥 }, [0], numbers) ↦ total
```

Pattern blocks `{ ... }` are Buran's anonymous functions (lambdas):

- `{ 𝑥 ↦ 𝑥 × 2 }` — takes `𝑥`, returns `𝑥 × 2`
- `{ 𝑛 | 𝑛 mod 2 = 0 }` — matches even numbers only
- `{ acc, 𝑥 ↦ acc + 𝑥 }` — takes accumulator and element

---

## Dictionaries → Maps

**Python:**

```python
person = {"name": "Alice", "age": 30}
name = person["name"]
```

**Buran:**

```
[map: "name", "Alice", "age", 30] ↦ person
# Access through pattern matching
```

Note: Buran maps are key-value pairs, not nested structures. Access happens through pattern matching rather than subscript notation.

---

## Sets

**Python:**

```python
unique = {1, 2, 3}
has_two = 2 in unique
combined = unique | {4, 5}
```

**Buran:**

```
[set: 1, 2, 3] ↦ unique
[2 ∈ unique] ↦ has-two          # membership test
[unique ∪ [set: 4, 5]] ↦ combined   # union
```

Set operations use mathematical symbols:

- `∈` — element of
- `∉` — not element of
- `∪` — union
- `∩` — intersection
- `⊂` — proper subset
- `∖` — set difference

---

## Error Handling

**Python:**

```python
try:
    result = risky_operation()
except FileNotFoundError as e:
    handle_error(e)
```

**Buran:**

```
risky-operation() ↦ {
    [error: message] ↦ handle-error(message)
    result ↦ process(result)
}
```

Buran uses pattern matching on error patterns. No `try/except` — errors are just patterns to match.

### Type-based Error Handling

```
handle {
    ⟨e | "type": "error", "category": "io"⟩ ↦ handle-io-error(e)
    ⟨e | "type": "error"⟩ ↦ handle-general-error(e)
    result ↦ process(result)
}
```

The `⟨...⟩` mirror brackets access metadata, including error type and category.

---

## File I/O

**Python:**

```python
# Reading
with open("data.txt") as f:
    content = f.read()

# Writing
with open("output.txt", "w") as f:
    f.write(data)

# JSON
import json
with open("config.json") as f:
    config = json.load(f)
```

**Buran:**

```
# Reading
[File: "data.txt"] ↦ content

# Writing
data ↦ [File: "output.txt"]

# JSON (automatic parsing)
[File: "config.json" :: json] ↦ config
```

Buran has built-in format support:

- `:: json` — parse/serialize JSON
- `:: yaml` — parse/serialize YAML
- `:: csv` — parse/serialize CSV
- `:: binary` — binary mode

---

## Classes → Patterns with Identity

**Python:**

```python
class Color:
    def __init__(self, r, g, b):
        self.r = r
        self.g = g
        self.b = b

red = Color(255, 0, 0)
```

**Buran:**

```
# Define type through identity
⟨[rgb: _, _, _]⟩ ↤ [identity:
    "type": "Color",
    "description": "RGB color with three components"
]

# Create instance
[rgb: 255, 0, 0] ↦ red
```

Buran doesn't have classes. Instead:

- Patterns define structure: `[rgb: r, g, b]`
- Identity defines type and metadata
- Pattern matching provides "methods"

### Methods → Pattern-Matching Functions

**Python:**

```python
class Color:
    def brightness(self):
        return (self.r + self.g + self.b) / 3
```

**Buran:**

```
brightness {
    [rgb: 𝑟, 𝑔, 𝑏] ↦ [(𝑟 + 𝑔 + 𝑏) ÷ 3]
}

brightness(red) ↦ result
```

---

## Type Checking

**Python:**

```python
if isinstance(x, int):
    handle_integer(x)
elif isinstance(x, str):
    handle_string(x)
```

**Buran:**

```
process {
    ⟨𝑥 | "type": "integer"⟩ ↦ handle-integer(𝑥)
    ⟨𝑥 | "type": "string"⟩ ↦ handle-string(𝑥)
}
```

Mirror brackets `⟨...⟩` with type guards provide type checking through pattern matching.

---

## Comprehensions → map/filter

**Python:**

```python
squares = [x**2 for x in range(10)]
evens = [x for x in numbers if x % 2 == 0]
pairs = [(x, y) for x in xs for y in ys]
```

**Buran:**

```
map({ 𝑥 ↦ 𝑥² }, [list: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9]) ↦ squares
filter({ 𝑥 | 𝑥 mod 2 = 0 }, numbers) ↦ evens
# Nested comprehensions through composition
```

---

## Mathematical Notation

Buran uses standard mathematical symbols instead of ASCII approximations:

| Python         | Buran         | Meaning               |
| -------------- | ------------- | --------------------- |
| `*`            | `×`           | Multiplication        |
| `/`            | `÷`           | Division              |
| `**`           | `²`, `³`, `ⁿ` | Exponentiation        |
| `math.sqrt(x)` | `√𝑥`          | Square root           |
| `math.pi`      | `𝛑`           | Pi                    |
| `<=`           | `≤`           | Less than or equal    |
| `>=`           | `≥`           | Greater than or equal |
| `!=`           | `≠`           | Not equal             |
| `and`          | `∧`           | Logical AND           |
| `or`           | `∨`           | Logical OR            |
| `not`          | `¬`           | Logical NOT           |
| `in`           | `∈`           | Element of            |
| `True`         | `[✔]`         | Boolean true          |
| `False`        | `[✘]`         | Boolean false         |

### Fractions

**Python:**

```python
from fractions import Fraction
half = Fraction(1, 2)
two_thirds = Fraction(2, 3)
result = half + two_thirds
```

**Buran:**

```
[½] ↦ half
[⅔] ↦ two-thirds
[½ + ⅔] ↦ result    # [7⁄6]
```

Buran has native Unicode fractions and exact rational arithmetic.

---

## String Formatting

**Python:**

```python
name = "World"
message = f"Hello, {name}!"
```

**Buran:**

```
["World"] ↦ name
[string: "Hello, ", name, "!"] ↦ message
```

The `string:` constructor concatenates strings.

---

## None → Empty Pattern

**Python:**

```python
result = None
if result is None:
    handle_empty()
```

**Buran:**

```
[] ↦ result    # empty pattern
result ↦ {
    [] ↦ handle-empty()
    value ↦ process(value)
}
```

The empty pattern `[]` represents absence of value.

---

## Loops → Recursion

Buran has no loops. All iteration is through recursion:

**Python:**

```python
def sum_list(lst):
    total = 0
    for x in lst:
        total += x
    return total
```

**Buran:**

```
sum-list {
    [list: ] ↦ [0]
    [list: head, ⟨tail⟩] ↦ [head + sum-list(tail)]
}
```

For efficiency, use tail recursion with an accumulator:

```
sum-list {
    list ↦ sum-with-acc(list, [0])
}

sum-with-acc {
    [list: ], acc ↦ acc
    [list: head, ⟨tail⟩], acc ↦ sum-with-acc(tail, [acc + head])
}
```

---

## Standard I/O

**Python:**

```python
name = input("Name: ")
print(f"Hello, {name}")
```

**Buran:**

```
["Name: "] ↦ [stdout]
[stdin] ↦ name
[string: "Hello, ", name] ↦ [stdout]
```

---

## Comparison Table

| Concept             | Python                    | Buran                        |
| ------------------- | ------------------------- | ---------------------------- |
| Function definition | `def f(x):`               | `f { pattern ↦ result }`     |
| Function call       | `f(x)`                    | `f(x)`                       |
| Conditional         | `if/elif/else`            | Pattern matching with guards |
| Loop                | `for`, `while`            | Recursion                    |
| Lambda              | `lambda x: x + 1`         | `{ 𝑥 ↦ 𝑥 + 1 }`              |
| List                | `[1, 2, 3]`               | `[list: 1, 2, 3]`            |
| Dict                | `{"a": 1}`                | `[map: "a", 1]`              |
| Set                 | `{1, 2, 3}`               | `[set: 1, 2, 3]`             |
| None                | `None`                    | `[]`                         |
| Boolean             | `True`, `False`           | `[✔]`, `[✘]`                 |
| Type check          | `isinstance(x, int)`      | `⟨𝑥 \| "type": "integer"⟩`   |
| String format       | `f"Hello {x}"`            | `[string: "Hello ", x]`      |
| File read           | `open("f").read()`        | `[File: "f"] ↦ content`      |
| File write          | `open("f", "w").write(d)` | `d ↦ [File: "f"]`            |
| Comment             | `# comment`               | `# comment`                  |

---

## Example: FizzBuzz

**Python:**

```python
def fizzbuzz(n):
    if n % 15 == 0:
        return "FizzBuzz"
    elif n % 3 == 0:
        return "Fizz"
    elif n % 5 == 0:
        return "Buzz"
    else:
        return str(n)
```

**Buran:**

```
fizzbuzz {
    [𝑛 | 𝑛 mod 15 = 0] ↦ ["FizzBuzz"]
    [𝑛 | 𝑛 mod 3 = 0] ↦ ["Fizz"]
    [𝑛 | 𝑛 mod 5 = 0] ↦ ["Buzz"]
    [𝑛] ↦ [string: 𝑛]
}
```

---

## Example: Quicksort

**Python:**

```python
def quicksort(lst):
    if len(lst) <= 1:
        return lst
    pivot = lst[0]
    less = [x for x in lst[1:] if x < pivot]
    greater = [x for x in lst[1:] if x >= pivot]
    return quicksort(less) + [pivot] + quicksort(greater)
```

**Buran:**

```
quicksort {
    [list: ] ↦ [list: ]
    [list: pivot] ↦ [list: pivot]
    [list: pivot, ⟨rest⟩] ↦ [list:
        ⟨quicksort(filter({ 𝑥 | 𝑥 < pivot }, rest))⟩,
        pivot,
        ⟨quicksort(filter({ 𝑥 | 𝑥 ≥ pivot }, rest))⟩
    ]
}
```

---

## Key Advantages of Buran

1. **Pattern Matching Everywhere** — No special `match` statement; the entire language is pattern matching
2. **Mathematical Notation** — Write `𝑥²` instead of `x**2`, `√𝑥` instead of `math.sqrt(x)`
3. **Unicode Identifiers** — Name functions in any language: `факториал`, `階乗`, `مضروب`
4. **Declarative Style** — Describe transformations, not procedures
5. **No Mutation** — Values don't change; patterns transform to new patterns
6. **Built-in I/O Formats** — JSON, YAML, CSV parsing without imports
7. **Type Through Identity** — Metadata-based type system without class hierarchies

---

## Philosophical Difference

Python asks: *"How do I compute this result?"*

Buran asks: *"What pattern does this pattern become?"*

This shift from imperative to declarative makes Buran code:

- Easier for AI systems to generate reliably
- More like mathematical notation
- Self-documenting through pattern structure

If you've enjoyed Python's readability, you may find Buran takes it further — code that reads like the specification of what it does, not instructions for how to do it.

---

*Buran is in development. Specification and reference implementation expected early 2026.*

© 2026 Danslav Slavenskoj
