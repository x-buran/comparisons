# Buran for JavaScript Programmers

## Introduction

If you know JavaScript, you know flexibility. You can write code almost any way you want — functions, classes, prototypes, callbacks, promises. JavaScript adapts to your style. Buran takes the opposite approach: one way to do everything, and that way is pattern transformation.

You also know JavaScript's quirks — `==` vs `===`, truthy/falsy, `this` binding, `undefined` vs `null`, hoisting. Buran has none of these. It trades JavaScript's flexibility for predictability.

This document will help you transition from JavaScript to Buran by showing correspondences between familiar constructs and new syntax.

---

## The Core Difference

JavaScript is imperative and multi-paradigm. Buran is purely declarative:

**JavaScript:**

```javascript
function factorial(n) {
  if (n === 0) return 1;
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

No `function`, no `if`, no `return`, no `===`. You declare what patterns become, and computation happens.

---

## No JavaScript Quirks

Buran eliminates JavaScript's gotchas:

| JavaScript Quirk      | Buran                     |
| --------------------- | ------------------------- |
| `==` vs `===`         | Only `=` (exact match)    |
| truthy/falsy          | Only `[✔]` and `[✘]`      |
| `null` vs `undefined` | Only `[]` (empty pattern) |
| `this` binding        | No `this`                 |
| Hoisting              | No hoisting               |
| `var`/`let`/`const`   | Just `↦` binding          |
| Type coercion         | No coercion               |
| `NaN !== NaN`         | No NaN quirks             |
| `[] + []` = `""`      | No weird coercions        |

---

## Basic Syntax Correspondences

### Variables

**JavaScript:**

```javascript
let x = 42;
const name = "hello";
var items = [1, 2, 3];
const person = { name: "Alice", age: 30 };
```

**Buran:**

```
[42] ↦ 𝑥
["hello"] ↦ name
[list: 1, 2, 3] ↦ items
[map: "name", "Alice", "age", 30] ↦ person
```

All values are in square brackets. No `let`/`const`/`var` — just `↦` for binding. Everything is immutable.

### Functions

**JavaScript:**

```javascript
function add(a, b) {
  return a + b;
}

const greet = (name) => {
  return `Hello, ${name}!`;
};

// Or shorter
const greet = name => `Hello, ${name}!`;
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

One syntax for all functions. No `function`, no `=>`, no `return`.

### Function Calls

**JavaScript:**

```javascript
const sum = add(2, 3);
const msg = greet("World");
```

**Buran:**

```
add([2], [3]) ↦ sum
greet("World") ↦ msg
```

Literal arguments need brackets: `add([2], [3])`. Variables don't: `add(𝑥, 𝑦)`.

---

## No `this`

JavaScript's `this` is notoriously confusing:

**JavaScript:**

```javascript
const obj = {
  value: 42,
  getValue() {
    return this.value;
  },
  getValueArrow: () => {
    return this.value; // Wrong `this`!
  }
};

const getValue = obj.getValue;
console.log(getValue()); // undefined — lost `this`

// Fix with bind
const boundGetValue = obj.getValue.bind(obj);
```

**Buran:**

```
# No this — data and functions are separate
[object: 42] ↦ obj

get-value {
    [object: value] ↦ value
}

get-value(obj) ↦ result    # [42] — always works
```

Functions take data explicitly. No binding issues, no arrow function gotchas.

---

## null and undefined → Empty Pattern

JavaScript has two kinds of "nothing":

**JavaScript:**

```javascript
let a = null;
let b = undefined;
let c;  // implicitly undefined

// Checking for both
if (value != null) {  // Catches both null and undefined
  process(value);
}

// Optional chaining
const name = user?.profile?.name ?? "Unknown";
```

**Buran:**

```
# One kind of nothing: empty pattern
[] ↦ a

# Pattern matching handles absence
value ↦ {
    [] ↦ []    # Nothing case
    v ↦ process(v)
}

# Nested access through pattern matching
get-name {
    [user: [profile: [name: n, ⟨_⟩], ⟨_⟩], ⟨_⟩] ↦ n
    _ ↦ ["Unknown"]
}
```

No `null`, no `undefined`, no `?.`, no `??`. Just pattern matching.

