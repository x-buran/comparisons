# Buran for TypeScript Programmers

## Introduction

If you know TypeScript, you understand the value of types — not just for catching bugs, but for expressing intent. You've used union types, discriminated unions, and type narrowing. These features point toward pattern matching. Buran takes that direction to its logical conclusion: everything is a pattern, and all computation is pattern transformation.

This document will help you transition from TypeScript to Buran by showing correspondences between familiar constructs and new syntax.

---

## Discriminated Unions → Constructors

TypeScript's discriminated unions are the closest thing to Buran's core model:

**TypeScript:**

```typescript
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "rectangle"; width: number; height: number }
  | { kind: "triangle"; a: number; b: number; c: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "rectangle":
      return shape.width * shape.height;
    case "triangle": {
      const s = (shape.a + shape.b + shape.c) / 2;
      return Math.sqrt(s * (s - shape.a) * (s - shape.b) * (s - shape.c));
    }
  }
}
```

**Buran:**

```
area {
    [circle: 𝑟] ↦ [𝛑 × 𝑟²]
    [rectangle: 𝑤, ℎ] ↦ [𝑤 × ℎ]
    [triangle: 𝑎, 𝑏, 𝑐] ↦
        [(𝑎 + 𝑏 + 𝑐) ÷ 2] ↦ 𝑠 ↦
        [√(𝑠 × (𝑠 − 𝑎) × (𝑠 − 𝑏) × (𝑠 − 𝑐))]
}
```

No `kind` discriminator needed. The pattern name *is* the discriminator.

---

## The Core Pattern

TypeScript is about adding types to JavaScript. Buran is about making patterns the only concept:

**TypeScript:**

```typescript
function factorial(n: number): number {
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

No `function`, no type annotations, no `if`, no `return`. Patterns match and transform.

---

## Basic Syntax Correspondences

### Variables and Constants

**TypeScript:**

```typescript
const x: number = 42;
const name: string = "hello";
const items: number[] = [1, 2, 3];
const person: Record<string, unknown> = { name: "Alice", age: 30 };
```

**Buran:**

```
[42] ↦ 𝑥
["hello"] ↦ name
[list: 1, 2, 3] ↦ items
[map: "name", "Alice", "age", 30] ↦ person
```

All values in square brackets. Arrow indicates binding. No type annotations.

### Functions

**TypeScript:**

```typescript
function add(a: number, b: number): number {
  return a + b;
}

const greet = (name: string): string => {
  return `Hello, ${name}!`;
};
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

Functions are pattern-matching blocks. No `function` keyword, no arrow syntax for definition.

### Function Calls

**TypeScript:**

```typescript
const sum = add(2, 3);
const msg = greet("World");
```

**Buran:**

```
add([2], [3]) ↦ sum
greet("World") ↦ msg
```

Literal arguments need brackets. Variables don't.

---

## Type Narrowing → Pattern Matching

TypeScript narrows types through control flow:

**TypeScript:**

```typescript
function process(value: string | number | null) {
  if (value === null) {
    return "nothing";
  }
  if (typeof value === "string") {
    return value.toUpperCase();
  }
  return value * 2;
}
```

**Buran:**

```
process {
    [] ↦ ["nothing"]
    ⟨value | "type": "string"⟩ ↦ uppercase(value)
    ⟨value | "type": "number"⟩ ↦ [value × 2]
}
```

Pattern matching with identity guards replaces type narrowing.

---

## Null and Undefined → Empty Pattern

**TypeScript:**

```typescript
// Null checks
const name: string | null = getName();
if (name !== null) {
  console.log(name.length);
}

// Optional chaining
const length = name?.length ?? 0;

// Optional parameters
function greet(name?: string) {
  return `Hello, ${name ?? "stranger"}!`;
}
```

**Buran:**

```
# Empty pattern represents absence
get-name() ↦ name ↦ {
    [] ↦ []
    n ↦ length(n) ↦ [stdout]
}

# Default through pattern matching
greet {
    [] ↦ [string: "Hello, stranger!"]
    name ↦ [string: "Hello, ", name, "!"]
}
```