---

## Truthy/Falsy → Explicit Booleans

**JavaScript:**

```javascript
// All falsy
if (0) { }
if ("") { }
if (null) { }
if (undefined) { }
if (NaN) { }
if (false) { }

// Truthy
if ([]) { }       // Empty array is truthy!
if ({}) { }       // Empty object is truthy!
if ("false") { }  // String "false" is truthy!
```

**Buran:**

```
# Only two boolean values
[✔]    # true
[✘]    # false

# Everything else is not a boolean
# Use explicit checks
is-empty {
    [list: ] ↦ [✔]
    _ ↦ [✘]
}

is-positive {
    [𝑛 | 𝑛 > 0] ↦ [✔]
    _ ↦ [✘]
}
```

No implicit coercion. You say exactly what you mean.

---

## == vs === → Just =

**JavaScript:**

```javascript
// The horror
0 == ""           // true
0 == "0"          // true
"" == "0"         // false
false == "false"  // false
false == "0"      // true
null == undefined // true

// Always use ===
0 === ""          // false
```

**Buran:**

```
# Just pattern matching — no coercion
[0] = [""]        # No — different patterns
[0] = [0]         # Yes — same pattern
```

Pattern matching is structural. No type coercion, no surprises.

---

## Arrays → Lists

**JavaScript:**

```javascript
const arr = [1, 2, 3];
const first = arr[0];
const rest = arr.slice(1);
arr.push(4);              // Mutates!
const newArr = [...arr, 5]; // Immutable way
```

**Buran:**

```
[list: 1, 2, 3] ↦ arr

# Destructuring
[list: first, ⟨rest⟩] ↦ arr

# Always immutable
[list: ⟨arr⟩, 4] ↦ arr2
[list: ⟨arr⟩, 5] ↦ arr3
```

No mutation. The `⟨...⟩` captures remaining elements or spreads into new patterns.

---

## Array Methods → Higher-Order Functions

**JavaScript:**

```javascript
const doubled = numbers.map(x => x * 2);
const evens = numbers.filter(x => x % 2 === 0);
const sum = numbers.reduce((acc, x) => acc + x, 0);

// Chained
const result = numbers
  .filter(x => x > 0)
  .map(x => x * 2)
  .reduce((acc, x) => acc + x, 0);
```

**Buran:**

```
map({ 𝑥 ↦ 𝑥 × 2 }, numbers) ↦ doubled
filter({ 𝑥 | 𝑥 mod 2 = 0 }, numbers) ↦ evens
fold({ acc, 𝑥 ↦ acc + 𝑥 }, [0], numbers) ↦ sum

# Chained
numbers ↦
filter({ 𝑥 | 𝑥 > 0 }, numbers) ↦ positives ↦
map({ 𝑥 ↦ 𝑥 × 2 }, positives) ↦ doubled ↦
fold({ acc, 𝑥 ↦ acc + 𝑥 }, [0], doubled) ↦ result
```

Pattern blocks `{ ... }` replace arrow functions.

---

## Objects → Maps and Constructors

**JavaScript:**

```javascript
const person = {
  name: "Alice",
  age: 30
};

console.log(person.name);
console.log(person["name"]);

// Destructuring
const { name, age } = person;
```

**Buran:**

```
# As a map (key-value pairs)
[map: "name", "Alice", "age", 30] ↦ person

# As a constructor (structured data)
[person: "Alice", 30] ↦ p

# Access through pattern matching
get-name {
    [person: name, _] ↦ name
}
```

Buran separates the concepts: maps for dynamic key-value data, constructors for structured data.

---

## Destructuring and Spread

**JavaScript:**

```javascript
// Array destructuring
const [first, second, ...rest] = items;

// Object destructuring
const { name, age, ...rest } = person;

// Spread
const newArr = [...arr1, ...arr2];
const newObj = { ...obj1, ...obj2 };
```

**Buran:**

```
# List destructuring
[list: first, second, ⟨rest⟩] ↦ items

# Constructor destructuring
[person: name, age] ↦ p

# Spread with mirror brackets
[list: ⟨arr1⟩, ⟨arr2⟩] ↦ new-arr
[map: ⟨obj1⟩, ⟨obj2⟩] ↦ new-map
```