No `null`, no `undefined`, no `?`, no `??`. Pattern matching handles all cases.

---

## Interfaces and Types → Identity

**TypeScript:**

```typescript
interface Point {
  x: number;
  y: number;
}

type Color = {
  r: number;
  g: number;
  b: number;
};

const p: Point = { x: 3, y: 4 };
const c: Color = { r: 255, g: 0, b: 0 };
```

**Buran:**

```
# Define types through identity
⟨[point: _, _]⟩ ↤ [identity:
    "type": "Point",
    "fields": ["x", "y"]
]

⟨[rgb: _, _, _]⟩ ↤ [identity:
    "type": "Color",
    "fields": ["r", "g", "b"]
]

[point: 3, 4] ↦ p
[rgb: 255, 0, 0] ↦ c
```

---

## Generics → Implicit Polymorphism

**TypeScript:**

```typescript
function first<T>(items: T[]): T | undefined {
  return items[0];
}

function map<T, U>(items: T[], f: (x: T) => U): U[] {
  return items.map(f);
}

function filter<T>(items: T[], pred: (x: T) => boolean): T[] {
  return items.filter(pred);
}
```

**Buran:**

```
first {
    [list: ] ↦ []
    [list: head, ⟨_⟩] ↦ head
}

map {
    f, [list: ] ↦ [list: ]
    f, [list: head, ⟨tail⟩] ↦ [list: f(head), ⟨map(f, tail)⟩]
}

filter {
    _, [list: ] ↦ [list: ]
    pred, [list: head, ⟨tail⟩] | pred(head) ↦ [list: head, ⟨filter(pred, tail)⟩]
    pred, [list: _, ⟨tail⟩] ↦ filter(pred, tail)
}
```

No `<T>` declarations. Patterns are naturally polymorphic.

---

## Arrow Functions → Pattern Blocks

**TypeScript:**

```typescript
const double = (x: number) => x * 2;
const add = (a: number, b: number) => a + b;
const isPositive = (x: number) => x > 0;

// Multi-line
const classify = (x: number) => {
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

# Multi-clause
{
    [𝑥 | 𝑥 > 0] ↦ ["positive"],
    [𝑥 | 𝑥 < 0] ↦ ["negative"],
    [0] ↦ ["zero"]
} ↦ classify
```

Pattern blocks are Buran's lambdas, but with built-in pattern matching.

---

## Array Methods → Higher-Order Functions

**TypeScript:**

```typescript
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

# Chained (using sequential bindings)
numbers ↦
filter({ 𝑥 | 𝑥 > 0 }, numbers) ↦ positives ↦
map({ 𝑥 ↦ 𝑥 × 2 }, positives) ↦ doubled ↦
fold({ acc, 𝑥 ↦ acc + 𝑥 }, [0], doubled) ↦ result
```

---

## Destructuring → Pattern Matching

**TypeScript:**

```typescript
// Array destructuring
const [first, second, ...rest] = items;

// Object destructuring
const { name, age } = person;
const { x, y } = point;

// Function parameter destructuring
function process({ x, y }: Point) {
  return x + y;
}
```

**Buran:**

```
# List destructuring
[list: first, second, ⟨rest⟩] ↦ items

# Constructor destructuring (in function)
process {
    [point: 𝑥, 𝑦] ↦ [𝑥 + 𝑦]
}
```

The `⟨...⟩` captures remaining elements, like `...rest`.

---

## Spread Operator → Mirror Brackets

**TypeScript:**

```typescript
const newArr = [...arr, 4, 5];
const merged = { ...obj1, ...obj2 };
const copy = [...original];
```

**Buran:**

```
[list: ⟨arr⟩, 4, 5] ↦ new-arr
[map: ⟨obj1⟩, ⟨obj2⟩] ↦ merged
[list: ⟨original⟩] ↦ copy
```

Mirror brackets `⟨...⟩` spread elements into new patterns.

---

## Error Handling