---

## Arrow Functions → Pattern Blocks

**JavaScript:**

```javascript
const double = x => x * 2;
const add = (a, b) => a + b;
const isPositive = x => x > 0;

// Multi-line
const process = x => {
  if (x > 0) return "positive";
  if (x < 0) return "negative";
  return "zero";
};
```

**Buran:**

```
{ 𝑥 ↦ 𝑥 × 2 } ↦ double
{ 𝑎, 𝑏 ↦ 𝑎 + 𝑏 } ↦ add
{ 𝑥 | 𝑥 > 0 } ↦ is-positive

# Multi-clause (no if/else needed)
{
    [𝑥 | 𝑥 > 0] ↦ ["positive"],
    [𝑥 | 𝑥 < 0] ↦ ["negative"],
    [0] ↦ ["zero"]
} ↦ process
```

Pattern blocks can have multiple clauses — built-in switch/case.

---

## Closures

**JavaScript:**

```javascript
function makeCounter() {
  let count = 0;
  return {
    increment() { count++; return count; },
    decrement() { count--; return count; },
    get() { return count; }
  };
}

const counter = makeCounter();
counter.increment(); // 1
counter.increment(); // 2
```

**Buran:**

```
# No mutable closures — use immutable transformations
increment {
    [counter: 𝑛] ↦ [counter: 𝑛 + 1]
}

decrement {
    [counter: 𝑛] ↦ [counter: 𝑛 − 1]
}

get-count {
    [counter: 𝑛] ↦ 𝑛
}

# Thread state through operations
[counter: 0] ↦ c ↦
increment(c) ↦ c ↦
increment(c) ↦ c ↦
get-count(c) ↦ result    # [2]
```

State is explicit, not hidden in closures.

---

## Classes → Patterns

**JavaScript:**

```javascript
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  distance() {
    return Math.sqrt(this.x ** 2 + this.y ** 2);
  }

  add(other) {
    return new Point(this.x + other.x, this.y + other.y);
  }
}

const p = new Point(3, 4);
console.log(p.distance()); // 5
```

**Buran:**

```
# Data is just a pattern
[point: 3, 4] ↦ p

# Methods are functions
distance {
    [point: 𝑥, 𝑦] ↦ [√(𝑥² + 𝑦²)]
}

add-points {
    [point: 𝑥₁, 𝑦₁], [point: 𝑥₂, 𝑦₂] ↦ [point: 𝑥₁ + 𝑥₂, 𝑦₁ + 𝑦₂]
}

distance(p) ↦ result    # [5]
```

No `class`, no `constructor`, no `new`, no `this`.

---

## Prototypes → Pattern Matching

**JavaScript:**

```javascript
// Prototype chain
const animal = {
  speak() { return "..."; }
};

const dog = Object.create(animal);
dog.speak = function() { return "Woof!"; };

const cat = Object.create(animal);
cat.speak = function() { return "Meow!"; };
```

**Buran:**

```
# Pattern matching handles polymorphism
speak {
    [dog: _] ↦ ["Woof!"]
    [cat: _] ↦ ["Meow!"]
    _ ↦ ["..."]
}
```

No prototype chain. Pattern matching dispatches based on structure.

---

## Error Handling

**JavaScript:**

```javascript
function divide(a, b) {
  if (b === 0) {
    throw new Error("Division by zero");
  }
  return a / b;
}

try {
  const result = divide(10, 0);
  process(result);
} catch (e) {
  console.error(`Error: ${e.message}`);
}
```

**Buran:**

```
divide {
    𝑎, [0] ↦ [error: division by zero]
    𝑎, 𝑏 ↦ [𝑎 ÷ 𝑏]
}

divide([10], [0]) ↦ {
    [error: msg] ↦ [string: "Error: ", msg] ↦ [stderr]
    result ↦ process(result)
}
```

No `throw`, no `try`/`catch`. Errors are just patterns to match.

---

## Promises and Async → Not Yet Specified

**JavaScript:**