**TypeScript:**

```typescript
function divide(a: number, b: number): number {
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

No `throw`, no `try`/`catch`. Errors are patterns.

### Result Type Pattern

**TypeScript:**

```typescript
type Result<T, E> = { ok: true; value: T } | { ok: false; error: E };

function divide(a: number, b: number): Result<number, string> {
  if (b === 0) return { ok: false, error: "Division by zero" };
  return { ok: true, value: a / b };
}

const result = divide(10, 2);
if (result.ok) {
  console.log(result.value);
} else {
  console.error(result.error);
}
```

**Buran:**

```
divide {
    𝑎, [0] ↦ [error: division by zero]
    𝑎, 𝑏 ↦ [𝑎 ÷ 𝑏]
}

divide([10], [2]) ↦ {
    [error: e] ↦ e ↦ [stderr]
    value ↦ value ↦ [stdout]
}
```

Buran's approach is simpler — errors are just patterns, not wrapped types.

---

## Async/Await → Not Yet Specified

**TypeScript:**

```typescript
async function fetchData(url: string): Promise<Data> {
  const response = await fetch(url);
  return response.json();
}

const data = await fetchData("https://api.example.com");
```

Buran's concurrency model is not yet specified. For now, I/O is synchronous:

```
[File: "data.json" :: json] ↦ data
```

---

## Classes → Patterns and Functions

**TypeScript:**

```typescript
class Counter {
  private count: number = 0;

  increment(): void {
    this.count++;
  }

  decrement(): void {
    this.count--;
  }

  getCount(): number {
    return this.count;
  }
}

const counter = new Counter();
counter.increment();
console.log(counter.getCount()); // 1
```

**Buran:**

```
# No classes — data is immutable
# Model state transformations as functions

increment {
    [counter: 𝑛] ↦ [counter: 𝑛 + 1]
}

decrement {
    [counter: 𝑛] ↦ [counter: 𝑛 − 1]
}

get-count {
    [counter: 𝑛] ↦ 𝑛
}

# Usage — each operation returns new state
[counter: 0] ↦ c ↦
increment(c) ↦ c ↦
get-count(c) ↦ [stdout]    # [1]
```

Immutable transformations instead of mutable state.

---

## File I/O

**TypeScript:**

```typescript
import { readFileSync, writeFileSync } from 'fs';

const content = readFileSync('data.txt', 'utf-8');
writeFileSync('output.txt', data);

// JSON
const config = JSON.parse(readFileSync('config.json', 'utf-8'));
writeFileSync('output.json', JSON.stringify(data));
```

**Buran:**

```
[File: "data.txt"] ↦ content
data ↦ [File: "output.txt"]

# JSON (built-in)
[File: "config.json" :: json] ↦ config
data ↦ [File: "output.json" :: json]
```

---

## Mathematical Notation

TypeScript uses JavaScript's ASCII operators. Buran uses mathematical symbols:

| TypeScript         | Buran         | Meaning               |
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

| Concept          | TypeScript            | Buran                     |
| ---------------- | --------------------- | ------------------------- |
| Function         | `function f(x: T): U` | `f { pattern ↦ result }`  |
| Arrow function   | `(x) => x + 1`        | `{ 𝑥 ↦ 𝑥 + 1 }`           |
| Interface        | `interface I { }`     | Identity metadata         |
| Union type       | `A \| B \| C`         | Different constructors    |
| Generic          | `<T>`                 | Implicit polymorphism     |
| Array            | `[1, 2, 3]`           | `[list: 1, 2, 3]`         |
| Object           | `{ a: 1, b: 2 }`      | `[map: "a", 1, "b", 2]`   |
| Spread           | `...arr`              | `⟨arr⟩`                   |
| Destructure      | `const [a, b] = arr`  | `[list: a, b] ↦ arr`      |
| Optional         | `?:`                  | Pattern with `[]`         |
| Null/undefined   | `null`, `undefined`   | `[]`                      |
| true/false       | `true`, `false`       | `[✔]`, `[✘]`              |
| Try/catch        | `try { } catch { }`   | `[error: ...]` patterns   |
| Template literal | `` `Hello ${x}` ``    | `[string: "Hello ", x]`   |
| Type guard       | `if (isString(x))`    | `⟨x \| "type": "string"⟩` |
| Type assertion   | `as Type`             | Identity guards           |

---

## Example: API Response Handling

**TypeScript:**

```typescript
type ApiResponse<T> =
  | { status: "success"; data: T }
  | { status: "error"; message: string }
  | { status: "loading" };