```javascript
fetch("https://api.example.com")
  .then(response => response.json())
  .then(data => process(data))
  .catch(error => console.error(error));

// Or with async/await
async function getData() {
  try {
    const response = await fetch("https://api.example.com");
    const data = await response.json();
    return process(data);
  } catch (error) {
    console.error(error);
  }
}
```

Buran's concurrency model is not yet specified. For now, operations are synchronous.

---

## Template Literals → String Constructor

**JavaScript:**

```javascript
const name = "Alice";
const age = 30;
const message = `Hello, ${name}! You are ${age} years old.`;
```

**Buran:**

```
["Alice"] ↦ name
[30] ↦ age
[string: "Hello, ", name, "! You are ", age, " years old."] ↦ message
```

---

## Conditionals and Loops

**JavaScript:**

```javascript
// If-else
if (x > 0) {
  result = "positive";
} else if (x < 0) {
  result = "negative";
} else {
  result = "zero";
}

// Ternary
const result = x > 0 ? "positive" : "non-positive";

// For loop
for (let i = 0; i < 10; i++) {
  process(i);
}

// For-of
for (const item of items) {
  process(item);
}

// While
while (condition) {
  // ...
}
```

**Buran:**

```
# Pattern matching replaces all conditionals
classify {
    [𝑥 | 𝑥 > 0] ↦ ["positive"]
    [𝑥 | 𝑥 < 0] ↦ ["negative"]
    [0] ↦ ["zero"]
}

# Map replaces for-of
map({ item ↦ process(item) }, items)

# Recursion replaces for/while
process-range {
    [0] ↦ []
    [𝑛] ↦ process([𝑛 − 1]) ↦ _ ↦ process-range([𝑛 − 1])
}
```

No `if`, no `for`, no `while`, no ternary. Pattern matching and recursion only.

---

## File and Console I/O

**JavaScript (Node.js):**

```javascript
const fs = require('fs');

// Read
const content = fs.readFileSync('data.txt', 'utf-8');

// Write
fs.writeFileSync('output.txt', data);

// Console
console.log("Hello");
console.error("Error!");

// JSON
const config = JSON.parse(fs.readFileSync('config.json'));
```

**Buran:**

```
# Read
[File: "data.txt"] ↦ content

# Write
data ↦ [File: "output.txt"]

# Console
["Hello"] ↦ [stdout]
["Error!"] ↦ [stderr]

# JSON (built-in)
[File: "config.json" :: json] ↦ config
```

---

## Mathematical Notation

JavaScript uses ASCII. Buran uses mathematical symbols:

| JavaScript         | Buran         | Meaning               |
| ------------------ | ------------- | --------------------- |
| `*`                | `×`           | Multiplication        |
| `/`                | `÷`           | Division              |
| `**`               | `²`, `³`, `ⁿ` | Exponentiation        |
| `Math.sqrt(x)`     | `√𝑥`          | Square root           |
| `Math.PI`          | `𝛑`           | Pi                    |
| `<=`               | `≤`           | Less than or equal    |
| `>=`               | `≥`           | Greater than or equal |
| `!==`              | `≠`           | Not equal             |
| `&&`               | `∧`           | Logical AND           |
| `\|\|`             | `∨`           | Logical OR            |
| `!`                | `¬`           | Logical NOT           |
| `true`             | `[✔]`         | Boolean true          |
| `false`            | `[✘]`         | Boolean false         |
| `null`/`undefined` | `[]`          | Empty                 |

---

## Comparison Table

| Concept          | JavaScript            | Buran                     |
| ---------------- | --------------------- | ------------------------- |
| Variable         | `let x = 1`           | `[1] ↦ x`                 |
| Constant         | `const x = 1`         | `[1] ↦ x` (all immutable) |
| Function         | `function f(x) { }`   | `f { pattern ↦ result }`  |
| Arrow function   | `x => x + 1`          | `{ 𝑥 ↦ 𝑥 + 1 }`           |
| Array            | `[1, 2, 3]`           | `[list: 1, 2, 3]`         |
| Object           | `{ a: 1 }`            | `[map: "a", 1]`           |
| Destructure      | `const [a, b] = arr`  | `[list: a, b] ↦ arr`      |
| Spread           | `[...arr, 4]`         | `[list: ⟨arr⟩, 4]`        |
| Template literal | `` `Hi ${x}` ``       | `[string: "Hi ", x]`      |
| null             | `null`                | `[]`                      |
| undefined        | `undefined`           | `[]`                      |
| true/false       | `true`, `false`       | `[✔]`, `[✘]`              |
| Equality         | `===`                 | `=` (pattern match)       |
| If/else          | `if (x) { } else { }` | Pattern matching          |
| For loop         | `for (...)`           | `map()` or recursion      |
| Try/catch        | `try { } catch { }`   | `[error: ...]` patterns   |
| Class            | `class C { }`         | Patterns + functions      |
| This             | `this.x`              | Explicit parameter        |
| Prototype        | `Object.create()`     | Pattern matching          |