function handleResponse<T>(
  response: ApiResponse<T>,
  onSuccess: (data: T) => void,
  onError: (msg: string) => void
): void {
  switch (response.status) {
    case "success":
      onSuccess(response.data);
      break;
    case "error":
      onError(response.message);
      break;
    case "loading":
      console.log("Loading...");
      break;
  }
}
```

**Buran:**

```
handle-response {
    [success: data], on-success, _ ↦ on-success(data)
    [error: message], _, on-error ↦ on-error(message)
    [loading], _, _ ↦ ["Loading..."] ↦ [stdout]
}
```

---

## Example: State Reducer

**TypeScript:**

```typescript
type State = { count: number; name: string };
type Action =
  | { type: "increment" }
  | { type: "decrement" }
  | { type: "setName"; payload: string };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case "increment":
      return { ...state, count: state.count + 1 };
    case "decrement":
      return { ...state, count: state.count - 1 };
    case "setName":
      return { ...state, name: action.payload };
  }
}
```

**Buran:**

```
reducer {
    [state: count, name], [increment] ↦ [state: count + 1, name]
    [state: count, name], [decrement] ↦ [state: count − 1, name]
    [state: count, _], [set-name: new-name] ↦ [state: count, new-name]
}
```

---

## Example: Type-Safe Builder

**TypeScript:**

```typescript
class QueryBuilder<T extends object> {
  private query: Partial<T> = {};

  where<K extends keyof T>(key: K, value: T[K]): this {
    this.query[key] = value;
    return this;
  }

  build(): Partial<T> {
    return { ...this.query };
  }
}

const query = new QueryBuilder<User>()
  .where("name", "Alice")
  .where("age", 30)
  .build();
```

**Buran:**

```
# Just build the pattern directly
[query: "name", "Alice", "age", 30]

# Or use a builder function
add-where {
    query, key, value ↦ [query: ⟨query⟩, key, value]
}

[query: ] ↦
add-where(query, "name", "Alice") ↦ query ↦
add-where(query, "age", 30) ↦ query
```

---

## What Buran Trades Away

Coming from TypeScript, you should know what Buran doesn't provide:

1. **Compile-time type checking** — No TypeScript-style static analysis
2. **IDE tooling** — No autocomplete, hover types, refactoring (yet)
3. **npm ecosystem** — No package manager (yet)
4. **Async/await** — Concurrency not yet specified
5. **Familiar syntax** — No JavaScript heritage
6. **Gradual adoption** — Can't mix with existing JS/TS

## What Buran Offers Instead

1. **Uniform syntax** — Everything is pattern transformation
2. **No null/undefined confusion** — Empty pattern is explicit
3. **Built-in pattern matching** — No switch statement needed
4. **Mathematical notation** — Write `𝑥²` not `x ** 2`
5. **Immutability everywhere** — No accidental mutations
6. **AI-friendly** — Consistent structure for code generation

---

## Philosophical Difference

TypeScript asks: *"What type does this value have?"*

Buran asks: *"What pattern does this pattern become?"*

TypeScript adds types to JavaScript to catch errors early. Buran replaces JavaScript entirely with a different model — one where types emerge from pattern structure rather than being declared.

If you've enjoyed TypeScript's discriminated unions and type narrowing, you've been practicing Buran's philosophy. Buran is what you get when pattern matching becomes the foundation, not a feature built on top.

---

*Buran is in development. Specification and reference implementation expected early 2026.*

© 2026 Danslav Slavenskoj