---

## Example: FizzBuzz

**JavaScript:**

```javascript
for (let i = 1; i <= 100; i++) {
  if (i % 15 === 0) {
    console.log("FizzBuzz");
  } else if (i % 3 === 0) {
    console.log("Fizz");
  } else if (i % 5 === 0) {
    console.log("Buzz");
  } else {
    console.log(i);
  }
}
```

**Buran:**

```
fizzbuzz {
    [𝑛 | 𝑛 mod 15 = 0] ↦ ["FizzBuzz"]
    [𝑛 | 𝑛 mod 3 = 0] ↦ ["Fizz"]
    [𝑛 | 𝑛 mod 5 = 0] ↦ ["Buzz"]
    [𝑛] ↦ [string: 𝑛]
}

map({ 𝑛 ↦ fizzbuzz([𝑛]) }, range([1], [100]))
```

---

## Example: Event Handler Dispatch

**JavaScript:**

```javascript
function handleEvent(event) {
  switch (event.type) {
    case "click":
      return handleClick(event.target);
    case "submit":
      return handleSubmit(event.data);
    case "keydown":
      return handleKeydown(event.key);
    default:
      return null;
  }
}
```

**Buran:**

```
handle-event {
    [click: target] ↦ handle-click(target)
    [submit: data] ↦ handle-submit(data)
    [keydown: key] ↦ handle-keydown(key)
    _ ↦ []
}
```

---

## Example: Data Transformation Pipeline

**JavaScript:**

```javascript
const result = users
  .filter(u => u.active)
  .map(u => ({
    name: u.name.toUpperCase(),
    email: u.email
  }))
  .reduce((acc, u) => {
    acc[u.email] = u.name;
    return acc;
  }, {});
```

**Buran:**

```
users ↦
filter({ [user: _, _, active] | active }, users) ↦ active-users ↦
map({ [user: name, email, _] ↦ [entry: uppercase(name), email] }, active-users) ↦ entries ↦
fold({ acc, [entry: name, email] ↦ [map: ⟨acc⟩, email, name] }, [map: ], entries) ↦ result
```

---

## What Buran Trades Away

Coming from JavaScript, you should know what Buran doesn't provide:

1. **Flexibility** — One way to do things, not many
2. **npm ecosystem** — No package manager (yet)
3. **Browser/DOM** — Not designed for web UIs
4. **Async/await** — Concurrency not yet specified
5. **Mutation** — Everything is immutable
6. **Familiar syntax** — Completely different from JS

## What Buran Offers Instead

1. **No quirks** — No `==` vs `===`, no truthy/falsy, no `this`
2. **Predictability** — Same code always does the same thing
3. **Immutability** — No accidental mutations
4. **Pattern matching** — Built into every function
5. **Mathematical notation** — Write `𝑥²` not `x ** 2`
6. **AI-friendly** — Consistent structure for code generation

---

## Philosophical Difference

JavaScript asks: *"How can I make this work?"*

Buran asks: *"What pattern does this pattern become?"*

JavaScript is pragmatic — it lets you do almost anything, almost any way. This flexibility is powerful but leads to inconsistency, bugs, and the famous "WAT" moments.

Buran is principled — there's one concept (pattern transformation) applied uniformly. You trade JavaScript's flexibility for predictability and clarity.

If you've ever been frustrated by JavaScript's quirks, Buran offers a language where the rules are simple and consistent. No more surprises.

---

*Buran is in development. Specification and reference implementation expected early 2026.*

© 2026 Danslav Slavenskoj
